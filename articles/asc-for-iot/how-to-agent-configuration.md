---
title: Azure 安全中心配置适用于 IoT 代理预览 |Microsoft Docs
description: 了解如何为 IoT 与 Azure 安全中心配置为使用代理。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: f95c445a-4f0d-4198-9c6c-d01446473bd0
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: 311a867c863cbd72292d8e34b4f7abdb949b12c6
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198446"
---
# <a name="tutorial-configure-security-agents"></a>教程：配置安全代理

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文介绍 Azure 安全中心 (ASC) 的 IoT 安全代理，如何更改它们 ASC 为 IoT 安全代理配置。

> [!div class="checklist"]
> * 配置安全代理
> * 通过编辑孪生属性来更改代理的行为
> * 发现默认配置

## <a name="agents"></a>代理

ASC 为 IoT 安全代理从 IoT 设备收集数据并执行安全措施来降低检测到的漏洞。 可使用一组可自定义的模块孪生属性控制安全代理配置。 一般情况下，不常辅助更新到这些属性。  

ASC IoT 的安全代理孪生配置对象是一个.json 格式对象。 配置对象是一组可控制可以定义来控制代理行为的属性。 

这些配置可帮助您自定义适用于每个方案所需的代理。 例如，自动排除某些事件，或保持最低级别的功率消耗是可以通过配置这些属性。  

IoT 安全代理配置为使用 ASC[架构](https://aka.ms/iot-security-github-module-schema)进行更改。  

## <a name="configuration-objects"></a>配置对象 

IoT 安全代理每个 ASC 相关属性位于所需的属性部分中的代理配置对象的**azureiotsecurity**模块。 

若要修改的配置，创建和修改此对象内的**azureiotsecurity**模块孪生的标识。 

如果代理配置对象中不存在**azureiotsecurity**模块孪生，所有安全代理属性值都设置为默认值。 

```json
"desired": {
  "azureiot*com^securityAgentConfiguration^1*0*0": {
  } 
}
```

请务必验证你对此的代理配置更改[架构](https://aka.ms/iot-security-github-module-schema)。
如果配置对象与架构不匹配，将不会启动该代理。

## <a name="configuration-schema-and-validation"></a>配置架构和验证 

请确保用于验证对此代理配置[架构](https://aka.ms/iot-security-github-module-schema)。 如果配置对象与架构不匹配，将不会启动代理。

 
如果在运行时代理，配置对象更改为非有效的配置 （配置与架构不匹配），该代理将忽略无效的配置，并将继续使用当前配置。 

## <a name="editing-a-property"></a>编辑属性 

所有自定义属性必须设置内的代理配置对象内**azureiotsecurity**模块孪生。
若要使用默认属性值，请从配置对象删除的属性。

### <a name="setting-a-property"></a>设置属性

1. 在 IoT 中心，找到并选择你想要更改的设备。

1. 单击你的设备，然后在**azureiotsecurity**模块。

1. 单击**孪生模块标识**。

1. 编辑安全模块的所需的属性。
   
   例如，若要将连接事件配置为高优先级和收集每 7 分钟的高优先级事件，使用下面的配置。
   
   ```json
    "desired": {
      "azureiot*com^securityAgentConfiguration^1*0*0": {
        "highPriorityMessageFrequency": "PT7M",    
        "eventPriorityConnectionCreate": "High" 
      } 
    }, 
    ```

1. 单击“ **保存**”。

### <a name="using-a-default-value"></a>使用默认值

若要使用默认属性值，请从配置对象删除的属性。

## <a name="default-properties"></a>默认属性 

下表包含有关 IoT 安全代理 ASC 可以控制属性。

默认值都正确架构中可用[Github](https://aka.ms/iot-security-module-default)。

| 名称| 状态 | 有效值| 默认值| 描述 |
|----------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
|highPriorityMessageFrequency|所需： false |有效值：格式为 ISO 8601 持续时间 |默认值：PT7M |发送高优先级的消息之前的最大时间。|
|lowPriorityMessageFrequency |所需： false|有效值：格式为 ISO 8601 持续时间 |默认值：PT5H |发送低优先级的消息之前的最大时间。| 
|snapshotFrequency |要求： false|无效值： 的持续时间以 ISO 8601 格式 |默认值 PT13H |设备状态快照的创建的时间间隔。| 
|maxLocalCacheSizeInBytes |所需： false |有效值： |默认值：2560000，大于 8192 | （以字节为单位） 的代理的消息缓存允许的最大存储。 最大允许在设备上存储消息，发送消息前的空间量。| 
|maxMessageSizeInBytes |所需： false |有效值：大于 8192，小于 262144 的正数值 |默认值：204800 |允许的最大大小的代理向云的消息。 此设置控制每个消息中发送的最大数据的量。 |
|eventPriority${EventName} |所需： false |有效值：最高价、 最低价，关闭 |默认值： |每个代理的优先级生成事件 | 

### <a name="supported-security-events"></a>支持的安全事件

|事件名称| PropertyName | 默认值| 快照事件| 详细信息状态  |
|----------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
|诊断事件|eventPriorityDiagnostic| 关闭| False| 代理相关的诊断事件。 使用此事件的详细日志记录。| 
|配置错误 |eventPriorityConfigurationError |低 |False |代理无法分析配置。 验证针对架构的配置。| 
|已删除的事件的统计信息 |eventPriorityDroppedEventsStatistics |低 |True|代理与相关的事件的统计信息。 |
|消息的统计信息|eventPriorityMessageStatistics |低 |True |代理与相关的消息的统计信息。 |
|已连接的硬件|eventPriorityConnectedHardware |低 |True |快照的所有硬件连接到设备。|
|侦听端口|eventPriorityListeningPorts |高 |True |在设备上的所有打开侦听端口的快照。|
|进程创建 |eventPriorityProcessCreate |低 |False |审核进程创建在设备上。|
|进程终止|eventPriorityProcessTerminate |低 |False |审核进程终止在设备上。| 
|系统信息 |eventPrioritySystemInformation |低 |True |系统信息的快照 (例如：操作系统或 CPU）。| 
|本地用户| eventPriorityLocalUsers |高 |True|在系统中已注册的本地用户的快照。 |
|登录|  eventPriorityLogin |高|False|审核到设备 （本地和远程登录名） 的登录事件。|
|创建连接 |eventPriorityConnectionCreate|低|False|创建与设备的 TCP 连接会进行审核。 |
|防火墙配置| eventPriorityFirewallConfiguration|低|True|设备防火墙配置 （防火墙规则） 的快照。 |
|OS 基线| eventPriorityOSBaseline| 低|True|检查设备 OS 基线快照。|
 

## <a name="next-steps"></a>后续步骤

- [了解有关 IoT 建议 ASC](concept-recommendations.md)
- [探索 IoT 警报 ASC](concept-security-alerts.md)
- [访问原始安全数据](how-to-security-data-access.md)
