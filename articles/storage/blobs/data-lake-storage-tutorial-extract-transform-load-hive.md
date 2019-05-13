---
title: 教程：使用 Apache Hive on Azure HDInsight 执行提取、转换、加载 (ETL) 操作
description: 本教程介绍如何从原始 CSV 数据集提取数据，使用 Apache Hive on Azure HDInsight 将其转换，然后使用 Sqoop 将已转换的数据加载到 Azure SQL 数据库。
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: be7ce4d96b7c1bd17853447448f06070637c7855
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64939195"
---
# <a name="tutorial-extract-transform-and-load-data-by-using-apache-hive-on-azure-hdinsight"></a>教程：使用 Apache Hive on Azure HDInsight 提取、转换和加载数据

本教程执行 ETL 操作：提取、转换和加载数据。 有了原始 CSV 数据文件以后，将其导入 Azure HDInsight 群集，使用 Apache Hive 对其进行转换，然后使用 Apache Sqoop 将其加载到 Azure SQL 数据库。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 提取数据并将其上传到 HDInsight 群集。
> * 使用 Apache Hive 转换数据。
> * 使用 Sqoop 将数据加载到 Azure SQL 数据库。

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

* **一个针对 HDInsight 配置的 Azure Data Lake Storage Gen2 存储帐户**

    请参阅[配合使用 Azure Data Lake Storage Gen2 和 Azure HDInsight 群集](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2)。

