---
title: 缩放单一数据库资源 - Azure SQL 数据库 | Microsoft Docs
description: 本文介绍如何在 Azure SQL 数据库中缩放适用于单一数据库的计算和存储资源。
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
manager: craigg
ms.date: 04/26/2019
ms.openlocfilehash: 1048b4e2ac3a8523d5539ddc1a1bdaca3ec2d912
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074264"
---
# <a name="scale-single-database-resources-in-azure-sql-database"></a>在 Azure SQL 数据库中缩放单一数据库资源

本文介绍如何在预配的计算层中缩放计算和存储资源可用于单一数据库。 或者，[无服务器 （预览版） 计算层](sql-database-serverless.md)提供自动缩放计算和每秒使用的计算费用。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> PowerShell Azure 资源管理器模块仍受 Azure SQL 数据库的支持，但所有未来的开发都是针对 Az.Sql 模块的。 若要了解这些 cmdlet，请参阅 [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/)。 Az 模块和 AzureRm 模块中的命令参数大体上是相同的。

## <a name="change-compute-size-vcores-or-dtus"></a>更改计算大小 （Vcore 或 Dtu）

最初选择 vCore 或 DTU 数量后，可以使用 [Azure 门户](sql-database-single-databases-manage.md#manage-an-existing-sql-database-server)、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、 [PowerShell](/powershell/module/az.sql/set-azsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](https://docs.microsoft.com/rest/api/sql/databases/update)，根据实际体验动态扩展或缩减单一数据库。

下面的视频演示了如何动态更改服务层级和计算大小以增加单一数据库的可用 DTU。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-dynamically-scale-up-or-scale-down/player]
>

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](sql-database-file-space-management.md)。

### <a name="impact-of-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小的影响

更改单一数据库的服务层级或计算大小主要涉及到由服务执行的以下步骤：

1. 为数据库创建新的计算实例  

    使用请求的服务层级和计算大小为数据库创建新的计算实例。 更改后，对于服务层级和计算大小的某些组合，必须在新的计算实例中创建数据库的副本，此过程涉及到数据复制，可能会对总体延迟造成很大的影响。 无论如何，在执行此步骤期间，数据库会保持联机，并且连接会继续定向到原始计算实例中的数据库。

2. 将连接路由切换到新的计算实例

    将删除与原始计算实例中的数据库建立的现有连接。 将与新计算实例中的数据库建立任何新的连接。 更改后，对于服务层级和计算大小的某些组合，在切换期间会分离再重新附加数据库文件。  无论如何，切换操作都可能会导致服务出现短暂的中断，此时，数据库一般会出现 30 秒以下的不可用情况（通常只有几秒钟）。 如果连接断开时有长时间运行的事务正在运行，则此步骤的持续时间可能会变长，以便恢复中止的事务。 [加速数据库恢复](sql-database-accelerated-database-recovery.md)可以减少从中止长时间运行的事务的影响。

> [!IMPORTANT]
> 执行工作流中的任何步骤期间都不会丢失数据。

### <a name="latency-of-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小所造成的延迟

可根据如下所述，将更改服务层级或者重新缩放单一数据库或弹性池的计算大小所造成的延迟参数化：

|服务层|基本单一数据库，</br>标准 (S0-S1)|基本弹性池，</br>标准 (S2-S12)， </br>超大规模， </br>常规用途单一数据库或弹性池|高级或业务关键型单一数据库或弹性池|
|:---|:---|:---|:---|
|**基本单一数据库，</br>标准 (S0-S1)**|&bull; &nbsp;延迟时间较为恒定，与已用空间无关</br>&bull; &nbsp;通常小于 5 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|
|**基本弹性池</br>标准 (S2-S12)</br>超大规模，</br>常规用途单一数据库或弹性池**|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;延迟时间较为恒定，与已用空间无关</br>&bull; &nbsp;通常小于 5 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|
|**高级或业务关键型单一数据库或弹性池**|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|

