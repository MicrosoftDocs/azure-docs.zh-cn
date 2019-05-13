---
title: 大规模提取、转换和加载 (ETL) - Azure HDInsight
description: 了解如何通过 Apache Hadoop 在 HDInsight 中使用 ETL。
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/14/2017
ms.author: ashishth
ms.openlocfilehash: a343caaa998505a1772096b058ec7ad300eec03c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64725697"
---
# <a name="extract-transform-and-load-etl-at-scale"></a>大规模提取、转换和加载 (ETL)

提取、转换和加载 (ETL) 是从各种源获取数据、将数据收集到标准位置、进行清理和处理，最终将其载入数据存储供查询的过程。 传统的 ETL 过程是导入数据、就地清理该数据，然后将其存储在关系数据引擎中。 使用 HDInsight 时，各种 Apache Hadoop 生态系统组件都支持大规模执行 ETL。 

在 ETL 过程中使用 HDInsight 可以通过以下管道来总结：

![HDInsight ETL 概述](./media/apache-hadoop-etl-at-scale/hdinsight-etl-at-scale-overview.png)

以下部分探讨每个 ETL 阶段及其关联的组件。

## <a name="orchestration"></a>业务流程

业务流程跨越 ETL 管道的所有阶段。 HDInsight 中的 ETL 作业通常涉及到多个相互配合运行的不同产品。  可以使用 Hive 来清理数据的一部分，同时使用 Pig 来清理另一部分。  可以使用 Azure 数据工厂将数据从 Azure Data Lake Store 载入 Azure SQL 数据库。

若要在适当的时间运行适当的作业，则需要业务流程。

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie 是一个管理 Hadoop 作业的工作流协调系统。 Oozie 在 HDInsight 群集中运行，并与 Hadoop 堆栈集成。 Oozie 支持 Apache Hadoop MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。 Oozie 还可用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。

有关详细信息，请参阅[在 HDInsight 中将 Apache Oozie 与 Apache Hadoop 配合使用以定义和运行工作流](../hdinsight-use-oozie-linux-mac.md)。如需深入演示如何使用 Oozie 驱动端到端管道，请参阅[操作数据管道](../hdinsight-operationalize-data-pipeline.md)。 

### <a name="azure-data-factory"></a>Azure 数据工厂

Azure 数据工厂以平台即服务的形式提供业务流程功能。 它是基于云的数据集成服务，用于在云中创建数据驱动型工作流，以便协调和自动完成数据移动和数据转换。 

使用 Azure 数据工厂可以：

1. 创建和计划数据驱动型工作流（称为管道），以便从不同的数据存储引入数据。
2. 使用计算服务（例如 Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics、Azure Batch 和 Azure 机器学习）处理和转换数据。
3. 将输出数据发布到数据存储（例如 Azure SQL 数据仓库），供商业智能 (BI) 应用程序使用。

有关 Azure 数据工厂的详细信息，请参阅[文档](../../data-factory/introduction.md)。

## <a name="ingest-file-storage-and-result-storage"></a>提取文件存储和结果存储

源数据文件通常载入 Azure 存储或 Azure Data Lake Storage 中的某个位置。 文件可以是任何格式，但通常是 CSV 等平面文件。 

### <a name="azure-storage"></a>Azure 存储 

