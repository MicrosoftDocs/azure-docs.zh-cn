---
title: PowerShell 示例-更新 SQL 数据同步的同步架构 | Microsoft Docs
description: 用于更新 SQL 数据同步的同步架构的 Azure PowerShell 示例脚本
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: allenwux
ms.author: xiwu
ms.reviewer: carlrab
manager: craigg
ms.date: 03/12/2019
ms.openlocfilehash: edbff68e96a2b9309e7bb490af14d582e27c46a5
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60387802"
---
# <a name="use-powershell-to-update-the-sync-schema-in-an-existing-sync-group"></a>使用 PowerShell 更新现有同步组中的同步架构

此 PowerShell 示例更新现有“SQL 数据同步”同步组中的同步架构。 同步多个表时，此脚本可帮助你有效地更新同步架构。 此示例演示如何使用 **UpdateSyncSchema** 脚本，该脚本在 GitHub 上以 [UpdateSyncSchema.ps1](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/sql-data-sync/UpdateSyncSchema.ps1) 的形式提供。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果选择在本地安装并使用 PowerShell，则本教程需要 AZ PowerShell 1.4.0 或更高版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 来创建与 Azure 的连接。

有关 SQL 数据同步的概述，请参阅[使用 Azure SQL 数据同步跨多个云和本地数据库同步数据](../sql-database-sync-data.md)。

> [!IMPORTANT]
> 目前，Azure SQL 数据同步不支持 Azure SQL 数据库托管实例。

## <a name="sample-script"></a>示例脚本

### <a name="example-1---add-all-tables-to-the-sync-schema"></a>示例 1 - 将所有表添加到同步架构

以下示例可刷新数据库架构，并可将中心数据库中的所有有效表添加到同步架构。

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscription_id> -ResourceGroupName <resource_group_name> -ServerName <server_name> -DatabaseName <database_name> -SyncGroupName <sync_group_name> -RefreshDatabaseSchema $true -AddAllTables $true
```

### <a name="example-2---add-and-remove-tables-and-columns"></a>示例 2 - 添加和删除表和列

以下示例将 `[dbo].[Table1]` 和 `[dbo].[Table2].[Column1]` 添加到同步架构并删除 `[dbo].[Table3]`。

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscription_id> -ResourceGroupName <resource_group_name> -ServerName <server_name> -DatabaseName <database_name> -SyncGroupName <sync_group_name> -TablesAndColumnsToAdd "[dbo].[Table1],[dbo].[Table2].[Column1]" -TablesAndColumnsToRemove "[dbo].[Table3]"
```

## <a name="script-parameters"></a>脚本参数

**UpdateSyncSchema** 脚本具有以下参数：

| 参数 | 说明 |
|---|---|
| $SubscriptionId | 在其中创建同步组的订阅。 |
| $ResourceGroupName | 在其中创建同步组的资源组。|
| $ServerName | 中心数据库的服务器名称。|
| $DatabaseName | 中心数据库名称。 |
| $SyncGroupName | 同步组名称。 |
| $MemberName | 如果要从同步成员而不是中心数据库加载数据库架构，请指定成员名称。 如果要从中心加载数据库架构，则将此参数保留为空。 |
| $TimeoutInSeconds | 该脚本刷新数据库架构时超时。 默认为 900 秒。 |
| $RefreshDatabaseSchema | 指定该脚本是否需要刷新数据库架构。 如果数据库架构已从以前的配置更改（例如，如果添加了新表或新列），则需要在重新配置之前刷新架构。 默认值为 false。 |
| $AddAllTables | 如果此值为 true，所有有效的表和列都将添加到同步架构。 $TablesAndColumnsToAdd 和 $TablesAndColumnsToRemove 的值将被忽略。 |
| $TablesAndColumnsToAdd | 指定要添加到同步架构的表或列。 每个表名或列名必须与架构名称完全分隔。 例如：`[dbo].[Table1]`、`[dbo].[Table2].[Column1]`。 可以指定多个表名或列名，并将其用逗号 (,) 分隔。 |
| $TablesAndColumnsToRemove | 指定要从同步架构中删除的表或列。 每个表名或列名必须与架构名称完全分隔。 例如：`[dbo].[Table1]`、`[dbo].[Table2].[Column1]`。 可以指定多个表名或列名，并将其用逗号 (,) 分隔。 |
|||

## <a name="script-explanation"></a>脚本说明

**UpdateSyncSchema** 脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [Get-AzSqlSyncGroup](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncgroup) | 返回有关同步组的信息。 |
| [Update-AzSqlSyncGroup](https://docs.microsoft.com/powershell/module/az.sql/update-azsqlsyncgroup) | 更新同步组。 |
| [Get-AzSqlSyncMember](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncmember) | 返回有关同步成员的信息。 |
| [Get-AzSqlSyncSchema](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlsyncschema) | 返回有关同步架构的信息。 |
| [Update-AzSqlSyncSchema](https://docs.microsoft.com/powershell/module/az.sql/update-azsqlsyncschema) | 更新同步架构。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](/powershell/azure/overview)。

可以在 [Azure SQL 数据库 PowerShell 脚本](../sql-database-powershell-samples.md)中找到更多 SQL 数据库 PowerShell 脚本示例。

有关 SQL 数据同步的详细信息，请参阅：

-   概述 - [使用 Azure SQL 数据同步跨多个云和本地数据库同步数据](../sql-database-sync-data.md)
-   设置数据同步
    - 在门户中 - [教程：设置 SQL 数据同步，以在 Azure SQL 数据库和本地 SQL Server 之间同步数据](../sql-database-get-started-sql-data-sync.md)
    - 使用 PowerShell
        -  [使用 PowerShell 在多个 Azure SQL 数据库之间进行同步](sql-database-sync-data-between-sql-databases.md)
        -  [使用 PowerShell 在 Azure SQL 数据库和 SQL Server 本地数据库之间进行同步](sql-database-sync-data-between-azure-onprem.md)
-   Data Sync Agent - [Azure SQL 数据同步的 Data Sync Agent](../sql-database-data-sync-agent.md)
-   最佳做法 - [Azure SQL 数据同步最佳做法](../sql-database-best-practices-data-sync.md)
-   监视 - [使用 Azure Monitor 日志监视 SQL 数据同步](../sql-database-sync-monitor-oms.md)
-   故障排除 - [排查 Azure SQL 数据同步问题](../sql-database-troubleshoot-data-sync.md)
-   更新同步架构
    -   使用 Transact-SQL - [在 Azure SQL 数据同步中自动复制架构更改](../sql-database-update-sync-schema.md)

有关 SQL 数据库的详细信息，请参阅：

-   [SQL 数据库概述](../sql-database-technical-overview.md)
-   [数据库生命周期管理](https://msdn.microsoft.com/library/jj907294.aspx)
