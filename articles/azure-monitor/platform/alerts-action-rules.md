---
title: Azure 监视器将发出警报的操作规则
description: 了解操作规则是什么，以及如何配置和管理它们。
author: anantr
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: anantr
ms.component: alerts
ms.openlocfilehash: d27adadc9720dd2ad6a0dd133524bfaf32e63045
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65227993"
---
# <a name="action-rules-preview"></a>操作规则 （预览版）

本文介绍了操作规则是什么，以及如何配置和管理它们。

## <a name="what-are-action-rules"></a>操作规则是什么？

操作规则，可在任何资源管理器作用域 （订阅、 资源组或资源） 中定义的操作 （或禁止显示的操作）。 它们具有多种筛选器，允许您缩小到需要采取措施的警报实例的特定子集。 

使用操作规则可以：

* 如果已进行了规划维护时段或周末/假日安排，而无需禁用每个警报规则单独，取消操作和通知。
* 定义操作和大规模的通知：无需定义单独为每个警报规则的操作组，现在可以定义要在任何范围内生成的警报触发的操作组。 例如，我可以选择允许生成我的订阅中每个警报的操作组 ContosoActionGroup 触发器。

## <a name="configuring-an-action-rule"></a>操作规则配置

可以通过选择访问功能**管理操作**从登录页上，在 Azure Monitor 中的警报。 然后选择**操作规则 （预览版）**。 可以通过选择访问它们**操作规则 （预览版）** 警报的登陆页面的仪表板。

![从 Azure 监视器登陆页面的操作规则](media/alerts-action-rules/action-rules-landing-page.png)

选择 **+ 新操作规则**。 

![添加新操作规则](media/alerts-action-rules/action-rules-new-rule.png)

或者，您还可以选择配置警报规则时创建操作规则。

![添加新操作规则](media/alerts-action-rules/action-rules-alert-rule.png)

现在应看到打开操作规则创建流。 配置以下元素： 

![新操作规则创建流](media/alerts-action-rules/action-rules-new-rule-creation-flow.png)

### <a name="scope"></a>范围

首先选择范围，也就是说，目标资源、 资源组或订阅。 您还可以选择多个以上的 （在单个订阅） 的任意组合。 

![操作规则作用域](media/alerts-action-rules/action-rules-new-rule-creation-flow-scope.png)

### <a name="filter-criteria"></a>筛选条件

此外，您可以进一步缩小到的警报定义的作用域上的特定子集定义筛选器。 

可用的筛选器是： 

