---
title: Azure SQL 数据库购买模型 | Microsoft Docs
description: 了解 Azure SQL 数据库服务中提供的数据库的购买模型。
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
manager: craigg
ms.date: 04/26/2019
ms.openlocfilehash: f17df53c1ea77bf99ab86329fe914d058eb00f64
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072695"
---
# <a name="choose-between-the-vcore-and-the-dtu-purchasing-model"></a>VCore 与 DTU 的购买模型之间选择

使用 Azure SQL 数据库，可以轻松购买适合你的性能和成本要求的完全托管的 PaaS 数据库引擎。 你可以根据 Azure SQL 数据库的部署模型选择适合你的需求的购买模型：

- [基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)（推荐）。 此购买模型提供了预配的计算层和无服务器 （预览版） 计算层之间进行选择。 使用预配的计算层中，选择工作负荷的确切始终预配的计算量。 使用无服务器计算层中，您配置自动缩放的计算可配置的计算范围内。 凭借此计算层，还可以选择自动暂停和恢复的数据库根据工作负荷活动。 每个单位的时间的 vCore 单位价格较低的预配的计算层不是在无服务器计算层中。
- [基于 DTU 的购买模型](sql-database-service-tiers-dtu.md)。 此购买模型提供捆绑的常见工作负载均衡的计算和存储包。

Azure SQL 数据库部署模型中提供了不同的购买模型：

- [Azure SQL 数据库](sql-database-technical-overview.md)中的[单一数据库](sql-database-single-databases-manage.md)和[弹性池](sql-database-elastic-pool.md)部署选项提供[基于 DTU 的购买模型](sql-database-service-tiers-dtu.md)和[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)。
- Azure SQL 数据库中的[托管实例](sql-database-managed-instance.md)部署选项仅提供[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)。


- [超大规模服务层](sql-database-service-tier-hyperscale.md)目前适用于使用单一数据库[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)。

下面的表和图表比较和对比的 vCore 和 DTU 的购买模型。

