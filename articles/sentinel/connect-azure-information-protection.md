---
title: 连接到 Azure Sentinel 预览版的 Azure 信息保护数据 |Microsoft Docs
description: 了解如何将 Azure 信息保护中 Azure Sentinel 数据连接。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 4b73edd10521b23fb4befbe4fe7d9f0c7b496de3
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204304"
---
# <a name="connect-data-from-azure-information-protection"></a>Azure 信息保护中连接数据

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

可以从日志流式传输[Azure 信息保护](https://docs.microsoft.com/azure/information-protection/reports-aip)到 Azure Sentinel 单击一次。 Azure 信息保护有助于保护你的数据，它是存储在云中还是在本地基础结构和控制并帮助保护电子邮件、 文档和向公司之外共享的敏感数据存储。 从简单的分类到嵌入式的标签和权限，增强在所有时间与 Azure 信息保护的数据保护。 当您连接 Azure 信息保护到 Azure Sentinel，你的流的所有警报从 Azure 信息保护到 Azure Sentinel。


## <a name="prerequisites"></a>必备组件

- 使用全局管理员、 安全管理员或信息保护权限的用户


## <a name="connect-to-azure-information-protection"></a>连接到 Azure 信息保护

如果已有 Azure 信息保护，请确保它是[在网络上启用](https://docs.microsoft.com/azure/information-protection/activate-service)。
如果部署 Azure 信息保护，并获取数据，警报数据可以轻松地进行流式传输到 Azure Sentinel。


1. 在 Azure Sentinel，选择**数据连接器**，然后单击**Azure 信息保护**磁贴。

2. 转到[Azure 信息保护门户](https://portal.azure.com/?ScannerConfiguration=true&EndpointDiscovery=true#blade/Microsoft_Azure_InformationProtection/DataClassGroupEditBlade/quickstartBlade) 

3. 下**连接**，设置日志流式传输 Azure 信息保护中到 Azure Sentinel 通过单击[配置分析](https://portal.azure.com/#blade/Microsoft_Azure_InformationProtection/DataClassGroupEditBlade/analyticsOnboardBlade)

4. 选择在其中部署 Azure Sentinel 的工作区。 

5. 单击“确定”。

6. 若要使用 Log Analytics 中的 Azure 信息保护警报相关的架构，搜索**InformationProtectionLogs_CL**。




## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Azure 信息保护连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
