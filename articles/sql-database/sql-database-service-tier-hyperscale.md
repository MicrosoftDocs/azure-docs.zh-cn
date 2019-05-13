---
title: 超大规模 Azure SQL 数据库概述 | Microsoft Docs
description: 本文介绍 Azure SQL 数据库中基于 vCore 的购买模型中的超大规模服务层级，并说明它与常规用途服务层级和业务关键服务层级的不同之处。
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
ms.openlocfilehash: 9455b8488bdf7c36c662a8f771e6b26d1a27b13e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236554"
---
# <a name="hyperscale-service-tier-for-up-to-100-tb"></a>最多 100 TB 的超大规模服务层

Azure SQL 数据库基于 SQL Server 数据库引擎体系结构，该体系结构已根据云环境做出调整，以确保即使在发生基础结构故障时，也仍能提供 99.99% 的可用性。 Azure SQL 数据库中使用了三种体系结构模型：
- 常规用途/标准 
-  超大规模
-  业务关键/高级

Azure SQL 数据库中的超大规模服务层级是基于 vCore 的购买模型中的最新服务层级。 此服务层级是一个高度可缩放的存储和计算性能层，它利用 Azure 体系结构来横向扩展 Azure SQL 数据库的存储和计算资源，远远超出了“常规用途”和“业务关键”服务层级的可用限制。

> 
> [!NOTE]
> 有关基于 vCore 的购买模型中的常规用途服务层级和业务关键服务层级的详细信息，请参阅[常规用途](sql-database-service-tier-general-purpose.md)服务层级和[业务关键](sql-database-service-tier-business-critical.md)服务层级。 有关基于 vCore 购买模型与基于 DTU 购买模型的比较，请参阅 [Azure SQL 数据库购买模型和资源](sql-database-service-tiers.md)。


## <a name="what-are-the-hyperscale-capabilities"></a>“超大规模”服务层级具有哪些功能

Azure SQL 数据库中的“超大规模”服务层级提供了以下附加功能：

- 支持高达 100 TB 的数据库大小
- 几乎瞬时完成数据库备份（基于存储在 Azure Blob 存储中的快照），无论数据库大小，也不会对计算造成 IO 影响   
- 在几分钟内快速完成数据库还原（基于文件快照），无需数小时或数天（不基于数据操作的大小）
- 无论数据卷如何，由于更高的日志吞吐量和更快的事务提交速度，整体性能更高
- 快速横向扩展 - 可预配一个或多个只读节点，以卸载读取工作负载并用作热备用服务器
- 快速纵向扩展 - 可在不变的时间内纵向扩展计算资源，以在需要时适应繁重的工作负载，然后在不需要时缩减计算资源。

“超大规模”服务层级消除了传统上云数据库中出现的许多实际限制。 当大多数其他数据库受单个节点中可用资源的限制时，“超大规模”服务层级中的数据库则没有此类限制。 它具备灵活的存储体系结构，存储可按需增长。 实际上，不会使用定义的最大大小创建“超大规模”数据库。 “超大规模”数据库会按需扩大，你仅需为所使用的容量付费。 对于读取密集型工作负载，“超大规模”服务层级通过按需预配其他只读副本来卸载读取工作负载，从而实现快速横向扩展。

此外，创建数据库备份或纵向扩展/横向扩展所需的时间不再与数据库中的数据卷相关。 几乎可以即时备份“超大规模”数据库。 还可在几分钟内纵向扩展或横向扩展数十 TB 的数据库。 此功能使你无需担心受初始配置选项的约束。

