---
title: Azure Database for PostgreSQL-单个服务器中的查询存储
description: 本文介绍用于 PostgreSQL 的单个服务器的 Azure 数据库中的查询存储功能。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: b622de3e21d26676bb11d81a6facf8fea18cabc1
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067188"
---
# <a name="monitor-performance-with-the-query-store"></a>使用查询存储监视性能

**适用于：** Azure Database for PostgreSQL-9.6 和 10 台服务器

Azure Database for PostgreSQL 中的查询存储功能提供了一种一段时间内跟踪查询性能的方法。 通过帮助快速查找运行时间最长且资源最密集的查询，查询存储可简化性能故障排除。 查询存储自动捕获查询和运行时统计信息的历史记录，并保留它们以供查看。 它按时间范围分隔数据，以便可以查看数据库使用模式。 所有用户、数据库和查询的数据都存储在 Azure Database for PostgreSQL 实例中的名为 azure_sys 的数据库中。

> [!IMPORTANT]
> 请勿修改 azure_sys 数据库或其架构。 执行此操作将阻止查询存储和相关的性能功能正常运行。

## <a name="enabling-query-store"></a>启用查询存储
查询存储是一项选择加入功能，因此默认情况下它在服务器上未处于活动状态。 对于给定服务器上的所有数据库，该存储处于全局启用或禁用状态，且无法为每个数据库打开或关闭存储。

### <a name="enable-query-store-using-the-azure-portal"></a>使用 Azure 门户启用查询存储
1. 登录到 Azure 门户，选择 Azure Database for PostgreSQL 服务器。
2. 在菜单的“设置”部分中选择“服务器参数”。
3. 搜索 `pg_qs.query_capture_mode` 参数。
4. 将值设置为 `TOP` 并**保存**。

若要在查询存储中启用等待统计信息，请执行以下操作： 
1. 搜索 `pgms_wait_sampling.query_capture_mode` 参数。
1. 将值设置为 `ALL` 并**保存**。


或者，可使用 Azure CLI 设置这些参数。
```azurecli-interactive
az postgres server configuration set --name pg_qs.query_capture_mode --resource-group myresourcegroup --server mydemoserver --value TOP
az postgres server configuration set --name pgms_wait_sampling.query_capture_mode --resource-group myresourcegroup --server mydemoserver --value ALL
```

允许第一批数据在 azure_sys 数据库中最多保留 20 分钟。

## <a name="information-in-query-store"></a>查询存储中的信息
查询存储有两个存储：
- 运行时统计信息存储，用于保存查询执行统计信息。
- 等待统计信息存储，用于保存等待统计信息。

使用查询存储的常见方案包括：
- 确定在给定时间范围内执行查询的次数
- 比较跨时间范围查询的平均执行时间，以查看较大的增量
- 标识过去 X 小时内运行时间最长的查询
- 标识正在等待资源的前 N 个查询
- 了解特定查询的等待性质

为尽量减少空间使用量，运行时统计信息存储中的运行时执行统计信息在一个固定的、可配置的时间范围内聚合。 通过查询这些查询存储视图，这些存储中的信息是可见的。

以下查询返回有关查询存储中的查询的信息：
```sql
SELECT * FROM query_store.qs_view; 
``` 

或此等待统计信息的查询：
```sql
SELECT * FROM query_store.pgms_wait_sampling_view;
```

## <a name="finding-wait-queries"></a>查找等待查询
等待事件类型按相似性将不同的等待事件组合到存储桶中。 查询存储提供等待事件类型、特定等待事件名称和有争议的查询。 能将此等待信息与查询运行时统计信息相关联，意味着可更深入地了解有助于查询性能特征的因素。

以下是一些示例，说明如何使用查询存储中的等待统计信息获得有关工作负载的更多见解：

| **观测** | **Action** |
|---|---|
|高锁定等待 | 检查受影响查询的查询文本，并确定目标实体。 在查询存储中查找修改同一实体的其他查询，这些查询经常执行和/或持续很长时间。 确定这些查询后，请考虑更改应用程序逻辑以提高并发性，或使用限制较少的隔离级别。|
| 高缓冲 IO 等待 | 在查询存储中查找具有大量物理读取的查询。 如果它们匹配具有高 IO 等待的查询，考虑在基础实体上引入索引，以便进行搜索而不是扫描。 这将最小化查询的 IO 开销。 检查门户中服务器的“性能建议”，以查看是否存在可优化查询的此服务器的索引建议。|
| 高内存等待 | 在查询存储中查找消耗内存最多的查询。 这些查询可能会延迟受影响查询的进度。 检查门户中服务器的“性能建议”，以查看是否存在可优化这些查询的索引建议。|

## <a name="configuration-options"></a>配置选项
启用查询存储时，它会在 15 分钟的聚合时段内保存数据，每个时段最多可存储 500 个不同查询。 

以下选项可用于配置查询存储参数。

| **Parameter** | **说明** | **默认** | **范围**|
|---|---|---|---|
| pg_qs.query_capture_mode | 设置跟踪哪些语句。 | 无 | none, top, all |
| pg_qs.max_query_text_length | 设置可保存的最大查询长度。 将截断较长的查询。 | 6000 | 100 - 10K |
| pg_qs.retention_period_in_days | 设置保持期。 | 7 | 1 - 30 |
| pg_qs.track_utility | 设置是否跟踪实用程序命令 | on | on, off |

以下选项专用于等待统计信息。

| **Parameter** | **说明** | **默认** | **范围**|
|---|---|---|---|
| pgms_wait_sampling.query_capture_mode | 设置跟踪哪些语句以获取等待统计信息。 | 无 | none, all|
| Pgms_wait_sampling.history_period | 设置等待事件采样的频率（以毫秒为单位）。 | 100 | 1-600000 |

