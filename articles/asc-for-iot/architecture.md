---
title: 了解 Azure 安全中心的 IoT 解决方案体系结构预览 |Microsoft Docs
description: 了解有关流的 IoT 服务在 Azure 安全中心中的信息。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 2cf6a49b-5d35-491f-abc3-63ec24eb4bc2
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/24/2019
ms.author: mlottner
ms.openlocfilehash: 3c775c8cb99b056c7f468c028a80dd0b27ae6438
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200684"
---
# <a name="azure-security-center-for-iot-architecture"></a>Azure 安全中心的 IoT 体系结构

本文介绍了功能的系统体系结构有关的 IoT 解决方案的 Azure 安全中心 (ASC)。 

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="asc-for-iot-components"></a>ASC 为 IoT 组件的

适用于 IoT 的 ASC 由以下组件组成：
- 设备代理
- 发送安全消息 SDK
- IoT 中心的集成
- 分析管道
 
### <a name="asc-for-iot-workflow"></a>ASC 为 IoT 工作流的

ASC 为 IoT 设备代理，可轻松地从设备收集的原始安全事件。 原始安全事件可以包括 IP 连接、 创建进程、 用户登录名和其他安全相关信息。 ASC 为 IoT 设备代理还处理事件聚合以帮助避免高网络吞吐量。 代理是高度可自定义，从而允许您使用它们有关特定任务，例如发送的重要信息仅在最快的 SLA，或用于将提供广泛的安全信息和上下文聚合到更大的段，避免成本较高的服务。
 
设备代理和其他应用程序用途**Azure ASC 发送安全消息 SDK**发送到 Azure IoT 中心的安全信息。 IoT 中心提取此信息并将其转发到 ASC 为 IoT 服务。

启用 ASC 为 IoT 服务后，除了转发的数据，IoT 中心还会发送出所有 asc 分析其内部数据的 IoT。 此数据包括设备云操作日志、 设备标识和集线器配置。 所有这些信息有助于创建 ASC 为 IoT 分析管道。
 
IoT 分析管道的 ASC 还从 Microsoft 和 Microsoft 中的各种源中接收其他威胁智能流合作伙伴。 ASC 为 IoT 整个分析管道可用于对该服务 （如自定义警报和使用的发送安全消息 SDK） 发出的每个客户配置。
 
使用分析管道，iot ASC 将合并所有流的信息来生成可行的建议和警报。 管道包含两个自定义规则创建的安全研究人员和专家以及机器学习搜索偏离标准设备行为和风险分析的模型。
 
ASC 为 IoT 建议和警报 （分析管道输出） 写入到 Log Analytics 工作区中的每个客户。 在工作区，以及警报和建议中包括的原始事件，深入调查和使用检测到的可疑活动的具体详细信息的查询。  

## <a name="next-steps"></a>后续步骤

在本文中，您学习了基本体系结构和工作流的 ASC IoT 解决方案。 若要详细了解先决条件，如何开始并启用您在 IoT 中心的安全解决方案，请参阅以下文章：

- [服务的先决条件](service-prerequisites.md)
- [入门](getting-started.md)
- [配置解决方案](quickstart-configure-your-solution.md)
- [启用 IoT 中心内的安全性](quickstart-onboard-iot-hub.md)
- [ASC iot 常见问题](resources-frequently-asked-questions.md)
- [ASC 为 IoT 安全警报的](concept-security-alerts.md)

