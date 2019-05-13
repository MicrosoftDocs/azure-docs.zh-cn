---
title: Azure Functions 1.x 的 host.json 参考
description: 使用 v1 运行时的 Azure Functions host.json 文件的参考文档。
services: functions
author: ggailey777
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 10/19/2018
ms.author: glenga
ms.openlocfilehash: 44bc5a245d1bcbc8ff53991af4193ef86f7cd704
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62107068"
---
# <a name="hostjson-reference-for-azure-functions-1x"></a>Azure Functions 1.x 的 host.json 参考

> [!div class="op_single_selector" title1="Select the version of the Azure Functions runtime you are using: "]
> * [版本 1](functions-host-json-v1.md)
> * [第 2 版](functions-host-json.md)

*host.json* 元数据文件包含对函数应用的所有函数产生影响的全局配置选项。 本文列出了可用于 v1 运行时的设置。 JSON 架构位于 http://json.schemastore.org/host。

> [!NOTE]
> 本文适用于 Azure Functions 1.x。  有关 Functions 2.x 中 host.json 的参考，请参阅 [Azure Functions 2.x 的 host.json 参考](functions-host-json.md)。

其他函数应用配置选项是在你的[应用设置](functions-app-settings.md)中管理的。

[local.settings.json](functions-run-local.md#local-settings-file) 文件中的某些 host.json 设置仅在本地运行时才使用。

## <a name="sample-hostjson-file"></a>示例 host.json 文件

以下示例 *host.json* 文件指定了所有可能的选项。


```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    },
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    },
    "functions": [ "QueueProcessor", "GitHubWebHook" ],
    "functionTimeout": "00:05:00",
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    },
    "http": {
        "routePrefix": "api",
        "maxOutstandingRequests": 20,
        "maxConcurrentRequests": 10,
        "dynamicThrottlesEnabled": false
    },
    "id": "9f4ea53c5136457d883d685e57164f08",
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    },
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    },
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00"
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    },
    "watchDirectories": [ "Shared" ],
}
```

本文的以下各部分解释了每个顶级属性。 除非另有说明，否则其中的所有属性都是可选的。

## <a name="aggregator"></a>aggregator

[!INCLUDE [aggregator](../../includes/functions-host-json-aggregator.md)]

## <a name="applicationinsights"></a>applicationInsights

[!INCLUDE [applicationInsights](../../includes/functions-host-json-applicationinsights.md)]

## <a name="durabletask"></a>durableTask

[!INCLUDE [durabletask](../../includes/functions-host-json-durabletask.md)]

## <a name="eventhub"></a>eventHub

[事件中心触发器和绑定](functions-bindings-event-hubs.md)的配置设置。

[!INCLUDE [functions-host-json-event-hubs](../../includes/functions-host-json-event-hubs.md)]

## <a name="functions"></a>functions

作业主机运行的函数列表。 空数组表示运行所有函数。 仅供在[本地运行](functions-run-local.md)时使用。 在 Azure 的函数应用中，应改为按照[如何在 Azure Functions 中禁用函数](disable-function.md)中的步骤禁用特定函数，而不是使用此设置。

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

指示所有函数的超时持续时间。 在无服务器消耗计划中，有效范围为 1 秒至 10 分钟，默认值为 5 分钟。 在应用服务计划中，没有总体限制，默认值取决于运行时版本。

```json
{
    "functionTimeout": "00:05:00"
}
```

## <a name="healthmonitor"></a>healthMonitor

[主机运行状况监视器](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Host-Health-Monitor)的配置设置。

```
{
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|enabled|true|指定是否已启用该功能。 | 
|healthCheckInterval|10 秒|定期后台运行状况检查之间的时间间隔。 | 
|healthCheckWindow|2 分钟|与 `healthCheckThreshold` 设置结合使用的滑动时间窗口。| 
|healthCheckThreshold|6|在启动主机回收之前，运行状况检查可以失败的最大次数。| 
|counterThreshold|0.80|性能计数器将被视为不正常的阈值。| 

## <a name="http"></a>http

[http 触发器和绑定](functions-bindings-http-webhook.md)的配置设置。

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="id"></a>id

*仅限版本 1.x*。

作业宿主的唯一 ID。 可以是不带短划线的小写 GUID。 在本地运行时必须指定。 在 Azure 中运行时，我们建议你不要设置 ID 值。 当省略 `id` 时，会自动在 Azure 中生成 ID。 

如果跨多个函数应用共享存储帐户，请确保每个函数应用都有不同的 `id`。 可省略 `id` 属性或手动将每个函数应用的 `id` 设置为不同的值。 计时器触发器使用存储锁来确保当函数应用横向扩展到多个实例时将只有一个计时器实例。 如果两个函数应用共享相同的 `id` 且每个都使用计时器触发器，则只会运行一个计时器。

```json
{
    "id": "9f4ea53c5136457d883d685e57164f08"
}
```

## <a name="logger"></a>logger

控制 [ILogger 对象](functions-monitoring.md#write-logs-in-c-functions)或 [context.log](functions-monitoring.md#write-logs-in-javascript-functions) 写入的日志的筛选。

```json
{
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|categoryFilter|不适用|指定按类别进行筛选| 
|defaultLevel|信息|对于 `categoryLevels` 数组中未指定的任何类别，会将此级别和更高级别的日志发送到 Application Insights。| 
|categoryLevels|不适用|一个类别数组，指定每个类别的、要发送到 Application Insights 的最低日志级别。 此处指定的类别控制以相同值开头的所有类别，较长的值优先。 在前面的示例 *host.json* 文件中，将在 `Information` 级别记录以“Host.Aggregator”开头的所有类别的日志。 在 `Error` 级别记录以“Host”开头的其他所有类别（例如“Host.Executor”）的日志。| 

## <a name="queues"></a>queues

[存储队列触发器和绑定](functions-bindings-storage-queue.md)的配置设置。

```json
{
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|maxPollingInterval|60000|队列轮询的最大间隔时间，以毫秒为单位。| 
|visibilityTimeout|0|消息处理失败时的重试间隔时间。| 
|batchSize|16|Functions 运行时同时检索并并行处理的队列消息数。 当处理的数量下降到 `newBatchThreshold` 时，运行时可获取另一个批，并开始处理这些消息。 因此，每个函数处理的最大并发消息数是 `batchSize` 加上 `newBatchThreshold`。 此限制分别应用于各个队列触发的函数。 <br><br>如果要避免对队列上收到的消息并行执行，可以将 `batchSize` 设置为 1。 但是，只有在函数于单个虚拟机 (VM) 上运行时，此设置才可消除并发。 如果函数应用横向扩展到多个 VM，每个 VM 可运行每个队列触发的函数的一个实例。<br><br>`batchSize` 的最大值为 32。 | 
|maxDequeueCount|5|在将某个消息移到有害队列之前，尝试处理该消息的次数。| 
|newBatchThreshold|batchSize/2|只要同时处理的消息数下降到此数值，运行时即检索另一个批次。| 

## <a name="servicebus"></a>serviceBus

[服务总线触发器和绑定](functions-bindings-service-bus.md)的配置设置。

```json
{
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00"
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|maxConcurrentCalls|16|消息泵应该对回调发起的最大并发调用数。 默认情况下，Functions 运行时同时处理多条消息。 若要指示运行时一次只处理单个队列或主题消息，请将 `maxConcurrentCalls` 设置为 1。 | 
|prefetchCount|不适用|基础 MessageReceiver 将要使用的默认 PrefetchCount。| 
|autoRenewTimeout|00:05:00|自动续订消息锁的最长持续时间。| 

## <a name="singleton"></a>singleton

单一实例锁行为的配置设置。 有关详细信息，请参阅[有关单一实例支持的 GitHub 问题](https://github.com/Azure/azure-webjobs-sdk-script/issues/912)。

```json
{
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|lockPeriod|00:00:15|占用函数级锁的时间段。 锁自动续订。| 
|listenerLockPeriod|00:01:00|占用侦听器锁的时间段。| 
|listenerLockRecoveryPollingInterval|00:01:00|在启动时无法获取侦听器锁的情况下，用于恢复侦听器锁的时间间隔。| 
|lockAcquisitionTimeout|00:01:00|运行时尝试获取锁的最长时间。| 
|lockAcquisitionPollingInterval|不适用|尝试获取锁的间隔时间。| 

## <a name="tracing"></a>tracing

版本 1.x

使用 `TraceWriter` 对象创建的日志的配置设置。 请参阅 [C# 日志记录](functions-reference-csharp.md#logging)和 [Node.js 日志记录](functions-reference-node.md#writing-trace-output-to-the-console)。

```json
{
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|consoleLevel|info|控制台日志记录的跟踪级别。 选项包括：`off`、`error`、`warning`、`info` 和 `verbose`。|
|fileLoggingMode|debugOnly|文件日志记录的跟踪级别。 选项包括 `never`、`always` 和 `debugOnly`。| 

## <a name="watchdirectories"></a>watchDirectories

应该监视其更改情况的一组[共享代码目录](functions-reference-csharp.md#watched-directories)。  确保当这些目录中的代码发生更改时，函数会拾取这些更改。

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何更新 host.json 文件](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [查看环境变量中的全局设置](functions-app-settings.md)
