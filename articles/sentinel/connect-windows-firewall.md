---
title: 将 Windows 防火墙数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Windows 防火墙数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 0e41f896-8521-49b8-a244-71c78d469bc3
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 9388e267c52ef53b59aacad844e964d3cfeb13d7
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233813"
---
# <a name="connect-windows-firewall"></a>连接 Windows 防火墙

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Windows 防火墙连接器，可轻松地连接 Windows 防火墙日志，如果连接到 Azure Sentinel 工作区。 此连接可以查看仪表板、 创建自定义警报和改进的调查。 这为您提供了更详细地了解您组织的网络，并提高你的安全操作功能。  


> [!NOTE]
> 数据将存储在其运行 Azure Sentinel 的工作区的地理位置。

## <a name="enable-the-connector"></a>启用连接器 

1. 在 Azure Sentinel 门户中，选择**数据连接器**，然后单击**Windows 防火墙**磁贴。 
1. 选择你想要流式传输的数据类型。
1. 单击“安装”。
6. 若要使用 Log Analytics 中的 Windows 防火墙相关的架构，搜索**SecurityEvent**。

## <a name="validate-connectivity"></a>验证连接

它可能需要 1-2 20 分钟，直到你的日志开始在 Log Analytics 中显示。 



## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Windows 防火墙连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

