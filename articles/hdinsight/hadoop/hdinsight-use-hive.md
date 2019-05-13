---
title: 什么是 Apache Hive 和 HiveQL - Azure HDInsight
description: Apache Hive 是适用于 Apache Hadoop 的数据仓库系统。 可以使用类似于 Transact-SQL 的 HiveQL 查询 Hive 中存储的数据。 本文档介绍如何在 Azure HDInsight 中使用 Hive 和 HiveQL。
keywords: hiveql,什么是 hive,hadoop hiveql,如何使用 hive,了解 hive,hive 是什么
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 03/26/2019
ms.openlocfilehash: fbf10e992843d9297363398ca5ea13b8e0aa10a6
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64715622"
---
# <a name="what-is-apache-hive-and-hiveql-on-azure-hdinsight"></a>Azure HDInsight 中的 Apache Hive 和 HiveQL 是什么？

[Apache Hive](https://hive.apache.org/) 是适用于 Apache Hadoop 的数据仓库系统。 使用 Hive 可以汇总、查询和分析数据。 Hive 查询使用 HiveQL 编写，它是类似于 SQL 的查询语言。

Hive 允许在很大程度上未结构化的数据上投影结构。 定义结构后，可以使用 HiveQL 来查询这些数据，而无需具备 Java 或 MapReduce 方面的知识。

HDInsight 提供已针对特定工作负荷进行优化的多种群集类型。 以下群集类型最常用于 Hive 查询：

* __交互式查询__：提供[低延迟分析处理 (LLAP)](https://cwiki.apache.org/confluence/display/Hive/LLAP) 功能的 Hadoop 群集，可改善交互式查询的响应时间。 有关详细信息，请参阅 [HDInsight 中的交互式查询入门](../interactive-query/apache-interactive-query-get-started.md)文档。

* __Hadoop__：针对批处理工作负荷进行优化的 Hadoop 群集。 有关详细信息，请参阅 [HDInsight 中的 Apache Hadoop 入门](../hadoop/apache-hadoop-linux-tutorial-get-started.md)文档。

* __Spark__：Apache Spark 提供内置的功能用于使用 Hive。 有关详细信息，请参阅 [HDInsight 中的 Apache Spark 入门](../spark/apache-spark-jupyter-spark-sql.md)文档。

* __HBase__：HiveQL 可用于查询 Apache HBase 中存储的数据。 有关详细信息，请参阅 [Apache HBase on HDInsight 入门](../hbase/apache-hbase-tutorial-get-started-linux.md)文档。

## <a name="how-to-use-hive"></a>如何使用 Hive

使用下表来了解将 Hive 与 HDInsight 配合使用的各种方法：

| **使用此方法**，如果想要... | ...**交互式**查询 | ...**批处理** | ...从此**客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [Visual Studio Code 的 HDInsight 工具](../hdinsight-for-vscode.md) |✔ |✔ | Linux、Unix、Mac OS X 或 Windows |
| [用于 Visual Studio 的 HDInsight 工具](../hadoop/apache-hadoop-use-hive-visual-studio.md) |✔ |✔ |Windows |
| [Hive 视图](../hadoop/apache-hadoop-use-hive-ambari-view.md) |✔ |✔ |任何（基于浏览器） |
| [Beeline 客户端](../hadoop/apache-hadoop-use-hive-beeline.md) |✔ |✔ |Linux、Unix、Mac OS X 或 Windows |
| [REST API](../hadoop/apache-hadoop-use-hive-curl.md) |&nbsp; |✔ |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](../hadoop/apache-hadoop-use-hive-powershell.md) |&nbsp; |✔ |Windows |


## <a name="hiveql-language-reference"></a>HiveQL 语言参考

HiveQL 语言参考可在[语言手册 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) 中找到。

## <a name="hive-and-data-structure"></a>Hive 和数据结构

Hive 知道如何处理结构化和半结构化数据。 例如，字段由特定字符分隔的文本文件。 以下 HiveQL 语句基于空格分隔的数据创建一个表：

```hiveql
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
```

Hive 还支持对复杂或不规则的结构化数据使用自定义**序列化程序/反序列化程序 (SerDe)**。 有关详细信息，请参阅[如何将自定义 JSON SerDe 与 HDInsight 配合使用](https://web.archive.org/web/20190217104719/https://blogs.msdn.microsoft.com/bigdatasupport/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight/)文档。

有关 Hive 支持的文件格式的详细信息，请参阅[语言手册 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

### <a name="hive-internal-tables-vs-external-tables"></a>Hive 内部表与外部表

使用 Hive 可以创建两种类型的表：

* __内部__：数据存储在 Hive 数据仓库中。 数据仓库位于群集的默认存储中的 `/hive/warehouse/` 上。

    当下列条件之一成立时，请使用内部表：

    * 数据是临时性的。
    * 希望 Hive 管理表和数据的生命周期。

* __外部__：数据存储在数据仓库外部。 数据可以存储在可由群集访问的任何存储中。

    当下列条件之一成立时，请使用外部表：

    * 数据也在 Hive 外部使用。 例如，另一个进程（不会锁定文件）会更新数据文件。
    * 数据需要保留在基础位置，即使在删除表之后。
    * 需要一个自定义位置，例如非默认存储帐户。
    * 由 Hive 以外的某个程序管理数据格式、位置等。

有关详细信息，请参阅 [Hive 内部和外部表简介][cindygross-hive-tables]博客文章。

## <a name="user-defined-functions-udf"></a>用户定义函数 (UDF)

还可以通过**用户定义函数 (UDF)** 扩展 Hive。 UDF 允许实现 HiveQL 中不容易建模的功能或逻辑。 有关将 UDF 与 Hive 配合使用的示例，请参阅以下文档：

* [将 Java 用户定义函数与 Apache Hive 配合使用](../hadoop/apache-hadoop-hive-java-udf.md)

* [将 Python 用户定义函数与 Apache Hive 配合使用](../hadoop/python-udf-hdinsight.md)

* [将 C# 用户定义函数与 Apache Hive 配合使用](../hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [如何将自定义的 Apache Hive 用户定义函数添加到 HDInsight](https://blogs.msdn.com/b/bigdatasupport/archive/2014/01/14/how-to-add-custom-hive-udfs-to-hdinsight.aspx)

* [将日期/时间格式转换为 Apache Hive 时间戳的 Hive 用户定义函数示例](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## <a id="data"></a>示例数据

Hive on HDInsight 中预先加载了名为 `hivesampletable` 的内部表。 HDInsight 还提供可与 Hive 配合使用的示例数据集。 这些数据集存储在 `/example/data` 和 `/HdiSamples` 目录中。 这些目录存在于群集的默认存储中。

## <a id="job"></a>Hive 查询示例

以下 HiveQL 语句将各列投影到 `/example/data/sample.log` 文件上：

```hiveql
DROP TABLE log4jLogs;
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs 
    WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' 
    GROUP BY t4;
```

在上例中，HiveQL 语句执行以下操作：


* `DROP TABLE`：如果该表已存在，则删除它。

* `CREATE EXTERNAL TABLE`：在 Hive 中创建一个新的“外部”表。 外部表只会在 Hive 中存储表定义。 数据以原始格式的形式保留在原始位置中。

* `ROW FORMAT`：让 Hive 知道数据的格式已如何进行了设置。 在此情况下，每个日志中的字段以空格分隔。

* `STORED AS TEXTFILE LOCATION`：让 Hive 知道数据的存储位置（`example/data` 目录），并且数据已存储为文本。 数据可以在一个文件中，也可以分散在目录的多个文件内。

* `SELECT`：选择“t4”列中包含值“[ERROR]”的所有行的计数。 此语句返回的值为 **3**，因为有三行包含此值。

* `INPUT__FILE__NAME LIKE '%.log'` - Hive 会尝试向目录中的所有文件应用架构。 在这种情况下，目录包含与架构不匹配的文件。 为防止结果中包含垃圾数据，此语句指示 Hive 应当仅返回以 .log 结尾的文件中的数据。

> [!NOTE]  
> 如果希望通过外部源更新基础数据，应使用外部表。 例如，自动化数据上传过程或 MapReduce 操作。
>
> 删除外部表**不会**删除数据，只会删除表定义。

若要创建**内部**表而非外部表，请使用以下 HiveQL：

```hiveql
CREATE TABLE IF NOT EXISTS errorLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
STORED AS ORC;
INSERT OVERWRITE TABLE errorLogs
SELECT t1, t2, t3, t4, t5, t6, t7 
    FROM log4jLogs WHERE t4 = '[ERROR]';
```

这些语句将执行以下操作：

* `CREATE TABLE IF NOT EXISTS`：如果该表不存在，则创建该表。 由于未使用 **EXTERNAL** 关键字，因此，此语句创建一个内部表。 该表存储在 Hive 数据仓库中，并完全由 Hive 管理。

* `STORED AS ORC`：以优化的行纵栏式 (ORC) 格式存储数据。 ORC 是高度优化且有效的 Hive 数据存储格式。

* `INSERT OVERWRITE ... SELECT`：从包含“[ERROR]”的“log4jLogs”表中选择行，然后将数据插入“errorLogs”表中。

> [!NOTE]  
> 与外部表不同，删除内部表会同时删除基础数据。

## <a name="improve-hive-query-performance"></a>提高 Hive 查询性能

### <a id="usetez"></a>Apache Tez

[Apache Tez](https://tez.apache.org) 是可让数据密集型应用程序（例如 Hive）大规模高效运行的框架。 默认情况下启用 Tez。  [Apache Hive on Tez 设计文档](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)包含有关实现选项和优化配置的详细信息。

### <a name="low-latency-analytical-processing-llap"></a>低延迟分析处理 (LLAP)

[LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP)（有时称为 Live Long and Process）是 Hive 2.0 中的一项新功能，可对查询进行内存中缓存。 LLAP 能够大幅加快 Hive 查询的速度，[在某些情况下，速度比 Hive 1.x 要快 2.6 倍](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/)。

HDInsight 在交互式查询群集类型中提供 LLAP。 有关详细信息，请参阅[交互式查询入门](../interactive-query/apache-interactive-query-get-started.md)文档。

## <a name="scheduling-hive-queries"></a>计划 Hive 查询

在计划的或按需工作流中，可以使用多个服务来运行 Hive 查询。

### <a name="azure-data-factory"></a>Azure 数据工厂

Azure 数据工厂允许将 HDInsight 用作数据工厂管道的一部分。 有关通过管道使用 Hive 的详细信息，请参阅[在 Azure 数据工厂中使用 Hive 活动转换数据](../../data-factory/transform-data-using-hadoop-hive.md)文档。

### <a name="hive-jobs-and-sql-server-integration-services"></a>Hive 作业和 SQL Server Integration Services

可以使用 SQL Server Integration Services (SSIS) 来运行 Hive 作业。 Azure Feature Pack for SSIS 提供适用于 HDInsight 上的 Hive 作业的以下组件。

* [Azure HDInsight Hive 任务][hivetask]

* [Azure 订阅连接管理器][connectionmanager]

有关详细信息，请参阅 [Azure 功能包][ssispack]文档。

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie 是一个管理 Hadoop 作业的工作流和协调系统。 有关将 Oozie 与 Hive 配合使用的详细信息，请参阅[使用 Apache Oozie 定义和运行工作流](../hdinsight-use-oozie-linux-mac.md)文档。

## <a id="nextsteps"></a>后续步骤

现在，已了解什么是 Hive，以及如何将它与 HDInsight 中的 Hadoop 配合使用，请使用以下链接来学习 Azure HDInsight 的其他用法。

* [将数据上传到 HDInsight][hdinsight-upload-data]
* [将 Apache Pig 和 HDInsight 配合使用][hdinsight-use-pig]
* [将 MapReduce 作业与 HDInsight 配合使用][hdinsight-use-mapreduce]

[azure-purchase-options]: https://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: https://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: https://azure.microsoft.com/pricing/free-trial/

[apache-tez]: https://tez.apache.org
[apache-hive]: https://hive.apache.org/
[apache-log4j]: https://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: https://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/
[hivetask]: https://msdn.microsoft.com/library/mt146771(v=sql.120).aspx
[connectionmanager]: https://msdn.microsoft.com/library/mt146773(v=sql.120).aspx
[ssispack]: https://msdn.microsoft.com/library/mt146770(v=sql.120).aspx

[hdinsight-use-pig]: hdinsight-use-pig.md
[hdinsight-use-oozie]: hdinsight-use-oozie-linux-mac.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md


[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: ../hdinsight-upload-data.md

[Powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-here-strings]: https://technet.microsoft.com/library/ee692792.aspx


[cindygross-hive-tables]: https://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx
