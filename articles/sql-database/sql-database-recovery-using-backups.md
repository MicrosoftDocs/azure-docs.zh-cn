---
title: 从备份还原 Azure SQL 数据库 | Microsoft Docs
description: 了解有关时间点还原的信息，它让你能够将 Azure SQL 数据库回滚到之前的时间点（最多 35 天）。
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab
manager: craigg
ms.date: 04/30/2019
ms.openlocfilehash: 47bf59adb33f3685b31430c652b31880d383833e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232644"
---
# <a name="recover-an-azure-sql-database-using-automated-database-backups"></a>使用自动数据库备份恢复 Azure SQL 数据库

默认情况下，SQL 数据库备份存储在异地复制 blob 存储 (RA-GRS) 中。 以下选项适用于使用[自动数据库备份](sql-database-automated-backups.md)的数据库恢复：

- 在恢复到保留期内指定时间点的同一 SQL 数据库服务器上创建新数据库。
- 在恢复到已删除数据库的删除时间的同一 SQL 数据库服务器上创建数据库。
- 在恢复到最新备份点的相同区域中的任一 SQL 数据库服务器上创建新数据库。
- 在恢复到最近复制备份点的其他任何区域中的任一 SQL 数据库服务器上创建新数据库。

如果你配置[备份长期保留](sql-database-long-term-retention.md)，还可以从任何 SQL 数据库服务器上任何 LTR 备份创建新的数据库。

> [!IMPORTANT]
> 还原期间无法覆盖现有数据库。

使用“标准”或“高级”服务层级时，在遇到下列情况时，已还原数据库将产生额外的存储费用：

- 如果数据库最大大小超过 500 GB，将 P11–P15 还原为 S4-S12 或 P1–P6。
- 如果数据库最大大小超过 250 GB，将 P1–P6 还原为 S4-S12。

