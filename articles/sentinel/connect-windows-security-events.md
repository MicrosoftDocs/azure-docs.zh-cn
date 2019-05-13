---
title: 将 Windows 安全事件数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何连接到 Azure Sentinel Windows 安全事件数据。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: d51d2e09-a073-41c8-b396-91d60b057e6a
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 88b066818d53fd92e8238e270b9bc785d4275186
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233951"
---
# <a name="connect-windows-security-events"></a>连接 Windows 安全事件 

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

可以对来自连接到 Azure Sentinel 工作区的 Windows 服务器的所有安全事件进行流式都传输。 此连接可以查看仪表板、 创建自定义警报和改进的调查。 这为您提供了更详细地了解您组织的网络，并提高你的安全操作功能。  您可以选择哪些事件进行流式处理：

- **所有事件**-所有 Windows 安全性和 AppLocker 事件。
- **常见**-一组标准的审核的事件。
- **最小**-少量的事件可能表示潜在的威胁。 通过启用此选项，你将无法具有完整的审核跟踪。
- **无**-没有安全性或 AppLocker 事件。

> [!NOTE]
> 
> - 数据将存储在其运行 Azure Sentinel 的工作区的地理位置。

## <a name="set-up-the-windows-security-events-connector"></a>设置 Windows 安全事件连接器

若要完全将与 Azure Sentinel 集成在 Windows 安全事件：

1. 在 Azure Sentinel 门户中，选择**数据连接器**，然后单击**Windows 安全事件**磁贴。 
1. 选择你想要流式传输的数据类型。
1. 单击“更新”。
6. 若要使用 Log Analytics 中的 Windows 安全事件相关的架构，搜索**SecurityEvent**。

## <a name="validate-connectivity"></a>验证连接

可能需要约 20 分钟，直到你的日志开始在 Log Analytics 中显示。 



## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Windows 安全事件连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

