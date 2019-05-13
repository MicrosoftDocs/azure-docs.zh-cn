---
title: 教程：使用 Azure Databricks 执行 ETL 操作
description: 了解如何将数据从 Data Lake Storage Gen2 提取到 Azure Databricks 中，对数据进行转换，然后将数据加载到 Azure SQL 数据仓库中。
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: azure-databricks
ms.custom: mvc
ms.topic: tutorial
ms.date: 05/07/2019
ms.openlocfilehash: e2110378d16ff5826b8ded4620276b784ef1d68e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203356"
---
# <a name="tutorial-extract-transform-and-load-data-by-using-azure-databricks"></a>教程：使用 Azure Databricks 提取、转换和加载数据

本教程使用 Azure Databricks 执行 ETL（提取、转换和加载数据）操作。 将数据从 Azure Data Lake Storage Gen2 提取到 Azure Databricks 中，在 Azure Databricks 中对数据运行转换操作，并将转换的数据加载到 Azure SQL 数据仓库中。

本教程中的步骤使用 Azure Databricks 的 SQL 数据仓库连接器将数据传输到 Azure Databricks。 而此连接器又使用 Azure Blob 存储来临时存储在 Azure Databricks 群集和 Azure SQL 数据仓库之间传输的数据。

下图演示了应用程序流：

![Azure Databricks 与 Data Lake Store 和 SQL 数据仓库](./media/databricks-extract-load-sql-data-warehouse/databricks-extract-transform-load-sql-datawarehouse.png "Azure Databricks 与 Data Lake Store 和 SQL 数据仓库")

本教程涵盖以下任务：

> [!div class="checklist"]
> * 创建 Azure Databricks 服务。
> * 在 Azure Databricks 中创建 Spark 群集。
> * 在 Data Lake Storage Gen2 帐户中创建文件系统。
> * 上传示例数据到 Azure Data Lake Storage Gen2 帐户。
> * 创建服务主体。
> * 从 Azure Data Lake Storage Gen2 帐户中提取数据。
> * 在 Azure Databricks 中转换数据。
> * 将数据载入 Azure SQL 数据仓库。

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

