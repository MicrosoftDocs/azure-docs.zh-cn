---
title: Azure Functions 的 Azure 队列存储绑定
description: 了解如何在 Azure Functions 中使用 Azure 队列存储触发器和输出绑定。
services: functions
documentationcenter: na
author: craigshoemaker
manager: jeconnoc
keywords: Azure Functions，函数，事件处理，动态计算，无服务体系结构
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: cc996988-fb4f-47
ms.openlocfilehash: 72e51deb31ad2894ccfc0fc71884c99863184f5b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203664"
---
# <a name="azure-queue-storage-bindings-for-azure-functions"></a>Azure Functions 的 Azure 队列存储绑定

本文介绍如何在 Azure Functions 中使用 Azure 队列存储绑定。 Azure Functions 支持对队列使用触发器和输出绑定。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>包 - Functions 1.x

[Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet 包 2.x 版中提供了队列存储绑定。 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Queue) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="packages---functions-2x"></a>包 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 包 3.x 版中提供了队列存储绑定。 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="encoding"></a>编码
函数需要 base64 编码字符串。 对编码类型进行的任何调整（若要将数据作为 base64 编码字符串进行准备）需要在调用服务中实现。

## <a name="trigger"></a>触发器

在队列中收到新项时，可以使用队列触发器启动一个函数。 队列消息作为输入提供给该函数。

## <a name="trigger---example"></a>触发器 - 示例

参阅语言特定的示例：

