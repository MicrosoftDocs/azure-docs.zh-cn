---
title: 使用 R 和 SQL 数据类型和对象
titleSuffix: Azure SQL Database Machine Learning Services (preview)
description: 了解如何使用数据类型和数据对象在 R 中使用 Azure SQL 数据库使用机器学习服务 （预览版），包括可能会遇到的常见问题。
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: r
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: davidph
manager: cgronlun
ms.date: 04/11/2019
ms.openlocfilehash: 01d3af14963e92393d34a952bddc8097b7b08f18
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232608"
---
# <a name="work-with-r-and-sql-data-in-azure-sql-database-machine-learning-services-preview"></a>使用 Azure SQL 数据库机器学习服务 （预览版） 中的 R 和 SQL 数据

本文讨论了一些 R 和 SQL 数据库之间移动数据时可能遇到的常见问题[（使用 R) Azure SQL 数据库中的机器学习服务](sql-database-machine-learning-services-overview.md)。 使用您自己的脚本中的数据时，通过此练习中获得的体验提供了基本的背景。

你可能会遇到的常见问题包括：

- 数据类型有时不匹配
- 隐式转换可能会发生
- Cast 和 convert 操作有时是必需的
- R 和 SQL 使用不同的数据对象

[!INCLUDE[ml-preview-note](../../includes/sql-database-ml-preview-note.md)]

## <a name="prerequisites"></a>必备组件

