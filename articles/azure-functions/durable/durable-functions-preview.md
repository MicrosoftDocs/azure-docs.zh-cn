---
title: Durable Functions 预览功能-Azure Functions
description: 了解 Durable Functions 用于预览功能。
services: functions
author: cgillum
manager: jeconnoc
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.date: 04/23/2019
ms.author: azfuncdf
ms.openlocfilehash: 6b3b49049ea1ed36a08fad9619183017b0f07d99
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077735"
---
# <a name="durable-functions-20-preview-azure-functions"></a>Durable Functions 2.0 预览版 (Azure Functions)

Durable Functions 是 [Azure Functions](../functions-overview.md) 和 [Azure WebJobs](../../app-service/web-sites-create-web-jobs.md) 的扩展，可用于在无服务器环境中编写有状态函数。 该扩展可用于管理状态、检查点和重启。 如果你尚不熟悉 Durable Functions，请参阅[概述文档](durable-functions-overview.md)。

Durable Functions 是 Azure Functions 的 GA （正式版） 功能，但是也包含多个当前处于公共预览状态的子功能。 本文介绍新发布的预览版功能，并进入有关它们的工作原理以及如何开始使用它们的详细信息。

> [!NOTE]
> 这些预览功能是 Durable Functions 2.0 版本中，这是当前的一部分**alpha 质量的发行版**与几个重大更改。 Azure Functions Durable 扩展包生成可以找到在 nuget.org 中的窗体中的版本与**2.0.0-alpha**。 这些内部版本不是适用于任何生产工作负荷，并且后续版本可能包含其他重大更改。

## <a name="breaking-changes"></a>重大变化

Durable Functions 2.0 中介绍了几个重大更改。 现有的应用程序不应为无需更改代码与 Durable Functions 2.0 兼容。 本部分列出了一些更改：

### <a name="dropping-net-framework-support"></a>删除.NET Framework 的支持

Durable Functions 2.0 为已删除适用于.NET Framework （和 Functions 1.0） 的支持。 主要原因是为了启用非 Windows contributors （参与者） 轻松构建和测试它们对 Durable Functions 做通过 macOS 和 Linux 平台的更改。 辅助的原因是为了帮助鼓励开发人员迁移到 Azure Functions 运行时的最新版本。

### <a name="hostjson-schema"></a>Host.json 架构

以下代码片段显示了 host.json 的新架构。 需要注意的我们的主要更改的新`"storageProvider"`部分中，和`"azureStorage"`其下面的部分。 此更改为了支持[备用存储提供程序](durable-functions-preview.md#alternate-storage-providers)。

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "azureStorage": {
          "connectionStringName": <string>,
          "controlQueueBatchSize": <int?>,
          "partitionCount": <int?>,
          "controlQueueVisibilityTimeout": <hh:mm:ss?>,
          "workItemQueueVisibilityTimeout": <hh:mm:ss?>,
          "trackingStoreConnectionStringName": <string?>,
          "trackingStoreNamePrefix": <string?>,
          "maxQueuePollingInterval": <hh:mm:ss?>
        }
      },
      "maxConcurrentActivityFunctions": <int?>,
      "maxConcurrentOrchestratorFunctions": <int?>,
      "traceInputAndOutputs": <bool?>,
      "eventGridTopicEndpoint": <string?>,
      "eventGridKeySettingName": <string?>,
      "eventGridPublishRetryCount": <string?>,
      "eventGridPublishRetryInterval": <hh:mm:ss?>,
      "eventGridPublishRetryHttpStatus": <int[]?>,
      "eventgridPublishEventTypes": <string[]?>,
      "customLifeCycleNotificationHelperType"
      "extendedSessionsEnabled": <bool?>,
      "extendedSessionIdleTimeoutInSeconds": <int?>,
      "logReplayEvents": <bool?>
  }
}
```

随着 Durable Functions 2.0 持续稳定，会向介绍更多更改`durableTask`部分 host.json。 有关这些更改的详细信息，请参阅[此 GitHub 问题](https://github.com/Azure/azure-functions-durable-extension/issues/641)。

### <a name="public-interface-changes"></a>公共接口的更改

Durable Functions 支持的各种"上下文"对象必须适合在单元测试中使用的抽象基类。 Durable Functions 2.0 的一部分，这些抽象基类的类已替换为与接口。 函数代码直接使用的具体类型不受影响。

下表显示主要更改：

| 旧类型 | 新的类型 |
|----------|----------|
| DurableOrchestrationClientBase | IDurableOrchestrationClient |
| DurableOrchestrationContextBase | IDurableOrchestrationContext |
| DurableActivityContextBase | IDurableActivityContext |

其中的抽象基类包含虚方法的情况下，这些虚拟方法已替换为中定义的扩展方法通过`DurableContextExtensions`。

## <a name="entity-functions"></a>实体函数

实体函数定义操作用于读取和更新的状态，称为少量*持久实体*。 业务流程协调程序函数，如实体函数是使用特殊触发器类型和函数*实体触发器*。 与业务流程协调程序函数，不同实体函数不具有任何特定代码约束。 实体函数还管理状态而隐式表示通过控制流的状态不是显式。

下面的代码是定义一个简单实体函数的示例*计数器*实体。 函数定义三个操作`add`， `remove`，并`reset`，则每个的其中的更新整数值， `currentValue`。

```csharp
public static async Task Counter(
    [EntityTrigger(EntityName = "Counter")] IDurableEntityContext ctx)
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

