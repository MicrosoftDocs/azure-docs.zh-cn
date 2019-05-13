---
title: 使用 PowerShell 创建和配置 Log Analytics 工作区 | Microsoft 文档
description: Azure Monitor 中的 Log Analytics 工作区存储来自本地或云基础结构中的服务器的数据。 当由 Azure 诊断生成时，可从 Azure 存储收集计算机数据。
services: log-analytics
author: bwren
ms.service: log-analytics
ms.devlang: powershell
ms.topic: conceptual
ms.date: 02/28/2019
ms.author: bwren
ms.openlocfilehash: 2d3f1ab6704a0f5ecd15190fd08b10485cdf1ee9
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510108"
---
# <a name="manage-log-analytics-workspace-in-azure-monitor-using-powershell"></a>使用 PowerShell 管理 Azure Monitor 中的 Log Analytics 工作区

可使用 [Log Analytics PowerShell cmdlet](https://docs.microsoft.com/powershell/module/az.operationalinsights/) 从命令行或作为脚本的一部分在 Azure Monitor 的 Log Analytics 工作区中执行各种函数。  可使用 PowerShell 执行的任务示例包括：

* 创建工作区
* 添加或删除解决方案
* 导入和导出保存的搜索
* 创建计算机组
* 从装有 Windows 代理的计算机启用 IIS 日志收集
* 从 Linux 和 Windows 计算机中收集性能计数器
* 从 Linux 计算机的 syslog 中收集事件
* 从 Windows 事件日志中收集事件
* 收集自定义事件日志
* 将日志分析代理添加到 Azure 虚拟机
* 配置 Log Analytics 以便为使用 Azure 诊断收集的数据编制索引

本文提供两个代码示例，用于演示一些可从 PowerShell 执行的函数。  有关其他函数，可以参阅 [Log Analytics PowerShell cmdlet 参考](https://docs.microsoft.com/powershell/module/az.operationalinsights/)。

> [!NOTE]
> Log Analytics 之前称为 Operational Insights，这就是在 cmdlet 中使用此名称的原因。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>必备组件
这些示例适用于 Az.OperationalInsights 模块 1.0.0 或更高版本。


## <a name="create-and-configure-a-log-analytics-workspace"></a>创建和配置 Log Analytics 工作区
以下脚本示例演示了如何：

1. 创建工作区
2. 列出可用解决方案
3. 向工作区添加解决方案
4. 导入保存的搜索
5. 导出保存的搜索
6. 创建计算机组
7. 从装有 Windows 代理的计算机启用 IIS 日志收集
8. 从 Linux 计算机中收集逻辑磁盘性能计数器 (% Used Inodes; Free Megabytes; % Used Space; Disk Transfers/sec; Disk Reads/sec; Disk Writes/sec)
9. 从 Linux 计算机中收集 syslog 事件
10. 从 Windows 计算机的应用程序事件日志中收集错误和警告事件
11. 从 Windows 计算机中收集可用内存 (MB) 性能计数器
12. 收集自定义日志

```powershell

$ResourceGroup = "oms-example"
$WorkspaceName = "log-analytics-" + (Get-Random -Maximum 99999) # workspace names need to be unique - Get-Random helps with this for the example code
$Location = "westeurope"

# List of solutions to enable
$Solutions = "Security", "Updates", "SQLAssessment"

# Saved Searches to import
$ExportedSearches = @"
[
    {
        "Category":  "My Saved Searches",
        "DisplayName":  "WAD Events (All)",
        "Query":  "Type=Event SourceSystem:AzureStorage ",
        "Version":  1
    },
    {
        "Category":  "My Saved Searches",
        "DisplayName":  "Current Disk Queue Length",
        "Query":  "Type=Perf ObjectName=LogicalDisk InstanceName=\"C:\" CounterName=\"Current Disk Queue Length\"",
        "Version":  1
    }
]
"@ | ConvertFrom-Json

# Custom Log to collect
$CustomLog = @"
{
    "customLogName": "sampleCustomLog1",
    "description": "Example custom log datasource",
    "inputs": [
        {
            "location": {
            "fileSystemLocations": {
                "windowsFileTypeLogPaths": [ "e:\\iis5\\*.log" ],
                "linuxFileTypeLogPaths": [ "/var/logs" ]
                }
            },
        "recordDelimiter": {
            "regexDelimiter": {
                "pattern": "\\n",
                "matchIndex": 0,
                "matchIndexSpecified": true,
                "numberedGroup": null
                }
            }
        }
    ],
    "extractions": [
        {
            "extractionName": "TimeGenerated",
            "extractionType": "DateTime",
            "extractionProperties": {
                "dateTimeExtraction": {
                    "regex": null,
                    "joinStringRegex": null
                    }
                }
            }
        ]
    }
"@

# Create the resource group if needed
try {
    Get-AzResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzResourceGroup -Name $ResourceGroup -Location $Location
}

# Create the workspace
New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku Standard -ResourceGroupName $ResourceGroup

# List all solutions and their installation status
Get-AzOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Add solutions
foreach ($solution in $Solutions) {
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true
}

# List enabled solutions
(Get-AzOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName).Where({($_.enabled -eq $true)})

# Import Saved Searches
foreach ($search in $ExportedSearches) {
    $id = $search.Category + "|" + $search.DisplayName
    New-AzOperationalInsightsSavedSearch -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -SavedSearchId $id -DisplayName $search.DisplayName -Category $search.Category -Query $search.Query -Version $search.Version
}

# Export Saved Searches
(Get-AzOperationalInsightsSavedSearch -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName).Value.Properties | ConvertTo-Json

# Create Computer Group based on a query
New-AzOperationalInsightsComputerGroup -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -SavedSearchId "My Web Servers" -DisplayName "Web Servers" -Category "My Saved Searches" -Query "Computer=""web*"" | distinct Computer" -Version 1

# Enable IIS Log Collection using agent
Enable-AzOperationalInsightsIISLogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Linux Perf
New-AzOperationalInsightsLinuxPerformanceObjectDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -ObjectName "Logical Disk" -InstanceName "*"  -CounterNames @("% Used Inodes", "Free Megabytes", "% Used Space", "Disk Transfers/sec", "Disk Reads/sec", "Disk Reads/sec", "Disk Writes/sec") -IntervalSeconds 20  -Name "Example Linux Disk Performance Counters"
Enable-AzOperationalInsightsLinuxCustomLogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Linux Syslog
New-AzOperationalInsightsLinuxSyslogDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -Facility "kern" -CollectEmergency -CollectAlert -CollectCritical -CollectError -CollectWarning -Name "Example kernel syslog collection"
Enable-AzOperationalInsightsLinuxSyslogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Windows Event
New-AzOperationalInsightsWindowsEventDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -EventLogName "Application" -CollectErrors -CollectWarnings -Name "Example Application Event Log"

# Windows Perf
New-AzOperationalInsightsWindowsPerformanceCounterDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -ObjectName "Memory" -InstanceName "*" -CounterName "Available MBytes" -IntervalSeconds 20 -Name "Example Windows Performance Counter"

# Custom Logs

New-AzOperationalInsightsCustomLogDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -CustomLogRawJson "$CustomLog" -Name "Example Custom Log Collection"

```
在上面的示例中，regexDelimiter 定义为“\\n”表示换行符。 日志分隔符也可能是时间戳。  以下是支持的格式：

| 格式 | Json 正则表达式格式使用两个 \\ 表示标准正则表达式中的每个 \，因此，如果在正则表达式应用中测试，请将 \\ 减少为 \ | | |
| --- | --- | --- | --- |
| `YYYY-MM-DD HH:MM:SS` | `((\\\\d{2})\|(\\\\d{4}))-([0-1]\\\\d)-(([0-3]\\\\d)\|(\\\\d))\\\\s((\\\\d)\|([0-1]\\\\d)\|(2[0-4])):[0-5][0-9]:[0-5][0-9]` | | |
| `M/D/YYYY HH:MM:SS AM/PM` | `(([0-1]\\\\d)\|[0-9])/(([0-3]\\\\d)\|(\\\\d))/((\\\\d{2})\|(\\\\d{4}))\\\\s((\\\\d)\|([0-1]\\\\d)\|(2[0-4])):[0-5][0-9]:[0-5][0-9]\\\\s(AM\|PM\|am\|pm)` | | |
| `dd/MMM/yyyy HH:MM:SS` | `((([0-3]\\\\d)\` | `(\\\\d))/(Jan\|Feb\|Mar\|May\|Apr\|Jul\|Jun\|Aug\|Oct\|Sep\|Nov\|Dec\|jan\|feb\|mar\|may\|apr\|jul\|jun\|aug\|oct\|sep\|nov\|dec)/((\\\\d{2})\|(\\\\d{4}))\\\\s((\\\\d)\` | `([0-1]\\\\d)\|(2[0-4])):[0-5][0-9]:[0-5][0-9])` |
| `MMM dd yyyy HH:MM:SS` | `(((?:Jan(?:uary)?\|Feb(?:ruary)?\|Mar(?:ch)?\|Apr(?:il)?\|May\|Jun(?:e)?\|Jul(?:y)?\|Aug(?:ust)?\|Sep(?:tember)?\|Sept\|Oct(?:ober)?\|Nov(?:ember)?\|Dec(?:ember)?)).*?((?:(?:[0-2]?\\\\d{1})\|(?:[3][01]{1})))(?![\\\\d]).*?((?:(?:[1]{1}\\\\d{1}\\\\d{1}\\\\d{1})\|(?:[2]{1}\\\\d{3})))(?![\\\\d]).*?((?:(?:[0-1][0-9])\|(?:[2][0-3])\|(?:[0-9])):(?:[0-5][0-9])(?::[0-5][0-9])?(?:\\\\s?(?:am\|AM\|pm\|PM))?))` | | |
| `yyMMdd HH:mm:ss` | `([0-9]{2}([0][1-9]\|[1][0-2])([0-2][0-9]\|[3][0-1])\\\\s\\\\s?([0-1]?[0-9]\|[2][0-3]):[0-5][0-9]:[0-5][0-9])` | | |
| `ddMMyy HH:mm:ss` | `(([0-2][0-9]\|[3][0-1])([0][1-9]\|[1][0-2])[0-9]{2}\\\\s\\\\s?([0-1]?[0-9]\|[2][0-3]):[0-5][0-9]:[0-5][0-9])` | | |
| `MMM d HH:mm:ss` | `(Jan\|Feb\|Mar\|Apr\|May\|Jun\|Jul\|Aug\|Sep\|Oct\|Nov\|Dec)\\\\s\\\\s?([0]?[1-9]\|[1-2][0-9]\|[3][0-1])\\\\s([0-1]?[0-9]\|[2][0-3]):([0-5][0-9]):([0-5][0-9])` | | |
| `MMM  d HH:mm:ss` <br> 在 MMM 后有两个空格 | `(Jan\|Feb\|Mar\|Apr\|May\|Jun\|Jul\|Aug\|Sep\|Oct\|Nov\|Dec)\\\\s\\\\s([0]?[1-9]\|[1-2][0-9]\|[3][0-1])\\\\s([0][0-9]\|[1][0-2]):([0-5][0-9]):([0-5][0-9])` | | |
| `MMM d HH:mm:ss` | `(Jan\|Feb\|Mar\|Apr\|May\|Jun\|Jul\|Aug\|Sep\|Oct\|Nov\|Dec)\\\\s([0]?[1-9]\|[1-2][0-9]\|[3][0-1])\\\\s([0][0-9]\|[1][0-2]):([0-5][0-9]):([0-5][0-9])` | | |
| `dd/MMM/yyyy:HH:mm:ss +zzzz` <br> 其中 + 是 + 或 - <br> 其中 zzzz 是时间偏移量 | `(([0-2][1-9]\|[3][0-1])\\\\/(Jan\|Feb\|Mar\|Apr\|May\|Jun\|Jul\|Aug\|Sep\|Oct\|Nov\|Dec)\\\\/((19\|20)[0-9][0-9]):([0][0-9]\|[1][0-2]):([0-5][0-9]):([0-5][0-9])\\\\s[\\\\+\|\\\\-][0-9]{4})` | | |
| `yyyy-MM-ddTHH:mm:ss` <br> T 是文字字母 T | `((\\\\d{2})\|(\\\\d{4}))-([0-1]\\\\d)-(([0-3]\\\\d)\|(\\\\d))T((\\\\d)\|([0-1]\\\\d)\|(2[0-4])):[0-5][0-9]:[0-5][0-9]` | | |

## <a name="configuring-log-analytics-to-send-azure-diagnostics"></a>将 Log Analytics 配置为发送 Azure 诊断
要对 Azure 资源进行无代理监视，则需要为资源启用 Azure 诊断，并将其配置为写入到 Log Analytics 工作区。 此方法将数据直接发送到工作区并且不要求将数据写入到存储帐户。 支持的资源包括：

| 资源类型 | 日志 | 度量值 |
| --- | --- | --- |
| 应用程序网关    | 是 | 是 |
| 自动化帐户     | 是 | |
| 批处理帐户          | 是 | 是 |
| Data Lake Analytics     | 是 | |
| Data Lake Store         | 是 | |
| SQL 弹性池        |     | 是 |
| 事件中心命名空间     |     | 是 |
| IoT 中心                |     | 是 |
| Key Vault               | 是 | |
| 负载均衡器          | 是 | |
| 逻辑应用              | 是 | 是 |
| 网络安全组 | 是 | |
| 用于 Redis 的 Azure 缓存             |     | 是 |
| 搜索服务         | 是 | 是 |
| 服务总线命名空间   |     | 是 |
| SQL (v12)               |     | 是 |
| 网站               |     | 是 |
| Web 服务器场        |     | 是 |

有关可用指标的详细信息，请参阅 [Azure 监视器支持的指标](../../azure-monitor/platform/metrics-supported.md)。

有关可用日志的详细信息，请参阅[诊断日志支持的服务和架构](../../azure-monitor/platform/diagnostic-logs-schema.md)。

```powershell
$workspaceId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$resourceId = "/SUBSCRIPTIONS/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/RESOURCEGROUPS/DEMO/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/DEMO"

