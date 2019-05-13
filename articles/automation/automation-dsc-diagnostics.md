---
title: 将 Azure Automation State Configuration 报表数据转发到 Azure Monitor 日志
description: 本文演示如何将 Desired State Configuration (DSC) 报表数据从 Azure Automation State Configuration 发送到 Azure Monitor 日志，以便为用户提供附加见解和管理信息。
services: automation
ms.service: automation
ms.subservice: dsc
author: bobbytreed
ms.author: robreed
ms.date: 11/06/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 0dad74f75fd7b73e7dab0b2dddbdfda193d5b2ec
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61073927"
---
# <a name="forward-azure-automation-state-configuration-reporting-data-to-azure-monitor-logs"></a>将 Azure Automation State Configuration 报表数据转发到 Azure Monitor 日志

Azure Automation State Configuration 会将节点状态数据保留 30 天。
如果希望节点状态数据能够保留更长的时间，则可将其发送到 Log Analytics 工作区。
节点和节点配置中的单个 DSC 资源的符合性状态可以通过 Azure 门户或 PowerShell 查看。
可以使用 Azure Monitor 日志进行以下操作：

- 获取托管节点和单个资源的符合性信息
- 基于符合性状态触发电子邮件或警报
- 跨托管节点编写高级查询
- 跨自动化帐户关联符合性状态
- 随着时间的推移，可视化节点符合性历史记录

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>必备组件

若要开始将 Automation State Configuration 报表发送到 Azure Monitor 日志，需要准备：

- 2016 年 11 月或之后发布的 [Azure PowerShell](/powershell/azure/overview) (v2.3.0) 版本。
- 一个 Azure 自动化帐户。 有关详细信息，请参阅 [Azure 自动化入门](automation-offering-get-started.md)
- 具有“自动化和控制”服务产品的 Log Analytics 工作区。 有关详细信息，请参阅 [Azure Monitor 日志入门](../log-analytics/log-analytics-get-started.md)。
- 至少一个 Azure Automation State Configuration 节点。 有关详细信息，请参阅[登记由 Azure Automation State Configuration 管理的计算机](automation-dsc-onboarding.md)

## <a name="set-up-integration-with-azure-monitor-logs"></a>设置与 Azure Monitor 日志的集成

若要开始将数据从 Azure Automation DSC 导入到 Azure Monitor 日志，请完成以下步骤：

