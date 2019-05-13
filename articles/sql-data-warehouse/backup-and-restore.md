---
title: Azure SQL 数据仓库备份和还原 - 快照，异地冗余 | Microsoft Docs
description: 了解 Azure SQL 数据仓库中备份和还原的工作方式。 使用数据仓库备份可以将数据仓库还原到主要区域的某个还原点。 使用异地冗余备份可还原到不同的地理区域。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: manage
ms.date: 04/30/2019
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 914513bc19cc81da29efef12d50a6485233d169f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236573"
---
# <a name="backup-and-restore-in-azure-sql-data-warehouse"></a>Azure SQL 数据仓库中的备份和还原

了解如何在 Azure SQL 数据仓库中使用备份和还原。 使用数据仓库还原点来恢复或复制数据仓库到主要区域中的之前的状态。 使用数据仓库异地冗余备份可还原到不同的地理区域。

## <a name="what-is-a-data-warehouse-snapshot"></a>什么是数据仓库快照

数据仓库快照会创建一个还原点，利用该还原点可将数据仓库恢复或复制到以前的状态。  由于 SQL 数据仓库属于分布式系统，因此数据仓库快照包含许多位于 Azure 存储中的文件。 快照捕获数据仓库中存储的数据的增量更改。

数据仓库还原是基于现有数据仓库或已删除数据仓库的还原点创建的新数据仓库。 还原数据仓库是任何业务连续性和灾难恢复策略的基本组成部分，因为数据库还原可以在意外损坏或删除数据后重新创建数据。 此外，数据仓库是出于测试或开发目的创建数据仓库副本的强大机制。  SQL 数据仓库还原费率根据数据库大小和源和目标数据仓库的位置而异。 在同一区域内的平均值，恢复率通常需要大约 20 分钟。 

## <a name="automatic-restore-points"></a>自动还原点

快照是创建还原点的服务的内置功能。 不需要启用此功能。 目前用户无法删除自动还原点，因为服务使用这些还原点来维持恢复 SLA。

SQL 数据仓库为数据仓库创建全天快照，并创建可用 7 天的还原点。 无法更改此保留期。 SQL 数据仓库支持八小时恢复点目标 (RPO)。 可从过去 7 天创建的任一快照还原主要区域中的数据仓库。

若要查看上一个快照的启动时间，可对联机 SQL 数据仓库运行以下查询。

```sql
select   top 1 *
from     sys.pdw_loader_backup_runs
order by run_id desc
;
```

## <a name="user-defined-restore-points"></a>用户定义的还原点

