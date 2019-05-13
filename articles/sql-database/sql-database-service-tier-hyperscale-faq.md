---
title: 超大规模 Azure SQL 数据库常见问题解答 | Microsoft Docs
description: 对客户关于“超大规模”服务层级中的 Azure SQL 数据库（通常称为“超大规模”数据库）提出的常见问题的回答。
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 05/06/2019
ms.openlocfilehash: 4f0691dcc8a0c4b0ebbfe26a49b75b86fa50f597
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236017"
---
# <a name="faq-about-azure-sql-hyperscale-databases"></a>关于 Azure SQL“超大规模”数据库的常见问题解答

本文提供的客户考虑在 Azure SQL 数据库超大规模服务层中，通常称为超大规模数据库的数据库产品的常见问题的解答。 本文介绍“超大规模”服务层级支持的方案，并且跨功能服务通常与 SQL 数据库“超大规模”服务层级兼容。

- 本常见问题解答适用于对“超大规模”服务层级有基本了解并希望其具体问题得到解答的读者。
- 它不是指南，不解答关于如何使用 SQL 数据库“超大规模”数据库的问题。 为此，建议参考[超大规模 Azure SQL 数据库](sql-database-service-tier-hyperscale.md)文档。

## <a name="general-questions"></a>一般问题

### <a name="what-is-a-hyperscale-database"></a>什么是“超大规模”数据库

“超大规模”数据库是“超大规模”服务层级中的 Azure SQL 数据库，由超大规模横向扩展存储技术提供支持。 一个“超大规模”数据库支持最多 100 TB 的数据，提供高吞吐量和高性能，可以快速缩放以适应工作负荷要求。 缩放对应用程序透明，在连接、查询处理等方面的工作方式与其他任何 SQL 数据库一样。

### <a name="what-resource-types-and-purchasing-models-support-hyperscale"></a>哪些资源类型和购买模型支持“超大规模”

“超大规模”服务层级仅适用于在 Azure SQL 数据库中使用基于 vCore 的购买模型的单一数据库。  

### <a name="how-does-the-hyperscale-service-tier-differ-from-the-general-purpose-and-business-critical-service-tiers"></a>“超大规模”服务层与“常规用途”和“业务关键”服务层有何区别

主要根据可用性、存储类型和 IOP 区分基于 vCore 的服务层。

- 常规用途服务层提供一组均衡的计算和存储选项（其中 IO 延迟或故障转移时间无需优先考虑），适用于大多数业务工作负荷。
- “超大规模”服务层级针对非常大的数据库工作负荷进行了优化。
- 业务关键服务层适用于需优先考虑 IO 延迟的业务工作负荷。

| | 资源类型 | 常规用途 |  超大规模 | 业务关键 |
|:---|:---:|:---:|:---:|:---:|:---:|
| **最适用于** |全部|  大多数业务工作负荷。 提供以预算导向的、均衡的计算和存储选项。 | 数据容量要求高且能够流畅地自动缩放存储和流畅缩放计算的数据应用程序。 | 事务率较高、延迟 IO 最低的 OLTP 应用程序。 使用多个独立副本，提供最高级别的故障恢复能力。|
|  **资源类型** ||单一数据库/弹性池/托管实例 | 单个数据库 | 单一数据库/弹性池/托管实例 |
| **计算大小**|单一数据库/弹性池* | 1 - 80 个 vCore | 1 - 80 个 vCore* | 1 - 80 个 vCore |
| |托管实例 | 8、16、24、32、40、64、80 个 vCore | 不适用 | 8、16、24、32、40、64、80 个 vCore |
| **存储类型** | 全部 |高级远程存储（每个实例） | 具有本地 SSD 缓存的分离的存储（每个实例） | 超快的本地 SSD 存储（每个实例） |
| **存储大小** | 单一数据库/弹性池 | 5 GB – 4 TB | 最多 100 TB | 5 GB – 4 TB |
| | 托管实例  | 32 GB – 8 TB | 不适用 | 32 GB – 4 TB |
| **IO 吞吐量** | 单一数据库** | 每个 vCore 提供 500 IOPS，最大 7000 IOPS | 未知 | 5000 IOPS，最大 200,000 IOPS|
| | 托管实例 | 取决于文件大小 | 不适用 | 托管实例：取决于文件大小|
|**可用性**|全部|1 个副本，无读取扩展副本，无本地缓存 | 多个副本，最多 15 个读取扩展副本，部分本地缓存 | 3 个副本，1 个读取扩展副本，区域冗余 HA，完整的本地缓存 |
|**备份**|全部|RA-GRS，7-35 天（默认为 7 天）| RA-GRS 的过去 7-35 天 （默认情况下为 7 天），常量时间内时间点恢复 (PITR) | RA-GRS，7-35 天（默认为 7 天） |

