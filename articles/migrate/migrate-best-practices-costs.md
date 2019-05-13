---
title: 为迁移到 Azure 的工作负荷计算成本和调整大小的最佳做法 | Microsoft Docs
description: 获取为迁移到 Azure 的工作负荷计算成本和调整大小的最佳做法。
author: rayne-wiselman
manager: carmonm
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 12/08/2018
ms.author: raynew
ms.openlocfilehash: 48ce99bd830d2c35e5cb9703d2ef754a602d534b
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64926148"
---
# <a name="best-practices-for-costing-and-sizing-workloads-migrated-to-azure"></a>为迁移到 Azure 的工作负荷计算成本和调整大小的最佳做法

在进行迁移的计划和设计时，重点考虑成本可保证 Azure 迁移获得长期成功。 在迁移项目中，非常重要的一点是让所有团队（财务、管理、应用团队等）都了解相关费用。
- 迁移前，估算迁移费用并设定月度、季度、年度预算目标基线，这是成功的关键。
- 迁移后，应优化成本、持续监视工作负荷并规划未来的使用模式。 迁移的资源可能一开始是一种类型的工作负荷，但会随着时间的迁移，根据使用情况、成本和不断变化的业务需求逐渐演变为另一种类型。

本文介绍迁移前后计算成本和调整大小的最佳做法。  

> [!IMPORTANT]
> 本文介绍的最佳做法和意见以撰写本文时可用的 Azure 平台和服务功能为基础。 各种特性和功能随时间而变化。 可能并非所有建议都适用于你的部署，请选择合适的建议。


## <a name="before-migration"></a>迁移前 

在将工作负荷移动到云之前，估算在 Azure 中运行它们每月所需的费用。 积极管理云成本有助于符合营运费用 (OpEx) 预算。 如果预算有限，请在迁移前考虑这一点。  考虑根据需要将工作负荷转换为 Azure无服务器技术，以降低成本。

本部分中提到的最佳做法有助于估算成本、适当调整 VM 和存储的大小，利用 Azure 混合权益、使用预留 VM 以及估算各订阅的云支出。



## <a name="best-practice-estimate-monthly-workload-costs"></a>最佳做法：估计每月工作负荷成本
 
为迁移的工作负荷预测每月账单费用，可以使用许多工具。

- **Azure 定价计算器**：选择要评估费用的产品，例如 VM 和存储。 将费用输入定价计算器中，生成一个估算。

  ![Azure 定价计算器](./media/migrate-best-practices-costs/pricing.png) *Azure pricing calculator*

- **Azure Migrate**：若要估算成本，你需要查看并考虑在 Azure 中运行工作负荷需要的所有资源。 要获取该数据，需创建资产清单（资产包括服务器、VM、数据库和存储）。 可以使用 Azure Migrate 来收集这些信息。

  - Azure Migrate 发现并评估你的本地环境，从而提供清单。
  - Azure Migrate 可以映射并显示 VM 之间的依赖关系，使你了解全貌。
  - Azure Migrate 评估包含估计成本。
    - 计算成本：在创建评估时使用推荐的 Azure VM 大小，Azure Migrate 会使用计费 API 计算每月预估 VM 费用。 该估算会考虑操作系统、软件保障、预留实例、VM 运行时间、位置和货币设置。 它将所有 VM 的成本求和，计算每月总计算成本。
    - 存储成本：Azure Migrate 通过在评估中将所有 VM 的存储成本求和，计算每月总存储成本。 可以按照如下方式计算某台计算机的每月存储成本：将连接到该它的所有磁盘的每月成本求和。 

    ![Azure Migrate](./media/migrate-best-practices-costs/assess.png)
    *Azure Migrate 评估*

