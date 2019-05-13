---
title: Azure Functions 中的 Durable Functions 模式和技术概念
description: 了解 Azure Functions 中的 Durable Functions 扩展如何在云中提供有状态代码执行优势。
services: functions
author: ggailey777
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: azfuncdf
ms.openlocfilehash: 95ec6a863f951a8c26abd865041c68df333a4e38
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65071348"
---
# <a name="durable-functions-patterns-and-technical-concepts-azure-functions"></a>Durable Functions 模式和技术概念 (Azure Functions)

Durable Functions 是 [Azure Functions](../functions-overview.md) 和 [Azure WebJobs](../../app-service/web-sites-create-web-jobs.md) 的扩展。 可以使用 Durable Functions 在无服务器环境中编写有状态函数。 该扩展可用于管理状态、检查点和重启。 

本文提供有关 Azure Functions 的 Durable Functions 扩展的行为和常用实现模式的详细信息。 这些信息可帮助你确定如何使用 Durable Functions 来解决开发难题。

> [!NOTE]
> Durable Functions 是 Azure Functions 的高级扩展，并不适用于所有应用程序。 本文假设读者非常熟悉 [Azure Functions](../functions-overview.md) 中的概念，以及无服务器应用程序开发的难题。

## <a name="patterns"></a>模式

本部分介绍一些可以使用 Durable Functions 的常见应用程序模式。

### <a name="chaining"></a>模式 #1：函数链

在函数链模式中，会按特定的顺序执行一系列函数。 在此模式中，一个函数的输出将应用到另一函数的输入。

![函数链模式的示意图](./media/durable-functions-concepts/function-chaining.png)

可以使用 Durable Functions 精简实现函数链模式，如以下示例所示：

#### <a name="c-script"></a>C# 脚本

```csharp
public static async Task<object> Run(DurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1");
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // Error handling or compensation goes here.
    }
}
```

> [!NOTE]
> 使用 C# 编写预编译的持久函数，与使用本示例中所示的 C# 脚本编写预编译的持久函数存在细微的差别。 在 C# 预编译函数中，必须使用相应的属性来修饰持久参数。 例如，使用 `[OrchestrationTrigger]` 属性修饰 `DurableOrchestrationContext` 参数。 在 C# 预编译持久函数中，如果未正确修饰参数，则运行时无法将变量注入该函数，并且会出现错误。 有关更多示例，请参阅 [GitHub 上的 azure-functions-durable-extension 示例](https://github.com/Azure/azure-functions-durable-extension/blob/master/samples)。

#### <a name="javascript-functions-2x-only"></a>JavaScript（仅限 Functions 2.x）

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const x = yield context.df.callActivity("F1");
    const y = yield context.df.callActivity("F2", x);
    const z = yield context.df.callActivity("F3", y);
    return yield context.df.callActivity("F4", z);
});
```

在此示例中，值 `F1`、`F2`、`F3` 和 `F4` 是函数应用中其他函数的名称。 可以使用一般命令性编码构造来实现控制流。 代码按从上到下的顺序执行。 代码可能涉及现有的语言控制流语义，例如条件语句和循环语句。 可在 `try`/`catch`/`finally` 块中包含错误处理逻辑。

可以使用 `context` 参数 [DurableOrchestrationContext] \(.NET\) 和 `context.df` 对象 (JavaScript) 按名称调用其他函数、传递参数并返回函数输出。 每当代码调用 `await` (C#) 或 `yield` (JavaScript) 时，Durable Functions 框架都会对当前函数实例的进度执行检查点操作。 如果在执行中途回收进程或 VM，则函数实例从上一个 `await` 或 `yield` 调用继续执行。 有关详细信息，请参阅下一部分“模式 #2：扇出/扇入”。

> [!NOTE]
> JavaScript 中的 `context` 对象表示整个[函数上下文](../functions-reference-node.md#context-object)，而不仅仅表示 [DurableOrchestrationContext] 参数。

### <a name="fan-in-out"></a>模式 #2：扇出/扇入

在扇出/扇入模式中，可以并行执行多个函数，然后等待所有函数完成。 通常会对这些函数返回的结果执行一些聚合操作。

![扇出/扇入模式的示意图](./media/durable-functions-concepts/fan-out-fan-in.png)

对于一般函数，可通过使函数向某个队列发送多条消息来完成扇出。 扇入回来的难度要大得多。 若要扇入，可在一般函数中编写代码，以跟踪队列触发的函数何时结束，然后存储函数输出。 

Durable Functions 扩展使用相对简单的代码处理此模式：

#### <a name="c-script"></a>C# 脚本

```csharp
public static async Task Run(DurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Get a list of N work items to process in parallel.
    object[] workBatch = await context.CallActivityAsync<object[]>("F1");
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript（仅限 Functions 2.x）

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const parallelTasks = [];

    // Get a list of N work items to process in parallel.
    const workBatch = yield context.df.callActivity("F1");
    for (let i = 0; i < workBatch.length; i++) {
        parallelTasks.push(context.df.callActivity("F2", workBatch[i]));
    }

    yield context.df.Task.all(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    const sum = parallelTasks.reduce((prev, curr) => prev + curr, 0);
    yield context.df.callActivity("F3", sum);
});
```

扇出操作将分散到 `F2` 函数的多个实例。 使用动态任务列表跟踪这些操作。 调用 .NET `Task.WhenAll` API 或 JavaScript `context.df.Task.all` API 等待所有调用的函数完成。 然后，从动态任务列表聚合 `F2` 函数输出，并将这些输出传递给 `F3` 函数。

在针对 `Task.WhenAll` 或 `context.df.Task.all` 调用 `await` 或 `yield` 时自动执行的检查点操作确保中途可能出现的任何崩溃或重新启动无需重启已完成的任务。

### <a name="async-http"></a>模式 #3：异步 HTTP API

异步 HTTP API 模式可解决使用外部客户端协调长时间运行的操作状态时出现的问题。 实现此模式的一种常用方式是让 HTTP 调用触发长时间运行的操作。 然后，将客户端重定向到某个状态终结点，客户端可轮询该终结点，以了解操作是何时完成的。

![HTTP API 模式的示意图](./media/durable-functions-concepts/async-http-api.png)

Durable Functions 提供内置 API，用于简化为与长时间运行的函数执行进行交互而编写的代码。 Durable Functions 快速入门示例（[C#](durable-functions-create-first-csharp.md) 和 [JavaScript](quickstart-js-vscode.md)）演示了可用于启动新业务流程协调程序函数实例的简单 REST 命令。 启动实例后，该扩展会公开 Webhook HTTP API 用于查询业务流程协调程序函数的状态。 

以下示例演示用于启动业务流程协调程序和查询其状态的 REST 命令。 为清楚起见，实例中省略了某些细节。

```
> curl -X POST https://myfunc.azurewebsites.net/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2017-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/admin/extensions/DurableTaskExtension/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2017-03-16T21:20:57Z", ...}
```

因为 Durable Functions 运行时会管理状态，因此你无需实现自己的状态跟踪机制。

Durable Functions 扩展提供内置的 Webhook 用于管理长时间运行的业务流程。 你可以使用自己的函数触发器（例如 HTTP、队列或 Azure 事件中心）和 `orchestrationClient` 绑定来自行实现此模式。 例如，可以使用队列消息触发终止。 或者，可以使用受 Azure Active Directory 身份验证策略保护的 HTTP 触发器，而不是使用利用生成的密钥进行身份验证的内置 Webhook。

以下示例演示如何使用 HTTP API 模式：

#### <a name="c"></a>C#

```csharp
// An HTTP-triggered function starts a new orchestrator function instance.
public static async Task<HttpResponseMessage> Run(
    HttpRequestMessage req,
    DurableOrchestrationClient starter,
    string functionName,
    ILogger log)
{
    // The function name comes from the request URL.
    // The function input comes from the request content.
    dynamic eventData = await req.Content.ReadAsAsync<object>();
    string instanceId = await starter.StartNewAsync(functionName, eventData);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript（仅限 Functions 2.x）

```javascript
// An HTTP-triggered function starts a new orchestrator function instance.
const df = require("durable-functions");

module.exports = async function (context, req) {
    const client = df.getClient(context);

    // The function name comes from the request URL.
    // The function input comes from the request content.
    const eventData = req.body;
    const instanceId = await client.startNew(req.params.functionName, undefined, eventData);

    context.log(`Started orchestration with ID = '${instanceId}'.`);

    return client.createCheckStatusResponse(req, instanceId);
};
```

在 .NET 中，[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) `starter` 参数是 `orchestrationClient` 输出绑定中的值，该绑定是 Durable Functions 扩展的一部分。 在 JavaScript 中，此对象是通过调用 `df.getClient(context)` 返回的。 这些对象提供了可用于启动、向其发送事件、终止和查询新的或现有的业务流程协调程序函数实例的方法。

在前面的示例中，HTTP 触发的函数采用传入的 URL 中的 `functionName` 值，并将该值传递给 [StartNewAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_StartNewAsync_)。 然后，[CreateCheckStatusResponse](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_CreateCheckStatusResponse_System_Net_Http_HttpRequestMessage_System_String_) 绑定 API 返回包含 `Location` 标头和有关实例的其他信息的响应。 稍后可以使用这些信息来查找已启动实例的状态或终止实例。

### <a name="monitoring"></a>模式 #4：监视

监视模式是指工作流中某个灵活的重复性过程。 例如，不断轮询，直到满足特定的条件为止。 可以使用常规[计时器触发器](../functions-bindings-timer.md)解决简单方案（例如定期清理作业），但该方案的间隔是静态的，并且管理实例生存期会变得复杂。 可以使用 Durable Functions 创建灵活的重复间隔、管理任务生存期，以及从单个业务流程创建多个监视过程。

监视模式的一个例子是反转前面所述的异步 HTTP API 方案。 监视模式不会公开终结点供外部客户端监视长时间运行的操作，而是让长时间运行的监视器使用外部终结点，然后等待某个状态发生更改。

![监视模式的示意图](./media/durable-functions-concepts/monitor.png)

只需编写少量的代码行，即可使用 Durable Functions 创建多个监视器来观察任意终结点。 满足某个条件时，监视器可以结束执行；或者，[DurableOrchestrationClient](durable-functions-instance-management.md) 可以终止监视器。 可以根据特定的条件（例如指数退避）更改监视器的 `wait` 间隔。 

以下代码实现一个基本的监视器：

#### <a name="c-script"></a>C# 脚本

```csharp
public static async Task Run(DurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript（仅限 Functions 2.x）

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const jobId = context.df.getInput();
    const pollingInternal = getPollingInterval();
    const expiryTime = getExpiryTime();

    while (moment.utc(context.df.currentUtcDateTime).isBefore(expiryTime)) {
        const jobStatus = yield context.df.callActivity("GetJobStatus", jobId);
        if (jobStatus === "Completed") {
            // Perform an action when a condition is met.
            yield context.df.callActivity("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        const nextCheck = moment.utc(context.df.currentUtcDateTime).add(pollingInterval, 's');
        yield context.df.createTimer(nextCheck.toDate());
    }

    // Perform more work here, or let the orchestration end.
});
```

收到请求时，会为该作业 ID 创建新的业务流程实例。 该实例会一直轮询状态，直到满足条件退出循环。 持久计时器控制轮询间隔。 然后可以执行其他操作，或者可以结束业务流程。 当 `context.CurrentUtcDateTime` (.NET) 或 `context.df.currentUtcDateTime` (JavaScript) 超出 `expiryTime` 值时，监视器将会终止。

### <a name="human"></a>模式 #5：人机交互

许多自动化过程涉及到某种人机交互。 自动化过程中涉及的人机交互非常棘手，因为人的可用性和响应能力不如云服务那样高。 自动化过程可以使用超时和补偿逻辑来实现此目的。

审批过程就是涉及到人机交互的业务过程的一个例子。 经理进行审批可能需要超过特定金额的费用报表。 如果经理未在 72 小时内审批该开支报表（经理可能正在度假），则会启动上报过程，让其他某人（可能是经理的经理）审批。

![人机交互模式的示意图](./media/durable-functions-concepts/approval.png)

在此示例中，可以使用业务流程协调程序函数实现该模式。 业务流程协调程序使用[持久计时器](durable-functions-timers.md)请求审批。 如果发生超时，业务流程协调程序会将事务上报。 业务流程协调程序等待发生某个[外部事件](durable-functions-external-events.md)，例如，人机交互生成的通知。

这些示例创建一个审批过程来演示人机交互模式：

#### <a name="c-script"></a>C# 脚本

```csharp
public static async Task Run(DurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval");
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate");
        }
    }
}
```

#### <a name="javascript-functions-2x-only"></a>JavaScript（仅限 Functions 2.x）

```javascript
const df = require("durable-functions");
const moment = require('moment');

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("RequestApproval");

    const dueTime = moment.utc(context.df.currentUtcDateTime).add(72, 'h');
    const durableTimeout = context.df.createTimer(dueTime.toDate());

    const approvalEvent = context.df.waitForExternalEvent("ApprovalEvent");
    if (approvalEvent === yield context.df.Task.any([approvalEvent, durableTimeout])) {
        durableTimeout.cancel();
        yield context.df.callActivity("ProcessApproval", approvalEvent.result);
    } else {
        yield context.df.callActivity("Escalate");
    }
});
```

若要创建持久计时器，请调用 `context.CreateTimer` (.NET) 或 `context.df.createTimer` (JavaScript)。 通知由 `context.WaitForExternalEvent` (.NET) 或 `context.df.waitForExternalEvent` (JavaScript) 接收。 然后，调用 `Task.WhenAny` (.NET) 或 `context.df.Task.any` (JavaScript) 来确定是上报（首先发生超时）还是处理审批（超时前收到审批）。

外部客户端可以使用[内置 HTTP API](durable-functions-http-api.md#raise-event) 或通过另一个函数使用 [DurableOrchestrationClient.RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_System_String_System_String_System_Object_) API 将事件通知传递给正在等待的业务流程协调程序函数：

```csharp
public static async Task Run(string instanceId, DurableOrchestrationClient client)
{
    bool isApproved = true;
    await client.RaiseEventAsync(instanceId, "ApprovalEvent", isApproved);
}
```

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const isApproved = true;
    await client.raiseEvent(instanceId, "ApprovalEvent", isApproved);
};
```

## <a name="pattern-6-aggregator-preview"></a>模式 #6:聚合器 （预览版）

第六个模式是有关一段时间内的事件数据聚合到单个可寻址*实体*。 在此模式下，处于聚合过程中的数据可能来自多个源、 可能会发送批，或可能分散在长时间段内的时间。 聚合器可能需要事件数据执行操作，如到达，并且外部客户端可能需要进行查询的聚合的数据。

![聚合器关系图](./media/durable-functions-concepts/aggregator.png)

比较棘手的尝试来实现此模式处理普通问题，无状态函数是并发控制将成为一个巨大的挑战。 不仅需要担心多个线程同时修改相同的数据，还需要考虑如何确保聚合器仅在运行在单个 VM 上一次。

使用[持久实体函数](durable-functions-preview.md#entity-functions)，其中一个可以实现此模式轻松地作为单个函数。

```csharp
public static async Task Counter(
    [EntityTrigger(EntityClassName = "Counter")] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    int operand = ctx.GetInput<int>();

    switch (ctx.OperationName)
    {
        case "add":
            currentValue += operand;
            break;
        case "subtract":
            currentValue -= operand;
            break;
        case "reset":
            await SendResetNotificationAsync();
            currentValue = 0;
            break;
    }

    ctx.SetState(currentValue);
}
```

客户端可以排入队列*operations* （也称为"信号"） 的实体函数使用`orchestrationClient`绑定。

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static async Task Run(
    [EventHubTrigger("device-sensor-events")] EventData eventData,
    [OrchestrationClient] IDurableOrchestrationClient entityClient)
{
    var metricType = (string)eventData.Properties["metric"];
    var delta = BitConverter.ToInt32(eventData.Body, eventData.Body.Offset);

    // The "Counter/{metricType}" entity is created on-demand.
    var entityId = new EntityId("Counter", metricType);
    await entityClient.SignalEntityAsync(entityId, "add", delta);
}
```

同样，客户端可以查询的使用方法的实体函数状态`orchestrationClient`绑定。

> [!NOTE]
> 实体函数目前仅在适用[Durable Functions 2.0 预览版](durable-functions-preview.md)。

## <a name="the-technology"></a>技术

在幕后，Durable Functions 扩展构建在 [Durable Task Framework](https://github.com/Azure/durabletask)（GitHub 上的用于生成持久任务业务流程的开源库）的基础之上。 如同 Azure Functions 是 Azure WebJobs 的无服务器演进一样，Durable Functions 是 Durable Task Framework 的无服务器演进。 Microsoft 和其他组织广泛使用 Durable Task Framework 来自动处理任务关键型过程。 它天生就很适合无服务器 Azure Functions 环境。

### <a name="event-sourcing-checkpointing-and-replay"></a>事件溯源、检查点和重播

业务流程协调程序函数通过使用可靠地维护其执行状态[事件溯源](https://docs.microsoft.com/azure/architecture/patterns/event-sourcing)设计模式。 Durable Functions 扩展使用仅限追加的存储来记录函数业务流程执行的一系列完整操作，而不是直接存储业务流程的当前状态。 与“转储”完整的运行时状态相比，仅限追加的存储具有诸多优势。 优势包括提升性能、可伸缩性和响应能力。 此外，还可以确保事务数据的最终一致性，保持完整的审核线索和历史记录。 审核线索支持可靠的补偿操作。

Durable Functions 以透明方式使用事件溯源。 在幕后，业务流程协调程序函数中的 `await` (C#) 或 `yield` (JavaScript) 运算符将对业务流程协调程序线程的控制权让回给 Durable Task Framework 调度程序。 然后，该调度程序向存储提交业务流程协调程序函数计划的任何新操作（如调用一个或多个子函数或计划持久计时器）。 透明的提交操作会追加到业务流程实例的执行历史记录中。 历史记录存储在存储表中。 然后，提交操作向队列添加消息，以计划实际工作。 此时，可从内存中卸载业务流程协调程序函数。 

如果使用 Azure Functions 消耗计划会停止业务流程协调程序函数的计费。 当没有更多工作要做，函数会重新启动，并且其状态并重新构造。

如果业务流程函数需要执行其他工作（例如，收到响应消息或持久计时器过期），业务流程协调程序将唤醒并从头开始重新执行整个函数，以重新生成本地状态。 

在重放期间，如果代码尝试调用某个函数（或执行任何其他异步工作），Durable Task Framework 会查询当前业务流程的执行历史记录。 如果该扩展发现[活动函数](durable-functions-types-features-overview.md#activity-functions)已执行并已生成某种结果，则会重放该函数的结果，并且业务流程协调程序代码会继续运行。 在函数代码完成或计划了新的异步工作之前，重放会一直继续。

### <a name="orchestrator-code-constraints"></a>业务流程协调程序代码约束

业务流程协调程序代码的重放行为针对可在业务流程协调程序函数中编写的代码类型创建约束。 例如，业务流程协调程序代码必须具有确定性，因为该代码将重放多次，每次必须生成相同的结果。 有关约束的完整列表，请参阅[业务流程协调程序代码约束](durable-functions-checkpointing-and-replay.md#orchestrator-code-constraints)。

## <a name="monitoring-and-diagnostics"></a>监视和诊断

Durable Functions 扩展可自动生成为结构化的跟踪数据[Application Insights](../functions-monitoring.md)如果 function app 设置使用 Azure Application Insights 检测密钥。 跟踪数据可用于监视操作和业务流程的进度。

下面是示例的跟踪事件 Durable Functions 在 Application Insights 门户中使用时[Application Insights Analytics](../../application-insights/app-insights-analytics.md):

![Application Insights 查询结果](./media/durable-functions-concepts/app-insights-1.png)

您可以找到有用的结构化的数据中`customDimensions`字段中每个日志条目。 下面是项的一个完全展开的示例：

![Application Insights 查询中的 customDimensions 字段](./media/durable-functions-concepts/app-insights-2.png)

由于 Durable Task Framework 调度程序的重播行为，用户应该会看到重播操作的冗余日志项目。 冗余日志项目可以帮助您了解核心引擎的重播行为。 [诊断](durable-functions-diagnostics.md)文章显示了筛选出重播日志，以便您可以看到只需"实时"日志的示例查询。

## <a name="storage-and-scalability"></a>存储和可伸缩性

Durable Functions 扩展使用 Azure 存储中的队列、表和 Blob 来持久保存执行历史记录状态和触发函数执行。 可以使用函数应用的默认存储帐户，也可以配置单独的存储帐户。 由于存储吞吐量存在限制，你可能需要配置单独的帐户。 编写的业务流程协调程序代码不会与这些存储帐户中的实体进行交互。 Durable Task Framework 直接将实体作为实现详细信息进行管理。

业务流程协调程序函数通过内部队列消息计划活动函数和接收这些函数的响应。 当函数应用在 Azure Functions 消耗计划中，在运行时[Azure Functions 缩放控制器](../functions-scale.md#how-the-consumption-and-premium-plans-work)监视这些队列。 根据需要添加了新的计算实例。 横向扩展到多个 VM 后，业务流程协调程序函数可在一个 VM 上运行，它调用的活动函数可在多个不同的 VM 上运行。 有关 Durable Functions 的缩放行为的详细信息，请参阅[性能和缩放](durable-functions-perf-and-scale.md)。

业务流程协调程序帐户的执行历史记录存储在表存储中。 每当某个实例在特定的 VM 上解冻时，业务流程协调程序会从表存储中获取该实例的执行历史记录，以便可以重新生成其本地状态。 在表存储中获取历史记录所带来的一项便利是，可以使用 [Azure 存储资源管理器](../../vs-azure-tools-storage-manage-with-storage-explorer.md)等工具查看业务流程的历史记录。

存储 Blob 主要用作一种租用机制，用于协调跨多个 VM 的业务流程实例的横向扩展。 存储 Blob 可以保存无法直接存储在表或队列中的大型消息的数据。

![Azure 存储资源管理器的屏幕截图](./media/durable-functions-concepts/storage-explorer.png)

> [!NOTE]
> 尽管可以在表存储中轻松查看执行历史记录，但请不要对此表有任何依赖。 该表可能会随着 Durable Functions 扩展的演变而发生变化。

## <a name="known-issues"></a>已知问题

应在 [GitHub 问题](https://github.com/Azure/azure-functions-durable-extension/issues)列表中跟踪所有已知问题。 如果遇到 GitHub 中未列出的问题，请提出新的问题。 请详细描述问题。

## <a name="next-steps"></a>后续步骤

若要详细了解 Durable Functions，请参阅 [Durable Functions 函数类型和功能](durable-functions-types-features-overview.md)。 

开始操作：

> [!div class="nextstepaction"]
> [创建第一个 Durable Function](durable-functions-create-first-csharp.md)

[DurableOrchestrationContext]: https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html