Set-AzDiagnosticSetting -ResourceId $resourceId -WorkspaceId $workspaceId -Enabled $true
```

还可以使用前面的 cmdlet 从不同订阅中的资源收集日志。 该 cmdlet 能够跨订阅工作，你同时提供创建日志的资源的 ID 和日志发送到的工作区的 ID。


## <a name="configuring-log-analytics-workspace-to-collect-azure-diagnostics-from-storage"></a>配置 Log Analytics 工作区，以便收集存储中的 Azure 诊断信息
要从正在运行的经典云服务或 service fabric 群集实例内收集日志数据，需要首先将数据写入到 Azure 存储。 然后，Log Analytics 工作区将配置为从存储帐户收集日志。 支持的资源包括：

* 经典云服务（Web 和辅助角色）
* Service Fabric 群集

以下示例介绍如何：

1. 列出工作区将为其中的数据编制索引的现有存储帐户和位置
2. 创建配置以从存储帐户读取
3. 更新新创建的配置以为来自其他位置的数据编制索引
4. 删除新创建的配置

```powershell
# validTables = "WADWindowsEventLogsTable", "LinuxsyslogVer2v0", "WADServiceFabric*EventTable", "WADETWEventTable"
$workspace = (Get-AzOperationalInsightsWorkspace).Where({$_.Name -eq "your workspace name"})

# Update these two lines with the storage account resource ID and the storage account key for the storage account you want the workspace to index
$storageId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/demo/providers/Microsoft.Storage/storageAccounts/wadv2storage"
$key = "abcd=="

# List existing insights
Get-AzOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name

# Create a new insight
New-AzOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight" -StorageAccountResourceId $storageId -StorageAccountKey $key -Tables @("WADWindowsEventLogsTable") -Containers @("wad-iis-logfiles")

# Update existing insight
Set-AzOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight" -Tables @("WADWindowsEventLogsTable", "WADETWEventTable") -Containers @("wad-iis-logfiles")

# Remove the insight
Remove-AzOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight"

```

还可以使用前面的脚本从不同订阅中的存储帐户收集日志。 该脚本能够跨订阅工作，你同时提供存储帐户资源 ID 和对应的访问密钥。 当更改了访问密钥时，需要更新存储见解以采用新密钥。


## <a name="next-steps"></a>后续步骤
* 有关使用 Log Analytics 的 PowerShell 配置的其他信息，请[查看 Log Analytics PowerShell cmdlet](https://docs.microsoft.com/powershell/module/az.operationalinsights/)。

