---
title: Azure 备份：使用 Azure Monitor 监视 Azure 备份
description: 监视 Azure 备份的工作负荷并创建使用 Azure Monitor 的自定义警报
services: backup
author: pvrk
manager: shivamg
keywords: Log Analytics;Azure 备份;警报;诊断设置;操作组
ms.service: backup
ms.topic: conceptual
ms.date: 02/26/2019
ms.author: pullabhk
ms.assetid: 01169af5-7eb0-4cb0-bbdb-c58ac71bf48b
ms.openlocfilehash: 94fde7714f3efe0a460983966923071bce1afcc6
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190508"
---
# <a name="monitoring-at-scale-using-azure-monitor"></a>大规模使用 Azure Monitor 监视

[内置的监视和警报文章](backup-azure-monitoring-built-in-monitor.md)列出的监视和警报中单个恢复服务保管库，并且其中没有任何附加的管理基础结构提供的功能。 但是在以下情况下限制的内置服务。

- 在订阅之间的多个 RS 保管库中的监视数据
- 如果电子邮件不是首选的通知通道
- 如果用户想要接收警报的更多方案
- 如果想要查看的本地组件，如 System Center DPM (SC DPM) 中的信息以在 Azure 中，而不会显示在[备份作业](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault)或[备份警报](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault)在门户中。

## <a name="using-log-analytics-workspace"></a>使用 Log Analytics 工作区

> [!NOTE]
> 从 Azure VM 备份，MAB 代理，System Center DPM (SC DPM)，在 Azure Vm 中 SQL 备份的数据是被抽取到 Log Analytics 工作区通过诊断设置。 支持 Azure 文件共享备份，Microsoft Azure 备份服务器 (MABS) 即将推出。

