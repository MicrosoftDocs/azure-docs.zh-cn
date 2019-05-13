---
title: 将你的安全消息发送到 Azure 安全中心，IoT 预览版 |Microsoft Docs
description: 了解如何将 iot 使用 Azure 安全中心安全消息发送。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: c611bb5c-b503-487f-bef4-25d8a243803d
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: a91a3538a9c176e3c76e351eb53eb84decc85938
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200537"
---
# <a name="send-security-messages-sdk"></a>发送安全消息 SDK

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本操作方法指南介绍 Azure 安全中心 (ASC) 的 IoT 服务功能，当你选择收集并将你的设备而无需为 IoT 代理使用 ASC 发送安全消息时，并说明如何执行此操作。  

本指南介绍如何： 
> [!div class="checklist"]
> * 使用 API for C# 发送安全消息
> * 使用 API for C 发送安全消息

## <a name="asc-for-iot-capabilities"></a>针对 IoT 功能 ASC

适用于 IoT 的 ASC 可以处理和分析任何类型的安全消息数据，只要发送的数据符合[ASC 为 IoT 架构](https://aka.ms/iot-security-schemas)消息被设置为一条安全消息。

## <a name="security-message"></a>安全消息

适用于 IoT 的 ASC 定义一条安全消息，使用以下条件：
- 如果使用 Azure IoT C 发送消息 /C# SDK
- 如果消息符合[安全消息架构](https://aka.ms/iot-security-schemas)
- 如果消息已设置为一条安全消息在发送前

每个安全消息包括发件人的元数据，如`AgentId`， `AgentVersion`，`MessageSchemaVersion`和安全事件的列表。
架构定义包括的事件类型的安全消息的有效和所需属性。

[!NOTE]
> 发送的消息如果不符合该架构，则将被忽略。 在开始发送数据之前，请务必验证架构，因为当前不会存储已忽略的消息。 
> 发送消息的未设置为使用 Azure IoT C 的安全消息 /C# SDK 将不会路由到 ASC 为 IoT 管道

## <a name="valid-message-example"></a>有效的消息示例

下面的示例显示了有效的安全消息对象。 此示例包含消息元数据，另一个`ProcessCreate`安全事件。

一次设置为一条安全消息，发送，此消息将由处理 ASC 对 IoT。

```json
"AgentVersion": "0.0.1",
"AgentId": "e89dc5f5-feac-4c3e-87e2-93c16f010c25",
"MessageSchemaVersion": "1.0",
"Events": [
    {
        "EventType": "Security",
        "Category": "Triggered",
        "Name": "ProcessCreate",
        "IsEmpty": false,
        "PayloadSchemaVersion": "1.0",
        "Id": "21a2db0b-44fe-42e9-9cff-bbb2d8fdf874",
        "TimestampLocal": "2019-01-27 15:48:52Z",
        "TimestampUTC": "2019-01-27 13:48:52Z",
        "Payload":
            [
                {
                    "Executable": "/usr/bin/echo",
                    "ProcessId": 11750,
                    "ParentProcessId": 1593,
                    "UserName": "nginx",
                    "CommandLine": "./backup .htaccess"
                }
            ]
    }
]
```

## <a name="send-security-messages"></a>发送安全消息 

发送安全消息，而无需使用 IoT 代理使用 ASC [Azure IoTC#设备 SDK](https://github.com/Azure/azure-iot-sdk-csharp/tree/preview)或[的 Azure IoT C 设备 SDK](https://github.com/Azure/azure-iot-sdk-c/tree/public-preview)。

若要从设备发送设备消息以供 ASC for IoT 处理，请使用以下 API 之一来标记消息，以正确路由到 ASC for IoT 处理管道。 以这种方式发送的消息将会得到处理，并以安全见解的形式显示在 IoT 中心或 Azure 安全中心的 ASC for IoT 内。 

即使已使用正确的标头进行标记，发送的所有数据也必须符合 [ASC for IoT 消息架构](https://aka.ms/iot-security-schemas)。 

### <a name="send-security-message-api"></a>发送安全消息 API

“发送安全消息”API 目前可在 C 和 C# 中使用。  

#### <a name="c-api"></a>C# API

```cs

private static async Task SendSecurityMessageAsync(string messageContent)
{
    ModuleClient client = ModuleClient.CreateFromConnectionString("<connection_string>");
    Message  securityMessage = new Message(Encoding.UTF8.GetBytes(messageContent));
    securityMessage.SetAsSecurityMessage();
    await client.SendEventAsync(securityMessage);
}
```

#### <a name="c-api"></a>C API

```c
bool SendMessageAsync(IoTHubAdapter* iotHubAdapter, const void* data, size_t dataSize) {
 
    bool success = true;
    IOTHUB_MESSAGE_HANDLE messageHandle = NULL;
 
    messageHandle = IoTHubMessage_CreateFromByteArray(data, dataSize);
 
    if (messageHandle == NULL) {
        success = false;
        goto cleanup;
    }
 
    if (IoTHubMessage_SetAsSecurityMessage(messageHandle) != IOTHUB_MESSAGE_OK) {
        success = false;
        goto cleanup;
    }
 
    if (IoTHubModuleClient_SendEventAsync(iotHubAdapter->moduleHandle, messageHandle, SendConfirmCallback, iotHubAdapter) != IOTHUB_CLIENT_OK) {
        success = false;
        goto cleanup;
    }
 
cleanup:
    if (messageHandle != NULL) {
        IoTHubMessage_Destroy(messageHandle);
    }
 
    return success;
}
 
static void SendConfirmCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback) {
    if (userContextCallback == NULL) {
        //error handling
        return;
    }
 
    if (result != IOTHUB_CLIENT_CONFIRMATION_OK){
        //error handling
    }
}
```

## <a name="next-steps"></a>后续步骤
- 阅读适用于 IoT 的 ASC 服务[概述](overview.md)
- 详细了解适用于 IoT 的 ASC [体系结构](architecture.md)
- 启用该[服务](quickstart-onboard-iot-hub.md)
- 阅读[常见问题解答](resources-frequently-asked-questions.md)
- 了解如何访问[原始安全数据](how-to-security-data-access.md)
- 了解[建议](concept-recommendations.md)
- 了解[警报](concept-security-alerts.md)
