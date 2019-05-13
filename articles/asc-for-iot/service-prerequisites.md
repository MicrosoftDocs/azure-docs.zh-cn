---
title: Azure 安全中心的 IoT 先决条件预览 |Microsoft Docs
description: 若要开始使用 Azure 安全中心的 IoT 服务先决条件所需的所有内容的详细信息。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 790cbcb7-1340-4cc1-9509-7b262e7c3181
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: 973b0044844fbdf38eefde2072631586a8ace593
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192606"
---
# <a name="azure-security-center-for-iot-prerequisites"></a>Azure 安全中心，IoT 系统必备组件

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文提供了不同的构造块的 Azure 安全中心 (ASC) 的 IoT 服务，您需要开始，基本概念以帮助您了解该服务的说明。 

## <a name="minimum-requirements"></a>最低要求

- IoT 中心标准层
    - RBAC 角色**所有者**级别权限 
- [Log Analytics 工作区](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace) 
- Azure 安全中心 （推荐）
    - Azure 安全中心的使用时仅为建议并不是必需的如果没有它，您将无法查看 IoT 中心内的其他 Azure 资源。 
 
## <a name="working-with-asc-for-iot-service"></a>使用 ASC 的 IoT 服务

可使用 Azure IoT 中心和 Azure 安全中心 ASC 为 IoT 见解和报告。 若要为 Azure IoT 中心，具有的帐户上的 IoT 启用 ASC**所有者**级别权限是必需的。 在启用之后 ASC iot 在 IoT 中心，ASC 获得 IoT 见解显示为**安全**功能在 Azure IoT 中心并作为**IoT** Azure 安全中心中。 

## <a name="supported-service-regions"></a>支持的服务区域 

以下 Azure 区域中的 IoT 中心目前支持适用于 IoT 的 ASC:
  - 美国中部
  - 欧洲北部
  - 东南亚

## <a name="wheres-my-iot-hub"></a>我的 IoT 中心位于何处？

检查你的 IoT 中心位置，以便在开始之前验证服务可用性。 

1. 打开 IoT 中心。 
2. 单击“概览”。 
3. 验证是否列出的地点与匹配之一[支持的服务区域](#supported-service-regions)。 


## <a name="supported-platforms-for-agents"></a>代理支持的平台 

ASC 为 IoT 代理支持越来越多的设备和平台。 请参阅[支持的平台列表](how-to-deploy-agent.md)来检查您现有或计划的设备库。  

## <a name="next-steps"></a>后续步骤
- [概述](overview.md)
- [启用该服务](quickstart-onboard-iot-hub.md)
- [ASC iot 常见问题](resources-frequently-asked-questions.md)
- [了解 IoT 警报 ASC](concept-security-alerts.md)
