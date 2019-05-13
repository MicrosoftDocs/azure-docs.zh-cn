---
title: 连接到 Azure Sentinel 预览版的 Microsoft web 应用程序防火墙数据 |Microsoft Docs
description: 了解如何将 Microsoft web 应用程序防火墙数据连接到 Azure Sentinel。
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
ms.openlocfilehash: 99c553145f0a600826e48bec437449f278b74fbf
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205641"
---
# <a name="connect-data-from-microsoft-web-application-firewall"></a>从 Microsoft web 应用程序防火墙连接数据

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

您可以从 Azure 应用程序网关的 Microsoft web 应用程序防火墙 (WAF) 的日志流式传输。 此 WAF 保护你的应用程序免受常见 web 漏洞，如 SQL 注入和跨站点脚本，并允许你自定义规则，以减少误报。 按照这些说明你的 Microsoft Web 应用程序防火墙日志流式传输到 Azure Sentinel。


## <a name="prerequisites"></a>必备组件

- 现有应用程序网关资源

## <a name="connect-to-microsoft-web-application-firewall"></a>连接到 Microsoft web 应用程序防火墙

如果已有 Microsoft web 应用程序防火墙，请确保你有现有网关资源。
一旦部署你的 Microsoft web 应用程序防火墙，且获取数据，警报数据可以轻松地进行流式传输到 Azure Sentinel。
    
1. 在 Azure Sentinel 门户中，选择**数据连接器**。
1. 在数据连接器页中，选择**WAF**磁贴。
1. 转到[应用程序网关资源](https://ms.portal.azure.com/#blade/HubsExtension/BrowseAllResourcesBlade/resourceType/Microsoft.Network%2FapplicationGateways) ，然后选择在 WAF。
    1. 选择 **诊断设置**。
    1. 选择 **+ 添加诊断设置**在表的下方。
    1. 在中 **诊断设置**页上，键入 **名称**，然后选择 **发送到 Log Analytics**。
    1. 下**Log Analytics 工作区**选择 Azure Sentinel 工作区。
    1. 选择你想要分析的日志类型。 我们建议：ApplicationGatewayAccessLog 和 ApplicationGatewayFirewallLog。
1. 若要使用 Log Analytics 中的 Microsoft web 应用程序防火墙警报相关的架构，搜索**AzureDiagnostics**。

## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何连接到 Azure Sentinel Microsoft web 应用程序防火墙。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
