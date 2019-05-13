---
title: Apache Storm 示例 Java 拓扑 - Azure HDInsight
description: 了解如何通过创建一个简单的单词计数拓扑，来以 Java 语言创建 Apache Storm 拓扑。
author: hrasheed-msft
ms.reviewer: jasonh
keywords: apache storm,apache storm 示例,storm java,storm 拓扑示例
ms.service: hdinsight
ms.topic: conceptual
ms.date: 03/14/2019
ms.author: hrasheed
ms.custom: H1Hack27Feb2017,hdinsightactive,hdiseo17may2017
ms.openlocfilehash: 43f68908c8549c2f1d8322b5c4ad3985618cfe6e
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64695641"
---
# <a name="create-an-apache-storm-topology-in-java"></a>以 Java 语言创建 Apache Storm 拓扑

了解如何为 [Apache Storm](https://storm.apache.org/) 创建基于 Java 的拓扑。 在此处，我们将创建一个实现单词计数应用程序的 Storm 拓扑。 将使用 [Apache Maven](https://maven.apache.org/) 构建并打包项目。 然后，了解如何使用 [Apache Storm Flux](https://storm.apache.org/releases/2.0.0-SNAPSHOT/flux.html) 框架定义拓扑。

完成本文档中的步骤之后，可将拓扑部署到 Apache Storm on HDInsight。

> [!NOTE]  
> [https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) 中提供了本文档中创建的 Storm 拓扑示例的完整版本。

## <a name="prerequisites"></a>必备组件

* [Java 开发人员工具包 (JDK) 版本 8](https://aka.ms/azure-jdks)

* 根据 Apache 要求正确[安装](https://maven.apache.org/install.html)的 [Apache Maven](https://maven.apache.org/download.cgi)。  Maven 是 Java 项目的项目生成系统。

## <a name="test-environment"></a>测试环境
本文使用的环境是一台运行 Windows 10 的计算机。  命令在命令提示符下执行，各种文件使用记事本进行编辑。

在命令提示符下，输入以下命令以创建工作环境：

```cmd
mkdir C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>创建 Maven 项目

输入以下命令，创建名为 **WordCount** 的 Maven 项目：

```cmd
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

cd WordCount
mkdir resources
```

此命令会在当前位置创建名为 `WordCount` 的目录，其中包含基本 Maven 项目。 第二条命令将现有工作目录更改为 `WordCount`。 第三条命令创建稍后要使用的新目录 `resources`。  `WordCount` 目录包含以下项：

* `pom.xml`：包含 Maven 项目的设置。
* `src\main\java\com\microsoft\example`：包含应用程序代码。
* `src\test\java\com\microsoft\example`：包含应用程序的测试。  

### <a name="remove-the-generated-example-code"></a>删除生成的示例代码

输入以下命令，删除生成的测试和应用程序文件 `AppTest.java` 与 `App.java`：

```cmd
DEL src\main\java\com\microsoft\example\App.java
DEL src\test\java\com\microsoft\example\AppTest.java
```

## <a name="add-maven-repositories"></a>添加 Maven 存储库

由于 HDInsight 基于 Hortonworks Data Platform (HDP)，因此我们建议使用 Hortonworks 存储库来下载 Apache Storm 项目的依赖项。  

输入以下命令打开 `pom.xml`：

```cmd
notepad pom.xml
```

然后，在 `<url> https://maven.apache.org</url>` 行的后面添加以下 XML：

```xml
<repositories>
    <repository>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
        <id>HDPReleases</id>
        <name>HDP Releases</name>
        <url>https://repo.hortonworks.com/content/repositories/releases/</url>
        <layout>default</layout>
    </repository>
    <repository>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
        <id>HDPJetty</id>
        <name>Hadoop Jetty</name>
        <url>https://repo.hortonworks.com/content/repositories/jetty-hadoop/</url>
        <layout>default</layout>
    </repository>
</repositories>
```

## <a name="add-properties"></a>添加属性

Maven 允许定义项目级的值，称为属性。 在 `pom.xml` 中的 `</repositories>` 行后面添加以下文本：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!--
    This is a version of Storm from the Hortonworks repository that is compatible with HDInsight 3.6.
    -->
    <storm.version>1.1.0.2.6.1.9-1</storm.version>
</properties>
```

现在，可以在 `pom.xml` 的其他部分中使用此值。 例如，在指定 Storm 组件的版本时，可以使用 `${storm.version}` 而无需将值硬编码。

## <a name="add-dependencies"></a>添加依赖项

添加 Storm 组件的依赖项。 在 `pom.xml` 的 `<dependencies>` 节中添加以下文本：

```xml
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>${storm.version}</version>
    <!-- keep storm out of the jar-with-dependencies -->
    <scope>provided</scope>
</dependency>
```

在编译时，Maven 会使用此信息在 Maven 存储库中查找 `storm-core`。 它会先查找本地计算机上的存储库。 如果文件不存在，Maven 会从公共 Maven 存储库下载这些文件，并将其存储在本地存储库中。

> [!NOTE]  
> 请注意该部分中的 `<scope>provided</scope>` 行。 此设置告诉 Maven 从创建的任何 JAR 文件中排除 **storm-core**，因为它由系统提供。

## <a name="build-configuration"></a>生成配置

Maven 插件可用于自定义项目的生成阶段。 例如，如何编译项目或者如何将其打包到 JAR 文件中。 在 `pom.xml` 中，紧靠在 `</project>` 行的上面添加以下文本：

```xml
<build>
    <plugins>
    </plugins>
    <resources>
    </resources>
</build>
```

此节用于添加插件、资源和其他生成配置选项。 有关 `pom.xml` 文件的完整参考，请参阅 [https://maven.apache.org/pom.html](https://maven.apache.org/pom.html)。

### <a name="add-plug-ins"></a>添加插件

* **Exec Maven 插件**

    对于以 Java 语言实现的 Apache Storm 拓扑，[Exec Maven 插件](https://www.mojohaus.org/exec-maven-plugin/)十分有用，因为它可让你轻松地在开发环境本地运行拓扑。 在 `pom.xml` 文件的 `<plugins>` 部分中添加以下内容，以包括 Exec Maven 插件：
    
    ```xml
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.6.0</version>
        <executions>
            <execution>
            <goals>
                <goal>exec</goal>
            </goals>
            </execution>
        </executions>
        <configuration>
            <executable>java</executable>
            <includeProjectDependencies>true</includeProjectDependencies>
            <includePluginDependencies>false</includePluginDependencies>
            <classpathScope>compile</classpathScope>
            <mainClass>${storm.topology}</mainClass>
            <cleanupDaemonThreads>false</cleanupDaemonThreads> 
        </configuration>
    </plugin>
    ```

* **Apache Maven Compiler Plugin**

    另一个有用的插件是用于更改编译选项的 [Apache Maven Compiler 插件](https://maven.apache.org/plugins/maven-compiler-plugin/)。 更改 Maven 用作应用程序源和目标的 Java 版本。
    
  * 对于 __HDInsight 3.4 或更早的版本__，请将源和目标 Java 版本设置为 __1.7__。
    
  * 对于 HDInsight __3.5__，请将源和目标 Java 版本设置为 __1.8__。
    
    在 `pom.xml` 文件的 `<plugins>` 部分添加以下文本，以包括 Apache Maven Compiler 插件。 此示例指定 1.8，因此目标 HDInsight 版本为 3.5。
    
    ```xml
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
      <source>1.8</source>
      <target>1.8</target>
      </configuration>
    </plugin>
    ```

### <a name="configure-resources"></a>配置资源

使用 resources 节可以包含非代码资源，例如拓扑中组件所需的配置文件。 本示例在 `pom.xml` 文件的 `<resources>` 节中添加以下文本。

```xml
<resource>
    <directory>${basedir}/resources</directory>
    <filtering>false</filtering>
    <includes>
        <include>log4j2.xml</include>
    </includes>
</resource>
```

本示例会将项目根目录 (`${basedir}`) 中的 resources 目录添加为包含资源的位置，并包含名为 `log4j2.xml` 的文件。 此文件用于配置拓扑所要记录的信息。

## <a name="create-the-topology"></a>创建拓扑

基于 Java 的 Apache Storm 拓扑包含必须编写（或引用）为依赖项的三个组件。

* **Spout**：读取外部源中的数据，并发出进入拓扑的数据流。

* **Bolt**：对 Spout 或其他 Bolt 所发出的流进行处理，并发出一个或多个流。

* **拓扑**：定义如何排列 Spout 和 Bolt，并为拓扑提供入口点。

### <a name="create-the-spout"></a>创建 Spout

为了降低设置外部数据源的要求，以下 Spout 只会发出随机句子。 它是 [Storm-Starter 示例](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter)随附的 Spout 的修改版本。  虽然此拓扑只使用一个 Spout，但其他拓扑可能存在将数据从不同源送入拓扑的多个 Spout。

输入以下命令，以创建并打开新文件 `RandomSentenceSpout.java`：

```cmd
notepad src\main\java\com\microsoft\example\RandomSentenceSpout.java
```

将以下 Java 代码复制并粘贴到新文件中。  然后关闭该文件。

```java
package com.microsoft.example;

import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;
import org.apache.storm.utils.Utils;

import java.util.Map;
import java.util.Random;

//This spout randomly emits sentences
public class RandomSentenceSpout extends BaseRichSpout {
  //Collector used to emit output
  SpoutOutputCollector _collector;
  //Used to generate a random number
  Random _rand;

  //Open is called when an instance of the class is created
  @Override
  public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
  //Set the instance collector to the one passed in
    _collector = collector;
    //For randomness
    _rand = new Random();
  }

  //Emit data to the stream
  @Override
  public void nextTuple() {
  //Sleep for a bit
    Utils.sleep(100);
    //The sentences that are randomly emitted
    String[] sentences = new String[]{ "the cow jumped over the moon", "an apple a day keeps the doctor away",
        "four score and seven years ago", "snow white and the seven dwarfs", "i am at two with nature" };
    //Randomly pick a sentence
    String sentence = sentences[_rand.nextInt(sentences.length)];
    //Emit the sentence
    _collector.emit(new Values(sentence));
  }

  //Ack is not implemented since this is a basic example
  @Override
  public void ack(Object id) {
  }

  //Fail is not implemented since this is a basic example
  @Override
  public void fail(Object id) {
  }

  //Declare the output fields. In this case, an sentence
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("sentence"));
  }
}
```

> [!NOTE]  
> 有关从外部数据源读取的 Spout 的示例，请参阅以下示例之一：
>
> * [TwitterSampleSpout](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter/spout/TwitterSampleSpout.java)：从Twitter 读取数据的示例 Spout。
> * [Storm-Kafka](https://github.com/apache/storm/tree/0.10.x-branch/external/storm-kafka)：从 Kafka 读取数据的 Spout。


### <a name="create-the-bolts"></a>创建 Bolt

Bolt 用于处理数据。 Bolt 可以执行任何操作，例如，计算、保存，或者与外部组件通信。 此拓扑使用两个 Bolt：

* **SplitSentence**：将 RandomSentenceSpout 发出的句子拆分成不同的单词。

* **WordCount**：统计每个单词的出现次数。


#### <a name="splitsentence"></a>SplitSentence

输入以下命令，以创建并打开新文件 `SplitSentence.java`：

```cmd
notepad src\main\java\com\microsoft\example\SplitSentence.java
```

将以下 Java 代码复制并粘贴到新文件中。  然后关闭该文件。

```java
package com.microsoft.example;

