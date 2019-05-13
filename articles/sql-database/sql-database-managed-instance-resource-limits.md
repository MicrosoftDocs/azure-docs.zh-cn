---
title: Azure SQL 数据库资源限制 - 托管实例 | Microsoft Docs
description: 本文概述托管实例的 Azure SQL 数据库资源限制。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: carlrab, jovanpop, sachinp
manager: craigg
ms.date: 02/27/2019
ms.openlocfilehash: e2068283414ef2fabb44e9876f6727cc0fe3530b
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233521"
---
# <a name="overview-azure-sql-database-managed-instance-resource-limits"></a>概述 Azure SQL 数据库托管实例资源限制

本文概述 Azure SQL 数据库托管实例资源限制，并说明如何创建提高默认区域订阅限制的请求。

> [!NOTE]
> 有关其他托管实例限制，请参阅[基于 vCore 的购买模型](sql-database-managed-instance.md#vcore-based-purchasing-model)和[“托管实例”服务层级](sql-database-managed-instance.md#managed-instance-service-tiers)。 有关支持的功能和 T-SQL 语句的差异，请参阅[功能差异](sql-database-features.md)和 [T-SQL 语句支持](sql-database-managed-instance-transact-sql-information.md)。

## <a name="instance-level-resource-limits"></a>实例级别的资源限制

托管实例具有取决于底层基础结构和体系结构的特性与资源限制。 这些限制取决于硬件代系和服务层级。

### <a name="hardware-generation-characteristics"></a>硬件代系特性

Azure SQL 数据库托管实例可部署在两代硬件 (Gen4 和 Gen5) 上。 各硬件代系具有不同的特性，如下表所述：

|   | **Gen4** | **Gen5** |
| --- | --- | --- |
| 硬件 | Intel E5-2673 v3 (Haswell) 2.4-GHz 处理器、附加的 SSD vCore = 1 PP（物理核心） | Intel E5-2673 v4 (Broadwell) 2.3-GHz 处理器、快速 NVMe SSD、vCore=1 LP（超线程） |
| vCore 数 | 8、16、24 个 vCore | 8、16、24、32、40、64、80 个 vCore |
| 内存 | 每个 vCore 7 GB | 每个 vCore 5.1 GB |
| 最大内存中 OLTP 内存 | 每个 vCore 3 GB | 每个 vCore 2.6 GB |
| 所需的最大实例存储 （常规用途） |  8 TB | 8 TB |
| 最大实例存储 （业务关键） | 1 TB | 1 TB、2 TB 或 4 TB，具体取决于核心数 |

### <a name="service-tier-characteristics"></a>服务层特征

托管实例具有两个服务层级：“常规用途”和“业务关键”。 这些层提供不同的功能，如下表所述：

| **功能** | **常规用途** | **业务关键** |
| --- | --- | --- |
| vCore 数目\* | 第 4 代：8、16、24<br/>第 5 代：8、16、24、32、40、64、80 | 第 4 代：8、16、24、32 <br/> 第 5 代：8、16、24、32、40、64、80 |
| 内存 | Gen4：56 GB-168 GB (7 GB/vCore)<br/>Gen5：40.8 GB-408 GB (5.1 GB/vCore) | Gen4：56 GB-168 GB (7 GB/vCore)<br/>Gen5：40.8 GB-408 GB (5.1 GB/vCore) |
| 最大实例存储大小 | 8 TB | Gen4：1 TB <br/> Gen5： <br/>- 8、16 个 vCore 1 TB<br/>- 24 个 vCore 2 TB<br/>- 4 TB（适用于 32、40、64、80 个 vCore） |
| 每个数据库的最大存储 | 由每个实例的最大存储大小决定 | 由每个实例的最大存储大小决定 |
| 每个实例的数据库数目上限 | 100 | 100 |
| 每个实例的数据库文件数目上限 | 最多 280 个 | 每个数据库 32,767 个文件 |
| 数据/日志 IOPS（近似） | 500 - 7,500（每个文件）<br/>\*[取决于文件大小](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage-performance#premium-storage-disk-sizes)| 11 K-110 K (1375年/vCore) |
| 日志吞吐量 | 22 MB/s（每个实例） | 3 MB/s（每个 vCore）<br/>最大为 48 MB/秒（每个实例）|
| 数据吞吐量（近似） | 100 - 250 MB/s（每个文件）<br/>\*[取决于文件大小](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage-performance#premium-storage-disk-sizes) | 24 - 48 MB/s（每个 vCore） |
| IO 延迟（近似） | 5-10 毫秒 | 1-2 毫秒 |
| 最大 tempDB 大小 | 192 - 1,920 GB（每个 vCore 为 24 GB） | 无约束 - 受最大实例存储大小限制 |

**注释**：

- 与最大存储大小限制进行比较的实例存储大小同时包括用户和系统数据库中的数据及日志文件大小。 可以使用 <a href="https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-master-files-transact-sql">sys.master_files</a> 系统视图来确定数据库使用的空间总量。 错误日志不会持久保存，不包括在大小中。 备份不包括在存储大小中。
- 吞吐量和 IOPS 还取决于未显式受到托管实例限制的页面大小。

## <a name="supported-regions"></a>支持的区域

托管实例只能在[支持的区域](https://azure.microsoft.com/global-infrastructure/services/?products=sql-database&regions=all)中创建。 如果要在当前不支持的区域中创建托管实例，可以[通过 Azure 门户发送支持请求](#obtaining-a-larger-quota-for-sql-managed-instance)。

## <a name="supported-subscription-types"></a>支持的订阅类型

当前仅支持在以下类型的订阅中部署托管实例：

- [企业协议 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)
- [即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [云服务提供商 (CSP)](https://docs.microsoft.com/partner-center/csp-documents-and-learning-resources)
- [Enterprise 开发/测试](https://azure.microsoft.com/offers/ms-azr-0148p/)
- [即用即付开发/测试](https://azure.microsoft.com/offers/ms-azr-0023p/)

> [!NOTE]
> 这是暂时性的限制。 未来会启用新的订阅类型。

## <a name="regional-resource-limitations"></a>区域资源限制

支持的订阅类型针对每个区域可包含有限的资源数。 托管实例针对每个 Azure 区域有两个默认限制，具体取决于订阅的类型：

- **子网限制**：在单一区域中部署托管实例的子网数上限。
- **实例数目限制**：可在单一区域中部署的实例数上限。

> [!Note]
> 这些限制是默认设置，而非技术限制。 限制可以增加按需通过创建特殊[在 Azure 门户中的支持请求](#obtaining-a-larger-quota-for-sql-managed-instance)如果您需要在当前区域中的多个托管实例。 或者，可以在另一个 Azure 区域中创建新的托管实例，而不需要发送支持请求。

下表显示支持的订阅所适用的默认区域限制：

|订阅类型| 托管实例子网数上限 | 实例数上限 |GP 托管实例数上限*|BC 托管实例数上限*|
| :---| :--- | :--- |:--- |:--- |
|即用即付|1*|4*|4*|1*|
|CSP |1*|4*|4*|1*|
|即用即付开发/测试|1*|4*|4*|1*|
|Enterprise 开发/测试|1*|4*|4*|1*|
|EA|3**|12**|12**|3**|

\* 可在一个子网中部署 1 个 BC 或 4 个 GP 实例，使子网中的“实例单位”总数绝不超过 4 个。

** 如果一个服务层级中没有任何实例，就会应用另一个服务层级中的实例数上限。 如果打算在相同的子网内混用 GP 和 BC 实例，请参考下一部分以了解允许的组合方式。 简单的规则是，子网总数不能超过 3 个，且实例单元的总数不能超过 12 个。



> [!IMPORTANT]
> 规划部署时，请考虑到业务关键 (BC) 实例（由于增加了冗余性）通常使用比常规用途 (GP) 实例多 4 倍的容量。 因此，进行计算时，1 个 GP 实例 = 1 个实例单位，而 1 个 BC 实例 = 4 个实例单位。 若要简化以默认限制为准的耗用量分析，请跨区域中所有部署了托管实例的子网汇总实例单位，并将结果与你的订阅类型的实例单位限制相比较。

## <a name="strategies-for-deploying-mixed-general-purpose-and-business-critical-instances"></a>部署混合的常规用途和业务关键实例的策略

[企业协议 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) 订阅可以组合使用 GP 和 BC 实例。 不过，在子网中的实例放置方面会有某些约束。

> [!Note]
> [即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)和[云服务提供商 (CSP)](https://docs.microsoft.com/partner-center/csp-documents-and-learning-resources) 订阅类型可以有一个业务关键实例，或最多 4 个常规用途实例。

以下示例介绍使用非空子网并混用 GP 与 BC 服务层级的部署案例。

|子网数|子网 1|子网 2|子网 3|
|:---|:---|:---|:---|
|1|1 个 BC 和最多 8 个 GP<br>2 个 BC 和最多 4 个 GP|不适用| 不适用|
|2|0 个BC，最多 4 个 GP|1 个 BC，最多 4 个 GP<br>2 个 BC，0 个 GP|不适用|
|2|1 个 BC，0 个 GP|0 个 BC，最多 8 个 GP<br>1 个BC，最多 4 个 GP|不适用|
|2|2 个 BC，0 个 GP|0 个BC，最多 4 个 GP|不适用|
|3|1 个 BC，0 个 GP|1 个 BC，0 个 GP|0 个BC，最多 4 个 GP|
|3|1 个 BC，0 个 GP|0 个 BC，最多 4 个 GP|0 个 BC，最多 4 个 GP|

## <a name="obtaining-a-larger-quota-for-sql-managed-instance"></a>获取更大的 SQL 托管实例配额

如果在当前区域中需要更多托管实例，可以使用 Azure 门户发送扩展配额的支持请求。
若要启动获取更大配额的过程，请执行以下操作：

1. 打开“帮助 + 支持”，单击“新建支持请求”。

   ![帮助和支持](media/sql-database-managed-instance-resource-limits/help-and-support.png)
2. 在新支持请求的“基本信息”选项卡上：
   - 对于“问题类型”，选择“服务和订阅限制(配额)”。
   - 对于“订阅”，请选择自己的订阅。
   - 对于“配额类型”，选择“SQL 数据库托管实例”。
   - 对于“支持计划”，选择自己的支持计划。

     ![问题类型配额](media/sql-database-managed-instance-resource-limits/issue-type-quota.png)

3. 单击“下一步”。
4. 在新支持请求的“问题”选项卡上：
   - 对于“严重性”，选择问题的严重性级别。
   - 对于“详细信息”，提供有关问题的其他信息，包括错误消息。
   - 对于“文件上传”，附加包含详细信息的文件（最多 4 MB）。

     ![问题详细信息](media/sql-database-managed-instance-resource-limits/problem-details.png)

     > [!IMPORTANT]
     > 有效的请求应包括：
     > - 需要提高订阅限制的区域
     > - 每个服务层级在配额增加后在现有的子网中所需的实例数目（如果需要扩展任何现有的子网）
     > - 所需的新子网数目，以及每个服务层级在新子网内的实例总数（如果需要在新子网中部署托管实例）。

5. 单击“下一步”。
6. 在新支持请求的“联系人信息”选项卡上，输入首选联系方式（电子邮件或电话）和联系人详细信息。
7. 单击**创建**。

## <a name="next-steps"></a>后续步骤

- 有关托管实例的详细信息，请参阅[什么是托管实例？](sql-database-managed-instance.md)。
- 有关定价信息，请参阅 [SQL 数据库托管实例定价](https://azure.microsoft.com/pricing/details/sql-database/managed/)。
- 若要了解如何创建第一个托管实例，请参阅[快速入门指南](sql-database-managed-instance-get-started.md)。
