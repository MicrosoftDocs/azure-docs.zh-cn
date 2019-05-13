---
title: 了解 Azure IoT 中心作业 | Microsoft Docs
description: 开发人员指南 - 计划要在连接到 IoT 中心的多个设备上运行的作业。 作业可以更新标记和所需属性，并可在多个设备上调用直接方法。
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 147dd0f454bd85673bcba5cd6148c5da9716c580
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409063"
---
# <a name="schedule-jobs-on-multiple-devices"></a>在多个设备上计划作业

Azure IoT 中心可启用多个构建基块（如[设备孪生属性和标记](iot-hub-devguide-device-twins.md)和[直接方法](iot-hub-devguide-direct-methods.md)）。 通常情况下，后端应用允许设备管理员和操作员在计划的时间批量更新 IoT 设备并与之交互。 作业在计划的时间针对一组设备执行设备孪生更新和直接方法。 例如，操作员可使用用于启动和跟踪作业的后端应用在不会中断大楼运作的时间重新启动 43 号大楼第 3 层中的一组设备。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

在以下情况下请考虑使用作业：需要计划并跟踪一组设备上的以下任何活动的进度：

* 更新所需属性
* 更新标记
* 调用直接方法

## <a name="job-lifecycle"></a>作业生命周期

作业由解决方案后端启动，并由 IoT 中心维护。 可以通过面向服务的 URI (`PUT https://<iot hub>/jobs/v2/<jobID>?api-version=2018-06-30`) 启动作业，并通过面向服务的 URI (`GET https://<iot hub>/jobs/v2/<jobID?api-version=2018-06-30`) 查询正在执行的作业的进度。 若要在启动作业后刷新正在运行的作业的状态，请运行作业查询。

> [!NOTE]
> 启动作业时，属性名称和值只能包含 US-ASCII 可打印字母数字，但下列组中的任一项除外：`$ ( ) < > @ , ; : \ " / [ ] ? = { } SP HT`

## <a name="jobs-to-execute-direct-methods"></a>用于执行直接方法的作业

以下代码片段显示了使用作业在一组设备上执行[直接方法](iot-hub-devguide-direct-methods.md)的 HTTPS 1.1 请求详细信息：

```
PUT /jobs/v2/<jobId>?api-version=2018-06-30

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8

{
    "jobId": "<jobId>",
    "type": "scheduleDeviceMethod",
    "cloudToDeviceMethod": {
        "methodName": "<methodName>",
        "payload": <payload>,
        "responseTimeoutInSeconds": methodTimeoutInSeconds
    },
    "queryCondition": "<queryOrDevices>", // query condition
    "startTime": <jobStartTime>,          // as an ISO-8601 date string
    "maxExecutionTimeInSeconds": <maxExecutionTimeInSeconds>
}
```

查询条件也可以位于单个设备 ID 上或位于设备 ID 列表中，如以下示例所示：

```
"queryCondition" = "deviceId = 'MyDevice1'"
"queryCondition" = "deviceId IN ['MyDevice1','MyDevice2']"
"queryCondition" = "deviceId IN ['MyDevice1']"
```

[IoT 中心查询语言](iot-hub-devguide-query-language.md)格外详细地介绍了 IoT 中心查询语言。

以下代码片段显示了请求和作业计划为调用一个名为 contoso-中心-1 上的所有设备上的 testMethod 的直接方法的响应：

```
PUT https://contoso-hub-1.azure-devices.net/jobs/v2/job01?api-version=2018-06-30 HTTP/1.1
Authorization: SharedAccessSignature sr=contoso-hub-1.azure-devices.net&sig=68iv------------------------------------v8Hxalg%3D&se=1556849884&skn=iothubowner
Content-Type: application/json; charset=utf-8
Host: contoso-hub-1.azure-devices.net
Content-Length: 317

{
    "jobId": "job01",
    "type": "scheduleDeviceMethod",
    "cloudToDeviceMethod": {
        "methodName": "testMethod",
        "payload": {},
        "responseTimeoutInSeconds": 30
    },
    "queryCondition": "*", 
    "startTime": "2019-05-04T15:53:00.077Z",
    "maxExecutionTimeInSeconds": 20
}

HTTP/1.1 200 OK
Content-Length: 65
Content-Type: application/json; charset=utf-8
Vary: Origin
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 03 May 2019 01:46:18 GMT

{"jobId":"job01","type":"scheduleDeviceMethod","status":"queued"}
```

## <a name="jobs-to-update-device-twin-properties"></a>用于更新设备孪生属性的作业

以下代码片段显示了使用作业更新设备克隆属性的 HTTPS 1.1 请求详细信息：