* [C#](#trigger---c-example)
* [C# 脚本 (.csx)](#trigger---c-script-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

### <a name="trigger---c-example"></a>触发器 - C# 示例

以下示例演示可在每次处理某个队列项之后轮询 `myqueue-items` 队列并写入日志的 [C# 函数](functions-dotnet-class-library.md)。

```csharp
public static class QueueFunctions
{
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
}
```

### <a name="trigger---c-script-example"></a>触发器 - C# 脚本示例

以下示例演示 function.json 文件中的一个队列触发器绑定以及使用该绑定的 [C# 脚本 (.csx)](functions-reference-csharp.md) 代码。 每次处理某个队列项之后，该函数会轮询 `myqueue-items` 队列并写入日志。

function.json 文件如下所示：

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

[配置](#trigger---configuration)部分解释了这些属性。

C# 脚本代码如下所示：

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, 
    DateTimeOffset expirationTime, 
    DateTimeOffset insertionTime, 
    DateTimeOffset nextVisibleTime,
    string queueTrigger,
    string id,
    string popReceipt,
    int dequeueCount,
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}\n" +
        $"queueTrigger={queueTrigger}\n" +
        $"expirationTime={expirationTime}\n" +
        $"insertionTime={insertionTime}\n" +
        $"nextVisibleTime={nextVisibleTime}\n" +
        $"id={id}\n" +
        $"popReceipt={popReceipt}\n" + 
        $"dequeueCount={dequeueCount}");
}
```

[用法](#trigger---usage)部分解释了 function.json 中的 `name` 属性命名的 `myQueueItem`。  [消息元数据部分](#trigger---message-metadata)解释了所有其他所示变量。

### <a name="trigger---javascript-example"></a>触发器 - JavaScript 示例

以下示例演示 *function.json* 文件中的一个队列触发器绑定以及使用该绑定的 [JavaScript 函数](functions-reference-node.md)。 每次处理某个队列项之后，该函数会轮询 `myqueue-items` 队列并写入日志。

function.json 文件如下所示：

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

[配置](#trigger---configuration)部分解释了这些属性。

> [!NOTE]
> name 参数在 JavaScript 代码中反映为 `context.bindings.<name>`，其中包含队列项有效负载。 此有效负载也作为第二个参数传递给函数。

JavaScript 代码如下所示：

```javascript
module.exports = async function (context, message) {
    context.log('Node.js queue trigger function processed work item', message);
    // OR access using context.bindings.<name>
    // context.log('Node.js queue trigger function processed work item', context.bindings.myQueueItem);
    context.log('expirationTime =', context.bindingData.expirationTime);
    context.log('insertionTime =', context.bindingData.insertionTime);
    context.log('nextVisibleTime =', context.bindingData.nextVisibleTime);
    context.log('id =', context.bindingData.id);
    context.log('popReceipt =', context.bindingData.popReceipt);
    context.log('dequeueCount =', context.bindingData.dequeueCount);
    context.done();
};
```

[用法](#trigger---usage)部分解释了 function.json 中的 `name` 属性命名的 `myQueueItem`。  [消息元数据部分](#trigger---message-metadata)解释了所有其他所示变量。

### <a name="trigger---java-example"></a>触发器 - Java 示例

以下 Java 示例显示了一个存储队列触发器函数，该函数用于记录放入队列 `myqueuename` 的触发消息。

 ```java
 @FunctionName("queueprocessor")
 public void run(
    @QueueTrigger(name = "msg",
                   queueName = "myqueuename",
                   connection = "myconnvarname") String message,
     final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
 ```

## <a name="trigger---attributes"></a>触发器 - 特性

在 [C# 类库](functions-dotnet-class-library.md)，请使用以下属性来配置队列触发器：

* [QueueTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/QueueTriggerAttribute.cs)

  该特性的构造函数采用要监视的队列的名称，如以下示例中所示：

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items")] string myQueueItem, 
      ILogger log)
  {
      ...
  }
  ```

  可以设置 `Connection` 属性来指定要使用的存储帐户，如以下示例中所示：

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem, 
      ILogger log)
  {
      ....
  }
  ```

  有关完整示例，请参阅[触发器 - C# 示例](#trigger---c-example)。

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  提供另一种方式来指定要使用的存储帐户。 构造函数采用包含存储连接字符串的应用设置的名称。 可以在参数、方法或类级别应用该特性。 以下示例演示类级别和方法级别：

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("QueueTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

要使用的存储帐户按以下顺序确定：

* `QueueTrigger` 特性的 `Connection` 属性。
* 作为 `QueueTrigger` 特性应用到同一参数的 `StorageAccount` 特性。
* 应用到函数的 `StorageAccount` 特性。
* 应用到类的 `StorageAccount` 特性。
* “AzureWebJobsStorage”应用设置。

## <a name="trigger---configuration"></a>触发器 - 配置

下表解释了在 function.json 文件和 `QueueTrigger` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
|类型 | 不适用| 必须设置为 `queueTrigger`。 在 Azure 门户中创建触发器时，会自动设置此属性。|
|direction| 不适用 | 只能在 *function.json* 文件中设置。 必须设置为 `in`。 在 Azure 门户中创建触发器时，会自动设置此属性。 |
|**name** | 不适用 |函数代码中包含队列项有效负载的变量的名称。  |
|**queueName** | **QueueName**| 要轮询的队列的名称。 |
|**连接** | **Connection** |包含要用于此绑定的存储连接字符串的应用设置的名称。 如果应用设置名称以“AzureWebJobs”开始，则只能在此处指定该名称的余下部分。 例如，如果将 `connection` 设置为“MyStorage”，函数运行时将会查找名为“AzureWebJobsMyStorage”的应用设置。 如果将 `connection` 留空，函数运行时将使用名为 `AzureWebJobsStorage` 的应用设置中的默认存储连接字符串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>触发器 - 用法

在 C# 和 C# 脚本中，可以使用 `string paramName` 等方法参数访问消息数据。 在 C# 脚本中，`paramName` 是在 *function.json* 的 `name` 属性中指定的值。 可以绑定到以下任何类型：

* 对象 - Functions 运行时将 JSON 负载反序列化为代码中定义的任意类的实例。 
* `string`
* `byte[]`
* [CloudQueueMessage]

如果尝试绑定到 `CloudQueueMessage` 并收到错误消息，请确保已引用[正确的存储 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在 JavaScript 中，可以使用 `context.bindings.<name>` 访问队列项有效负载。 如果有效负载为 JSON，则会将它反序列化为对象。

## <a name="trigger---message-metadata"></a>触发器 - 消息元数据

[队列触发器提供了数个元数据属性。](./functions-bindings-expressions-patterns.md#trigger-metadata) 这些属性可在其他绑定中用作绑定表达式的一部分，或者用作代码中的参数。 以下是 [CloudQueueMessage](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage) 类的属性。

|属性|Type|描述|
|--------|----|-----------|
|`QueueTrigger`|`string`|队列有效负载（如果是有效的字符串）。 如果队列消息有效负载是字符串，则 `QueueTrigger` 包含的值与 *function.json* 中 `name` 属性命名的变量的值相同。|
|`DequeueCount`|`int`|此消息取消排队的次数。|
|`ExpirationTime`|`DateTimeOffset`|消息过期的时间。|
|`Id`|`string`|队列消息 ID。|
|`InsertionTime`|`DateTimeOffset`|消息添加到队列的时间。|
|`NextVisibleTime`|`DateTimeOffset`|消息下一次可见的时间。|
|`PopReceipt`|`string`|消息的 pop 接收方。|

## <a name="trigger---poison-messages"></a>触发器 - 有害消息

队列触发函数失败时，Azure Functions 会针对给定的队列消息重试该函数最多 5 次（包括第一次尝试）。 如果 5 次尝试全部失败，函数运行时会将一条消息添加到名为 *&lt;originalqueuename>-poison* 的队列。 可以编写一个函数来处理有害队列中的消息，并记录这些消息，或者发送需要注意的通知。

若要手动处理有害消息，请检查队列消息的 [dequeueCount](#trigger---message-metadata)。

## <a name="trigger---polling-algorithm"></a>触发器 - 轮询算法

队列触发器实现了随机指数退让算法，以降低空闲队列轮询对存储交易成本造成的影响。  当找到消息时，运行时将等待两秒钟，然后检查另一条消息；如果未找到消息，它将等待大约四秒，然后重试。 如果后续尝试获取队列消息失败，则等待时间会继续增加，直到达到最长等待时间（默认为 1 分钟）。 可以通过 [host.json 文件](functions-host-json.md#queues)中的 `maxPollingInterval` 属性配置最大等待时间。

## <a name="trigger---concurrency"></a>触发器 - 并发

当有多个队列消息在等待时，队列触发器会检索一批消息并以并发方式调用函数实例来处理它们。 默认情况下，批大小为 16。 当处理的数量下降到 8 时，运行时可获取另一个批，并开始处理这些消息。 因此，单个虚拟机 (VM) 上每个函数处理的最大并发消息数是 24。 此限制分别应用于每个 VM 上各个队列触发的函数。 如果你的函数应用横向扩展到多个 VM，则每个 VM 将等待触发器并尝试运行函数。 例如，如果某个函数应用横向扩展到 3 个 VM，则每个队列触发的函数的默认最大并发实例数是 72。

可以在 [host.json 文件](functions-host-json.md#queues)中配置用于获取新批的批大小和阈值。 如果希望在函数应用中最大程度地降低查询触发的函数的并行执行，可以将批大小设置为 1。 只有当函数在单个虚拟机 (VM) 上运行时，此设置才可消除并发。 

队列触发器会自动阻止函数多次处理队列消息；无需将函数编写为幂等函数。

## <a name="trigger---hostjson-properties"></a>触发器 - host.json 属性

[host.json](functions-host-json.md#queues) 文件包含控制队列触发器行为的设置。 有关可用设置的详细信息，请参阅 [host.json 设置](#hostjson-settings)部分。

## <a name="output"></a>输出

使用 Azure 队列存储输出绑定可将消息写入队列。

## <a name="output---example"></a>输出 - 示例

参阅语言特定的示例：

* [C#](#output---c-example)
* [C# 脚本 (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)
* [Java](#output---java-example)

### <a name="output---c-example"></a>输出 - C# 示例

以下示例演示针对收到的每个 HTTP 请求创建队列消息的[C# 函数](functions-dotnet-class-library.md)。

```csharp
[StorageAccount("AzureWebJobsStorage")]
public static class QueueFunctions
{
    [FunctionName("QueueOutput")]
    [return: Queue("myqueue-items")]
    public static string QueueOutput([HttpTrigger] dynamic input,  ILogger log)
    {
        log.LogInformation($"C# function processed: {input.Text}");
        return input.Text;
    }
}
```

### <a name="output---c-script-example"></a>输出 - C# 脚本示例

以下示例演示 function.json 文件中的一个 HTTP 触发器绑定以及使用该绑定的 [C# 脚本 (.csx)](functions-reference-csharp.md) 代码。 该函数针对收到的每个 HTTP 请求创建一个包含 CustomQueueMessage 对象有效负载的队列项。

function.json 文件如下所示：

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting",
    }
  ]
}
```

[配置](#output---configuration)部分解释了这些属性。

下面是可创建一条队列消息的 C# 脚本代码：

```cs
public class CustomQueueMessage
{
    public string PersonName { get; set; }
    public string Title { get; set; }
}

public static CustomQueueMessage Run(CustomQueueMessage input, ILogger log)
{
    return input;
}
```

可以使用 `ICollector` 或 `IAsyncCollector` 参数一次性发送多条消息。 以下 C# 脚本代码发送多条消息，其中一条消息包含 HTTP 请求数据，另一条消息包含硬编码值：

```cs
public static void Run(
    CustomQueueMessage input, 
    ICollector<CustomQueueMessage> myQueueItems, 
    ILogger log)
{
    myQueueItems.Add(input);
    myQueueItems.Add(new CustomQueueMessage { PersonName = "You", Title = "None" });
}
```

### <a name="output---javascript-example"></a>输出 - JavaScript 示例

以下示例演示 function.json 文件中的一个 HTTP 触发器绑定以及使用该绑定的 [JavaScript 函数](functions-reference-node.md)。 该函数针对收到的每个 HTTP 请求创建一个队列项。

function.json 文件如下所示：

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting",
    }
  ]
}
```

[配置](#output---configuration)部分解释了这些属性。

JavaScript 代码如下所示：

```javascript
module.exports = function (context, input) {
    context.done(null, input.body);
};
```

可以通过定义 `myQueueItem` 输出绑定的消息数组，一次性发送多条消息。 以下 JavaScript 代码针对收到的每个 HTTP 请求发送两条包含硬编码值的队列消息。

```javascript
module.exports = function(context) {
    context.bindings.myQueueItem = ["message 1","message 2"];
    context.done();
};
```

### <a name="output---java-example"></a>输出 - Java 示例

 以下示例显示了一个 Java 函数，该函数在 HTTP 请求触发时创建队列消息。

```java
@FunctionName("httpToQueue")
@QueueOutput(name = "item", queueName = "myqueue-items", connection = "AzureWebJobsStorage")
 public String pushToQueue(
     @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
     final String message,
     @HttpOutput(name = "response") final OutputBinding&lt;String&gt; result) {
       result.setValue(message + " has been added.");
       return message;
 }
