---
title: Azure SQL 数据库发行说明 | Microsoft Docs
description: 了解 Azure SQL 数据库服务和 Azure SQL 数据库文档中的新功能和改进
services: sql-database
author: CarlRabeler
manager: craigg
ms.service: sql-database
ms.subservice: service
ms.devlang: ''
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: carlrab
ms.openlocfilehash: 923e475cd690902c61c2f89578c2c62effe4cd86
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406585"
---
# <a name="sql-database-release-notes"></a>SQL 数据库发行说明

本文列出了 SQL 数据库服务和 SQL 数据库文档中的新功能和改进。 有关 SQL 数据库服务的改进，另请参阅[SQL 数据库服务更新](https://azure.microsoft.com/updates/?product=sql-database)。 对其他 Azure 服务的改进，请参阅[服务更新](https://azure.microsoft.com/updates)。

## <a name="features-in-public-preview"></a>公共预览版中的功能

| Feature | 详细信息 |
| ---| --- |
| 无服务器计算层 | 有关信息，请参阅[SQL 数据库无服务器 （预览版）](sql-database-serverless.md)。|
| 灵活的数据库作业 | 有关信息，请参阅[创建、 配置和管理弹性作业](elastic-jobs-overview.md) |
| 弹性事务 | [跨云数据库的分布式事务](sql-database-elastic-transactions-overview.md) |
| 弹性查询 | 有关信息，请参阅[弹性查询概述](sql-database-elastic-query-overview.md) |
| 与托管实例的复制 |有关信息，请参阅[在 Azure SQL 数据库托管的实例数据库中配置复制](replication-with-sql-database-managed-instance.md)|
| 与托管实例的实例排序规则 |有关信息，请参阅[使用 PowerShell 与 Azure 资源管理器模板在 Azure SQL 数据库中创建的托管的实例](./scripts/sql-managed-instance-create-powershell-azure-resource-manager-template.md)|
| R services/机器学习与单一数据库和弹性池 |有关信息，请参阅[Azure SQL 数据库中的机器学习服务](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#machine-learning-services-in-azure-sql-database)|
| 使用单一数据库和弹性池的加速的数据库恢复 | 有关信息，请参阅[加速数据库恢复](sql-database-accelerated-database-recovery.md)|
| 数据发现和分类  |有关信息，请参阅[Azure SQL 数据库和 SQL 数据仓库数据发现和分类](sql-database-data-discovery-and-classification.md)|
| 透明数据加密 (TDE 使用自带带密钥 (BYOK) 与托管实例) |有关信息，请参阅[Azure SQL 透明数据加密，使用 Azure Key Vault 中客户托管密钥：自带密钥支持](transparent-data-encryption-byok-azure-sql.md)|
| 重新创建与托管实例的删除的数据库 |有关信息，请参阅[重新创建 Azure SQL 托管实例中删除数据库](https://medium.com/azure-sqldb-managed-instance/re-create-dropped-databases-in-azure-sql-managed-instance-dc369ed60266)|
| 与托管实例威胁检测 |有关信息，请参阅[配置威胁检测的 Azure SQL 数据库托管实例](sql-database-managed-instance-threat-detection.md)|
| 与单一数据库的超大规模服务层 |有关信息，请参阅[达 100 TB 的超大规模服务层](sql-database-service-tier-hyperscale.md)|
| 在 Azure 门户中的查询编辑器 |有关信息，请参阅[使用 Azure 门户的 SQL 查询编辑器进行连接和查询数据](sql-database-connect-query-portal.md)|
|估计非重复计数|有关信息，请参阅[近似 Count Distinct](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#approximate-query-processing)|
|行存储 （在兼容性级别 150） 的批处理模式|有关信息，请参阅[行存储在批处理模式](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#batch-mode-on-rowstore)|
|（在兼容性级别 150） 的内存授予反馈 （行模式）|有关信息，请参阅[内存授予反馈 （行模式）](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#row-mode-memory-grant-feedback)|
|表变量延迟编译 （在兼容性级别 150）|有关信息，请参阅[表变量延迟的编译](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing#table-variable-deferred-compilation)|
|SQL Analytics|有关信息，请参阅[Azure SQL Analytics](../azure-monitor/insights/azure-sql.md)|
| 托管实例的时区支持|有关详细信息，请参阅[中 Azure SQL 数据库托管实例所在的时区](sql-database-managed-instance-timezone.md)|
|||

## <a name="may-2019"></a>2019 年 5

### <a name="service-improvements"></a>服务改进

| 服务改进 | 详细信息 |
| --- | --- |
|“超大规模”服务层级| 有关详细信息，请参阅[达 100 TB 的超大规模服务层](sql-database-service-tier-hyperscale.md)。|
|无服务器计算层| 有关详细信息，请参阅[SQL 数据库无服务器 （预览版）](sql-database-serverless.md)。|


## <a name="april-2019"></a>2019 年 4 月

### <a name="service-improvements"></a>服务改进

| 服务改进 | 详细信息 |
| --- | --- |
| 托管实例的公共终结点 | 有关详细信息，请参阅[使用 Azure SQL 数据库托管实例安全地与公共终结点](sql-database-managed-instance-public-endpoint-securely.md)
| 托管实例的时区支持 | 有关详细信息，请参阅[中 Azure SQL 数据库托管实例 （预览版） 所在的时区](sql-database-managed-instance-timezone.md)

### <a name="documentation-improvements"></a>文档改进

| 文档改进 | 详细信息 |
| --- | --- |
| 托管实例的公共终结点 | 有关详细信息，请参阅[使用 Azure SQL 数据库托管实例安全地与公共终结点](sql-database-managed-instance-public-endpoint-securely.md)
| 托管实例的时区支持 | 有关详细信息，请参阅[中 Azure SQL 数据库托管实例 （预览版） 所在的时区](sql-database-managed-instance-timezone.md)

## <a name="march-2019"></a>2019 年 3 月

### <a name="service-improvements"></a>服务改进

| 服务改进 | 详细信息 |
| --- | --- |
| 正式发布：Azure SQL 数据库的读取扩展支持 | 有关详细信息，请参阅[读取横向扩展](sql-database-read-scale-out.md)|
| &nbsp; |

### <a name="documentation-improvements"></a>文档改进

| 文档改进 | 详细信息 |
| --- | --- |
| 托管实例的时区支持|有关详细信息，请参阅[中 Azure SQL 数据库托管实例所在的时区](sql-database-managed-instance-timezone.md)|
| 添加了单一数据库的日志限制|有关详细信息，请参阅[单一数据库 vCore 资源限制](sql-database-vcore-resource-limits-single-databases.md)。|
| 添加了弹性池和共用数据库的日志限制|有关详细信息，请参阅[弹性池 vCore 资源限制](sql-database-vcore-resource-limits-elastic-pools.md)。|
| 添加了事务日志速率调控| 为[事务日志速率调控](sql-database-resource-limits-database-server.md#transaction-log-rate-governance)添加了新内容。|
| 更新了单一数据库和弹性池的 PowerShell 示例，使其能够使用 az.sql 模块 | 有关详细信息，请参阅[单一数据库和弹性池的 PowerShell 示例](sql-database-powershell-samples.md#single-database-and-elastic-pools)。|
| &nbsp; |

## <a name="february-2019"></a>2019 年 2 月

### <a name="service-improvements"></a>服务改进

| 服务改进 | 详细信息 |
| --- | --- |
|创建可恢复联机索引现已公开发布| 有关详细信息，请参阅[Create Index](https://docs.microsoft.com/sql/t-sql/statements/create-index-transact-sql)。|
|托管的实例支持的改进的路由表| 有关详细信息，请参阅[网络要求](sql-database-managed-instance-connectivity-architecture.md#network-requirements)。|
|托管实例支持数据库重命名 | 有关更多详细信息，请参阅 [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-mi-current) 和 [sp_rename](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql) 语法。|
|SQL 数据库充当流分析的参考数据源。 | 有关详细信息，请参阅[流分析](https://azure.microsoft.com/services/stream-analytics/)。|
|数据迁移帮助增加了对托管实例支持。 |有关详细信息，请参阅[DMA 中新增](https://docs.microsoft.com/sql/dma/dma-whatsnew)。|
|SQL Server 迁移助手添加目标准备情况评估托管实例的支持。 | 有关详细信息，请参阅[SQL Server Migration Assistant](https://docs.microsoft.com/sql/ssma/sql-server-migration-assistant)。
|数据迁移服务支持从 Amazon RDS 迁移到托管实例 | 有关详细信息，请参阅[教程：RDS SQL Server 迁移到 Azure SQL 数据库或 Azure SQL 数据库上联机托管的实例使用 DMS](../dms/tutorial-rds-sql-server-azure-sql-and-managed-instance-online.md)。|
| &nbsp; |

### <a name="documentation-improvements"></a>文档改进

| 文档改进 | 详细信息 |
| --- | --- |
|添加托管实例部署选项说明|更新很多文章，以阐明适用于单个数据库、 弹性池和托管的实例的部署选项。 |
|更新了基于 DTU 的购买模型的 tempdb 大小 | 有关详细信息，请参阅 [SQL 数据库中的 Tempdb 数据库](https://docs.microsoft.com/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database)。|
|更新导入和导出与托管的实例支持的 bacpac 文件| 有关详细信息，请参阅[从 BACPAC 导入](sql-database-import.md)并[导出到 BACPAC](sql-database-export.md)。 |
| &nbsp; |


## <a name="january-2019"></a>2019 年 1 月

### <a name="service-improvements"></a>服务改进

| 服务改进 | 详细信息 |
| --- | --- |
| 计算资源的其他粒度选项 | [单一数据库](sql-database-vcore-resource-limits-single-databases.md)和[弹性池](sql-database-vcore-resource-limits-elastic-pools.md)的“常规用途”和“业务关键”服务层现在有更多细致的计算选项。|
| 查看在 Azure 门户中的托管实例的审核记录 | 查看[审核记录的托管实例](sql-database-managed-instance-auditing.md)在 Azure 门户现在支持。|
| 高级威胁检测功能已重命名为“高级数据安全” | 高级威胁检测功能重命名为[高级数据安全](sql-advanced-threat-protection.md)单一数据库、 弹性池和托管的实例。 |
| &nbsp; |

### <a name="documentation-improvements"></a>文档改进

| 文档改进 | 详细信息 |
| --- | --- |
| 托管的实例和事务复制 | 添加了有关使用文章[具有托管实例的事务复制](replication-with-sql-database-managed-instance.md) |
| 添加了 Azure AD 与托管的实例教程 | 这[托管实例的 Azure AD](sql-database-managed-instance-aad-security-tutorial.md)教程，说明您必须配置和测试管理实例安全性使用 Azure AD 登录名。 |
| 更新了使用 Transact-SQL 脚本进行的作业自动化的内容 | 更新，并且阐明了有关使用的内容[使用 Transact SQL 脚本的作业自动化](sql-database-job-automation-overview.md)单一数据库、 弹性池和托管的实例 |
| 更新托管实例的安全内容 | 更新并阐明的内容[托管实例的安全模型](sql-database-security-overview.md)，并对照使用单一数据库和弹性池的安全模型 |
| 刷新了所有快速入门和教程 | [文档](https://docs.microsoft.com/azure/sql-database)中的所有快速入门和教程都已根据 Azure 门户中的更改进行更新和刷新 |
| 添加了快速入门概述指南 | 添加的快速入门概述指南[单一数据库](sql-database-quickstart-guide.md)以及[托管实例](sql-database-managed-instance-quickstart-guide.md) |
| 添加了 SQL 数据库术语表 | 此[术语表](sql-database-glossary-terms.md)文章提供了一个最终列表，其中包含 SQL 数据库术语以及在上下文中解释术语的主要概念页的链接。 |
| &nbsp; |

## <a name="contribute-to-content-improvement"></a>参与内容改进

Azure SQL 文档集是开源的。 采用开源方式有几项优势：

- 开源存储库公开进行计划，可以通过反馈了解用户最需要什么样的文档。
- 开源存储库公开进行评审，可以在初次发布时就发布最有用的内容。
- 开源存储库公开进行更新，更容易持续改进内容。

若要提供 Azure SQL 数据库文档内容，请参阅[Microsoft Docs 参与者指南概述](https://docs.microsoft.com/contribute/)。 上的用户体验[docs.microsoft.com](https://docs.microsoft.com/)集成[GitHub](https://github.com/)直接要更轻松的工作流。 首先[编辑正在查看的文档](https://docs.microsoft.com/contribute/#quick-edits-to-existing-documents)。 或者，通过帮助[查看新主题](https://docs.microsoft.com/contribute/#review-open-prs)，或[创建质量问题](https://docs.microsoft.com/contribute/#create-quality-issues)。
