---
title: 使用 IDENTITY 创建代理键 - Azure SQL 数据仓库 | Microsoft Docs
description: 关于如何使用 IDENTITY 属性在 Azure SQL 数据仓库中创建基于表的代理键的建议和示例。
services: sql-data-warehouse
author: XiaoyuL-Preview
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: implement
ms.date: 04/30/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: be7d1f7d04b759a5d60ba26d391d19d1bc57f325
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154314"
---
# <a name="using-identity-to-create-surrogate-keys-in-azure-sql-data-warehouse"></a>使用 IDENTITY 在 Azure SQL 数据仓库中创建代理键

关于如何使用 IDENTITY 属性在 Azure SQL 数据仓库中创建基于表的代理键的建议和示例。

## <a name="what-is-a-surrogate-key"></a>什么是代理键

基于表的代理键是一个列，其中包含针对每个行的唯一标识符。 此键不是从表数据生成的。 数据建模者想要在设计数据仓库模型时在其表上创建代理键。 可以使用 IDENTITY 属性轻松高效地实现此目标，而不会影响负载性能。  

## <a name="creating-a-table-with-an-identity-column"></a>创建包含 IDENTITY 列的表

IDENTITY 属性设计为能够在数据仓库的所有分发中扩展，而不会影响负载性能。 因此，IDENTITY 的实现旨在实现这些目标。

在首次使用类似以下语句的语法创建表时，可以将表定义为具有 IDENTITY 属性：

```sql
CREATE TABLE dbo.T1
(   C1 INT IDENTITY(1,1) NOT NULL
,   C2 INT NULL
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;
```

然后，可以使用 `INSERT..SELECT` 来填充表。

本部分的此剩余部分重点介绍实现的细微差别，以帮助用户更全面地了解这些实现。  

### <a name="allocation-of-values"></a>值的分配

IDENTITY 属性不保证分配代理值的顺序，这反映了 SQL Server 和 Azure SQL 数据库的行为。 但是，在 Azure SQL 数据仓库中，保证的缺乏更为明显。

下面的示例进行了说明：

```sql
CREATE TABLE dbo.T1
(   C1 INT IDENTITY(1,1)    NOT NULL
,   C2 VARCHAR(30)              NULL
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;

INSERT INTO dbo.T1
VALUES (NULL);

INSERT INTO dbo.T1
VALUES (NULL);

SELECT *
FROM dbo.T1;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

在前面的示例中，两行位于分布 1 中。 第一行在列 `C1` 中包含代理值 1，且第二行包含代理值 61。 这两个值均由 IDENTITY 属性生成。 但是，值的分配是不连续的。 此行为是设计使然。

### <a name="skewed-data"></a>偏斜数据

数据类型的值范围跨分发均匀分配。 如果分布式表受偏斜数据的影响，则可用于数据类型的值范围可能会过早耗尽。 例如，如果所有数据最终都会处于单个分发中，则表实际上只能访问六十分之一的数据类型值。 为此，IDENTITY 属性仅限于 `INT` 和 `BIGINT` 数据类型。

### <a name="selectinto"></a>SELECT..INTO

在将现有的 IDENTITY 列选入新表时，新列将继承该 IDENTITY 属性，除非下列条件之一为 true：

- SELECT 语句包含联接。
- 使用 UNION 联接多个 SELECT 语句。
- IDENTITY 列在 SELECT 列表中多次列出。
- IDENTITY 列是表达式的一部分。

如果其中的任一条件为 true，则创建属性为 NOT NULL 的列，而不继承 IDENTITY 属性。

### <a name="create-table-as-select"></a>CREATE TABLE AS SELECT

CREATE TABLE AS SELECT (CTAS) 遵循 SELECT..INTO 中记录的相同 SQL Server 行为。 但是，不能指定语句的 `CREATE TABLE` 部分的列定义中的 IDENTITY 属性。 同样，也不能在 CTAS 的 `SELECT` 部分中使用 IDENTITY 函数。 若要填充表，需要使用 `CREATE TABLE` 来定义后跟 `INSERT..SELECT` 的表来进行填充。

## <a name="explicitly-inserting-values-into-an-identity-column"></a>将值显式插入到 IDENTITY 列

SQL 数据仓库支持 `SET IDENTITY_INSERT <your table> ON|OFF` 语法。 可以使用此语法将值显式插入 IDENTITY 列。

许多数据建模者喜欢在其维度中为某些行使用预定义的负值。 例如，-1 或“未知成员”行。

下一个脚本演示如何使用 SET IDENTITY_INSERT 显式添加此行：

```sql
SET IDENTITY_INSERT dbo.T1 ON;

