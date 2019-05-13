---
title: 将 Azure 自动化作业数据转发到 Azure Monitor 日志
description: 本文演示如何将作业状态和 runbook 作业流发送到 Azure Monitor 日志，以提供附加见解和管理。
services: automation
ms.service: automation
ms.subservice: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 02/05/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8672bc28ea5e8562472408810a38ea0de6778cfd
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200619"
---
# <a name="forward-job-status-and-job-streams-from-automation-to-azure-monitor-logs"></a>将作业状态和作业流从自动化转发到 Azure Monitor 日志

自动化可以将 Runbook 作业状态和作业流发送到 Log Analytics 工作区。 此过程不涉及工作区链接，并且完全独立。 可在 Azure 门户中或使用 PowerShell 查看单个作业的作业日志和作业流，这使用户可执行简单的调查。 现在，可以使用 Azure Monitor 日志执行以下操作：

* 获取有关自动化作业的见解。
* 基于 Runbook 作业状态（例如失败或暂停）触发电子邮件或警报。
* 编写跨作业流的高级查询。
* 跨自动化帐户关联作业。
* 可视化不同时间段的作业历史记录。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites-and-deployment-considerations"></a>先决条件和部署注意事项

若要开始将自动化日志发送到 Azure Monitor 日志，需要：

* 最新版本的[Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/)。
* Log Analytics 工作区。 有关详细信息，请参阅 [Azure Monitor 日志入门](../log-analytics/log-analytics-get-started.md)。 
* Azure 自动化帐户的 ResourceId。

若要查找 Azure 自动化帐户的 ResourceId，请执行以下操作：

```powershell-interactive
# Find the ResourceId for the Automation Account
Get-AzResource -ResourceType "Microsoft.Automation/automationAccounts"
```

要查找 Log Analytics 工作区的 ResourceId，请运行以下 PowerShell：

```powershell-interactive
# Find the ResourceId for the Log Analytics workspace
Get-AzResource -ResourceType "Microsoft.OperationalInsights/workspaces"
```

如果有多个自动化帐户或工作区，请在上述命令的输出中找到需要配置的*名称*，并复制 *ResourceId* 的值。

要查找自动化帐户的名称，请在 Azure 门户中，从“自动化帐户”边栏选项卡选择自动化帐户，并选择“所有设置”。 在“所有设置”边栏选项卡中，选择“帐户设置”下面的“属性”。  在“属性”边栏选项卡中，可以记下这些值。<br> ![自动化帐户属性](media/automation-manage-send-joblogs-log-analytics/automation-account-properties.png)。

## <a name="set-up-integration-with-azure-monitor-logs"></a>设置与 Azure Monitor 日志的集成

1. 在计算机上，从“开始”屏幕启动 **Windows PowerShell**。
2. 运行以下 PowerShell，并使用前面步骤中获得的值编辑 `[your resource id]` 和 `[resource id of the log analytics workspace]` 的值。

   ```powershell-interactive
   $workspaceId = "[resource id of the log analytics workspace]"
   $automationAccountId = "[resource id of your automation account]"

   Set-AzDiagnosticSetting -ResourceId $automationAccountId -WorkspaceId $workspaceId -Enabled 1
   ```

运行此脚本后，可能需要一小时才能开始在 Azure Monitor 日志中看到写入新 JobLogs 或 JobStreams 的记录。

若要查看日志，请在 Log Analytics 日志搜索中运行以下查询：`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION"`

### <a name="verify-configuration"></a>验证配置

要确认自动化帐户是否会将日志发送到 Log Analytics 工作区，请使用以下 PowerShell 检查是否在自动化帐户上正确配置了诊断：

```powershell-interactive
Get-AzDiagnosticSetting -ResourceId $automationAccountId
```

在输出中确保：

* 在 *Logs* 下，*Enabled* 的值为 *True*。
* *WorkspaceId* 的值设置为 Log Analytics 工作区的 ResourceId。

## <a name="azure-monitor-log-records"></a>Azure Monitor 日志记录