我们利用两个 Azure 服务的功能**诊断设置**（若要将数据从多个 Azure 资源管理器资源发送到另一个资源） 和**Log Analytics** (LA-生成您可以在其中定义使用操作组的其他通知通道的自定义警报） 用于监视在规模较大。 有关如何使用 LA 大规模监视 Azure 备份以下各节详细信息。

### <a name="configuring-diagnostic-settings"></a>配置诊断设置

例如，Azure 恢复服务保管库的 Azure 资源管理器资源记录有关计划的操作、 用户触发的操作为诊断数据的所有可能的信息。 单击监视部分中的诊断设置并指定 RS 保管库的诊断数据的目标。

![RS 保管库与最新为目标的诊断设置](media/backup-azure-monitoring-laworkspace/rs-vault-diagnostic-setting.png)

从另一个订阅作为目标，可以选择一个 LA 工作区。 *通过选择多个 RS 保管库的同一个 LA 工作区，可以跨订阅在一个位置监视保管库。* 选择 azure 备份报表作为日志通道所有 Azure 备份相关的信息到最新工作区。

> [!IMPORTANT]
> 已完成配置后，应等待 24 小时后，初始数据推送来完成。 此后所有事件推送中所述[频率部分](#diagnostic-data-update-frequency)。

### <a name="deploying-solution-to-log-analytics-workspace"></a>解决方案部署到 Log Analytics 工作区

当数据在 LA 工作区中后,[部署 github 模板](https://azure.microsoft.com/resources/templates/101-backup-oms-monitoring/)到 LA 来可视化数据。 请确保你提供相同的资源组、 工作区名称和工作区的位置正确标识工作区并在其上安装此模板。

### <a name="view-azure-backup-data-using-log-analytics-la"></a>查看 Azure 备份数据使用日志分析 (LA)

一旦部署模板，用于监视 Azure 备份解决方案将显示在工作区摘要区域中。 可以通过遍历

- Azure 监视器-"的详细信息"> Insights 部分下，并选择相关的工作区或
- Log Analytics 工作区-> 选择相关的工作区-在常规部分中的工作区摘要 >。

![AzureBackupLAMonitoringTile](media/backup-azure-monitoring-laworkspace/la-azurebackup-azuremonitor-tile.png)

一系列有关如 Azure 备份中的基本监视数据的关系图上单击磁贴，打开设计器的模板

#### <a name="all-backup-jobs"></a>所有备份作业

![LABackupJobs](media/backup-azure-monitoring-laworkspace/la-azurebackup-allbackupjobs.png)

#### <a name="restore-jobs"></a>还原作业

![LARestoreJobs](media/backup-azure-monitoring-laworkspace/la-azurebackup-restorejobs.png)

#### <a name="inbuilt-azure-backup-alerts-for-azure-resources"></a>Azure 资源的内置的 Azure 备份警报

![LAInbuiltAzureBackupAlertsForAzureResources](media/backup-azure-monitoring-laworkspace/la-azurebackup-activealerts.png)

#### <a name="inbuilt-azure-backup-alerts-for-on-prem-resources"></a>在本地资源的内置的 Azure 备份警报

![LAInbuiltAzureBackupAlertsForOnPremResources](media/backup-azure-monitoring-laworkspace/la-azurebackup-activealerts-onprem.png)

#### <a name="active-datasources"></a>活动数据源

![LAActiveBackedUpEntities](media/backup-azure-monitoring-laworkspace/la-azurebackup-activedatasources.png)

#### <a name="rs-vault-cloud-storage"></a>RS 保管库的云存储

![LARSVaultCloudStorage](media/backup-azure-monitoring-laworkspace/la-azurebackup-cloudstorage-in-gb.png)

上面的关系图模板一起提供，并且客户可以随时编辑/添加更多的关系图。

> [!IMPORTANT]
> 在部署模板时，它实质上是创建只读锁，并需要先删除该编辑该模板并保存。 若要删除锁，请在 Log Analytics 工作区的设置部分下的锁定窗格中查找。

### <a name="create-alerts-using-log-analytics"></a>创建使用 Log Analytics 警报

Azure 监视器允许用户从 LA 工作区，可以创建自己的警报*利用 Azure 操作组，以选择您首选的通知机制*。 单击任意上述打开 LA 工作区的日志部分的图形***在可以编辑查询和创建基于这些警报***。

![LACreateAlerts](media/backup-azure-monitoring-laworkspace/la-azurebackup-customalerts.png)

单击"新建警报规则"如上所示将打开 Azure Monitor 警报创建屏幕。

如你可以看到如下，资源已标记为 LA 工作区，并提供操作组集成。

![LAAzureBackupCreateAlert](media/backup-azure-monitoring-laworkspace/inkedla-azurebackup-createalert.jpg)

> [!IMPORTANT]
> 请注意，提供创建此查询的相关的定价影响[此处](https://azure.microsoft.com/pricing/details/monitor/)。

#### <a name="alert-condition"></a>警报条件

重要的方面是警报的触发条件。 单击条件将自动加载 Kusto 查询，如下所示的日志屏幕中，您可以编辑它以根据自己的方案。 中提供了一些示例 Kusto 查询[下面的部分](#sample-kusto-queries)。

![LAAzureBackupAlertCondition](media/backup-azure-monitoring-laworkspace/la-azurebackup-alertlogic.png)

编辑 Kusto 查询，如有必要，请选择正确的阈值 （这将决定时，将触发警报）、 右段 （时间窗口对其运行查询），以及的频率。 例如：如果阈值为大于 0，是 5 分钟且频率为 5 分钟，然后该规则将被转换为"过去 5 分钟内每隔 5 分钟运行查询和结果数是否大于 0，通知我通过选定的操作组"

#### <a name="action-group-integration"></a>操作组集成

操作组指定可供用户使用的通知通道。 单击"新建"下"操作组"部分显示可用的通知机制列表。

![LAAzureBackupNewActionGroup](media/backup-azure-monitoring-laworkspace/LA-AzureBackup-ActionGroup.png)

详细了解如何[LA 工作区中的警报](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log)而是有关[操作组](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups)从 Azure Monitor 文档。

因此可以满足所有警报和监视从 LA 单独的要求，或将其用作在生成的通知机制的补充技术。

### <a name="sample-kusto-queries"></a>示例 Kusto 查询

默认图形将为您提供 Kusto 查询其上生成警报的基本方案。 此外可以修改它们以获取你想要发出警报的数据。 这里我们提供可以一些示例 Kusto 查询粘贴到日志窗口，然后基于该查询创建警报。

#### <a name="all-successful-backup-jobs"></a>所有成功的备份作业

````Kusto
AzureDiagnostics
| where Category == "AzureBackupReport"
| where SchemaVersion_s == "V2"
| where OperationName == "Job" and JobOperation_s == "Backup"
| where JobStatus_s == "Completed"
````

#### <a name="all-failed-backup-jobs"></a>所有失败的备份作业

````Kusto
AzureDiagnostics
| where Category == "AzureBackupReport"
| where SchemaVersion_s == "V2"
| where OperationName == "Job" and JobOperation_s == "Backup"
| where JobStatus_s == "Failed"
````

#### <a name="all-successful-azure-vm-backup-jobs"></a>所有成功的 Azure VM 备份作业

````Kusto
AzureDiagnostics
| where Category == "AzureBackupReport"
| where SchemaVersion_s == "V2"
| extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
| where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s != "Log" and JobOperationSubType_s != "Recovery point_Log"
| join kind=inner
(
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where OperationName == "BackupItem"
    | where SchemaVersion_s == "V2"
    | where BackupItemType_s == "VM" and BackupManagementType_s == "IaaSVM"
    | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
    | project BackupItemUniqueId_s , BackupItemFriendlyName_s
)
on BackupItemUniqueId_s
| extend Vault= Resource
| project-away Resource
````

#### <a name="all-successful-sql-log-backup-jobs"></a>所有成功的 SQL 日志备份作业

````Kusto
AzureDiagnostics
| where Category == "AzureBackupReport"
| where SchemaVersion_s == "V2"
| extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
| where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s == "Log"
| join kind=inner
(
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where OperationName == "BackupItem"
    | where SchemaVersion_s == "V2"
    | where BackupItemType_s == "SQLDataBase" and BackupManagementType_s == "AzureWorkload"
    | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
    | project BackupItemUniqueId_s , BackupItemFriendlyName_s
)
on BackupItemUniqueId_s
| extend Vault= Resource
| project-away Resource
````

#### <a name="all-successful-mab-agent-backup-jobs"></a>所有成功的 MAB 代理备份作业

````Kusto
AzureDiagnostics
| where Category == "AzureBackupReport"
| where SchemaVersion_s == "V2"
| extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
| where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s != "Log" and JobOperationSubType_s != "Recovery point_Log"
| join kind=inner
(
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where OperationName == "BackupItem"
    | where SchemaVersion_s == "V2"
    | where BackupItemType_s == "FileFolder" and BackupManagementType_s == "MAB"
    | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
    | project BackupItemUniqueId_s , BackupItemFriendlyName_s
)
on BackupItemUniqueId_s
| extend Vault= Resource
| project-away Resource
````

### <a name="diagnostic-data-update-frequency"></a>诊断数据更新频率

在保管库中的诊断数据抽取到某些基准并以 LA 工作区中。 每个事件到达 LA 工作区***20-30 分钟后它会从 RS 保管库中推送为基准。***

- 在创建时，就立即推送 （跨所有解决方案） 的备份服务内置警报。 这意味着它们通常会出现在 LA 工作区后存在 20-30 分钟的延迟。
- 即席备份作业和还原作业 （跨所有解决方案） 只要它们推送***完成***。
- 只要它们推送 （除 SQL 备份） 的所有解决方案从计划的备份作业***完成***。
- 为 SQL 备份，因为我们有日志备份每 15 分钟，对于所有已完成计划备份作业，包括日志、 批处理和信息推送每 6 小时。
- 所有其他信息，例如备份项、 策略、 恢复点、 存储跨所有解决方案等推送**至少每天一次。**
- 例如，更改策略，编辑策略等的备份配置中的更改将触发备份的所有相关信息的推送。

## <a name="using-rs-vaults-activity-logs"></a>使用 RS 保管库的活动日志

此外可以使用活动日志以获取事件，例如备份成功的通知。

> [!CAUTION]
> **请注意这仅适用于 Azure VM 备份。** 您不能使用此 SQL 备份在 Azure 中，Azure 文件等 Azure 备份代理，如其他解决方案。

### <a name="sign-in-into-azure-portal"></a>登录到 Azure 门户

登录到 Azure 门户，转到相关的 Azure 恢复服务保管库，并单击在属性中的"活动日志"部分。

### <a name="identify-appropriate-log-and-create-alert"></a>标识相应的日志，并创建警报

应用下图所示的筛选器，以便验证是否接收到成功备份的活动日志。 相应地更改时间跨度以查看视图。

![Azure VM 备份的活动日志](media/backup-azure-monitoring-laworkspace/activitylogs-azurebackup-vmbackups.png)

可以单击“JSON”段，获取更多详细信息，并将其复制粘贴到文本编辑器进行查看。 它应显示保管库详细信息和触发活动的项的它是日志备份的项。

然后单击“添加活动日志警报”以生成所有此类日志的警报。

你可以单击"添加活动日志警报"上面所示，它将打开类似于警报创建屏幕的警报创建屏幕[上文所述](#create-alerts-using-log-analytics)。

此处的资源是 RS 保管库本身，因此您需要为要在其中通过活动日志通知的所有保管库中重复相同的操作。 由于这是基于事件的警报，条件将不会有任何阈值、 句点、 频率。 只要生成相关的活动日志，触发警报。

## <a name="recommendation"></a>建议

***从活动日志创建的所有警报，LA 工作区可以是 Azure Monitor 中在警报窗格的左侧。***

尽管可以使用通知通过活动日志，但***Azure 备份服务强烈建议用于在规模和不活动日志监视由于以下原因 LA***。

- **有限的情况：** 仅适用于 Azure VM 备份，并且应重复的每个 RS 保管库。
- **定义适合：** 计划的备份活动不适合使用活动日志的最新定义，并符合[诊断日志](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-logs-overview#what-are-azure-monitor-diagnostic-logs)。 意外的影响，如下面所述更改通过活动日志通道发送的数据时此销售线索。
- **与活动日志通道相关的问题：** 我们已经转移到新的模型，即从恢复服务保管库上的 Azure 备份中抽取活动日志。 遗憾的是，在移动影响 Azure 主权云中的活动日志的生成。 如果 Azure 主权云用户创建或配置任何从通过 Azure Monitor 活动日志警报，它们不会触发。 此外，在所有 Azure 公共区域中，如果用户正在将恢复服务活动日志收集到[此处](https://docs.microsoft.com/azure/azure-monitor/platform/collect-activity-logs)提到的 Log Analytics 工作区中，那么这些日志也不会显示。

因此，强烈建议用于日志分析工作区进行监视和大规模警报针对所有 Azure 备份保护工作负荷。

## <a name="next-steps"></a>后续步骤

- 请参阅[Log analytics 数据模型](backup-azure-log-analytics-data-model.md)若要创建自定义查询。
