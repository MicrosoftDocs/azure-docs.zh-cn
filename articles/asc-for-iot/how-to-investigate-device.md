---
title: IoT 设备调查指南预览版的 azure 安全中心 |Microsoft Docs
description: 此方法指南说明如何使用 Azure IoT 的安全中心调查可疑使用 Log Analytics 在 IoT 设备。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: b18b48ae-b445-48f8-9ac0-365d6e065b64
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/18/2019
ms.author: mlottner
ms.openlocfilehash: 15e65c155a98ae12c156587735d34a16ed2c9109
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192661"
---
# <a name="investigate-a-suspicious-iot-device"></a>调查可疑的 IoT 设备

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

IoT 服务警报和证据的 azure 安全中心 (ASC) 提供明确的指示时涉及的可疑活动或迹象存在设备受到怀疑 IoT 设备。 

在本指南中，使用调查建议可以帮助确定你的组织可能面临的风险，决定如何修正并发现在将来防止类似攻击的最佳方法。  

> [!div class="checklist"]
> * 查找设备数据
> * 使用 kql 查询进行调查


## <a name="how-can-i-access-my-data"></a>如何访问我的数据？

ASC for IoT 默认将安全警报和建议存储在 Log Analytics 工作区中。 你还可以选择存储原始安全数据。

查找 Log Analytics 工作区中的数据存储位置：

1. 打开 IoT 中心， 
1. 下**安全**，单击**概述**，然后选择**设置**。
1. 更改 Log Analytics 工作区配置详细信息。 
1. 单击“ **保存**”。 

根据配置执行以下操作，以访问 Log Analytics 工作区中存储的数据：

1. 在 IoT 中心选择并单击某个 ASC for IoT 警报。 
1. 单击“进一步调查”。 
1. 选择“若要查看哪些设备生成了此警报，请单击此处并查看 DeviceId 列”。

## <a name="investigation-steps-for-suspicious-iot-devices"></a>可疑 IoT 设备的调查步骤

若要访问的见解和有关 IoT 设备的原始数据，请转到 Log Analytics 工作区[来访问你的数据](#how-can-i-access-my-data)。

检查并调查以下详细信息和活动使用以下 kql 查询的设备数据。

### <a name="related-alerts"></a>相关的警报

若要确定在大致相同的时间是否触发了其他警报，请使用以下 kql 查询：

  ~~~
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityAlert
  | where ExtendedProperties contains device and ResourceId contains tolower(hub)
  | project TimeGenerated, AlertName, AlertSeverity, Description, ExtendedProperties
  ~~~

### <a name="users-with-access"></a>具有访问权限的用户

若要确定哪些用户有权访问此设备，请使用以下 kql 查询： 

  ~~~
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "LocalUsers"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     GroupNames=extractjson("$.GroupNames", EventDetails, typeof(string)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string))
  | summarize FirstObserved=min(TimestampLocal) by GroupNames, UserName
  ~~~
使用此数据来确定： 
  1. 哪些用户有权访问该设备？
  2. 具有访问权限的用户是否按预期的权限级别？ 

### <a name="open-ports"></a>打开端口

若要找出哪些端口的设备中当前正在使用或使用了，使用以下 kql 查询： 

  ~~~
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ListeningPorts"
     and extractjson("$.LocalPort", EventDetails, typeof(int)) <= 1024 // avoid short-lived TCP ports (Ephemeral)
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Protocol=extractjson("$.Protocol", EventDetails, typeof(string)),
     LocalAddress=extractjson("$.LocalAddress", EventDetails, typeof(string)),
     LocalPort=extractjson("$.LocalPort", EventDetails, typeof(int)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     RemotePort=extractjson("$.RemotePort", EventDetails, typeof(string))
  | summarize MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), AllowedRemoteIPAddress=makeset(RemoteAddress), AllowedRemotePort=makeset(RemotePort) by Protocol, LocalPort
  ~~~

    Use this data to discover:
  1. 哪些侦听套接字当前在设备上处于活动状态？
  2. 应允许当前处于活动状态的侦听套接字
  3. 是否有任何可疑的远程地址连接到设备？

### <a name="user-logins"></a>用户登录名

若要确定登录到设备的用户，请使用以下 kql 查询： 
 
  ~~~
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "Login"
     // filter out local, invalid and failed logins
     and EventDetails contains "RemoteAddress"
     and EventDetails !contains '"RemoteAddress":"127.0.0.1"'
     and EventDetails !contains '"UserName":"(invalid user)"'
     and EventDetails !contains '"UserName":"(unknown user)"'
     //and EventDetails !contains '"Result":"Fail"'
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string)),
     LoginHandler=extractjson("$.Executable", EventDetails, typeof(string)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     Result=extractjson("$.Result", EventDetails, typeof(string))
  | summarize CntLoginAttempts=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), CntIPAddress=dcount(RemoteAddress), IPAddress=makeset(RemoteAddress) by UserName, Result, LoginHandler
  ~~~

    Use the query results to discover:
  1. 哪些用户登录到了设备？
  2. 是在中，记录的用户应该能够登录？
  3. 登录的用户是从预期还是意外的 IP 地址连接的？
  
### <a name="process-list"></a>进程列表

若要查找的进程列表是否按预期方式，请使用以下 kql 查询： 

  ~~~
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ProcessCreate"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Executable=extractjson("$.Executable", EventDetails, typeof(string)),
     UserId=extractjson("$.UserId", EventDetails, typeof(string)),
     CommandLine=extractjson("$.CommandLine", EventDetails, typeof(string))
  | join kind=leftouter (
     // user UserId details
     SecurityIoTRawEvent
     | where
        DeviceId == device and AssociatedResourceId contains tolower(hub)
        and RawEventName == "LocalUsers"
     | project
        UserId=extractjson("$.UserId", EventDetails, typeof(string)),
        UserName=extractjson("$.UserName", EventDetails, typeof(string))
     | distinct UserId, UserName
  ) on UserId
  | extend UserIdName = strcat("Id:", UserId, ", Name:", UserName)
  | summarize CntExecutions=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), ExecutingUsers=makeset(UserIdName), ExecutionCommandLines=makeset(CommandLine) by Executable
  ~~~

    Use the query results to discover:

  1. 是否有任何可疑的进程在设备上运行？
  2. 进程是否由适当的用户执行？
  3. 所有命令行执行是否包含正确的预期参数？

## <a name="next-steps"></a>后续步骤

调查设备并更好地了解风险后，可能需要考虑[配置自定义警报](quickstart-create-custom-alerts.md)，以改善 IoT 解决方案的安全态势。 如果没有设备代理，请考虑[部署安全代理](how-to-deploy-agent.md)或[更改现有设备代理的配置](how-to-agent-configuration.md)，以改善结果。 