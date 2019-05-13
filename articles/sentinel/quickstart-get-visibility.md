---
title: Azure Sentinel 快速入门 - Azure Sentinel 预览版入门 | Microsoft Docs
description: Azure Sentinel 快速入门 - Azure Sentinel 入门
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 5a4ae93c-d648-41fb-8fb8-96a025d2f73e
ms.service: sentinel
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 3/20/2019
ms.author: rkarlin
ms.openlocfilehash: 07839c6372e0048048b62fbfedd58bfa9b9942f5
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65207518"
---
# <a name="quickstart-get-started-with-azure-sentinel-preview"></a>快速入门：Azure Sentinel 预览版入门

> [!IMPORTANT]
> Azure Sentinel 目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


本快速入门介绍如何使用 Azure Sentinel 快速查看和监视整个环境中发生的情况。 将数据源连接到 Azure Sentinel 之后，可以即时可视化和分析数据，以了解所有已连接的数据源中发生的情况。 Azure Sentinel 提供仪表板让你利用 Azure 中已提供的工具的强大功能，并提供内置的表和图表用于分析日志与查询。 可以使用内置的仪表板，或者从头开始或基于现有的仪表板轻松创建新的仪表板。 

## <a name="get-visualization"></a>获取可视化效果

若要可视化和分析环境中发生的情况，请先查看概述仪表板，以大致了解组织的安全态势。 可以单击这些磁贴的每个元素，向下钻取到创建这些元素时所依据的原始数据。 为了帮助降低干扰并尽量减少需要检查和调查的警报数目，Azure Sentinel 使用一种融合技术将警报关联到案例。 **案例**是相关警报的分组，它们共同创建了可以调查和解决的可处理事件。

- 在 Azure 门户中选择“Azure Sentinel”，然后选择要监视的工作区。

  ![Azure Sentinel 概览](./media/qs-get-visibility/overview.png)

- 顶部工具栏会告知在选定的时间段发生了多少个事件，并将该数字与过去 24 小时的事件数进行比较。 工具栏会告知，在这些事件中触发了哪些警报（较小的数字表示与过去 24 小时的变化），然后告知其中哪些事件未予处理、正在处理和已结案。 检查事件数是否不存在明显的增加或减少。 如果事件数减少，可能表示某个连接已停止向 Azure Sentinel 报告。 如果事件数增加，可能表示发生了可疑的情况。 检查是否有新的警报。

   ![Azure Sentinel 漏斗图](./media/qs-get-visibility/funnel.png)

概述页的主体提供工作区安全状态的概览：

- **一段时间的事件和警报数**：列出事件数，以及基于这些事件创建的警报数。 如果看到了异常的高峰，应会看到相应的警报 - 如果出现事件高峰时发生了某种异常，但未看到警报，则可能需要引以关注。

- **潜在的恶意事件**：检测到来自已知恶意的源的流量时，Azure Sentinel 会在地图上发出警报。 橙色表示入站流量：有人正在尝试从已知恶意的 IP 地址访问你的组织。 如果看到出站（红色）活动，表示网络中的数据正在从你的组织流向已知恶意的 IP 地址。

   ![Azure Sentinel 地图](./media/qs-get-visibility/map.png)


- **最近的案例**：查看最近的案例、其严重性及其关联的警报数。 如果特定类型的警报出现突发性的高峰，可能意味着某种攻击正在活跃地进行。 例如，如果 Azure ATP 中突然引发了多达 20 个传递哈希事件，可能意味着某人正在试图攻击你。

- **数据源异常**：Microsoft 的数据分析师创建了模型用于不间断地搜索数据源中数据的异常。 如果未出现任何异常，则不会显示任何信息。 如果检测到异常，则你应该进行深入调查，以确定发生了什么情况。 例如，单击“Azure 活动”中的高峰。 可以单击“图表”了解高峰是何时发生的，然后筛选在该时间段发生的活动，以确定哪些因素造成了高峰。

   ![Azure Sentinel 地图](./media/qs-get-visibility/anomolies.png)

## 使用内置仪表板<a name="dashboards"></a>

内置仪表板提供连接的数据源中的集成数据，让你深入调查这些服务中生成的事件。 内置仪表板包括 Azure ID、Azure 活动事件和本地信息，这些数据可能来自 Windows 服务器事件、第一方警报或任何第三方，其中包括防火墙流量日志、Office 365 和基于 Windows 的不安全协议。

1. 在“设置”下选择“仪表板”。 在“已安装”下，可以看到所有已安装的仪表板。 在“全部”下，可以看到可供安装的整个内置仪表板库。 
2. 搜索特定的仪表板以查看整个列表，以及每个仪表板的功能说明。 
3. 假设你使用 Azure AD，若要正常运行 Azure Sentinel，我们建议至少安装以下仪表板：
   - **Azure AD**：使用以下两项中的一个或两个：
       - “Azure AD 登录”可分析不同时间的登录活动，以确定是否存在异常。 此仪表板按应用程序、设备和位置列出失败的登录，使你能够即时注意到有异常情况发生。 请注意是否出现了多个失败的登录活动。 
       - “Azure AD 审核日志”可分析管理活动，例如用户更改（添加、删除等）、组创建和修改。  

   - 添加防火墙仪表板。 例如，添加 Palo Alto 仪表板。 仪表板可分析防火墙流量，在防火墙数据与威胁事件之间提供关联，并突出显示各个实体的可疑事件。 仪表板提供有关流量趋势的信息，并允许向下钻取和筛选结果。 

      ![Pal Alto 仪表板](./media/qs-get-visibility/palo-alto-week-query.png)