import java.text.BreakIterator;

import org.apache.storm.topology.BasicOutputCollector;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

//There are a variety of bolt types. In this case, use BaseBasicBolt
public class SplitSentence extends BaseBasicBolt {

  //Execute is called to process tuples
  @Override
  public void execute(Tuple tuple, BasicOutputCollector collector) {
    //Get the sentence content from the tuple
    String sentence = tuple.getString(0);
    //An iterator to get each word
    BreakIterator boundary=BreakIterator.getWordInstance();
    //Give the iterator the sentence
    boundary.setText(sentence);
    //Find the beginning first word
    int start=boundary.first();
    //Iterate over each word and emit it to the output stream
    for (int end=boundary.next(); end != BreakIterator.DONE; start=end, end=boundary.next()) {
      //get the word
      String word=sentence.substring(start,end);
      //If a word is whitespace characters, replace it with empty
      word=word.replaceAll("\\s+","");
      //if it's an actual word, emit it
      if (!word.equals("")) {
        collector.emit(new Values(word));
      }
    }
  }

  //Declare that emitted tuples contain a word field
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("word"));
  }
}
```

#### <a name="wordcount"></a>WordCount

输入以下命令，以创建并打开新文件 `WordCount.java`：

```cmd
notepad src\main\java\com\microsoft\example\WordCount.java
```

将以下 Java 代码复制并粘贴到新文件中。  然后关闭该文件。

```java
package com.microsoft.example;