INSERT INTO dbo.T1
(   C1
,   C2
)
VALUES (-1,'UNKNOWN')
;

SET IDENTITY_INSERT dbo.T1 OFF;

SELECT  *
FROM    dbo.T1
;
```

## <a name="loading-data"></a>加载数据

IDENTITY 属性的存在对数据加载代码有一定影响。 本节重点介绍使用 IDENTITY 将数据加载到表中的一些基本模式。

若要使用 IDENTITY 将数据加载到表中并生成代理键，请创建表，然后使用 INSERT..SELECT 或 INSERT..VALUES 执行加载。

以下示例重点介绍基本模式：

```sql
--CREATE TABLE with IDENTITY
CREATE TABLE dbo.T1
(   C1 INT IDENTITY(1,1)
,   C2 VARCHAR(30)
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;

--Use INSERT..SELECT to populate the table from an external table
INSERT INTO dbo.T1
(C2)
SELECT  C2
FROM    ext.T1
;

SELECT *
FROM   dbo.T1
;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

> [!NOTE]
> 在将数据加载到包含 IDENTITY 列的表时，当前无法使用 `CREATE TABLE AS SELECT`。
>

有关如何加载数据的详细信息，请参阅[为 Azure SQL 数据仓库设计提取、加载和转换 (ELT)](design-elt-data-loading.md) 和[有关加载的最佳做法](guidance-for-loading-data.md)。

## <a name="system-views"></a>系统视图

可以使用 [sys.identity_columns](/sql/relational-databases/system-catalog-views/sys-identity-columns-transact-sql) 目录视图来标识具有 IDENTITY 属性的列。

为了更好地了解数据库架构，本示例演示如何将 sys.identity_column 与其他系统目录视图集成：

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       CASE WHEN ic.column_id IS NOT NULL
             THEN 1
        ELSE 0
        END AS is_identity
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
LEFT JOIN   sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## <a name="limitations"></a>限制

以下情况不能使用 IDENTITY 属性：

- 当列数据类型不是 INT 或 BIGINT 时
- 当列也同样是分发键时
- 当表是外部表时

SQL 数据仓库中不支持以下相关函数：

- [IDENTITY()](/sql/t-sql/functions/identity-function-transact-sql)
- [@@IDENTITY](/sql/t-sql/functions/identity-transact-sql)
- [SCOPE_IDENTITY](/sql/t-sql/functions/scope-identity-transact-sql)
- [IDENT_CURRENT](/sql/t-sql/functions/ident-current-transact-sql)
- [IDENT_INCR](/sql/t-sql/functions/ident-incr-transact-sql)
- [IDENT_SEED](/sql/t-sql/functions/ident-seed-transact-sql)

## <a name="common-tasks"></a>常见任务

本部分提供在使用 IDENTITY 列时可用于执行常见任务的一些示例代码。

在以下所有任务中，列 C1 是 IDENTITY。

### <a name="find-the-highest-allocated-value-for-a-table"></a>查找表的最大分配值

使用 `MAX()` 函数来确定分布式表的最大分配值：

```sql
SELECT MAX(C1)
FROM dbo.T1
```

### <a name="find-the-seed-and-increment-for-the-identity-property"></a>查找 IDENTITY 属性的种子和增量

目录视图可用于通过使用以下查询来发现表的标识增量和种子配置值：

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       ic.seed_value
,       ic.increment_value
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
JOIN        sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## <a name="next-steps"></a>后续步骤

- [表概述](/azure/sql-data-warehouse/sql-data-warehouse-tables-overview)
- [CREATE TABLE (Transact-SQL) IDENTITY (Property)](/sql/t-sql/statements/create-table-transact-sql-identity-property?view=azure-sqldw-latest)
- [DBCC CHECKINDENT](/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql)