来自 Azure 自动化的诊断将在 Azure Monitor 日志中创建两种类型的记录，并将其标记为 **AzureDiagnostics**。 以下查询使用升级的 Azure Monitor 日志查询语言。 有关旧查询语言与新 Azure Kusto 查询语言之间的共有查询的相关信息，请访问[新旧 Azure Kusto 查询语言速查表](https://docs.loganalytics.io/docs/Learn/References/Legacy-to-new-to-Azure-Log-Analytics-Language)。

### <a name="job-logs"></a>作业日志

| 属性 | 描述 |
| --- | --- |
| TimeGenerated |执行 Runbook 作业的日期和时间。 |
| RunbookName_s |Runbook 的名称。 |
| Caller_s |谁启动了该操作。 可能的值为电子邮件地址或计划作业的系统。 |
| Tenant_g | GUID，用于为 Caller 标识租户。 |
| JobId_g |用作 Runbook 作业 ID 的 GUID。 |
| ResultType |Runbook 作业的状态。 可能的值包括：<br>- New（新）<br>- Started（已启动）<br>- Stopped（已停止）<br>- Suspended（已暂停）<br>- Failed（失败）<br>- Completed（已完成） |
| 类别 | 数据类型的分类。 对于自动化，该值为 JobLogs。 |
| OperationName | 指定在 Azure 中执行的操作类型。 对于自动化，该值为 Job。 |
| 资源 | 自动化帐户的名称 |
| SourceSystem | Azure Monitor 日志收集数据的方式。 对于 Azure 诊断，始终为 Azure。 |
| ResultDescription |描述 Runbook 作业结果状态。 可能的值包括：<br>- 作业已启动<br>- 作业失败<br>- 作业已完成 |
| CorrelationId |用作 Runbook 作业相关性 ID 的 GUID。 |
| ResourceId |指定 Runbook 的 Azure 自动化帐户资源 ID。 |
| SubscriptionId | 自动化帐户的 Azure 订阅 ID (GUID)。 |
| resourceGroup | 自动化帐户的资源组的名称。 |
| ResourceProvider | MICROSOFT.AUTOMATION |
| ResourceType | AUTOMATIONACCOUNTS |


### <a name="job-streams"></a>作业流
| 属性 | 描述 |
| --- | --- |
| TimeGenerated |执行 Runbook 作业的日期和时间。 |
| RunbookName_s |Runbook 的名称。 |
| Caller_s |谁启动了该操作。 可能的值为电子邮件地址或计划作业的系统。 |
| StreamType_s |作业流的类型。 可能的值包括：<br>- Progress（进度）<br>- Output（输出）<br>- Warning（警告）<br>- Error（错误）<br>- Debug（调试）<br>- Verbose（详细） |
| Tenant_g | GUID，用于为 Caller 标识租户。 |
| JobId_g |用作 Runbook 作业 ID 的 GUID。 |
| ResultType |Runbook 作业的状态。 可能的值包括：<br>- In Progress |
| 类别 | 数据类型的分类。 对于自动化，该值为 JobStreams。 |
| OperationName | 指定在 Azure 中执行的操作类型。 对于自动化，该值为 Job。 |
| 资源 | 自动化帐户的名称 |
| SourceSystem | Azure Monitor 日志收集数据的方式。 对于 Azure 诊断，始终为 Azure。 |
| ResultDescription |包括来自 Runbook 的输出流。 |
| CorrelationId |用作 Runbook 作业相关性 ID 的 GUID。 |
| ResourceId |指定 Runbook 的 Azure 自动化帐户资源 ID。 |
| SubscriptionId | 自动化帐户的 Azure 订阅 ID (GUID)。 |
| resourceGroup | 自动化帐户的资源组的名称。 |
| ResourceProvider | MICROSOFT.AUTOMATION |
| ResourceType | AUTOMATIONACCOUNTS |

## <a name="viewing-automation-logs-in-azure-monitor-logs"></a>在 Azure Monitor 日志中查看自动化日志

开始将自动化作业日志发送到 Azure Monitor 日志后，让我们看一下在 Azure Monitor 日志中可对这些日志执行哪些操作。

若要查看日志，请运行以下查询：`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION"`

### <a name="send-an-email-when-a-runbook-job-fails-or-suspends"></a>Runbook 作业失败或暂停时发送电子邮件
客户的主要诉求之一是，当 Runbook 作业出现问题时能够发送电子邮件或短信。   

若要创建警报规则，首先请针对应该调用警报的 Runbook 作业记录创建日志搜索。 单击“警报”按钮以创建和配置警报的规则。

1. 在“Log Analytics 工作区概述”页中，单击“查看日志”。
2. 在查询字段中键入以下搜索，针对警报创建日志搜索查询：`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended")`；也可以使用以下命令按 RunbookName 分组：`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended") | summarize AggregatedValue = count() by RunbookName_s`

   如果设置了在工作区中收集来自多个自动化帐户或订阅的日志，则可以按照订阅或自动化帐户来为警报分组。 可以在 JobLogs 搜索中的“资源”字段中找到自动化帐户名称。
3. 若要打开“创建规则”屏幕，请单击页面顶部的“+ 新建警报规则”。 有关用于配置警报的选项的详细信息，请参阅 [Azure 中的日志警报](../azure-monitor/platform/alerts-unified-log.md)。

### <a name="find-all-jobs-that-have-completed-with-errors"></a>查找已完成但出错的所有作业
除了在失败时发出警报外，还可以发现 Runbook 作业何时发生非终止错误。 在这些情况下，PowerShell 会生成一个错误流，但非终止错误不会导致作业暂停或失败。    

1. 在 Log Analytics 工作区中单击“日志”。
2. 在“查询”字段中键入 `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error" | summarize AggregatedValue = count() by JobId_g`，然后单击“搜索”。

### <a name="view-job-streams-for-a-job"></a>查看作业的作业流
调试作业时，你可能还希望深入查看作业流。 以下查询会显示 GUID 为 2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0 的单个作业的所有流：   

`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and JobId_g == "2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0" | sort by TimeGenerated asc | project ResultDescription`

### <a name="view-historical-job-status"></a>查看历史作业状态
最后，可能需要随时间对作业历史记录进行可视化。 可以使用此查询来搜索作业在不同时间段的状态。

`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and ResultType != "started" | summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h)`  
<br> ![Log Analytics 历史作业状态图标](media/automation-manage-send-joblogs-log-analytics/historical-job-status-chart.png)<br>

## <a name="remove-diagnostic-settings"></a>删除诊断设置

若要从自动化帐户中删除诊断设置，请运行以下命令：

```powershell-interactive
$automationAccountId = "[resource id of your automation account]"

Remove-AzDiagnosticSetting -ResourceId $automationAccountId
```

## <a name="summary"></a>摘要

将自动化作业状态和流数据发送到 Azure Monitor 日志后，可执行以下操作来更好地洞察自动化作业的状态：
+ 设置警报，以便在出现问题时获得通知。
+ 使用自定义视图和搜索查询直观地显示 Runbook 结果、Runbook 作业状态，以及其他相关的关键指标。  

Azure Monitor 日志可以更直观地显示自动化作业的运行情况，并且可以帮助更快地解决事件。  

## <a name="next-steps"></a>后续步骤
* 若要详细了解如何使用 Azure Monitor 日志构造不同的搜索查询和查看自动化作业日志，请参阅 [Azure Monitor 日志中的日志搜索](../log-analytics/log-analytics-log-searches.md)。
* 若要了解如何通过 Runbook 创建和检索输出及错误消息，请参阅 [Runbook 输出和消息](automation-runbook-output-and-messages.md)。
* 若要详细了解 Runbook 执行方式、如何监视 Runbook 作业和其他技术详细信息，请参阅[跟踪 Runbook 作业](automation-runbook-execution.md)。
* 若要了解有关 Azure Monitor 日志和数据收集源的详细信息，请参阅[在 Azure Monitor 日志中收集 Azure 存储数据概述](../azure-monitor/platform/collect-azure-metrics-logs.md)。