```
PUT /jobs/v2/<jobId>?api-version=2018-06-30

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8

{
    "jobId": "<jobId>",
    "type": "scheduleUpdateTwin",
    "updateTwin": <patch>                 // Valid JSON object
    "queryCondition": "<queryOrDevices>", // query condition
    "startTime": <jobStartTime>,          // as an ISO-8601 date string
    "maxExecutionTimeInSeconds": <maxExecutionTimeInSeconds>
}
```

> [!NOTE]
> *UpdateTwin*属性需要有效的 etag 匹配; 例如， `etag="*"`。

以下代码片段显示了请求和响应作业计划更新的测试设备在 contoso-中心-1 的设备孪生属性：

```
PUT https://contoso-hub-1.azure-devices.net/jobs/v2/job02?api-version=2018-06-30 HTTP/1.1
Authorization: SharedAccessSignature sr=contoso-hub-1.azure-devices.net&sig=BN0U-------------------------------------RuA%3D&se=1556925787&skn=iothubowner
Content-Type: application/json; charset=utf-8
Host: contoso-hub-1.azure-devices.net
Content-Length: 339

{
    "jobId": "job02",
    "type": "scheduleUpdateTwin",
    "updateTwin": {
      "properties": {
        "desired": {
          "test1": "value1"
        }
      },
     "etag": "*"
     },
    "queryCondition": "deviceId = 'test-device'",
    "startTime": "2019-05-08T12:19:56.868Z",
    "maxExecutionTimeInSeconds": 20
}

HTTP/1.1 200 OK
Content-Length: 63
Content-Type: application/json; charset=utf-8
Vary: Origin
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 03 May 2019 22:45:13 GMT

{"jobId":"job02","type":"scheduleUpdateTwin","status":"queued"}
```

## <a name="querying-for-progress-on-jobs"></a>查询作业的进度

以下代码片段显示了用于查询作业的 HTTPS 1.1 请求详细信息：

```
GET /jobs/v2/query?api-version=2018-06-30[&jobType=<jobType>][&jobStatus=<jobStatus>][&pageSize=<pageSize>][&continuationToken=<continuationToken>]

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8
```

从响应提供 continuationToken。

可以使用[设备孪生、作业和消息路由的 IoT 中心查询语言](iot-hub-devguide-query-language.md)在每台设备上查询作业执行状态。

## <a name="jobs-properties"></a>作业属性

以下列表显示了属性和相应说明，在查询作业或作业结果时可使用这些属性。

| 属性 | 描述 |
| --- | --- |
| **jobId** |应用程序提供的作业 ID。 |
| **startTime** |应用程序提供的作业开始时间(ISO-8601)。 |
| **endTime** |IoT 中心提供的作业完成时的日期(ISO-8601)。 只有在作业达到“完成”状态后才有效。 |
| **type** |作业的类型： |
| | **scheduleUpdateTwin**:用于更新一组所需的属性或标记的作业。 |
| | **scheduleDeviceMethod**:用于调用设备方法对一组设备孪生的作业。 |
| **status** |作业的当前状态。 可能的状态值： |
| | **挂起**:计划并等待作业服务选取。 |
| | **计划**:计划在将来某个时间。 |
| | **运行**:当前活动的作业。 |
| | **取消**:作业已取消。 |
| | **失败**:作业失败。 |
| | **完成**:作业已完成。 |
| **deviceJobStatistics** |有关作业执行的统计信息。 |
| | **deviceJobStatistics** 属性： |
| | **deviceJobStatistics.deviceCount**:作业中的设备数。 |
| | **deviceJobStatistics.failedCount**:作业失败的设备数。 |
| | **deviceJobStatistics.succeededCount**:作业成功的设备数。 |
| | **deviceJobStatistics.runningCount**:当前正在运行作业的设备数。 |
| | **deviceJobStatistics.pendingCount**:等待运行作业的设备数。 |

### <a name="additional-reference-material"></a>其他参考资料

IoT 中心开发人员指南中的其他参考主题包括：

* [IoT 中心终结点](iot-hub-devguide-endpoints.md)介绍了每个 IoT 中心针对运行时和管理操作公开的各种终结点。

* [限制和配额](iot-hub-devguide-quotas-throttling.md)介绍了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。

* [Azure IoT 设备和服务 SDK](iot-hub-devguide-sdks.md) 列出了开发与 IoT 中心交互的设备和服务应用时可使用的各种语言 SDK。

* [设备孪生、作业和消息路由的 IoT 中心查询语言](iot-hub-devguide-query-language.md)介绍 IoT 中心查询语言。 使用此查询语言从 IoT 中心检索设备孪生和作业的相关信息。

* [IoT 中心 MQTT 支持](iot-hub-mqtt-support.md)提供了有关 IoT 中心对 MQTT 协议的支持的详细信息。

## <a name="next-steps"></a>后续步骤

要尝试本文中介绍的一些概念，请参阅以下 IoT 中心教程：

* [计划和广播作业](iot-hub-node-node-schedule-jobs.md)