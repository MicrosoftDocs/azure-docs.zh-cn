---
title: Azure SQL 数据库服务 - vCore | Microsoft 文档
description: 使用基于 vCore 的购买模型，可以单独缩放计算和存储资源、匹配本地性能，以及优化价格。
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake, carlrab
manager: craigg
ms.date: 05/06/2019
ms.openlocfilehash: c85ee31a54cdbbb09686a2d20200f65fdcd8994a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235918"
---
# <a name="choose-among-the-vcore-service-tiers-and-migrate-from-dtu-service-tiers"></a>在 vCore 的服务层之间进行选择，并将 DTU 服务层从迁移

使用基于 vCore 的购买模型，可以单独缩放计算和存储资源、匹配本地性能，以及优化价格。 它还允许你选择硬件世代：

- 第 4 代 - 最多 24 个基于 Intel E5-2673 v3 (Haswell) 2.4 GHz 处理器的逻辑 CPU，vCore = 1 PP（物理核心），每核心 7 GB，附加了 SSD
- 第 5 代 - 最多 80 个基于 Intel E5-2673 v4 (Broadwell) 2.3 GHz 处理器的逻辑 CPU，vCore=1 LP（超线程），每个核心 5.1 GB，快速 eNVM SSD


第 4 代为每个 vCore 提供的内存要大得多。 但是，第 5 代硬件允许以高得多的力度纵向扩展计算资源。

> [!NOTE]
> 有关基于 DTU 的服务层级的信息，请参阅[基于 DTU 的服务层级](sql-database-service-tiers-dtu.md)。 有关区分基于 DTU 的服务层级和基于 vCore 的服务层级的信息，请参阅 [Azure SQL 数据库购买模型](sql-database-purchase-models.md)。

## <a name="service-tier-characteristics"></a>服务层特征

vCore 模型提供了三个服务层级：“常规用途”、“业务关键”和“超大规模”。 服务层级根据一系列计算大小、高可用性设计、故障隔离、存储类型和大小以及 IO 范围进行区分。 必须单独配置所需的存储和备份保持期。 在 Azure 门户中，转到“服务器”（而不是数据库）>“托管备份”>“配置策略”>“时间点还原配置”>“7 - 35 天”。

下表可帮助你了解这三个层之间的差别：