有关“超大规模”服务层级计算大小的详细信息，请参阅[服务层级特征](sql-database-service-tiers-vcore.md#service-tier-characteristics)。

## <a name="who-should-consider-the-hyperscale-service-tier"></a>哪些群体应考虑使用“超大规模”服务层级

“超大规模”服务层级主要面向在本地拥有大型数据库并希望通过迁移到云来实现应用程序现代化的客户，或已在云中并受到最大数据库大小限制 (1-4 TB) 的客户。 它也适用于那些寻求存储和计算的高性能和高可伸缩性的客户。

“超大规模”服务层级支持所有 SQL Server 工作负载，但它主要针对 OLTP 进行优化。 “超大规模”服务层级还支持混合和分析（数据市场）工作负载。

> [!IMPORTANT]
> 弹性池不支持“超大规模”服务层级。

## <a name="hyperscale-pricing-model"></a>“超大规模”定价模型

仅 [vCore 模型](sql-database-service-tiers-vcore.md)提供“超大规模”服务层级。 为了适应新的体系结构，它的定价模型与“常规用途”或“业务关键”服务层级略有不同：

- **计算**：

  “超大规模”计算单位按副本计费。 [Azure 混合权益](https://azure.microsoft.com/pricing/hybrid-benefit/)价格会自动应用到读取扩展副本。 我们默认情况下创建了主副本和每个超大规模数据库的一个只读副本。  用户可能会调整包括从 1-5 的主要副本的总数。

- **存储**：

  配置“超大规模”数据库时，无需指定最大数据大小。 超大规模层中根据实际用量收取数据库存储费用。 自动分配 10 GB 和 100 TB 之间动态调整 10 GB 到 40 GB 之间的增量进行存储。  

有关“超大规模”服务层级定价的详细信息，请参阅 [Azure SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/single/)

## <a name="distributed-functions-architecture"></a>分布式功能体系结构

不同于将所有数据管理功能集中在一个位置/进程中的传统数据库引擎（即便是当今生产中所谓的分布式数据库也有单片数据引擎的多个副本），“超大规模”数据库将查询处理引擎（其中各种数据引擎的语义不同）与为数据提供长期存储和持久性的组件分隔开来。 通过这种方式，可根据需要顺利地扩大存储容量（初始目标是 100 TB）。 只读副本共享相同的存储组件，以便快速创建新的可读副本所不需的任何数据副本。 

下图说明了“超大规模”数据库中不同类型的节点：

![体系结构](./media/sql-database-hyperscale/hyperscale-architecture.png)

“超大规模”数据库包含以下不同类型的节点：

### <a name="compute-node"></a>计算节点

计算节点是关系引擎的所在位置，因此会出现所有语言元素、查询处理等。 所有用户与“超大规模”数据库的交互都通过这些计算节点进行。 计算节点具有基于 SSD 的缓存（在上图中标记为 RBPEX - 可复原缓冲池扩展），可最小化提取一页数据所需的网络往返次数。 其中有一个处理所有读写工作负载和事务的主计算节点。 有一个或多个充当热备用服务器节点的辅助计算节点，用于进行故障转移，也充当用于卸载只读工作负载的只读计算节点（如需此功能）。

### <a name="page-server-node"></a>页面服务器节点

页面服务器是表示横向扩展存储引擎的系统。  每个页面服务器负责数据库中页面的一个子集。  名义上，每个页服务器控件之间 128 GB 和 1 TB 的数据。 多个页面服务器上不共享任何数据（为冗余和可用性而保留的副本除外）。 页面服务器的任务是按需向计算节点提供数据库页面，并在事务更新数据时持续更新页面。 页面服务器通过从日志服务播放日志记录来保持最新。 页面服务器还保留基于 SSD 的缓存，可提高性能。 数据页的长期存储保存在 Azure 存储中，可提高可靠性。

### <a name="log-service-node"></a>日志服务节点

日志服务节点接受来自主计算节点的日志记录，将其保存在永久缓存中，并将日志记录转发给其余计算节点（以便它们可以更新缓存）以及一个或多个相关页面服务器，以便可在此处更新数据。 通过这种方式，主计算节点中的所有数据更改都通过日志服务传播到所有辅助计算节点和页面服务器中。 最后，日志记录被推送到 Azure 存储的长期存储（无限存储库）中。 此机制消除了频繁截断日志的必要性。 日志服务还具有本地缓存，可加快访问速度。

### <a name="azure-storage-node"></a>Azure 存储节点

Azure 存储节点是页面服务器中数据的最终目标。 此存储用于备份用途以及 Azure 区域之间的复制。 备份包括数据文件的快照。 从这些快照还原操作很快，数据可还原到任何时间点。

## <a name="backup-and-restore"></a>备份和还原

备份是文件快照库，因此它们几乎是瞬时完成的。 存储和计算分离可将备份/还原操作推送到存储层，以减少主计算节点的处理负担。 因此，大型数据库的备份不会影响主计算节点的性能。 同样，还原是通过复制文件快照完成的，因此不是数据操作的大小。 对于同一存储帐户中的还原，还原操作很快。

## <a name="scale-and-performance-advantages"></a>缩放和性能优势

超大规模体系结构能快速启动/关闭其他只读计算节点，拥有重要的读取扩展功能，还可以释放主计算节点，以满足更多写入请求。 此外，由于超大规模体系结构的共享存储体系结构，可快速纵向扩展/横向扩展计算节点。

## <a name="create-a-hyperscale-database"></a>创建超大规模数据库

可以使用 [Azure 门户](https://portal.azure.com)、[T-SQL](https://docs.microsoft.com/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current)[Powershell](https://docs.microsoft.com/powershell/module/azurerm.sql/new-azurermsqldatabase) 或者 [CLI](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-create) 创建超大规模数据库。 仅可通过[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)使用超大规模数据库。

以下 T-SQL 命令可创建一个“超大规模”数据库。 必须在 `CREATE DATABASE` 语句中指定版本和服务目标。

```sql
-- Create a HyperScale Database
CREATE DATABASE [HyperScaleDB1] (EDITION = 'HyperScale', SERVICE_OBJECTIVE = 'HS_Gen5_4');
GO
```
这将创建具有 4 个内核的 Gen5 硬件上的超大规模数据库。

## <a name="migrate-an-existing-azure-sql-database-to-the-hyperscale-service-tier"></a>将现有 Azure SQL 数据库迁移到“超大规模”服务层级

可以使用 [Azure 门户](https://portal.azure.com)[T-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current)[Powershell](https://docs.microsoft.com/powershell/module/azurerm.sql/set-azurermsqldatabase) 或者 [CLI](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-update)将现有的 Azure SQL 数据库迁移到“超大规模”层级。 在此期间，这是单向的迁移。 无法将数据库从“超大规模”服务层级移到另一个服务层级。 建议创建生产数据库的副本，并将副本迁移到“超大规模”服务层级以获取概念证明 (POC)。

以下 T-SQL 命令可将数据库迁移到“超大规模”服务层级。 必须在 `ALTER DATABASE` 语句中指定版本和服务目标。

```sql
-- Alter a database to make it a HyperScale Database
ALTER DATABASE [DB2] MODIFY (EDITION = 'HyperScale', SERVICE_OBJECTIVE = 'HS_Gen5_4');
GO
```

## <a name="connect-to-a-read-scale-replica-of-a-hyperscale-database"></a>连接到“超大规模”数据库的读取扩展副本

在超大规模数据库中，由客户端提供的连接字符串中的 `ApplicationIntent` 参数决定连接是路由到写入副本，还是路由到只读的次要副本。 如果将 `ApplicationIntent` 设置为 `READONLY`并且数据库不具有辅助副本，连接将路由到主副本，默认执行 `ReadWrite` 行为。

```cmd
-- Connection string with application intent
Server=tcp:<myserver>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```
## <a name="disaster-recovery-for-hyperscale-databases"></a>对于超大规模的数据库的灾难恢复
### <a name="restoring-a-hyperscale-database-to-a-different-geography"></a>超大规模的数据库还原到另一个地理位置
如果你需要一个 Azure SQL 数据库的超大规模 DB 还原到当前托管在中，作为灾难恢复操作或向下钻、 重定位或任何其他原因，一部分以外区域的主要方法是数据库的执行异地还原。  这涉及到与要还原到不同区域的任何其他 AZURE SQL DB 会使用相同的步骤：
1. 如果你还没有相应的服务器存在，请在目标区域中创建 SQL 数据库服务器。  此服务器应拥有的原始 （源） 服务器所在的同一订阅。
2. 按照中的说明[异地还原](https://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups#geo-restore)从自动备份还原 Azure SQL 数据库页的主题。

#### <a name="notes-on-geo-restores-of-a-hyperscale-database"></a>异地还原的超大规模数据库上的说明
由于源和目标是在不同区域中，数据库不能共享存储快照与源数据库与非异地还原，极快地完成。  对于超大规模数据库异地还原，它将大小的数据操作，即使目标是异地复制存储在配对区域中。  这意味着，执行异地还原将需要与正在还原的数据库的大小成比例的时间。  如果目标是在配对区域中，副本将位于在数据中心内，它将是明显快于人長途電复制通过 internet，但它仍将复制的所有位。

## <a name=regions></a>可用区域

Azure SQL 数据库超大规模层目前已在以下区域：

- 澳大利亚东部
- 澳大利亚东南部
- 巴西南部
- 加拿大中部
- 美国中部
- 中国东部 2
- 中国北部 2
- 东亚
- 美国东部
- 东部我们 2
- 法国中部
- 日本东部
- 日本西部
- 韩国中部
- 韩国南部
- 美国中北部
- 北欧
- 南非北部
- 美国中南部
- 东南亚
- 英国南部
- 英国西部
- 西欧
- 美国西部
- 美国西部 2

如果你想要在未列出所支持的区域中创建超大规模数据库，可以发送通过 Azure 门户的载入请求。 我们正在努力展开的列表受支持的区域，因此请回来查看最新的区域列表。

若要请求未列出的区域中创建超大规模的数据库的功能：

1. 导航到[Azure 帮助和支持边栏选项卡](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

2. 单击[**新建支持请求**](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)

    ![Azure 帮助和支持边栏选项卡](media/sql-database-service-tier-hyperscale/whitelist-request-screen-1.png)

3. 有关**问题类型**，选择**服务和订阅限制 （配额）**

4. 选择将用于创建数据库的订阅

5. 有关**配额类型**，选择**SQL 数据库**

6. 单击“下一步:**解决方案**

1. 单击**提供详细信息**

    ![问题详细信息](media/sql-database-service-tier-hyperscale/whitelist-request-screen-2.png)

8. 选择**SQL 数据库配额类型**:**其他配额请求**

9. 填写以下模板：

    ![配额详细信息](media/sql-database-service-tier-hyperscale/whitelist-request-screen-3.png)

    在模板中，提供以下信息

    > 若要在新的区域中创建 Azure 超大规模 SQL 数据库的请求<br/> 区域: [填写请求的区域]  <br/>
    > 计算包括可读副本的 SKU/总核心数 <br/>
    > 估计的 TB 数 
    >

10. 选择“严重性 C”

11. 选择相应的联系方法，并填入详细信息。

12. 单击**保存**和**继续**

## <a name="known-limitations"></a>已知限制
这些是超大规模服务层提供 ga 版。 截至到当前的限制  我们努力删除任意数量的尽可能这些限制。

| 问题 | 描述 |
| :---- | :--------- |
| 逻辑服务器的“管理备份”窗格不显示将从 SQL Server 筛选的“超大规模”数据库  | “超大规模”具有用于管理备份的单独方法，因此长期保留和备份保留设置中的时间点不适用/将失效。 相应地，“超大规模”数据库不会显示在“管理备份”窗格中。 |
| 时间点还原 | 将数据库迁移到“超大规模”服务层级后，不支持还原到迁移之前的某个时间点。|
| 还原的非-超大规模 DB 到 Hypserscale，反之亦然 | 你无法将超大规模数据库还原到非超大规模数据库中，也可以在非超大规模的数据库还原到超大规模数据库。|
| 迁移期间，如果数据库文件由于活动的工作负荷而增大，并且超过每个文件的边界 (1 TB)，迁移将失败 | 缓解措施： <br> - 如果可能，请在没有运行任何更新工作负荷时迁移数据库。<br> - 重试迁移，只要在迁移期间文件大小不超过 1 TB 边界，迁移就会成功。|
| 托管实例 | Azure SQL 数据库托管实例目前不支持使用超大规模的数据库中。 |
| 弹性池 |  弹性池目前不支持 SQL 数据库的超大规模。|
| 迁移到“超大规模”服务层级目前是单向操作 | 将数据库迁移到“超大规模”服务层级后，它不能直接迁移到非“超大规模”服务层级。 目前，将数据库从“超大规模”服务层级迁移到非“超大规模”服务层级的唯一方法是使用 BACPAC 文件进行导出/导入。|
| 具有内存中对象的数据库的迁移 | 将数据库迁移到“超大规模”服务层级之前，必须删除内存中文件并重新创建为非内存中对象。|
| 更改数据跟踪 | 你将无法再用于更改数据跟踪超大规模的数据库。 |
| 异地复制  | 尚不能为 Azure SQL 数据库超大规模配置异地复制。  你可以执行异地还原 （灾难恢复或其他目的还原的数据库中的不同地理位置） |
| TDE/AKV 集成 | 使用 Azure 密钥保管库 （通常称为自带您的自己的密钥，简称 BYOK） 的透明数据库加密是尚不支持为 Azure SQL 数据库超大规模，但完全支持使用服务托管密钥的 TDE。 |
|智能数据库功能 | 1.创建索引，Drop Index 顾问的超大规模数据库未训练模型。 <br/>2.架构问题、 DbParameterization-最近添加的顾问不支持超大规模数据库。|



## <a name="next-steps"></a>后续步骤

- 有关“超大规模”的常见问题，请参阅[“超大规模”常见问题解答](sql-database-service-tier-hyperscale-faq.md)。
- 有关服务层级的信息，请参阅[服务层级](sql-database-service-tiers.md)
- 有关服务器和订阅级别限制的信息，请参阅[逻辑服务器上的资源限制概述](sql-database-resource-limits-logical-server.md)。
- 有关单一数据库的购买模型限制的信息，请参阅 [适用于单一数据库的 Azure SQL 数据库基于 vCore 的购买模型限制](sql-database-vcore-resource-limits-single-databases.md)。
- 有关功能和比较列表，请参阅 [SQL 常用功能](sql-database-features.md)。
