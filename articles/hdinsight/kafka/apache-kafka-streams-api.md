---
title: '教程：使用 Apache Kafka Streams API - Azure HDInsight '
description: 了解如何将 Apache Kafka Streams API 与 Kafka on HDInsight 配合使用。 此 API 允许在 Kafka 中的主题之间执行流处理。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 04/02/2019
ms.openlocfilehash: 9425af0f39d14287b49fe06a81172281feb24e83
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64715969"
---
# <a name="tutorial-apache-kafka-streams-api"></a>教程：Apache Kafka Streams API

了解如何创建一个使用 Apache Kafka Streams API 的应用程序并在 Kafka on HDInsight 上运行该应用程序。 

本教程中使用的应用程序是流式处理字数统计。 它从 Kafka 主题读取文本数据、提取各个单词，然后将单词和计数存储到另一个 Kafka 主题。

> [!NOTE]  
> Kafka 流处理通常使用 Apache Spark 或 Apache Storm 完成。 Kafka 版本 1.1.0（在 HDInsight 3.5 和 3.6 中）引入了 Kafka Streams API。 通过此 API 可以在输入和输出主题间转换数据流。 在某些情况下，这可以作为创建 Spark 或 Storm 流式处理解决方案的替代方法。 
>
> 有关 Kafka Streams 的详细信息，请参阅 Apache.org 上的 [Streams 简介](https://kafka.apache.org/10/documentation/streams/)文档。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 了解代码
> * 生成并部署应用程序
> * 配置 Kafka 主题
> * 运行代码

## <a name="prerequisites"></a>先决条件

* Kafka on HDInsight 3.6 群集。 若要了解如何创建 Kafka on HDInsight 群集，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)文档。

* 完成 [Apache Kafka 使用者和生成者 API](apache-kafka-producer-consumer-api.md) 文档中的步骤。 本文档中的步骤使用本教程中创建的示例应用程序和主题。

