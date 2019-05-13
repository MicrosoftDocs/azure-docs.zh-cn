---
title: 使用适用于 Apache Spark 的 Azure 数据资源管理器连接器 Azure 数据资源管理器和 Spark 群集之间移动数据。
description: 本主题演示如何将 Azure 数据资源管理器和 Apache Spark 群集之间移动数据。
author: orspod
ms.author: orspodek
ms.reviewer: michazag
ms.service: data-explorer
ms.topic: conceptual
ms.date: 4/29/2019
ms.openlocfilehash: 854e29b67b6e24c583a98b5851bf17551cfcbf61
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441350"
---
# <a name="azure-data-explorer-connector-for-apache-spark-preview"></a>Azure 数据资源管理器适用于 Apache Spark （预览版） 的连接器

[Apache Spark](https://spark.apache.org/)是一个统一的分析引擎，用于大规模数据处理。 Azure 数据资源管理器是对数据量很大的实时分析的快速、 完全托管的数据分析服务。 

Spark 的 azure 资源管理器中的数据连接器实现数据源和跨 Azure 数据资源管理器和 Spark 群集，使用这两个其功能将数据移动的数据接收器。 使用 Azure 数据资源管理器和 Apache Spark，可以构建面向数据驱动型方案（如机器学习 (ML)、提取-转换-加载 (ETL) 和 Log Analytics）的可缩放快速应用程序。 写入到 Azure 数据资源管理器可以在批处理和流式处理模式下完成。
从 Azure 数据资源管理器读取支持删除列和谓词下推，从而降低通过过滤掉 Azure 数据资源管理器中的数据传输的数据量。

Azure 数据资源管理器 Spark 连接器[开放源代码项目](https://github.com/Azure/azure-kusto-spark)，可以在任何 Spark 群集上运行。

> [!NOTE]
> 虽然下面的示例的一些指[Azure Databricks](https://docs.azuredatabricks.net/) Spark 群集，Azure 数据资源管理器 Spark 连接器不会直接在 Databricks 或任何其他 Spark 分发上的依赖项。

## <a name="prerequisites"></a>必备组件

* [创建 Azure 数据资源管理器群集和数据库](/azure/data-explorer/create-cluster-database-portal) 
* 创建 Spark 群集
* 安装 Azure 数据资源管理器连接器库和库中列出[依赖项](https://github.com/Azure/azure-kusto-spark#dependencies)包括以下[Kusto Java SDK](/azure/kusto/api/java/kusto-java-client-library)库：
    * [Kusto 数据客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data)
    * [Kusto 引入客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest)
* 预生成库[Spark 2.4、 Scala 2.11](https://github.com/Azure/azure-kusto-spark/releases)

## <a name="how-to-build-the-spark-connector"></a>如何生成 Spark 连接器

可从生成 Spark 连接器[源](https://github.com/Azure/azure-kusto-spark)详情如下。

> [!NOTE]
> 此步骤是可选的。 如果使用预建的库转到[Spark 群集设置](#spark-cluster-setup)。

### <a name="build-prerequisites"></a>生成系统必备组件

* 安装 Java 1.8 SDK
* [Maven 3.x](https://maven.apache.org/download.cgi)安装
* Apache Spark 版本 2.4.0 或更高版本

> [!TIP]
> 2.3.x 版本也受支持，但可能需要 pom.xml 依赖项中的某些更改。

针对 Scala/Java 应用程序使用 Maven 项目定义链接将应用程序与以下项目 （最新版本可能不同）：

```Maven
   <dependency>
     <groupId>com.microsoft.azure</groupId>
     <artifactId>spark-kusto-connector</artifactId>
     <version>1.0.0-Beta-02</version>
   </dependency>
```

### <a name="build-commands"></a>生成命令

生成 jar 并运行所有测试：

```
mvn clean package
```

若要生成 jar，请运行所有测试，并安装到本地 Maven 存储库的 jar:

```
mvn clean install
```

有关详细信息，请参阅[连接器使用](https://github.com/Azure/azure-kusto-spark#usage)。

## <a name="spark-cluster-setup"></a>Spark 群集设置

> [!NOTE]
> 建议执行以下步骤时使用的最新的 Azure 数据资源管理器 Spark 连接器版本：

1. 将以下 Spark 群集设置，使用 Spark 2.4 和 Scala 2.11 的 Azure Databricks 群集上基于： 

    ![Databricks 群集设置](media/spark-connector/databricks-cluster.png)

1. 导入 Azure 数据资源管理器连接器库：

    ![导入 Azure 数据资源管理器库](media/spark-connector/db-create-library.png)

1. 添加其他依赖项：

    ![添加依赖项](media/spark-connector/db-dependencies.png)

    > [!TIP]
    > 找到每个 Spark 版本的正确的 java 版本[此处](https://github.com/Azure/azure-kusto-spark#dependencies)。

1. 验证所有必需的库安装：

    ![验证安装库](media/spark-connector/db-libraries-view.png)

## <a name="authentication"></a>Authentication

Azure 数据资源管理器 Spark 连接器，可通过使用 Azure Active Directory (Azure AD) 进行身份验证[Azure AD 应用程序](#azure-ad-application-authentication)， [Azure AD 访问令牌](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#direct-authentication-with-access-token)，[设备身份验证](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#device-authentication) （适用于非生产方案中） 或[Azure 密钥保管库](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#key-vault)。 用户必须安装 azure 密钥保管库包，并提供应用程序凭据来访问密钥保管库资源。

### <a name="azure-ad-application-authentication"></a>Azure AD 应用程序身份验证

大多数简单且常见身份验证方法。 建议对 Azure 数据资源管理器 Spark 连接器使用此方法。

|属性  |描述  |
|---------|---------|
|**KUSTO_AAD_CLIENT_ID**     |   Azure AD 应用程序 （客户端） 标识符。      |
|**KUSTO_AAD_AUTHORITY_ID**     |  Azure AD 身份验证机构。 Azure AD 目录 （租户） id。        |
|**KUSTO_AAD_CLIENT_PASSWORD**    |    客户端的 azure AD 应用程序密钥。     |

### <a name="azure-data-explorer-privileges"></a>Azure 数据资源管理器权限

必须在 Azure 数据资源管理器群集上授予以下权限：

* 用于读取 （数据源），Azure AD 应用程序必须具有*查看器*上，对目标数据库的权限或*管理员*目标表的特权。
* 对于编写 （数据接收器），Azure AD 应用程序必须具有*采集器*上目标数据库的权限。 它必须还具有*用户*上目标数据库以创建新表的权限。 如果目标表已存在，*管理员*可以配置目标表的特权。
 
有关 Azure 数据资源管理器主体角色的详细信息，请参阅[基于角色的授权](/azure/kusto/management/access-control/role-based-authorization)。 管理安全角色，请参阅[安全角色管理](/azure/kusto/management/security-roles)。

## <a name="spark-sink-writing-to-azure-data-explorer"></a>Spark 接收器：写入到 Azure 的数据资源管理器

1. 设置接收器参数：

     ```scala
    val KustoSparkTestAppId = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppId")
    val KustoSparkTestAppKey = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppKey")
 
    val appId = KustoSparkTestAppId
    val appKey = KustoSparkTestAppKey
    val authorityId = "72f988bf-86f1-41af-91ab-2d7cd011db47"
    val cluster = "Sparktest.eastus2"
    val database = "TestDb"
    val table = "StringAndIntTable"
    ```

1. 以批处理形式写入到 Azure 数据资源管理器群集的 Spark 数据帧：

    ```scala
    df.write
      .format("com.microsoft.kusto.spark.datasource")
      .option(KustoOptions.KUSTO_CLUSTER, cluster)
      .option(KustoOptions.KUSTO_DATABASE, database)
      .option(KustoOptions.KUSTO_TABLE, table)
      .option(KustoOptions.KUSTO_AAD_CLIENT_ID, appId)
      .option(KustoOptions.KUSTO_AAD_CLIENT_PASSWORD, appKey) 
      .option(KustoOptions.KUSTO_AAD_AUTHORITY_ID, authorityId)
      .save()
    ```

1. 写入流式处理数据：

    ```scala    
    import org.apache.spark.sql.streaming.Trigger
    import java.util.concurrent.TimeUnit
    
    // Set up a checkpoint and disable codeGen. Set up a checkpoint and disable codeGen as a workaround for an known issue 
    spark.conf.set("spark.sql.streaming.checkpointLocation", "/FileStore/temp/checkpoint")
    spark.conf.set("spark.sql.codegen.wholeStage","false")
    
    // Write to a Kusto table fro streaming source
    val kustoQ = csvDf
          .writeStream
          .format("com.microsoft.kusto.spark.datasink.KustoSinkProvider")
          .options(Map(
            KustoOptions.KUSTO_CLUSTER -> cluster,
            KustoOptions.KUSTO_TABLE -> table,
            KustoOptions.KUSTO_DATABASE -> database,
            KustoOptions.KUSTO_AAD_CLIENT_ID -> appId,
            KustoOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey,
            KustoOptions.KUSTO_AAD_AUTHORITY_ID -> authorityId))
          .trigger(Trigger.Once)
    
    kustoQ.start().awaitTermination(TimeUnit.MINUTES.toMillis(8))
    ```

## <a name="spark-source-reading-from-azure-data-explorer"></a>Spark 源：从 Azure 数据资源管理器中读取

1. 在读取较少的数据，定义数据的查询：

    ```scala
    val conf: Map[String, String] = Map(
          KustoOptions.KUSTO_AAD_CLIENT_ID -> appId,
          KustoOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey,
          KustoOptions.KUSTO_QUERY -> s"$table | where (ColB % 1000 == 0) | distinct ColA"      
        )
    
    // Simplified syntax flavor
    import org.apache.spark.sql._
    import com.microsoft.kusto.spark.sql.extension.SparkExtension._
    import org.apache.spark.SparkConf
    
    val df = spark.read.kusto(cluster, database, "", conf)
    display(df)
    ```

1. 在读取大量数据时，必须提供临时 blob 存储。 提供存储容器 SAS 密钥，或存储帐户名称、 帐户密钥和容器名称。 此步骤只是所需的 Spark 连接器的当前预览版本。

    ```scala
    // Use either container/account-key/account name, or container SaS
    val container = dbutils.secrets.get(scope = "KustoDemos", key = "blobContainer")
    val storageAccountKey = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountKey")
    val storageAccountName = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountName")
    // val storageSas = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageSasUrl")
    ```

    在上面的示例中，我们不访问密钥保管库中使用的连接器接口。 或者，我们使用使用 Databricks 机密的更简单方法。

1. 从 Azure 数据资源管理器中读取：

    ```scala
    val df2 = spark.read.kusto(cluster, database, "ReallyBigTable", conf3)
    
    val dfFiltered = df2
      .where(df2.col("ColA").startsWith("row-2"))
      .filter("ColB > 12")
      .filter("ColB <= 21")
      .select("ColA")
    
    display(dfFiltered)
    ```
