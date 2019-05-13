---
title: 将 Symantec ICDX 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Symantec ICDX 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: d068223f-395e-46d6-bb94-7ca1afd3503c
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 0410b052f17a868aed70ce407b9c9fdefbe023df
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233633"
---
# <a name="connect-your-symantec-icdx-appliance"></a>连接你的 Symantec ICDX 设备 

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Symantec ICDX 连接器，可轻松地连接所有到 Symantec 安全解决方案日志在 Azure Sentinel，若要查看的仪表板、 创建自定义警报和改进的调查。 这为您提供了更详细地了解您组织的网络，并提高你的安全操作功能。 Symantec ICDX 和 Azure Sentinel 之间的集成，可以使用 REST api。


> [!NOTE]
> 数据将存储在其运行 Azure Sentinel 的工作区的地理位置。

## <a name="configure-and-connect-symantec-icdx"></a>配置和连接 Symantec ICDX 

Symantec ICDX 可以集成并将日志导出 Azure Sentinel 到直接。

1. 打开 ICDX 管理控制台。
2. 在左侧的导航菜单中，选择 **配置**，然后 **转发器**选项卡。
3. 在 Microsoft Azure Log Analytics 行中，单击**更多**后, 跟**编辑**。 
4. 在中**Microsoft Azure Log Analytics 转发器**窗口中，将以下设置：
    - 将自定义日志名称保留为默认情况下，SymantecICDX。
    - 复制工作区 ID 并将其粘贴 **客户标识符**字段。 复制**主键**并将其粘贴在共享密钥字段中。 可以通过选择从 Azure Sentinel 门户中复制这些值**数据连接器**，然后**Symantec ICDX**。
6. 若要使用 Log Analytics 中的 Symantec ICDX 事件相关的架构，搜索**SymantecICDX_CL**。


## <a name="validate-connectivity"></a>验证连接

它可能需要 1-2 20 分钟，直到你的日志开始在 Log Analytics 中显示。 



## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Symantec ICDX 连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