* [Java Developer Kit (JDK) 版本 8](https://aka.ms/azure-jdks) 或等效工具，例如 OpenJDK。

* 根据 Apache 要求正确[安装](https://maven.apache.org/install.html)的 [Apache Maven](https://maven.apache.org/download.cgi)。  Maven 是 Java 项目的项目生成系统。

* SSH 客户端。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="understand-the-code"></a>了解代码

示例应用程序位于 `Streaming` 子目录的 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 中。 应用程序由两个文件组成：

* `pom.xml`：此文件定义项目依赖项、Java 版本和打包方法。
* `Stream.java`：此文件实现流式处理逻辑。

### <a name="pomxml"></a>Pom.xml

在 `pom.xml` 文件中要了解的重要事项：

* 依赖项：此项目依赖于 `kafka-clients` 包提供的 Kafka Streams API。 以下 XML 代码定义此依赖项：

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>${kafka.version}</version>
    </dependency>
    ```

    > [!NOTE]  
    > `${kafka.version}` 条目在 `pom.xml` 的 `<properties>..</properties>` 部分进行声明，并配置为 HDInsight 群集的 Kafka 版本。

* 插件：Maven 插件提供各种功能。 此项目使用了以下插件：

    * `maven-compiler-plugin`：用于将项目使用的 Java 版本设置为 8。 HDInsight 3.6 需要 Java 8。
    * `maven-shade-plugin`：用于生成包含此应用程序以及任何依赖项的 uber jar。 它还用于设置应用程序的入口点，以便直接运行 Jar 文件，而无需指定主类。

### <a name="streamjava"></a>Stream.java

[Stream.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Streaming/src/main/java/com/microsoft/example/Stream.java) 文件使用 Streams API 实现字数统计应用程序。 它从名为 `test` 的 Kafka 主题读取数据，并将字数统计写到名为 `wordcounts` 的主题中。

以下代码定义字数统计应用程序：

```java
package com.microsoft.example;

import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KStreamBuilder;

import java.util.Arrays;
import java.util.Properties;

public class Stream
{
    public static void main( String[] args ) {
        Properties streamsConfig = new Properties();
        // The name must be unique on the Kafka cluster
        streamsConfig.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-example");
        // Brokers
        streamsConfig.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, args[0]);
        // SerDes for key and values
        streamsConfig.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        streamsConfig.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        // Serdes for the word and count
        Serde<String> stringSerde = Serdes.String();
        Serde<Long> longSerde = Serdes.Long();

        KStreamBuilder builder = new KStreamBuilder();
        KStream<String, String> sentences = builder.stream(stringSerde, stringSerde, "test");
        KStream<String, Long> wordCounts = sentences
                .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
                .map((key, word) -> new KeyValue<>(word, word))
                .countByKey("Counts")
                .toStream();
        wordCounts.to(stringSerde, longSerde, "wordcounts");

        KafkaStreams streams = new KafkaStreams(builder, streamsConfig);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

## <a name="build-and-deploy-the-example"></a>生成并部署示例

按照以下步骤生成项目并将其部署到 Kafka on HDInsight 群集：

1. 将当前目录设置为 `hdinsight-kafka-java-get-started-master\Streaming` 目录的位置，然后使用以下命令创建一个 jar 包：

    ```cmd
    mvn clean package
    ```

    此命令在 `target/kafka-streaming-1.0-SNAPSHOT.jar` 创建包。

2. 将 `sshuser` 替换为群集的 SSH 用户，并将 `clustername` 替换为群集的名称。 使用以下命令将 `kafka-streaming-1.0-SNAPSHOT.jar` 文件复制到 HDInsight 群集。 出现提示时，输入 SSH 用户帐户的密码。

    ```cmd
    scp ./target/kafka-streaming-1.0-SNAPSHOT.jar sshuser@clustername-ssh.azurehdinsight.net:kafka-streaming.jar
    ```

## <a name="create-apache-kafka-topics"></a>创建 Apache Kafka 主题

1. 将 `sshuser` 替换为群集的 SSH 用户，并将 `CLUSTERNAME` 替换为群集的名称。 输入以下命令，打开到群集的 SSH 连接。 出现提示时，输入 SSH 用户帐户的密码。

    ```bash
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. 安装 [jq](https://stedolan.github.io/jq/)，一个命令行 JSON 处理程序。 在打开的 SSH 连接中，输入以下命令以安装 `jq`：

    ```bash
    sudo apt -y install jq
    ```

3. 设置环境变量。 将 `PASSWORD` 和 `CLUSTERNAME` 分别替换为群集登录密码和群集名称，然后输入命令：

    ```bash
    export password='PASSWORD'
    export clusterNameA='CLUSTERNAME'
    ```

4. 提取具有正确大小写格式的群集名称。 群集名称的实际大小写格式可能出乎预期，具体取决于群集的创建方式。 此命令会获取实际的大小写格式，将其存储在变量中，然后显示具有正确大小写格式的名称，以及此前提供的名称。 输入以下命令：

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "https://$clusterNameA.azurehdinsight.net/api/v1/clusters" \
  	| jq -r '.items[].Clusters.cluster_name')
    echo $clusterName, $clusterNameA
    ```

5. 若要获取 Kafka 代理主机和 Apache Zookeeper 主机，请使用以下命令。 出现提示时，输入群集登录（管理员）帐户的密码。 系统会提示输入密码两次。

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin:$password -G \
    https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER \
  	| jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`;
    export KAFKABROKERS=`curl -sS -u admin:$password -G \
    https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER \
  	| jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`;
    ```

6. 若要创建流式处理操作所使用的主题，请使用以下命令：

    > [!NOTE]  
    > 可能会收到错误：`test` 主题已存在。 此错误可忽略，因为该主题可能在生成者和使用者 API 教程中创建。

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

    主题用于以下目的：

   * `test`：本主题是接收记录的位置。 流式处理应用程序从此处读取。
   * `wordcounts`：本主题是流式处理应用程序存储其输出的位置。
   * `RekeyedIntermediateTopic`：本主题用于在 `countByKey` 运算符更新计数时对数据进行重新分区。
   * `wordcount-example-Counts-changelog`：本主题是 `countByKey` 操作使用的状态存储

     > [!IMPORTANT]  
     > Kafka on HDInsight 也可以配置为自动创建主题。 有关详细信息，请参阅[配置自动主题创建](apache-kafka-auto-create-topics.md)文档。

## <a name="run-the-code"></a>运行代码

1. 若要将流式处理应用程序作为后台进程启动，请使用以下命令：

    ```bash
    java -jar kafka-streaming.jar $KAFKABROKERS $KAFKAZKHOSTS &
    ```

    > [!NOTE]  
    > 可能会收到有关 Apache log4j 的警告。 可将其忽略。

2. 若要将记录发送到 `test` 主题，请使用以下命令启动生成者应用程序：

    ```bash
    java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
    ```

3. 生成者完成后，使用以下命令查看 `wordcounts` 主题中存储的信息：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic wordcounts --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer --from-beginning
    ```

    > [!NOTE]  
    > `--property` 参数指示控制台 consumer 输出关键字（字）以及计数（值）。 此参数还配置从 Kafka 中读取这些值时要使用的反序列化程序。

    输出与以下文本类似：
   
        dwarfs  13635
        ago     13664
        snow    13636
        dwarfs  13636
        ago     13665
        a       13803
        ago     13666
        a       13804
        ago     13667
        ago     13668
        jumped  13640
        jumped  13641
   
    > [!NOTE]  
    > 参数 `--from-beginning` 将使用者配置为从主题中存储的记录开头启动。 每遇到一个字，计数都会递增，主题包含每个字的多个条目，因此计数不断增加。

4. 使用 __Ctrl + C__ 退出 producer。 继续使用 __Ctrl + C__ 退出应用程序和 consumer。

5. 若要删除流式处理操作所使用的主题，请使用以下命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

## <a name="next-steps"></a>后续步骤

本文档介绍了如何将 Apache Kafka Streams API 与 Kafka on HDInsight 配合使用。 使用以下内容，详细了解如何使用 Kafka：

* [分析 Apache Kafka 日志](apache-kafka-log-analytics-operations-management.md)
* [在 Apache Kafka 集群之间复制数据](apache-kafka-mirroring.md)
