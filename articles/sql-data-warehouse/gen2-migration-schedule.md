---
title: 将现有 Azure SQL 数据仓库迁移到 Gen2 | Microsoft Docs
description: 有关将现有数据仓库迁移到 Gen2 以及按区域迁移计划的说明。
services: sql-data-warehouse
author: mlee3gsd
ms.author: anumjs
ms.reviewer: jrasnick
manager: craigg
ms.assetid: 04b05dea-c066-44a0-9751-0774eb84c689
ms.service: sql-data-warehouse
ms.topic: article
ms.date: 04/03/2019
ms.openlocfilehash: a5d93a77652f540fde44b33963b13df04b45ecee
ms.sourcegitcommit: 60606c5e9a20b2906f6b6e3a3ddbcb6c826962d6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/01/2019
ms.locfileid: "64987943"
---
# <a name="upgrade-your-data-warehouse-to-gen2"></a>将数据仓库升级到 Gen2

Microsoft 帮助降低运行数据仓库的入门级的成本。  降低计算层能够处理非常高的查询现可用于 Azure SQL 数据仓库。 阅读完整公告[较低计算层支持第 2 代](https://azure.microsoft.com/blog/azure-sql-data-warehouse-gen2-now-supports-lower-compute-tiers/)。 下表中所述的区域中提供了新产品/服务。 对于支持的区域，现有的 Gen1 数据仓库可以通过以下任一方式升级到 Gen2：

- **自动升级过程：** 一旦服务可用的区域中，不要开始自动升级。  当自动升级在特定区域启动时，将在你选择的维护计划期间进行单独的 DW 升级。
- [**自升级到第 2 代：**](#self-upgrade-to-gen2)您可以控制何时升级通过执行自升级到第 2 代。 如果尚不支持你的区域，您可以从还原点还原直接到受支持的区域中的第 2 代实例。

## <a name="automated-schedule-and-region-availability-table"></a>自动计划和区域可用性表

下表按地区汇总了较低的 Gen2 计算层可用的时间以及启动自动升级的时间。 日期随时会变化。 可返回查看，了解所在地区的可用时间。

\* 表示该区域的特定时间表当前不可用。

| **区域** | **较低的 Gen2 可用** | **自动升级开始时间** |
|:--- |:--- |:--- |
| 澳大利亚东部 |可用 |2019 年 6 月 1日日 |
| 澳大利亚东南部 |可用 |2019 年 5 月 1 日 |
| 巴西南部 |可用 |2019 年 6 月 1日日 |
| 加拿大中部 |可用 |2019 年 6 月 1日日 |
| 加拿大东部 |\* |\* |
| 美国中部 |可用 |2019 年 6 月 1日日 |
| 中国东部 |\* |\* |
| 中国东部 2 |\* |\* |
| 中国北部 |\* |\* |
| 中国北部 2 |\* |\* |
| 东亚 |可用 |2019 年 6 月 1日日 |
| 美国东部 |可用 |2019 年 6 月 1日日 |
| 美国东部 2 |可用 |2019 年 6 月 1日日 |
| 法国中部 |\* |2019 年 6 月 1日日 |
| 德国中部 |\* |\* |
| 德国中西部 |2019 年 9 月 1日日|于 2020 年 1 月 2日日 |
| 印度中部 |可用 |2019 年 6 月 1日日 |
| 印度南部 |可用 |2019 年 6 月 1日日 |
| 日本东部 |可用 |2019 年 6 月 1日日 |
| 日本西部 |可用 |2019 年 5 月 1 日 |
| 韩国中部 |可用 |2019 年 6 月 1日日 |
| 韩国南部 |可用 |2019 年 5 月 1 日 |
| 美国中北部 |可用 |2019 年 5 月 1 日 |
| 北欧 |可用 |2019 年 6 月 1日日 |
| 美国中南部 |可用 |2019 年 6 月 1日日 |
| 东南亚 |可用 |2019 年 6 月 1日日 |
| 英国南部 |可用 2019 |2019 年 6 月 1日日 |
| 英国西部 |\*|\* |
| 美国中西部 |2019 年 9 月 2日日 |于 2020 年 1 月 2日日|
| 西欧 |可用 |2019 年 6 月 1日日 |
| 美国西部 |可用 |2019 年 6 月 1日日 |
| 美国西部 2 |可用 |2019 年 6 月 1日日 |

## <a name="automatic-upgrade-process"></a>自动升级过程

基于在上图的可用性，我们将进行计划进行自动的升级为 Gen1 实例。 为了避免数据仓库可用性发生任何意外中断，将在维护计划期间安排自动升级。 创建新的 Gen1 实例的功能将禁用正在自动升级到第 2 代的区域中。 有关计划的详细信息，请参阅[查看维护计划](viewing-maintenance-schedule.md)

升级过程将连接 （大约 5 分钟） 涉及简要放置，因为我们重启数据仓库。  重启数据仓库后，它将完全可用。 但是，可能会遇到性能下降，而升级过程将继续进行升级在后台中的数据文件。 性能下降的总时间将根据数据文件的大小而有所不同。

还可以通过在重启后使用更大的 SLO 和资源类在所有主列存储表上运行 [Alter Index rebuild](sql-data-warehouse-tables-index.md) 来加快数据文件升级过程。

> [!NOTE]
> Alter Index rebuild 是一项脱机操作，在重新生成完成之前，这些表将不可用。

## <a name="self-upgrade-to-gen2"></a>自行升级至 Gen2

您可以选择自升级现有的 Gen1 数据仓库上执行这些步骤。 如果选择自行升级，则必须在你所在区域的自动升级过程开始之前完成它。 这样做可确保避免导致冲突的自动升级任何风险。

进行自行升级时有两种选择。  你可以就地升级当前数据仓库，也可以将 Gen1 数据仓库还原为 Gen2 实例。

- [就地升级](upgrade-to-latest-generation.md) - 此选项会将现有的 Gen1 数据仓库升级到 Gen2。 升级过程将连接 （大约 5 分钟） 涉及简要放置，因为我们重启数据仓库。  重启数据仓库后，它将完全可用。 如果在升级过程中遇到问题，请打开[支持请求](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket)和引用"第 2 代升级"寻找可能的原因。
- [从还原点升级](sql-data-warehouse-restore.md) - 在当前 Gen1 数据仓库中创建用户定义的还原点，然后直接还原到 Gen2 实例。 现有的 Gen1 数据仓库将保持现状。 恢复完成后，Gen2 数据仓库将完全可用。  在已还原的 Gen2 实例上运行所有测试和验证过程后，可以删除原始 Gen1 实例。

   - 步骤 1：在 Azure 门户中，[创建用户定义的还原点](sql-data-warehouse-restore.md#create-a-user-defined-restore-point-using-the-azure-portal)。
   - 步骤 2：从用户定义的还原点还原时，将“性能级别”设置为首选的 Gen2 层。

升级过程继续在后台升级数据文件时，可能会经历一段时间的性能下降。 性能下降的总时间将根据数据文件的大小而有所不同。

要加快后台数据迁移进程，可以通过以更大的 SLO 和资源类对要查询的所有主要列存储表运行 [Alter Index rebuild](sql-data-warehouse-tables-index.md) 来立即强制数据移动。

> [!NOTE]
> Alter Index rebuild 是一项脱机操作，在重新生成完成之前，这些表将不可用。

如果有任何关于数据仓库的问题，请创建[支持请求](sql-data-warehouse-get-started-create-support-ticket.md)并引用“Gen2 升级”作为可能的原因。

有关详细信息，请参阅[升级到 Gen2](upgrade-to-latest-generation.md)。

## <a name="migration-frequently-asked-questions"></a>迁移常见问题

**问：Gen2 的成本与 Gen1 相同吗？**

- 答：是的。

**问：升级将如何影响我的自动化脚本？**

- 答：引用服务级别目标的任何自动化脚本都应更改为与 Gen2 等效项相对应。  详见[此处](upgrade-to-latest-generation.md#sign-in-to-the-azure-portal)。

**问：自行升级通常需要多长时间？**

- 答：可以就地升级或从还原点升级。  
   - 就地升级将导致数据仓库暂时暂停和继续。  数据仓库联机时，后台进程将继续。  
   - 如果要通过还原点进行升级，则需要更长时间，因为升级将完成整个还原过程。

**问：自动升级需要多长时间？**

- 答：升级的实际停机时间仅为暂停和恢复服务所需的时间，即 5 到 10 分钟。 在短暂的停机时间之后，后台进程将运行存储迁移。 后台进程的时间长度取决于数据仓库的大小。

**问：这种自动升级何时进行？**

- 答：在维护计划期间。 利用选择的维护计划可以最大限度地减少对业务的干扰。

**问：如果我的后台升级过程似乎被卡住了该怎么办？**

 - 答：开始为列存储表重新编制索引。 请注意，在此操作期间，为表重新编制索引将处于脱机状态。

**问：如果 Gen2 没有 Gen1 上的服务级别目标怎么办？**
- 答：如果在 Gen1 上运行 DW600 或 DW1200，建议分别使用 DW500c 或 DW1000c，因为 Gen2 提供的内存、资源和性能比 Gen1 更高。

**问：我可以禁用异地备份吗？**
- 答：不。 异地备份是一项企业功能，可在区域不可用时保留数据仓库的可用性。 若有其他疑虑，请创建[支持请求](sql-data-warehouse-get-started-create-support-ticket.md)。

**问：Gen1 和 Gen2 之间的 T-SQL 语法有区别吗？**

- 答：从 Gen1 到 Gen2 的 T-SQL 语言语法没有变化。

**问：Gen2 是否支持维护时段？**

- 答：是的。

**问：在我的区域升级后，我能够创建新的 Gen1 实例吗？**

- 答：不。 区域升级后，将禁用新 Gen1 实例的创建。

## <a name="next-steps"></a>后续步骤

- [升级步骤](upgrade-to-latest-generation.md)
- [维护时段](maintenance-scheduling.md)
- [资源运行状况监视器](https://docs.microsoft.com/azure/service-health/resource-health-overview)
- [开始迁移前查看](upgrade-to-latest-generation.md#before-you-begin)
- [就地升级和从还原点升级](upgrade-to-latest-generation.md)
- [创建用户定义的还原点](sql-data-warehouse-restore.md#restore-through-the-azure-portal)
- [了解如何还原到 Gen2](sql-data-warehouse-restore.md#restore-an-active-or-paused-database-using-the-azure-portal)
- [创建 SQL 数据仓库支持请求](https://go.microsoft.com/fwlink/?linkid=857950)
