---
title: 工作负荷重要性 | Microsoft Docs
description: 有关为 Azure SQL 数据仓库中的查询设置重要性的指导。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: workload management
ms.date: 05/01/2019
ms.author: rortloff
ms.reviewer: jrasnick
ms.openlocfilehash: 9c26bca66b0f82ea58d01d0eb8358f521168a799
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154143"
---
# <a name="sql-data-warehouse-workload-importance"></a>SQL 数据仓库工作负荷重要性

本文介绍工作负荷重要性如何影响 SQL 数据仓库请求的执行顺序。

## <a name="importance"></a>Importance

> [!Video https://www.youtube.com/embed/_2rLMljOjw8]

业务需求可能要求数据仓库工作负荷的重要性高于其他工作负荷。  假设存在这样一种情况：需要加载财务周期结束之前的任务关键型销售数据。  其他信息源（例如天气数据）的数据加载不需要遵守严格的 SLA。   为加载销售数据的请求设置高重要性，并为加载天气数据的请求设置低重要性，可以确保销售数据加载操作能够首先访问资源，并更快地完成。

## <a name="importance-levels"></a>重要性级别

有五个重要性级别：low、below_normal、normal、above_normal 和 high。  对于未设置重要性的请求，将为其分配默认级别 normal。  重要性级别相同的请求具有当前存在的相同计划行为。

## <a name="importance-scenarios"></a>重要性场景

除了上面所述的有关销售数据和天气数据的基本重要性场景以外，在其他某些场景中，工作负荷重要性也有助于满足数据处理和查询需求。

### <a name="locking"></a>锁定

访问读取和写入活动锁是自然争用的一个方面。  [分区切换](/azure/sql-data-warehouse/sql-data-warehouse-tables-partition)或 [RENAME OBJECT](/sql/t-sql/statements/rename-transact-sql) 等活动需要权限提升的锁。  如果不设置工作负荷重要性，SQL 数据仓库将会针对吞吐量进行优化。  针对吞吐量进行优化意味着，当正在运行的和排队的请求具有相同的锁定需求，并且资源可用时，排队的请求可能会绕过提前抵达请求队列的、具有更高锁定需求的请求。  将工作负荷重要性应用到具有较高锁定需求的请求后， 会先运行具有较高重要性的请求，然后再运行具有较低重要性的请求。

下面是一个示例：

Q1 正在运行，它从 SalesFact 中选择数据。
Q2 正在排队等待 Q1 完成。  该请求是在上午 9:00 提交的，目前正在尝试将新数据分区切换到 SalesFact。
Q3 是在上午 9:01 提交的，希望从 SalesFact 中选择数据。

如果 Q2 和 Q3 的重要性相同，而 Q1 仍在执行，则 Q3 将开始执行。 Q2 将继续等待 SalesFact 上的独占锁。  如果 Q2 的重要性高于 Q3，则 Q3 将等待 Q2 完成，然后它才能开始执行。

### <a name="non-uniform-requests"></a>非统一请求

另一个可以借助重要性满足查询需求的场景是提交了具有不同资源类的请求。  如前所述，在重要性相同的情况下，SQL 数据仓库会针对吞吐量进行优化。  如果混合大小的请求（例如 smallrc 或 mediumrc）已排队，则 SQL 数据仓库会选择最早抵达的、可用资源能够解决的请求。  如果应用了工作负荷重要性，则计划执行的下一个请求是重要性最高的请求。
  
请考虑 DW500c 中的以下示例：

Q1、Q2、Q3 和 Q4 正在运行 smallrc 查询。
Q5 是在上午 9:00 提交的，具有 mediumrc 资源类。
Q6 是在上午 9:01 提交的，具有 smallrc 资源类。

由于 Q5 是 mediumrc，因此需要两个并发槽。  Q5 需要等待两个正在运行的查询完成。  但是，当一个正在运行的查询 (Q1-Q4) 完成时，会紧接着计划 Q6，因为可以提供用于执行查询的资源。  如果 Q5 的重要性高于 Q6，则 Q6 会等待 Q5 运行，然后才能开始执行。

## <a name="next-steps"></a>后续步骤

若要开始创建分类器，请参阅[创建工作负荷分类器 (Transact SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-workload-classifier-transact-sql)。  有关 SQL 数据仓库工作负荷分类的详细信息，请参阅[SQL 数据仓库工作负荷分类](sql-data-warehouse-workload-classification.md)。  有关如何创建工作负荷分类器，请参阅快速入门[创建工作负荷分类器](quickstart-create-a-workload-classifier-tsql.md)。 参阅 [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql) 以查看查询和分配的重要性。