```

在 [Java 函数运行时库](/java/api/overview/azure/functions/runtime)中，对其值将写入队列存储的参数使用 `@QueueOutput` 注释。  参数类型应为 `OutputBinding<T>`，其中 T 是 POJO 的任何本机 Java 类型。


## <a name="output---attributes"></a>输出 - 特性

在 [C# 类库](functions-dotnet-class-library.md)中，使用 [QueueAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/QueueAttribute.cs)。

该特性将应用到 `out` 参数，或应用到函数的返回值。 该特性的构造函数采用队列的名称，如以下示例中所示：

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

可以设置 `Connection` 属性来指定要使用的存储帐户，如以下示例中所示：

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items", Connection = "StorageConnectionAppSetting")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

有关完整示例，请参阅[输出 - C# 示例](#output---c-example)。

可以使用 `StorageAccount` 特性在类、方法或参数级别指定存储帐户。 有关详细信息，请参阅“触发器 - 特性”。

## <a name="output---configuration"></a>输出 - 配置

下表解释了在 function.json 文件和 `Queue` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
|类型 | 不适用 | 必须设置为 `queue`。 在 Azure 门户中创建触发器时，会自动设置此属性。|
|direction | 不适用 | 必须设置为 `out`。 在 Azure 门户中创建触发器时，会自动设置此属性。 |
|**name** | 不适用 | 表示函数代码中的队列的变量的名称。 设置为 `$return` 可引用函数返回值。|
|**queueName** |**QueueName** | 队列的名称。 |
|**连接** | **Connection** |包含要用于此绑定的存储连接字符串的应用设置的名称。 如果应用设置名称以“AzureWebJobs”开始，则只能在此处指定该名称的余下部分。 例如，如果将 `connection` 设置为“MyStorage”，函数运行时将会查找名为“AzureWebJobsMyStorage”的应用设置。 如果将 `connection` 留空，函数运行时将使用名为 `AzureWebJobsStorage` 的应用设置中的默认存储连接字符串。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>输出 - 用法

在 C# 和 C# 脚本中，可以使用 `out T paramName` 等方法参数编写一条队列消息。 在 C# 脚本中，`paramName` 是在 *function.json* 的 `name` 属性中指定的值。 可以使用方法返回类型而不使用 `out` 参数，`T` 可为以下任何类型：

* 可序列化为 JSON 的对象
* `string`
* `byte[]`
* [CloudQueueMessage] 

如果尝试绑定到 `CloudQueueMessage` 并收到错误消息，请确保已引用[正确的存储 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在 C# 和 C# 脚本中，可使用以下类型之一编写多条队列消息： 

* `ICollector<T>` 或 `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.azure.storage.queue.cloudqueue)

