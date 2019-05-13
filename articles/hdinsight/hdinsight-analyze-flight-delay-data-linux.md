---
title: '教程：在 HDInsight 上使用 Hive 执行提取、转换、加载 (ETL) 操作 - Azure '
description: 了解如何从原始 CSV 数据集提取数据，在 HDInsight 上使用 Hive 将其转换，然后使用 Apache Sqoop 将已转换的数据加载到 Azure SQL 数据库。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.date: 05/07/2018
ms.author: hrasheed
ms.custom: H1Hack27Feb2017,hdinsightactive,mvc
ms.openlocfilehash: eb86dc8c5c3b215a2c90380b4009efd00d2a243c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64723139"
---
# <a name="tutorial-extract-transform-and-load-data-using-apache-hive-on-azure-hdinsight"></a>教程：使用 Apache Hive on Azure HDInsight 提取、转换和加载数据

在此教程中，需将原始 CSV 数据文件导入 HDInsight 群集存储，然后在 Azure HDInsight 上使用 [Apache Hive](https://hive.apache.org/) 转换数据。 数据转换完毕后，使用 [Apache Sqoop](https://sqoop.apache.org/) 将数据加载到 Azure SQL 数据库。 本文使用公开提供的航班数据。

> [!IMPORTANT]  
> 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是 Azure HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上停用](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

本教程涵盖以下任务： 

> [!div class="checklist"]
> * 下载示例航班数据
> * 将数据上传到 HDInsight 群集
> * 使用 Hive 转换数据
> * 在 Azure SQL 数据库中创建表
> * 使用 Sqoop 将数据导出到 Azure SQL 数据库


下图演示了典型的 ETL 应用程序流。

![在 Azure HDInsight 上使用 Apache Hive 执行 ETL 操作](./media/hdinsight-analyze-flight-delay-data-linux/hdinsight-etl-architecture.png "在 Azure HDInsight 上使用 Apache Hive 执行 ETL 操作")

如果没有 Azure 订阅，请在开始之前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

* **HDInsight 上基于 Linux 的 Hadoop 群集**。 有关创建新的基于 Linux 的 HDInsight 群集的步骤，请参阅[开始在 HDInsight 中使用 Apache Hadoop](hadoop/apache-hadoop-linux-tutorial-get-started.md)。

* **Azure SQL 数据库**。 使用 Azure SQL 数据库作为目标数据存储。 如果没有 SQL 数据库，请参阅[在 Azure 门户中创建 Azure SQL 数据库](../sql-database/sql-database-get-started.md)。

* **Azure CLI**。 如果尚未安装 Azure CLI，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)了解详细步骤。

* **SSH 客户端**。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="download-the-flight-data"></a>下载航班数据

1. 浏览到[美国研究与技术创新管理部门、运输统计局][rita-website]。

2. 在该页面上，选择以下值：

   | 名称 | 值 |
   | --- | --- |
   | 筛选年份 |2013 |
   | 筛选期间 |1 月 |
   | 字段 |Year、FlightDate、UniqueCarrier、Carrier、FlightNum、OriginAirportID、Origin、OriginCityName、OriginState、DestAirportID、Dest、DestCityName、DestState、DepDelayMinutes、ArrDelay、ArrDelayMinutes、CarrierDelay、WeatherDelay、NASDelay、SecurityDelay、LateAircraftDelay。 |
   
   清除所有其他字段。 

3. 选择“下载”。 你将得到一个具有所选数据字段的 zip 文件。

## <a name="upload-data-to-an-hdinsight-cluster"></a>将数据上传到 HDInsight 群集

可通过多种方式将数据上传到与 HDInsight 群集相关的存储。 本部分使用 `scp` 上传数据。 若要了解上传数据的其他方式，请参阅[将数据上传到 HDInsight](hdinsight-upload-data.md)。

1. 打开命令提示符，使用以下命令将 zip 文件上传到 HDInsight 群集头节点：

    ```bash
    scp <FILENAME>.zip <SSH-USERNAME>@<CLUSTERNAME>-ssh.azurehdinsight.net:<FILENAME.zip>
    ```

    将“FILENAME”替换为 zip 文件的名称。 将 *USERNAME* 替换为 HDInsight 群集的 SSH 登录名。 将 *CLUSTERNAME* 替换为 HDInsight 群集的名称。

   > [!NOTE]  
   > 如果使用密码对 SSH 登录名进行身份验证，系统会提示输入密码。 如果使用公钥，可能需要使用 `-i` 参数并指定匹配私钥的路径。 例如，`scp -i ~/.ssh/id_rsa FILENAME.zip USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:`。

2. 上传完成后，使用 SSH 连接到群集。 在命令提示符中输入以下命令：

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

3. 使用以下命令解压缩 .zip 文件：

    ```bash
    unzip FILENAME.zip
    ```

    此命令会提取约为 60 MB 的 .csv 文件。

4. 使用以下命令在 HDInsight 存储上创建目录，然后将 .csv 文件复制到该目录：

    ```bash
    hdfs dfs -mkdir -p /tutorials/flightdelays/data
    hdfs dfs -put <FILENAME>.csv /tutorials/flightdelays/data/
    ```

## <a name="transform-data-using-a-hive-query"></a>使用 Hive 查询转换数据

可通过多种方式在 HDInsight 群集上运行 Hive 作业。 本部分使用 [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline%E2%80%93CommandLineShell) 运行 Hive 作业。 有关以其他方式运行 Hive 作业的信息，请参阅[在 HDInsight 上使用 Apache Hive](./hadoop/hdinsight-use-hive.md)。

在 Hive 作业运行期间，请将 .csv 文件中的数据导入到名为“Delays”的 Hive 表中。

1. 在 HDInsight 群集已有的 SSH 提示符中，使用以下命令创建并编辑名为 flightdelays.hql 的新文件：

    ```bash
    nano flightdelays.hql
    ```

2. 将以下文本用作此文件的内容：

    ```hiveql
    DROP TABLE delays_raw;
    -- Creates an external table over the csv file
    CREATE EXTERNAL TABLE delays_raw (
        YEAR string,
        FL_DATE string,
        UNIQUE_CARRIER string,
        CARRIER string,
        FL_NUM string,
        ORIGIN_AIRPORT_ID string,
        ORIGIN string,
        ORIGIN_CITY_NAME string,
        ORIGIN_CITY_NAME_TEMP string,
        ORIGIN_STATE_ABR string,
        DEST_AIRPORT_ID string,
        DEST string,
        DEST_CITY_NAME string,
        DEST_CITY_NAME_TEMP string,
        DEST_STATE_ABR string,
        DEP_DELAY_NEW float,
        ARR_DELAY_NEW float,
        CARRIER_DELAY float,
        WEATHER_DELAY float,
        NAS_DELAY float,
        SECURITY_DELAY float,
        LATE_AIRCRAFT_DELAY float)
    -- The following lines describe the format and location of the file
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
    LOCATION '/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays AS
    SELECT YEAR AS year,
        FL_DATE AS flight_date,
        substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier,
        substring(CARRIER, 2, length(CARRIER) -1) AS carrier,
        substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num,
        ORIGIN_AIRPORT_ID AS origin_airport_id,
        substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code,
        substring(ORIGIN_CITY_NAME, 2) AS origin_city_name,
        substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr,
        DEST_AIRPORT_ID AS dest_airport_id,
        substring(DEST, 2, length(DEST) -1) AS dest_airport_code,
        substring(DEST_CITY_NAME,2) AS dest_city_name,
        substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr,
        DEP_DELAY_NEW AS dep_delay_new,
        ARR_DELAY_NEW AS arr_delay_new,
        CARRIER_DELAY AS carrier_delay,
        WEATHER_DELAY AS weather_delay,
        NAS_DELAY AS nas_delay,
        SECURITY_DELAY AS security_delay,
        LATE_AIRCRAFT_DELAY AS late_aircraft_delay
    FROM delays_raw;
    ```

2. 要保存文件，请按 Esc，然后按 `:x`。

3. 若要启动 Hive 并运行 **flightdelays.hql** 文件，请使用以下命令：

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
    ```

4. flightdelays.hql 脚本完成运行后，使用以下命令打开交互式 Beeline 会话：

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

5. 收到 `jdbc:hive2://localhost:10001/>` 提示时，使用以下查询从导入的航班延误数据中检索数据：

    ```hiveql
    INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    SELECT regexp_replace(origin_city_name, '''', ''),
        avg(weather_delay)
    FROM delays
    WHERE weather_delay IS NOT NULL
    GROUP BY origin_city_name;
    ```

    此查询会检索遇到天气延迟的城市的列表以及平均延迟时间，并将其保存到 `/tutorials/flightdelays/output` 中。 稍后，Sqoop 会从该位置读取数据并将其导出到 Azure SQL 数据库。

6. 若要退出 Beeline，请在提示符处输入 `!quit`。

## <a name="create-a-sql-database-table"></a>创建 SQL 数据库表

本部分假设已创建 Azure SQL 数据库。 如果没有 SQL 数据库，请使用[在 Azure 门户中创建 Azure SQL 数据库](../sql-database/sql-database-get-started.md)中的信息进行创建。

