---
title: 开始使用针对 IoT 预览版的 Azure 安全中心 (ASC) |Microsoft Docs
description: 立即开始了解的基本工作流的 Azure 安全中心 IoT 功能和服务。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 55c8d3b6-3126-4246-8d07-ef88fe5ea84f
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: bdc5e858286d7db03281408cf3ed00e0760e0c3a
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200624"
---
# <a name="get-started-with-azure-security-center-for-iot"></a>开始使用 Azure 安全中心的 IoT 

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文为 IoT 服务提供不同的构造块的 Azure 安全中心 (ASC) 的说明，并说明如何开始使用[启用服务](quickstart-onboard-iot-hub.md)。 

适用于 IoT 的 ASC 可以无缝集成到 IoT 中心提供安全分析 IoT 中心配置、 设备标识和中心设备的通信模式。
增强的安全性功能，对于 ASC 为 IoT 提供了基于代理从 IoT 设备的安全数据集合。

## <a name="asc-for-iot-seamless-iot-hub-integration"></a>ASC 为 IoT IoT 中心的无缝集成的

在尝试保护单个 IoT 设备时，直接从设备，或从其网络收集数据的功能需要。 若要支持这项工作，ASC 为 IoT 提供了一整套低内存占用安全代理，以提供设备监视和强化。

在 ASC IoT 预览版中，对于 Linux 和 Windows 中的安全代理，参考体系结构C#和 C 提供。
代理处理从设备操作系统，以降低成本，并通过设备模块孪生的配置的事件聚合的原始事件集合。
安全消息通过 IoT 中心，发送到 ASC 为 IoT 分析服务。

## <a name="asc-for-iot-basics"></a>ASC 为 IoT 基础知识的

选择最符合你的 IoT 设备和环境要求的工作流方案：

### <a name="get-started-with-asc-for-iot-seamless-iot-hub-integration"></a>开始使用 ASC 适用于 IoT IoT 中心的无缝集成 

>[!Note]
>此工作流，可使用该服务，而不使用 ASC 适用于 IoT 安全代理。 

若要启用监视你的设备标识管理，设备到云和云到设备通信模式，请使用以下基本工作流进行测试和启动服务： 

1. [为 IoT 中心上的 IoT 服务启用 ASC](quickstart-onboard-iot-hub.md)
1. 如果你的 IoT 中心都不有任何已注册的设备[注册新设备](https://docs.microsoft.com/azure/iot-accelerators/quickstart-device-simulation-deploy)。
1. [创建你的设备将 azureiotsecurity 安全模块](quickstart-create-security-twin.md)为你的设备。 
1. 定义通过正常的设备和系统行为[自定义警报](quickstart-create-custom-alerts.md)。 
1. 执行系统测试，以验证服务和设备状态。 
1. 探索[警报](concept-security-alerts.md)，[建议](concept-recommendations.md)，并[使用 Log Analytics 的深入探讨](how-to-security-data-access.md)使用 IoT 中心。 


### <a name="get-started-with-asc-for-iot-security-agents"></a>开始使用 ASC IoT 安全代理

请使用 ASC IoT 增强安全功能，例如通过使用以下基本工作流进行测试和启用该服务监视远程连接、 活动应用程序、 登录事件和 OS 配置最佳做法： 

1. [为到 IoT 中心的 IoT 服务启用 ASC](quickstart-onboard-iot-hub.md)
1. 如果你的 IoT 中心都不有任何已注册的设备[注册新设备](https://docs.microsoft.com/azure/iot-accelerators/quickstart-device-simulation-deploy)。
1. [创建 azureiotsecurity 安全模块](quickstart-create-security-twin.md)为你的设备。
1. 若要在 Azure 而不是实际的设备上安装模拟设备上安装代理[数值调节钮向上新 Azure 虚拟机 (VM)](https://docs.microsoft.com/azure/virtual-machines/linux/quick-create-portal)可用区域中。 
1. [部署 IoT 安全代理 ASC](how-to-deploy-linux-cs.md) IoT 设备或新的 VM 上。
1. 按照说明[trigger_events](https://aka.ms/iot-security-github-trigger-events)运行模拟无害的攻击。
1. IoT 中的警报响应模拟攻击上, 一步骤中验证 ASC。 开始验证 5 分钟后运行脚本。
1. 探索[警报](concept-security-alerts.md)，[建议](concept-recommendations.md)，并[使用 Log Analytics 的深入探讨](how-to-security-data-access.md)使用 IoT 中心。 

## <a name="next-steps"></a>后续步骤

- 启用[iot ASC](quickstart-onboard-iot-hub.md)
- 配置你[解决方案](quickstart-configure-your-solution.md)
- [创建安全模块](quickstart-create-security-twin.md)
- 配置[自定义警报](quickstart-create-custom-alerts.md)
- [部署安全代理](how-to-deploy-agent.md)
