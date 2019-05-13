---
title: 快速入门：使用 Azure PowerShell 在 HDInsight 中创建 Spark 群集
description: 本快速入门演示如何使用 Azure PowerShell 在 Azure HDInsight 中创建 Apache Spark 群集，以及如何运行简单的 Spark SQL 查询。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.date: 05/03/2019
ms.author: hrasheed
ms.custom: mvc
ms.openlocfilehash: e6b4ba902e9951cd04dc282cc2a163200a38607a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142902"
---
# <a name="quickstart-create-an-apache-spark-cluster-in-hdinsight-using-powershell"></a>快速入门：使用 PowerShell 在 HDInsight 中创建 Apache Spark 群集
了解如何在 Azure HDInsight 中创建 [Apache Spark](https://spark.apache.org/) 群集，以及如何对 [Apache Hive](https://hive.apache.org/) 表运行 Spark SQL 查询。 通过 Apache Spark 可以使用内存处理进行快速数据分析和群集计算。 有关 Spark on HDInsight 的信息，请参阅[概述：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)。

在此快速入门中，使用 Azure PowerShell 创建 HDInsight Spark 群集。 群集将 Azure 存储 Blob 用作群集存储。 有关使用 Data Lake Storage Gen2 的详细信息，请参阅[快速入门：在 HDInsight 中设置群集](../../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

> [!IMPORTANT]  
> HDInsight 群集是基于分钟按比例收费，而不管用户是否正在使用它们。 请务必在使用完之后删除群集。 有关详细信息，请参阅本文的[清理资源](#clean-up-resources)部分。

## <a name="prerequisites"></a>先决条件

* 如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

* 已安装 PowerShell [Az 模块](https://docs.microsoft.com/powershell/azure/overview)。

## <a name="create-an-hdinsight-spark-cluster"></a>创建 HDInsight Spark 群集

创建 HDInsight 群集包括创建以下 Azure 对象和资源：

- Azure 资源组。 Azure 资源组是 Azure 资源的容器。 
- Azure 存储帐户或 Azure Data Lake Storage。  每个 HDInsight 群集都需要依赖的数据存储。 在本快速入门中，创建存储帐户。
- 不同群集类型的 HDInsight 群集。  在本快速入门中，你将创建 Spark 2.3 群集。

使用 PowerShell 脚本创建资源。  运行脚本时，系统会提示输入以下值：

|参数|值|
|------|------|
|Azure 资源组名称 | 提供资源组的唯一名称。|
|位置| 指定 Azure 区域，例如“美国中部”。 |
|默认存储帐户名 | 为存储帐户提供唯一名称。 |
|群集名称 | 提供 HDInsight Spark 群集的唯一名称。|
|群集登录凭据 | 在本快速入门中稍后使用该帐户连接到群集仪表板。|
|SSH 用户凭据 | SSH 客户端可用于创建与 HDInsight 群集的远程命令行会话。|

1. 选择以下代码块右上角的“试一试”，打开 [Azure Cloud Shell](../../cloud-shell/overview.md)，然后按照说明连接到 Azure。

2. 在 Cloud Shell 中复制并粘贴以下 PowerShell 脚本。

    ```azurepowershell-interactive
    ### Create a Spark 2.3 cluster in Azure HDInsight

    # Default cluster size (# of worker nodes), version, and type
    $clusterSizeInNodes = "1"
    $clusterVersion = "3.6"
    $clusterType = "Spark"
    
    # Create the resource group
    $resourceGroupName = Read-Host -Prompt "Enter the resource group name"
    $location = Read-Host -Prompt "Enter the Azure region to create resources in, such as 'Central US'"
    $defaultStorageAccountName = Read-Host -Prompt "Enter the default storage account name"
    
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    
    # Create an Azure storage account and container
    # Note: Storage account kind BlobStorage can only be used as secondary storage for HDInsight clusters.
    New-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly 1

    $defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    
    $defaultStorageContext = New-AzStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $defaultStorageAccountKey
    
    # Create a Spark 2.3 cluster
    $clusterName = Read-Host -Prompt "Enter the name of the HDInsight cluster"

    # Cluster login is used to secure HTTPS services hosted on the cluster
    $httpCredential = Get-Credential -Message "Enter Cluster login credentials" -UserName "admin"
    
    # SSH user is used to remotely connect to the cluster using SSH clients
    $sshCredentials = Get-Credential -Message "Enter SSH user credentials" -UserName "sshuser"
    
    # Set the storage container name to the cluster name
    $defaultBlobContainerName = $clusterName
    
    # Create a blob container. This holds the default data store for the cluster.
    New-AzStorageContainer `
        -Name $clusterName `
        -Context $defaultStorageContext 
    
    $sparkConfig = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
    $sparkConfig.Add("spark", "2.3")

    # Create the HDInsight cluster
    New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $clusterSizeInNodes `
        -ClusterType $clusterType `
        -OSType "Linux" `
        -Version $clusterVersion `
        -ComponentVersion $sparkConfig `
        -HttpCredential $httpCredential `
        -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultStorageContainer $clusterName `
        -SshCredential $sshCredentials 
    
    Get-AzHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName
    ```
   创建群集大约需要 20 分钟时间。 必须先创建群集，才能继续下一会话。

如果在创建 HDInsight 群集时遇到问题，可能是因为没有这样做的适当权限。 有关详细信息，请参阅[访问控制要求](../hdinsight-hadoop-customize-cluster-linux.md#access-control)。

## <a name="create-a-jupyter-notebook"></a>创建 Jupyter 笔记本

[Jupyter Notebook](https://jupyter.org/) 是支持各种编程语言的交互式笔记本环境。 通过此笔记本可以与数据进行交互、结合代码和 markdown 文本以及执行简单的可视化效果。

1. 打开 [Azure 门户](https://portal.azure.com)。
2. 选择“HDInsight 群集”，然后选择所创建的群集。

    ![在 Azure 门户中打开 HDInsight 群集](./media/apache-spark-jupyter-spark-sql/azure-portal-open-hdinsight-cluster.png)

3. 在门户中，选择“群集仪表板”，然后选择“Jupyter Notebook”。 出现提示时，请输入群集的群集登录凭据。

   ![打开 Jupyter Notebook 来运行交互式 Spark SQL 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-open-jupyter-interactive-spark-sql-query.png "打开 Jupyter Notebook 来运行交互式 Spark SQL 查询")

4. 选择“新建” > “PySpark”，创建笔记本。 

   ![创建 Jupyter Notebook 来运行交互式 Spark SQL 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-create-jupyter-interactive-spark-sql-query.png "创建 Jupyter Notebook 来运行交互式 Spark SQL 查询")

   新笔记本随即已创建，并以 Untitled(Untitled.pynb) 名称打开。


## <a name="run-spark-sql-statements"></a>运行 Spark SQL 语句

SQL（结构化查询语言）是用于查询和定义数据的最常见、最广泛使用的语言。 Spark SQL 作为 Apache Spark 的扩展使用，可使用熟悉的 SQL 语法处理结构化数据。

1. 验证 kernel 已就绪。 如果在 Notebook 中的内核名称旁边看到空心圆，则内核已准备就绪。 实心圆表示内核正忙。

    ![HDInsight Spark 中的 Hive 查询](./media/apache-spark-jupyter-spark-sql/jupyter-spark-kernel-status.png "HDInsight Spark 中的 Hive 查询")

    首次启动 Notebook 时，内核在后台执行一些任务。 等待内核准备就绪。 
2. 将以下代码粘贴到一个空单元格中，然后按 **SHIFT + ENTER** 来运行这些代码。 此命令列出群集上的 Hive 表：

    ```PySpark
    %%sql
    SHOW TABLES
    ```
    将 Jupyter Notebook 与 HDInsight Spark 群集配合使用时，会获得一个预设的 `sqlContext`，可以使用它通过 Spark SQL 来运行 Hive 查询。 `%%sql` 指示 Jupyter Notebook 使用预设 `sqlContext` 运行 Hive 查询。 该查询从默认情况下所有 HDInsight 群集都带有的 Hive 表 (hivesampletable) 检索前 10 行。 需要大约 30 秒才能获得结果。 输出如下所示： 

    ![HDInsight Spark 中的 Hive 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-get-started-hive-query.png "HDInsight Spark 中的 Hive 查询")

    每次在 Jupyter 中运行查询时，Web 浏览器窗口标题中都会显示“(繁忙)”状态和 Notebook 标题。 右上角“PySpark”文本的旁边还会出现一个实心圆。
    
2. 运行另一个查询，请查看 `hivesampletable` 中的数据。

    ```PySpark
    %%sql
    SELECT * FROM hivesampletable LIMIT 10
    ```
    
    屏幕在刷新后会显示查询输出。

    ![HDInsight Spark 中的 Hive 查询输出](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-get-started-hive-query-output.png "HDInsight Spark 中的 Hive 查询输出")

2. 请在 Notebook 的“文件”菜单中选择“关闭并停止”。 关闭 Notebook 会释放群集资源。

## <a name="clean-up-resources"></a>清理资源
HDInsight 将数据保存在 Azure 存储或 Azure Data Lake Storage 中，因此可以在未使用群集时安全地删除群集。 此外，还需要为 HDInsight 群集付费，即使不用也是如此。 由于群集费用数倍于存储空间费用，因此在群集不用时删除群集可以节省费用。 如果要立即开始[后续步骤](#next-steps)中所列的教程，可能需要保留群集。

切换回 Azure 门户，并选择“删除”。

![删除 HDInsight 群集](./media/apache-spark-jupyter-spark-sql/hdinsight-azure-portal-delete-cluster.png "删除 HDInsight 群集")

还可以选择资源组名称，打开“资源组”页，然后选择“删除资源组”。 通过删除资源组，可以删除 HDInsight Spark 群集和默认存储帐户。

### <a name="piecemeal-clean-up-with-powershell-az-module"></a>使用 PowerShell Az 模块进行段落清理

```powershell
# Removes the specified HDInsight cluster from the current subscription.
Remove-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName

# Removes the specified storage container.
Remove-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

# Removes a Storage account from Azure.
Remove-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName

# Removes a resource group.
Remove-AzResourceGroup `
    -Name $resourceGroupName
```

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何创建 HDInsight Spark 群集并运行基本的 Spark SQL 查询。 转到下一教程，了解如何使用 HDInsight Spark 群集针对示例数据运行交互式查询。

> [!div class="nextstepaction"]
>[在 Apache Spark 上运行交互式查询](./apache-spark-load-data-run-query.md)