- 如果还没有 Azure 订阅，可以在开始前[创建一个帐户](https://azure.microsoft.com/free/)。

- 若要在这些练习中运行示例代码，必须先有一个启用了机器学习服务（使用 R）的 Azure SQL 数据库。 在发布公共预览版期间，Microsoft 会将你加入该版本并为你的现有数据库或新数据库启用机器学习。 执行[注册预览版](sql-database-machine-learning-services-overview.md#signup)中的步骤。

- 确保已安装了最新的 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS)。 你可以使用其他数据库管理或查询工具运行 R 脚本，但在本快速入门中，你将使用 SSMS。

## <a name="working-with-a-data-frame"></a>使用数据帧

当您的脚本结果从 R 返回到 SQL 时，它必须返回数据作为**data.frame**。 如果你想要输出为存储的过程结果的一部分，任何其他类型的对象，无论它是列表、 因子、 向量或二进制数据-在脚本中的生成必须转换成数据帧。 幸运的是，有多个 R 函数来支持其他对象更改为数据帧。 甚至可以序列化二进制模型并将其返回数据帧，您将在本文后面来执行此操作。

首先，让我们试着-向量、 矩阵和列表的一些基本 R 对象并查看如何转换成数据帧变化的输出传递给 SQL。

比较在 R 中的这两个"Hello World"脚本脚本看上去大致相同，但第一个返回三个值的单个列，而第二个返回三个列具有单个值每个。

**示例 1**

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N' mytextvariable <- c("hello", " ", "world");
OutputDataSet <- as.data.frame(mytextvariable);
'
    , @input_data_1 = N'';
```

**示例 2**

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N' OutputDataSet<- data.frame(c("hello"), " ", c("world"));'
    , @input_data_1 = N'';
```

结果为何有这么大的差别？

通常可以通过使用 R 找到答案`str()`命令。 添加函数`str(object_name)`任意位置以使数据在 R 脚本中指定的 R 对象的架构作为返回一条信息性消息。 您可以查看中的消息**消息**在 SSMS 中的选项卡。

要找出为什么示例 1 和示例 2 有这种不同的结果，请将行插入`str(OutputDataSet)`末尾的`@script`变量在每个语句中，此类定义：

**添加使用 str 函数的示例 1**

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
mytextvariable <- c("hello", " ", "world");
OutputDataSet <- as.data.frame(mytextvariable);
str(OutputDataSet);
'
    , @input_data_1 = N'  ';
```

**添加使用 str 函数的示例 2**

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
OutputDataSet <- data.frame(c("hello"), " ", c("world"));
str(OutputDataSet);
'
    , @input_data_1 = N'  ';
```

现在，查看中的文本**消息**若要查看的输出是不同的原因。

**结果-示例 1**

```text
STDOUT message(s) from external script:
'data.frame':   3 obs. of  1 variable:
$ mytextvariable: Factor w/ 3 levels " ","hello","world": 2 1 3
```

**结果-示例 2**

```text
STDOUT message(s) from external script:
'data.frame':   1 obs. of  3 variables:
$ c..hello..: Factor w/ 1 level "hello": 1
$ X...      : Factor w/ 1 level " ": 1
$ c..world..: Factor w/ 1 level "world": 1
```

如您所见，R 的语法略有变化将很大的影响结果的架构。 所有的详细信息，R 数据类型的差异中的详细信息中所述*数据结构*主题中[由 Hadley wickham 编写的"Advanced R"](http://adv-r.had.co.nz)。

现在，只需注意，您需要将 R 对象强制转换成数据框架时检查预期的结果。

> [!TIP]
> 您还可以使用 R 标识函数，如`is.matrix`， `is.vector`，以返回有关内部数据结构的信息。

## <a name="implicit-conversion-of-data-objects"></a>数据对象的隐式转换

每个 R 数据对象都有其自己的规则的结合其他数据对象，如果两个数据对象具有相同数量的维度，或任何数据对象包含异构数据类型，如何处理的值。

例如，假设你想要使用。 执行矩阵乘法你想要具有三个值的单列矩阵乘以包含四个值的数组，因此期望 4x3 矩阵。

首先，创建测试数据的小型表。

```sql
CREATE TABLE RTestData (col1 INT NOT NULL)

INSERT INTO RTestData
VALUES (1);

INSERT INTO RTestData
VALUES (10);

INSERT INTO RTestData
VALUES (100);
GO
```

现在，运行以下脚本。

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
x <- as.matrix(InputDataSet);
y <- array(12:15);
OutputDataSet <- as.data.frame(x %*% y);
'
    , @input_data_1 = N' SELECT [Col1]  from RTestData;'
WITH RESULT SETS((
            [Col1] INT
            , [Col2] INT
            , [Col3] INT
            , Col4 INT
            ));
```

事实上，三个值的列转换为单列矩阵。 由于矩阵只是一种特殊的情况，在 R 中，该数组的数组`y`隐式强制转换为单列矩阵，使两个参数符合。

**结果**

|Col1|第 2 列|Col3|第 4 列|
|---|---|---|---|
|12|13|14|15|
|120|130|140|150|
|1200|1300|1400|1500|

但请注意更改数组的大小会发生什么情况`y`。

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
x <- as.matrix(InputDataSet);
y <- array(12:14);
OutputDataSet <- as.data.frame(y %*% x);
'
    , @input_data_1 = N' SELECT [Col1]  from RTestData;'
WITH RESULT SETS(([Col1] INT));
```

现在 R 返回单个值作为结果。

**结果**
    
|Col1|
|---|
|1542|

为什么？ 在这种情况下，两个参数可以为相同长度的向量进行处理，因为 R 将以矩阵形式返回内部乘积。  这是预期的行为根据线性代数的规则。 但是，它可能导致问题，如果下游应用程序预期输出架构永远不会更改 ！

## <a name="merge-or-multiply-columns-of-different-length"></a>合并或相乘不同长度的列

R 提供了极大的灵活性，使用矢量不同的大小，以及将这些类似于列的结构组合成数据帧。 向量列表看起来像是一个表，但它们不遵循控制数据库表的所有规则。

例如，以下脚本定义长度为 6 的数值数组并将其存储在 R 变量`df1`。 该数值数组然后组合与整数 （上面创建） 的 RTestData 表包含三 （3） 值，以使新的数据帧`df2`。

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
df1 <- as.data.frame( array(1:6) );
df2 <- as.data.frame( c( InputDataSet , df1 ));
OutputDataSet <- df2;
'
    , @input_data_1 = N' SELECT [Col1]  from RTestData;'
WITH RESULT SETS((
            [Col2] INT NOT NULL
            , [Col3] INT NOT NULL
            ));
```

若要填充数据框架，R 将重复多次，以匹配的数组中的元素数从 RTestData 检索的元素`df1`。

**结果**
    
|*Col2*|*Col3*|
|----|----|
|第|1|
|10|2|
|100|3|
|1|4|
|10|5|
|100|6|

请记住，数据帧只看起来像一个表，但实际上是一个列表的向量。

## <a name="cast-or-convert-sql-data"></a>Cast 或 convert SQL 数据

R 和 SQL 使用相同的数据类型，因此如果在 SQL 获取数据，然后将其传递给 R 运行时运行的查询，某些类型的隐式转换通常会发生。 另一套转换发生时将数据从 R 返回到 SQL。

- SQL 将数据从查询推送到 R 进程，并将其转换为更高的效率的内部表示形式。
- R 运行时将数据加载到 data.frame 变量，并执行其自身的数据的操作。
- 数据库引擎将数据返回到使用受保护的内部连接的 SQL，并显示在 SQL 数据类型方面的数据。
- 您获取数据，通过连接到 SQL 使用能够发出 SQL 查询和处理表格数据集的客户端或网络库。 此客户端应用程序可能会影响其他方法中的数据。

若要阐释其工作方式上, 运行的查询类似于本[AdventureWorksDW](https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks)数据仓库。 此视图返回用于创建预测的销售数据。

```sql
USE AdventureWorksDW
GO

SELECT ReportingDate
    , CAST(ModelRegion AS VARCHAR(50)) AS ProductSeries
    , Amount
FROM [AdventureWorksDW].[dbo].[vTimeSeries]
WHERE [ModelRegion] = 'M200 Europe'
ORDER BY ReportingDate ASC
```

> [!NOTE]
> 可以使用任何版本的 AdventureWorks，也可以创建不同的查询使用你自己的数据库。 要点是尝试处理一些包含文本、 日期时间和数字值的数据。

现在，请尝试使用此查询作为存储过程的输入。

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
str(InputDataSet);
OutputDataSet <- InputDataSet;
'
    , @input_data_1 = N'
           SELECT ReportingDate
         , CAST(ModelRegion as varchar(50)) as ProductSeries
         , Amount
           FROM [AdventureWorksDW].[dbo].[vTimeSeries]
           WHERE [ModelRegion] = ''M200 Europe''
           ORDER BY ReportingDate ASC ;'
WITH RESULT SETS undefined;
```

如果遇到错误，可能需要进行一些编辑对查询文本。 例如，WHERE 子句中的字符串谓词必须用两组单引号括起来。

获取正常运行查询后，检查的结果`str`函数来了解 R 如何处理输入的数据。

**结果**

```text
STDOUT message(s) from external script: 'data.frame':    37 obs. of  3 variables:
STDOUT message(s) from external script: $ ReportingDate: POSIXct, format: "2010-12-24 23:00:00" "2010-12-24 23:00:00"
STDOUT message(s) from external script: $ ProductSeries: Factor w/ 1 levels "M200 Europe",..: 1 1 1 1 1 1 1 1 1 1
STDOUT message(s) from external script: $ Amount       : num  3400 16925 20350 16950 16950
```

- 使用 R 数据类型处理日期时间列**POSIXct**。
- 已标识为"ProductSeries"的文本列**身份**，这意味着一个分类变量。 默认情况下，作为因子处理字符串值。 如果字符串传递到 R 时，它是转换为整数供内部使用，然后映射回到输出中的字符串。

## <a name="summary"></a>摘要

从这些简短的示例，可以看到需要传递 SQL 查询中用作输入时，检查数据转换的效果。 因为某些 SQL 数据类型不受 R，请考虑这些方法，从而避免发生错误：

- 提前测试你的数据并验证列或架构中可能存在问题时传递给 R 代码中的值。
- 指定输入的数据源中的列单独，而不是使用`SELECT *`，并且知道如何处理每个列。
- 准备输入的数据，以避免意外情况时，请执行根据需要显式强制转换。
- 避免传递列的数据 （如 GUID 或 rowguid） 会导致错误并不是适用于建模。

有关支持和不支持 R 数据类型的详细信息，请参阅[R 库和数据类型](/sql/advanced-analytics/r/r-libraries-and-data-types)。
