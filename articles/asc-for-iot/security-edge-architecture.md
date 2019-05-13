---
title: 了解 Azure 安全中心的 IoT 安全 IoT Edge 模块 |Microsoft Docs
description: 了解 IoT Edge 体系结构和 Azure 安全中心的 IoT 安全模块的功能。
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
ms.openlocfilehash: 847d6238bd42dc7da723dcc9acd47ed09c16dbf9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200584"
---
# <a name="azure-iot-edge-security-module"></a>Azure IoT Edge 安全模块

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版本没有附带服务级别协议提供，不建议用于生产 worklo§1ads。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

[Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/)提供了强大的功能来管理和执行在边缘的业务工作流。
IoT Edge 在 IoT 环境中扮演的关键部分使其成为尤其吸引人恶意执行组件。

IoT 安全模块的 azure 安全中心 (ASC) 提供了 IoT Edge 设备的全面的安全解决方案。
ASC 为 IoT 模块收集、 聚合和分析来自你的操作系统和容器系统的原始安全数据到切实可行的安全建议和警报。

与 ASC 为 IoT 设备的 IoT 安全代理类似，ASC 为 IoT Edge 模块是通过其模块孪生高度可自定义。
请参阅[将代理配置为](how-to-agent-configuration.md)若要了解详细信息。

IoT 安全 IoT Edge 模块的 ASC 提供以下功能：

- 从基础操作系统 (Linux) 和 IoT Edge 容器系统收集原始安全事件。
  
  请参阅[ASC IoT 代理配置为](how-to-agent-configuration.md)若要了解有关可用的安全数据收集器的详细信息。

- IoT Edge 部署清单进行签名的分析。

- 将原始安全事件聚合到通过发送的消息[IoT Edge 中心](https://docs.microsoft.com/azure/iot-edge/iot-edge-runtime#iot-edge-hub)。

- 删除通过使用安全模块孪生的配置。

  请参阅[配置 IoT 代理 ASC](how-to-agent-configuration.md)若要了解详细信息。

ASC 为 IoT 安全模块的 IoT Edge 在 IoT Edge 在特权模式下运行。
特权模式下需要允许模块来监视操作系统和其他 IoT Edge 模块。

## <a name="agent-supported-platforms"></a>支持的代理的平台

IoT 安全 IoT Edge 模块的 ASC 目前仅适用于 Linux。

## <a name="next-steps"></a>后续步骤

在本文中，您学习了有关体系结构和功能的 ASC IoT 安全模块的 IoT Edge。

若要继续开始使用 ASC 为 IoT 部署，请使用以下文章：

- 部署[IoT Edge 的安全模块](how-to-deploy-edge.md)
- 了解如何[配置安全模块](how-to-agent-configuration.md)
- 查看 IoT ASC[服务先决条件](service-prerequisites.md)
- 了解如何[启用 ASC 中 IoT 中心的 IoT 服务](quickstart-onboard-iot-hub.md)
- 要详细了解从服务[ASC 为 IoT 常见问题](resources-frequently-asked-questions.md)