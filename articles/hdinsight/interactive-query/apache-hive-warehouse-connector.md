---
title: 将 Apache Spark 和 Apache Hive 使用的配置单元仓库连接器
description: 了解如何使用 Azure HDInsight 上的 Hive 仓库连接器将 Apache Spark 和 Apache Hive 进行集成。
ms.service: hdinsight
author: nakhanha
ms.author: nakhanha
ms.reviewer: hrasheed
ms.topic: conceptual
ms.date: 04/29/2019
ms.openlocfilehash: b2b3d1ac0a7c0e917f87be1dd131120f63a70f8e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142816"
---
# <a name="integrate-apache-spark-and-apache-hive-with-the-hive-warehouse-connector"></a>将 Apache Spark 和 Apache Hive 使用的配置单元仓库连接器

Apache Hive 仓库连接器 (HWC) 是一个库，可用于更轻松地与 Apache Spark 和 Apache Hive 处理通过支持任务，例如 Spark 数据帧和 Hive 表之间移动数据并也使 Spark 流式数据处理到 Hive 表。 配置单元仓库连接器工作方式类似于 Spark 和 Hive 之间的桥梁。 它支持 Scala、 Java 和 Python 进行开发。

配置单元仓库连接器，可充分利用 Hive 和 Spark 来构建功能强大的大数据应用程序的独特功能。 Apache Hive 提供了对数据库事务的原子、 一致、 隔离和持久 (ACID) 的支持。 有关 ACID 的详细信息和配置单元中的事务，请参阅[Hive 事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)。 Hive 还提供了详细的安全控件通过 Apache Ranger 和低延迟分析处理在 Apache Spark 中不可用。

Apache Spark，有一个为提供 Apache Hive 中不可用的流式处理功能的结构化流式处理 API。 从开始使用 Hortonworks 数据平台 (HDP) 3.0，Apache Spark 和 Apache Hive 具有单独的元存储，可以进行互操作性非常困难。 配置单元仓库连接器可以容易地使用 Spark 和 Hive 组合在一起。 HWC 库将数据从 LLAP 守护程序加载到并行的 Spark 执行器使其更高效、 可缩放比使用从 Spark 到 Hive 的标准 JDBC 连接。

![体系结构](./media/apache-hive-warehouse-connector/hive-warehouse-connector-architecture.png)

下面是一些 Hive 仓库连接器支持的操作：

* 描述表
* 为 ORC 格式的数据创建表
* 选择 Hive 数据和检索数据帧
* 在批处理中写入 Hive 数据帧
* 执行 Hive update 语句
* 从 Hive 读取表数据、 将其转换在 Spark 中，并写入到新的 Hive 表
* 数据帧或 Spark 流写入 Hive 使用 HiveStreaming

## <a name="hive-warehouse-connector-setup"></a>配置单元仓库连接器安装程序

请执行以下步骤设置 Azure HDInsight 中的 Spark 和交互式查询群集之间的 Hive 仓库连接器：

