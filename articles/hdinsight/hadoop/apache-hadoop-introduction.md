---
title: 什么是 Apache Hadoop 和 Apache Spark 技术堆栈？ - Azure HDInsight
description: 介绍 HDInsight 以及 Apache Hadoop 和 Apache Spark 技术堆栈和组件，其中包括适用于大数据分析的 Kafka、Hive、Storm 和 HBase。
keywords: azure hadoop, hadoop azure, hadoop 简介, hadoop 技术堆栈, 什么是 hadoop 群集, hadoop 的用途
author: hrasheed-msft
ms.author: hrasheed
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.topic: overview
ms.date: 01/28/2019
ms.openlocfilehash: f4c291d5684054d606dcb438df71721bd209e5fb
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64708405"
---
# <a name="what-is-azure-hdinsight-and-the-apache-hadoop-technology-stack"></a>什么是 Azure HDInsight 和 Apache Hadoop 技术堆栈

本文介绍 Azure HDInsight 上的 Apache Hadoop。 Azure HDInsight 是适用于企业的分析服务，具有完全托管、全面且开源的特点。 可以使用开源框架，例如 Hadoop、Apache Spark、Apache Hive、LLAP、Apache Kafka、Apache Storm、R 等等。 


## <a name="what-is-hdinsight-and-the-hadoop-technology-stack"></a>什么是 HDInsight 和 Hadoop 技术堆栈？

