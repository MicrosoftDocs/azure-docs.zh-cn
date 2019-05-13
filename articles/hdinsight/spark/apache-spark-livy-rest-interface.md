---
title: 使用 Livy Spark 向 Azure HDInsight 上的 Spark 群集提交作业
description: 了解如何使用 Apache Spark REST API 将 Spark 作业远程提交到 Azure HDInsight 群集。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 11/06/2018
ms.openlocfilehash: 2c1497589153f1dc5a79cc1d3414966deaf11f21
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228102"
---
# <a name="use-apache-spark-rest-api-to-submit-remote-jobs-to-an-hdinsight-spark-cluster"></a>使用 Apache Spark REST API 将远程作业提交到 HDInsight Spark 群集

了解如何使用 [Apache Livy](https://livy.incubator.apache.org/)，这是 [Apache Spark](https://spark.apache.org/) REST API，用来将远程作业提交到 Azure HDInsight Spark 群集。 有关详细文档，请参阅 [https://livy.incubator.apache.org/](https://livy.incubator.apache.org/)。

可以使用 Livy 运行交互式 Spark shell，或提交要在 Spark 上运行的批处理作业。 本文介绍如何使用 Livy 提交批处理作业。 本文中的代码片段使用 cURL 向Livy Spark 终结点发出 REST API 调用。

**先决条件：**

* HDInsight 上的 Apache Spark 群集。 有关说明，请参阅[在 Azure HDInsight 中创建 Apache Spark 群集](apache-spark-jupyter-spark-sql.md)。

* [cURL](https://curl.haxx.se/)。 本文使用 cURL 演示如何对 HDInsight Spark 群集发出 REST API 调用。

## <a name="submit-an-apache-livy-spark-batch-job"></a>提交 Apache Livy Spark 批处理作业
在提交批处理作业之前，必须将应用程序 jar 上传到与群集关联的群集存储。 可以使用命令行实用工具 [**AzCopy**](../../storage/common/storage-use-azcopy.md) 来执行此操作。 可以使用其他各种客户端来上传数据。 有关详细信息，请参阅[在 HDInsight 中上传 Apache Hadoop 作业的数据](../hdinsight-upload-data.md)。

    curl -k --user "<hdinsight user>:<user password>" -v -H "Content-Type: application/json" -X POST -d '{ "file":"<path to application jar>", "className":"<classname in jar>" }' 'https://<spark_cluster_name>.azurehdinsight.net/livy/batches' -H "X-Requested-By: admin"

**示例**：

* 如果 jar 文件位于群集存储 (WASB) 中
  
        curl -k --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST -d '{ "file":"wasb://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
* 如果想要传递 jar 文件名和类名作为输入文件（在本示例中为 input.txt）的一部分
  
        curl -k  --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"

## <a name="get-information-on-livy-spark-batches-running-on-the-cluster"></a>获取在群集上运行的 Spark 批处理的相关信息
    curl -k --user "<hdinsight user>:<user password>" -v -X GET "https://<spark_cluster_name>.azurehdinsight.net/livy/batches"

**示例**：

* 如果想要检索在群集上运行的所有Livy Spark 批处理：
  
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches" 
* 如果想要检索具有给定 batchId 的特定批
  
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"

## <a name="delete-a-livy-spark-batch-job"></a>删除 Livy Spark 批处理作业
    curl -k --user "<hdinsight user>:<user password>" -v -X DELETE "https://<spark_cluster_name>.azurehdinsight.net/livy/batches/{batchId}"

**示例**：

    curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"

## <a name="livy-spark-and-high-availability"></a>Livy Spark 和高可用性
Livy 可为群集上运行的 Spark 作业提供高可用性。 下面是几个示例。

* 将作业远程提交到 Spark 群集之后，如果 Livy 服务关闭，作业将继续在后台运行。 当 Livy 恢复运行时，会还原并报告作业的状态。
* 适用于 HDInsight 的 Jupyter 笔记本由后端中的 Livy 提供支持。 如果在 Notebook 运行 Spark 作业时重启 Livy 服务，Notebook 会继续运行代码单元。 

## <a name="show-me-an-example"></a>举个例子
本部分通过示例介绍如何使用 Livy Spark 提交批处理作业、监视作业进度，并删除作业。 本示例中使用的应用程序是[创建独立的 Scala 应用程序并在 HDInsight Spark 群集上运行](apache-spark-create-standalone-application.md)一文中开发的应用程序。 此处的步骤假设：

* 已将应用程序 jar 复制到与群集关联的存储帐户。
* 已将 CuRL 安装在用于执行这些步骤的计算机上。

执行以下步骤：

1. 让我们首先验证 Livy Spark 是否正在群集上运行。 为此，我们可以获取正在运行的批的列表。 首次使用 Livy 运行作业时，输出应返回零。
   
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches"
   
    应会看到类似于以下代码片段的输出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:47:53 GMT
        < Content-Length: 34
        <
        {"from":0,"total":0,"sessions":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    请注意输出中的最后一行显示为 **total:0**，这意味着未运行任何批。

2. 现在，让我们提交批处理作业。 以下代码片段使用输入文件 (input.txt) 传递 jar 名称和类名称作为参数。 如果要从 Windows 计算机运行这些步骤，我们建议使用输入文件。
   
        curl -k --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
   
    文件 **input.txt** 中的参数定义如下：
   
        { "file":"wasb:///example/jars/SparkSimpleApp.jar", "className":"com.microsoft.spark.example.WasbIOTest" }
   
    应会看到类似于以下代码片段的输出：
   
        < HTTP/1.1 201 Created
        < Content-Type: application/json; charset=UTF-8
        < Location: /0
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:51:30 GMT
        < Content-Length: 36
        <
        {"id":0,"state":"starting","log":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    请注意输出的最后一行显示为 **state:starting**。 此外显示了 **id:0**。 此处的 **0** 为批 ID。

3. 现在，可以使用批 ID 来检索此特定批的状态。
   
        curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/0"
   
    应会看到类似于以下代码片段的输出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Fri, 20 Nov 2015 23:54:42 GMT
        < Content-Length: 509
        <
        {"id":0,"state":"success","log":["\t diagnostics: N/A","\t ApplicationMaster host: 10.0.0.4","\t ApplicationMaster RPC port: 0","\t queue: default","\t start time: 1448063505350","\t final status: SUCCEEDED","\t tracking URL: http://hn0-myspar.lpel1gnnvxne3gwzqkfq5u5uzh.jx.internal.cloudapp.net:8088/proxy/application_1447984474852_0002/","\t user: root","15/11/20 23:52:47 INFO Utils: Shutdown hook called","15/11/20 23:52:47 INFO Utils: Deleting directory /tmp/spark-b72cd2bf-280b-4c57-8ceb-9e3e69ac7d0c"]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    现在，输出显示 **state:success**，这意味着作业已成功完成。

4. 现在，可以根据需要删除该批。
   
        curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/0"
   
    应会看到类似于以下代码片段的输出：
   
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=UTF-8
        < Server: Microsoft-IIS/8.5
        < X-Powered-By: ARR/2.5
        < X-Powered-By: ASP.NET
        < Date: Sat, 21 Nov 2015 18:51:54 GMT
        < Content-Length: 17
        <
        {"msg":"deleted"}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
   
    输出的最后一行显示批已成功删除。 删除正在运行的作业也会终止该作业 如果删除已完成的作业，则不管该作业是否已成功完成，都将完全删除该作业的信息。

## <a name="updates-to-livy-configuration-starting-with-hdinsight-35-version"></a>更新到从 HDInsight 3.5 版本开始的 Livy 配置

HDInsight 3.5 群集及更高版本群集默认情况下禁止使用本地文件路径访问示例数据文件或 jar。 建议改用 `wasb://` 路径访问群集中的 jar 或示例数据文件。 

## <a name="submitting-livy-jobs-for-a-cluster-within-an-azure-virtual-network"></a>在 Azure 虚拟网络中提交群集的 Livy 作业

如果从 Azure 虚拟网络内部连接到 HDInsight Spark 群集，可以直接连接到群集上的 Livy。 在这种情况下，Livy 终结点的 URL 为 `http://<IP address of the headnode>:8998/batches`。 此处的 **8998** 是群集头节点上运行 Livy 的端口。 有关在非公共端口上访问服务的详细信息，请参阅 [HDInsight 上的 Apache Hadoop 服务使用的端口](../hdinsight-hadoop-port-settings-for-services.md)。





## <a name="next-step"></a>后续步骤

* [Apache Livy REST API 文档](https://livy.incubator.apache.org/docs/latest/rest-api.html)
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight（跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业）](apache-spark-job-debugging.md)