> [!Note]
> 不能使用 **Azure 免费试用订阅**完成本教程。
> 若要使用免费帐户创建 Azure Databricks 群集，请在创建群集前转到你的配置文件并将订阅更改为**即用即付**。 有关详细信息，请参阅 [Azure 免费帐户](https://azure.microsoft.com/free/)。
     
## <a name="prerequisites"></a>先决条件

在开始本教程之前，完成以下任务：

* 以服务器管理员身份创建 Azure SQL 数据仓库、创建服务器级防火墙规则并连接到服务器。请参阅[快速入门：在 Azure 门户中创建和查询 Azure SQL 数据仓库](../sql-data-warehouse/create-data-warehouse-portal.md)。

* 为 Azure SQL 数据仓库创建数据库主密钥。 请参阅[创建数据库主密钥](https://docs.microsoft.com/sql/relational-databases/security/encryption/create-a-database-master-key)。

* 创建 Azure Blob 存储帐户并在其中创建容器。 另外，请检索用于访问该存储帐户的访问密钥。 请参阅[快速入门：使用 Azure 门户上传、下载和列出 Blob](../storage/blobs/storage-quickstart-blobs-portal.md)。

* 创建 Azure Data Lake Storage Gen2 存储帐户。 请参阅[快速入门：创建 Azure Data Lake Storage Gen2 存储帐户](../storage/blobs/data-lake-storage-quickstart-create-account.md)。

*  创建服务主体。 请参阅[如何：使用门户创建可访问资源的 Azure AD 应用程序和服务主体](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)。

   在执行该文中的步骤时，需要完成一些特定的事项。

   * 执行该文中[将应用程序分配给角色](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#assign-the-application-to-a-role)部分中的步骤时，请确保将“存储 Blob 数据参与者”角色分配给服务主体。

     > [!IMPORTANT]
     > 请确保在 Data Lake Storage Gen2 存储帐户的范围内分配角色。 可以将角色分配给父资源组或订阅，但在这些角色分配传播到存储帐户之前，你将收到与权限相关的错误。

   * 在执行文章的[获取用于登录的值](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)部分的步骤时，请将租户 ID、应用程序 ID 和身份验证密钥值粘贴到文本文件中。 很快就会需要这些值。

* 登录到 [Azure 门户](https://portal.azure.com/)。

## <a name="gather-the-information-that-you-need"></a>收集所需信息

确保完成本教程的先决条件。

   在开始之前，应具有以下这些信息项：

   :heavy_check_mark:Azure SQL 数据仓库的数据库名称、数据库服务器名称、用户名和密码。

   :heavy_check_mark:blob 存储帐户的访问密钥。

   :heavy_check_mark:Data Lake Storage Gen2 存储帐户的名称。

   :heavy_check_mark:订阅的租户 ID。

   :heavy_check_mark:向 Azure Active Directory (Azure AD) 注册的应用的应用程序 ID。

   :heavy_check_mark:向 Azure AD 注册的应用的身份验证密钥。

## <a name="create-an-azure-databricks-service"></a>创建 Azure Databricks 服务

在本部分中，你将使用 Azure 门户创建 Azure Databricks 服务。

1. 在 Azure 门户中，选择“创建资源” > “分析” > “Azure Databricks”。

    ![Azure 门户上的 Databricks](./media/databricks-extract-load-sql-data-warehouse/azure-databricks-on-portal.png "Azure 门户上的 Databricks")

2. 在“Azure Databricks 服务”下，提供以下值来创建 Databricks 服务：

    |属性  |说明  |
    |---------|---------|
    |**工作区名称**     | 为 Databricks 工作区提供一个名称。        |
    |**订阅**     | 从下拉列表中选择自己的 Azure 订阅。        |
    |**资源组**     | 指定是要创建新的资源组还是使用现有的资源组。 资源组是用于保存 Azure 解决方案相关资源的容器。 有关详细信息，请参阅 [Azure 资源组概述](../azure-resource-manager/resource-group-overview.md)。 |
    |**位置**     | 选择“美国西部 2”。  有关其他可用区域，请参阅[各区域推出的 Azure 服务](https://azure.microsoft.com/regions/services/)。      |
    |**定价层**     |  选择“标准”。     |

3. 选择“固定到仪表板”，然后选择“创建”。

4. 创建帐户需要几分钟时间。 在创建帐户过程中，门户会在右侧显示“正在提交 Azure Databricks 的部署”磁贴。 若要监视操作状态，请查看顶部的进度栏。

    ![Databricks 部署磁贴](./media/databricks-extract-load-sql-data-warehouse/databricks-deployment-tile.png "Databricks 部署磁贴")

## <a name="create-a-spark-cluster-in-azure-databricks"></a>在 Azure Databricks 中创建 Spark 群集

1. 在 Azure 门户中，转到所创建的 Databricks 服务，然后选择“启动工作区”。

2. 系统随后会将你重定向到 Azure Databricks 门户。 在门户中选择“群集”。

    ![Azure 上的 Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-on-azure.png "Azure 上的 Databricks")

3. 在“新建群集”页中，提供用于创建群集的值。

    ![在 Azure 上创建 Databricks Spark 群集](./media/databricks-extract-load-sql-data-warehouse/create-databricks-spark-cluster.png "在 Azure 上创建 Databricks Spark 群集")

4. 填写以下字段的值，对于其他字段接受默认值：

    * 输入群集的名称。

    * 在本文中，请创建运行时为 **5.1** 的群集。

    * 请务必选中“在不活动超过 \_\_ 分钟后终止”复选框。 如果未使用群集，则请提供一个持续时间（以分钟为单位），超过该时间后群集会被终止。

    * 选择“创建群集”。 群集运行后，可将笔记本附加到该群集，并运行 Spark 作业。

## <a name="create-a-file-system-in-the-azure-data-lake-storage-gen2-account"></a>在 Azure Data Lake Storage Gen2 帐户中创建文件系统

在本部分中，你将在 Azure Databricks 工作区中创建一个 Notebook，然后运行代码片段来配置存储帐户

1. 在 [Azure 门户](https://portal.azure.com)中，转到你创建的 Azure Databricks 服务，然后选择“启动工作区”。

2. 在左侧选择“工作区”。 在**工作区**下拉列表中，选择**创建** > **笔记本**。

    ![在 Databricks 中创建笔记本](./media/databricks-extract-load-sql-data-warehouse/databricks-create-notebook.png "在 Databricks 中创建笔记本")

3. 在“创建 Notebook”对话框中，输入 Notebook 的名称。 选择“Scala”作为语言，然后选择前面创建的 Spark 群集。

    ![在 Databricks 中提供笔记本的详细信息](./media/databricks-extract-load-sql-data-warehouse/databricks-notebook-details.png "在 Databricks 中提供笔记本的详细信息")

4. 选择“创建”。

5. 将以下代码块复制并粘贴到第一个单元中。

   ```scala
   spark.conf.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.windows.net", "OAuth")
   spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
   spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.windows.net", "<application-id>")
   spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.windows.net", "<authentication-key>")
   spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.windows.net", "https://login.microsoftonline.com/<tenant-id>/oauth2/token")
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "true")
   dbutils.fs.ls("abfss://<file-system-name>@<storage-account-name>.dfs.core.windows.net/")
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "false")
   ```

6. 在此代码块中，请将 `application-id`、`authentication-id`、`tenant-id` 和 `storage-account-name` 占位符值替换为在完成本教程的先决条件时收集的值。 将 `file-system-name` 占位符值替换为你想要为文件系统指定的任何名称。

   * `application-id` 和 `authentication-id` 来自在创建服务主体的过程中向 active directory 注册的应用。

   * `tenant-id` 来自你的订阅。

   * `storage-account-name` 是 Azure Data Lake Storage Gen2 存储帐户的名称。

7. 按 **SHIFT + ENTER** 键，运行此块中的代码。

## <a name="ingest-sample-data-into-the-azure-data-lake-storage-gen2-account"></a>将示例数据引入 Azure Data Lake Storage Gen2 帐户

开始学习本部分之前，必须完成以下先决条件：

将以下代码输入到 Notebook 单元格中：

    %sh wget -P /tmp https://raw.githubusercontent.com/Azure/usql/master/Examples/Samples/Data/json/radiowebsite/small_radio_json.json

在单元格中，按 SHIFT + ENTER 来运行代码。

现在，请在此单元格下方的新单元格中输入以下代码，将括号中出现的值替换为此前使用的相同值：

    dbutils.fs.cp("file:///tmp/small_radio_json.json", "abfss://<file-system>@<account-name>.dfs.core.windows.net/")

在单元格中，按 SHIFT + ENTER 来运行代码。

## <a name="extract-data-from-the-azure-data-lake-storage-gen2-account"></a>从 Azure Data Lake Storage Gen2 帐户中提取数据

1. 现在可以将示例 json 文件加载为 Azure Databricks 中的数据帧。 将以下代码粘贴到新单元格中。 将括号中显示的占位符替换为你的值。

   ```scala
   val df = spark.read.json("abfss://<file-system-name>@<storage-account-name>.dfs.core.windows.net/small_radio_json.json")
   ```

   * 将 `file-system-name` 占位符值替换为在存储资源管理器中提供给文件系统的名称。

   * 将 `storage-account-name` 占位符替换为存储帐户的名称。

2. 按 **SHIFT + ENTER** 键，运行此块中的代码。

3. 运行以下代码来查看数据帧的内容：

    ```scala
    df.show()
    ```
   会显示类似于以下代码片段的输出：

   ```bash
   +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
   |               artist|     auth|firstName|gender|itemInSession|  lastName|   length|  level|            location|method|    page| registration|sessionId|                song|status|           ts|userId|
   +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
   | El Arrebato         |Logged In| Annalyse|     F|            2|Montgomery|234.57914| free  |  Killeen-Temple, TX|   PUT|NextSong|1384448062332|     1879|Quiero Quererte Q...|   200|1409318650332|   309|
   | Creedence Clearwa...|Logged In|   Dylann|     M|            9|    Thomas|340.87138| paid  |       Anchorage, AK|   PUT|NextSong|1400723739332|       10|        Born To Move|   200|1409318653332|    11|
   | Gorillaz            |Logged In|     Liam|     M|           11|     Watts|246.17751| paid  |New York-Newark-J...|   PUT|NextSong|1406279422332|     2047|                DARE|   200|1409318685332|   201|
   ...
   ...
   ```

   现在，你已将数据从 Azure Data Lake Storage Gen2 提取到 Azure Databricks 中。

## <a name="transform-data-in-azure-databricks"></a>在 Azure Databricks 中转换数据

原始示例数据 **small_radio_json.json** 文件捕获某个电台的听众，有多个不同的列。 在此部分，请对该数据进行转换，仅检索数据集中的特定列。

1. 首先，仅从已创建的 dataframe 检索 **firstName**、**lastName**、**gender**、**location** 和 **level** 列。

   ```scala
   val specificColumnsDf = df.select("firstname", "lastname", "gender", "location", "level")
   specificColumnsDf.show()
   ```

   接收的输出如以下代码片段所示：

   ```bash
   +---------+----------+------+--------------------+-----+
   |firstname|  lastname|gender|            location|level|
   +---------+----------+------+--------------------+-----+
   | Annalyse|Montgomery|     F|  Killeen-Temple, TX| free|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |     Tess|  Townsend|     F|Nashville-Davidso...| free|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |Gabriella|   Shelton|     F|San Jose-Sunnyval...| free|
   |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Tess|  Townsend|     F|Nashville-Davidso...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   +---------+----------+------+--------------------+-----+
   ```

2. 可以进一步转换该数据，将 **level** 列重命名为 **subscription_type**。

   ```scala
   val renamedColumnsDF = specificColumnsDf.withColumnRenamed("level", "subscription_type")
   renamedColumnsDF.show()
   ```

   接收的输出如以下代码片段所示。

   ```bash
   +---------+----------+------+--------------------+-----------------+
   |firstname|  lastname|gender|            location|subscription_type|
   +---------+----------+------+--------------------+-----------------+
   | Annalyse|Montgomery|     F|  Killeen-Temple, TX|             free|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |Gabriella|   Shelton|     F|San Jose-Sunnyval...|             free|
   |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   +---------+----------+------+--------------------+-----------------+
   ```

## <a name="load-data-into-azure-sql-data-warehouse"></a>将数据加载到 Azure SQL 数据仓库

在此部分，请将转换的数据上传到 Azure SQL 数据仓库中。 使用适用于 Azure Databricks 的 Azure SQL 数据仓库连接器直接上传 dataframe，在 SQL 数据仓库中作为表来存储。

如前所述，SQL 数据仓库连接器使用 Azure Blob 存储作为临时存储，以便将数据从 Azure Databricks 上传到 Azure SQL 数据仓库。 因此，一开始请提供连接到存储帐户所需的配置。 必须已经按照本文先决条件部分的要求创建了帐户。

1. 提供从 Azure Databricks 访问 Azure 存储帐户所需的配置。

   ```scala
   val blobStorage = "<blob-storage-account-name>.blob.core.windows.net"
   val blobContainer = "<blob-container-name>"
   val blobAccessKey =  "<access-key>"
   ```

2. 指定一个在 Azure Databricks 和 Azure SQL 数据仓库之间移动数据时需要使用的临时文件夹。

   ```scala
   val tempDir = "wasbs://" + blobContainer + "@" + blobStorage +"/tempDirs"
   ```

3. 运行以下代码片段，以便在配置中存储 Azure Blob 存储访问密钥。 此操作可确保不需将访问密钥以纯文本形式存储在笔记本中。

   ```scala
   val acntInfo = "fs.azure.account.key."+ blobStorage
   sc.hadoopConfiguration.set(acntInfo, blobAccessKey)
   ```

4. 提供连接到 Azure SQL 数据仓库实例所需的值。 必须已经将 SQL 数据仓库作为先决条件创建。

   ```scala
   //SQL Data Warehouse related settings
   val dwDatabase = "<database-name>"
   val dwServer = "<database-server-name>"
   val dwUser = "<user-name>"
   val dwPass = "<password>"
   val dwJdbcPort =  "1433"
   val dwJdbcExtraOptions = "encrypt=true;trustServerCertificate=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
   val sqlDwUrl = "jdbc:sqlserver://" + dwServer + ":" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass + ";$dwJdbcExtraOptions"
   val sqlDwUrlSmall = "jdbc:sqlserver://" + dwServer + ":" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass
   ```

5. 运行以下代码片段来加载转换的 dataframe **renamedColumnsDF**，在 SQL 数据仓库中作为表来存储。 此代码片段在 SQL 数据库中创建名为 **SampleTable** 的表。

   ```scala
   spark.conf.set(
       "spark.sql.parquet.writeLegacyFormat",
       "true")

   renamedColumnsDF.write
       .format("com.databricks.spark.sqldw")
       .option("url", sqlDwUrlSmall) 
       .option("dbtable", "SampleTable")
       .option( "forward_spark_azure_storage_credentials","True")
       .option("tempdir", tempDir)
       .mode("overwrite")
       .save()
   ```

   > [!NOTE]
   > 此示例使用 `forward_spark_azure_storage_credentials` 标志，该标志导致 SQL 数据仓库使用访问密钥访问 blob 存储中的数据。 这是唯一支持的身份验证方法。
   >
   > 如果将 Azure Blob 存储限制为选择虚拟网络，则 SQL 数据仓库需要[托管服务标识，而不是访问密钥](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage)。 这将导致错误“此请求无权执行此操作”。

6. 连接到 SQL 数据库，验证是否看到名为 **SampleTable** 的数据库。

   ![验证示例表](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table.png "验证示例表")

7. 运行一个 select 查询，验证表的内容。 该表的数据应该与 **renamedColumnsDF** dataframe 相同。

    ![验证示例表内容](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table-content.png "验证示例表内容")

## <a name="clean-up-resources"></a>清理资源

完成本教程后，可以终止群集。 在 Azure Databricks 工作区的左侧选择“群集”。 对于要终止的群集，请将鼠标指向“操作”下面的省略号 (...)，然后选择“终止”图标。

![停止 Databricks 群集](./media/databricks-extract-load-sql-data-warehouse/terminate-databricks-cluster.png "停止 Databricks 群集")

如果不手动终止群集，但在创建群集时选中了“在不活动 \_\_ 分钟后终止”复选框，则该群集会自动停止。 在这种情况下，如果群集保持非活动状态超过指定的时间，则会自动停止。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 创建 Azure Databricks 服务
> * 在 Azure Databricks 中创建 Spark 群集
> * 在 Azure Databricks 中创建 Notebook
> * 从 Data Lake Storage Gen2 帐户提取数据
> * 在 Azure Databricks 中转换数据
> * 将数据加载到 Azure SQL 数据仓库

请继续学习下一教程，了解如何使用 Azure 事件中心将实时数据流式传输到 Azure Databricks 中。

> [!div class="nextstepaction"]
>[使用事件中心将数据流式传输到 Azure Databricks](databricks-stream-from-eventhubs.md)