import java.util.HashMap;
import java.util.Map;
import java.util.Iterator;

import org.apache.storm.Constants;
import org.apache.storm.topology.BasicOutputCollector;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;
import org.apache.storm.Config;

// For logging
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

//There are a variety of bolt types. In this case, use BaseBasicBolt
public class WordCount extends BaseBasicBolt {
  //Create logger for this class
  private static final Logger logger = LogManager.getLogger(WordCount.class);
  //For holding words and counts
  Map<String, Integer> counts = new HashMap<String, Integer>();
  //How often to emit a count of words
  private Integer emitFrequency;

  // Default constructor
  public WordCount() {
      emitFrequency=5; // Default to 60 seconds
  }

  // Constructor that sets emit frequency
  public WordCount(Integer frequency) {
      emitFrequency=frequency;
  }

  //Configure frequency of tick tuples for this bolt
  //This delivers a 'tick' tuple on a specific interval,
  //which is used to trigger certain actions
  @Override
  public Map<String, Object> getComponentConfiguration() {
      Config conf = new Config();
      conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, emitFrequency);
      return conf;
  }

  //execute is called to process tuples
  @Override
  public void execute(Tuple tuple, BasicOutputCollector collector) {
    //If it's a tick tuple, emit all words and counts
    if(tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
            && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)) {
      for(String word : counts.keySet()) {
        Integer count = counts.get(word);
        collector.emit(new Values(word, count));
        logger.info("Emitting a count of " + count + " for word " + word);
      }
    } else {
      //Get the word contents from the tuple
      String word = tuple.getString(0);
      //Have we counted any already?
      Integer count = counts.get(word);
      if (count == null)
        count = 0;
      //Increment the count and store it
      count++;
      counts.put(word, count);
    }
  }

  //Declare that this emits a tuple containing two fields; word and count
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("word", "count"));
  }
}
```

### <a name="define-the-topology"></a>定义拓扑

拓扑将 Spout 和 Bolt 一起绑定到图形，该图形定义了组件之间的数据流动方式。 它还提供 Storm 在群集内创建组件的实例时使用的并行度提示。

下图是此拓扑的组件的基本原理图。

![显示 Spout 和 Bolt 排列方式的示意图](./media/apache-storm-develop-java-topology/wordcount-topology.png)

若要实现该拓扑，请输入以下命令，以创建并打开新文件 `WordCountTopology.java`：

```cmd
notepad src\main\java\com\microsoft\example\WordCountTopology.java
```

将以下 Java 代码复制并粘贴到新文件中。  然后关闭该文件。

```java
package com.microsoft.example;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;

