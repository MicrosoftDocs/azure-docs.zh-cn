---
title: 连接到 Azure Sentinel 预览版的 Azure AD 数据 |Microsoft Docs
description: 了解如何将 Azure Active Directory 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 0a8f4a58-e96a-4883-adf3-6b8b49208e6a
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 357435b8a4ac396c1548c89206f269730e871f6b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204497"
---
# <a name="connect-data-from-azure-active-directory"></a>将数据从 Azure Active Directory 连接

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Azure Sentinel，可从中收集数据[Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)和其流式处理到 Azure Sentinel。 你可以选择流[登录日志](../active-directory/reports-monitoring/concept-sign-ins.md)并[审核日志](../active-directory/reports-monitoring/concept-audit-logs.md)。

## <a name="prerequisites"></a>必备组件

- 如果你想要从 Active Directory 导出登录数据，必须具有 Azure AD P1 或 P2 许可证。

- 具有你想要从日志流式传输的租户的全局管理员或安全管理权限的用户。


## <a name="connect-to-azure-ad"></a>连接到 Azure AD

1. 在 Azure Sentinel，选择**数据连接器**，然后单击**Azure Active Directory**磁贴。

2. 你想要流式传输到 Azure Sentinel 的日志，旁边单击**Connect**。

6. 若要使用 Log Analytics 中的 Azure AD 警报相关的架构，搜索**SigninLogs**并**AuditLogs**。




## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Azure AD 连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