可以通过编辑主要查询 ![按钮](./media/qs-get-visibility/edit-query-button.png) 来自定义仪表板。 可以单击按钮 ![按钮](./media/qs-get-visibility/go-to-la-button.png) 转到 [Log Analytics 以编辑查询](../azure-monitor/log-query/get-started-portal.md)；可以选择省略号 (...) 并选择“自定义磁贴数据”，以编辑主要时间筛选器，或者从仪表板中删除特定的磁贴。

有关使用查询的详细信息，请参阅[教程：Log Analytics 中的视觉数据](../azure-monitor/learn/tutorial-logs-dashboards.md)

### <a name="add-a-new-tile"></a>添加新磁贴

若要添加新磁贴，可将其添加到现有仪表板 - 你创建的仪表板，或 Azure Sentinel 的内置仪表板。 
1. 在 Log Analytics 中，遵照以下教程中的说明创建磁贴：[教程：Log Analytics 中的视觉数据](../azure-monitor/learn/tutorial-logs-dashboards.md)。 
2. 创建磁贴后，在“固定”下，选择要在其中显示该磁贴的仪表板。

## <a name="create-new-dashboards"></a>创建新仪表板
可以从头开始创建新仪表板，或者基于某个内置仪表板创建新仪表板。

1. 若要从头开始创建新仪表板，请依次选择“仪表板”、“+新建仪表板”。
2. 选择要在其中创建该仪表板的订阅，并为其指定一个描述性的名称。 与其他任何元素一样，每个仪表板都是一个 Azure 资源，可为其分配角色 (RBAC) 以定义和限制哪些用户可以访问它。 
3. 若要使其显示在要将可视化效果固定到的仪表板中，必须将其共享。 依次单击“共享”、“管理用户”。 
 
1. 像设置其他任何 Azure 资源一样，使用“检查访问权限”和“角色分配”。 有关详细信息，请参阅[使用 RBAC 共享 Azure 仪表板](../azure-portal/azure-portal-dashboard-share-access.md)。


## <a name="new-dashboard-examples"></a>新仪表板示例

使用以下示例查询可以比较不同周次的流量趋势。 可以轻松切换要对其运行查询的设备供应商和数据源。 此示例使用来自 Windows 的 SecurityEvent。可将其切换为针对其他任何防火墙中的 AzureActivity 或 CommonSecurityLog 运行。

     |where DeviceVendor = = "Palo Alto Networks":
      // week over week query
      SecurityEvent
      | where TimeGenerated > ago(14d)
      | summarize count() by bin(TimeGenerated, 1d)
      | extend Week = iff(TimeGenerated>ago(7d), "This Week", "Last Week"), TimeGenerated = iff(TimeGenerated>ago(7d), TimeGenerated, TimeGenerated + 7d)


可以创建一个查询用于合并多个源中的数据。 可以创建一个查询，用于在 Azure Active Directory 审核日志中查找刚刚创建的新用户，然后检查 Azure 日志，以确定该用户在创建后的 24 小时内，是否开始进行角色分配更改。 该可疑活动会显示在此仪表板上：

    AuditLogs
    | where OperationName == "Add user"
    | project AddedTime = TimeGenerated, user = tostring(TargetResources[0].userPrincipalName)
    | join (AzureActivity
    | where OperationName == "Create role assignment"
    | project OperationName, RoleAssignmentTime = TimeGenerated, user = Caller) on user
    | project-away user1

可以基于用户的角色创建不同的仪表板，以查看该用户的数据并了解其正在查找哪些信息。 例如，可以针对网络管理员创建包含防火墙数据的仪表板。 此外，可以根据数据的查找频率创建仪表板，不管这些数据是每日都要查看的数据，还是每隔一小时检查一次的其他项（例如，你可能想要每隔一小时查看自己的 Azure AD 登录，以搜索异常）。 

## <a name="create-new-detections"></a>创建新的检测

在[连接到 Azure Sentinel 的数据源](connect-data-sources.md)上生成检测，以调查组织中的威胁。

创建新的检测时，请利用 Microsoft 安全研究人员为你连接的数据源量身定制的内置检测。

1. [在 GitHub 社区中](https://github.com/Azure/Azure-Sentinel/tree/master/Detections)，转到“检测”文件夹并选择相关文件夹。
   ![相关文件夹](./media/qs-get-visibility/detection-folders.png)
 
3.  转到“Analytics”选项卡并选择“添加”。
   ![在 Log Analytics 中创建规则](./media/qs-get-visibility/query-params.png)

3.  将所有参数复制到规则，然后单击“创建”。
   ![创建警报规则](./media/qs-get-visibility/create-alert-rule.png)

 
## <a name="next-steps"></a>后续步骤
本快速入门介绍了如何开始使用 Azure Sentinel。 请继续学习有关[如何检测威胁](tutorial-detect-threats.md)的教程。
> [!div class="nextstepaction"]
> [检测威胁](tutorial-detect-threats.md)并自动针对威胁做出响应。

