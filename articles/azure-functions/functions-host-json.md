---
title: Azure Functions 2.x 的 host.json 参考
description: 使用 v2 运行时的 Azure Functions host.json 文件的参考文档。
services: functions
author: ggailey777
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/08/2018
ms.author: glenga
ms.openlocfilehash: e24c5b2be1df41d84fa4461250f51cb009f77529
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60737174"
---
# <a name="hostjson-reference-for-azure-functions-2x"></a>Azure Functions 2.x 的 host.json 参考  

> [!div class="op_single_selector" title1="Select the version of the Azure Functions runtime you are using: "]
> * [版本 1](functions-host-json-v1.md)
> * [第 2 版](functions-host-json.md)

*host.json* 元数据文件包含对函数应用的所有函数产生影响的全局配置选项。 本文列出了可用于 v2 运行时的设置。  

> [!NOTE]
> 本文适用于 Azure Functions 2.x。  有关 Functions 1.x 中 host.json 的参考，请参阅 [Azure Functions 1.x 的 host.json 参考](functions-host-json-v1.md)。

其他函数应用配置选项是在你的[应用设置](functions-app-settings.md)中管理的。

[local.settings.json](functions-run-local.md#local-settings-file) 文件中的某些 host.json 设置仅在本地运行时才使用。

## <a name="sample-hostjson-file"></a>示例 host.json 文件

以下示例 *host.json* 文件指定了所有可能的选项。


```json
{
    "version": "2.0",
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "extensions": {
        "cosmosDb": {},
        "durableTask": {},
        "eventHubs": {},
        "http": {},
        "queues": {},
        "sendGrid": {},
        "serviceBus": {}
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
    "logging": {
        "fileLoggingMode": "debugOnly",
        "logLevel": {
          "Function.MyFunction": "Information",
          "default": "None"
        },
        "applicationInsights": {
            "samplingSettings": {
              "isEnabled": true,
              "maxTelemetryItemsPerSecond" : 5
            }
        }
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "watchDirectories": [ "Shared", "Test" ]
}
```

本文的以下各部分解释了每个顶级属性。 除非另有说明，否则其中的所有属性都是可选的。

## <a name="aggregator"></a>aggregator

[!INCLUDE [aggregator](../../includes/functions-host-json-aggregator.md)]

## <a name="applicationinsights"></a>applicationInsights

此设置是[日志记录](#logging)的子项。

控制 [Application Insights 中的采样功能](./functions-monitoring.md#configure-sampling)。

```json
{
    "applicationInsights": {
        "samplingSettings": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    }
}
```

> [!NOTE]
> 日志采样可能会导致一些执行不会显示在 Application Insights 监视器边栏选项卡中。

|属性  |默认 | 描述 |
|---------|---------|---------| 
|isEnabled|true|启用或禁用采样。| 
|maxTelemetryItemsPerSecond|5|开始采样所要达到的阈值。| 

## <a name="cosmosdb"></a>CosmosDB

可在 [Cosmos DB 触发器和绑定](functions-bindings-cosmosdb-v2.md#host-json)中查找配置设置。

## <a name="durabletask"></a>durableTask

可在 [Durable Functions 的绑定](durable/durable-functions-bindings.md#host-json)中查找配置设置。

## <a name="eventhub"></a>eventHub

可在[事件中心触发器和绑定](functions-bindings-event-hubs.md#host-json)中查找配置设置。 

## <a name="extensions"></a>扩展

该属性返回一个对象，其中包含所有特定于绑定的设置，例如 [http](#http) 和 [eventHub](#eventhub)。

## <a name="functions"></a>functions

作业主机运行的函数列表。 空数组表示运行所有函数。 仅供在[本地运行](functions-run-local.md)时使用。 在 Azure 的函数应用中，应改为按照[如何在 Azure Functions 中禁用函数](disable-function.md)中的步骤禁用特定函数，而不是使用此设置。

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

指示所有函数的超时持续时间。 在无服务器消耗计划中，有效范围为 1 秒至 10 分钟，默认值为 5 分钟。 在应用服务计划中，没有总体限制，默认值取决于运行时版本。 在版本 2.x 中，适用于应用服务计划的默认值为 30 分钟。 在版本 1.x 中，它为 *null*，表示无超时。

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

可在 [http 触发器和绑定](functions-bindings-http-webhook.md)中查找配置设置。

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="logging"></a>logging

控制函数应用（包括 Application Insights）的日志记录行为。

```json
"logging": {
    "fileLoggingMode": "debugOnly",
    "logLevel": {
      "Function.MyFunction": "Information",
      "default": "None"
    },
    "applicationInsights": {
        ...
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------|
|fileLoggingMode|debugOnly|定义启用哪种级别的文件日志记录。  选项包括 `never`、`always` 和 `debugOnly`。 |
|logLevel|不适用|一个对象，它定义了用于筛选应用中的函数的日志类别。 版本 2.x 遵循 ASP.NET Core 布局进行日志类别筛选。 这允许你筛选特定函数的日志记录。 有关详细信息，请参阅 ASP.NET Core 文档中的[日志筛选](https://docs.microsoft.com/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#log-filtering)。 |
|console|不适用| [控制台](#console)日志记录设置。 |
|applicationInsights|不适用| [applicationInsights](#applicationinsights) 设置。 |

## <a name="console"></a>console

此设置是[日志记录](#logging)的子项。 它在未处于调试模式时控制控制台日志记录。

```json
{
    "logging": {
    ...
        "console": {
          "isEnabled": "false"
        },
    ...
    }
}
```

|属性  |默认 | 描述 |
|---------|---------|---------| 
|isEnabled|false|启用或禁用控制台日志记录。| 

## <a name="queues"></a>queues

可在[存储队列触发器和绑定](functions-bindings-storage-queue.md#host-json)中查找设置。  

## <a name="sendgrid"></a>SendGrid

可在 [SendGrid 触发器和绑定](functions-bindings-sendgrid.md#host-json)中查找配置设置。

## <a name="servicebus"></a>serviceBus

可在[服务总线触发器和绑定](functions-bindings-service-bus.md#host-json)中查找配置设置。

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

## <a name="version"></a>版本

对于面向 v2 运行时的函数应用，版本字符串 `"version": "2.0"` 是必需的。

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