使用此功能，可以在大型修改之前和之后手动触发快照，以便创建数据仓库的还原点。 此功能可确保在出现工作负荷中断或用户错误的情况下，还原点在逻辑上是一致的，这样可以提供额外的数据保护，缩短恢复时间。 用户定义的还原点可用 7 天，7 天后系统会自动将其删除。 无法更改用户定义的还原点的保留期。 无论在任何时间点，均会保证 **42 个用户定义的还原点**，因此，它们必须在创建另一个还原点之前[删除](https://go.microsoft.com/fwlink/?linkid=875299)。 可以通过 [PowerShell](https://docs.microsoft.com/powershell/module/az.sql/new-azsqldatabaserestorepoint#examples) 或 Azure 门户触发快照来创建用户定义的还原点。

> [!NOTE]
> 如需将还原点保留 7 天以上，请在[此处](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/35114410-user-defined-retention-periods-for-restore-points)为此功能投票。 此外，可以创建用户定义的还原点，然后从新建的还原点还原到新数据仓库。 还原后，数据仓库将会联机，可以无限期将其暂停，以节省计算成本。 暂停的数据库按 Azure 高级存储费率收取存储费用。 如需已还原数据仓库的活动副本，可以执行恢复，只需花费几分钟时间。

### <a name="restore-point-retention"></a>还原点保留期

下面列出了还原点保留期的详细信息：

1. SQL 数据仓库会在达到 7 天保留期**并且**总共至少有 42 个还原点（包括用户定义的还原点和自动还原点）时删除还原点
2. 数据仓库暂停时不会创建快照
3. 还原点的存在时长是从创建还原点的时间算起的绝对日历天数（包括数据仓库暂停的时间）
4. 在任何时间点，数据仓库均保证能够存储最多 42 个用户定义的还原点和 42 个自动还原点，只要这些还原点尚未达到 7 天保留期
5. 如果创建了快照，然后数据仓库暂停 7 天以上的时间，然后进行了恢复，那么还原点可能持续存在，直到总共有 42 个还原点（包括用户定义的还原点和自动还原点）

### <a name="snapshot-retention-when-a-data-warehouse-is-dropped"></a>删除数据仓库时的快照保留期

删除数据仓库时，SQL 数据仓库将创建一个最终快照并保存七天。 可以将数据仓库还原至删除时所创建的最终还原点。

> [!IMPORTANT]
> 如果删除某个逻辑 SQL Server 实例，则属于该实例的所有数据库也会删除，且无法恢复。 无法还原已删除的服务器。

## <a name="geo-backups-and-disaster-recovery"></a>异地备份和灾难恢复

SQL 数据仓库对[已配对数据中心](../best-practices-availability-paired-regions.md)每天执行一次异地备份。 异地还原的 RPO 为 24 小时。 你可以将异地备份恢复到支持 SQL 数据仓库的任何其他地区的服务器。 使用异地备份可在无法访问主要区域中的还原点时还原数据仓库。

默认情况下，异地备份处于启用状态。 如果数据仓库为 Gen1，则可按需[选择退出](/powershell/module/az.sql/set-azsqldatabasegeobackuppolicy)。 不能为 Gen2 禁用异地备份，因为数据保护是固有的保证。

> [!NOTE]
> 如果希望异地备份的 RPO 更短，请在[此处](https://feedback.azure.com/forums/307516-sql-data-warehouse)为此功能投票。 此外，可以创建用户定义的还原点，然后从新建的还原点还原到其他区域中的新数据仓库。 还原后，数据仓库将会联机，可以无限期将其暂停，以节省计算成本。 暂停的数据库按 Azure 高级存储费率收取存储费用。 如需数据仓库的活动副本，可以执行恢复，只需花费几分钟时间。

## <a name="backup-and-restore-costs"></a>备份和还原成本

Azure 帐单上将列出存储的明细项目，以及灾难恢复存储的明细项目。 存储费用是在主要区域中存储数据的费用加上快照捕获增量更改的费用。 有关快照收费方式的更详细说明，请参阅[了解快照如何收取费用](https://docs.microsoft.com/rest/api/storageservices/Understanding-How-Snapshots-Accrue-Charges?redirectedfrom=MSDN#snapshot-billing-scenarios)。 异地冗余费用是指存储异地备份的费用。  

主数据仓库和 7 天快照更改的总费用根据 TB 数的舍入近似值计算。 例如，如果数据仓库为 1.5 TB，快照捕获了 100 GB，则会根据 Azure 高级存储费率计收 2 TB 数据的费用。

如果使用的是异地冗余存储，则会单独收取异地存储费。 异地冗余存储按标准的读取访问异地冗余存储 (RA-GRS) 费率计费。

有关 SQL 数据仓库定价的详细信息，请参阅 [SQL 数据仓库定价]。 跨区域还原时，不会收费针对数据流出量。

## <a name="restoring-from-restore-points"></a>从还原点还原

每个快照创建一个代表快照开始时间的还原点。 要还原数据仓库，请选择一个还原点，并发出还原命令。  

可以保留还原的数据仓库和当前的数据仓库，也可以删除其中一个。 如果希望使用已还原数据仓库替换当前数据仓库，可使用 [ALTER DATABASE（Azure SQL 数据仓库）](/sql/t-sql/statements/alter-database-azure-sql-data-warehouse)的“修改名称”选项对其进行重命名。

若要还原数据仓库，请参阅[使用 Azure 门户还原数据仓库](sql-data-warehouse-restore-database-portal.md)、[使用 PowerShell 还原数据仓库](sql-data-warehouse-restore-database-powershell.md)或[使用 REST API 还原数据仓库](sql-data-warehouse-restore-database-rest-api.md)。

若要还原已删除或已暂停的数据仓库，则可以[创建支持票证](sql-data-warehouse-get-started-create-support-ticket.md)。

## <a name="cross-subscription-restore"></a>跨订阅还原

如果你需要直接跨订阅还原，为此功能投票[此处](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/36256231-enable-support-for-cross-subscription-restore)。 还原到不同的逻辑服务器和[移动](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)跨订阅以执行跨订阅还原服务器。 

## <a name="geo-redundant-restore"></a>异地冗余还原

可[将数据仓库还原](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-restore-database-powershell#restore-from-an-azure-geographical-region)到支持所选性能级别的 SQL 数据仓库的任何区域。

> [!NOTE]
> 若要执行异地冗余还原，不能选择退出此功能。

## <a name="next-steps"></a>后续步骤

有关灾难规划的详细信息，请参阅[业务连续性概述](../sql-database/sql-database-business-continuity.md)
