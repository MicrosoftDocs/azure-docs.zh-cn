---
title: 在 Azure 安全中心管理安全警报 | Microsoft 文档
description: 本文档旨在帮助使用 Azure 安全中心功能来管理和响应安全警报。
services: security-center
documentationcenter: na
author: rkarlin
manager: barbkess
editor: ''
ms.assetid: b88a8df7-6979-479b-8039-04da1b8737a7
ms.service: security-center
ms.topic: conceptual
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/22/2018
ms.author: rkarlin
ms.openlocfilehash: 582912160c8ed514401be3522e52dcc6eb45d263
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235764"
---
# <a name="managing-and-responding-to-security-alerts-in-azure-security-center"></a>管理和响应 Azure 安全中心的安全警报
本文档旨在帮助使用 Azure 安全中心来管理和响应安全警报。

> [!NOTE]
> 若要启用高级检测，请升级到 Azure 安全中心标准版。 免费试用版已提供。 要升级，请选择 [安全策略](tutorial-security-policy.md)中的“定价层”。 请参阅 [Azure 安全中心定价](security-center-pricing.md)，了解详细信息。
>
>

## <a name="what-are-security-alerts"></a>什么是安全警报？
安全中心会自动收集、分析以及整合 Azure 资源、网络和所连合作伙伴解决方案（如，防火墙和终结点保护解决方案）的日志数据，检测真正的威胁并减少误报。 安全中心显示了一系列安全警报（按严重程度排序），并显示了快速调查问题所需的信息以及修复攻击的建议。


> [!NOTE]
> 有关安全中心检测功能工作原理的详细信息，请参阅 [Azure 安全中心检测功能](security-center-detection-capabilities.md)。
>
>

## <a name="managing-security-alerts"></a>管理安全警报
可以通过查看“安全警报”  磁贴查看当前警报。 按照以下步骤查看每个警报的更多详细信息：

1. 在安全中心仪表板中，可以看到“安全警报”磁贴。

    ![安全中心的“安全警报”磁贴](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig1-ga.png)

2. 单击该磁贴打开“安全警报”，查看有关警报的更多详细信息。

   ![安全中心的“安全警报”](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig2-ga.png)

此页底部是每个警报的详细信息。 要进行排序，请单击按其排序的列。 下面给出了每个列的定义：

* **说明**：警报的简要说明。
* **计数**：在特定日期检测到的此特定类型的所有警报的列表。
* **检测方**：负责触发警报的服务。
* **日期**：事件发生的日期。
* **状态**：该警报的当前状态。 状态有两种：
  * **活动**：已检测到安全警报。
  * **已消除**：用户已消除安全警报。 此状态通常用于已调查但已缓解或发现不是真正攻击的警报。
* **严重性**：严重性级别，可以是“高”、“中”或“低”。

> [!NOTE]
> 安全中心生成的安全警报也会显示在“Azure 活动日志”下。 若要深入了解如何访问 Azure 活动日志，请阅读[查看活动日志以审核对资源的操作](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit)。
>


### <a name="alert-severity"></a>警报严重性

-   **高**：资源遭到泄露的可能性较高。 应立即进行调查。 安全中心在所检测出的恶意意图和用于发出警报的发现结果方面的可信度较高。 例如，检测到执行已知的恶意工具的警报，例如用于凭据盗窃的一种常见工具 Mimikatz。 
-   **中**：这可能是一个可疑活动，此类活动可能表明资源遭到泄露。
安全中心对分析或发现结果的可信度为中等，所检测到的恶意意图的可信度为中等到高。 这些通常是机器学习或基于异常的检测。 例如，从异常位置进行的登录尝试。
-   **低**：这可能是无危险或已被阻止的攻击。 
    - 安全中心不足以肯定此意图是否带有恶意，并且此活动可能无恶意。 例如，日志清除是当攻击者尝试隐藏踪迹时可能发生的操作，但在许多情况下此操作是由管理员执行的例行操作。
    - 安全中心通常不会告知你攻击何时被阻止的，除非这是我们建议你应该仔细查看的一个引发关注的案例。 
-   **信息性**:只有在深化到某个安全事件时，或者如果将 REST API 与特定警报 ID 配合使用，才会看到信息警报。 一个事件通常由大量警报组成，有一些警报可能仅会单独出现以提供信息，而其他一些警报的上下文中的信息可能值得你深入探查一下。  

> [!NOTE]
> 如果使用的**2015年-06-01-预览**API 版本，则会出现差异中的警报严重性类型应用于哪些情况下，从上面列出的内容。  

### <a name="filtering-alerts"></a>筛选警报
可以根据日期、状态和严重性筛选警报。 对于需要缩小安全警报显示范围的情况，筛选警报非常有用。 例如，假设正在调查系统中的潜在危害，需要处理过去 24 小时内发生的安全警报。

1. 单击“安全警报”上的“筛选器”。 此时会打开“筛选器”，请选择要查看的日期、状态和严重性值。

    ![筛选安全中心的警报](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig3-2017.png)

### <a name="respond-to-security-alerts"></a>响应安全警报
选择某个安全警报，了解触发该警报的事件详细信息以及修复攻击所需的步骤（如有）。 安全警报按类型和日期进行分组。 单击某个安全警报会打开一个页面，其中包含一个分组警报列表。

![响应 Azure 安全中心的安全警报](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig5-ga.png)

在这种情况下，触发的警报指的是可疑的远程桌面协议 (RDP) 活动。 第一列显示受到攻击的资源；第二列显示该资源受到攻击的次数；第三列显示攻击的时间；第四列显示警报的状态；第五列显示攻击的严重性。 查看此信息后，请单击受攻击的资源。

![关于如何处理 Azure 安全中心安全警报的建议](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig6-ga.png)

在“说明”字段可以找到此事件的更多详细信息。 通过这些额外的详细信息，可深入了解是什么触发了该安全警报、其目标资源、源 IP 地址（如果适用）以及修复建议。  在某些情况下，源 IP 地址为空（不可用），因为并不是所有 Windows 安全事件日志都包含 IP 地址。

安全中心提供的修正建议因安全警报而异。 在某些情况下，可能需要使用其他 Azure 功能来实现建议的修复操作。 例如，此攻击的修复方法是，使用[网络 ACL](../virtual-network/virtual-networks-acl.md) 或[网络安全组](../virtual-network/security-overview.md#security-rules)规则将生成此攻击的 IP 地址加入阻止名单。 有关不同类型的警报的详细信息，请参阅 [Azure 安全中心的安全警报（按类型）](security-center-alerts-type.md)。

> [!NOTE]
> 安全中心在功能有限的预览版中发布了新的一组检测。这些检测利用审核记录这种常用审核框架，检测 Linux 计算机上的恶意行为。 若要体验预览版，请向[我们](mailto:ASC_linuxdetections@microsoft.com)发送包含订阅 ID 的电子邮件。


## <a name="see-also"></a>另请参阅
本文档中已经介绍了如何在安全中心配置安全策略。 要了解有关安全中心的详细信息，请参阅以下内容：

* [处理 Azure 安全中心的安全事件](security-center-incident.md)
* [Azure 安全中心检测功能](security-center-detection-capabilities.md)
* [Azure 安全中心规划和操作指南](security-center-planning-and-operations-guide.md)
* [Azure 安全中心常见问题解答](security-center-faq.md) — 查找有关使用该服务的常见问题解答。
* [Azure 安全性博客](https://blogs.msdn.com/b/azuresecurity/) - 查找关于 Azure 安全性及合规性的博客文章。