|**购买模型**|**说明**|**最适用于**|
|---|---|---|
|基于 DTU 的模型|此模型基于计算、存储和 IO 资源的捆绑度量。 单一数据库的计算大小以数据库事务单位 (DTU) 表示，弹性池则以弹性数据库事务单位 (eDTU) 表示。 有关 DTU 和 eDTU 的详细信息，请参阅[什么是 DTU 和 eDTU？](sql-database-purchase-models.md#dtu-based-purchasing-model)|最适合希望获得简单预配置资源选项的客户。|
|基于 vCore 的模型|此模型允许单独选择计算和存储资源。 基于 vCore 的购买模型还允许使用[适用于 SQL Server 的 Azure 混合权益](https://azure.microsoft.com/pricing/hybrid-benefit/)来节省成本。|最适合注重灵活性、控制度和透明度的客户。|
||||  

![定价模型](./media/sql-database-service-tiers/pricing-model.png)

## <a name="compute-costs"></a>计算成本

### <a name="provisioned-compute-costs"></a>预配的计算成本

在预配的计算层中，计算成本反映针对应用程序预配的总计算容量。  在“业务关键”服务层级中，我们会自动分配至少 3 个副本。 为了反映计算资源的此额外分配，在基于 vCore 的购买模型中，“业务关键”服务层级的价格比“常规用途”服务层级的价格高约 2.7 倍。 出于相同的原因，“业务关键”服务层级中更高的每 GB 存储价格反映了 SSD 存储的高 IO 和低延迟。 同时，备份存储成本在这两个服务层级之间并无不同，因为在这两种情况下，我们都使用某类标准存储。

### <a name="serverless-compute-costs"></a>无服务器计算成本

无服务器计算层中，请参阅[SQL 数据库无服务器 （预览版）](sql-database-serverless.md)计算的计算能力的定义方式的说明和成本。

## <a name="storage-costs"></a>存储成本

不同存储类型的计费方式各不相同。 对于数据存储，你要根据所选的最大数据库或池大小支付预配存储的费用。 除非减小或增大该最大值，否则费用不会变化。 备份存储与实例的自动备份相关联，并动态分配。 延长备份保留期会使实例使用的备份存储空间增大。

默认情况下，数据库的 7 天自动备份会复制到 RA-GRS 标准 blob 存储。 存储由每周完整备份、每日差异备份和 5 分钟复制一次的事务日志备份使用。 事务日志的大小取决于数据库的变化率。 提供与 100% 数据库大小相等的最小存储量，不收取额外费用。 超出此部分的其他备份存储用量将以 GB 为单位每月进行收费。

有关存储价格的详细信息，请参阅[定价](https://azure.microsoft.com/pricing/details/sql-database/single/)页。

## <a name="vcore-based-purchasing-model"></a>基于 vCore 的购买模型

虚拟核心表示通过一个选项提供的逻辑 CPU，该选项允许在硬件的层代和硬件的物理特性（例如，核心数、内存、存储大小）之间进行选择。 基于 vCore 的购买模型提供使用单项资源的灵活性、控制度和透明度，并提供简单明了的方法将本地工作负荷要求转换到云。 此模型允许根据工作负荷需求来选择计算、内存和存储。 在基于 vCore 的购买模型中，可以为[单一数据库](sql-database-single-database-scale.md)、[弹性池](sql-database-elastic-pool.md)和[托管实例](sql-database-managed-instance.md)选择[常规用途](sql-database-high-availability.md#basic-standard-and-general-purpose-service-tier-availability)或[业务关键](sql-database-high-availability.md#premium-and-business-critical-service-tier-availability)服务层级。 对于单一数据库，还可以选择[超大规模服务层](sql-database-service-tier-hyperscale.md)。

使用基于 vCore 的购买模型，可以单独选择计算和存储资源、匹配本地性能，以及优化价格。 在基于 vCore 的购买模型中，客户的费用包括：

- 计算（服务层级 + vCore 数目和内存量 + 硬件层代）
- 数据和日志存储的类型与数量
- 备份存储 (RA-GRS)

> [!IMPORTANT]
> 计算、IO、数据和日志存储按数据库或弹性池收费。 备份存储按每个数据库收费。 有关托管实例费用的详细信息，请参阅[托管实例](sql-database-managed-instance.md)。
> 区域限制：有关受支持区域的当前列表，请参阅[可用产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/?products=sql-database&regions=all)。 如果要在当前不支持的区域中创建托管实例，可以[通过 Azure 门户发送支持请求](sql-database-managed-instance-resource-limits.md#obtaining-a-larger-quota-for-sql-managed-instance)。
.

如果单一数据库或弹性池消耗的 DTU 超过 300 个，则转换为基于 vCore 的购买模型可以降低成本。 如果决定进行转换，可以使用所选的 API 或 Azure 门户进行转换，无需停机。 但是，转换不是必需的并且不会自动完成。 如果基于 DTU 的购买模型可以满足性能和业务要求，应继续使用它。 如果决定从基于 DTU 的购买模型转换为基于 vCore 的购买模型，请使用以下经验法则来选择计算大小：

- 标准层中的每 100 个 DTU 至少需要常规用途层中的 1 个 vCore
- 高级层中的每 125 个 DTU 至少需要业务关键层中的 1 个 vCore

## <a name="dtu-based-purchasing-model"></a>基于 DTU 的购买模型

数据库事务单位 (DTU) 表示 CPU、内存、读取和写入的混合度量。 基于 DTU 的购买模型提供了一组预配置的计算资源套件和所包含存储以促成不同级别的应用程序的性能。 偏爱预配置套件简易性和每月定价付款的客户，可能会发现基于 DTU 的模型更适合其需求。 在基于 DTU 的购买模型中，客户可为[单一数据库](sql-database-single-database-scale.md)和[弹性池](sql-database-elastic-pool.md)选择“基本”、“标准”或“高级”服务层级。 此购买模型在[托管实例](sql-database-managed-instance.md)中不可用。

### <a name="database-transaction-units-dtus"></a>数据库事务单位 (DTU)

对于[服务层级](sql-database-single-database-scale.md)内特定计算大小的单一数据库，Microsoft 保证该数据库具有一定级别的资源（独立于 Azure 云中的任何其他数据库），从而提供可预测的性能级别。 此资源量是以若干数据库事务单位或 DTU 计算的，是计算资源、存储资源和 IO 资源的捆绑度量值。 这些资源之间的比例最初由 [OLTP 基准工作负荷](sql-database-benchmark-overview.md)确定，该工作负荷是一种典型的真实 OLTP 工作负荷。 工作负荷超过任何以上资源量时，吞吐量将受到限制，从而导致性能下降和超时。 工作负荷使用的资源不会影响 Azure 云中其他 SQL 数据库可用的资源，而其他工作负荷使用的资源也不会影响用户自己的 SQL 数据库可用的资源。

![边界框](./media/sql-database-what-is-a-dtu/bounding-box.png)

要了解 Azure SQL 数据库在不同计算大小和服务层级之间的资源相对数量，DTU 将最为有用。 例如，提高数据库的计算大小可使 DTU 加倍，这等同于使该数据库可用的资源集增加一倍。 例如，具有 1750 个 DTU 的高级 P11 数据库提供的 DTU 计算能力是具有 5 个 DTU 的基本数据库的 350 倍。  

若要更深入地了解工作负荷的资源 (DTU) 消耗，请使用[查询性能见解](sql-database-query-performance.md)：

- 按 CPU/持续时间/执行计数确定热门查询，可以对这些查询进行调整来提高性能。 例如，IO 密集型查询可能会受益于使用[内存中优化技术](sql-database-in-memory.md)，以便通过特定的服务层级和计算大小更好地利用可用内存。
- 深入了解查询详情，查看其文本和资源使用历史记录。
- 访问性能优化建议可显示 [SQL 数据库顾问](sql-database-advisor.md)执行的操作。

### <a name="elastic-database-transaction-units-edtus"></a>弹性数据库事务单位 (eDTU)

可将数据库放在 SQL 数据库服务器上的[弹性池](sql-database-elastic-pool.md)中，该服务器在这些数据库之间共享资源池，而不必为 SQL 数据库提供一组始终可用的专用资源 (DTU)，因为有时可能并不需要。 弹性池中的共享资源用弹性数据库事务单位或 eDTU 度量。 弹性池提供一种简单的低成本高效益的解决方案，用于管理使用模式变化很大且不可预测的多个数据库的性能目标。 弹性池可保证不出现一个数据库使用池中所有资源的情况，同时确保池中的每个数据库始终可以使用最少量的必需资源。

为池提供的 eDTU 的数量和价格是固定的。 在弹性池中，单个数据库能够在配置的边界范围内灵活自动缩放。 负载较重的数据库消耗较多的 eDTU 来满足需求。 负载较轻的数据库消耗较少的 eDTU。 无负载的数据库不消耗任何 eDTU。 为整个池而不是每个数据库预配资源可以简化管理任务，使池的预算可预测。

可以对现有池增加额外的 eDTU，这不会造成数据库关闭，也不会影响池中的数据库。 同样，随时可以从现有池中删除不再需要的额外 eDTU。 可以在池中添加或删除数据库，或者限制重负载数据库的 eDTU 用量，以便为其他数据库预留 eDTU。 如果可以预见到某个数据库的资源利用率不高，可将其移出池，并使用所需的可预测资源量将它配置为单一数据库。

### <a name="determine-the-number-of-dtus-needed-by-a-workload"></a>确定工作负荷所需的 DTU 数

如果打算将现有的本地或 SQL Server 虚拟机工作负荷迁移到 Azure SQL 数据库，可以使用 [DTU 计算器](https://dtucalculator.azurewebsites.net/)来估计所需的 DTU 数。 对于现有的 Azure SQL 数据库工作负荷，可以使用[查询性能见解](sql-database-query-performance.md)来了解数据库资源使用量 (DTU)，更深入地了解如何优化工作负荷。 也可以使用 [sys.dm_db_ resource_stats](https://msdn.microsoft.com/library/dn800981.aspx) DMV 查看最近一小时的资源消耗。 目录视图 [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx) 也可显示最近 14 天的资源消耗，不过，五分钟平均值的准确性较低。

### <a name="workloads-that-benefit-from-an-elastic-pool-of-resources"></a>受益于资源弹性池的工作负荷

池适合具有特定使用模式的大量数据库。 对于给定的数据库，此模式的特征是低平均使用量与相对不频繁的使用高峰。 SQL数据库自动评估现有 SQL 数据库服务器中数据库的历史资源使用率，并在 Azure 门户中推荐适当的池配置。 有关详细信息，请参阅[何时使用弹性池？](sql-database-elastic-pool.md)

## <a name="purchase-models-frequently-asked-questions-faq"></a>购买模型： 常见问题 (FAQ)

### <a name="do-i-need-to-take-my-application-offline-to-convert-from-a-dtu-based-database-to-a-vcore-based-service-tier"></a>是否需要将应用程序脱机，才能将基于 DTU 的数据库转换为基于 vCore 的服务层级

新服务层级提供简单的在线转换方式，与现有“标准”和“高级”服务层级间的升级和降级过程类似。 可以使用 Azure 门户、PowerShell、Azure CLI、T-SQL 或 REST API 启动此转换。 请参阅[管理单一数据库](sql-database-single-database-scale.md)和[管理弹性池](sql-database-elastic-pool.md)。

### <a name="can-i-convert-a-database-from-a-service-tier-using-the-vcore-based-purchase-to-a-service-tier-using-the-dtu-based-purchasing-model"></a>是否可以将数据库从使用基于 vCore 的购买模型的服务层级转换为使用基于 DTU 的购买模型的服务层级？

是的，可以使用 Azure 门户、PowerShell、Azure CLI、T-SQL 或 REST API 轻松将数据库转换为支持的任何性能目标。 请参阅[管理单一数据库](sql-database-single-database-scale.md)和[管理弹性池](sql-database-elastic-pool.md)。

## <a name="next-steps"></a>后续步骤

- 若要了解基于 vCore 的购买模型，请参阅[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)
- 有关基于 DTU 的购买模型，请参阅[基于 DTU 的购买模型](sql-database-service-tiers-dtu.md)。
