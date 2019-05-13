---
title: 教程：Apache Spark 结构化流式处理与 Apache Kafka - Azure HDInsight
description: 了解如何使用 Apache Spark 流式处理将数据传入或传出 Apache Kafka。 本教程使用 Spark on HDInsight 中的 Jupyter Notebook 流式传输数据。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,seodec18
ms.topic: tutorial
ms.date: 11/06/2018
ms.author: hrasheed
ms.openlocfilehash: 388ce607cf75a12705c9a32fe19086dbf9f15e71
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64707993"
---
# <a name="tutorial-use-apache-spark-structured-streaming-with-apache-kafka-on-hdinsight"></a>教程：将 Apache Spark 结构化流式处理与 Apache Kafka on HDInsight 配合使用

本教程说明如何使用 [Apache Spark 结构化流式处理](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)和 [Apache Kafka](https://kafka.apache.org/) on Azure HDInsight 来读取和写入数据。

Spark 结构化流式处理是建立在 Spark SQL 上的流处理引擎。 这允许以与批量计算相同的方式表达针对静态数据的流式计算。  

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 将结构化流式处理与 Kafka 配合使用
> * 创建 Kafka 和 Spark 群集
> * 将 Notebook 上传到 Spark
> * 使用 Notebook
> * 清理资源

完成本文档中的步骤后，请记得删除这些群集，避免支付额外费用。

## <a name="prerequisites"></a>先决条件

* 熟悉 [Jupyter Notebook](https://jupyter.org/) 和 Spark on HDInsight 的结合使用。 有关详细信息，请参阅[使用 Apache Spark on HDInsight 加载数据并运行查询](spark/apache-spark-load-data-run-query.md)文档。

* 熟悉 [Scala](https://www.scala-lang.org/) 编程语言。 本教程所使用的代码是使用 Scala 编写的。

* 熟悉 Kafka 主题的创建。 有关详细信息，请参阅 [Apache Kafka on HDInsight 快速入门](kafka/apache-kafka-get-started.md)文档。

> [!IMPORTANT]  
> 本文档中的步骤需要一个包含 Spark on HDInsight 和 Kafka on HDInsight 群集的 Azure 资源组。 这些群集都位于 Azure 虚拟网络中，允许 Spark 群集直接与 Kafka 群集进行通信。
> 
> 为方便起见，本文档链接到了一个模板，该模板可创建所有所需 Azure 资源。 
>
> 有关在虚拟网络中使用 HDInsight 的详细信息，请参阅[使用虚拟网络扩展 HDInsight](hdinsight-extend-hadoop-virtual-network.md) 文档。

## <a name="structured-streaming-with-apache-kafka"></a>将结构化流式处理与 Apache Kafka 配合使用

Spark 结构化流式处理是建立在 Spark SQL 引擎上的流处理引擎。 使用结构化流式处理时，可以使用与编写批处理查询相同的方式来编写流式处理查询。

以下代码片段演示了从 Kafka 读取数据并存储到文件。 第一个为批处理操作，而第二个为流式处理操作：

```scala
// Read a batch from Kafka
val kafkaDF = spark.read.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()
// Select data and write to file
kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .write
                .format("parquet")
                .option("path","/example/batchtripdata")
                .option("checkpointLocation", "/batchcheckpoint")
                .save()
```

```scala
// Stream from Kafka
val kafkaStreamDF = spark.readStream.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()
// Select data from the stream and write to file
kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .writeStream
                .format("parquet")
                .option("path","/example/streamingtripdata")
                .option("checkpointLocation", "/streamcheckpoint")
                .start.awaitTermination(30000)
```

在这两个代码片段中，从 Kafka 读取数据并写入文件。 示例之间的区别如下：

| Batch | 流式处理 |
| --- | --- |
| `read` | `readStream` |
| `write` | `writeStream` |
| `save` | `start` |

流式处理操作还使用 `awaitTermination(30000)`，这会在 30000 毫秒后停止流。 

若要将结构化流式处理与 Kafka 配合使用，项目必须具有针对 `org.apache.spark : spark-sql-kafka-0-10_2.11` 包的依赖项。 此包的版本应与 Spark on HDInsight 的版本相匹配。 对于 Spark 2.2.0（已在 HDInsight 3.6 中提供），可以在 [https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar](https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar) 找到不同项目类型的依赖项信息。

对于随本教程提供的 Jupyter Notebook，以下单元格会加载此包依赖项：

```
%%configure -f
{
    "conf": {
        "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
        "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11"
    }
}
```

## <a name="create-the-clusters"></a>创建群集

Apache Kafka on HDInsight 不提供通过公共 Internet 访问 Kafka 中转站的权限。 使用 Kafka 的任何项都必须位于同一 Azure 虚拟网络中。 在本教程中，Kafka 和 Spark 群集位于同一 Azure 虚拟网络。 

下图显示通信在 Spark 和 Kafka 之间的流动方式：

![Azure 虚拟网络中的 Spark 和 Kafka 群集图表](./media/hdinsight-apache-spark-with-kafka/spark-kafka-vnet.png)

> [!NOTE]  
> Kafka 服务仅限于虚拟网络内的通信。 通过 Internet 可访问群集上的其他服务，例如 SSH 和 Ambari。 有关可用于 HDInsight 的公共端口的详细信息，请参阅 [HDInsight 使用的端口和 URI](hdinsight-hadoop-port-settings-for-services.md)。

若要创建 Azure 虚拟网络，然后在其中创建 Kafka 和 Spark 群集，请使用以下步骤：

1. 使用以下按钮登录到 Azure，并在 Azure 门户中打开模板。
    
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-kafka-structured-streaming%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apache-spark-with-kafka/deploy-to-azure.png" alt="Deploy to Azure"></a>
    
    Azure 资源管理器模板位于 **https://raw.githubusercontent.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming/master/azuredeploy.json** 。

    此模板可创建以下资源：

   * Kafka on HDInsight 3.6 群集。
   * Spark 2.2.0 on HDInsight 3.6 群集。
   * 包含 HDInsight 群集的 Azure 虚拟网络。

     > [!IMPORTANT]  
     > 本教程使用的结构化流式处理笔记本需要 Spark 2.2.0 on HDInsight 3.6。 如果使用早期版本的 Spark on HDInsight，则使用笔记本时会收到错误消息。

2. 使用以下信息填充“自定义模板”部分的条目：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | Azure 订阅 |
    | 资源组 | 包含资源的资源组。 |
    | 位置 | 创建资源时所在的 Azure 区域。 |
    | Spark 群集名称 | Spark 群集的名称。 前六个字符必须与 Kafka 群集名称不同。 |
    | Kafka 群集名称 | Kafka 群集的名称。 前六个字符必须与 Spark 群集名称不同。 |
    | 群集登录用户名 | 群集的管理员用户名。 |
    | 群集登录密码 | 群集的管理员用户密码。 |
    | SSH 用户名 | 要为群集创建的 SSH 用户。 |
    | SSH 密码 | 用于 SSH 用户的密码。 |
   
    ![自定义模板的屏幕截图](./media/hdinsight-apache-kafka-spark-structured-streaming/spark-kafka-template.png)

3. 阅读“条款和条件”，然后选择“我同意上述条款和条件”

4. 最后，选中“固定到仪表板”，并选择“购买”。 

> [!NOTE]  
> 创建群集可能需要长达 20 分钟的时间。

## <a name="upload-the-notebook"></a>上传 Notebook

若要将项目中的 Notebook 上传到 Spark on HDInsight 群集，请使用下列步骤：

1. 从 [https://github.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming](https://github.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming) 下载项目。

1. 在 Web 浏览器中，连接到 Spark 群集上的 Jupyter 笔记本。 在下列 URL 中，将 `CLUSTERNAME` 替换为你的 __Spark__ 群集名：

        https://CLUSTERNAME.azurehdinsight.net/jupyter

    出现提示时，输入创建群集时使用的群集登录名（管理员）和密码。

2. 在页面右上角，使用“上传”按钮将 __spark-structured-streaming-kafka.ipynb__ 文件上传到群集。 选择“打开”开始上传。

    ![使用“上传”按钮来选择并上传笔记本](./media/hdinsight-apache-kafka-spark-structured-streaming/upload-button.png)

    ![选择 KafkaStreaming.ipynb 文件](./media/hdinsight-apache-kafka-spark-structured-streaming/select-notebook.png)

3. 在 Notebook 列表中找到 __spark-structured-streaming-kafka.ipynb__ 项，然后选择其旁边的“上传”按钮。

    ![若要上传笔记本，请使用 KafkaStreaming.ipynb 项的“上传”按钮](./media/hdinsight-apache-kafka-spark-structured-streaming/upload-notebook.png)


## <a name="use-the-notebook"></a>使用 Notebook

上传文件后，选择 __spark-structured-streaming-kafka.ipynb__ 项，以便打开 Notebook。 若要了解如何将 Spark 结构化流与 Kafka on HDInsight 配合使用，请按 Notebook 中的说明操作。

## <a name="clean-up-resources"></a>清理资源

若要清理本教程创建的资源，可以删除资源组。 删除资源组也会删除相关联的 HDInsight 群集，以及与资源组相关联的任何其他资源。

若要使用 Azure 门户删除资源组，请执行以下操作：

1. 在 Azure 门户中展开左侧的菜单，打开服务菜单，然后选择“资源组”以显示资源组的列表。
2. 找到要删除的资源组，然后右键单击列表右侧的“更多”按钮 (...)。
3. 选择“删除资源组”，然后进行确认。

> [!WARNING]  
> 创建群集后便开始 HDInsight 群集计费，删除群集后停止计费。 群集以每分钟按比例收费，因此无需再使用群集时，应始终将其删除。
> 
> 删除 Kafka on HDInsight 群集会删除存储在 Kafka 中的任何数据。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何使用 [Apache Spark 结构化流式处理](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)从 [Apache Kafka](https://kafka.apache.org/) on HDInsight 写入和读取数据。 使用以下链接，了解如何将 [Apache Storm](https://storm.apache.org/) 和 Kafka 结合使用。

> [!div class="nextstepaction"]
> [将 Apache Storm 与 Apache Kafka 配合使用](hdinsight-apache-storm-with-kafka.md)
