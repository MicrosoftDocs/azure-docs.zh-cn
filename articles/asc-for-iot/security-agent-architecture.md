---
title: 了解 Azure 安全中心的 IoT 安全代理体系结构预览 |Microsoft Docs
description: 了解用于在 Azure 安全中心的 IoT 服务使用的代理安全代理体系结构。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: e78523ae-d70a-456a-818d-f8b1b025d7cb
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/25/2019
ms.author: mlottner
ms.openlocfilehash: 3c05b7e9b1c6d1b9214da168f7abfcbb322f8f6d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192518"
---
# <a name="security-agent-reference-architecture"></a>安全代理参考体系结构

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


Azure 安全中心 (ASC) 适用于 IoT 的安全代理，记录、 处理、 聚合和发送通过 IoT 中心的安全数据提供参考体系结构。

安全代理用于处理在受约束 IoT 环境中，并且高度可自定义方面时与他们使用的资源相比，它们提供的值。

安全代理支持以下功能：

- 从基础操作系统 （Linux、 Windows） 收集原始安全事件。 若要了解有关可用的安全数据收集器的详细信息，请参阅[ASC IoT 代理配置为](how-to-agent-configuration.md)。

- 聚合成通过 IoT 中心发送消息的原始安全事件。

- 使用现有的设备标识或专用的模块标识进行身份验证。 请参阅[安全代理身份验证方法](concept-security-agent-authentication-methods.md)若要了解详细信息。

- 通过使用远程配置**azureiotsecurity**模块孪生。 若要了解详细信息，请参阅[配置 IoT 代理 ASC](how-to-agent-configuration.md)。

IoT 安全代理的 ASC 作为开放源代码项目开发的可从 GitHub: 

- [IoT C 基于代理的 ASC](https://github.com/Azure/Azure-IoT-Security-Agent-C) 
- [适用于 IoT 的 ASC C#-基于代理](https://github.com/Azure/Azure-IoT-Security-Agent-CS)

## <a name="agent-supported-platforms"></a>支持的代理的平台

ASC 为 IoT 提供了不同的安装程序的 32 位和 64 位 Windows，代理和相同的 32 位和 64 位 Linux。 请确保具有正确的代理安装程序为每个设备根据下表：

| 32 位还是 64 位 | Linux | Windows |    详细信息|
|----------|----------------------------------------------|-------------|-------------------------------------------|
| 32 位  | C  | C#  ||
| 64 位  | C#或 C           | C#      | 使用 C 代理进行最少的资源的设备|

## <a name="next-steps"></a>后续步骤

在本文中，您了解的关于 ASC 的 IoT 安全代理体系结构和可用安装程序。

若要继续开始使用 ASC 为 IoT 部署，请使用以下文章：

- 了解[安全代理身份验证方法](concept-security-agent-authentication-methods.md)
- 选择并将其部署[安全代理](how-to-deploy-agent.md)
- 查看 IoT ASC[服务先决条件](service-prerequisites.md)
- 了解如何[启用 ASC 中 IoT 中心的 IoT 服务](quickstart-onboard-iot-hub.md)
- 要详细了解从服务[ASC 为 IoT 常见问题](resources-frequently-asked-questions.md)
