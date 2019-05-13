---
title: 优化 autovacuum 上 Azure Database for PostgreSQL-单个服务器
description: 本指南介绍了如何来优化 autovacuum 上 Azure Database for PostgreSQL-单个服务器
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: fb1ab9525974601a8b8c22ccc44e2cf37baf21a1
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65069121"
---
# <a name="optimize-autovacuum-on-an-azure-database-for-postgresql---single-server"></a>优化 autovacuum 上 Azure Database for PostgreSQL-单个服务器
本文介绍如何在 Azure Database for PostgreSQL 服务器中有效优化 autovacuum。

## <a name="overview-of-autovacuum"></a>Autovacuum 概述
PostgreSQL 使用多版本并发控制 (MVCC) 实现更高的数据库并发性。 每次更新都会导致插入和删除，而每次删除都会导致要删除的行被软标记。 软标记用于标识随后要清除的死元组。 为执行这些任务，PostgreSQL 将运行一个清扫作业。

清扫作业可以手动触发或自动触发。 在数据库进行大量更新或删除操作时，死元组会更多。 数据库空闲时，死元组较少。 数据库负载过大时，需要更频繁地运行清扫作业，因此手动运行清扫作业会有所不便。

可以配置 autovacuum 并从优化中获益。 PostgreSQL 附带的默认值尝试确保产品在所有类型的设备上正常运行。 这些设备包括 Raspberry Pi。 理想的配置值取决于：
- 可用资源总数，例如 SKU 和存储大小。
- 资源使用情况。
- 单独对象特征。

## <a name="autovacuum-benefits"></a>Autovacuum 的优势
如果不时常运行清扫作业，累积的死元组可能会导致以下问题：
- 数据膨胀，例如数据库和表变大。
- 更大的非最优索引。
- I/O 增加。

## <a name="monitor-bloat-with-autovacuum-queries"></a>使用 autovacuum 查询监视膨胀情况
以下示例查询的目的是确定名为 XYZ 的表中的非活动元组和活动元组的数量：
 
    'SELECT relname, n_dead_tup, n_live_tup, (n_dead_tup/ n_live_tup) AS DeadTuplesRatio, last_vacuum, last_autovacuum FROM pg_catalog.pg_stat_all_tables WHERE relname = 'XYZ' order by n_dead_tup DESC;'

## <a name="autovacuum-configurations"></a>Autovacuum 配置
控制 autovacuum 的配置参数基于两个关键问题的答案：
- 何时应该启动它？
- 启动它以后应清除多少内容？

下面是一些可以基于以上问题更新的 autovacuum 配置参数以及一些指导信息。

参数|描述|默认值
---|---|---
autovacuum_vacuum_threshold|指定在任一表中触发清扫操作所需的已更新或已删除元组的最小数量。 默认值为 50 个元组。 只能在 postgresql.conf 文件中或服务器命令行上设置此参数。 若要替代单个表的设置，请更改表存储参数。|50
autovacuum_vacuum_scale_factor|指定在决定是否触发清扫操作时要添加到 autovacuum_vacuum_threshold 的表大小的占比。 默认值为 0.2，即表大小的 20%。 只能在 postgresql.conf 文件中或服务器命令行上设置此参数。 若要替代单个表的设置，请更改表存储参数。|百分之 5
autovacuum_vacuum_cost_limit|指定自动清扫操作中使用的成本限制值。 如果指定为 -1（默认值），则会使用常规的 vacuum_cost_limit 值。 如果存在多个辅助角色，则该值会按比例分配给这些运行 autovacuum 的辅助角色。 每个辅助角色的限制的总和不超过此变量的值。 只能在 postgresql.conf 文件中或服务器命令行上设置此参数。 若要替代单个表的设置，请更改表存储参数。|-1
autovacuum_vacuum_cost_delay|指定自动清扫操作中使用的成本延迟值。 如果指定为 -1，则会使用常规的 vacuum_cost_delay 值。 默认值为 20 毫秒。 只能在 postgresql.conf 文件中或服务器命令行上设置此参数。 若要替代单个表的设置，请更改表存储参数。|20 ms
autovacuum_nap_time|指定任一给定数据库上运行的 autovacuum 之间的最小延迟。 守护程序会在每一轮中检查数据库并根据需要为数据库中的表发出清扫和分析命令。 延迟以秒为单位，默认值为一分钟 (1 min)。 只能在 postgresql.conf 文件中或服务器命令行上设置此参数。|15 s
autovacuum_max_workers|指定在任何时候可能运行的 autovacuum 进程（不是 autovacuum 启动器）的最大数量。 默认值为三个。 只能在服务器启动时设置此参数。|3