||**常规用途**|**业务关键**|**Hyperscale**|
|---|---|---|---|
|最适用于|大多数业务工作负荷。 提供预算导向的、均衡且可缩放的计算和存储选项。|IO 要求高的业务应用程序。 使用多个独立副本，提供最高级别的故障恢复能力。|具有很高的可缩放存储和读取缩放要求的大多数业务工作负荷|
|计算|**预配计算**:<br/>Gen4：1 到 24 个 vCore<br/>Gen5：2 到 80 vCore<br/>**无服务器计算**<br/>Gen5：0.5 - 4 vCore|**预配计算**:<br/>Gen4：1 到 24 个 vCore<br/>Gen5：2 到 80 vCore|**预配计算**:<br/>Gen4：1 到 24 个 vCore<br/>Gen5：2 到 80 vCore|
|内存|**预配计算**:<br/>Gen4：每个 vCore 7 GB<br/>Gen5：每个 vCore 5.1 GB<br/>**无服务器计算**<br/>Gen5：每个 vCore 3 GB|**预配计算**:<br/>Gen4：每个 vCore 7 GB<br/>Gen5：每个 vCore 5.1 GB |**预配计算**:<br/>Gen4：每个 vCore 7 GB<br/>Gen5：每个 vCore 5.1 GB|
|存储|使用远程存储：<br/>**单个数据库预配计算**:<br/>5 GB – 4 TB<br/>**单个数据库无服务器计算**:<br/>5 GB - 1 TB<br/>**托管实例**:32 GB - 8 TB |使用本地 SSD 存储：<br/>**单个数据库预配计算**:<br/>5 GB – 4 TB<br/>**托管实例**:<br/>32 GB - 4 TB |可以根据需要灵活地自动扩展存储。 支持高达 100 TB 存储及更多存储。 使用本地 SSD 存储作为本地缓冲池缓存和本地数据存储。 使用 Azure 远程存储作为最终的长期数据存储。 |
|内存|Gen4：每个核心 7 GB<br>Gen5：每个核心 5.1 GB | Gen4：每个核心 7 GB<br>Gen5：每个核心 5.1 GB |Gen4：每个核心 7 GB<br>Gen5：每个核心 5.1 GB|
|存储|使用远程存储：<br/>单一数据库：5 GB – 4 TB<br/>托管实例：32 GB - 8 TB |使用本地 SSD 存储：<br/>单一数据库：5 GB – 4 TB<br/>托管实例：32 GB - 4 TB |可以根据需要灵活地自动扩展存储。 支持高达 100 TB 存储及更多存储。 使用本地 SSD 存储作为本地缓冲池缓存和本地数据存储。 使用 Azure 远程存储作为最终的长期数据存储。 |
|IO 吞吐量（近似）|单一数据库：每个 vCore 提供 500 IOPS，最大 7000 IOPS</br>托管实例：取决于[文件大小](../virtual-machines/windows/premium-storage-performance.md#premium-storage-disk-sizes)|每个 vCore 提供 5000 IOPS，最大 200,000 IOPS|TBD|
|可用性|1 个副本，无读取缩放组|3 个副本，1 个[读取缩放副本](sql-database-read-scale-out.md)，<br/>区域冗余 HA|1 的读/写副本以及 0 到 4[读取缩放副本](sql-database-read-scale-out.md)|
|备份|[RA-GRS](../storage/common/storage-designing-ha-apps-with-ragrs.md)，7-35 天（默认为 7 天）|[RA-GRS](../storage/common/storage-designing-ha-apps-with-ragrs.md)，7-35 天（默认为 7 天）|Azure 远程存储中基于快照的备份和还原使用这些快照进行快速恢复。 备份瞬间完成，不会影响计算的 IO 性能。 还原速度非常快，不基于数据操作的大小（需要几分钟，而不是几小时或几天）。|
|内存中|不支持|支持|不支持|
|||

> [!NOTE]
> 在配合 Azure 免费帐户探索 Azure 中的基本服务层，可以获取免费的 Azure SQL 数据库。 有关信息，请参阅[使用 Azure 免费帐户创建托管的云数据库](https://azure.microsoft.com/free/services/sql-database/)。

- 有关详细信息，请参阅[单一数据库中的 vCore 资源限制](sql-database-vcore-resource-limits-single-databases.md)和[托管实例中的 vCore 资源限制](sql-database-managed-instance.md#vcore-based-purchasing-model)。
- 若要详细了解“常规用途”服务层级和“业务关键”服务层级，请参阅[“常规用途”服务层级和“业务关键”服务层级](sql-database-service-tiers-general-purpose-business-critical.md)。
- 若要详细了解基于 vCore 的购买模型中的超大规模服务层级，请参阅[超大规模服务层级](sql-database-service-tier-hyperscale.md)。  

## <a name="azure-hybrid-benefit"></a>Azure 混合权益

可以在基于 vCore 的购买模型设置的计算机层中，交换现有许可证，有关使用 SQL 数据库的折扣价格[面向 SQL Server 的 Azure 混合权益](https://azure.microsoft.com/pricing/hybrid-benefit/)。 借助这项 Azure 权益，可以使用附带软件保障的本地 SQL Server 许可证，将 Azure SQL 数据库的成本最多节省 30%。

![定价](./media/sql-database-service-tiers/pricing.png)

使用 Azure 混合权益，你可以选择仅为底层 Azure 基础结构付费且将现有的 SQL Server 许可证用于 SQL 数据库引擎自身 (**BasePrice**)，也可以选择同时为底层基础结构和 SQL Server 许可证付费 (**LicenseIncluded**)。


可以使用 Azure 门户或下列 API 之一来选择或更改许可模型。

- 使用 PowerShell 设置或更新许可证类型：

  - [New-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/new-azsqldatabase)：
  - [Set-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql)
  - [New-AzSqlInstance](https://docs.microsoft.com/powershell/module/az.sql/new-azsqlinstance)
  - [Set-AzSqlInstance](https://docs.microsoft.com/powershell/module/az.sql)

- 使用 Azure CLI 设置或更新许可证类型：

  - [az sql db create](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-create)
  - [az sql db update](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-update)
  - [az sql mi create](https://docs.microsoft.com/cli/azure/sql/mi#az-sql-mi-create)
  - [az sql mi update](https://docs.microsoft.com/cli/azure/sql/mi#az-sql-mi-update)

- 使用 REST API 设置或更新许可证类型：

  - [数据库 - 创建或更新](https://docs.microsoft.com/rest/api/sql/databases/createorupdate)
  - [数据库 - 更新](https://docs.microsoft.com/rest/api/sql/databases/update)
  - [托管实例 - 创建或更新](https://docs.microsoft.com/rest/api/sql/managedinstances/createorupdate)
  - [托管实例 - 更新](https://docs.microsoft.com/rest/api/sql/managedinstances/update)

## <a name="migration-from-dtu-model-to-vcore-model"></a>从 DTU 模型迁移到 vCore 模型

### <a name="migration-of-a-database"></a>迁移数据库

将数据库从基于 DTU 的购买模型迁移到基于 vCore 的购买模型，与在基于 DTU 的购买模型中的标准数据库和高级数据库之间进行升级或降级的操作类似。

### <a name="migration-of-databases-with-geo-replication-links"></a>使用异地复制链接迁移数据库

从基于 DTU 的模型迁移到基于 vCore 的模型类似于在标准和高级数据库之间升级或降级异地复制关系。 这不需要终止异地复制，但用户必须遵守顺序规则。 升级时，必须先升级辅助数据库，再升级主数据库。 降级时，必须反转顺序：先降级主数据库，再降级辅助数据库。

在两个弹性池之间使用异地复制时，建议将一个池指定为主池，将另一个池指定为辅助池。 在这种情况下，迁移弹性池应遵循相同的指导原则。  但是，从技术上讲，弹性池可能包含主数据库和辅助数据库。 在这种情况下，若要正确迁移，应将利用率较高的池视为主池，并相应地遵循顺序规则。  

下表提供具体迁移场景的指导：

|当前服务层级|目标服务层级|迁移类型|用户操作|
|---|---|---|---|
|标准|常规用途|横向|可按任意顺序迁移，但需确保 vCore 大小适当*|
|高级|业务关键|横向|可按任意顺序迁移，但需确保 vCore 大小适当*|
|标准|业务关键|升级|必须先迁移辅助数据库|
|业务关键|标准|降级|必须先迁移主数据库|
|高级|常规用途|降级|必须先迁移主数据库|
|常规用途|高级|升级|必须先迁移辅助数据库|
|业务关键|常规用途|降级|必须先迁移主数据库|
|常规用途|业务关键|升级|必须先迁移辅助数据库|
||||

\* 标准层中的每 100 个 DTU 至少需要 1 个 vCore，高级层中的每 125 个 DTU 至少需要 1 个 vCore

### <a name="migration-of-failover-groups"></a>迁移故障转移组

迁移包含多个数据库的故障转移组需要单独迁移主数据库和辅助数据库。 在此过程中，请遵循相同的注意事项和顺序规则。 将数据库转换到基于 vCore 的模型后，故障转移组将保持有效并使用相同的策略设置。

### <a name="creation-of-a-geo-replication-secondary"></a>创建异地复制辅助数据库

只能使用与主数据库相同的服务层级来创建异地辅助数据库。 对于日志生成速率较高的数据库，建议使用与主数据库相同的计算大小来创建辅助数据库。 如果在弹性池中为单个主数据库创建异地辅助数据库，建议对该池使用与主数据库计算大小匹配的 `maxVCore` 设置。 如果在弹性池中为另一个弹性池的主数据库创建异地辅助数据库，建议对该池使用相同的 `maxVCore` 设置

### <a name="using-database-copy-to-convert-a-dtu-based-database-to-a-vcore-based-database"></a>使用数据库复制将基于 DTU 的数据库转换为基于 vCore 的数据库

可将采用基于 DTU 的计算大小的任何数据库复制到采用基于 vCore 的计算大小的数据库，且无需遵守上述限制或特殊的顺序，前提是目标计算大小支持源数据库的最大数据库大小。 数据库复制会在复制操作启动时创建数据快照，且不会在源数据库与目标数据库之间执行数据同步。

## <a name="next-steps"></a>后续步骤

- 有关适用于单一数据库的特定计算大小和存储大小选项的详细信息，请参阅[适用于单一数据库的 SQL 数据库基于 vCore 的资源限制](sql-database-vcore-resource-limits-single-databases.md)。
- 若要详细了解适用于弹性池的特定计算大小和存储大小选项，请参阅[适用于弹性池的 SQL 数据库基于 vCore 的资源限制](sql-database-vcore-resource-limits-elastic-pools.md#general-purpose-service-tier-storage-sizes-and-compute-sizes)。