import com.microsoft.example.RandomSentenceSpout;

public class WordCountTopology {

  //Entry point for the topology
  public static void main(String[] args) throws Exception {
  //Used to build the topology
    TopologyBuilder builder = new TopologyBuilder();
    //Add the spout, with a name of 'spout'
    //and parallelism hint of 5 executors
    builder.setSpout("spout", new RandomSentenceSpout(), 5);
    //Add the SplitSentence bolt, with a name of 'split'
    //and parallelism hint of 8 executors
    //shufflegrouping subscribes to the spout, and equally distributes
    //tuples (sentences) across instances of the SplitSentence bolt
    builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");
    //Add the counter, with a name of 'count'
    //and parallelism hint of 12 executors
    //fieldsgrouping subscribes to the split bolt, and
    //ensures that the same word is sent to the same instance (group by field 'word')
    builder.setBolt("count", new WordCount(), 12).fieldsGrouping("split", new Fields("word"));

    //new configuration
    Config conf = new Config();
    //Set to false to disable debug information when
    // running in production on a cluster
    conf.setDebug(false);

    //If there are arguments, we are running on a cluster
    if (args != null && args.length > 0) {
      //parallelism hint to set the number of workers
      conf.setNumWorkers(3);
      //submit the topology
      StormSubmitter.submitTopology(args[0], conf, builder.createTopology());
    }
    //Otherwise, we are running locally
    else {
      //Cap the maximum number of executors that can be spawned
      //for a component to 3
      conf.setMaxTaskParallelism(3);
      //LocalCluster is used to run locally
      LocalCluster cluster = new LocalCluster();
      //submit the topology
      cluster.submitTopology("word-count", conf, builder.createTopology());
      //sleep
      Thread.sleep(10000);
      //shut down the cluster
      cluster.shutdown();
    }
  }
}
```

### <a name="configure-logging"></a>配置日志记录

Storm 使用 [Apache Log4j 2](https://logging.apache.org/log4j/2.x/) 来记录信息。 如果未配置日志记录，拓扑将发出诊断信息。 若要控制所要记录的内容，请输入以下命令，在 `resources` 目录中创建名为 `log4j2.xml` 的文件：

```cmd
notepad resources\log4j2.xml
```

将以下 XML 文本复制并粘贴到新文件中。  然后关闭该文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
<Appenders>
    <Console name="STDOUT" target="SYSTEM_OUT">
        <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
</Appenders>
<Loggers>
    <Logger name="com.microsoft.example" level="trace" additivity="false">
        <AppenderRef ref="STDOUT"/>
    </Logger>
    <Root level="error">
        <Appender-Ref ref="STDOUT"/>
    </Root>
</Loggers>
</Configuration>
```