若要替代单个表的设置，请更改表存储参数。 

## <a name="autovacuum-cost"></a>Autovacuum 成本
以下是运行清扫操作的“成本”：

- 锁定运行清扫作业的数据页面。
- 运行清扫作业时使用计算和内存。

因此，清扫作业的运行频率不宜过高或过低。 清扫作业需要适应工作负荷。 由于不同的 autovacuum 参数存在利弊，请对所有参数更改进行测试。

## <a name="autovacuum-start-trigger"></a>Autovacuum 启动触发器
当死元组的数量超过 autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * reltuples 时，会触发 autovacuum。 此处的 reltuples 为一个常数。

通过 autovacuum 进行的清理工作必须与数据库负载保持一致。 否则可能会耗尽存储并导致查询速度普遍下降。 在随时间分摊后，清扫操作清理死元组的速率应该等于创建死元组的速率。

带有很多更新和删除内容的数据库具有更多的死元组，且需要更多的空间。 通常情况下，对于带有很多更新和删除内容的数据库，将 autovacuum_vacuum_scale_factor 和 autovacuum_vacuum_threshold 设为较低的值比较有利。 较低的值可以防止死元组长时间累积。 而较小的数据库可将这两个参数设为较高的值，因为它们的清扫需求没有那么迫切。 频繁的清扫作业会带来计算和内存成本。

默认比例因子 20% 适用于死元组占比较低的表。 它不适用于死元组占比较高的表。 例如，对于 20 GB 的表，此比例因子相当于存在 4 GB 的死元组。 对于 1 TB 的表，相当于存在 200 GB 的死元组。

使用 PostgreSQL，可以在表级别或实例级别设置这些参数。 目前，只能在 Azure Database for PostgreSQL 中的表级别设置这些参数。

## <a name="estimate-the-cost-of-autovacuum"></a>预估 autovacuum 的成本
运行 autovacuum 的成本比较高昂，有一些参数可以控制清扫操作的运行时。 以下参数有助于预估运行清扫作业的成本：
- vacuum_cost_page_hit = 1
- vacuum_cost_page_miss = 10
- vacuum_cost_page_dirty = 20

清扫进程读取物理页面并检查死元组。 shared_buffers 中的每个页面产生的成本为 1 (vacuum_cost_page_hit)。 至于所有其他页面，如果存在死元组，则产生的成本为 20 (vacuum_cost_page_dirty)，如果不存在死元组，则成本为 10 (vacuum_cost_page_miss)。 在进程超过 autovacuum_vacuum_cost_limit 时会停止清扫操作。 

达到此限制后，进程会进入睡眠状态，在经过 autovacuum_vacuum_cost_delay 参数所指定的持续时间后，会再次启动。 如果未达到该限制，会在经过 autovacuum_nap_time 所指定的时间后启动 autovacuum。

总之，autovacuum_vacuum_cost_delay 和 autovacuum_vacuum_cost_limit 参数用于控制每个时间单位允许清除多少数据。 请注意，对于大多数定价层而言，默认值都过低。 这些参数的最佳值取决于不同的定价层，应根据定价层进行相应的配置。

autovacuum_max_workers 参数确定能同时运行的最大 autovacuum 进程数。

使用 PostgreSQL，可以在表级别或实例级别设置这些参数。 目前，只能在 Azure Database for PostgreSQL 中的表级别设置这些参数。

## <a name="optimize-autovacuum-per-table"></a>优化每个表的 autovacuum
可以针对每个表配置上述所有配置参数。 下面是一个示例：
```sql
ALTER TABLE t SET (autovacuum_vacuum_threshold = 1000);
ALTER TABLE t SET (autovacuum_vacuum_scale_factor = 0.1);
ALTER TABLE t SET (autovacuum_vacuum_cost_limit = 1000);
ALTER TABLE t SET (autovacuum_vacuum_cost_delay = 10);
```

Autovacuum 一个针对每个表的同步进程。 死元组在一个表中的占比越高，执行 autovacuum 的“成本”就越高。 可将更新和删除内容占比较高的表拆分为多个表。 拆分表有助于并行化 autovacuum，并降低在一个表上完成 autovacuum 的“成本”。 还可以增加并行 autovacuum 辅助角色，从而确保安排了充足的辅助角色。

## <a name="next-steps"></a>后续步骤
若要详细了解如何使用和优化 autovacuum，请参阅以下 PostgreSQL 文档：

 - [第 18 章：服务器配置](https://www.postgresql.org/docs/9.5/static/runtime-config-autovacuum.html)
 - [第 24 章：日常数据库维护任务](https://www.postgresql.org/docs/9.6/static/routine-vacuuming.html)
