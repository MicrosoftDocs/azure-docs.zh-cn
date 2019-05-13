---
title: 威胁智能数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何连接到 Azure Sentinel 威胁智能数据。
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 56412543-5664-44c1-b026-2dbaf78a9a50
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 266e487a7c345f75e966afbde567c5bc4683b5c0
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233759"
---
# <a name="connect-data-from-threat-intelligence-providers"></a>威胁智能提供商处从连接的数据 

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

流式数据传输到 Azure Sentinel 后，你可以加强其威胁智能源与整个组织内使用的。 

若要使您能够跨检查警报和规则使用，则返回 true 的威胁智能，例如，如果警报是由特定的 IP 地址，将能够让你知道是否最近找到该 IP 地址为恶意威胁智能提供程序集成Azure Sentinel 可与集成[威胁智能提供商处](https://aka.ms/graphsecuritytips)。 

可以通过单击一次流式威胁智能提供程序中的日志传输到 Azure Sentinel。 此连接，可将合并指标包含各种类型的可观察量，如 IP 地址、 域名、 URL 和文件哈希以搜索以及创建自定义警报规则在 Azure Sentinel。  
> [!NOTE]
> 你可以输入自定义的威胁指示器 Azure Sentinel 在警报规则、 仪表板，以及搜索方案中使用通过集成[Microsoft Graph Security tiIndicator](https://aka.ms/graphsecuritytiindicators)实体或通过使用[Microsoft图形安全集成威胁智能平台](https://aka.ms/graphsecuritytips)。

## <a name="prerequisites"></a>必备组件  

- 具有全局管理员或安全管理员权限的用户 

- 威胁智能应用程序与 Microsoft Intelligent Security Graph 集成 

## <a name="connect-to-threat-intelligence"></a>连接到的威胁情报 

1. 如果你已在使用威胁智能提供程序，请确保以浏览到提示应用程序并授予权限以向 Microsoft 发送指标并指定该服务为 Azure Sentinel。  

2. 在 Azure Sentinel，选择**数据连接器**，然后单击**威胁智能**磁贴。

3. 单击“连接”。 

4. 若要在 Log Analytics 中相关的架构用于威胁情报馈送，搜索**ThreatIntelligenceIndicator**。 

 
## <a name="next-steps"></a>后续步骤

在本文档中，您学习了如何连接到 Azure Sentinel 威胁智能提供程序。 若要了解有关 Azure Sentinel 的详细信息，请参阅以下文章。

- 若要开始使用 Azure Sentinel，需要订阅 Microsoft Azure。 如果尚无订阅，可注册[免费试用版](https://azure.microsoft.com/free/)。
- 了解如何[将数据载入到 Azure Sentinel](quickstart-onboard.md)，以及[获取数据和潜在威胁的见解](quickstart-get-visibility.md)。