[Apache Hadoop](https://hadoop.apache.org/) 是原始的开源框架，适用于对群集上的大数据集进行分布式处理和分析。 Hadoop 技术堆栈包括相关的软件和实用程序，例如 Apache Hive、Apache HBase、Spark、Kafka 等等。


Azure HDInsight 是 [Hortonworks Data Platform (HDP)](https://hortonworks.com/products/data-center/hdp/) 提供的 Hadoop 组件的云发行版。 可以通过 Azure HDInsight 轻松、快速且经济有效地处理大量数据。 可以使用 Hadoop、Spark、Hive、LLAP、Kafka、Storm、R 等最常用的开源框架。 可以通过这些框架启用各种各样的方案，例如提取、转换和加载 (ETL)；数据仓库操作；机器学习；IoT。

若要查看 HDInsight 上的可用 Hadoop 技术堆栈组件，请参阅[可以与 HDInsight 配合使用的组件和版本][component-versioning]。 若要详细了解 HDInsight 中的 Hadoop，请参阅 [HDInsight 的 Azure 功能页](https://azure.microsoft.com/services/hdinsight/)。

## <a name="what-is-big-data"></a>什么是大数据？

与以前相比，大数据的收集量在增加，收集速度在加快，收集格式在增多。 大数据可以是历史数据（即已存储的数据），也可以是实时数据（即从数据源流式传输的数据）。 请参阅[使用 HDInsight 的方案](#scenarios-for-using-hdinsight)，了解大数据的最常见用例。

## <a name="why-should-i-use-hadoop-on-hdinsight"></a>为何应使用 HDInsight 上的 Hadoop？

本部分列出了 Azure HDInsight 的功能。


|功能  |说明  |
|---------|---------|
|云原生     |     可以使用 Azure HDInsight 在 Azure 上为 [Hadoop](apache-hadoop-linux-tutorial-get-started.md)、 [Spark](../spark/apache-spark-jupyter-spark-sql.md)、 [交互式查询 (LLAP)](../interactive-query/apache-interactive-query-get-started.md)、 [Kafka](../kafka/apache-kafka-get-started.md)、 [Storm](../storm/apache-storm-tutorial-get-started-linux.md)、 [HBase](../hbase/apache-hbase-tutorial-get-started-linux.md) 和  [ML Services](../r-server/r-server-get-started.md) 创建优化群集。 HDInsight 还在所有生产工作负荷上提供端到端 SLA。  |
|低成本且可缩放     | 可以通过 HDInsight 纵向[缩放](../hdinsight-administer-use-portal-linux.md#scale-clusters) 工作负荷。 可以通过 [创建按需群集](../hdinsight-hadoop-create-linux-clusters-adf.md) 来降低成本，只为自己使用的东西付费。 还可以生成数据管道，使作业可操作化。 使计算和存储分离，提高性能和灵活性。 |
|既安全又合规    | HDInsight 允许通过 [Azure 虚拟网络](../hdinsight-extend-hadoop-virtual-network.md)、[加密](../hdinsight-hadoop-create-linux-clusters-with-secure-transfer-storage.md)以及与 [Azure Active Directory](../domain-joined/apache-domain-joined-introduction.md) 集成来保护企业数据资产。 HDInsight 还满足最常用的行业和政府[符合性标准](https://azure.microsoft.com/overview/trusted-cloud)。        |
|监视    | Azure HDInsight 集成 [Azure Monitor 日志](../hdinsight-hadoop-oms-log-analytics-tutorial.md)，可以通过单个界面来监视所有群集。        |
|全球推出 | 提供 HDInsight 的 [区域](https://azure.microsoft.com/regions/services/) 要多于提供任何其他大数据分析产品/服务的区域。 Azure HDInsight 还在中国和德国的 Azure 政府版中提供，满足关键主权领域的企业需求。 |  
|工作效率     |  Azure HDInsight 允许将各种适用于 Hadoop 和 Spark 的高效工具与首选的开发环境配合使用。 这些开发环境包括 [Visual Studio](apache-hadoop-visual-studio-tools-get-started.md)、[VSCode](../hdinsight-for-vscode.md)、[Eclipse](../spark/apache-spark-eclipse-tool-plugin.md) 和 [IntelliJ](../spark/apache-spark-intellij-tool-plugin.md)，可以提供 Scala、Python、R、Java 和 .NET 支持。 数据科学家也可以使用 [Jupyter](../spark/apache-spark-jupyter-notebook-kernels.md) 和 [Zeppelin](../spark/apache-spark-zeppelin-notebook.md) 等常用 Notebook 进行协作。    |
|可扩展性     |  可以使用[脚本操作](../hdinsight-hadoop-customize-cluster-linux.md)通过安装的组件（Hue、Presto 等）来扩展 HDInsight 群集，具体方法是：[添加边缘节点](../hdinsight-apps-use-edge-node.md)或[集成其他大数据认证应用程序](../hdinsight-apps-install-applications.md)。 HDInsight 允许通过[单击](https://azure.microsoft.com/services/hdinsight/partner-ecosystem/)部署方式无缝集成最常用的大数据解决方案。|

## <a name="scenarios-for-using-hdinsight"></a>使用 HDInsight 的方案

Azure HDInsight 适用于各种方案的大数据处理。 大数据可以是历史数据（已收集和存储的数据），也可以是实时数据（直接从源流式传输的数据）。 处理此类数据的方案可以汇总成以下类别： 

### <a name="batch-processing-etl"></a>批处理 (ETL)

提取、转换和加载 (ETL) 是指将非结构化或结构化数据从异类数据源中提取出来， 转换成某种结构化格式，然后加载到数据存储中这一过程。 可以将转换的数据用于数据科学或数据仓库。

### <a name="data-warehousing"></a>数据仓库

可以使用 HDInsight 对任何格式的结构化或非结构化数据执行 PB 规模的交互式查询。 也可以通过生成模型将其连接到 BI 工具。 有关详细信息，请[阅读此客户经历](https://customers.microsoft.com/story/milliman)。 

![HDInsight 体系结构：数据仓库](./media/apache-hadoop-introduction/hdinsight-architecture-data-warehouse.png "HDInsight 数据仓库体系结构")

### <a name="internet-of-things-iot"></a>物联网 (IoT)

可以使用 HDInsight 处理从各种设备实时接收的流数据。 有关详细信息，请阅读 Azure 提供的[此博客文章](https://azure.microsoft.com/blog/announcing-public-preview-of-apache-kafka-on-hdinsight-with-azure-managed-disks/)，了解使用 Azure 托管磁盘的 Apache Kafka on HDInsight 公共预览版。

![HDInsight 体系结构：物联网](./media/apache-hadoop-introduction/hdinsight-architecture-iot.png "HDInsight IoT 体系结构") 

### <a name="data-science"></a>数据科学

可以使用 HDInsight 生成从数据中提取关键见解的应用程序。 也可在此基础上使用 Azure 机器学习来预测业务的未来趋势。 有关详细信息，请[阅读此客户经历](https://customers.microsoft.com/story/pros)。

![HDInsight 体系结构：数据科学](./media/apache-hadoop-introduction/hdinsight-architecture-data-science.png "HDInsight 数据科学体系结构")

### <a name="hybrid"></a>混合

可以使用 HDInsight 将现有的本地大数据基础结构扩展到 Azure，充分利用云的高级分析功能。

![HDInsight 体系结构：混合](./media/apache-hadoop-introduction/hdinsight-architecture-hybrid.png "HDInsight 混合体系结构")

## <a name="cluster-types-in-hdinsight"></a>HDInsight 中的群集类型

HDInsight 包括特定的群集类型和群集自定义功能，例如添加组件、实用程序和语言的功能。 HDInsight 提供以下群集类型：

* **[Apache Hadoop](https://wiki.apache.org/hadoop)**：一个框架，使用 HDFS、YARN 资源管理和简单的 MapReduce 编程模型并行处理和分析批处理数据。

* **[Apache Spark](https://spark.apache.org/)**：一种开源并行处理框架，支持使用内存中处理来提升大数据分析应用程序的性能。 请参阅[什么是 HDInsight 中的 Apache Spark？](../spark/apache-spark-overview.md)

* **[Apache HBase](https://hbase.apache.org/)**：基于 Hadoop 上的 NoSQL 数据库构建，为大量非结构化和半结构化数据（可能为数十亿行乘以数百万列）提供随机访问和高度一致性。 请参阅[什么是 HDInsight 上的 HBase？](../hbase/apache-hbase-overview.md)

* **[ML Services](https://docs.microsoft.com/machine-learning-server/rebranding-microsoft-r-server)**：用于托管和管理并行分布式 R 进程的服务器。 它可让数据科研人员、统计人员和 R 程序员根据需要访问 HDInsight 上可缩放的分布式分析方法。 请参阅 [HDInsight 上的 ML Services 概述](../r-server/r-server-overview.md)。

* **[Apache Storm](https://storm.incubator.apache.org/)**：分布式实时计算系统，用于快速处理大型数据流。 Storm 以 HDInsight 中的托管群集形式提供。 请参阅 [使用 Storm 和 Hadoop 分析实时传感器数据](../storm/apache-storm-sensor-data-analysis.md)。

* **[Apache 交互查询预览（又称为：Live Long and Process）](https://cwiki.apache.org/confluence/display/Hive/LLAP)**：更快的交互式 Hive 查询的内存中缓存。 请参阅[在 HDInsight 中使用交互式查询](../interactive-query/apache-interactive-query-get-started.md)。

* **[Apache Kafka](https://kafka.apache.org/)**：一种开源平台，用于生成流式处理的数据管道和应用程序。 Kafka 还提供了消息队列功能，允许用户发布和订阅数据流。 请参阅 [Apache Kafka on HDInsight 简介](../kafka/apache-kafka-introduction.md)。

## <a name="open-source-components-in-hdinsight"></a>HDInsight 中的开源组件


通过 Azure HDInsight，可以使用 Hadoop、Spark、Hive、LLAP、Kafka、Storm、HBase 和 R 之类的开源框架来创建群集。这些群集默认情况下附带其他开源组件，例如 [Apache Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)、[Avro](https://avro.apache.org/docs/current/spec.html)、[Apache Hive](https://hive.apache.org)、[HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog/)、[Apache Mahout](https://mahout.apache.org/)、[Apache Hadoop MapReduce](https://wiki.apache.org/hadoop/MapReduce)、[Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)、[Apache Phoenix](https://phoenix.apache.org/)、[Apache Pig](https://pig.apache.org/)、[Apache Sqoop](https://sqoop.apache.org/)、[Apache Tez](https://tez.apache.org/)、[Apache Oozie](https://oozie.apache.org/)、[Apache ZooKeeper](https://zookeeper.apache.org/)。  



## <a name="programming-languages-in-hdinsight"></a>HDInsight 中的编程语言
HDInsight 群集包括 Spark、HBase、Kafka、Hadoop 和其他群集，支持多种编程语言。 某些编程语言默认情况下未安装。 对于默认情况下未安装的库、模块或程序包，请[使用脚本操作来安装组件](../hdinsight-hadoop-script-actions-linux.md)。


|编程语言  |信息  |
|---------|---------|
|默认编程语言支持     | 默认情况下，HDInsight 群集支持：<ul><li>Java</li><li>Python</li><li>.NET</li><li>Go</li></ul>  |
|Java 虚拟机 (JVM) 语言     | 除 Java 之外的许多语言都可以在 Java 虚拟机 (JVM) 上运行。 但是，运行这其中的部分语言时，可能必须在群集上安装其他组件。 HDInsight 群集支持以下基于 JVM 的语言： <ul><li>Clojure</li><li>Jython (Python for Java)</li><li>Scala</li></ul>     |
|Hadoop 特定的语言     | HDInsight 群集支持以下特定于 Hadoop 技术堆栈的语言： <ul><li>用于 Pig 作业的 Pig Latin</li><li>用于 Hive 作业的 HiveQL 和 SparkSQL</li></ul>        |

## <a name="development-tools-for-hdinsight"></a>适用于 HDInsight 的开发工具

可以使用 HDInsight 开发工具（包括 IntelliJ、Eclipse、Visual Studio Code 和 Visual Studio）通过与 Azure 的无缝集成来创作和提交 HDInsight 数据查询和作业。

* [用于 IntelliJ 的 Azure 工具包](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-intellij-tool-plugin)

* [用于 Eclipse 的 Azure 工具包](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-eclipse-tool-plugin)

* [用于 VS Code 的 Azure HDInsight 工具](https://docs.microsoft.com/azure/hdinsight/hdinsight-for-vscode?branch=pr-en-us-22999)

* [用于 Visual Studio 的 Azure Data Lake 工具](https://docs.microsoft.com/azure/hdinsight/hadoop/apache-hadoop-visual-studio-tools-get-started)

## <a name="business-intelligence-on-hdinsight"></a>HDInsight 上的商业智能

大家熟悉的商业智能 (BI) 工具使用 Power Query 外接程序或 Microsoft Hive ODBC 驱动程序来检索、分析和报告与 HDInsight 集成的数据：

* [使用 Azure HDInsight 上的数据可视化工具的 Apache Spark BI](../spark/apache-spark-use-bi-tools.md)

* [在 Azure HDInsight 中使用 Microsoft Power BI 直观显示 Apache Hive 数据](apache-hadoop-connect-hive-power-bi.md) 

* [在 Azure HDInsight 中使用 Power BI 可视化交互式查询 Hive 数据](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)


* [使用 Power Query 将 Excel 连接到 Apache Hadoop](apache-hadoop-connect-excel-power-query.md)（需使用 Windows） 


* [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Apache Hadoop](apache-hadoop-connect-excel-hive-odbc-driver.md)（需使用 Windows） 

* [将 SQL Server Analysis Services 与 HDInsight 结合使用](https://docs.microsoft.com/previous-versions/msp-n-p/dn749857(v=pandp.10))

* [将 SQL Server Reporting Services 与 HDInsight 结合使用](https://docs.microsoft.com/previous-versions/msp-n-p/dn749856(v=pandp.10))

## <a name="next-steps"></a>后续步骤

本文介绍了 Azure HDInsight 及其在 Azure 上提供 Hadoop 和其他群集类型的方式。 请继续阅读下一篇文章，了解如何在 HDInsight 中创建 Apache Hadoop 群集。

> [!div class="nextstepaction"]
> [在 HDInsight 中创建 Apache Hadoop 群集](apache-hadoop-linux-create-cluster-get-started-portal.md)


[component-versioning]: ../hdinsight-component-versioning.md
[zookeeper]: https://zookeeper.apache.org/