实体*实例*的唯一标识符，通过访问*实体 ID*。 实体 ID 是只需一对字符串唯一标识实体实例。 该环境包括：

1. **实体名称**： 用于标识的实体类型 （例如，"计数器"） 的名称
2. **实体键**： 唯一标识的实体相同的名称 (例如，GUID) 的所有其他实体之间的字符串

例如，*计数器*实体函数可能会用于保存在制作联机游戏的分数。 游戏的每个实例将具有唯一的实体 ID，如`@Counter@Game1`， `@Counter@Game2`，依次类推。

### <a name="comparison-with-virtual-actors"></a>与虚拟执行组件的比较

持久的实体的设计大程度上受到[actor 模型](https://en.wikipedia.org/wiki/Actor_model)。 如果你已熟悉执行组件，则应为你所熟悉持久实体的概念。 特别是，持久的实体是类似于[虚拟执行组件](https://research.microsoft.com/en-us/projects/orleans/)在许多方面：

* 持久的实体进行寻址通过*实体 ID*。
* 持久实体操作按顺序执行一次，以防止争用条件的其中一个。
* 在调用或发出信号时，将自动创建持久的实体。
* 不执行操作，持久实体时，以无提示方式从内存中卸载。

有一些重要差异，但是，一些值得一提：

* 持久实体被建模为纯函数。 这种设计是不同的表示使用特定于语言的支持类、 属性和方法的执行组件的大多数面向对象的框架。
* 持久的实体设置优先级*持久性*转移*延迟*，并因此不可能适用于具有严格的延迟要求的应用程序。
* 实体之间发送的消息传送可靠地按顺序。
* 持久的实体可以与持久业务流程，结合使用，并且可作为分布式锁，在本文后面部分所述。
* 在实体中的请求/响应模式被限制为业务流程。 对于实体到实体的通信，只有单向消息 （也称为"信号"） 允许使用，如原始的执行组件模型中所示。 此行为可防止分布式死锁。

### <a name="durable-entity-apis"></a>持久实体 Api

实体支持涉及到几个 Api。 其中一个，没有一个新的 API 用于定义实体函数，如上所示，指定实体上调用操作时应发生。 此外，对客户端和业务流程的现有 Api 已更新了新的功能来与实体交互。

### <a name="implementing-entity-operations"></a>实现实体操作

实体上的操作的执行上下文对象上可以调用这些成员 (`IDurableEntityContext`在.NET 中):

* **OperationName**： 获取操作的名称。
* **GetInput\<T >**： 获取操作的输入。
* **GetState\<T >**： 获取实体的当前状态。
* **SetState**： 更新实体的状态。
* **SignalEntity**： 向实体发送单向消息。
* **自助**： 获取实体的 ID。
* **返回**： 将值返回到客户端或调用操作的业务流程。
* **IsNewlyConstructed**： 返回`true`如果实体不存在操作前。
* **DestructOnExit**： 完成该操作后删除实体。

较低的限制比业务流程进行的操作：

* 操作可以调用外部 I/O，使用同步或异步 Api （我们建议使用仅异步的）。
* 操作可能会不确定。 例如，它是安全地调用`DateTime.UtcNow`，`Guid.NewGuid()`或`new Random()`。

### <a name="accessing-entities-from-clients"></a>从客户端访问的实体

可以通过普通函数从调用持久实体`orchestrationClient`绑定 (`IDurableOrchestrationClient`在.NET 中)。 支持以下方法：

* **ReadEntityStateAsync\<T >**： 读取实体的状态。
* **SignalEntityAsync**： 将单向消息发送到实体，并等待它要排队。

这些方法优先于一致性性能：`ReadEntityStateAsync`可以返回过时的值，和`SignalEntityAsync`之前在操作完成后，可以返回。 与此相反，从业务流程调用实体 （按下文所述） 是强一致性。

### <a name="accessing-entities-from-orchestrations"></a>从业务流程中访问的实体

业务流程可以访问使用上下文对象的实体。 他们可以选择单向通信 （发后不理） 和双向通信 （请求和响应）。 相应的方法

* **SignalEntity**： 向实体发送单向消息。
* **CallEntityAsync**： 将消息发送到实体，并等待响应，指示操作已完成。
* **CallEntityAsync\<T >**： 将消息发送到实体，并等待包含 T 类型的结果的响应

在使用双向通信时，在执行操作期间引发的任何异常也传输回调用业务流程和重新引发。 与此相反，当使用即发即弃，异常不会观察到。

### <a name="locking-entities-from-orchestrations"></a>从业务流程的锁定实体

业务流程可以锁定的实体。 此功能提供了简单方法来通过阻止不需要的资源争用*临界区*。

上下文对象提供了以下方法：

* **LockAsync**： 一个或多个实体上获得锁。
* **IsLocked**： 返回在当前关键部分中，如果 true false 否则为。

关键部分结束，并且所有锁都被释放，业务流程结束时。 在.NET 中，`LockAsync`将返回`IDisposable`结束关键节时释放，可用于与`using`子句，以获取关键节的语法表示形式。

例如，考虑需要测试两个玩家是否可用，业务流程，然后为游戏同时分配。 可以使用关键节，如下所示来实现此任务：

```csharp

EntityId player1 = /* ... */;
EntityId player2 = /* ... */;

using (await ctx.LockAsync(player1, player2))
{
    bool available1 = await ctx.CallEntityAsync<bool>(player1, "is-available");
    bool available2 = await ctx.CallEntityAsync<bool>(player2, "is-available");

    if (available1 && available2)
    {
        Guid gameId = ctx.NewGuid();

        await ctx.CallEntityAsync(player1, "assign-game", gameId);
        await ctx.CallEntityAsync(player2, "assign-game", gameId);
    }
}
```

在关键节中，这两个播放器实体会被锁定，这意味着它们不执行任何操作而不是从内部的关键部分调用的）。 此行为可防止与相冲突的操作，如赋给一个不同的播放器关闭游戏，或签名。

我们对多个限制如何关键部分上可以使用。 这些限制用于防止死锁和可重入性。

* 关键节无法嵌套。
* 临界区无法创建 suborchestrations。
* 临界区可以调用仅已锁定的实体。
* 临界区不能调用相同的实体使用多个并行调用。
* 临界区，可以发出只有它们未锁定的实体。

## <a name="alternate-storage-providers"></a>备用存储提供程序

Durable Task Framework 多个存储提供程序目前支持，包括[Azure 存储](https://github.com/Azure/durabletask/tree/master/src/DurableTask.AzureStorage)， [Azure 服务总线](https://github.com/Azure/durabletask/tree/master/src/DurableTask.ServiceBus)，则[内存中仿真程序](https://github.com/Azure/durabletask/tree/master/src/DurableTask.Emulator)，和是试验性[Redis](https://github.com/Azure/durabletask/tree/redis/src/DurableTask.Redis)提供程序。 但是，到目前为止，适用于 Azure Functions 的 Durable Task 扩展仅支持 Azure 存储提供程序。 Durable 函数从 2.0 开始，支持备用存储提供程序添加的开始使用 Redis 提供程序。

> [!NOTE]
> Durable Functions 2.0 仅支持.NET Standard 2.0 兼容的提供程序。 在撰写本文时，Azure 服务总线提供程序不支持.NET Standard 2.0，并因此不能作为备用存储提供程序。

### <a name="emulator"></a>仿真程序

[DurableTask.Emulator](https://www.nuget.org/packages/Microsoft.Azure.DurableTask.Emulator/)提供程序是一种本地内存、 非持久存储提供程序适用于本地测试方案。 可以配置使用以下最小**host.json**架构：

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "emulator": { }
      }
    }
  }
}
```

### <a name="redis-experimental"></a>Redis （实验性）

[DurableTask.Redis](https://www.nuget.org/packages/Microsoft.Azure.DurableTask.Redis/)提供程序保存到配置 Redis 群集的所有业务流程状态。

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": <string>,
      "storageProvider": {
        "redis": {
          "connectionStringName": <string>,
        }
      }
    }
  }
}
```

`connectionStringName`必须引用的应用程序设置或环境变量的名称。 该应用程序设置或环境变量应包含的窗体中的 Redis 连接字符串值*服务器： 端口*。 例如，`localhost:6379`用于连接到本地的 Redis 群集。

> [!NOTE]
> Redis 提供程序当前处于实验阶段，仅支持单个节点上运行的 function app。