\* 超大规模服务层中不支持弹性池

### <a name="who-should-use-the-hyperscale-service-tier"></a>哪些群体应使用“超大规模”服务层级

“超大规模”服务层级主要面向拥有大型本地 SQL Server 数据库并希望通过迁移到云来实现应用程序现代化的客户，或已使用 Azure SQL 数据库并且想大大拓展数据库增长可能性的客户。 “超大规模”服务层级也适用于那些寻求高性能和高可伸缩性的客户。 使用“超大规模”服务层级，可以：

- 支持高达 100 TB 的数据库大小
- 快速进行数据库备份，无需考虑数据库大小（备份基于文件快照）
- 快速进行数据库还原，无需考虑数据库大小（从文件快照还原）
- 无论数据库大小如何，提高日志吞吐量，导致事务提交时间快
- 使读取横向扩展到一个或多个只读节点，以卸载读取工作负荷，并用于热备用服务器。
- 在恒定时间内快速纵向扩展计算，以便提高适应繁重工作负荷的能力；然后在恒定时间内减少。 例如，这与在 P6 到 P11 之间来回缩放类似，但速度更快，因为这不是一种数据大小操作。

### <a name="what-regions-currently-support-hyperscale"></a>哪些区域当前支持“超大规模”

Azure SQL 数据库超大规模层目前已在下列出的区域[Azure SQL 数据库的超大规模概述](sql-database-service-tier-hyperscale.md#regions)。

### <a name="can-i-create-multiple-hyperscale-databases-per-logical-server"></a>能否为每个逻辑服务器创建多个“超大规模”数据库

可以。 有关每个逻辑服务器的“超大规模”数据库数量的详细信息和限制，请参阅[逻辑服务器上单一和共用数据库的 SQL 数据库资源限制](sql-database-resource-limits-logical-server.md)。

### <a name="what-are-the-performance-characteristic-of-a-hyperscale-database"></a>“超大规模”数据库的性能特征有哪些

SQL 数据库“超大规模”服务层级体系结构提供高性能和吞吐量，同时支持大型数据库。 

### <a name="what-is-the-scalability-of-a-hyperscale-database"></a>什么是“超大规模”数据库的可伸缩性

SQL 数据库“超大规模”服务层级根据工作负荷需求，提供快速的可伸缩性。

- **增加/减少**

  使用“超大规模”数据库，可以纵向扩展 CPU、内存等资源的主要计算大小，然后在恒定的时间内将其减少。 因为是共享存储，所以纵向扩展和缩减不是数据大小操作。  
- **缩小/扩大**

  借助“超大规模”数据库，还能够预配一个或多个额外的计算节点，用于为读取请求提供服务。 这意味着，可以将这些额外的计算节点用作只读节点，从主计算卸载读取工作负荷。 除用作只读节点外，在主计算发生故障转移时，它们还充当热备用节点。

  对每个额外的计算节点的预配可在恒定时间内完成，并且是联机操作。 将连接字符串的 `ApplicationIntent` 参数设置为 `read_only`，即可连接到这些额外的只读计算节点。 任何标记为 `read-only` 的连接均自动路由到某个额外的只读计算节点。

## <a name="deep-dive-questions"></a>深入的问题

### <a name="can-i-mix-hyperscale-and-single-databases-in-a-single-logical-server"></a>可以混合使用超大规模和一台逻辑服务器中的单个数据库

是的，你可以。

### <a name="does-hyperscale-require-my-application-programming-model-to-change"></a>“超大规模”数据库需要更改应用程序编程模型吗

不，应用程序编程模型保持原样。 可以照常使用连接字符串和其他常规模式，以与 Azure SQL 数据库进行交互。

### <a name="what-transaction-isolation-levels-are-going-to-be-default-on-sql-database-hyperscale-database"></a>SQL 数据库“超大规模”数据库上的默认事务隔离级别是什么

在主节点上，事务隔离级别为 RCSI（已提交读快照隔离）。 在读取扩展辅助节点上，隔离级别为“快照”。

### <a name="can-i-bring-my-on-premises-or-iaas-sql-server-license-to-sql-database-hyperscale"></a>能否将我的本地或 IaaS SQL Server 许可证引入 SQL 数据库“超大规模”服务层级

能，[Azure 混合权益](https://azure.microsoft.com/pricing/hybrid-benefit/)适用于“超大规模”。 每个 SQL Server Standard 核心都可以映射到 1 个超大规模 vCore。 每个 SQL Server Enterprise 核心都可以映射到 4 个“超大规模”vCore。 对于次要副本，不需要 SQL 许可证。 Azure 混合权益价格会自动应用到读取扩展（次要）副本。

### <a name="what-kind-of-workloads-is-sql-database-hyperscale-designed-for"></a>哪种工作负荷专为 SQL 数据库“超大规模”服务层级设计

SQL 数据库“超大规模”服务层级支持所有 SQL Server 工作负荷，但它主要针对 OLTP 进行了优化。 你可以将混合 (HTAP) 和分析 （数据市场） 工作负载。

### <a name="how-can-i-choose-between-azure-sql-data-warehouse-and-sql-database-hyperscale"></a>如何在 Azure SQL 数据仓库和 SQL 数据库“超大规模”服务层级之间做出选择

如果目前运行的是使用 SQL Server 作为数据仓库的交互式分析查询，SQL 数据库“超大规模”服务层级是很好的选择，因为能以较低费用托管相对较小的数据仓库（例如几 TB 到数十 TB），并且可以在不更改 T-SQL 代码的情况下，将数据仓库工作负荷迁移到 SQL 数据库“超大规模”服务层级。

如果大规模运行包含复杂查询的数据分析并且使用并行数据仓库 (PDW)、Teradata 或其他大规模并行处理器 (MPP) 数据仓库，则 SQL 数据仓库可能是最佳选择。
  
## <a name="sql-database-hyperscale-compute-questions"></a>SQL 数据库“超大规模”服务层级计算问题

### <a name="can-i-pause-my-compute-at-any-time"></a>能不能随时暂停计算

不在此期间，但可以缩放计算和向下的副本数减少成本在非高峰时段。

### <a name="can-i-provision-a-compute-with-extra-ram-for-my-memory-intensive-workload"></a>能不能为内存密集型工作负荷预配包含额外 RAM 的计算

不。 要获取更多 RAM，需要升级到更大的计算大小。 有关详细信息，请参阅[超大规模存储和计算大小](sql-database-vcore-resource-limits-single-databases.md#hyperscale-service-tier)。

### <a name="can-i-provision-multiple-compute-nodes-of-different-sizes"></a>能不能预配大小不同的多个计算节点

不。

### <a name="how-many-read-scale-replicas-are-supported"></a>支持多少个读取扩展副本

默认情况下，可将超大规模数据库创建带有一个读取缩放副本 （总共两个副本）。 您可以介于 0 和 4 使用之间的只读副本数量进行缩放[Azure 门户](https://portal.azure.com)， [T-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current)， [Powershell](https://docs.microsoft.com/powershell/module/azurerm.sql/set-azurermsqldatabase)或者[CLI](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-update)...

### <a name="for-high-availability-do-i-need-to-provision-additional-compute-nodes"></a>若要实现高可用性，是否需要预配额外的计算节点

在超大规模数据库中，在存储级别提供复原能力。 您只需要一个副本，以提供复原能力。 计算副本关闭后，系统自动创建一个新副本，且不会丢失数据。

但是，如何只有一个副本，故障转移后可能需要一些时间才能在新副本中生成本地缓存。 在缓存重新生成阶段，数据库直接从页服务器中提取数据，导致 IOPS 和查询性能下降。

对于需要高可用性的任务关键应用，应该预配至少 2 个计算节点（默认包括主计算节点）。 这样，在发生故障转移时有热备用服务器可用。

## <a name="data-size-and-storage-questions"></a>数据大小和存储问题

### <a name="what-is-the-max-db-size-supported-with-sql-database-hyperscale"></a>SQL 数据库“超大规模”服务层级支持的最大数据库大小

100 TB

### <a name="what-is-the-size-of-the-transaction-log-with-hyperscale"></a>“超大规模”数据库事务日志的大小

“超大规模”数据库的事务日志几乎是无穷大的。 不需要担心在具有高日志吞吐量的系统上耗尽日志空间。 但是，可能为连续主动工作负荷限制日志生成速率。 峰值持续的日志生成速率是大约 100 MB/秒。

### <a name="does-my-temp-db-scale-as-my-database-grows"></a>临时数据库是否会随着数据库增长而缩放

`tempdb` 数据库位于本地 SSD 存储，是根据预配的计算大小配置的。 为提供最大性能优势，对 `tempdb` 进行了优化和排列。 `tempdb` 大小不可配置，它由存储子系统托管。

### <a name="does-my-database-size-automatically-grow-or-do-i-have-to-manage-the-size-of-the-data-files"></a>数据库大小是否自动增长，我是否需要管理数据文件的大小

你的数据库会随着你插入/引入更多数据自动增长。

### <a name="what-is-the-smallest-database-size-that-sql-database-hyperscale-supports-or-starts-with"></a>SQL 数据库“超大规模”服务层级支持或使用的最小数据库大小

10 GB

### <a name="in-what-increments-does-my-database-size-grow"></a>数据库的大小按多少增量增长

1 GB

### <a name="is-the-storage-in-sql-database-hyperscale-local-or-remote"></a>SQL 数据库“超大规模”服务层级中的存储是本地存储还是远程存储

在“超大规模”数据库中，数据文件存储在 Azure 标准存储中。 数据大多数缓存在本地 SSD 存储靠近计算节点的计算机中。 此外，计算节点在本地 SSD 和内存中（缓冲池等）均有缓存，以便降低从远程节点提取数据的频率。

### <a name="can-i-manage-or-define-files-or-filegroups-with-hyperscale"></a>能否使用“超大规模”数据库管理或定义文件或文件组

否
  
### <a name="can-i-provision-a-hard-cap-on-the-data-growth-for-my-database"></a>能否为我的数据库的数据增长预配硬上限

否

### <a name="how-are-data-files-laid-out-with-sql-database-hyperscale"></a>如何使用 SQL 数据库“超大规模”服务层级排列数据文件

数据文件受页服务器控制。 随着数据大小的增长添加数据文件和关联的页服务器节点。

### <a name="is-database-shrink-supported"></a>是否支持数据库收缩

否

### <a name="is-database-compression-supported"></a>是否支持数据库压缩

是

### <a name="if-i-have-a-huge-table-does-my-table-data-get-spread-out-across-multiple-data-files"></a>如果表格巨大，表格数据是否会分布在多个数据文件中

可以。 与给定表格关联的数据页可在多个数据文件中出现，它们均属于相同文件组。 SQL Server 使用[按比例填充策略](https://docs.microsoft.com/sql/relational-databases/databases/database-files-and-filegroups#file-and-filegroup-fill-strategy)在数据文件间分布数据。

## <a name="data-migration-questions"></a>数据迁移问题

### <a name="can-i-move-my-existing-azure-sql-databases-to-the-hyperscale-service-tier"></a>能否将现有 Azure SQL 数据库迁移到“超大规模”服务层级

可以。 可以将现有 Azure SQL 数据库迁移到“超大规模”服务层级。 这是单向的迁移。 无法将数据库从“超大规模”服务层级移到另一个服务层级。 建议创建生产数据库的副本，并将副本迁移到“超大规模”服务层级以获取概念证明 (POC)。
  
### <a name="can-i-move-my-hyperscale-databases-to-other-editions"></a>能否将“超大规模”数据库迁移到其他版本

不。 在此期间，不能将超大规模数据库移动到另一个服务层。

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>迁移到“超大规模”服务层级后，是否会丢失一些功能

可以。 Azure SQL 数据库功能的一些尚不支持在超大规模，包括但不是限定长术语保留备份。 将数据库迁移到超大规模服务层后，这些功能将停止运行。  我们预期这些限制是临时性的。

### <a name="can-i-move-my--on-premises-sql-server-database-or-my-sql-server-virtual-machine-database-to-hyperscale"></a>能否将我的本地 SQL Server 数据库或 SQL Server 虚拟机数据库迁移到“超大规模”服务层级

可以。 可以利用现有的所有迁移技术（包括 BACPAC、事务复制和逻辑数据加载）迁移到“超大规模”服务层级。 另请参阅 [Azure 数据库迁移服务](../dms/dms-overview.md)。

### <a name="what-is-my-downtime-during-migration-from-an-on-premises-or-virtual-machine-environment-to-hyperscale-and-how-can-i-minimize-it"></a>从本地或虚拟机环境迁移到“超大规模”服务层级期间，我的停机时间有多长，如何尽量减少停机时间

故障时间与将数据库迁移到 Azure SQL 数据库中的单一数据库相同。 可以使用[事务复制](replication-to-sql-database.md#data-migration-scenario
)尽量减少大小最多几 TB 的数据库的故障时间迁移。 对于非常大的数据库（10 TB 以上），可以考虑使用 ADF、Spark 或其他数据移动技术迁移数据。

### <a name="how-much-time-would-it-take-to-bring-in-x-amount-of-data-to-sql-database-hyperscale"></a>向 SQL 数据库“超大规模”服务层级引入 X 数据量需要多少时间

超大规模是能够使用 100 MB/秒的新/更改数据。

### <a name="can-i-read-data-from-blob-storage-and-do-fast-load-like-polybase-and-sql-data-warehouse"></a>能否从 blob 存储读取数据并执行快速加载（如 Polybase 和 SQL 数据仓库）

可以从 Azure 存储中读取数据并将数据加载到“超大规模”数据库（就像对常规的单一数据库执行的操作一样）。 Azure SQL 数据库当前不支持 Polybase。 可以通过使用 [Azure 数据工厂](https://docs.microsoft.com/azure/data-factory/)或通过[适用于 SQL 的 Spark 连接器](sql-database-spark-connector.md)在 [Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/) 中运行 Spark 作业，执行 Polybase。 SQL 的 Spark 连接器支持批量插入。

“超大规模”数据库中不支持简单恢复或批量日志记录模式。 提供高可用性需要完整恢复模式。 但是，相比于单个 Azure SQL 数据库而言，“超大规模”数据库由于具有新的日志体系结构，可提供更快的数据引入速率。

### <a name="does-sql-database-hyperscale-allow-provisioning-multiple-nodes-for-ingesting-large-amounts-of-data"></a>SQL 数据库“超大规模”层级是否允许预配多个节点，用于引入大量数据

不。 SQL 数据库“超大规模”服务层级是 SMP 体系结构，而不是非对称多重处理或多主数据库体系结构。 只能创建多个副本来横向扩展只读工作负荷。

### <a name="what-is-the-oldest-sql-server-version-will-sql-database-hyperscale-support-migration-from"></a>对于从 SQL Server 迁移，SQL 数据库“超大规模”服务层级支持的 SQL Server 最早版本是什么

SQL Server 2005。 有关详细信息，请参阅[迁移到单一数据库或共用数据库](sql-database-single-database-migrate.md#migrate-to-a-single-database-or-a-pooled-database)。 对于兼容性问题，请参阅[解决数据库迁移的兼容性问题](sql-database-single-database-migrate.md#resolving-database-migration-compatibility-issues)。

### <a name="does-sql-database-hyperscale-support-migration-from-other-data-sources-such-as-aurora-mysql-oracle-db2-and-other-database-platforms"></a>SQL 数据库“超大规模”服务层级是否支持从其他数据源（如 Aurora、MySQL、Oracle、DB2）和其他数据库平台进行迁移

可以。 来自 SQL Server 之外的其他数据源需要逻辑迁移。 可以使用 [Azure 数据库迁移服务](../dms/dms-overview.md)进行逻辑迁移。

## <a name="business-continuity-and-disaster-recovery-questions"></a>业务连续性和灾难恢复问题

### <a name="what-slas-are-provided-for-a-hyperscale-database"></a>对“超大规模”数据库提供的 SLA

主要任务和 1 可读辅助副本的默认值，SLA 为 99.95%的可用性。  包含多个副本，SLA 将高达 99.99%。  

### <a name="are-the-database-backups-managed-for-me-by-the-azure-sql-database-service"></a>Azure SQL 数据库服务是否托管我的数据库备份

是

### <a name="how-often-are-the-database-backups-taken"></a>创建数据库备份的频率

对于 SQL 数据库“超大规模”数据库，没有传统的完整、差异和日志备份。 但有数据文件的定期快照，并且在配置或提供的保持期内仅按原样保留生成的日志。

### <a name="does-sql-database-hyperscale-support-point-in-time-restore"></a>SQL 数据库“超大规模”服务层级是否支持时间点还原

是

### <a name="what-is-the-recovery-point-objective-rporecovery-time-objective-rto-with-backuprestore-in-sql-database-hyperscale"></a>SQL 数据库“超大规模”数据库中备份/还原的恢复点目标 (RPO) 和恢复时间目标 (RTO)

无论数据库大小，RPO 最少为 0，RTO 目标不超过 10 分钟。 

### <a name="do-backups-of-large-databases-affect-compute-performance-on-my-primary"></a>大型数据库的备份是否影响主计算节点的计算性能

不。 备份由存储子系统管理，利用文件快照。 它们不会影响主计算节点上的用户工作负荷。

### <a name="can-i-perform-geo-restore-with-a-sql-database-hyperscale-database"></a>能否使用 SQL 数据库“超大规模”数据库执行异地还原

可以。  完全支持异地还原。

### <a name="can-i-setup-geo-replication-with-sql-database-hyperscale-database"></a>能否使用 SQL 数据库“超大规模”数据库设置异地复制

现在不行。

### <a name="do-my-secondary-compute-nodes-get-geo-replicated-with-sql-database-hyperscale"></a>使用 SQL 数据库“超大规模”服务层级，能否对我的辅助计算节点进行异地复制

现在不行。

### <a name="can-i-take-a-sql-database-hyperscale-database-backup-and-restore-it-to-my-on-premises-server-or-sql-server-in-vm"></a>能不能备份 SQL 数据库“超大规模”数据库，并还原到我的本地服务器或 VM 中的 SQL Server

不。 “超大规模”数据库的存储格式不同于传统 SQL Server，你不控制备份且无权访问备份。 若要将数据从 SQL 数据库“超大规模”数据库提取出来，请使用导出服务或使用脚本加上 BCP。

## <a name="cross-feature-questions"></a>跨功能问题

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>迁移到“超大规模”服务层级后，是否会丢失一些功能

可以。 在超大规模，包括但不是限于的长期保留备份中不支持某些 Azure SQL 数据库功能。 将数据库迁移到超大规模服务层后，这些功能将停止运行。

### <a name="will-polybase-work-with-sql-database-hyperscale"></a>Polybase 是否适用于 SQL 超大规模数据库

不。 Azure SQL 数据库不支持 Polybase。

### <a name="does-the-compute-have-support-for-r-and-python"></a>计算是否支持 R 和 Python

不。 Azure SQL 数据库中不支持 R 和 Python。

### <a name="are-the-compute-nodes-containerized"></a>计算节点是否是容器化的

不。 数据库驻留在计算 VM（而非容器）中。

## <a name="performance-questions"></a>性能问题

### <a name="how-much-throughput-can-i-push-on-the-largest-sql-database-hyperscale-compute"></a>能在最大的 SQL 数据库“超大规模”服务层级计算上推送多少吞吐量

我们已了解变更数据 （事务日志数据生成） 一致 100 MB/秒

### <a name="how-many-iops-do-i-get-on-the-largest-sql-database-hyperscale-compute"></a>能在最大的 SQL 数据库“超大规模”服务层级计算上获得多少 IOPS

IOPS 和 IO 延迟而异的工作负载模式。  如果无需访问的数据在本地计算的缓存，它将为本地 SSD IO 模式。   

### <a name="does-my-throughput-get-affected-by-backups"></a>吞吐量是否受备份影响

不。 计算将脱离存储层，以避免对计算的影响。

### <a name="does-my-throughput-get-affected-as-i-provision-additional-compute-nodes"></a>当我预配其他计算节点时，吞吐量是否会受到影响

因为存储共享并且没有发生的主要和辅助计算节点之间没有直接物理复制从技术上讲，主节点上的吞吐量将不会影响添加读取缩放节点。 但我们可以限制连续主动工作负荷，从而允许日志应用到次要节点和页服务器上，并避免次要节点上读取性能不佳。

## <a name="scalability-questions"></a>可伸缩性问题

### <a name="how-long-would-it-take-to-scale-up-and-down-a-compute-node"></a>纵向扩展和减少计算节点需要多长时间

向上缩放计算或向下应采用而不考虑数据大小的 5 到 10 分钟。

### <a name="is-my-database-offline-while-the-scaling-updown-operation-is-in-progress"></a>进行纵向扩展/缩减操作时，数据库是否处于脱机状态

不。 纵向扩展/缩减操作为联机操作。

### <a name="should-i-expect-connection-drop-when-the-scaling-operations-are-in-progress"></a>缩放操作过程中，连接是否会断开

如果具有目标大小的计算节点发生故障转移，纵向扩展/缩减操作会导致现有连接断开。 添加读取副本不会导致连接断开。

### <a name="is-the-scaling-up-and-down-of-compute-nodes-automatic-or-end-user-triggered-operation"></a>纵向扩展和缩减计算节点是自动的操作还是由最终用户触发的操作

是由最终用户触发的。 不是自动的。  

### <a name="does-my-tempb-also-grow-as-the-compute-is-scaled-up"></a>计算纵向扩展时，`tempb` 是否也会随之增长

可以。 临时数据库会随着计算自动纵向扩展。  

### <a name="can-i-provision-multiple-primary-computes-such-as-a-multi-master-system-where-multiple-primary-compute-heads-can-drive-a-higher-level-of-concurrency"></a>能不能预配多个主计算节点（例如多主数据库系统，其中多个主要计算节点可促进更高的并发级别）

不。 只有主计算节点接受读/写请求。 次要计算节点只接受只读请求。

## <a name="read-scale-questions"></a>读取扩展问题

### <a name="how-many-secondary-compute-nodes-can-i-provision"></a>可以预配多少个次要计算节点

默认情况下，我们创建超大规模数据库 2 个的副本。 如果你想要调整副本数，您可以使用来完成[Azure 门户](https://portal.azure.com)。

### <a name="how-do-i-connect-to-these-secondary-compute-nodes"></a>如何连接到这些次要计算节点

将连接字符串的 `ApplicationIntent` 参数设置为 `read_only`，即可连接到这些额外的只读计算节点。 任何标记为 `read-only` 的连接均自动路由到某个额外的只读计算节点。  

### <a name="can-i-create-a-dedicated-endpoint-for-the-read-scale-replica"></a>能不能为读取扩展副本创建专用终结点

不。 仅可以通过指定连接到读取缩放副本`ApplicationIntent=ReadOnly`。

### <a name="does-the-system-do-intelligent-load-balancing-of-the-read-workload"></a>系统是否对读取工作负荷进行智能负载均衡

不。 读取的唯一工作负荷将重新定向到的随机读取缩放副本。

### <a name="can-i-scale-updown-the-secondary-compute-nodes-independently-of-the-primary-compute"></a>能否独立于主计算纵向扩展/缩减次要计算节点

不。 辅助计算节点还用于高可用性，因此它们需要的主数据库，在故障转移时相同的配置。

### <a name="do-i-get-different-temp-db-sizing-for-my-primary-compute-and-my-additional-secondary-compute-nodes"></a>对于我的主要计算节点和额外的次要计算节点，能否获得不同的临时数据库大小

不。 你`tempdb`配置根据计算大小预配，辅助计算节点都与主计算大小相同。

### <a name="can-i-add-indexes-and-views-on-my-secondary-compute-nodes"></a>能否对次要计算节点添加索引和视图

不。 “超大规模”数据库有共享存储，这表示所有计算节点会看到相同的表格、索引和视图。 如果要使次要计算节点上的额外索引得到读取优化，必须先在主计算节点上添加索引。

### <a name="how-much-delay-is-there-going-to-be-between-the-primary-and-secondary-compute-node"></a>主计算节点和次要计算节点之间的延迟

从主计算节点上提交事务的时间起，具体取决于日志生成速率，可能是瞬时的，也可能为低延迟（毫秒计）。

## <a name="next-steps"></a>后续步骤

有关超大规模服务层的详细信息，请参阅[超大规模服务层](sql-database-service-tier-hyperscale.md)。
