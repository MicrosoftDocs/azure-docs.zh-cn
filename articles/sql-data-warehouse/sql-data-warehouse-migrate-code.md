---
title: 将 SQL 代码迁移到 SQL 数据仓库 | Microsoft 文档
description: 有关在开发解决方案时将 SQL 代码迁移到 Azure SQL 数据仓库的技巧。
services: sql-data-warehouse
author: WenJason
manager: digimobile
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: implement
origin.date: 04/17/2018
ms.date: 04/01/2019
ms.author: v-jay
ms.reviewer: igorstan
ms.openlocfilehash: fae3ae16ee0100ad446c0b6c7851553a3376bb4f
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61126781"
---
# <a name="migrate-your-sql-code-to-sql-data-warehouse"></a>将 SQL 代码迁移到 SQL 数据仓库

本文介绍了将代码从其他数据库迁移到 SQL 数据仓库时可能需要做出的代码更改。 某些 SQL 数据仓库功能设计为以分布方式运行，因此可以大幅改善性能。 但是，为了保持性能和缩放性，某些功能还无法使用。

## <a name="common-t-sql-limitations"></a>常见的 T-SQL 限制

下面的列表汇总了 SQL 数据仓库不支持的最常用功能。 单击相应链接可以查看不支持的功能对应的解决方法：

* [Update 中的 ANSI Join][ANSI joins on updates]
* [对删除操作使用 ANSI 联接][ANSI joins on deletes]
* [合并语句][merge statement]
* 跨数据库联接
* [游标][cursors]
* [INSERT..EXEC][INSERT..EXEC]
* output 子句
* 内联用户定义的函数
* 多语句函数
* 通用表表达式
* [递归通用表表达式 (CTE)](#Recursive-common-table-expressions-(CTE)
* CLR 函数和过程
* $partition 函数
* 表变量
* 表值参数
* 分布式事务
* 提交/回滚工作
* 保存事务
* 执行上下文 (EXECUTE AS)
* [结合 rollup / cube / grouping sets 选项的 Group By 子句][group by clause with rollup / cube / grouping sets options]
* [嵌套级别超过 8][nesting levels beyond 8]
* [通过视图更新][updating through views]
* [动态 SQL 字符串没有 MAX 数据类型][no MAX data type for dynamic SQL strings]

幸好可以解决其中的大多数限制。 上面提到的相关开发文章已提供了说明。

## <a name="supported-cte-features"></a>支持的 CTE 功能

SQL 数据仓库支持部分通用表表达式 (CTE)。  目前支持以下 CTE 功能：

* 可以在 SELECT 语句中指定 CTE。
* 可以在 CREATE VIEW 语句中指定 CTE。
* 可以在 CREATE TABLE AS SELECT (CTAS) 语句中指定 CTE。
* 可以在 CREATE REMOTE TABLE AS SELECT (CRTAS) 语句中指定 CTE。
* 可以在 CREATE EXTERNAL TABLE AS SELECT (CETAS) 语句中指定 CTE。
* 可以从 CTE 引用远程表。
* 可以从 CTE 引用外部表。
* 可以在 CTE 中定义多个 CTE 查询定义。

## <a name="cte-limitations"></a>CTE 限制

在 SQL 数据仓库中，通用表表达式存在一些限制，其中包括：

* CTE 必须后接单个 SELECT 语句。 不支持 INSERT、UPDATE、DELETE 和 MERGE 语句。
* 包含自身引用的通用表表达式（即递归通用表表达式）不受支持（参见以下部分）。
* 不允许在 CTE 中指定多个 WITH 子句。 例如，如果 CTE_query_definition 包含子查询，则该子查询不能包含用于定义其他 CTE 的嵌套式 WITH 子句。
* ORDER BY 子句不能用于 CTE_query_definition，除非指定了 TOP 子句。
* 将 CTE 用在属于批处理一部分的语句中时，该 CTE 之前的语句必须以分号结尾。
* 用在通过 sp_prepare 进行准备的语句中时，CTE 的表现方式与 PDW 中的其他 SELECT 语句相同。 但是，如果 CTE 用作 sp_prepare 所准备的 CETAS 的一部分，则因为针对 sp_prepare 而实现绑定的方式不同，CTE 的行为将与 SQL Server 和其他 PDW 语句不同。 如果引用 CTE 的 SELECT 使用了 CTE 中不存在的错误列，sp_prepare 将会通过而不检测错误，但在 sp_execute 期间将引发错误。

## <a name="recursive-ctes"></a>递归 CTE

SQL 数据仓库不支持递归 CTE。  递归 CTE 的迁移过程可能有点复杂，最佳做法是将此过程拆分为多步过程。 通常可以使用循环，并在循环访问递归的临时查询时填充临时表。 填充临时表之后，可以使用单个结果集返回数据。 类似的方法已用于解决[结合 rollup/cube/grouping sets 选项的 Group By 子句][group by clause with rollup / cube / grouping sets options]一文中所述的 `GROUP BY WITH CUBE`。

## <a name="unsupported-system-functions"></a>不支持的系统函数

还有一些不支持的系统函数。 在数据仓库中，可能会经常发现使用了下面这些主要函数：

* NEWSEQUENTIALID()
* @@NESTLEVEL()
* @@IDENTITY()
* @@ROWCOUNT()
* ROWCOUNT_BIG
* ERROR_LINE()

其中的许多问题都可以得到解决。

## <a name="rowcount-workaround"></a>@@ROWCOUNT 解决方法

若要解决缺少对 @@ROWCOUNT 的支持的问题，请创建一个将从 sys.dm_pdw_request_steps 中检索最后一个行计数的存储过程，然后在 DML 语句后执行 `EXEC LastRowCount`。

```sql
CREATE PROCEDURE LastRowCount AS
WITH LastRequest as
(   SELECT TOP 1    request_id
    FROM            sys.dm_pdw_exec_requests
    WHERE           session_id = SESSION_ID()
    AND             resource_class IS NOT NULL
    ORDER BY end_time DESC
),
LastRequestRowCounts as
(
    SELECT  step_index, row_count
    FROM    sys.dm_pdw_request_steps
    WHERE   row_count >= 0
    AND     request_id IN (SELECT request_id from LastRequest)
)
SELECT TOP 1 row_count FROM LastRequestRowCounts ORDER BY step_index DESC
;
```

## <a name="next-steps"></a>后续步骤

有关所有支持的 T-SQL 语句的完整列表，请参阅 [Transact-SQL 主题][Transact-SQL topics]。

<!--Image references-->

<!--Article references-->
[ANSI joins on updates]: ./sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-update-statements
[ANSI joins on deletes]: ./sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-delete-statements
[merge statement]: ./sql-data-warehouse-develop-ctas.md#replace-merge-statements
[INSERT..EXEC]: ./sql-data-warehouse-tables-temporary.md#modularizing-code
[Transact-SQL topics]: ./sql-data-warehouse-reference-tsql-statements.md

[cursors]: ./sql-data-warehouse-develop-loops.md
[group by clause with rollup / cube / grouping sets options]: ./sql-data-warehouse-develop-group-by-options.md
[nesting levels beyond 8]: ./sql-data-warehouse-develop-transactions.md
[updating through views]: ./sql-data-warehouse-develop-views.md
[use of select for variable assignment]: ./sql-data-warehouse-develop-variable-assignment.md
[no MAX data type for dynamic SQL strings]: ./sql-data-warehouse-develop-dynamic-sql.md

<!--MSDN references-->

<!--Other Web references-->

<!--Update_Description: update meta properties, wording update-->