1. 通过 PowerShell 登录 Azure 帐户。 请参阅[使用 Azure PowerShell 登录](https://docs.microsoft.com/powershell/azure/authenticate-azureps)
1. 通过运行以下 PowerShell 命令获取自动化帐户的 ResourceId：（如果具有多个自动化帐户，选择想要配置的帐户的 ResourceID）。

   ```powershell
   # Find the ResourceId for the Automation Account
   Get-AzureRmResource -ResourceType 'Microsoft.Automation/automationAccounts'
   ```

1. 通过运行以下 PowerShell 命令获取 Log Analytics 工作区的 ResourceId：（如果具有多个工作区，选择想要配置的工作区的 ResourceID）。

   ```powershell
   # Find the ResourceId for the Log Analytics workspace
   Get-AzureRmResource -ResourceType 'Microsoft.OperationalInsights/workspaces'
   ```

1. 运行以下 PowerShell 命令，将 `<AutomationResourceId>` 和 `<WorkspaceResourceId>` 替换为前面每个步骤中的 ResourceId 值：

   ```powershell
   Set-AzureRmDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $true -Categories 'DscNodeStatus'
   ```

若要停止将数据从 Azure Automation State Configuration 导入到 Azure Monitor 日志，请运行以下 PowerShell 命令：

```powershell
Set-AzureRmDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $false -Categories 'DscNodeStatus'
```

## <a name="view-the-state-configuration-logs"></a>查看 Automation State Configuration 日志

为 Automation State Configuration 数据设置与 Azure Monitor 日志的集成后，“日志搜索”按钮会出现在自动化帐户的“DSC 节点”边栏选项卡上。 单击“日志搜索”按钮，查看 DSC 节点数据的日志。

![日志搜索按钮](media/automation-dsc-diagnostics/log-search-button.png)

“日志搜索”边栏选项卡将打开，并且你会看到针对每个 State Configuration 节点的“DscNodeStatusData”操作，以及针对在应用于该节点的节点配置中调用的每个 [DSC 资源](/powershell/dsc/resources)的“DscResourceStatusData”操作。

“DscResourceStatusData”操作包含针对失败的任何 DSC 资源的错误信息。

单击列表中的每个操作可查看该操作的数据。

还可以通过在 Azure Monitor 日志中进行搜索来查看日志。
请参阅[使用日志搜索查找数据](../log-analytics/log-analytics-log-searches.md)。
键入以下查询以查找 State Configuration 日志：`Type=AzureDiagnostics ResourceProvider='MICROSOFT.AUTOMATION' Category='DscNodeStatus'`

还可以通过操作名称缩小查询范围。 例如： `Type=AzureDiagnostics ResourceProvider='MICROSOFT.AUTOMATION' Category='DscNodeStatus' OperationName='DscNodeStatusData'`

### <a name="send-an-email-when-a-state-configuration-compliance-check-fails"></a>State Configuration 符合性检查失败时发送一封电子邮件

我们的一家重要客户提出的请求是，当 DSC 配置出现问题时能够发送电子邮件或短信。

若要创建预警规则，需要首先针对应调用警报的 State Configuration 报表记录创建日志搜索。 单击“+ 新建警报规则”按钮以创建并配置警报规则。

1. 在“Log Analytics 工作区概述”页中，单击“日志”。
1. 在查询字段中键入以下搜索，针对警报创建日志搜索查询：`Type=AzureDiagnostics Category='DscNodeStatus' NodeName_s='DSCTEST1' OperationName='DscNodeStatusData' ResultType='Failed'`

   如果已设置在工作区中收集来自多个自动化帐户或订阅的日志，则可以按照订阅或自动化帐户来为警报分组。  
   自动化帐户名称可能派生自 DscNodeStatusData 搜索中的 Resource 字段。  
1. 若要打开“创建规则”屏幕，请单击页面顶部的“+ 新建警报规则”。 有关警报配置选项的详细信息，请参阅[创建警报规则](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)。

### <a name="find-failed-dsc-resources-across-all-nodes"></a>在所有节点中查找失败的 DSC 资源

使用 Azure Monitor 日志的一个优点是，可以在节点中搜索失败的检查。
若要查找失败的 DSC 资源的所有实例。

1. 在“Log Analytics 工作区概述”页中，单击“日志”。
1. 在查询字段中键入以下搜索，针对警报创建日志搜索查询：`Type=AzureDiagnostics Category='DscNodeStatus' OperationName='DscResourceStatusData' ResultType='Failed'`

### <a name="view-historical-dsc-node-status"></a>查看历史 DSC 节点状态

最后，可能需要可视化不同时间段的 DSC 节点状态历史记录。  
可以使用此查询来搜索 DSC 节点状态在不同时间段的状态。

`Type=AzureDiagnostics ResourceProvider="MICROSOFT.AUTOMATION" Category=DscNodeStatus NOT(ResultType="started") | measure Count() by ResultType interval 1hour`  

这将显示不同时间段的节点状态的图表。

## <a name="azure-monitor-logs-records"></a>Azure Monitor 日志记录

来自 Azure 自动化的诊断将在 Azure Monitor 日志中创建两种类别的记录。

### <a name="dscnodestatusdata"></a>DscNodeStatusData

| 属性 | 描述 |
| --- | --- |
| TimeGenerated |符合性检查运行的日期和时间。 |
| OperationName |DscNodeStatusData |
| ResultType |节点是否符合。 |
| NodeName_s |托管节点的名称。 |
| NodeComplianceStatus_s |节点是否符合。 |
| DscReportStatus |符合性检查是否已成功运行。 |
| ConfigurationMode | 如何将配置应用到节点。 可能的值为“ApplyOnly”、“ApplyandMonitior”和“ApplyandAutoCorrect”。 <ul><li>__ApplyOnly__：DSC 将应用配置，且不执行进一步操作，除非有新配置被推送到目标节点或从服务器请求新配置。 首次应用新配置后，DSC 将不检查以前配置状态的偏离。 在 ApplyOnly 生效之前，DSC 将尝试应用配置，直到成功。 </li><li> __ApplyAndMonitor__：这是默认值。 LCM 将应用任意新配置。 首次应用新配置后，如果目标节点偏离所需状态，DSC 将在日志中报告差异。 在 ApplyAndMonitor 生效之前，DSC 将尝试应用配置，直到成功。</li><li>__ApplyAndAutoCorrect__：DSC 将应用任意新配置。 首次应用新配置后，如果目标节点偏离所需状态，DSC 将在日志中报告差异，然后重新应用当前配置。</li></ul> |
| HostName_s | 托管节点的名称。 |
| IPAddress | 托管节点的 IPv4 地址。 |
| Category | DscNodeStatus |
| Resource | Azure 自动化帐户的名称。 |
| Tenant_g | GUID，用于为 Caller 标识租户。 |
| NodeId_g |标识托管节点的 GUID。 |
| DscReportId_g |标识报表的 GUID。 |
| LastSeenTime_t |上一次查看报表的日期和时间。 |
| ReportStartTime_t |报表开始的日期和时间。 |
| ReportEndTime_t |报表完成的日期和时间。 |
| NumberOfResources_d |在应用于节点的配置中调用的 DSC 资源数。 |
| SourceSystem | Azure Monitor 日志收集数据的方式。 对于 Azure 诊断，始终为 Azure。 |
| ResourceId |指定 Azure 自动化帐户。 |
| ResultDescription | 此操作的说明。 |
| SubscriptionId | 自动化帐户的 Azure 订阅 ID (GUID)。 |
| resourceGroup | 自动化帐户的资源组的名称。 |
| ResourceProvider | MICROSOFT.AUTOMATION |
| ResourceType | AUTOMATIONACCOUNTS |
| CorrelationId |用作相容性报告相关性 ID 的 GUID。 |

### <a name="dscresourcestatusdata"></a>DscResourceStatusData

| 属性 | 描述 |
| --- | --- |
| TimeGenerated |符合性检查运行的日期和时间。 |
| OperationName |DscResourceStatusData|
| ResultType |资源是否符合。 |
| NodeName_s |托管节点的名称。 |
| Category | DscNodeStatus |
| Resource | Azure 自动化帐户的名称。 |
| Tenant_g | GUID，用于为 Caller 标识租户。 |
| NodeId_g |标识托管节点的 GUID。 |
| DscReportId_g |标识报表的 GUID。 |
| DscResourceId_s |DSC 资源实例的名称。 |
| DscResourceName_s |DSC 资源的名称。 |
| DscResourceStatus_s |DSC 资源是否具有符合性。 |
| DscModuleName_s |包含 DSC 资源的 PowerShell 模块的名称。 |
| DscModuleVersion_s |包含 DSC 资源的 PowerShell 模块的版本。 |
| DscConfigurationName_s |应用于节点的配置的名称。 |
| ErrorCode_s | 资源失败时的错误代码。 |
| ErrorMessage_s |资源失败时的错误消息。 |
| DscResourceDuration_d |DSC 资源运行的时间（以秒为单位）。 |
| SourceSystem | Azure Monitor 日志收集数据的方式。 对于 Azure 诊断，始终为 Azure。 |
| ResourceId |指定 Azure 自动化帐户。 |
| ResultDescription | 此操作的说明。 |
| SubscriptionId | 自动化帐户的 Azure 订阅 ID (GUID)。 |
| resourceGroup | 自动化帐户的资源组的名称。 |
| ResourceProvider | MICROSOFT.AUTOMATION |
| ResourceType | AUTOMATIONACCOUNTS |
| CorrelationId |用作相容性报告相关性 ID 的 GUID。 |

## <a name="summary"></a>摘要

将 Automation State Configuration 数据发送到 Azure Monitor 日志后，可以通过以下操作更好地了解 Automation State Configuration 节点的状态：

- 设置警报，以便在出现问题时获得通知
- 使用自定义视图和搜索查询直观地显示 Runbook 结果、Runbook 作业状态，以及其他相关的关键指标。  

Azure Monitor 日志可以更直观地显示 Automation State Configuration 数据的运行情况，并且有助于更快地解决事件。

## <a name="next-steps"></a>后续步骤

- 有关概述，请参阅 [Azure Automation State Configuration](automation-dsc-overview.md)
- 若要开始使用，请参阅 [Azure Automation State Configuration 入门](automation-dsc-getting-started.md)
- 若要了解如何编译 DSC 配置，以便将它们分配给目标节点，请参阅[在 Azure Automation State Configuration 中编译配置](automation-dsc-compile.md)
- 有关 PowerShell cmdlet 参考，请参阅 [Azure Automation State Configuration cmdlet](/powershell/module/azurerm.automation/#automation)
- 有关定价信息，请参阅 [Azure Automation State Configuration 定价](https://azure.microsoft.com/pricing/details/automation/)
- 若要查看在持续部署管道中使用 Azure Automation State Configuration 的示例，请参阅[使用 Azure Automation State Configuration 和 Chocolatey 进行持续部署](automation-dsc-cd-chocolatey.md)
- 若要详细了解如何使用 Azure Monitor 日志构造不同的搜索查询和查看 Automation State Configuration 日志，请参阅 [Azure Monitor 日志中的日志搜索](../log-analytics/log-analytics-log-searches.md)
- 若要了解有关 Azure Monitor 日志和数据收集源的详细信息，请参阅[在 Azure Monitor 日志中收集 Azure 存储数据概述](../azure-monitor/platform/collect-azure-metrics-logs.md)
