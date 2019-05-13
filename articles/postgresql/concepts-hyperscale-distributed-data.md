---
title: Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 中的分布式的数据
description: 表和分片分发服务器组中。
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 9020ee690d93a1b477471fac4a482a909fca5935
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077330"
---
# <a name="distributed-data-in-azure-database-for-postgresql--hyperscale-citus-preview"></a>Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 中的分布式的数据

本文概述了超大规模 (Citus) 中的三个表类型。
它显示了如何分布式的表存储为分片和分片放置在节点的方法。

## <a name="table-types"></a>表类型

有三种类型的超大规模的服务器组中的表，每个用于不同目的。

### <a name="type-1-distributed-tables"></a>类型 1： 分布式的表

第一种类型，和大多数常见*分布式*表。 它们显示为普通表的 SQL 语句，但水平*分区*在辅助节点之间。 这意味着存储中名为片段表的不同节点上存储的表的行*分片*。

超大规模运行不仅 SQL，但在整个群集，因此更改分布式表的架构的 DDL 语句级层叠了更新工作线程之间的所有表的分片。

#### <a name="distribution-column"></a>分布列

超大规模使用算法分片的行分配到分片。 分配明确地根据名为的表的列的值进行*分布列。* 分布表时，群集管理员必须指定此列。
做出正确的选择很重要的性能和功能。

### <a name="type-2-reference-tables"></a>类型为 2： 引用表

引用表是一种其整个内容都集中到单个分片的分布式表。 分片被复制每个辅助角色上，因此任何辅助角色上的查询可以访问的参考信息本地，而从另一个节点请求行的网络开销。 引用表具有没有分布列，因为无需将独立分片中的每行区分开来。

引用表是通常很小，并且用于存储与任何辅助角色节点上运行的查询相关的数据。 例如，枚举值，如订单状态或产品类别。

### <a name="type-3-local-tables"></a>类型 3： 本地表

当使用超大规模时，连接到的协调器节点是一个常规的 PostgreSQL 数据库。 可以使用协调器上创建普通表并选择不到分片它们。

本地表的良好候选项将不参与联接的查询的小型管理表。 例如，应用程序登录名和身份验证的用户表。

## <a name="shards"></a>分片

上一节所述的分布式的表存储用作分片的辅助角色节点。 本部分中获取更多技术详细信息。

`pg_dist_shard`元数据表上协调器系统中包含每个分片的每个分布式表的行。 该行与具有一系列整数哈希空间 （shardminvalue，shardmaxvalue） 中的分片 ID 的匹配：

```sql
SELECT * from pg_dist_shard;
 logicalrelid  | shardid | shardstorage | shardminvalue | shardmaxvalue 
---------------+---------+--------------+---------------+---------------
 github_events |  102026 | t            | 268435456     | 402653183
 github_events |  102027 | t            | 402653184     | 536870911
 github_events |  102028 | t            | 536870912     | 671088639
 github_events |  102029 | t            | 671088640     | 805306367
 (4 rows)
```

协调器节点想要确定哪个分片来保存的某一行`github_events`，它在行中，分布列的值进行哈希处理并检查哪个分片\'的范围包含哈希的值。 （范围定义以便其不连续的并集的图像的哈希函数。）

### <a name="shard-placements"></a>分片放置

假设该分片 102027 是与有问题的行相关联。 将读取或写入名为的表行`github_events_102027`中的辅助角色。 哪个工作线程？ 确定完全由元数据表和辅助角色到分片的映射称为分片*放置*。

协调器节点的对等的特定表，请参阅片段到重写查询`github_events_102027`，并相应辅助进程上运行这些片段。 下面是查询的查找包含分片 ID 102027 的节点在后台运行的示例。

```sql
SELECT
    shardid,
    node.nodename,
    node.nodeport
FROM pg_dist_placement placement
JOIN pg_dist_node node
  ON placement.groupid = node.groupid
 AND node.noderole = 'primary'::noderole
WHERE shardid = 102027;
```

    ┌─────────┬───────────┬──────────┐
    │ shardid │ nodename  │ nodeport │
    ├─────────┼───────────┼──────────┤
    │  102027 │ localhost │     5433 │
    └─────────┴───────────┴──────────┘

## <a name="next-steps"></a>后续步骤
- 了解如何[选择分布列](concepts-hyperscale-choose-distribution-column.md)对于分布式表