* **HDInsight 上基于 Linux 的 Hadoop 群集**

    请参阅[快速入门：通过 Azure 门户在 Azure HDInsight 中开始使用 Apache Hadoop 和 Apache Hive](https://docs.microsoft.com/azure/hdinsight/hadoop/apache-hadoop-linux-create-cluster-get-started-portal)。

* **Azure SQL 数据库**：使用 Azure SQL 数据库作为目标数据存储。 如果没有 SQL 数据库，请参阅[在 Azure 门户中创建 Azure SQL 数据库](../../sql-database/sql-database-get-started.md)。

* **Azure CLI**：如果尚未安装 Azure CLI，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。

* **安全外壳 (SSH) 客户端**：有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Hadoop)](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。

> [!IMPORTANT]
> 本文中的步骤需要一个使用 Linux 的 HDInsight 群集。 Linux 是 Azure HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上停用](../../hdinsight/hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="download-the-flight-data"></a>下载航班数据

1. 浏览到[美国研究与技术创新管理部门、运输统计局](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time)。

2. 在该页面上，选择以下值：

   | 名称 | 值 |
   | --- | --- |
   | 筛选年份 |2013 |
   | 筛选期间 |1 月 |
   | 字段 |Year、FlightDate、Reporting_Airline、IATA_CODE_Reporting_Airline、Flight_Number_Reporting_Airline、OriginAirportID、Origin、OriginCityName、OriginState、DestAirportID、Dest、DestCityName、DestState、DepDelayMinutes、ArrDelay、ArrDelayMinutes、CarrierDelay、WeatherDelay、NASDelay、SecurityDelay、LateAircraftDelay。 |
   
   清除所有其他字段。

3. 选择“下载”。 你将得到一个具有所选数据字段的 zip 文件。

## <a name="extract-and-upload-the-data"></a>提取并上传数据

在此部分中，会将数据上传到 HDInsight 群集，然后将该数据复制到 Data Lake Storage Gen2 帐户。

1. 打开命令提示符，使用以下安全负责 (Scp) 命令将 zip 文件上传到 HDInsight 群集头节点：

   ```bash
   scp <file-name>.zip <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net:<file-name.zip>
   ```

   * 将 `<file-name>` 占位符替换为 .zip 文件的名称。
   * 将 `<ssh-user-name>` 占位符替换为 HDInsight 群集的 SSH 登录名。
   * 将 `<cluster-name>` 占位符替换为 HDInsight 群集的名称。

   如果使用密码对 SSH 登录名进行身份验证，系统会提示输入密码。

   如果使用公钥，可能需要使用 `-i` 参数并指定匹配私钥的路径。 例如，`scp -i ~/.ssh/id_rsa <file_name>.zip <user-name>@<cluster-name>-ssh.azurehdinsight.net:`。

2. 上传完成后，使用 SSH 连接到群集。 在命令提示符中输入以下命令：

   ```bash
   ssh <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net
   ```

3. 使用以下命令解压缩 .zip 文件：

   ```bash
   unzip <file-name>.zip
   ```

   此命令会提取 **.csv** 文件。

4. 使用以下命令创建 Data Lake Storage Gen2 文件系统。

   ```bash
   hadoop fs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/
   ```

   将 `<file-system-name>` 占位符替换为你要为文件系统提供的名称。

   将 `<storage-account-name>` 占位符替换为存储帐户的名称。

5. 使用以下命令创建目录。

   ```bash
   hdfs dfs -mkdir -p abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data
   ```

6. 使用以下命令将 *.csv* 文件复制到目录：

   ```bash
   hdfs dfs -put "<file-name>.csv" abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data/
   ```

   如果文件名包含空格或特殊字符，请对文件名使用引号。

## <a name="transform-the-data"></a>转换数据

本部分使用 Beeline 运行 Apache Hive 作业。

在 Apache Hive 作业运行期间，请将 .csv 文件中的数据导入到名为“delays”的 Apache Hive 表中。

1. 在 HDInsight 群集已有的 SSH 提示符中，使用以下命令创建并编辑名为 flightdelays.hql 的新文件：

   ```bash
   nano flightdelays.hql
   ```

2. 修改以下文本，将 `<file-system-name>` 和 `<storage-account-name>` 占位符值替换为文件系统名称和存储帐户名称。 然后将文本复制并粘贴到 nano 控制台中，方法是同时按 SHIFT 键和鼠标右键单击按钮。

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
    LOCATION 'abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays
    LOCATION 'abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/processed'
    AS
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

3. 保存文件，方法是使用 CTRL+X，然后根据提示键入 `Y`。

4. 若要启动 Hive 并运行 **flightdelays.hql** 文件，请使用以下命令：

   ```bash
   beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
   ```

5. flightdelays.hql 脚本完成运行后，使用以下命令打开交互式 Beeline 会话：

   ```bash
   beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
   ```

6. 收到 `jdbc:hive2://localhost:10001/>` 提示时，使用以下查询从导入的航班延误数据中检索数据：

    ```hiveql
    INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    SELECT regexp_replace(origin_city_name, '''', ''),
        avg(weather_delay)
    FROM delays
    WHERE weather_delay IS NOT NULL
    GROUP BY origin_city_name;
    ```

   此查询会检索遇到天气延迟的城市的列表以及平均延迟时间，并将其保存到 `abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output` 中。 稍后，Sqoop 会从该位置读取数据并将其导出到 Azure SQL 数据库。

7. 若要退出 Beeline，请在提示符处输入 `!quit`。

## <a name="create-a-sql-database-table"></a>创建 SQL 数据库表

需要使用 SQL 数据库中的服务器名称才能执行此操作。 完成以下步骤即可找到服务器名称。

1. 转到 [Azure 门户](https://portal.azure.com)。

2. 选择“SQL 数据库”。

3. 针对选择使用的数据库的名称进行筛选。 服务器名称在“服务器名称”列中列出。

4. 针对要使用的数据库的名称进行筛选。 服务器名称在“服务器名称”列中列出。

    ![获取 Azure SQL 服务器的详细信息](./media/data-lake-storage-tutorial-extract-transform-load-hive/get-azure-sql-server-details.png "获取 Azure SQL 服务器的详细信息")

    有多种方法可连接到 SQL 数据库并创建表。 以下步骤从 HDInsight 群集中使用 [FreeTDS](http://www.freetds.org/)。

5. 若要安装 FreeTDS，请使用以下命令从 SSH 连接到群集：

   ```bash
   sudo apt-get --assume-yes install freetds-dev freetds-bin
   ```

6. 安装完成后，使用以下命令连接到 SQL 数据库服务器。

   ```bash
   TDSVER=8.0 tsql -H '<server-name>.database.windows.net' -U '<admin-login>' -p 1433 -D '<database-name>'
    ```
   * 将 `<server-name>` 占位符替换为 SQL 数据库服务器名称。

   * 使用 SQL 数据库的管理员登录名替换 `<admin-login>` 占位符。

   * 将 `<database-name>` 占位符替换为数据库名称

   出现提示时，输入 SQL 数据库管理员登录名的密码。

   将收到类似于以下文本的输出：

   ```
   locale is "en_US.UTF-8"
   locale charset is "UTF-8"
   using default charset "UTF-8"
   Default database being set to sqooptest
   1>
   ```

7. 在 `1>` 提示符下输入以下语句：

   ```hiveql
   CREATE TABLE [dbo].[delays](
   [OriginCityName] [nvarchar](50) NOT NULL,
   [WeatherDelay] float,
   CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
   ([OriginCityName] ASC))
   GO
   ```

8. 输入 `GO` 语句后，将评估前面的语句。

   此查询创建一个名为 **delays** 且具有聚集索引的表。

9. 使用以下查询验证是否已创建表：

   ```hiveql
   SELECT * FROM information_schema.tables
   GO
   ```

   输出与以下文本类似：

   ```
   TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
   databaseName       dbo             delays        BASE TABLE
   ```

10. 在 `1>` 提示符下输入 `exit` 以退出 tsql 实用工具。

## <a name="export-and-load-the-data"></a>导出和加载数据

在前面的部分中，已经在 `abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output` 位置复制了转换后的数据。 本部分使用 Sqoop 将数据从 `abfs://<file-system-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output` 导出到在 Azure SQL 数据库中创建的表。

1. 使用以下命令验证 Sqoop 是否可以查看 SQL 数据库：

   ```bash
   sqoop list-databases --connect jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433 --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD>
   ```

   此命令会返回数据库列表，其中包括创建的 **delays** 表所在的数据库。

2. 使用以下命令将 **hivesampletable** 表中的数据导出到 **delays** 表：

   ```bash
   sqoop export --connect 'jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433;database=<DATABASE_NAME>' --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD> --table 'delays' --export-dir 'abfs://<file-system-name>@.dfs.core.windows.net/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
   ```

   Sqoop 连接到包含 **delays** 表的数据库，并将数据从 `/tutorials/flightdelays/output` 目录导出到 **delays** 表。

3. `sqoop` 命令完成后，使用 tsql 实用程序连接到数据库：

   ```bash
   TDSVER=8.0 tsql -H <SERVER_NAME>.database.windows.net -U <ADMIN_LOGIN> -P <ADMIN_PASSWORD> -p 1433 -D <DATABASE_NAME>
   ```

4. 使用以下语句验证数据是否已导出到 **delays** 表：

   ```sql
   SELECT * FROM delays
   GO
   ```

   会在表中看到一系列数据。 该表包括城市名称和该城市的平均航班延迟时间。

5. 输入 `exit`，退出 tsql 实用程序。

## <a name="clean-up-resources"></a>清理资源

本教程中使用的所有资源都预先存在。 不需清理。

## <a name="next-steps"></a>后续步骤

若要了解使用 HDInsight 中的数据的更多方式，请参阅以下文章：

> [!div class="nextstepaction"]
> [将 Azure Data Lake Storage Gen2 用于 Azure HDInsight 群集](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
