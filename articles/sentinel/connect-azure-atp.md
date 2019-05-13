---
title: 将 Azure ATP 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Azure ATP 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 5bf3cc44-ecda-4c78-8a63-31ab42f43605
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: f0d86a62c59df5bebd34137d0903fcaa7014573d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204272"
---
# <a name="connect-data-from-azure-advanced-threat-protection-atp"></a>将数据连接从 Azure 高级威胁防护 (ATP)

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


可以从日志流式传输[Azure 高级威胁防护](https://docs.microsoft.com/azure-advanced-threat-protection/what-is-atp)到 Azure Sentinel 单击一次。

## <a name="prerequisites"></a>必备组件

- 具有全局管理员或安全管理员权限的用户
- 您必须是 Azure ATP 个人预览版客户

## <a name="connect-to-azure-atp"></a>连接到 Azure ATP

请确保 Azure ATP 个人预览版版本[在网络上启用](https://docs.microsoft.com/azure-advanced-threat-protection/install-atp-step1)。
如果部署 Azure ATP，引入你的数据，可疑警报可以轻松地进行流式传输到 Azure Sentinel。 可能需要 24 小时的警报，若要开始流式传输到 Azure Sentinel。



1. 在 Azure Sentinel，选择**数据连接器**，然后单击**Azure ATP**磁贴。

2. 单击“连接” 。

6. 若要使用 Log Analytics 中的 Azure ATP 警报相关的架构，搜索**SecurityAlert**。

## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Azure 高级威胁防护连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