[Azure 存储](https://azure.microsoft.com/services/storage/blobs/)具有[特定的可伸缩性目标](../../storage/common/storage-scalability-targets.md)。  对于大多数分析节点而言，在处理许多较小的文件时，Azure 存储的可伸缩性最佳。  Azure 存储可以保证无论有多少文件或文件有多大（只要在限制范围内），都能提供相同的性能。  这意味着，可以存储 TB 量级的数据，同时获得一致的性能，无论使用的是一部分数据还是全部数据。

Azure 存储包含多种不同类型的 Blob。  “追加 Blob”是存储 Web 日志或传感器数据的极佳选项。  

尽管可以在众多服务器之间分布多个 Blob 以便扩大对其的访问权限，但只能由单个服务器为单个 Blob 提供服务。 虽然 Blob 可在 Blob 容器中进行逻辑分组，但这种分组不会对分区产生影响。

Azure 存储还为 Blob 存储提供一个 WebHDFS API 层。  HDInsight 中的所有服务都可以访问 Azure Blob 存储中的文件来进行数据清理和数据处理，就像这些服务使用 Hadoop 分布式文件系统 (HDFS) 时一样。

通常使用 PowerShell、Azure 存储 SDK 或 AZCopy 将数据引入 Azure 存储。

### <a name="azure-data-lake-storage"></a>Azure Data Lake 存储

Azure Data Lake Storage (ADLS) 是一个托管的超大规模存储库，用于分析与 HDFS 兼容的数据。  ADLS 使用类似于 HDFS 的设计范例，并在总容量及单个文件的大小方面提供无限可伸缩性。 ADLS 非常适合与大型文件配合运行，因为大型文件可以跨多个节点存储。  ADLS 中的数据分区在幕后执行。  通过数以千计的并发执行程序，可高效读取和写入数百 TB 的数据，从而可获得极大的吞吐量来运行分析作业。

通常使用 Azure 数据工厂、ADLS SDK、AdlCopy 服务、Apache DistCp 或 Apache Sqoop 将数据引入 ADLS。  要使用其中的哪些服务在很大程度上取决于数据的所在位置。  如果数据当前在现有的 Hadoop 群集中，则可以使用 Apache DistCp、AdlCopy 服务或 Azure 数据工厂。  如果数据在 Azure Blob 存储中，则可以使用 Azure Data Lake Storage .NET SDK、Azure PowerShell 或 Azure 数据工厂。

ADLS 还针对使用 Azure 事件中心或 Apache Storm 运行的事件引入进行了优化。

#### <a name="considerations-for-both-storage-options"></a>两种存储选项的注意事项

如果上传的数据集达到 TB 量级，网络延迟可能是一个重要问题，尤其是当数据来自本地位置时。  这种情况下，可使用以下选项：

* Azure ExpressRoute：使用 Azure ExpressRoute 可在 Azure 数据中心与本地基础结构之间创建专用连接。 这些连接为传输大量数据提供了可靠选项。 有关详细信息，请参阅[ Azure ExpressRoute 文档](../../expressroute/expressroute-introduction.md)。

* 数据“离线上传”。 可以使用 [Azure 导入/导出服务](../../storage/common/storage-import-export-service.md)将包含数据的硬盘驱动器发送到 Azure 数据中心。 数据会首先上传到 Azure 存储 Blob。 然后可使用 [Azure 数据工厂](../../data-factory/connector-azure-data-lake-store.md)或 [AdlCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md) 工具从 Azure 存储 Blob 将数据复制到 Data Lake Storage。

### <a name="azure-sql-data-warehouse"></a>Azure SQL 数据仓库

Azure SQL DW 是存储已清理且准备好的数据供今后分析的绝佳选项。  可以使用 Azure HDInsight 对 Azure SQL DW 执行这些服务。

Azure SQL 数据仓库 (SQL DW) 是已针对分析工作负荷进行优化的关系数据库。  Azure SQL DW 根据分区表进行缩放。  表可以跨多个节点分区。  在创建 Azure SQL DW 节点时便选择了节点。  可以在事后缩放节点，但是，该主动过程可能需要移动数据。 有关详细信息，请参阅 [SQL 数据仓库 - 管理计算](../../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)。

### <a name="apache-hbase"></a>Apache HBase

Apache HBase 是 Azure HDInsight 中提供的键/值存储。  Apache HBase 是一种开源 NoSQL 数据库，它构建于 Hadoop 基础之上，并基于 Google BigTable 模型化。 HBase 针对按列系列组织的无架构数据库中的大量非结构化和结构化数据提供高性能的随机访问和强一致性。

数据存储在表的各行中，行中的数据按列系列分组。 HBase 是无架构数据库，也就是说，在使用其数据前，不必定义列以及列中存储的数据类型。 开放源代码可进行线性伸缩，以处理上千节点上数 PB 的数据。 HBase 可依赖数据冗余、批处理以及 Hadoop 生态系统中的分布式应用程序提供的其他功能。   

HBase 是供将来分析的传感器和日志数据的绝佳目标。

HBase 可伸缩性取决于 HDInsight 群集中的节点数。

### <a name="azure-sql-database-and-azure-database"></a>Azure SQL 数据库和 Azure 数据库

Azure 以平台即服务 (PAAS) 的形式提供三种不同的关系数据库。

* [Azure SQL 数据库](../../sql-database/sql-database-technical-overview.md)是 Microsoft SQL Server 的一种实现。 有关性能的详细信息，请参阅[优化 Azure SQL 数据库性能](../../sql-database/sql-database-performance-guidance.md)。
* [Azure Database for MySQL](../../mysql/overview.md) 是 Oracle MySQL 的一种实现。
* [Azure Database for PostgreSQL](../../postgresql/quickstart-create-server-database-portal.md) 是 PostgreSQL 的一种实现。

这些产品可以纵向扩展，即，可以通过添加更多 CPU 和内存来扩展它们。  还可以选择使用高级磁盘配合这些产品来获得更好的 I/O 性能。

## <a name="azure-analysis-services"></a>Azure Analysis Services 

Azure Analysis Services (AAS) 是用于决策支持和业务分析的分析数据引擎，可为业务报告和客户端应用程序（例如 Power BI、Excel、Reporting Services 报告及其他可视化工具）提供分析数据。

可以通过更改每个多维数据集的层来缩放这些分析多维数据集。  有关详细信息，请参阅 [Azure Analysis Services 定价](https://azure.microsoft.com/pricing/details/analysis-services/)。

## <a name="extract-and-load"></a>提取和加载

将数据存储到 Azure 中后，可以使用许多服务将这些数据提取和载入其他产品。  HDInsight 支持 Sqoop 和 Flume。 

### <a name="apache-sqoop"></a>Apache Sqoop

Apache Sqoop 是专用于在结构化、半结构化和非结构化数据源之间有效传输数据的工具。 

Sqoop 使用 MapReduce 导入和导出数据，可提供并行操作和容错。

### <a name="apache-flume"></a>Apache Flume

Apache Flume 是分布式、可靠且高度可用的服务，能够有效地收集、聚合与移动大量日志数据。 Flume 采用基于流式传输数据流的简单弹性体系结构。 Flume 既可靠又能容错，提供可优化的可靠性机制和许多故障转移与恢复机制。 Flume 使用一个允许联机分析应用程序的简单可扩展数据模型。

Apache Flume 无法与 Azure HDInsight 配合使用。  本地 Hadoop 安装可以使用 Flume 将数据发送到 Azure 存储 Blob 或 Azure Data Lake Storage。  有关详细信息，请参阅[将 Apache Flume 与 HDInsight 配合使用](https://web.archive.org/web/20190217104751/https://blogs.msdn.microsoft.com/bigdatasupport/2014/03/18/using-apache-flume-with-hdinsight/)。

## <a name="transform"></a>转换

将数据存储到所选的位置后，需要根据特定的使用模式清理、合并或准备这些数据。  Hive、Pig 和 Spark SQL 都是适合用于完成此类任务的极佳选择。  HDInsight 支持所有这些产品。 

## <a name="next-steps"></a>后续步骤

* [将 Apache Pig 与 Apache Hadoop on HDInsight 配合使用](hdinsight-use-pig.md)
* [使用 Apache Hive 作为 ETL 工具](apache-hadoop-using-apache-hive-as-an-etl-tool.md) 
* [将 Azure Data Lake Storage Gen2 用于 Azure HDInsight 群集](../hdinsight-hadoop-use-data-lake-storage-gen2.md)