如果已拥有 SQL 数据库，必须获取服务器的名称。 要在 [Azure 门户](https://portal.azure.com)中查找服务器名称，请选择“SQL 数据库”，然后筛选要使用的数据库的名称。 服务器名称在“服务器名称”列中列出。

![获取 Azure SQL 服务器的详细信息](./media/hdinsight-analyze-flight-delay-data-linux/get-azure-sql-server-details.png "获取 Azure SQL 服务器的详细信息")

> [!NOTE]  
> 有多种方法可连接到 SQL 数据库并创建表。 以下步骤从 HDInsight 群集中使用 [FreeTDS](http://www.freetds.org/)。


1. 若要安装 FreeTDS，请使用以下命令从 SSH 连接到群集：

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

3. 安装完成后，使用以下命令连接到 SQL 数据库服务器。 使用 SQL 数据库服务器名称替换“serverName”。 使用 SQL 数据库登录信息替换 **adminLogin** 和 **adminPassword**。 使用数据库名称替换 **databaseName**。

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -p 1433 -D <databaseName>
    ```

    出现提示时，输入 SQL 数据库管理员登录名的密码。

    将收到类似于以下文本的输出：

    ```
    locale is "en_US.UTF-8"
    locale charset is "UTF-8"
    using default charset "UTF-8"
    Default database being set to sqooptest
    1>
    ```

4. 在 `1>` 提示符下，输入以下行：

    ```hiveql
    CREATE TABLE [dbo].[delays](
    [origin_city_name] [nvarchar](50) NOT NULL,
    [weather_delay] float,
    CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED   
    ([origin_city_name] ASC))
    GO
    ```

    输入 `GO` 语句后，将评估前面的语句。 此查询会创建一个名为 **delays** 且具有聚集索引的表。

    使用以下查询验证是否已创建该表：

    ```hiveql
    SELECT * FROM information_schema.tables
    GO
    ```

    输出与以下文本类似：

    ```
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    databaseName       dbo             delays        BASE TABLE
    ```

5. 在 `1>` 提示符下输入 `exit` 以退出 tsql 实用工具。

## <a name="export-data-to-sql-database-using-apache-sqoop"></a>使用 Apache Sqoop 将数据导出到 SQL 数据库

在前面的部分中，已经在 `/tutorials/flightdelays/output` 复制了转换后的数据。 本部分使用 Sqoop 将数据从 '/tutorials/flightdelays/output` 导出到在 Azure SQL 数据库中创建的表。 

1. 使用以下命令验证 Sqoop 是否可以查看 SQL 数据库：

    ```bash
    sqoop list-databases --connect jdbc:sqlserver://<serverName>.database.windows.net:1433 --username <adminLogin> --password <adminPassword>
    ```

    此命令会返回数据库列表，其中包括此前创建的 delays 表所在的数据库。

2. 使用以下命令将 hivesampletable 中的数据导出到 delays 表：

    ```bash
    sqoop export --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=<databaseName>' --username <adminLogin> --password <adminPassword> --table 'delays' --export-dir '/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
    ```

    Sqoop 连接到包含 delays 表的数据库，并将数据从 `/tutorials/flightdelays/output` 目录导出到 delays 表。

3. Sqoop 命令完成后，使用 tsql 实用工具连接到数据库：

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D <databaseName>
    ```

    使用以下语句验证数据是否已导出到 delays 表：

    ```sql
    SELECT * FROM delays
    GO
    ```

    会在表中看到一系列数据。 该表包括城市名称和该城市的平均航班延迟时间。 

    键入 `exit` 退出 tsql 实用工具。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何在 HDInsight 中使用 Apache Hadoop 群集执行提取、转换和加载数据操作。 前往下一教程，了解如何使用 Azure 数据工厂创建按需 HDInsight Hadoop 群集。

> [!div class="nextstepaction"]
>[使用 Azure 数据工厂在 HDInsight 中创建按需 Apache Hadoop 群集](hdinsight-hadoop-create-linux-clusters-adf.md)

若要了解使用 HDInsight 中的数据的更多方式，请参阅以下文章：

* [教程：使用 Apache Hive on Azure HDInsight 提取、转换和加载数据](../storage/data-lake-storage/tutorial-extract-transform-load-hive.md)
* [将 Apache Hive 和 HDInsight 配合使用][hdinsight-use-hive]
* [将 Apache Pig 和 HDInsight 配合使用][hdinsight-use-pig]
* [为 Apache Hadoop on HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]

* [将 Apache Oozie 和 HDInsight 配合使用][hdinsight-use-oozie]
* [将 Apache Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]



[azure-purchase-options]: https://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: https://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: https://azure.microsoft.com/pricing/free-trial/


[rita-website]: https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
[cindygross-hive-tables]: https://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[hdinsight-use-oozie]: hdinsight-use-oozie-linux-mac.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-get-started]: hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-use-sqoop]:hadoop/apache-hadoop-use-sqoop-mac-linux.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md

[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[hadoop-hiveql]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL

[technetwiki-hive-error]: https://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