> [!TIP]
> 要监视进行中的操作，请参阅：[使用 SQL REST API 管理操作](https://docs.microsoft.com/rest/api/sql/operations/list)、[使用 CLI 管理操作](/cli/azure/sql/db/op)、[使用 T-SQL 监视操作](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database)以及这两个 PowerShell 命令：[Get-AzSqlDatabaseActivity](/powershell/module/az.sql/get-azsqldatabaseactivity) 和 [Stop-AzSqlDatabaseActivity](/powershell/module/az.sql/stop-azsqldatabaseactivity)。

### <a name="cancelling-service-tier-changes-or-compute-rescaling-operations"></a>正在取消服务层更改或重新缩放操作的计算

服务层更改或计算可以取消重新缩放操作。

#### <a name="azure-portal"></a>Azure 门户

在数据库概述边栏选项卡，导航到**通知**，然后单击磁贴，指示没有正在进行的操作：

![正在进行的操作](media/sql-database-single-database-scale/ongoing-operations.png)

接下来，单击标记为按钮**取消此操作**。

![取消正在进行的操作](media/sql-database-single-database-scale/cancel-ongoing-operation.png)

#### <a name="powershell"></a>PowerShell

从 PowerShell 命令提示符下设置`$ResourceGroupName`， `$ServerName`，和`$DatabaseName`，然后运行以下命令：

```PowerShell
$OperationName = (az sql db op list --resource-group $ResourceGroupName --server $ServerName --database $DatabaseName --query "[?state=='InProgress'].name" --out tsv)
if(-not [string]::IsNullOrEmpty($OperationName))
    {
        (az sql db op cancel --resource-group $ResourceGroupName --server $ServerName --database $DatabaseName --name $OperationName)
        "Operation " + $OperationName + " has been canceled"
    }
    else
    {
        "No service tier change or compute rescaling operation found"
    }
```

### <a name="additional-considerations-when-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小时的其他注意事项

- 如果要升级到更高的服务层级或计算大小，除非显式指定了更大的大小（最大），否则，最大数据库大小不会增大。
- 若要对数据库进行降级，数据库所用空间必须小于目标服务层级和计算大小允许的最大大小。
- 从高级层级降级至标准层级时，如果同时满足 (1) 目标计算大小支持该数据库的最大大小，(2) 最大大小超出目标计算大小包括的存储量，那么将产生额外存储费用。 例如，如果将最大大小为 500 GB 的 P1 数据库缩小至 S3，那么将产生额外的存储费用，因为 S3 支持的最大大小为 500 GB，而它的附送存储量仅为 250 GB。 因此，额外存储量为 500 GB – 250 GB = 250 GB。 有关额外存储定价的信息，请参阅 [SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/)。 如果实际使用的空间量小于附送的存储量，只要将数据库最大大小减少到附送的量，就能避免此项额外费用。
- 在启用了[异地复制](sql-database-geo-replication-portal.md)的情况下升级数据库时，请先将其辅助数据库升级到所需的服务层级和计算大小，然后再升级主数据库（用于实现最佳性能的常规指南）。 在升级到另一版本时，必须首先升级辅助数据库。
- 在启用了[异地复制](sql-database-geo-replication-portal.md)的情况下降级数据库时，请先将其主数据库降级到所需的服务层级和计算大小，然后再降级辅助数据库（用于实现最佳性能的常规指南）。 在降级到另一版本时，必须首先降级主数据库。
- 各服务层级的还原服务不同。 如果要降级到基本层，则备份保持期也将缩短。 请参阅 [Azure SQL 数据库备份](sql-database-automated-backups.md)。
- 所做的更改完成之前不会应用数据库的新属性。

### <a name="billing-during-compute-rescaling"></a>计费期间重新缩放计算

将根据使用最高服务层级的数据库存在的每个小时 + 在该小时适用的计算大小进行计费，无论使用方式或数据库处于活动状态是否少于一小时。 例如，如果创建了单一数据库，五分钟后删除了它，则将按该数据库存在一小时收费。

## <a name="change-storage-size"></a>更改存储大小

### <a name="vcore-based-purchasing-model"></a>基于 vCore 的购买模型

- 可以使用 1GB 作为增量，将存储预配到最大大小限制。 最小可配置数据存储为 5 GB
- 可通过 [Azure 门户](https://portal.azure.com)、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/az.sql/set-azsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](https://docs.microsoft.com/rest/api/sql/databases/update) 为单一数据库增加或减少大小上限，以预配存储。
- SQL 数据库会自动为日志文件额外分配 30% 的存储，并为 TempDB 的每个 vCore 分配 32GB，但不会超过 384GB。 TempDB 位于所有服务层级中的附加 SSD 上。
- 单一数据库的存储价格等于数据存储与日志存储量之和乘以服务层级的存储单价。 vCore 价格已包括 TempDB 费用。 有关额外存储价格的详细信息，请参阅 [SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](sql-database-file-space-management.md)。

### <a name="dtu-based-purchasing-model"></a>基于 DTU 的购买模型

- 单一数据库的 DTU 价格附送了一定容量的存储，无需额外费用。 超出附送的量后，可花费额外的费用预配额外的存储，但不能超过存储上限，不超过 1 TB 时，以 250 GB 为增量进行预配，超出 1 TB 时，以 256 GB 为增量进行预配。 有关包括的存储量和大小上限，请参阅[单一数据库：存储大小和计算大小](sql-database-dtu-resource-limits-single-databases.md#single-database-storage-sizes-and-compute-sizes)。
- 可通过 Azure 门户、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/az.sql/set-azsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](https://docs.microsoft.com/rest/api/sql/databases/update) 为单一数据库增加大小上限，以预配额外存储。
- 单一数据库的额外存储价格等于额外存储量乘以服务层级的额外存储单价。 有关额外存储价格的详细信息，请参阅 [SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](sql-database-file-space-management.md)。

## <a name="p11-and-p15-constraints-when-max-size-greater-than-1-tb"></a>P11 和 P15 约束时最大大小超过 1 TB

除以下区域外，其他所有区域的高级层目前均可提供超过 1 TB 的存储：中国东部、中国北部、德国中部、德国东北部、美国中西部、US DoD 区域和美国政府中部。 在这些区域，高级层中的最大存储限制为 1 TB。 对于最大大小超过 1 TB 的 P11 和 P15 数据库，存在以下注意事项和限制：

- 如果 P11 或 P15 数据库的最大大小曾经设置为一个值大于 1 TB，则它只能还原或复制到 P11 或 P15 数据库。  随后，提供在缩放操作时分配的空间量不超过新的计算大小的最大大小限制为不同的计算大小重新缩放数据库。
- 对于“活动异地复制”方案：
  - 设置异地复制关系：如果主数据库是 P11 或 P15，则辅助数据库也必须为 P11 或 P15，更低的计算大小会被拒绝作为辅助数据库，因为它们不能支持超过 1 TB。
  - 升级异地复制关系中的主数据库：在主数据库上将最大大小更改为超过 1 TB 将触发辅助数据库上的相同更改。 这两个升级都必须成功才能使主数据库上的更改生效。 超过 1 TB 选项的区域限制适用。 如果辅助数据库位于不支持超过 1 TB 的区域，则不会升级主数据库。
- 不支持将导入/导出服务用于加载超过 1 TB 的 P11/P15 数据库。 使用 SqlPackage.exe 可[导入](sql-database-import.md)和[导出](sql-database-export.md)数据。

## <a name="next-steps"></a>后续步骤

有关总体资源限制，请参阅 [SQL 数据库基于 vCore 的资源限制 - 单一数据库](sql-database-vcore-resource-limits-single-databases.md)和 [SQL 数据库基于 DTU 的资源限制 - 弹性池](sql-database-dtu-resource-limits-single-databases.md)。
