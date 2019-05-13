---
title: 教程：使用 Spark 通过 Azure Databricks 访问 Azure Data Lake Storage Gen2 数据 | Microsoft Docs
description: 本教程介绍如何在 Azure Databricks 群集上运行 Spark 查询，以便访问 Azure Data Lake Storage Gen2 存储帐户中的数据。
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 03/11/2019
ms.author: normesta
ms.reviewer: dineshm
ms.openlocfilehash: 02cff1be85f4489a9529383d90694581f2599cba
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64939181"
---
# <a name="tutorial-access-data-lake-storage-gen2-data-with-azure-databricks-using-spark"></a>教程：使用 Spark 通过 Azure Databricks 访问 Data Lake Storage Gen2 数据

本教程介绍如何将 Azure Databricks 群集连接到启用了 Azure Data Lake Storage Gen2 的 Azure 存储帐户中存储的数据。 建立此连接后，即可在群集本机上针对数据运行查询和分析。

在本教程中，将：

> [!div class="checklist"]
> * 创建 Databricks 群集
> * 将非结构化数据引入存储帐户中
> * 对 Blob 存储中的数据运行分析

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

* 创建 Azure Data Lake Storage Gen2 帐户。

  请参阅[创建 Azure Data Lake Storage Gen2 帐户](data-lake-storage-quickstart-create-account.md)。

* 请确保你的用户帐户分配有[存储 Blob 数据参与者角色](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac)。