**了解更多：**
- [使用](https://azure.microsoft.com/pricing/calculator/) Azure 定价计算器。
- [获取 ](https://docs.microsoft.com/azure/migrate/migrate-overview) Azure Migrate 的概述。
- [阅读 ](https://docs.microsoft.com/azure/migrate/concepts-assessment-calculation) Azure Migrate 评估相关内容。
- [了解](https://docs.microsoft.com/azure/dms/dms-overview)有关数据库迁移服务 (DMS) 的更多信息。

## <a name="best-practice-right-size-vms"></a>最佳做法：将 VM 调整为适当大小

可在部署 Azure VM 时选择许多支持工作负荷的选项。 每种 VM 类型都有特定的功能和不同的 CPU、存储及磁盘组合。 虚拟机的分组情况如下所示。

类型 | **详细信息** | **使用**
--- | --- | ---
**常规用途** | CPU 与内存之比均衡。 | 适用于测试和开发、小到中型数据库以及低到中等流量的 Web 服务器。
**计算优化** | 高 CPU 与内存之比。 | 适用于中等流量的 Web 服务器、网络设备、批处理和应用服务器。
**内存优化** | 高内存与 CPU 之比。 | 适用于关系数据库、中到大型缓存和内存中分析。
**存储优化** | 高磁盘吞吐量和 IO。 | 适用于大数据、SQL 和 NoSQL 数据库。
**GPU 优化** | 专用 VM。 一个或多个 GPU。 | 大量图形和视频编辑。
**高性能** | 最快且最强大的 CPU。 VM 具有可选的高吞吐量网络接口 (RDMA) | 关键的高性能应用。

- 了解这些 VM 之间的定价差异和长期预算效应非常重要。
- 每种类型内都有许多 VM 系列。
- 此外，选择系列中的某个 VM 时，只能在该系列内对该 VM 进行纵向扩展和缩减。 例如，可将 DSv2\_2 纵向扩展为 DSv2\_4，但不能将它更改为不同的系列，例如 Fsv2\_2。

**了解更多：**
- [了解](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)有关 VM 的类型和大小调整以及将大小映射到类型的详细信息。
- [计划](https://docs.microsoft.com/azure/cloud-services/cloud-services-sizes-specs) VM 大小调整。
- [查看](https://docs.microsoft.com/azure/migrate/contoso-migration-assessment)对虚构公司 Contoso 的示例评估。

## <a name="best-practice-select-the-right-storage"></a>最佳做法：选择适当的存储

优化和维护本地存储（SAN 或 NAS）以及支持它们的网络可能成本高且耗时。 为了帮助缓解运营和管理方面的问题，通常将文件（存储）数据迁移到云中。 Microsoft 提供若干将数据迁移到 Azure 的选项，你需要决定使用哪一个。 为数据选取适当的存储类型每月可为组织节省数千美元费用。 几个注意事项：

- 不需要将访问频率低的非业务关键数据放在最昂贵的存储中。
- 反之，应将重要的业务关键数据放在更高层的存储中。
- 在迁移规划期间，获取数据清单，按重要性对数据分类，以便将其映射到最适合的存储。 考虑预算、成本和性能。 成本不一定是主要决策因素。 选取费用最低的选项可能会为工作负荷带来性能和可用性风险。 

### <a name="storage-data-types"></a>存储数据类型

Azure 提供了不同类型的存储数据。

| **数据类型** | **详细信息** | **使用情况** |
|--- | --- |  --- |
|**Blob** | 优化为可存储巨量的非结构化对象，例如文本或二进制数据<br/>通过 HTTP/HTTPS 从任何位置访问数据。 | 用于流式处理和随机访问方案。 例如，直接向浏览器提供图像和文档，流式传输视频和音频，以及存储备份和灾难恢复数据。|
|**文件** | 通过 SMB 3.0 访问的托管文件共享 | 在迁移本地文件共享时使用，用于提供对文件数据的多路访问/连接。|
|**磁盘** | 基于页 blob。<br/><br/> 磁盘类型（速度）：标准（HDD 或 SSD）或高级 (SSD)。<br/><br/>磁盘管理：非托管（你管理磁盘设置和存储）或托管（你选择磁盘类型，Azure 为你管理磁盘）。 | 为 VM 使用高级磁盘。 将托管磁盘用于简单的管理和缩放。|
|**队列** | 存储并检索大量由通过了身份验证的调用（HTTP 或 HTTPS）访问的消息 | 通过异步消息排队连接应用组件。|
|**表** | 存储表。 | 现属于 Azure Cosmos DB 表 API。|



### <a name="access-tiers"></a>访问层级
Azure 存储提供不同的选项，用于访问块 Blob 数据。 选择适当的访问权限层，有助于确保以最经济高效的方式存储块 Blob 数据。

类型 | **详细信息** | **使用情况**
--- | --- | ---
**热访问层** | 存储成本比冷访问层高。 访问费用比冷访问层低。<br/><br/>这是默认层级。 | 用于正在使用的访问频率较高的数据。
**冷访问层** | 存储成本比热访问层低。 访问费用比热访问层高。<br/><br/> 至少 30 天的存储。 | 短期存储，数据可用，但不经常访问。
**存档** | 用于各个块 Blob。<br/><br/> 最经济高效的存储选项。 数据访问费用比冷热两个存储层高。 | 用于可以容忍数小时的检索延迟且会保留在层中至少 180 天的数据。 

### <a name="storage-account-types"></a>存储帐户类型

Azure 提供了不同类型的存储帐户和性能层。

**帐户类型** | **详细信息** | **使用情况**
--- | --- | ---
**常规用途 v2 标准** | 支持 blob（块 blob、页 blob、追加 blob）、文件、磁盘、队列和表。<br/><br/> 支持热、冷和存档三个访问层。 支持 ZRS。 | 用于大多数方案和大多数数据类型。 标准存储帐户可以基于 HDD 或 SSD。
**常规用途 v2 高级** | 支持 Blob 存储数据（页 blob）。 支持热、冷和存档三个访问层。 支持 ZRS。<br/><br/> 存储在 SSD 上。 | Microsoft 建议用于所有 VM。
**常规用途 v1** | 不支持访问分层。 不支持 ZRS | 在应用需要 Azure 经典部署模型时使用。
**Blob** | 专用存储帐户，用于存储非结构化对象。 仅提供块 blob 和追加 blob（不提供任何文件、队列、表或磁盘存储服务）。 提供与常规用途 v2 一样的持续性、可用性、可伸缩性和性能。 | 不能在这些帐户中存储页 blob，因此不能存储 VHD 文件。 可以将访问层设置为“热”或“冷”。

### <a name="storage-redundancy-options"></a>存储冗余选项

存储帐户可以为获得恢复能力和高可用性使用不同的冗余类型。

类型 | **详细信息** | **使用情况**
--- | --- | ---
**本地冗余存储 (LRS)** | 从一个存储单元内复制到单独的容错域和更新域，防止出现本地中断。 在一个数据中心内保留多个数据副本。 在给定的一年时间内提供至少 99.999999999%（11 个 9）的对象持续性。 | 请考虑你的应用是否存储可以轻松重新构造的数据。
**区域冗余存储 (ZRS)** | 通过在一个区域中跨三个存储群集复制防止数据中心出现中断。 各个存储群集在物理上分离，位于其自己的可用性区域中。 将数据的多个副本保留在多个数据中心或多个区域，从而在给定年份提供至少 99.9999999999%（12 个 9）的对象持续性。 | 考虑是否需要一致性、持续性和高可用性。 如果多个区域永久受到影响，在发生区域性的灾难时，可能无法提供保护。
**地域冗余存储 (GRS)** | 通过将数据复制到距主要区域数百英里的次要区域，防止整个区域发生中断。 在给定的一年时间内提供至少 99.99999999999999%（16 个 9）的对象持续性。 | 在 Microsoft 启动故障转移到次要区域之前，副本数据不可用。 如果发生故障转移，读写访问权限可用。
**读取访问地域冗余存储 (RA-GRS)** | 类似于 GRS。 在给定的一年时间内提供至少 99.99999999999999%（16 个 9）的对象持续性 | 通过允许从用于 GRS 的第二个区域进行读取访问，提供 99.99% 的读取可用性。

**了解更多：**
- [查看](https://azure.microsoft.com/pricing/details/storage/) Azure 存储定价。
- [了解](https://docs.microsoft.com/azure/storage/common/storage-import-export-service)用于迁移大量 Azure blob 及文件数据的 Azure 导入/导出。 
- [比较](https://docs.microsoft.com/azure/storage/common/storage-decide-blobs-files-disks?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) blob、文件和磁盘这三种存储的数据类型。
- [详细了解](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers)访问层。
- [查看](https://docs.microsoft.com/azure/storage/common/storage-account-overview?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)不同的存储帐户类型。
- 了解[存储冗余](https://docs.microsoft.com/azure/storage/common/storage-redundancy)、[LRS](https://docs.microsoft.com/azure/storage/common/storage-redundancy-lrs?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)、[ZRS](https://docs.microsoft.com/azure/storage/common/storage-redundancy-zrs?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)、[GRS](https://docs.microsoft.com/azure/storage/common/storage-redundancy-grs?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) 和[读取访问 GRS](https://docs.microsoft.com/azure/storage/common/storage-redundancy-grs?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#read-access-geo-redundant-storage)。
- [详细了解](https://docs.microsoft.com/azure/storage/files/storage-files-introduction) Azure 文件。



## <a name="best-practice-leverage-azure-hybrid-benefits"></a>最佳做法：利用 Azure 混合权益

Microsoft 多年来对 Windows Server 和 SQL Server 等系统提供了许多软件投资，在为客户提供云价值方面独具优势，能够提供其他云提供商不一定能提供的大幅优惠。 

集成的 Microsoft 本地产品或 Azure 产品组合带来竞争和成本优势。 如果你当前拥有通过软件保障授权的操作系统或其他软件，可将这些许可证迁移到云端获得 Azure 混合权益。

**了解更多：**

- [看一看](https://azure.microsoft.com/pricing/hybrid-benefit/)混合权益节省额计算器。
- [了解](https://azure.microsoft.com/pricing/hybrid-benefit/) Windows Server 的混合权益。
- [查看](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance) SQL Server Azure VM 的定价指南。


## <a name="best-practice-use-reserved-vm-instances"></a>最佳做法：使用虚拟机预留实例

大多数云平台都设置为即用即付。 这种模型有一些缺点，因为你不一定知道工作负荷的动态程度。 为工作负荷指明意图，有助于基础结构计划。

使用 Azure 虚拟机预留实例，就是预付一年或三年的 VM 实例费用。 

- 预付向你使用的资源提供折扣。
- 可大幅减少（最多减少即用即付价格的 72%）VM、SQL 数据库计算、Azure Cosmos DB 或其他资源的成本。 
- 预订提供计费折扣，并且不会影响资源的运行时状态。
- 你可以取消预留实例。

![预留实例](./media/migrate-best-practices-costs/reserve.png)
*Azure 预留 VM*

**了解更多：**
- [了解](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations) Azure 预留。
- [阅读](https://azure.microsoft.com/pricing/reserved-vm-instances/#faq)预留实例 FAQ。
- [获取](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance) SQL Server Azure VM 的定价指南。


## <a name="best-practice-aggregate-cloud-spend-across-subscriptions"></a>最佳做法：合计各订阅之间的云支出

最终拥有多个 Azure 订阅是不可避免的。 例如，你可能需要用一个额外的订阅来为开发和生成划分界限，也有可能你的平台需要为每个客户端使用单独的订阅。 将跨所有订阅报告的数据聚合到一个平台，这是一项非常有用的功能。

要执行此操作，可以使用 Azure 成本管理 API。 然后，在将数据聚合到一个源（例如 Azure SQL）内后，你可以使用 Power BI 等工具访问聚合的数据。 你可以创建聚合订阅报表和粒度报表。 例如，对于需要主动了解成本管理的用户，你可以根据部门、资源组等要素创建具体的成本视图。不需要向他们提供对 Azure 账单数据的完全访问权限。

**了解更多：**

- [获取](https://docs.microsoft.com/azure/billing/billing-consumption-api-overview) Azure 使用情况 API 的概述。
- [了解](https://docs.microsoft.com/power-bi/desktop-connect-azure-consumption-insights)如何在 Power BI Desktop 中连接到 Azure 使用见解。
- [了解如何](https://docs.microsoft.com/azure/billing/billing-manage-access)使用基于角色的访问控制 (RBAC) 管理对 Azure 计费信息的访问。
 
## <a name="after-migration"></a>迁移之后

在成功迁移工作负荷后，花了数周时间收集使用情况数据，对资源成本有了清晰的了解。
- 在分析数据时，你可以开始为 Azure 资源组和资源生成预算基线。
- 了解云预算的开支分配情况后，便可以分析如何进一步降低成本。

本部分中的最佳做法包括：使用 Azure 成本管理进行成本预算和分析，监视资源并实现资源组预算，以及优化监视、存储和 VM。

## <a name="best-practice-use-azure-cost-management"></a>最佳做法：使用 Azure 成本管理

Microsoft 提供 Azure 成本管理来帮助你跟踪支出情况，如下所示：

- 帮助监视并控制 Azure 支出以及优化资源的使用。
- 查看你的整个订阅及其所有资源，并提出建议。
- 提供完整的 API，用于集成用于报告的外部工具和财务系统。
- 在一个统一的视图中跟踪资源使用情况和管理云成本。
- 提供丰富的运营和财务见解，帮助你做出明智的决策。

在“成本管理”中，你可以：


- **创建预算**：创建财务问责预算。
  - 可以考虑在特定时期（每月、每季度、每年）内使用或订阅的服务以及范围（订阅/资源组）。 例如，可以按月、按季度或按年创建 Azure 订阅预算。
  - 创建预算后，它会显示在成本分析中。 分析成本和支出所需的第一步是：对照当前支出查看预算。
  - 可在达到预算阈值时发送电子邮件通知。
  - 可以将成本管理数据导出到 Azure 存储，以供分析。

    ![成本的管理和预算](./media/migrate-best-practices-costs/budget.png)
    *Azure 成本管理预算*

- **执行成本分析**：获取成本分析，以便浏览和分析组织的成本，帮助你了解这些成本是如何产生的以及识别支出趋势。
  - 成本分析可供 EA 用户使用。
  - 可查看多个范围的成本分析数据，包括按部门、帐户、订阅或资源组查看。
  - 你可以获取一份显示当前月份总成本和每日累积成本的成本分析。 

    ![成本管理分析](./media/migrate-best-practices-costs/analysis.png)
    *Azure 成本管理分析*
- **获取建议**：获取说明如何优化和提高效率的顾问建议。


**了解更多：**

- [简要了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。
- [了解如何](https://docs.microsoft.com/azure/cost-management/cost-mgt-best-practices)通过 Azure 成本管理优化云投资。
- [了解如何](https://docs.microsoft.com/azure/cost-management/use-reports)使用 Azure 成本管理报表。
- [获取](https://docs.microsoft.com/azure/cost-management/tutorial-acm-opt-recommendations?toc=/azure/billing/TOC.json)关于根据建议优化成本的教程。
- [查看](https://docs.microsoft.com/rest/api/consumption/budgets) Azure 使用情况 API。

## <a name="best-practice-monitor-resource-utilization"></a>最佳做法：监视资源利用率

在 Azure 中，使用资源时付费，不使用资源时不付费，只需为使用的量付费。 就 VM 而言，则是在分配了 VM 时计费，解除分配 VM 后不会向你收费。 考虑到上述内容，你应当监视正在使用的 VM 并验证 VM 的大小。

- 持续评估 VM 工作负荷以确定基线。
- 例如，如果你的工作负荷在周一到周五上午 8 点至下午 6 点的使用率非常高，而在这些时间以外很少使用，便可以在高峰时间外降级 VM。 这可能意味着通过更改 VM 大小或使用虚拟机规模集来自动纵向扩展或缩减 VM。
- 一些公司会为 VM 设置日历，指明它们的可用时间，我们称之为“推迟”。 
- 除了监视 VM 外，还应监视 ExpressRoute 和虚拟网络网关等其他网络资源的使用情况，是否利用不足或利用过度。
- 可用使用许多 Microsoft 工具（包括 Azure 成本管理、Azure Monitor 和 Azure 顾问）监视 VM 使用情况。 还可使用第三方工具。  

**了解更多：**
- 简要了解 [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview)和 [Azure 顾问](https://docs.microsoft.com/azure/advisor/advisor-overview)。
- [获取](https://docs.microsoft.com/azure/advisor/advisor-cost-recommendations)顾问成本建议。
- 了解如何[根据建议优化成本](https://docs.microsoft.com/azure/cost-management/tutorial-acm-opt-recommendations?toc=/azure/billing/TOC.json)以及[防止意外的费用](https://docs.microsoft.com/azure/billing/billing-getting-started)。
- [了解](https://github.com/Azure/azure-quickstart-templates/tree/master/azure-resource-optimization-toolkit/) Azure 资源优化 (ARO) 工具包

## <a name="best-practice-implement-resource-group-budgets"></a>最佳做法：实现资源组预算

资源组通常用于表示成本边界。 Azure 团队继续开发增强的新方式，用于跟踪和分析此使用模式以及不同级别的资源支出（包括在资源组和资源级别创建预算的能力）。  

- 资源组预算有助于跟踪与资源组相关的成本。
- 达到或超过预算时，你可以触发警报并运行各种各样的 playbook。 

**了解更多：**

- [了解如何](https://docs.microsoft.com/azure/billing/billing-cost-management-budget-scenario)使用 Azure 预算管理成本。
- [按照教程](https://docs.microsoft.com/azure/cost-management/tutorial-acm-create-budgets?toc=/azure/billing/TOC.json)创建和管理 Azure 预算。


## <a name="best-practice-optimize-azure-monitor-retention"></a>最佳做法：优化 Azure Monitor 保留期

将资源移到 Azure 中并为它们启用诊断日志记录，会生成大量日志数据。 通常会将此日志数据发送到一个存储帐户，该帐户映射到 Log Analytics 工作区。

- 日志数据的保留期越长，数据就越多。
- 不一定所有日志数据都相等，一些资源生成的日志数据比其他资源多。
- 出于法规和符合性因素，某些资源的日志数据保留期可能比其他资源长。
- 对于优化日志存储成本和保留日志数据，你应当谨慎权衡、谋求平衡。
- 我们建议迁移完成后立即评估和设置日志记录，以免因保留无关紧要的日志而浪费资金。

**了解更多：**

- [了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs)监视使用情况和预估成本。
 
## <a name="best-practice-optimize-storage"></a>最佳做法：优化存储

如果在迁移前按照最佳做法选择了存储，可能会收到一些成效。 但可能仍然有一些额外的存储成本需要优化。 Blob 和文件随着时间的推移而变得陈旧。 数据可能不再可用，但根据法规要求，你可能需要将这些数据保留特定的一段时间。 于是，你可能不需要将它们存储在用于初始迁移的高性能存储中。

识别过时数据并将它们移动到费用更低的存储区域对每月存储预算和成本节省额有着巨大的影响。 Azure 提供许多方法，帮助你识别并存储这类过时数据。

- 利用常规用途 v2 存储的访问层，将不那么重要的数据从热访问层迁移到冷访问层和存档访问层。
- 使用 StorSimple 帮助移动基于自定义策略的过时数据。 

**了解更多：**
- [详细了解](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers)访问层。
- [简要了解](https://docs.microsoft.com/azure/azure-monitor/overview) StorSimple 以及 [StorSimple 定价](https://azure.microsoft.com/pricing/details/storsimple/)。

## <a name="best-practice-automate-vm-optimization"></a>最佳做法：自动执行 VM 优化

在云中运行 VM 的最终目标是尽量增加它占用的 CPU、内存和磁盘空间。 如果发现未优化的 VM 或者经常未使用 VM 的时段，就可以将它们关闭或通过 VM 规模集进行缩减。

可以使用 Azure 自动化、VM 规模集、自动缩减以及脚本化解决方案或第三方解决方案来优化 VM。 

**了解更多：**

- [了解如何](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-vertical-scale-reprovision)使用垂直自动缩放。
- [计划](https://azure.microsoft.com/updates/azure-devtest-labs-schedule-vm-auto-start/) VM 自动启动。
- [了解如何](https://docs.microsoft.com/azure/automation/automation-solution-vm-management)在 Azure 自动化中的空闲时间启动/停止 VM。
- [详细了解] [Azure 顾问](https://docs.microsoft.com/azure/advisor/advisor-overview)和 [Azure 资源优化 (ARO) 工具包](https://github.com/Azure/azure-quickstart-templates/tree/master/azure-resource-optimization-toolkit/)。

## <a name="best-practices-use-logic-apps-and-runbooks-with-budgets-api"></a>最佳做法：将逻辑应用和 runbook 与预算 API 结合使用

Azure 提供了一个 REST API，它有权访问你的租户账单信息。

- 你可以使用预算 API 集成外部系统和由你从 API 数据生成的指标触发的工作流。
- 可以将用量和资源数据拉取到你偏好的数据分析工具。
- Azure 资源用量和 RateCard API 可以帮助你准确预测及管理成本。
- 这些 API 是作为资源提供程序实现的，属于由 Azure 资源管理器公开的 API。
- 可以将预算 API 与 Azure 逻辑应用和 Runbook 集成。

**了解更多：**

- [详细了解](https://docs.microsoft.com/rest/api/consumption/budgets)预算 API。
- [使用计费 API](https://docs.microsoft.com/azure/billing/billing-usage-rate-card-overview) 洞察 Azure 使用情况。


## <a name="best-practice-implement-serverless-technologies"></a>最佳做法：实现无服务器技术

通常按原样迁移 VM 工作负荷，以避免造成故障。 VM 通常可以托管间歇性的任务，这些任务运行短短一段时间或交替运行数小时。 例如，运行 Windows 任务计划程序和 PowerShell 脚本等计划任务的 VM。 这些任务未运行时，你依然要承担 VM 和磁盘存储费用。

迁移后，你彻底查看了这些类型的任务，可能会考虑将它们迁移到 Azure Functions 或 Azure Batch 作业等无服务器技术。 使用此解决方案，你不再需要管理和维护 VM，额外地节省一些成本。 

**了解更多：**
- [了解](https://azure.microsoft.com/services/functions/) Azure Functions
- [详细了解](https://azure.microsoft.com/services/batch/) Azure Batch
  
## <a name="next-steps"></a>后续步骤 

查看其他最佳做法：

- [迁移后](migrate-best-practices-security-management.md)安全和管理的最佳做法。
- [迁移后](migrate-best-practices-networking.md)网络的最佳做法。