1. 创建存储帐户和自定义的 Azure 虚拟网络中使用 Azure 门户的 HDInsight Spark 4.0 群集。 有关在 Azure 虚拟网络中创建群集的信息，请参阅[到现有的虚拟网络添加 HDInsight](../../hdinsight/hdinsight-extend-hadoop-virtual-network.md#existingvnet)。
1. 创建与 Spark 群集使用相同的存储帐户和 Azure 虚拟网络中使用 Azure 门户的 HDInsight 交互式查询 (LLAP) 4.0 群集。
1. 将复制的内容`/etc/hosts`文件上的交互式查询群集到 headnode0`/etc/hosts`上 Spark 群集的 headnode0 文件。 此步骤将允许你的 Spark 群集，若要解决的交互式查询群集中节点的 IP 地址。 查看与更新的文件的内容`cat /etc/hosts`。 输出应如下所示的屏幕截图中所示。

    ![查看主机文件](./media/apache-hive-warehouse-connector/hive-warehouse-connector-hosts-file.png)

1. 通过执行以下步骤配置的 Spark 群集设置： 
    1. 转到 Azure 门户，选择 HDInsight 群集，然后单击你的群集名称上。
    1. 在右侧下,**群集仪表板**，选择**Ambari 家庭**。
    1. 在 Ambari web UI 中，单击**SPARK2** > **配置** > **自定义 spark2-defaults**。

        ![Spark2 Ambari 配置](./media/apache-hive-warehouse-connector/hive-warehouse-connector-spark2-ambari.png)

    1. 设置`spark.hadoop.hive.llap.daemon.service.hosts`为相同的值作为属性**LLAP 应用名称**下**高级 hive-交互式 env**。 例如： `@llap0`

    1. 设置`spark.sql.hive.hiveserver2.jdbc.url`到 JDBC 连接字符串，它连接到 Hiveserver2 在交互式查询群集上。 你的群集的连接字符串将类似下面的 URI。 `CLUSTERNAME` Spark 群集的名称和`user`和`password`参数设置为你的群集的正确值。

        ```
        jdbc:hive2://LLAPCLUSTERNAME.azurehdinsight.net:443/;user=admin;password=PWD;ssl=true;transportMode=http;httpPath=/hive2
        ```

        >[!Note] 
        > JDBC URL 应包含用来连接到 Hiveserver2 凭据包括用户名和密码。

    1. 设置`spark.datasource.hive.warehouse.load.staging.dir`到合适的 HDFS 兼容临时目录。 如果您有两个不同的群集，临时目录应是 LLAP 群集的存储帐户的临时目录中的文件夹，以便有权访问 HiveServer2。 例如，`wasb://STORAGE_CONTAINER_NAME@STORAGE_ACCOUNT_NAME.blob.core.windows.net/tmp`其中`STORAGE_ACCOUNT_NAME`是群集中，正在使用的存储帐户的名称和`STORAGE_CONTAINER_NAME`是存储容器的名称。

    1. 设置`spark.datasource.hive.warehouse.metastoreUri`元存储的交互式查询群集的 URI 的值。 若要查找 LLAP 群集 metastoreUri，寻找**hive.metastore.uris**属性在 Ambari UI 中为你 LLAP 群集下**Hive** > **高级**  > **常规**。 值将类似于下面的 URI:

        ```
        thrift://hn0-hwclla.0iv2nyrmse1uvp2caa4e34jkmf.cx.internal.cloudapp.net:9083,
        thrift://hn1-hwclla.0iv2nyrmse1uvp2caa4e34jkmf.cx.internal.cloudapp.net:9083
        ```

    1. 设置`spark.security.credentials.hiveserver2.enabled`到`false`YARN 客户端部署模式。
    1. 设置`spark.hadoop.hive.zookeeper.quorum`到 LLAP 群集的 Zookeeper 仲裁。 若要查找 LLAP 群集的 Zookeeper 仲裁，寻找**hive.zookeeper.quorum**属性在 Ambari UI 中为你 LLAP 群集下**Hive** > **高级** > **高级 hive 站点**。 值将类似于以下字符串：

        ```
        zk1-nkhvne.0iv2nyrmse1uvp2caa4e34jkmf.cx.internal.cloudapp.net:2181,
        zk4-nkhvne.0iv2nyrmse1uvp2caa4e34jkmf.cx.internal.cloudapp.net:2181,
        zk6-nkhvne.0iv2nyrmse1uvp2caa4e34jkmf.cx.internal.cloudapp.net:2181
        ```

若要测试的配置单元仓库连接器配置，请执行部分中的步骤[连接和运行查询](#connecting-and-running-queries)。

## <a name="using-the-hive-warehouse-connector"></a>使用 Hive 仓库连接器

### <a name="connecting-and-running-queries"></a>连接并运行查询

您可以选择几个不同的方法来连接到交互式查询群集并使用 Hive 仓库连接器执行查询。 受支持的方法包括以下工具：

* [spark-shell](../spark/apache-spark-shell.md)
* PySpark
* spark-submit
* [Zeppelin](../spark/apache-spark-zeppelin-notebook.md)
* [Livy](../spark/apache-spark-livy-rest-interface.md)

在本文中的所有示例将都执行通过 spark shell。

若要启动 spark shell 会话，请执行以下步骤：

1. 通过 ssh 连接到群集头节点。 有关连接到使用 SSH 在群集的详细信息，请参阅[连接到 HDInsight (Apache Hadoop) 使用 SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。
1. 通过键入更改到正确的目录`cd /usr/hdp/current/hive_warehouse_connector`或提供用作 spark shell 命令中的参数的所有 jar 文件的完整路径。
1. 输入以下命令以启动 spark shell:

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/3.0.1.0-183/hive_warehouse_connector/hive-warehouse-connector-assembly-1.0.0.3.0.1.0-183.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. 您将看到一条欢迎消息和一个`scala>`可以在其中输入命令提示符。

1. 启动 spark shell 之后, 以配置单元仓库连接器实例都可以使用以下命令启动：

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

### <a name="connecting-and-running-queries-on-enterprise-security-package-esp-clusters"></a>连接并在企业安全包 (ESP) 群集上运行查询

企业安全包 (ESP) 提供了基于 Active Directory 的身份验证、 多用户支持和 Azure HDInsight 中的 Apache Hadoop 群集的基于角色的访问控制等企业级功能。 ESP 的详细信息，请参阅[含有 Enterprise Security 包的 Apache Hadoop 安全性简介](../domain-joined/apache-domain-joined-introduction.md)。

1. 请执行初始步骤 1 和 2 下的[连接和运行查询](#connecting-and-running-queries)。
1. 类型`kinit`，并使用域用户登录。
1. 启动 spark shell 配置参数，如下所示的完整列表。 必须在群集上基于指定所有字母都大写尖括号之间的值。 如果您需要找出要为以下参数的任何输入的值，请参阅部分上[Hive 仓库连接器安装程序](#hive-warehouse-connector-setup)。:

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/3.0.1.0-183/hive_warehouse_connector/hive-warehouse-connector-assembly-1.0.0.3.0.1.0-183.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    --conf spark.hadoop.hive.llap.daemon.service.hosts='<LLAP_APP_NAME>'
    --conf spark.sql.hive.hiveserver2.jdbc.url='jdbc:hive2://<ZOOKEEPER_QUORUM>;serviceDiscoveryMode=zookeeper;zookeeperNamespace=hiveserver2-interactive'
    --conf spark.datasource.hive.warehouse.load.staging.dir='<STAGING_DIR>'
    --conf spark.datasource.hive.warehouse.metastoreUri='<METASTORE_URI>'
    --conf spark.hadoop.hive.zookeeper.quorum='<ZOOKEEPER_QUORUM>'
   ```

### <a name="creating-spark-dataframes-from-hive-queries"></a>从 Hive 查询创建 Spark 数据帧

使用 HWC 库的所有查询的结果作为数据帧。 以下示例演示如何创建基本查询。

```scala
hive.setDatabase("default")
val df = hive.executeQuery("select * from hivesampletable")
df.filter("state = 'Colorado'").show()
```

查询的结果是可用于 Spark 库，如 MLIB 和 SparkSQL Spark 数据框架。

### <a name="writing-out-spark-dataframes-to-hive-tables"></a>写出 Spark DataFrames 到 Hive 表

Spark 本机不支持写入 Hive 的管理 ACID 表。 使用 HWC，但是，您可以写出到 Hive 表的任何数据帧。 您可以看到此功能在下面的示例中的工作：

1. 创建一个名为表`sampletable_colorado`并指定其列使用以下命令：

    ```scala
    hive.createTable("sampletable_colorado").column("clientid","string").column("querytime","string").column("market","string").column("deviceplatform","string").column("devicemake","string").column("devicemodel","string").column("state","string").column("country","string").column("querydwelltime","double").column("sessionid","bigint").column("sessionpagevieworder","bigint").create()
    ```

2. 筛选表`hivesampletable`其中该列`state`等于`Colorado`。 此 Hive 表的查询作为 Spark 数据帧返回。 数据帧保存在 Hive 表，然后`sampletable_colorado`使用`write`函数。
    
    ```scala
    hive.table("hivesampletable").filter("state = 'Colorado'").write.format(HiveWarehouseSession.HIVE_WAREHOUSE_CONNECTOR).option("table","sampletable_colorado").save()
    ```

可以看到生成的表中的屏幕截图。

![显示生成的表](./media/apache-hive-warehouse-connector/hive-warehouse-connector-show-hive-table.png)

### <a name="structured-streaming-writes"></a>结构化流写入

使用 Hive 仓库连接器，可以使用 Spark 流式处理将数据写入 Hive 表。

请按照以下步骤来创建一个配置单元仓库连接器的示例，从 Spark 流到 Hive 表中的 localhost 端口 9999 上引入数据。

1. 打开你的 Spark 群集上的终端。
1. 首先，使用以下命令的 spark 流：

    ```scala
    val lines = spark.readStream.format("socket").option("host", "localhost").option("port",9988).load()
    ```

1. 生成数据的 Spark 流的创建，通过执行以下步骤：
    1. 打开另一个终端同一个 Spark 群集上。
    1. 在命令提示符处，键入：`nc -lk 9999`。 此命令使用 netcat 就实用工具从命令行，将数据发送到指定的端口。
    1. 键入你想要引入，Spark 流的单词后跟回车符。
        ![spark 流的输入的数据](./media/apache-hive-warehouse-connector/hive-warehouse-connector-spark-stream-data-input.png)
1. 创建一个新的 Hive 表以保存流式处理数据。 在 spark shell 中，键入以下命令：

    ```scala
    hive.createTable("stream_table").column("value","string").create()
    ```

1. 流式处理的数据写入新创建的表使用以下命令：

    ```scala
    lines.filter("value = 'HiveSpark'").writeStream.format(HiveWarehouseSession.STREAM_TO_STREAM).option("database", "default").option("table","stream_table").option("metastoreUri",spark.conf.get("spark.datasource.hive.warehouse.metastoreUri")).option("checkpointLocation","/tmp/checkpoint1").start()
    ```

    >[!Important]
    > `metastoreUri`和`database`必须选项当前设置手动由于中的 Apache Spark 的已知问题。 有关此问题的详细信息，请参阅[SPARK 25460](https://issues.apache.org/jira/browse/SPARK-25460)。

1. 您可以查看的数据插入到表中使用以下命令：

    ```scala
    hive.table("stream_table").show()
    ```

### <a name="securing-data-on-spark-esp-clusters"></a>在 Spark ESP 群集保护数据安全

1. 创建一个表`demo`用一些示例数据通过输入以下命令：

    ```scala
    create table demo (name string);
    INSERT INTO demo VALUES ('HDinsight');
    INSERT INTO demo VALUES ('Microsoft');
    INSERT INTO demo VALUES ('InteractiveQuery');
    ```

1. 查看表的内容，使用以下命令。 应用该策略之前`demo`表显示了完整的列。

    ```scala
    hive.executeQuery("SELECT * FROM demo").show()
    ```

    ![演示表，然后应用 ranger 策略](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-before-ranger-policy.png)

1. 将应用屏蔽策略，仅显示的列的最后四个字符的列。  
    1. 转到 Ranger 管理 UI 在`https://CLUSTERNAME.azurehdinsight.net/ranger/`。
    1. 单击你的群集下的 Hive 服务**Hive**。
        ![演示表，然后应用 ranger 策略](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-service-manager.png)
    1. 单击**屏蔽**选项卡，然后**添加新策略**![策略列表](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-hive-policy-list.png)
    1. 提供所需的策略的名称。 选择数据库：**默认值**，Hive 表：**演示**，Hive 列：**名称**，用户： **rsadmin2**，访问类型：**选择**，和**部分掩码： 显示最后 4**从**选择屏蔽选项**菜单。 单击“添加”。
                ![策略列表](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-create-policy.png)
1. 再次查看表的内容。 在应用之后的 ranger 策略，我们可以看到只有列的最后四个字符。

    ![演示应用 ranger 策略后的表](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-after-ranger-policy.png)

## <a name="next-steps"></a>后续步骤

* [通过 HDInsight 使用交互式查询](https://docs.microsoft.com/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
* [与使用 Zeppelin，Livy，Hive 仓库连接器进行交互的示例将 spark 提交和 pyspark](https://community.hortonworks.com/articles/223626/integrating-apache-hive-with-apache-spark-hive-war.html)
