---
title: Azure Database for PostgreSQL-单个服务器中的使用 PostgreSQL 扩展
description: 描述扩展的数据库使用 Azure database for PostgreSQL-单台服务器扩展功能的能力。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 962e2b10136cf1cbab7cc5d3d06059922c363b15
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410273"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---single-server"></a>Azure Database for PostgreSQL-单个服务器中的 PostgreSQL 扩展
PostgreSQL 支持使用扩展来扩展数据的功能。 扩展允许在单个包中将多个相关 SQL 对象捆绑在一起，可以使用单个命令在数据库中加载或删除该包。 在数据库中加载之后，扩展可以如同内置功能一样运行。 有关 PostgreSQL 扩展的详细信息，请参阅  [Packaging Related Objects into an Extension](https://www.postgresql.org/docs/9.6/static/extend-extensions.html)（将相关对象打包到扩展中）。

## <a name="how-to-use-postgresql-extensions"></a>如何使用 PostgreSQL 扩展
必须先在数据库中安装 PostgreSQL 扩展，然后才能使用它们。 若要安装特定扩展，请通过 psql 工具运行  [CREATE EXTENSION](https://www.postgresql.org/docs/9.6/static/sql-createextension.html)  命令，将打包的对象加载到数据库中。

Azure Database for PostgreSQL 目前支持部分关键扩展（已在下面列出）。 未列出的扩展不受支持；无法使用 Azure Database for PostgreSQL 服务创建自己的扩展。

## <a name="extensions-supported-by-azure-database-for-postgresql"></a>Azure Database for PostgreSQL 支持的扩展
下表列出了用于 PostgreSQL 的 Azure 数据库目前支持的标准 PostgreSQL 扩展。 还可以通过运行 `SELECT * FROM pg_available_extensions;` 获取此信息。

### <a name="data-types-extensions"></a>数据类型扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [chkpass](https://www.postgresql.org/docs/9.6/static/chkpass.html) | 提供用于自动加密密码的数据类型。 |
> | [citext](https://www.postgresql.org/docs/9.6/static/citext.html) | 提供不区分大小写的字符串类型。 |
> | [cube](https://www.postgresql.org/docs/9.6/static/cube.html) | 提供用于多维数据集的数据类型。 |
> | [hstore](https://www.postgresql.org/docs/9.6/static/hstore.html) | 提供用于存储键/值对集的数据类型。 |
> | [isn](https://www.postgresql.org/docs/9.6/static/isn.html) | 提供用于国际产品编号标准的数据类型。 |
> | [ltree](https://www.postgresql.org/docs/9.6/static/ltree.html) | 提供用于分层树形结构的数据类型。 |

### <a name="functions-extensions"></a>函数扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [earthdistance](https://www.postgresql.org/docs/9.6/static/earthdistance.html) | 提供一种计算地球表面上的大圆距离的方法。 |
> | [fuzzystrmatch](https://www.postgresql.org/docs/9.6/static/fuzzystrmatch.html) | 提供多个函数，用于确定字符串间的相似性和差异。 |
> | [intarray](https://www.postgresql.org/docs/9.6/static/intarray.html) | 提供用于操作无 null 整数数组的函数和运算符。 |
> | [pgcrypto](https://www.postgresql.org/docs/9.6/static/pgcrypto.html) | 提供加密函数。 |
> | [pg\_partman](https://pgxn.org/dist/pg_partman/doc/pg_partman.html) | 按时间或 ID 管理已分区表。 |
> | [pg\_trgm](https://www.postgresql.org/docs/9.6/static/pgtrgm.html) | 提供函数和运算符，用于基于三元匹配确定字母数字文本的相似性。 |
> | [tablefunc](https://www.postgresql.org/docs/9.6/static/tablefunc.html) | 提供可操作整个表（包括交叉表）的函数。 |
> | [uuid ossp](https://www.postgresql.org/docs/9.6/static/uuid-ossp.html) | 生成全局唯一标识符 (UUID)。 |
> | [orafce](https://github.com/orafce/orafce) | 提供了功能和包从商业数据库模拟的子集。 |

### <a name="full-text-search-extensions"></a>全文搜索扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [dict\_int](https://www.postgresql.org/docs/9.6/static/dict-int.html) | 提供用于整数的文本搜索字典模板。 |
> | [unaccent](https://www.postgresql.org/docs/9.6/static/unaccent.html) | 删除了词素中重音（附加符号）的文本搜索字典。 |

### <a name="index-types-extensions"></a>索引类型扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [btree\_gin](https://www.postgresql.org/docs/9.6/static/btree-gin.html) | 提供示例 GIN 运算符类，该类对特定数据类型实现类似 B-tree 的行为。 |
> | [btree\_gist](https://www.postgresql.org/docs/9.6/static/btree-gist.html) | 提供实施 B-tree 的 GiST 索引运算符类。 |

### <a name="language-extensions"></a>语言扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [plpgsql](https://www.postgresql.org/docs/9.6/static/plpgsql.html) | PL/pgSQL 可加载过程语言。 |

### <a name="miscellaneous-extensions"></a>其他扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [pg\_buffercache](https://www.postgresql.org/docs/9.6/static/pgbuffercache.html) | 提供一种方法用于实时检查共享缓冲区缓存的当前状况。 |
> | [pg\_prewarm](https://www.postgresql.org/docs/9.6/static/pgprewarm.html) | 提供一种方法用于将相关数据加载到缓冲区缓存中。 |
> | [pg\_stat\_statements](https://www.postgresql.org/docs/9.6/static/pgstatstatements.html) | 提供一种方法用于跟踪服务器执行的所有 SQL 语句的执行统计信息。 （请参阅下文了解此扩展的说明）。 |
> | [pgrowlocks](https://www.postgresql.org/docs/9.6/static/pgrowlocks.html) | 提供一种显示行级锁定信息的方法。 |
> | [pgstattuple](https://www.postgresql.org/docs/9.6/static/pgstattuple.html) | 提供一种显示元组级别统计信息的方法。 |
> | [postgres\_fdw](https://www.postgresql.org/docs/9.6/static/postgres-fdw.html) | 外部数据包装器，用于访问外部 PostgreSQL 服务器中存储的数据。 （请参阅下文了解此扩展的说明）。|
> | [hypopg](https://hypopg.readthedocs.io/en/latest/) | 提供了一种创建不耗费 CPU 或磁盘的假设索引的方法。 |
> | [dblink](https://www.postgresql.org/docs/current/dblink.html) | 一个支持从数据库会话中连接到其他 PostgreSQL 数据库的模块。 （请参阅下文了解此扩展的说明）。 |


### <a name="postgis-extensions"></a>PostGIS 扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [PostGIS](https://www.postgis.net/), postgis\_topology, postgis\_tiger\_geocoder, postgis\_sfcgal | PostgreSQL 的空间和地理对象。 |
> | address\_standardizer, address\_standardizer\_data\_us | 用于将地址分析成构成元素。 用于支持地理编码地址规范化步骤。 |
> | [pgrouting](https://pgrouting.org/) | 扩展 PostGIS / PostgreSQL 地理空间数据库，以提供地理空间路由功能。 |


### <a name="time-series-extensions"></a>时序扩展

> [!div class="mx-tableFixed"]
> | **扩展** | **说明** |
> |---|---|
> | [TimescaleDB](https://docs.timescale.com/latest) | 一个时序 SQL 数据库，支持通过自动分区来加快引入和查询速度。 提供以时间为导向的分析功能、优化，并根据时序工作负荷来缩放 PostgreSQL。 TimescaleDB 是由 [Timescale, Inc.](https://www.timescale.com/) 开发的的，是其注册商标。（请参阅下文了解此扩展的说明）。 |


## <a name="pgstatstatements"></a>pg_stat_statements
[pg\_stat\_ 扩展](https://www.postgresql.org/docs/9.6/static/pgstatstatements.html)预加载在每个 Azure Database for PostgreSQL 服务器上，以便跟踪 SQL 语句的执行统计信息。
设置 `pg_stat_statements.track`，它可以控制哪些语句由扩展计数，默认为 `top`，这意味着跟踪所有由客户端直接发布的语句。 另外两个跟踪级别为 `none` 和 `all`。 此设置可通过 [Azure 门户](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-portal)或 [Azure CLI](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-cli) 作为服务器参数进行配置。

查询执行信息 pg_stat_statements 提供的权限与记录每个 SQL 语句时对服务器性能的影响之间存在权衡。 如果不经常使用 pg_stat_statements 扩展，则建议将 `pg_stat_statements.track` 设置为 `none`。 请注意，某些第三方监视服务可能依赖 pg_stat_statements 来提供查询性能见解，因此，请确认这是否适合你。

## <a name="dblink-and-postgresfdw"></a>dblink 和 postgres_fdw
dblink 和 postgres_fdw 允许从一台 PostgreSQL 服务器连接到另一台服务器，或者连接到同一服务器中的另一个数据库。 接收服务器需要允许来自发送服务器的连接通过其防火墙。 当使用这些扩展在 Azure Database for PostgreSQL 服务器之间进行连接时，可以通过将“允许访问 Azure 服务”设置为“开启”来实现此目的。 如果希望使用扩展来环回到同一服务器，也需要进行此设置。 可以在 Postgres 服务器的 Azure 门户页面中的“连接安全性”下找到“允许访问 Azure 服务”设置。 开启“允许访问 Azure 服务”会将所有 Azure IP 加入允许列表。

目前不支持从 Azure Database for PostgreSQL 进行出站连接，连接到其他 Azure Database for PostgreSQL 服务器的情况除外。

## <a name="timescaledb"></a>TimescaleDB
TimescaleDB 是一个时序数据库，已作为 PostgreSQL 的扩展打包。 TimescaleDB 提供以时间为导向的分析功能、优化，并根据时序工作负荷来缩放 Postgres。

[详细了解 TimescaleDB](https://docs.timescale.com/latest)，[Timescale, Inc.](https://www.timescale.com/) 的注册商标。

### <a name="installing-timescaledb"></a>安装 TimescaleDB
若要安装 TimescaleDB，需将其包括在服务器的共享预加载库中。 对 Postgres 的共享预加载库进行的更改需**重启服务器**才能生效。

> [!NOTE]
> 可以在 Azure Database for PostgreSQL 版本 9.6 和 10 上启用 TimescaleDB

使用 [Azure 门户](https://portal.azure.com/)：

1. 选择你的 Azure Database for PostgreSQL 服务器。

2. 在侧栏中选择“服务器参数”。

3. 搜索 `shared_preload_libraries` 参数。

4. 复制并粘贴以下内容作为 `shared_preload_libraries` 的值
   ```
   timescaledb
   ```

5. 选择“保存”，保留所做的更改。 保存更改后会获得通知。 

6. 获得通知后，请**重启**服务器以应用这些更改。 若要了解如何重启服务器，请参阅[重启 Azure Database for PostgreSQL 服务器](howto-restart-server-portal.md)。


现在可以在 Postgres 数据库中启用 TimescaleDB。 连接到数据库并发出以下命令：
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```
> [!TIP]
> 如果看到错误，请确认是否已在保存 shared_preload_libraries 后[重启服务器](howto-restart-server-portal.md)。 

现在可以[从头开始](https://docs.timescale.com/getting-started/creating-hypertables)创建 TimescaleDB hypertable，也可以迁移 [PostgreSQL 中的现有时序数据](https://docs.timescale.com/getting-started/migrating-data)。


## <a name="next-steps"></a>后续步骤
如果未看到要使用的扩展，请告诉我们。 若要支持现有请求或提出新反馈和请求，请访问我们的[客户反馈论坛](https://feedback.azure.com/forums/597976-azure-database-for-postgresql)。