* 安装 AzCopy v10。 请参阅[使用 AzCopy v10 传输数据](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

* 创建服务主体。 请参阅[如何：使用门户创建可访问资源的 Azure AD 应用程序和服务主体](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)。

  在执行该文中的步骤时，需要完成一些特定的事项。

  :heavy_check_mark:执行该文中[将应用程序分配给角色](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#assign-the-application-to-a-role)部分中的步骤时，请确保将“存储 Blob 数据参与者”角色分配给服务主体。

  > [!IMPORTANT]
  > 请确保在 Data Lake Storage Gen2 存储帐户的范围内分配角色。 可以将角色分配给父资源组或订阅，但在这些角色分配传播到存储帐户之前，你将收到与权限相关的错误。

  :heavy_check_mark:在执行文章的[获取用于登录的值](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)部分的步骤时，请将租户 ID、应用程序 ID 和身份验证密钥值粘贴到文本文件中。 很快就会需要这些值。

### <a name="download-the-flight-data"></a>下载航班数据

本教程使用美国运输统计局的航班数据，演示如何执行 ETL 操作。 必须下载该数据才能完成本教程。

1. 转到[美国运输统计局的研究与技术创新管理部门](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time)。

2. 选中“预压缩的文件”复选框以选择所有数据字段。

3. 选择“下载”按钮并将结果保存到计算机。 

4. 将压缩文件的内容解压缩，并记下文件名和文件路径。 在稍后的步骤中需要使用此信息。

## <a name="create-an-azure-databricks-service"></a>创建 Azure Databricks 服务

在本部分中，你将使用 Azure 门户创建 Azure Databricks 服务。

1. 在 Azure 门户中，选择“创建资源” > “分析” > “Azure Databricks”。

    ![Azure 门户上的 Databricks](./media/data-lake-storage-use-databricks-spark/azure-databricks-on-portal.png "Azure 门户上的 Databricks")

2. 在“Azure Databricks 服务”下，提供以下值来创建 Databricks 服务：

    |属性  |说明  |
    |---------|---------|
    |**工作区名称**     | 为 Databricks 工作区提供一个名称。  |
    |**订阅**     | 从下拉列表中选择自己的 Azure 订阅。        |
    |**资源组**     | 指定是要创建新的资源组还是使用现有的资源组。 资源组是用于保存 Azure 解决方案相关资源的容器。 有关详细信息，请参阅 [Azure 资源组概述](../../azure-resource-manager/resource-group-overview.md)。 |
    |**位置**     | 选择“美国西部 2”。 有关其他可用区域，请参阅[各区域推出的 Azure 服务](https://azure.microsoft.com/regions/services/)。       |
    |**定价层**     |  选择“标准”。     |

    ![创建 Azure Databricks 工作区](./media/data-lake-storage-use-databricks-spark/create-databricks-workspace.png "创建 Azure Databricks 服务")

3. 选择“固定到仪表板”，然后选择“创建”。

4. 创建帐户需要几分钟时间。 在创建帐户过程中，门户会在右侧显示“正在提交 Azure Databricks 的部署”磁贴。 若要监视操作状态，请查看顶部的进度栏。

    ![Databricks 部署磁贴](./media/data-lake-storage-use-databricks-spark/databricks-deployment-tile.png "Databricks 部署磁贴")

## <a name="create-a-spark-cluster-in-azure-databricks"></a>在 Azure Databricks 中创建 Spark 群集

1. 在 Azure 门户中，转到所创建的 Databricks 服务，然后选择“启动工作区”。

2. 系统随后会将你重定向到 Azure Databricks 门户。 在门户中选择“群集”。

    ![Azure 上的 Databricks](./media/data-lake-storage-use-databricks-spark/databricks-on-azure.png "Azure 上的 Databricks")

3. 在“新建群集”页中，提供用于创建群集的值。

    ![在 Azure 上创建 Databricks Spark 群集](./media/data-lake-storage-use-databricks-spark/create-databricks-spark-cluster.png "在 Azure 上创建 Databricks Spark 群集")

4. 填写以下字段的值，对于其他字段接受默认值：

    * 输入群集的名称。

    * 在本文中，请创建运行时为 **5.1** 的群集。

    * 请务必选中“在不活动超过 \_\_ 分钟后终止”复选框。 如果未使用群集，则请提供一个持续时间（以分钟为单位），超过该时间后群集会被终止。

    * 选择“创建群集”。 群集运行后，可将笔记本附加到该群集，并运行 Spark 作业。

## <a name="create-a-file-system-and-mount-it"></a>创建并装载文件系统

在本部分，你将在存储帐户中创建一个文件系统和文件夹。

1. 在 [Azure 门户](https://portal.azure.com)中，转到所创建的 Azure Databricks 服务，然后选择“启动工作区”。

2. 在左侧选择“工作区”。 在**工作区**下拉列表中，选择**创建** > **笔记本**。

    ![在 Databricks 中创建笔记本](./media/data-lake-storage-use-databricks-spark/databricks-create-notebook.png "在 Databricks 中创建笔记本")

3. 在“创建 Notebook”对话框中，输入 Notebook 的名称。 选择“Python”作为语言，然后选择前面创建的 Spark 群集。

4. 选择“创建”。

5. 将以下代码块复制并粘贴到第一个单元格中，但目前请勿运行此代码。

    ```Python
    configs = {"fs.azure.account.auth.type": "OAuth",
           "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
           "fs.azure.account.oauth2.client.id": "<application-id>",
           "fs.azure.account.oauth2.client.secret": "<authentication-id>",
           "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant-id>/oauth2/token",
           "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

    dbutils.fs.mount(
    source = "abfss://<file-system-name>@<storage-account-name>.dfs.core.windows.net/folder1",
    mount_point = "/mnt/flightdata",
    extra_configs = configs)
    ```

18. 在此代码块中，请将 `application-id`、`authentication-id`、`tenant-id` 和 `storage-account-name` 占位符值替换为在完成本教程的先决条件时收集的值。 将 `file-system-name` 占位符值替换为你想要为文件系统指定的任何名称。

   * `application-id` 和 `authentication-id` 来自在创建服务主体的过程中向 active directory 注册的应用。

   * `tenant-id` 来自你的订阅。

   * `storage-account-name` 是 Azure Data Lake Storage Gen2 存储帐户的名称。

   > [!NOTE]
   > 在生产设置中，请考虑将身份验证密钥存储在 Azure Databricks 中。 然后，将查找密钥（而不是身份验证密钥）添加到代码块。 完成了本快速入门之后，请参阅 Azure Databricks 网站上的 [Azure Data Lake Storage Gen2](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-datalake-gen2.html) 一文以查看此方法的示例。

19. 按 **SHIFT + ENTER** 键，运行此块中的代码。

   请将此笔记本保持打开状态，因为稍后要在其中添加命令。

## <a name="ingest-data"></a>引入数据

### <a name="copy-source-data-into-the-storage-account"></a>将源数据复制到存储帐户中

使用 AzCopy 将 *.csv* 文件中的数据复制到 Data Lake Storage Gen2 帐户。

1. 打开命令提示符窗口，输入以下命令登录到存储帐户。

   ```bash
   azcopy login
   ```

   遵照命令提示符窗口中的说明对用户帐户进行身份验证。

2. 若要复制 *.csv* 帐户中的数据，请输入以下命令。

   ```bash
   azcopy cp "<csv-folder-path>" https://<storage-account-name>.dfs.core.windows.net/<file-system-name>/folder1/On_Time.csv
   ```

   * 将 `<csv-folder-path>` 占位符值替换为 .csv 文件的路径。

   * 将 `storage-account-name` 占位符值替换为存储帐户的名称。

   * 将 `file-system-name` 占位符替换为你要为文件系统提供的任何名称。

### <a name="use-databricks-notebook-to-convert-csv-to-parquet"></a>使用 Databricks Notebook 将 CSV 转换为 Parquet

在前面创建的笔记本中添加一个新的单元，并将以下代码粘贴到该单元中。 

```python
# Use the previously established DBFS mount point to read the data.
# create a data frame to read data.

flightDF = spark.read.format('csv').options(header='true', inferschema='true').load("/mnt/flightdata/*.csv")

# read the airline csv file and write the output to parquet format for easy query.
flightDF.write.mode("append").parquet("/mnt/flightdata/parquet/flights")
print("Done")
```

## <a name="explore-data"></a>浏览数据

在新单元中粘贴以下代码，以获取通过 AzCopy 上传的 CSV 文件列表。

```python
import os.path
import IPython
from pyspark.sql import SQLContext
display(dbutils.fs.ls("/mnt/flightdata"))
```

若要创建新文件并列出 *parquet/flights* 文件夹中的文件，请运行以下脚本：

```python
dbutils.fs.put("/mnt/flightdata/1.txt", "Hello, World!", True)
dbutils.fs.ls("/mnt/flightdata/parquet/flights")
```

通过这些代码示例，你已使用启用了 Data Lake Storage Gen2 的存储帐户中存储的数据探讨了 HDFS 的层次结构性质。

## <a name="query-the-data"></a>查询数据

接下来，可以开始查询上传到存储帐户中的数据。 将下面的每个代码块输入到 **Cmd 1** 中，然后按 **Cmd + Enter** 运行 Python 脚本。

若要为数据源创建数据帧，请运行以下脚本：

* 将 `<csv-folder-path>` 占位符值替换为 .csv 文件的路径。

```python
#Copy this into a Cmd cell in your notebook.
acDF = spark.read.format('csv').options(header='true', inferschema='true').load("/mnt/flightdata/On_Time.csv")
acDF.write.parquet('/mnt/flightdata/parquet/airlinecodes')

#read the existing parquet file for the flights database that was created earlier
flightDF = spark.read.format('parquet').options(header='true', inferschema='true').load("/mnt/flightdata/parquet/flights")

#print the schema of the dataframes
acDF.printSchema()
flightDF.printSchema()

#print the flight database size
print("Number of flights in the database: ", flightDF.count())

#show the first 20 rows (20 is the default)
#to show the first n rows, run: df.show(n)
acDF.show(100, False)
flightDF.show(20, False)

#Display to run visualizations
#preferably run this in a separate cmd cell
display(flightDF)
```

输入此脚本，以针对数据运行一些基本分析查询。

```python
#Run each of these queries, preferably in a separate cmd cell for separate analysis
#create a temporary sql view for querying flight information
FlightTable = spark.read.parquet('/mnt/flightdata/parquet/flights')
FlightTable.createOrReplaceTempView('FlightTable')

#create a temporary sql view for querying airline code information
AirlineCodes = spark.read.parquet('/mnt/flightdata/parquet/airlinecodes')
AirlineCodes.createOrReplaceTempView('AirlineCodes')

#using spark sql, query the parquet file to return total flights in January and February 2016
out1 = spark.sql("SELECT * FROM FlightTable WHERE Month=1 and Year= 2016")
NumJan2016Flights = out1.count()
out2 = spark.sql("SELECT * FROM FlightTable WHERE Month=2 and Year= 2016")
NumFeb2016Flights=out2.count()
print("Jan 2016: ", NumJan2016Flights," Feb 2016: ",NumFeb2016Flights)
Total= NumJan2016Flights+NumFeb2016Flights
print("Total flights combined: ", Total)

# List out all the airports in Texas
out = spark.sql("SELECT distinct(OriginCityName) FROM FlightTable where OriginStateName = 'Texas'") 
print('Airports in Texas: ', out.show(100))

#find all airlines that fly from Texas
out1 = spark.sql("SELECT distinct(Reporting_Airline) FROM FlightTable WHERE OriginStateName='Texas'")
print('Airlines that fly to/from Texas: ', out1.show(100, False))
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要本文中创建的资源，可以删除资源组和所有相关资源。 为此，请选择存储帐户所在的资源组，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"] 
> [使用 Apache Hive on Azure HDInsight 提取、转换和加载数据](data-lake-storage-tutorial-extract-transform-load-hive.md)