* **严重性**：选择一个或多个警报的严重级别。 严重性 = Sev1 意味着操作规则适用于具有 Sev1 严重性的所有警报。
* **监视服务**:基于原始监视服务筛选器。 这也是多选。 例如，监视服务 ="Application Insights"意味着操作规则适用于所有"Application Insights"基于警报。
* **资源类型**:基于特定资源类型的筛选器。 这也是多选。 例如，资源类型 ="虚拟机"表示操作规则是适用于所有虚拟机。
* **警报规则 ID**:允许您筛选有关使用警报规则的资源管理器 ID 的特定警报规则。
* **监视条件**:与"已触发"或"已解决"作为监视器条件的警报实例的筛选器。
* **说明**：正则表达式匹配中定义为警报规则的一部分的说明。
* **警报上下文 （有效负载）**:正则表达式中匹配[警报上下文](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-common-schema-definitions#alert-context-fields)警报实例的字段。

两台一起应用这些筛选器。 例如，如果我设置资源类型 = 虚拟机和严重性 = Sev0，则我在我的 Vm 上已筛选的所有 Sev0 警报。 

![操作规则筛选器](media/alerts-action-rules/action-rules-new-rule-creation-flow-filters.png)

### <a name="suppression-or-action-group-configuration"></a>禁止显示或操作组配置

接下来配置警报抑制或操作组支持的操作规则。 您不能同时选择。 配置作用于以前定义的范围和筛选器匹配的所有警报实例。

#### <a name="suppression"></a>禁止显示

如果选择**抑制**，配置的操作和通知的禁止显示的持续时间。 选择以下项之一：
* **从现在起 （始终）**:无限期地禁止显示所有通知。
* **在计划时间**:有限持续时间内取消通知。
* **并且指定了 recurrence**:按定期计划每日、 每周或每月可禁止显示。

![操作规则禁止显示](media/alerts-action-rules/action-rules-new-rule-creation-flow-suppression.png)

#### <a name="action-group"></a>操作组

如果选择**操作组**的切换按钮中添加现有操作组或新建一个。 

> [!NOTE]
> 可以将只有一个操作组和操作规则相关联。

![操作规则操作组](media/alerts-action-rules/action-rules-new-rule-creation-flow-action-group.png)

### <a name="action-rule-details"></a>操作规则的详细信息

最后，配置操作规则的以下详细信息
* 名称
* 在其中保存的资源组
* 描述 

## <a name="example-scenarios"></a>示例方案

### <a name="scenario-1-suppression-of-alerts-based-on-severity"></a>方案 1：基于严重性的警报中禁止显示

Contoso 想要取消其订阅 ContosoSub 每个周末都中的所有 Vm 上的所有 Sev4 警报的通知。

**解决方案：** 创建与操作规则
* Scope = 'ContosoSub'
* 筛选器
    * 严重性 = Sev4
    * 资源类型 = 虚拟机
* 禁止显示有重复周期设置为每周，并选中星期六和星期日

### <a name="scenario-2-suppression-of-alerts-based-on-alert-context-payload"></a>方案 2：禁止显示的警报基于警报上下文 （有效负载）

Contoso 希望为所有日志生成的"计算机-01"警报抑制通知中 ContosoSub 无限期地因为它会维护。

**解决方案：** 创建与操作规则
* Scope = 'ContosoSub'
* 筛选器
    * Monitor Service = 'Log Analytics'
    * 警报上下文 （有效负载） 包含"计算机-01"
* 禁止显示设置为从现在起 （始终）

### <a name="scenario-3-action-group-defined-at-a-resource-group"></a>方案 3：在资源组中定义的操作组

具有定义的 Contoso[指标警报在订阅级别](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-metric-overview#monitoring-at-scale-using-metric-alerts-in-azure-monitor)，但想要定义为其资源组 ContosoRG 单独的警报触发的操作。

**解决方案：** 创建与操作规则
* Scope = 'ContosoRG'
* 没有筛选器
* 设置为 ContosoActionGroup 的操作组

## <a name="managing-your-action-rules"></a>管理操作规则

可以查看和管理操作规则，从列表视图，如下所示。

![操作规则列表视图](media/alerts-action-rules/action-rules-list-view.png)

在这里，你可以启用/禁用/删除操作规则在规模较大，通过选择其旁边的复选框。 单击任何操作规则会打开其配置页中，从而可以更新其定义和启用/禁用它。

## <a name="best-practices"></a>最佳做法

记录与创建的警报[结果数](alerts-unified-log.md)选项生成**单个警报实例**使用整个搜索结果 （这可能会在多台计算机为例）。 在此方案中，如果操作规则使用警报上下文 （有效负载） 筛选器，它将作用于警报实例，只要没有匹配项。 在方案 2 中前面所述，如果生成的日志警报的搜索结果包含"计算机-01"和"计算机-02"整个通知被取消 （即，处于不为"计算机-02"在所有生成显示通知）。

![操作规则和日志警报 （结果数）](media/alerts-action-rules/action-rules-log-alert-number-of-results.png)

到最佳利用的日志警报使用操作规则，我们建议你创建的日志警报[指标度量](alerts-unified-log.md)选项。 使用此选项，根据定义的组字段生成单独的警报实例。 然后在方案 2 中，单独的警报实例生成的"计算机-01"和"计算机-02"。 使用方案中所述操作规则时，"计算机-02"的通知将继续照常触发时，将抑制仅对于"计算机-01"通知。

![操作规则和日志警报 （结果数）](media/alerts-action-rules/action-rules-log-alert-metric-measurement.png)

## <a name="faq"></a>常见问题解答

* 问： 在配置操作规则时，我想要查看所有可能的重叠操作都规则，以便避免重复的通知。 是可以这样做？

    A. 一旦配置操作规则时定义作用域，可以看到操作规则重叠在同一作用域上 （如果有） 的列表。 这种重叠可以是下列选项之一：
    * 完全匹配：例如，要定义的操作规则和重叠操作规则是在同一订阅。
    * 子集：例如，要定义的操作规则是某一订阅和重叠操作规则是在订阅中的资源组。
    * 超集：例如，要定义的操作规则是上一个资源组，和重叠操作规则位于包含资源组的订阅。
    * 交集：例如，要定义的操作规则处于 VM1 和 VM2，重叠操作规则处于 VM2 和 VM3。

    ![重叠操作规则](media/alerts-action-rules/action-rules-overlapping.png)

* 问： 时配置警报规则，它可以知道有没有已定义的操作规则可能会对我定义的警报规则？

    A. 一旦你的警报规则定义的目标资源，可以看到操作规则执行操作的相同作用域 （如果有） 上通过单击配置视图的操作的操作部分下的列表。 基于范围的以下应用场景填充此列表：
    * 完全匹配：例如，要定义警报规则和操作规则是在同一订阅。
    * 子集：例如，某一订阅是您定义的警报规则和操作规则是在订阅中的资源组。
    * 超集：例如上一个资源组，, 是您定义的警报规则和操作规则位于包含资源组的订阅。
    * 交集：例如，要定义警报规则启用 VM1 和 VM2，并且操作规则位于 VM2 和 VM3。
    
    ![重叠操作规则](media/alerts-action-rules/action-rules-alert-rule-overlapping.png)

* 问： 可以看到已取消操作规则的警报？

    A. 在中[警报列表页](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-managing-alert-instances)，还有一个额外的列，可以选择名为禁止显示状态。 如果一个警报实例的通知被禁止，它会在列表中显示该状态。

    ![禁止显示警报实例](media/alerts-action-rules/action-rules-suppressed-alerts.png)

* 问： 如果使用操作组的操作规则，并禁止显示与另一个活动上的同一作用域，会发生什么情况？

    A. **禁止显示始终将优先于相同作用域**。

* 问： 如果我在两个单独的操作规则监视的资源，会发生什么情况？ 获取一个或两个通知？ 例如，VM2 在此方案中：

      action rule 'AR1' defined for 'VM1' and 'VM2' with action group 'AG1'
      action rule 'AR2' defined for 'VM2' and 'VM3' with action group 'AG1'

    A. 对于 VM1 和 VM3 上的每个警报，将一次触发操作组 AG1。 对于 VM2 上的每个警报，操作组 AG1 会触发两次 (**操作规则不消除重复操作**)。 

* 问： 如果具有在两个单独的操作规则监视的资源，其中一个调用的操作，而另一个用于抑制，会发生什么情况？ 例如，VM2 在此方案中：

      action rule 'AR1' defined for 'VM1' and 'VM2' with action group 'AG1' 
      action rule 'AR2' defined for 'VM2' and 'VM3' with suppression

    A. 对于 VM1 上的每个警报，将一次触发操作组 AG1。 操作和 VM2 和 VM3 上的每个警报的通知将被抑制。 

* 问： 如果我的警报规则，并为相同资源调用不同的操作组定义的操作规则，会发生什么情况？ 例如，在此方案中的 VM1:

      alert rule  'rule1' on          'VM1' with action group 'AG2'
      action rule 'AR1'   defined for 'VM1' with action group 'AG1' 
 
    A. 对于 VM1 上的每个警报，将一次触发操作组 AG1。 每当触发警报规则 rule1 时，它还将另外触发 AG2。 **操作规则中定义的操作组和警报规则操作独立，与任何重复数据消除**。 

## <a name="next-steps"></a>后续步骤

- [了解有关 Azure 中的警报的详细信息](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview)
