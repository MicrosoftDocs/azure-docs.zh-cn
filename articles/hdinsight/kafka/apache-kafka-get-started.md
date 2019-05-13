---
title: 使用 Azure 门户设置 Apache Kafka on HDInsight - 快速入门
description: 在此快速入门中，了解如何在 Azure HDInsight 上使用 Azure 门户创建 Apache Kafka 群集。 还可以了解 Kafka 主题、订阅服务器和使用者。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.custom: mvc
ms.topic: quickstart
ms.date: 04/01/2019
ms.openlocfilehash: 8600e9c9d8566d3b9f6cd0fbca60d7db904a6265
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64687175"
---
# <a name="quickstart-create-an-apache-kafka-on-hdinsight-cluster"></a>快速入门：创建 Apache Kafka on HDInsight 群集

Apache Kafka 是开源分布式流式处理平台。 通常用作消息代理，因为它可提供类似于发布-订阅消息队列的功能。 

本快速入门介绍了如何使用 Azure 门户创建 [Apache Kafka](https://kafka.apache.org) 群集。 还介绍了如何使用已包含的实用程序发送并接收使用 Apache Kafka 的信息。

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

> [!IMPORTANT]  
> 仅可通过相同虚拟网络内的资源访问 Apache Kafka API。 本快速入门使用 SSH 直接访问群集。 若要将其他服务、网络或虚拟机连接到 Apache Kafka，则必须首先创建虚拟机，然后才能在网络中创建资源。
>
> 有关详细信息，请参阅[使用虚拟网络连接到 Apache Kafka](apache-kafka-connect-vpn-gateway.md) 文档。

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* SSH 客户端。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="create-an-apache-kafka-cluster"></a>创建 Apache Kafka 群集

若要创建 Apache Kafka on HDInsight 群集，请使用以下步骤：

1. 登录到 [Azure 门户](https://portal.azure.com)。

2. 在左侧菜单中，导航到“+ 创建资源” > “Analytics” > “HDInsight”。
   
    ![创建 HDInsight 群集](./media/apache-kafka-get-started/create-hdinsight.png)

3. 在“基本信息”中，输入或选择以下信息：

    | 设置 | 值 |
    | --- | --- |
    | 群集名称 | HDInsight Spark 群集的唯一名称。 |
    | 订阅 | 选择订阅。 |
    
   选择“群集类型”，以显示“群集配置”。
   
   ![基于 HDInsight 基本配置的 Apache Kafka 群集](./media/apache-kafka-get-started/hdinsight-basic-configuration-1.png)

4. 从“群集配置”中选择以下值：

    | 设置 | 值 |
    | --- | --- |
    | 群集类型 | Kafka |
    | 版本 | Kafka 1.1.0 (HDI 3.6) |

    选择“选择”以保存群集类型设置，然后返回到“基本信息”。

    ![选择群集类型](./media/apache-kafka-get-started/kafka-cluster-type.png)

5. 在“基本信息”中，输入或选择以下信息：

    | 设置 | 值 |
    | --- | --- |
    | 群集登录用户名 | 访问群集上托管的 Web 服务或 REST API 时的登录名。 保留默认值 (admin)。 |
    | 群集登录密码 | 访问群集上托管的 Web 服务或 REST API 时的登录密码。 |
    | 安全外壳 (SSH) 用户名 | 通过 SSH 访问群集时使用的登录名。 默认情况下，密码与群集登录密码相同。 |
    | 资源组 | 要在其中创建群集的资源组。 |
    | 位置 | 要在其中创建群集的 Azure 区域。 |

    > [!TIP]  
    > 每个 Azure 区域（位置）均提供_容错域_。 容错域是 Azure 数据中心基础硬件的逻辑分组。 每个容错域共享公用电源和网络交换机。 在 HDInsight 群集中实现节点的虚拟机和托管磁盘跨这些容错域分布。 此体系结构可限制物理硬件故障造成的潜在影响。
    >
    > 为实现数据的高可用性，请选择包含三个容错域的区域（位置）。 有关区域中容错域数的信息，请参阅 [Linux 虚拟机的可用性](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set)文档。

   ![选择订阅](./media/apache-kafka-get-started/hdinsight-basic-configuration-2.png)

    选择“下一步”，完成基本配置。

6. 对于本快速入门，请保留默认的安全设置。 若要详细了解企业安全性套餐，请访问[使用 Azure Active Directory 域服务配置具有企业安全性套餐的 HDInsight 群集](../domain-joined/apache-domain-joined-configure-using-azure-adds.md)。 若要了解如何将自己的密钥用于 Apache Kafka 磁盘加密，请访问[适用于 Apache Kafka on Azure HDInsight 的自带密钥](apache-kafka-byok.md)

   若要将群集连接到虚拟网络，请从“虚拟网络”下拉列表中选择一个虚拟网络。

   ![将群集添加到虚拟网络](./media/apache-kafka-get-started/kafka-security-config.png)

7. 在“存储”中选择或创建存储帐户。 对于本文档中的步骤，请让其他字段保留默认值。 使用“下一步”按钮保存存储配置。 有关使用 Data Lake Storage Gen2 的详细信息，请参阅[快速入门：在 HDInsight 中设置群集](../../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

   ![设置 HDInsight 的存储帐户设置](./media/apache-kafka-get-started/storage-configuration.png)

8. 在“应用程序(可选)”中，选择“下一步”以使用默认设置继续。

9. 在“群集大小”中，选择“下一步”以使用默认设置继续。

    > [!IMPORTANT]  
    > 若要确保 Apache Kafka on HDInsight 的可用性，辅助角色节点数条目必须设置为 3 或以上。 默认值为 4。
    
    > [!TIP]  
    > “每个工作节点的磁盘数”条目配置 Apache Kafka on HDInsight 的可伸缩性。 Apache Kafka on HDInsight 在群集中使用虚拟机的本地磁盘来存储数据。 由于 Apache Kafka 的 I/O 很高，因此会使用 [Azure 托管磁盘](../../virtual-machines/windows/managed-disks-overview.md)为每个节点提供高吞吐量和更多存储。 托管磁盘的类型可以为“标准”(HDD) 或“高级”(SSD)。 磁盘类型取决于辅助角色节点（Apache Kafka 代理）所使用的 VM 大小。 高级磁盘可自动与 DS 和 GS 系列 VM 一起使用。 所有其他的 VM 类型使用“标准”。

   ![设置 Apache Kafka 群集大小](./media/apache-kafka-get-started/kafka-cluster-size.png)

10. 在“高级设置”中，选择“下一步”以使用默认设置继续。

11. 在“摘要”中，查看群集的配置。 使用“编辑”链接更改不正确的设置。 最后，选择“创建”以创建群集。
   
    ![群集配置摘要](./media/apache-kafka-get-started/kafka-configuration-summary.png)
   
    > [!NOTE]
    > 创建群集可能需要 20 分钟。

## <a name="connect-to-the-cluster"></a>连接至群集

1. 若要连接到 Apache Kafka 群集的主要头节点，请使用以下命令。 将 `sshuser` 替换为 SSH 用户名。 将 `mykafka` 替换为 Apache Kafka 群集的名称。

    ```bash
    ssh sshuser@mykafka-ssh.azurehdinsight.net
    ```

2. 首次连接到群集时，SSH 客户端可能会显示一个警告，提示无法验证主机。 当系统提示时，请键入“yes”，然后按 Enter，将主机添加到 SSH 客户端的受信任服务器列表。

3. 出现提示时，请输入 SSH 用户名密码。

    连接后，显示的信息类似于以下文本：
    
    ```text
    Authorized uses only. All activity may be monitored and reported.
    Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1011-azure x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      Get cloud support with Ubuntu Advantage Cloud Guest:
        https://www.ubuntu.com/business/services/cloud
    
    83 packages can be updated.
    37 updates are security updates.


    Welcome to Apache Kafka on HDInsight.
    
    Last login: Thu Mar 29 13:25:27 2018 from 108.252.109.241
    ssuhuser@hn0-mykafk:~$
    ```

## <a id="getkafkainfo"></a>获取 Apache Zookeeper 主机和代理主机信息

使用 Kafka 时，必须了解 Apache Zookeeper 和代理主机。 这些主机配合 Apache Kafka API 和 Kafka 随附的许多实用程序一起使用。

在本部分中，可以从群集上的 Apache Ambari REST API 获取主机信息。

1. 安装 [jq](https://stedolan.github.io/jq/)，一个命令行 JSON 处理程序。 此实用程序用于分析 JSON 文档和主机信息。 在打开的 SSH 连接中，输入以下命令以安装 `jq`：
   
    ```bash
    sudo apt -y install jq
    ```

2. 设置环境变量。 将 `PASSWORD` 和 `CLUSTERNAME` 分别替换为群集登录密码和群集名称，然后输入命令：

    ```bash
    export password='PASSWORD'
    export clusterNameA='CLUSTERNAME'
    ```

3. 提取具有正确大小写格式的群集名称。 群集名称的实际大小写格式可能出乎预期，具体取决于群集的创建方式。 此命令会获取实际的大小写格式，将其存储在变量中，然后显示具有正确大小写格式的名称，以及此前提供的名称。 输入以下命令：

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "https://$clusterNameA.azurehdinsight.net/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
    echo $clusterName, $clusterNameA
    ```

4. 若要使用 Zookeeper 主机信息来设置环境变量，请使用以下命令：
    
    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    > [!TIP]  
    > 此命令直接查询群集头节点上的 Ambari 服务。 也可以使用公共地址 `https://$CLUSTERNAME.azurehdinsight.net:80/` 访问 Ambari。 某些网络配置可以阻止访问公用地址。 例如，使用网络安全组 (NSG) 限制对虚拟网络中的 HDInsight 的访问。

    > [!NOTE]  
    > 此命令检索所有 Zookeeper 主机，然后仅返回前两个条目。 这是由于某个主机无法访问时，需要一些冗余。

5. 若要验证是否已正确设置了环境变量，请使用以下命令：

    ```bash
    echo $KAFKAZKHOSTS
    ```

    此命令返回类似于以下文本的信息：

    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`

6. 若要使用 Apache Kafka 代理主机信息来设置环境变量，请使用以下命令：

    ```bash
    export KAFKABROKERS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

7. 若要验证是否已正确设置了环境变量，请使用以下命令：

    ```bash   
    echo $KAFKABROKERS
    ```

    此命令返回类似于以下文本的信息：
   
    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

## <a name="manage-apache-kafka-topics"></a>管理 Apache Kafka 主题

Kafka 在主题中存储数据流。 可以使用 `kafka-topics.sh` 实用工具来管理主题。

* 若要创建主题，请在 SSH 连接中使用以下命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    ```

    此命令使用存储在 `$KAFKAZKHOSTS` 中的主机信息连接到 Zookeeper， 然后创建名为 **test** 的 Apache Kafka 主题。 

    * 本主题中存储的数据已分区到八个分区。

    * 每个分区在群集中的三个辅助角色节点上进行复制。

        > [!IMPORTANT]  
        > 如果在 Azure 区域中已创建提供三个容错域的群集，则复制因子使用 3。 否则，复制因子使用 4.
        
        在具有三个容错域的区域中，复制因子为 3 可让副本分布在容错域中。 在具有两个容错域的区域中，复制因子为 4 可将副本均匀分布在域中。
        
        有关区域中容错域数的信息，请参阅 [Linux 虚拟机的可用性](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set)文档。

        > [!IMPORTANT]  
        > Apache Kafka 不识别 Azure 容错域。 在创建主题的分区副本时，它可能未针对高可用性正确分发副本。

        若要确保高可用性，请使用 [Apache Kafka 分区重新均衡工具](https://github.com/hdinsight/hdinsight-kafka-tools)。 必须通过 SSH 连接运行此工具，以便连接到 Apache Kafka 群集的头节点。

        为确保 Apache Kafka 数据的最高可用性，应在出现以下情况时为主题重新均衡分区副本：

        * 创建新主题或分区

        * 纵向扩展群集

* 若要列出主题，请使用以下命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    此命令列出 Apache Kafka 群集上可用的主题。

* 若要删除主题，使用以下命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic topicname --zookeeper $KAFKAZKHOSTS
    ```

    此命令删除名为 `topicname` 的主题。

    > [!WARNING]  
    > 如果删除了之前创建的 `test` 主题，则必须重新创建。 稍后会在本文档中使用此主题。

有关适用于 `kafka-topics.sh` 实用工具的命令的详细信息，请使用以下命令：

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh
```

## <a name="produce-and-consume-records"></a>生成和使用记录

Kafka 将记录存储在主题中。 记录由生成者生成，由使用者使用。 生产者与使用者通过 Kafka 代理服务通信。 HDInsight 群集中的每个工作节点都是 Apache Kafka 代理主机。

若要将记录存储到之前创建的测试主题，并通过使用者对其进行读取，请使用以下步骤：

1. 若要为该主题写入记录，请从 SSH 连接使用 `kafka-console-producer.sh` 实用工具：
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```
   
    此命令之后是一个空行。

2. 在空行中键入文本消息，然后点击 Enter。 以这种方式输入一些消息，然后使用 **Ctrl + C** 返回到正常的提示符处。 每行均作为单独的记录发送到 Apache Kafka 主题。

3. 若要读取该主题的记录，请从 SSH 连接使用 `kafka-console-consumer.sh` 实用工具：
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```
   
    此命令从主题中检索并显示记录。 使用 `--from-beginning` 告知使用者从流的开头开始，以检索所有记录。

    > [!NOTE]
    > 如果使用的是较旧版本的 Kafka，请将 `--bootstrap-server $KAFKABROKERS` 替换为 `--zookeeper $KAFKAZKHOSTS`。

4. 使用 __Ctrl + C__ 阻止使用者。

还可以以编程方式创建生产者和使用者。 有关如何使用此 API 的示例，请参阅[将 Apache Kafka 生产者和使用者 API 与 HDInsight 配合使用](apache-kafka-producer-consumer-api.md)文档。

## <a name="clean-up-resources"></a>清理资源

若要清理本快速入门创建的资源，可以删除资源组。 删除资源组也会删除相关联的 HDInsight 群集，以及与资源组相关联的任何其他资源。

若要使用 Azure 门户删除资源组，请执行以下操作：

1. 在 Azure 门户中展开左侧的菜单，打开服务菜单，然后选择“资源组”以显示资源组的列表。
2. 找到要删除的资源组，然后右键单击列表右侧的“更多”按钮 (...)。
3. 选择“删除资源组”，然后进行确认。

> [!WARNING]  
> 创建群集后便开始 HDInsight 群集计费，删除群集后停止计费。 群集以每分钟按比例收费，因此无需再使用群集时，应始终将其删除。
> 
> 删除 Apache Kafka on HDInsight 群集会删除存储在 Kafka 中的任何数据。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [将 Apache Spark 与 Apache Kafka 配合使用](../hdinsight-apache-kafka-spark-structured-streaming.md)
