---
title: 分析 Azure Monitor 中的日志数据 | Microsoft Docs
description: 需要执行日志查询来检索 Azure Monitor 提供的日志数据。  本文介绍新的日志查询在 Azure Monitor 中的用法以及创建搜索之前需要了解的概念。
services: log-analytics
author: bwren
ms.service: log-analytics
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: bwren
ms.openlocfilehash: 1cb3946a93cbeff6a9b95e0a21edbf0523b53d5e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203602"
---
# <a name="analyze-log-data-in-azure-monitor"></a>在 Azure Monitor 中分析日志数据

Azure Monitor 收集的日志数据存储在 Log Analytics 工作区中，该工作区基于 [Azure 数据资源管理器](/azure/data-explorer)。 它从各种源收集遥测数据，并使用数据资源管理器所用的 [Kusto 查询语言](/azure/kusto/query)来检索和分析数据。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="log-queries"></a>日志查询

需要执行日志查询来检索 Azure Monitor 提供的任何日志数据。  无论是要[在门户中分析数据](portals.md)、[配置警报规则](../platform/alerts-metric.md)以在特定条件下发出通知，还是要使用 [Azure Monitor 日志 API](https://dev.loganalytics.io/) 检索数据，都需要使用查询来指定所需数据。  本文介绍了日志查询在 Azure Monitor 中的用法以及创建查询之前应了解的概念。

## <a name="where-log-queries-are-used"></a>在何处使用日志查询

在 Azure Monitor 中使用查询的方式包括以下几种：

- **门户。** 可以在 [Azure 门户](portals.md)中对日志数据执行交互式分析。  这样便可采用各种格式和可视化效果编辑查询并分析结果。  
- **警报规则。** [警报规则](../platform/alerts-overview.md)主动识别工作区中数据的问题。  每个警报规则均基于定期自动运行的日志搜索。  对结果进行检查，确定是否应创建警报。
- **仪表板。** 可以将任何查询的结果固定到 [Azure 仪表板](../learn/tutorial-logs-dashboards.md)，这使得你可以将日志和指标数据一起可视化，并且还可以将其与其他 Azure 用户共享。 
- **视图。**  可以使用[视图设计器](../platform/view-designer.md)创建要包含在用户仪表板中的数据的可视化效果。  日志查询提供每个视图中[磁贴](../platform/view-designer-tiles.md)和[可视化部件](../platform/view-designer-parts.md)使用的数据。  

- **导出。**  将日志数据从 Azure Monitor 导入到 Excel 或 [Power BI](../platform/powerbi.md) 中时，请创建日志查询来定义要导出的数据。
- **PowerShell。** 可从命令行运行 PowerShell 脚本或运行使用 [Get-AzOperationalInsightsSearchResults](/powershell/module/az.operationalinsights/get-azoperationalinsightssearchresult) 的 Azure 自动化 runbook，从 Azure Monitor 中检索日志数据。  此 cmdlet 需要一个查询来确定要检索的数据。
- **Azure Monitor 日志 API。**  [Azure Monitor 日志 API](../platform/alerts-overview.md) 允许任何 REST API 客户端从工作区中检索日志数据。  API 请求包括针对 Azure Monitor 运行的查询，用于确定要检索的数据。

![日志搜索](media/log-query-overview/queries-overview.png)

## <a name="write-a-query"></a>编写查询
Azure Monitor 使用某个[查询语言版本](get-started-queries.md)以各种方式检索和分析日志数据。  通常先使用基本查询，随着要求变得复杂，逐渐开始使用更高级的功能。

查询的基本结构是一个源表，后跟一系列以竖线字符 `|` 隔开的运算符。  可以将多个运算符链接起来以优化数据和执行高级函数。

例如，假设要查找过去一天中错误事件最多的前 10 台计算机。

```Kusto
Event
| where (EventLevelName == "Error")
| where (TimeGenerated > ago(1days))
| summarize ErrorCount = count() by Computer
| top 10 by ErrorCount desc
```

或者，要查找在最后一天中没有检测信号的计算机。

```Kusto
Heartbeat
| where TimeGenerated > ago(7d)
| summarize max(TimeGenerated) by Computer
| where max_TimeGenerated < ago(1d)  
```

来一个折线图如何？显示过去一周中每台计算机的处理器利用率

```Kusto
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where TimeGenerated  between (startofweek(ago(7d)) .. endofweek(ago(7d)) )
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5min)
| render timechart    
```

从这些快速示例可以看出，无论处理何种数据，查询的结构是类似的。  可以将其分解为不同的步骤，其中从一条命令生成的数据会通过管道发送到下一条命令。

还可在订阅中跨 Log Analytics 工作区查询数据。

```Kusto
union Update, workspace("contoso-workspace").Update
| where TimeGenerated >= ago(1h)
| summarize dcount(Computer) by Classification 
```

## <a name="how-azure-monitor-log-data-is-organized"></a>如何组织 Azure Monitor 日志数据
生成查询时，首先确定哪些表含要查找的数据。 各种类型的数据被划分到每个 [Log Analytics 工作区](../learn/quick-create-workspace.md)中的专用表中。  各种数据源的文档包括其创建的数据类型的名称及其每个属性的说明。  许多查询只需要单个表中的数据，但其他查询可能会使用多种选项以包含多个表中的数据。

虽然 [Application Insights](../app/app-insights-overview.md) 将诸如请求、异常、跟踪和使用情况之类的应用程序数据存储在 Azure Monitor 日志中，但此数据存储在与其他日志数据不同的分区中。 你可以使用相同的查询语言来访问此数据，但必须使用 [Application Insights 控制台](../app/analytics.md)或 [Application Insights REST API](https://dev.applicationinsights.io/) 来访问它。 可以使用[跨资源查询](../log-query/cross-workspace-query.md)将 Application Insights 数据与 Azure Monitor 中的其他日志数据配合使用。


![表](media/log-query-overview/queries-tables.png)




## <a name="next-steps"></a>后续步骤
- 了解如何使用[Log Analytics，以创建和编辑日志搜索](../log-query/portals.md)。
- 查看使用新查询语言的[查询编写教程](../log-query/get-started-queries.md)。