额外成本是 icurred 时还原的数据库的最大大小大于具有目标数据库的服务层和性能级别附送的存储量。 超过已包含数量预配的额外存储额外收费。 有关额外存储定价的详细信息，请参阅 [SQL 数据库定价页面](https://azure.microsoft.com/pricing/details/sql-database/)。 如果实际的已用空间量小于附送的存储量，可以避免此项额外费用的最大数据库大小设置为所提供的数量。

> [!NOTE]
> 在创建[数据库副本](sql-database-copy.md)时，将用到[自动数据库备份](sql-database-automated-backups.md)。

## <a name="recovery-time"></a>恢复时间

使用自动数据库备份还原数据库的恢复时间受几个因素的影响：

- 数据库的大小
- 数据库的计算大小
- 所涉及的事务日志数
- 需要重新播放以恢复到还原点的活动数量
- 还原到不同区域时的网络带宽
- 目标区域中正在处理的并行还原请求数

对于大型和/或非常活跃的数据库，还原可能要花费几个小时。 如果一个区域出现长时间的服务中断，则可能是因为存在大量正在由其他区域处理的异地还原请求。 当存在很多请求时，则可能会延长该区域中数据库的恢复时间。 大部分数据库还原操作可在 12 小时内完成。

对于单个订阅，有的并发还原请求数的限制。  这些限制适用于任何组合中时还原、 异地还原和恢复点从长期保留备份）：

| | **处理的并发请求数最多为 #** | **提交的并发请求数最多为 #** |
| :--- | --: | --: |
|单个数据库（每个订阅）|10|60|
|弹性池（每个池）|4|200|
||||

当前没有用于还原整个服务器的内置方法。 [Azure SQL 数据库：完整的服务器恢复](https://gallery.technet.microsoft.com/Azure-SQL-Database-Full-82941666)脚本是如何完成此任务的示例。

> [!IMPORTANT]
> 若要使用自动备份进行恢复，用户必须是订阅中的 SQL Server 参与者角色的成员或是订阅所有者 - 请参阅 [RBAC：内置角色](../role-based-access-control/built-in-roles.md)。 可以使用 Azure 门户、PowerShell 或 REST API 进行恢复。 但不能使用 Transact-SQL。

## <a name="point-in-time-restore"></a>时间点还原

可以还原独立，汇集在一起，或者使用 Azure 门户中，实例到以前的点的数据库[PowerShell](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase)，或[REST API](https://docs.microsoft.com/rest/api/sql/databases)。 请求可以指定任何服务层或计算还原的数据库的大小。 确保要将数据库还原到其中的服务器上有足够的资源。 完成后，将在与原始数据库相同的服务器上创建新的数据库。 还原的数据库将根据服务层和计算实例大小的标准费率收费。 在数据库还原完成之前，不会产生费用。

为了恢复目的，通常会将数据库还原到一个较早的点。 可以以替换原始数据库将还原的数据库，也可以将它用作源数据来更新原始数据库。

- **数据库替换**

  如果还原的数据库旨在替换原始数据库，则应指定 orinal 数据库的计算大小和服务层级。 然后可以重命名原始数据库，并为还原的数据库使用原始名称[ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) T-SQL 命令。

- **数据恢复**

  如果你打算从还原的数据库，若要从用户或应用程序错误中恢复中检索数据，您需要编写和执行数据恢复脚本从还原的数据库中提取数据并将应用到原始数据库。 尽管还原操作可能需要很长时间才能完成，但整个还原过程中，都可在数据库列表中看到还原数据库。 如果在还原期间删除数据库，将取消还原操作，并且，将不收取未完成还原的数据库。

要使用 Azure 门户将单一数据库、共用数据库或实例数据库恢复到某个时间点，请打开该数据库的页面，并在工具栏上单击“还原”。

![时间点还原](./media/sql-database-recovery-using-backups/point-in-time-recovery.png)

> [!IMPORTANT]
> 若要以编程方式从备份还原数据库，请参阅[使用自动备份以编程方式执行恢复](sql-database-recovery-using-backups.md#programmatically-performing-recovery-using-automated-backups)

## <a name="deleted-database-restore"></a>已删除的数据库还原

可以使用 Azure 门户中，在同一个 SQL 数据库服务器上的时间已删除的数据库还原到删除时间或早的时间点[PowerShell](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase)，或[REST (createMode = Restore)](https://docs.microsoft.com/rest/api/sql/databases/createorupdate)。 可以[使用 PowerShell 在托管实例上还原已删除的数据库](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../recreate-dropped-database-on-azure-sql-managed-instance)。 

> [!TIP]
> 有关展示了如何还原已删除数据库的示例 PowerShell 脚本，请参阅[使用 PowerShell 还原 SQL 数据库](scripts/sql-database-restore-database-powershell.md)。
> [!IMPORTANT]
> 如果删除 Azure SQL 数据库服务器实例，其所有数据库也会一并删除，并且无法恢复。 目前不支持还原已删除的服务器。

### <a name="deleted-database-restore-using-the-azure-portal"></a>使用 Azure 门户还原已删除的数据库

若要恢复已删除的数据库使用 Azure 门户中，打开的页面，为您的服务器并在操作区中，单击**已删除的数据库**。

![deleted-database-restore-1](./media/sql-database-recovery-using-backups/deleted-database-restore-1.png)

![deleted-database-restore-2](./media/sql-database-recovery-using-backups/deleted-database-restore-2.png)

> [!IMPORTANT]
> 若要以编程方式还原已删除的数据库，请参阅[使用自动备份以编程方式执行恢复](sql-database-recovery-using-backups.md#programmatically-performing-recovery-using-automated-backups)

## <a name="geo-restore"></a>异地还原

可在任何 Azure 区域的任何服务器上从最新异地复制的备份中还原 SQL 数据库。 异地还原使用异地复制备份作为其源。 即使由于停电而无法访问数据库或数据中心也可以请求它。

如果你的数据库不可用的原因是托管的区域发生事故，异地还原是默认的恢复选项。 您可以将数据库还原到其他区域中的服务器。 创建备份后，将其异地复制到其他区域中的 Azure Blob 时会出现延迟。 因此，还原的数据库可以是最多一小时滞后于原始数据库。 下图显示的是从其他区域中的最后一个可用备份中还原数据库。

![异地还原](./media/sql-database-geo-restore/geo-restore-2.png)

> [!TIP]
> 有关展示了如何执行异地还原的示例 PowerShell 脚本，请参阅[使用 PowerShell 还原 SQL 数据库](scripts/sql-database-restore-database-powershell.md)。

当前，异地辅助数据库上不支持时间点还原。 仅主数据库支持时间点还原。 有关使用异地还原在中断后恢复的详细信息，请参阅[在中断后恢复](sql-database-disaster-recovery.md)。

> [!IMPORTANT]
> 异地还原是 SQL 数据库中可用的最基本灾难恢复解决方案。 它依赖于自动使用 RPO 创建异地复制的备份 = 1 小时和 12 小时的估计的恢复时间。 它不保证在目标区域，将具有区域 ourage 后还原的数据库，因为将可能急剧增加的需求的能力。 对于非业务关键应用程序使用相对较小的数据库，异地还原是适当的灾难恢复解决方案。 对于 busniess 关键应用程序使用大型数据库，必须确保业务连续性，应使用[自动故障转移组](sql-database-auto-failover-group.md)。 它提供低得多的 RPO 和 RTO，并且始终保证容量。 有关业务连续性选项的详细信息，请参阅[业务连续性概述](sql-database-business-continuity.md)。

### <a name="geo-restore-using-the-azure-portal"></a>使用 Azure 门户进行异地还原

要在其[基于 DTU 的模型保留期](sql-database-service-tiers-dtu.md)或[基于 vCore 的模型保留期](sql-database-service-tiers-vcore.md)期间使用 Azure 门户对数据库进行异地还原，请打开 SQL 数据库页，并单击“添加”。 在“选择源”文本框中，选择“备份”。 指定在选择的区域和服务器上执行还原的备份。

> [!Note]
> 异地还原使用 Azure 门户不可用在托管实例中。 请改为使用 PowerShell。

## <a name="programmatically-performing-recovery-using-automated-backups"></a>使用自动备份以编程方式执行恢复

如前所述，除了使用 Azure 门户外，还可以使用 Azure PowerShell 或 REST API 以编程方式执行数据库恢复。 下表描述了可用的命令集。

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> PowerShell Azure 资源管理器模块仍受 Azure SQL 数据库的支持，但所有未来的开发都是针对 Az.Sql 模块的。 若要了解这些 cmdlet，请参阅 [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/)。 Az 模块和 AzureRm 模块中的命令参数大体上是相同的。

- 若要还原的独立或共用的数据库，请参阅[还原 AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase)。

  | Cmdlet | 描述 |
  | --- | --- |
  | [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) |获取一个或多个数据库。 |
  | [Get-AzSqlDeletedDatabaseBackup](/powershell/module/az.sql/get-azsqldeleteddatabasebackup) | 获取可以还原的已删除数据库。 |
  | [Get-AzSqlDatabaseGeoBackup](/powershell/module/az.sql/get-azsqldatabasegeobackup) |获取数据库的异地冗余备份。 |
  | [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase) |还原 SQL 数据库。 |

  > [!TIP]
  > 有关展示了如何执行数据库的时间点还原的示例 PowerShell 脚本，请参阅[使用 PowerShell 还原 SQL 数据库](scripts/sql-database-restore-database-powershell.md)。

- 若要还原托管实例的数据库，请参阅[还原 AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase)。

  | Cmdlet | 描述 |
  | --- | --- |
  | [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance) |获取一个或多个托管的实例。 |
  | [Get-AzSqlInstanceDatabase](/powershell/module/az.sql/get-azsqlinstancedatabase) | 获取实例的数据库。 |
  | [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase) |将实例数据库还原。 |

### <a name="rest-api"></a>REST API

使用 REST API 还原单一数据库或共用数据库：

| API | 描述 |
| --- | --- |
| [REST (createMode=Recovery)](https://docs.microsoft.com/rest/api/sql/databases) |还原数据库 |
| [获取创建或更新数据库状态](https://docs.microsoft.com/rest/api/sql/operations) |在还原操作过程中返回状态 |

### <a name="azure-cli"></a>Azure CLI

- 若要使用 Azure CLI 还原单一数据库或共用数据库，请参阅 [az sql db restore](/cli/azure/sql/db#az-sql-db-restore)。
- 若要还原使用 Azure CLI 的托管的实例，请参阅[az sql midb 还原](/cli/azure/sql/midb#az-sql-midb-restore)

## <a name="summary"></a>摘要

自动备份可保护数据库，使其免受用户和应用程序错误、意外的数据库删除和长时间中断的影响。 此内置的功能适用于所有服务层级和计算大小。

## <a name="next-steps"></a>后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](sql-database-business-continuity.md)。
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](sql-database-automated-backups.md)。
- 若要了解长期保留，请参阅[长期保留](sql-database-long-term-retention.md)。
- 若要了解更快的恢复选项，请参阅[活动异地复制](sql-database-active-geo-replication.md)或[自动故障转移组](sql-database-auto-failover-group.md)。