在 JavaScript 函数中，可以使用 `context.bindings.<name>` 访问输出队列消息。 可对队列项有效负载使用字符串或 JSON 可序列化对象。


## <a name="exceptions-and-return-codes"></a>异常和返回代码

| 绑定 |  参考 |
|---|---|
| 队列 | [队列错误代码](https://docs.microsoft.com/rest/api/storageservices/queue-service-error-codes) |
| Blob、表、队列 | [存储错误代码](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Blob、表、队列 |  [故障排除](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

<a name="host-json"></a>  

## <a name="hostjson-settings"></a>host.json 设置

本部分介绍版本 2.x 中可用于此绑定的全局配置设置。 下面的示例 host.json 文件仅包含此绑定的 2.x 版本设置。 有关版本 2.x 中的全局配置设置的详细信息，请参阅 [Azure Functions 版本 2.x 的 host.json 参考](functions-host-json.md)。

> [!NOTE]
> 有关 Functions 1.x 中 host.json 的参考，请参阅 [Azure Functions 1.x 的 host.json 参考](functions-host-json-v1.md)。

```json
{
    "version": "2.0",
    "extensions": {
        "queues": {
            "maxPollingInterval": "00:00:02",
            "visibilityTimeout" : "00:00:30",
            "batchSize": 16,
            "maxDequeueCount": 5,
            "newBatchThreshold": 8
        }
    }
}
```


|属性  |默认 | 描述 |
|---------|---------|---------|
|maxPollingInterval|00:00:02|队列轮询的最大间隔时间。 最小值为 00:00:00.100（100 毫秒）。 |
|visibilityTimeout|00:00:00|消息处理失败时的重试间隔时间。 |
|batchSize|16|Functions 运行时同时检索并并行处理的队列消息数。 当处理的数量下降到 `newBatchThreshold` 时，运行时可获取另一个批，并开始处理这些消息。 因此，每个函数处理的最大并发消息数是 `batchSize` 加上 `newBatchThreshold`。 此限制分别应用于各个队列触发的函数。 <br><br>如果要避免对队列上收到的消息并行执行，可以将 `batchSize` 设置为 1。 但是，只有在函数于单个虚拟机 (VM) 上运行时，此设置才可消除并发。 如果函数应用横向扩展到多个 VM，每个 VM 可运行每个队列触发的函数的一个实例。<br><br>`batchSize` 的最大值为 32。 |
|maxDequeueCount|5|在将某个消息移到有害队列之前，尝试处理该消息的次数。|
|newBatchThreshold|batchSize/2|只要同时处理的消息数下降到此数值，运行时即检索另一个批次。|

## <a name="next-steps"></a>后续步骤

* [详细了解 Azure Functions 触发器和绑定](functions-triggers-bindings.md)

<!--
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Queue storage trigger](functions-create-storage-queue-triggered-function.md)
-->

> [!div class="nextstepaction"]
> [转到有关使用队列存储输出绑定的教程](functions-integrate-storage-queue-output-binding.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage
