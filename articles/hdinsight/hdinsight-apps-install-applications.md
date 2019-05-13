---
title: 在 Azure HDInsight 上安装第三方应用程序
description: 了解如何在 Azure HDInsight 上安装第三方 Hadoop 应用程序。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: hrasheed
ms.openlocfilehash: b916cbf690544d5887569e052a0456ca952140c8
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409543"
---
# <a name="install-third-party-apache-hadoop-applications-on-azure-hdinsight"></a>在 Azure HDInsight 上安装第三方 Apache Hadoop 应用程序

了解如何在 Azure HDInsight 上安装第三方 [Apache Hadoop](https://hadoop.apache.org/) 应用程序。 有关如何安装自己的应用程序的说明，请参阅[安装自定义 HDInsight 应用程序](hdinsight-apps-install-custom-applications.md)。

HDInsight 应用程序是用户可以在 HDInsight 群集上安装的应用程序。 这些应用程序可能是 Microsoft、独立软件供应商 (ISV) 或自己开发的。  

以下列表显示已发布的应用程序：

|应用程序 |群集类型 | 描述 |
|---|---|---|
|[AtScale 智能平台](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/atscale.atscale) |Hadoop |AtScale 将允许您查询几十亿行数据以交互方式使用 BI 工具已经知道，拥有，并喜爱 – 从 Microsoft Excel、 Power BI、 Tableau Software qlikview 到横向扩展 OLAP 服务器，在 HDInsight 群集。 |
|[CDAP for HDInsight](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/cask.cdap-for-hdinsight) |Hbase |CDAP 是第一款用于处理大数据的统一集成平台，可以加速的 Hadoop 的价值实现，使 IT 人员能够提供自助服务数据。 CDAP 采用开源设计并且可扩展，消除了创新所面临的障碍。 要求：4 个区域节点，至少需要安装 D3 v2。 |
|[Datameer](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/datameer.datameer) |Hadoop |Datameer 是一个可缩放的自助服务平台，用于准备、浏览和管理要分析的数据，可以加速将复杂多源数据转变为随时可在业务中使用的宝贵信息，以企业规模提供更快、更智能的见解。 |
|[在 HDInsight 上的 Dataiku DSS](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/dataiku.dss-on-hdi) |Hadoop、Spark |企业数据科学平台中的 Dataiku DSS，可让数据科学家和数据分析师更有效地协作设计和运行新的数据产品与服务，将原始数据转变为有影响力的预测结果。 |
|[WANdisco Fusion HDI App](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/wandisco.fusion-hdi-app) |Hadoop、Spark、HBase、Storm、Kafka |在分布式环境中保持数据一致性是一个很大的数据操作难题。 企业级的软件平台 WANdisco Fusion 可在任一环境中实现非结构化数据的一致性，从而可以解决此问题。 |
|[HDInsight 的 H2O SparklingWater](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/h2o-ai.h2o-sparklingwater) |Spark |H2O Sparkling Water 支持以下分布式算法：GLM、朴素贝叶斯、分布式随机森林、梯度提升机、深度神经网络、深度学习、K 平均、PCA、广义低秩模型、异常情况检测和自编码器。 |
|[Striim 的实时数据集成到 HDInsight](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/striim.hdinsightintegration) |Hadoop、HBase、Storm、Spark、Kafka |Striim（读作“stream”）是一个端到端流式处理数据集成和智能平台，可用于实现不同数据流的持续引入、处理和分析。 |
|[Jumbune 企业加速 bigdata 时都会分析](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/impetus-infotech-india-pvt-ltd.impetus_jumbune) |Hadoop、Spark |在较高层面，Jumbune 可通过以下方式为企业提供辅助：1. 加速基于 Tez、MapReduce 和 Spark 引擎的 Hive、Java 和 Scala 工作负荷性能。 2. 主动监视 Hadoop 群集。3. 在分布式文件系统中建立数据质量管理。 |
|[Kyligence 企业](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/kyligence.kyligence) |Hadoop、HBase、Spark |Kyligence Enterprise 以 Apache Kylin 为后盾，可以基于大数据实现商业智能 (BI)。 作为 Hadoop 上的企业级 OLAP 引擎，Kyligence Enterprise 可让业务分析师使用行业标准的数据仓库和 BI 方法在 Hadoop 上构建 BI。 |
|[Spark Job Server for KNIME Spark Executor](https://azuremarketplace.microsoft.com/marketplace/apps/knime.spark-job-server-for-knime-spark-executor) |Spark |适用于 KNIME Spark 执行器的 Spark 作业服务器用于将 KNIME Analytics Platform 连接到 HDInsight 群集。 |
|[Presto 的 Azure HDInsight 的星爆图案](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/starburst.starburst-presto) |Hadoop |Presto 是一个快速的可缩放分布式 SQL 查询引擎。 Presto 针对存储和计算的分离进行了架构设计，非常适用于查询 Azure Data Lake Storage、Azure Blob 存储、SQL 和 NoSQL 数据库以及其他数据源中的数据。 |
|[StreamSets Data Collector for HDInsight 云](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/streamsets.streamsets-data-collector-hdinsight) |Hadoop、HBase、Spark、Kafka |StreamSets 数据收集器是一个轻量的强大引擎，可以实时流式传输数据。 使用数据收集器可以路由和处理数据流中的数据。 该产品附带 30 天试用许可证。 |
|[Trifacta Wrangler Enterprise](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/trifacta.tr01) |Hadoop、Spark、HBase |Trifacta Wrangler Enterprise for HDInsight 支持对任何规模的数据进行企业级的数据整理。 在 Azure 上运行 Trifacta 的成本包括 Trifacta 订阅成本加上虚拟机的 Azure 基础结构成本。 |
|[Unifi 数据平台](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/unifi-software.unifi-data-platform) |Hadoop、HBase、Storm、Spark |Unifi Data Platform 是一个无缝集成的自助服务数据工具套件，旨在帮助企业用户解决推动收入增长、降低成本或运营复杂性的数据难题。 |
|[Unraveldata APM](https://azuremarketplace.microsoft.com/marketplace/apps/unravel-data.unravel-app) |Spark |适用于 HDInsight Spark 群集的 Unravel Data 应用。 |
|[吃水 AI 驱动的数据目录](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/waterline_data.waterline_data) |Spark |Waterline 使用 AI 来编录、组织和管理数据以使用业务术语自动标记数据。 Waterline 的商业文献目录是一个用于自助分析、合规性和管理，以及 IT 管理计划的关键成功组件。 |

本文提供的说明将使用 Azure 门户。 也可以从门户导出 Azure 资源管理器模板或从供应商处获取 Resource Manage 模板的副本，并使用 Azure PowerShell 和 Azure 经典 CLI 部署模板。  请参阅[使用资源管理器模板在 HDInsight 中创建 Apache Hadoop 群集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)。

## <a name="prerequisites"></a>先决条件
如果想要在现有的 HDInsight 群集上安装 HDInsight 应用程序，必须有一个 HDInsight 群集。 若要创建群集，请参阅 [创建群集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。 也可以在创建 HDInsight 群集时安装 HDInsight 应用程序。

## <a name="install-applications-to-existing-clusters"></a>将应用程序安装到现有群集
下面的过程演示如何将 HDInsight 应用程序安装到现有的 HDInsight 群集。

**安装 HDInsight 应用程序**

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中，导航到“所有服务” > “分析” > “HDInsight 群集”。
3. 在列表中选择一个 HDInsight 群集。  如果没有群集，必须先创建一个。  请参阅 [创建群集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。
4. 在“设置”类别下，选择“应用程序”。 主窗口中会显示已安装的应用程序列表。 
   
    ![HDInsight 应用程序门户菜单](./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png)
5. 在菜单中选择“+添加”。 此时会显示可用应用程序的列表。  如果“+添加”灰显，则表示没有任何应用程序适用于此版本的 HDInsight 群集。
   
    ![HDInsight 应用程序可用应用程序](./media/hdinsight-apps-install-applications/hdinsight-apps-list.png)
6. 选择某个可用的应用程序，然后遵照说明接受法律条款。

可以通过门户通知查看安装状态（选择门户顶部的铃铛图标）。 安装应用程序后，应用程序会出现在“已安装的应用”列表中。

## <a name="install-applications-during-cluster-creation"></a>在群集创建期间安装应用程序
可以选择在创建群集时安装 HDInsight 应用程序。 在此过程中，HDInsight 应用程序会在群集创建并处于运行状态后安装。 若要使用 Azure 门户创建群集期间安装应用程序，请使用**自定义**而不是默认选项**快速创建**选项。

## <a name="list-installed-hdinsight-apps-and-properties"></a>列出已安装的 HDInsight 应用和属性
门户会显示群集的已安装 HDInsight 应用程序列表，以及每个已安装应用程序的属性。

**列出 HDInsight 应用程序并显示属性**

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中，导航到“所有服务” > “分析” > “HDInsight 群集”。
3. 在列表中选择一个 HDInsight 群集。
4. 在“设置”类别下，选择“应用程序”。 主窗口中会显示已安装的应用程序列表。 
   
    ![HDInsight 应用程序已安装的应用](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png)
5. 选择某个已安装的应用程序以显示属性。 属性列表：

    |属性 | 描述 |
    |---|---|
    |应用程序名称 |应用程序名称。 |
    |状态 |应用程序状态。 |
    |网页 |已部署到边缘节点的 Web 应用程序的 URL。 此凭据与针对群集配置的 HTTP 用户凭据相同。 |
    |SSH 终结点 |可以使用 SSH 连接到边缘节点。 SSH 凭据与针对群集配置的 SSH 用户凭据相同。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](hdinsight-hadoop-linux-use-ssh-unix.md)。 |
    |描述 | 应用程序说明。 |

6. 若要删除应用程序，请右键单击应用程序，并单击上下文菜单中的“删除”。

## <a name="connect-to-the-edge-node"></a>连接到边缘节点
可以使用 HTTP 和 SSH 连接到边缘节点。 可在 [门户](#list-installed-hdinsight-apps-and-properties)中找到终结点信息。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](hdinsight-hadoop-linux-use-ssh-unix.md)。

HTTP 终结点凭据是针对 HDInsight 群集配置的 HTTP 用户凭据；SSH 终结点凭据是针对 HDInsight 群集配置的 SSH 凭据。

## <a name="troubleshoot"></a>故障排除
请参阅 [故障排除安装问题](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation)。

## <a name="next-steps"></a>后续步骤
* [安装自定义 HDInsight 应用程序](hdinsight-apps-install-custom-applications.md)：了解如何将未发布的 HDInsight 应用程序部署到 HDInsight。
* [发布 HDInsight 应用程序](hdinsight-apps-publish-applications.md)：了解如何将自定义 HDInsight 应用程序发布到 Azure 市场。
* [MSDN：安装 HDInsight 应用程序](https://msdn.microsoft.com/library/mt706515.aspx)：了解如何定义 HDInsight 应用程序。
* [使用脚本操作自定义基于 Linux 的 HDInsight 群集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用脚本操作安装其他应用程序。
* [使用资源管理器模板在 HDInsight 中创建基于 Linux 的 Apache Hadoop 群集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)：了解如何调用资源管理器模板创建 HDInsight 群集。
* [在 HDInsight 中使用空边缘节点](hdinsight-apps-use-edge-node.md)：了解如何使用空边缘节点访问 HDInsight 群集、测试 HDInsight 应用程序以及托管 HDInsight 应用程序。