此 XML 为 `com.microsoft.example` 类（其中包含本示例拓扑中的组件）配置一个新记录器。 此记录器的级别设置为“跟踪”，可以捕获此拓扑中的组件发出的任何日志记录信息。

`<Root level="error">` 部分将日志记录的根级别（不在 `com.microsoft.example` 中的所有内容）配置为只记录错误信息。

有关为 Log4j 2 配置日志记录的详细信息，请参阅 [https://logging.apache.org/log4j/2.x/manual/configuration.html](https://logging.apache.org/log4j/2.x/manual/configuration.html)。

> [!NOTE]  
> Storm 0.10.0 版及更高版本使用 Log4j 2.x。 早期版本的 Storm 使用 Log4j 1.x（为日志配置使用的格式不同）。 有关较旧配置的信息，请参阅 [https://wiki.apache.org/logging-log4j/Log4jXmlFormat](https://wiki.apache.org/logging-log4j/Log4jXmlFormat)。

## <a name="test-the-topology-locally"></a>在本地测试拓扑

保存文件之后，请使用以下命令在本地测试拓扑。

```cmd
mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology
```

运行该命令时，拓扑显示启动信息。 以下文本是单词计数输出的示例：

    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
    17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word snow

此示例日志指示单词 'and' 已发出了 113 次。 只要拓扑运行，计数就会持续增加，因为 Spout 会连续发出相同的句子。

每两次发出单词和句子的间隔为 5 秒。 **WordCount** 组件配置为仅当 tick 元组到达时才发出信息。 它要求每五秒钟传送一次 tick 元组。

## <a name="convert-the-topology-to-flux"></a>将拓扑转换为 Flux

[Flux](https://storm.apache.org/releases/2.0.0-SNAPSHOT/flux.html) 是 Storm 0.10.0 及更高版本随附的一个新框架，可以将配置和实现分离开来。 组件仍然是以 Java 语言定义的，但拓扑是使用 YAML 文件定义的。 可以随项目一起打包默认的拓扑定义，也可以在提交拓扑时使用独立的文件。 将拓扑提交到 Storm 时，可以使用环境变量或配置文件来填充 YAML 拓扑定义中的值。

YAML 文件定义了要用于拓扑的组件以及它们之间的数据流。 可以包括一个 YAML 文件（作为 jar 文件的一部分），也可以使用外部 YAML 文件。

有关 Flux 的详细信息，请参阅 [Flux 框架 (https://storm.apache.org/releases/1.0.6/flux.html)](https://storm.apache.org/releases/1.0.6/flux.html)。

> [!WARNING]  
> 由于 Storm 1.0.1 的一个 [bug (https://issues.apache.org/jira/browse/STORM-2055)](https://issues.apache.org/jira/browse/STORM-2055)，可能需要安装 [Storm 开发环境](https://storm.apache.org/releases/current/Setting-up-development-environment.html)才能在本地运行 Flux 拓扑。

1. 以前，`WordCountTopology.java` 会定义拓扑，但使用 Flux 时无需这样做。 使用以下命令删除该文件：

    ```cmd
    DEL src\main\java\com\microsoft\example\WordCountTopology.java
    ```

2. 输入以下命令，以创建并打开新文件 `topology.yaml`：

    ```cmd
    notepad resources\topology.yaml
    ```

    将以下文本复制并粘贴到新文件中。  然后关闭该文件。

    ```yaml
    name: "wordcount"       # friendly name for the topology

    config:                 # Topology configuration
      topology.workers: 1     # Hint for the number of workers to create

    spouts:                 # Spout definitions
    - id: "sentence-spout"
      className: "com.microsoft.example.RandomSentenceSpout"
      parallelism: 1      # parallelism hint

    bolts:                  # Bolt definitions
    - id: "splitter-bolt"
      className: "com.microsoft.example.SplitSentence"
      parallelism: 1
        
    - id: "counter-bolt"
      className: "com.microsoft.example.WordCount"
      constructorArgs:
        - 10
      parallelism: 1

    streams:                # Stream definitions
    - name: "Spout --> Splitter" # name isn't used (placeholder for logging, UI, etc.)
      from: "sentence-spout"       # The stream emitter
      to: "splitter-bolt"          # The stream consumer
      grouping:                    # Grouping type
        type: SHUFFLE
    
    - name: "Splitter -> Counter"
      from: "splitter-bolt"
      to: "counter-bolt"
      grouping:
        type: FIELDS
        args: ["word"]           # field(s) to group on
    ```

3. 输入以下命令打开 `pom.xml`，并做出下面所述的修改：

    ```cmd
    notepad pom.xml
    ```

   * 在 `<dependencies>` 节中添加以下新依赖关系：

        ```xml
        <!-- Add a dependency on the Flux framework -->
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>flux-core</artifactId>
            <version>${storm.version}</version>
        </dependency>
        ```

   * 将以下插件添加到 `<plugins>` 节。 此插件处理项目包（jar 文件）的创建，并在创建包时应用一些特定于 Flux 的转换。

        ```xml
        <!-- build an uber jar -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.1</version>
            <configuration>
                <transformers>
                    <!-- Keep us from getting a "can't overwrite file error" -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                    <!-- We're using Flux, so refer to it as main -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>org.apache.storm.flux.Flux</mainClass>
                    </transformer>
                </transformers>
                <!-- Keep us from getting a bad signature error -->
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        ```

   * 在 **exec-maven-plugin** `<configuration>` 节中，将 `<mainClass>` 的值从 `${storm.topology}` 更改为 `org.apache.storm.flux.Flux`。 在开发环境中本地运行拓扑时，Flux 可以使用此设置处理拓扑运行。

   * 将以下内容添加到 `<resources>` 节中的 `<includes>`。 此 XML 包括了将拓扑定义为项目一部分的 YAML 文件。

        ```xml
        <include>topology.yaml</include>
        ```

## <a name="test-the-flux-topology-locally"></a>在本地测试 Flux 拓扑

1. 输入以下命令，以使用 Maven 编译并执行 Flux 拓扑：

    ```cmd
    mvn compile exec:java -Dexec.args="--local -R /topology.yaml"
    ```

    > [!WARNING]  
    > 如果拓扑使用 Storm 1.0.1 位，此命令会失败。 此失败是由 [https://issues.apache.org/jira/browse/STORM-2055](https://issues.apache.org/jira/browse/STORM-2055) 导致的。 相反，[在开发环境中安装 Storm](https://storm.apache.org/releases/current/Setting-up-development-environment.html)，并按照以下步骤操作：
    >
    > 如果已[在开发环境中安装 Storm](https://storm.apache.org/releases/current/Setting-up-development-environment.html)，则可以改用以下命令：
    >
    > ```cmd
    > mvn compile package
    > storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /topology.yaml
    > ```

    `--local` 参数在开发环境中以本地模式运行拓扑。 `-R /topology.yaml` 参数使用 jar 文件中的 `topology.yaml` 文件资源来定义拓扑。

    运行该命令时，拓扑显示启动信息。 以下文本是输出的示例：

        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
        17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs

    不同批次的记录信息之间存在 10 秒的延迟。

2. 基于项目创建新的拓扑 yaml。
 
    a. 输入以下命令打开 `topology.xml`：

    ```cmd
    notepad resources\topology.yaml
    ```

    b. 找到以下节，将 `10` 的值更改为 `5`。 此修改会将发出单词计数批的间隔时间从 10 秒更改为 5 秒。  

    ```yaml
    - id: "counter-bolt"
      className: "com.microsoft.example.WordCount"
      constructorArgs:
        - 5
      parallelism: 1  
    ```  

    c. 将文件另存为 `newtopology.yaml`。

3. 若要运行拓扑，请输入以下命令：

    ```cmd
    mvn exec:java -Dexec.args="--local resources/newtopology.yaml"
    ```

    或者，如果开发环境中有 Storm，则执行以下操作：

    ```cmd
    storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local resources/newtopology.yaml
    ```

     此命令使用 `newtopology.yaml` 作为拓扑定义。 由于没有包含 `compile` 参数，Maven 使用前面步骤中生成的项目的版本。

    启动拓扑后，你将发现，发出批的间隔时间已更改，会反映 `newtopology.yaml` 中的值。 因此可以看到，无需重新编译拓扑即可通过 YAML 文件更改配置。

有关 Flux 框架的上述功能和其他功能的详细信息，请参阅 [Flux (https://storm.apache.org/releases/current/flux.html)](https://storm.apache.org/releases/current/flux.html)。

## <a name="trident"></a>Trident

[Trident](https://storm.apache.org/releases/current/Trident-API-Overview.html) 是 Storm 提供的高级抽象。 它支持有状态处理。 Trident 的主要优点在于，它可以保证进入拓扑的每个消息只会处理一次。 如果不使用 Trident，则拓扑只能保证至少将消息处理一次。 两者还有其他方面的差异，例如，可以使用内置组件，而无需创建 Bolt。 事实上，可以使用低泛型组件（例如筛选、投影和函数）来取代 Bolt。

可以使用 Maven 项目来创建 Trident 应用程序。 使用本文前面所述的相同基本步骤 - 只有代码不同。 Trident（目前）还不能与 Flux 框架配合使用。

有关 Trident 的详细信息，请参阅 [Trident API 概述](https://storm.apache.org/releases/current/Trident-API-Overview.html)。

## <a name="next-steps"></a>后续步骤

已学习如何使用 Java 创建 Apache Storm 拓扑。 接下来，请学习如何：

* [在 HDInsight 上部署和管理 Apache Storm 拓扑](apache-storm-deploy-monitor-topology-linux.md)

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](apache-storm-develop-csharp-visual-studio-topology.md)

如需更多 Apache Storm 拓扑示例，请访问 [Apache Storm on HDInsight 示例拓扑](apache-storm-example-topology.md)。