> [!NOTE] 
> 将 pgms_wait_sampling.query_capture_mode 替代为 pg_qs.query_capture_mode。 如果 pg_qs.query_capture_mode 为 NONE，则 pgms_wait_sampling.query_capture_mode 设置无效。


使用 [Azure 门户](howto-configure-server-parameters-using-portal.md)或 [Azure CLI](howto-configure-server-parameters-using-cli.md) 获取或设置参数的不同值。

## <a name="views-and-functions"></a>视图和函数
使用以下视图和函数查看并管理查询存储。 PostgreSQL 公共角色中的任何人都可使用这些视图来查看查询存储中的数据。 这些视图仅在 azure_sys 数据库中可用。

删除文本和常数后，通过查看查询的结构来规范化查询。 如果除文本值之外两个查询相同，则它们将具有相同的哈希值。

### <a name="querystoreqsview"></a>query_store.qs_view
此视图返回查询存储中的所有数据。 每个不同的数据库 ID、用户 ID 和查询 ID 都有一行。 

|**名称**   |类型 | **参考**  | **说明**|
|---|---|---|---|
|runtime_stats_entry_id |bigint | | runtime_stats_entries 表的 ID|
|user_id    |oid    |pg_authid.oid  |执行此语句的用户的 OID|
|db_id  |oid    |pg_database.oid    |在其中执行语句的数据库的 OID|
|query_id   |bigint  || 根据语句的分析树计算的内部哈希代码|
|query_sql_text |Varchar(10000)  || 代表语句的文本。 具有相同结构的不同查询聚集在一起；此文本是群集中第一个查询的文本。|
|plan_id    |bigint |   |与此查询对应的计划 ID，尚不可用|
|start_time |timestamp  ||  查询按时段聚合 - 默认情况下，存储桶的时间跨度为 15 分钟。 这是与此条目的时段相对应的开始时间。|
|end_time   |timestamp  ||  对应于此条目的时段的结束时间。|
|calls  |bigint  || 执行查询的次数|
|total_time |双精度   ||  总查询执行时间（毫秒）|
|min_time   |双精度   ||  最小查询执行时间（毫秒）|
|max_time   |双精度   ||  最大查询执行时间（毫秒）|
|mean_time  |双精度   ||  平均查询执行时间（毫秒）|
|stddev_time|   双精度    ||  查询执行时间的标准偏差（毫秒） |
|行   |bigint ||  由语句检索或影响的总行数|
|shared_blks_hit|   bigint  ||  语句的共享块缓存命中总数|
|shared_blks_read|  bigint  ||  由语句读取的共享块总数|
|shared_blks_dirtied|   bigint   || 由语句更新的共享块总数 |
|shared_blks_written|   bigint  ||  由语句编写的共享块总数|
|local_blks_hit|    bigint ||   语句的本地块缓存命中总数|
|local_blks_read|   bigint   || 由语句读取的本地块总数|
|local_blks_dirtied|    bigint  ||  由语句更新的本地块总数|
|local_blks_written|    bigint  ||  由语句编写的本地块总数|
|temp_blks_read |bigint  || 由语句读取的临时块总数|
|temp_blks_written| bigint   || 由语句编写的临时块总数|
|blk_read_time  |双精度    || 语句读取块所花费的总时间（以毫秒为单位）（如果启用了 track_io_timing，否则为零）|
|blk_write_time |双精度    || 语句写入块所花费的总时间（以毫秒为单位）（如果启用了 track_io_timing，否则为零）|
    
### <a name="querystorequerytextsview"></a>query_store.query_texts_view
此视图返回查询存储中的查询文本数据。 每个不同的 query_text 都有一行。

|**名称**|  类型|   **说明**|
|---|---|---|
|query_text_id  |bigint     |query_texts 表的 ID|
|query_sql_text |Varchar(10000)     |代表语句的文本。 具有相同结构的不同查询聚集在一起；此文本是群集中第一个查询的文本。|

### <a name="querystorepgmswaitsamplingview"></a>query_store.pgms_wait_sampling_view
此视图返回查询存储中的等待事件数据。 每个不同的数据库 ID、用户 ID、查询 ID 和事件都有一行。

|**名称**|  类型|   **参考**| **说明**|
|---|---|---|---|
|user_id    |oid    |pg_authid.oid  |执行此语句的用户的 OID|
|db_id  |oid    |pg_database.oid    |在其中执行语句的数据库的 OID|
|query_id   |bigint     ||根据语句的分析树计算的内部哈希代码|
|event_type |Text       ||后端正在等待的事件类型|
|event  |Text       ||后端当前正在等待的等待事件名称|
|calls  |Integer        ||捕获的相同事件的数量|


### <a name="functions"></a>函数
Query_store.qs_reset() 返回无效值

`qs_reset` 丢弃查询存储到目前为止收集的所有统计信息。 只能由服务器管理员角色执行此函数。

Query_store.staging_data_reset() 返回无效值

`staging_data_reset` 丢弃查询存储在内存中收集的所有统计信息（即内存中尚未刷新到数据库的数据）。 只能由服务器管理员角色执行此函数。

## <a name="limitations-and-known-issues"></a>限制和已知问题
- 如果 PostgreSQL 服务器具有参数 default_transaction_read_only，则查询存储无法捕获数据。
- 如果遇到较长的 Unicode 查询（> = 6000 个字节），查询存储功能可能会中断。


## <a name="next-steps"></a>后续步骤
- 了解有关[查询存储可能特别有用的方案](concepts-query-store-scenarios.md)的详细信息。
- 了解有关[使用查询存储的最佳做法](concepts-query-store-best-practices.md)的详细信息。
