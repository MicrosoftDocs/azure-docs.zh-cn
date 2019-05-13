---
title: 将 Azure AD Identity Protection 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Azure AD Identity Protection 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 91c870e5-2669-437f-9896-ee6c7fe1d51d
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 10dc31e21f20618450de6d99b3fce40d63272d31
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204371"
---
# <a name="connect-data-from-azure-ad-identity-protection"></a>从 Azure AD Identity Protection 连接数据

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

可以从日志流式传输[Azure AD Identity Protection](https://docs.microsoft.com/azure/information-protection/reports-aip)到 Azure Sentinel 流到 Azure Sentinel 若要查看的仪表板、 创建自定义警报和改进的调查的警报。 Azure Active Directory Identity Protection 提供存在风险的用户、 风险事件和漏洞的综合的视图，能够立即修复风险，并设置策略以自动修复以后的事件。 该服务基于 Microsoft 保护用户身份的经验，并获得超过一天的 13 亿个日志的项的信号的准确性。 


## <a name="prerequisites"></a>必备组件

- 您必须具有[Azure Active Directory Premium P1 或 P2 许可证](https://azure.microsoft.com/pricing/details/active-directory/)
- 具有全局管理员或安全管理员权限的用户


## <a name="connect-to-azure-ad-identity-protection"></a>连接到 Azure AD Identity Protection

如果已有 Azure AD Identity Protection，请确保它是[在网络上启用](../active-directory/identity-protection/enable.md)。
如果 Azure AD Identity Protection 已部署且获取数据，警报数据可以轻松地进行流式传输到 Azure Sentinel。


1. 在 Azure Sentinel，选择**数据连接器**，然后单击**Azure AD Identity Protection**磁贴。

2. 单击**Connect**开始 Azure AD Identity Protection 事件传输到 Azure Sentinel。


6. 若要使用 Log Analytics 中的 Azure AD Identity Protection 警报相关的架构，搜索**IdentityProtectionLogs_CL**。

## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Azure AD Identity Protection 连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
