---
title: 停用的安全中心功能 (2019 年 7 月) |Microsoft Docs
description: 本文详细介绍将于 2019 年 7 月 31 日停用的安全中心中的功能。
services: security-center
author: yoavfrancis
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.date: 4/16/2019
ms.author: yoafr
ms.openlocfilehash: 614dabe842c7fe99da3ddb486c27003c79382ea1
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231392"
---
# <a name="retirement-of-security-center-features-july-2019"></a>停用的安全中心功能 (2019 年 7 月)

我们做了几个[改进](https://azure.microsoft.com/updates/?product=security-center)到过去 6 个月的 Azure 安全中心。  
使用改进的功能，我们将删除数冗余功能，以及相关的 Api，从安全中心于 2019 年 7 月 31 日。  

大部分已停用功能可以替换为 Azure 安全中心或 Log Analytics; 中的新增功能可以使用实现了一些功能，并[Azure Sentinel （预览版）](https://azure.microsoft.com/services/azure-sentinel/)、 最近推出。

安全中心从即将停用的功能列表包括：

- [事件仪表板](#menu_events)
- [搜索菜单项](#menu_search)
- [标识和访问 （预览版） 上查看经典标识和访问链接](#menu_classicidentity)
- [安全事件映射安全警报地图 （预览版） 上的按钮](#menu_securityeventsmap)
- [自定义警报规则 （预览版）](#menu_customalerts)
- [调查威胁保护安全警报中的按钮](#menu_investigate)
- [安全解决方案的一个子集](#menu_solutions)
- [编辑安全策略的安全配置](#menu_securityconfigurations)
- [安全和审核仪表板 （最初在 OMS 门户中使用） 的 Log Analytics 工作区。](#menu_securityomsdashboard)

下面提供了有关每个已停用的功能和步骤可能需要使用替代功能的详细的信息。

## 事件仪表板<a name="menu_events"></a>
安全中心使用 Microsoft Monitoring Agent 收集计算机中各种与安全有关的配置和事件，并将这些事件存储在工作区中。 [事件仪表板](https://docs.microsoft.com/azure/security-center/security-center-events-dashboard)允许您查看此数据，并实质上是向 Log Analytics 中提供另一个入口点。

今后事件仪表板将在停用：

![事件的工作区选择屏幕][1]

此外将停用用户在工作区，单击后，会显示事件仪表板：

![事件仪表板][2]

### <a name="events-dashboard---new-experience"></a>事件仪表板的新体验

建议使用 Log Analytics 的本机功能来查看其工作区上的值得注意的事件的客户。
如果你已从安全中心创建自定义值得注意的事件，这些可通过日志分析-> 选择工作区-> 保存的搜索。 你的数据不会丢失和修改。 本机值得注意的事件，还提供从同一屏幕。

![工作区保存的搜索][3]

## 搜索菜单项<a name="menu_search"></a>
目前，azure 安全中心使用 Azure Monitor 日志搜索来检索和分析安全数据。 此屏幕实质上是充当门面到 Log Analytics"[搜索](https://docs.microsoft.com/azure/security-center/security-center-search)"页-允许用户在其所选工作区上运行搜索查询。 今后此搜索窗口将在停用：

![搜索页][4]

### <a name="search-menu-entry---new-experience"></a>搜索菜单项的新体验

使用我们鼓励客户**Log Analytics**本机功能来执行他们的工作区中的搜索查询。 若要执行此操作，他们可以转到 Log analytics 在 Azure 中，并选择："日志":

![Log Analytics 日志页][5]

## 经典的标识和访问 （预览）<a name="menu_classicidentity"></a>
安全中心中的"经典"标识和访问体验提供客户能够查看其标识和访问 log analytics 中的相关的信息的方法。 按照下面单击执行此操作：

单击"查看经典标识和访问"

![标识页][6]

此时将打开此页，在屏幕以及"标识和访问仪表板":

![-工作区选择标识页][7]

单击工作区将打开"标识和访问"log analytics 仪表板，客户可以查看标识和访问他们的工作区上的信息：

![标识页-仪表板][8]

今后所有三个屏幕更高版本将在停用。 你的数据，仍可在 log analytics 安全解决方案并不会修改或删除。
用于填充标识和访问仪表板的原始的 Log Analytics 查询位于安全中心[GitHub 存储库](https://github.com/Azure/Azure-Security-Center/tree/master/Legacy%20Log%20Analytics%20dashboards)。

### <a name="classic-identity--access-preview---new-experience"></a>经典的标识和访问 （预览版）-新的体验
Log analytics 仪表板上给定工作区提供了见解，而本机的安全中心功能提供可见性的所有订阅以及与它们关联中一种简单的所有工作区-若要使用视图，让您专注的重要的是，根据安全标识和访问建议的分数。
可在安全中心中通过选择"标识和访问 （预览）"的标识和访问日志分析仪表板的所有功能：

![标识页-经典体验停用][9]

## 安全事件映射<a name="menu_securityeventsmap"></a>
安全中心提供了[映射](https://docs.microsoft.com/azure/security-center/security-center-threat-intel)，可帮助你根据环境识别安全威胁。 仪表板，从中可查看所选工作区上的原始安全事件会导致该映射中的转到安全事件映射按钮。
不推荐使用后，将删除的按钮，以及每个工作区的仪表板。

![安全警报图-按钮][10]

现在当你单击"转到安全事件映射到"威胁智能仪表板打开。 威胁智能仪表板将停用。  

![威胁智能仪表板][11]

当选择一个工作区以查看其威胁智能仪表板时，安全警报 map(Preview) 屏幕*Log Analytics 中*打开。 此屏幕将停用。

![在 Log Analytics 中的安全警报映射][12]

现有数据，仍可在 log analytics 安全解决方案并将不能修改也不删除。
用于填充威胁智能仪表板的原始的 Log Analytics 查询位于安全中心[GitHub 存储库](https://github.com/Azure/Azure-Security-Center/tree/master/Legacy%20Log%20Analytics%20dashboards)。

### <a name="security-events-map---new-experience"></a>安全事件映射的新体验
我们鼓励我们的客户能够使用内置到安全中心的警报映射功能-"安全警报映射 （预览版）"。 这提供了优化的体验并跨所有订阅和关联的工作区、 允许宏的工作在环境中，查看和不专注于单个工作区。

## 自定义警报规则 （预览版）<a name="menu_customalerts"></a>
自定义警报体验不会[停用](https://docs.microsoft.com/azure/security-center/security-center-custom-alert)2019 年 6 月 30 日，停用的底层基础结构的由于它基于。 在之前不推荐使用时间范围内，用户将能够编辑现有的自定义警报规则，但将不能添加新的。 建议用户将启用[Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/)与单击一次载入自动迁移其现有的警报并创建新的或或者重新创建其使用 Azure Monitor 警报的日志警报。

若要保留现有警报并将其迁移到 Azure Sentinel，请启动 Azure Sentinel。 第一步，选择自定义警报的存储位置工作区，然后选择分析菜单项将自动迁移你的警报。

![自定义警报][13]

我们鼓励重新创建其警报与 Azure Monitor 日志警报中载入到 Azure Sentinel 不感兴趣的客户。 有关说明，请参阅： [创建、 查看和管理使用 Azure Monitor 的日志警报](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log)。 有关如何创建日志警报的说明，请参阅：[Azure Monitor 中的日志警报](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-unified-log)。

有关自定义警报停用的详细信息，请参阅[自定义安全中心警报文档](https://docs.microsoft.com/azure/security-center/security-center-custom-alert)。

## 安全警报调查<a name="menu_investigate"></a>
[调查功能](https://docs.microsoft.com/azure/security-center/security-center-investigation)在安全中心，可会审，了解其范围并跟踪潜在的安全事件的根本原因。 此功能将删除从安全中心，因为它已替换中改进的体验[Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/)。

![安全事件][14]

当您单击上面的"调查"按钮时，将打开"调查中的仪表板 （预览版）"Log Analytics。 调查仪表板将停用。  
现有数据，仍可在 Log Analytics 的安全解决方案并将不能修改也不删除。

![在 Log Analytics 中的调查仪表板][15]

### <a name="investigation---new-experience"></a>调查的新体验

我们鼓励客户载入[Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/)提供丰富的调查体验，请提供支持查找功能的警报。 Azure Sentinel 提供了功能强大的搜索搜索和查询工具要在组织的数据源之间的安全威胁。  

## 安全解决方案的子集<a name="menu_solutions"></a>

安全中心已启用的功能[集成 Azure 中的安全解决方案](https://docs.microsoft.com/azure/security-center/security-center-partner-integration)。 将删除，并且支持在以下合作伙伴解决方案[Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/)，以及更多数据源。

- 下一代防火墙和 Web 应用程序防火墙解决方案 (Azure Sentinel[文档](https://docs.microsoft.com/azure/sentinel/connect-data-sources))
- 支持通用事件格式 (CEF) 的安全解决方案的集成 (Azure Sentinel[文档](https://docs.microsoft.com/azure/sentinel/connect-common-event-format))
- Microsoft 高级威胁分析 (Azure Sentinel[文档](https://docs.microsoft.com/azure/sentinel/connect-azure-atp))
- Azure AD Identity Protection (Azure Sentinel[文档](https://docs.microsoft.com/azure/sentinel/connect-azure-ad-identity-protection))

以下不推荐使用，用户将无法再以添加新的或修改前面提及的类型，从 UI 和 API 的现有已连接的解决方案。
现有的已连接的解决方案，建议将移动到 Azure Sentinel 此时间段结束。

![安全中心解决方案][16]

## 编辑安全策略的安全配置<a name="menu_securityconfigurations"></a>
Azure 安全中心应用 [150 多种建议的规则](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335)来监视安全配置。这些规则用于强化 OS，包括与防火墙、审核、密码策略等相关的规则。 如果发现计算机中的某项配置有漏洞，则安全中心会生成安全建议。 [编辑安全配置屏幕](https://docs.microsoft.com/azure/security-center/security-center-customize-os-security-config)允许客户自定义安全中心中的默认 OS 安全配置。

此功能已在预览版中，将停用。

![编辑安全配置][17]

### <a name="edit-security-configurations---new-experience"></a>编辑安全配置的新体验

安全中心将支持[来宾的配置代理](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration)在不久的将来，允许更丰富的功能集-包括支持其他操作系统和来宾配置使用的 Azure 策略 （与集成在来宾策略）。 这将提供在规模较大的控制和自动应用新的资源的能力。

## 安全和审核仪表板 （最初在 OMS 门户中使用） 的 Log Analytics 工作区<a name="menu_securityomsdashboard"></a>

在 Log analytics 中的安全仪表板提供了值得注意的安全事件和威胁、 威胁智能地图和保存工作区中的安全事件的标识和访问评估的每个工作区概述。 今后将删除仪表板。 根据我们已建议在仪表板 UI 中，我们建议用户使用 Azure 安全中心今后。

![日志分析安全仪表板][18]

### <a name="security--audit-dashboard---new-experience"></a>安全和审核仪表板的新体验
建议客户使用 Azure 安全中心，跨多个订阅和工作区关联，以及更丰富的功能集提供相同的安全性概述。

用于填充安全和审核仪表板的原始的 Log Analytics 查询位于安全中心[GitHub 存储库](https://github.com/Azure/Azure-Security-Center/tree/master/Legacy%20Log%20Analytics%20dashboards)。

## <a name="next-steps"></a>后续步骤
- 了解有关 [Azure 安全中心](https://docs.microsoft.com/azure/security-center/)的详细信息
- 详细了解[Azure Sentinel](https://docs.microsoft.com/azure/sentinel)

<!--Image references - events-->
[1]: ./media/security-center-features-retirement-july2019/asc_events_dashboard.png
[2]: ./media/security-center-features-retirement-july2019/asc_events_dashboard_inner.png
[3]: ./media/security-center-features-retirement-july2019/workspace_saved_searches.png
<!--Image references - search-->
[4]: ./media/security-center-features-retirement-july2019/asc_search.png
[5]: ./media/security-center-features-retirement-july2019/workspace_logs.png
<!--Image references - classic identity and access-->
[6]: ./media/security-center-features-retirement-july2019/asc_identity.png
[7]: ./media/security-center-features-retirement-july2019/asc_identity_workspace_selection.png
[8]: ./media/security-center-features-retirement-july2019/loganalytics_dashboard_identity.png
[9]: ./media/security-center-features-retirement-july2019/asc_identity_nobuttonhighlight.png
<!--Image references - alerts map-->
[10]: ./media/security-center-features-retirement-july2019/asc_security_alerts_map.png
[11]: ./media/security-center-features-retirement-july2019/asc_threat_intellignece_dashboard.png
[12]: ./media/security-center-features-retirement-july2019/loganalytics_security_alerts_map.png
<!--Image references - custom alerts-->
[13]: ./media/security-center-features-retirement-july2019/asc_custom_alerts.png
<!--Image references - Investigation-->
[14]: ./media/security-center-features-retirement-july2019/asc-security-incident.png
[15]: ./media/security-center-features-retirement-july2019/loganalytics_investigation_dashboard.png
<!--Image references - Solutions-->
[16]: ./media/security-center-features-retirement-july2019/asc_security_solutions.png
<!--Image references - Edit security configurations-->
[17]: ./media/security-center-features-retirement-july2019/asc_edit_security_configurations.png
<!--Image references - Security dashboard in log analytics-->
[18]: ./media/security-center-features-retirement-july2019/loganalytics_security_dashboard.png
