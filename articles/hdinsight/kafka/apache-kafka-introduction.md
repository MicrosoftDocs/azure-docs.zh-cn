---
title: Apache Kafka on HDInsight 简介 - Azure
description: 了解 Apache Kafka on HDInsight：了解它的涵义和用途以及在何处可找到示例和入门信息。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: overview
ms.date: 04/11/2018
ms.openlocfilehash: 50a67ca5c7d1fa3165889b1e59565d1a1030690d
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64712140"
---
# <a name="what-is-apache-kafka-on-hdinsight"></a>什么是 Apache Kafka on HDInsight？

[Apache Kafka](https://kafka.apache.org) 是一个分布式流式处理平台，以开源方式提供，可用于构建实时流式处理数据管道和应用程序。 Kafka 还提供了类似于消息队列的消息中转站，可在其中发布和订阅命名数据流。 

Kafka on HDInsight 的具体特征如下：

* 它是一种托管服务，可提供简化的配置过程。 其结果是经 Microsoft 测试并支持的配置。

* Microsoft 就 Kafka 正常运行时间提供 99.9 % 的服务级别协议(SLA)。 有关详细信息，请参阅 [HDInsight 的 SLA 信息](https://azure.microsoft.com/support/legal/sla/hdinsight/v1_0/)文档。

* 它使用 Azure 托管磁盘作为 Kafka 的后备存储。 托管磁盘可为每个 Kafka 代理提供高达 16 TB 的存储空间。 有关为 Kafka on HDInsight 配置托管磁盘的信息，请参阅[提高 Apache Kafka on HDInsight 的可伸缩性](apache-kafka-scalability.md)。

    有关托管磁盘的详细信息，请参阅 [Azure 托管磁盘](../../virtual-machines/windows/managed-disks-overview.md)。

* Kafka 采用一维机架视图设计。 Azure 将机架分为两个维度，即更新域 (UD) 和容错域 (FD)。 Microsoft 提供相关工具，重新均衡 UD 和 FD 中的 Kafka 分区与副本。 

    有关详细信息，请参阅[使用 Apache Kafka on HDInsight 实现高可用性](apache-kafka-high-availability.md)。

* 创建群集后，HDInsight 允许更改辅助角色节点（托管 Kafka 代理）的数目。 可以通过 Azure 门户、Azure PowerShell 和其他 Azure 管理界面执行缩放。 对于 Kafka，在执行缩放操作后，应重新均衡分区副本。 重新均衡分区可让 Kafka 利用新的工作节点数。

    有关详细信息，请参阅[使用 Apache Kafka on HDInsight 实现高可用性](apache-kafka-high-availability.md)。

* Azure Monitor 日志可用于监视 Kafka on HDInsight。 Azure Monitor 日志可以显示虚拟机级别的信息，例如磁盘和 NIC 指标，以及 Kafka 中的 JMX 指标。

    有关详细信息，请参阅[分析 Apache Kafka on HDInsight 的日志](apache-kafka-log-analytics-operations-management.md)。

### <a name="apache-kafka-on-hdinsight-architecture"></a>Apache Kafka on HDInsight 体系结构

下图显示了一个典型的 Kafka 配置，该配置利用使用者组、分区和复制提供带容错功能的事件并行读取：

![Kafka 群集配置关系图](./media/apache-kafka-introduction/kafka-cluster.png)

Apache ZooKeeper 管理 Kafka 群集的状态。 Zookeeper 专用于并发、可复原和低延迟事务。 

Kafka 将记录（数据）存储在主题中。 记录由生成者生成，由使用者使用。 生成者将记录发送到 Kafka 代理。 HDInsight 群集中的每个辅助角色节点都是 Kafka 代理。 

主题跨代理对记录进行分区。 在使用记录时，每个分区最多可使用一个使用者来实现数据并行处理。

利用复制功能将分区复制到各个节点上，以防止发生节点（代理）服务中断。 关系图中用 (L) 表示的分区是给定分区的前导者。 生成方流量将根据由 ZooKeeper 管理的状态路由到每个节点的前导者。

## <a name="why-use-apache-kafka-on-hdinsight"></a>为何使用 Apache Kafka on HDInsight？

以下是可使用 Kafka on HDInsight 执行的常见任务和模式：

* **Apache Kafka 数据复制**：Kafka 提供 MirrorMaker 实用工具，用于在 Kafka 群集之间复制数据。

    有关使用 MirrorMaker 的信息，请参阅[使用 Apache Kafka on HDInsight 复制 Apache Kafka 主题](apache-kafka-mirroring.md)。

* **发布-订阅消息传送模式**：Kafka 提供一个生成者 API，用于将记录发布到 Kafka 主题。 订阅主题时，将使用 Consumer API。

    有关详细信息，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)。

* **流处理**：Kafka 通常与 Apache Storm 或 Spark 配合使用，以实现实时流式处理。 Kafka 0.10.0.0（HDInsight 版本 3.5 和 3.6）引入了流式处理 API，可用于构建流式处理解决方案，而无需使用 Storm 或 Spark。

    有关详细信息，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)。

* **横向缩放**：Kafka 可将 HDInsight 群集中不同节点之间的流进行分区。 使用者进程可与单个分区关联，以便在使用记录时提供负载均衡。

    有关详细信息，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)。

* **有序传送**：在每个分区中，记录按接收顺序存储在流中。 通过为每个分区关联一个使用者进程，可保证按顺序处理记录。

    有关详细信息，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)。

## <a name="use-cases"></a>用例

* **消息传送**：由于 Kafka 支持发布-订阅消息模式，因此它经常用作消息中转站。

* **活动跟踪**：由于 Kafka 提供按顺序进行日志记录的记录，因此它还可用于跟踪和重新创建活动。 例如，网站上或应用程序内的用户操作。

* **聚合**：使用流处理可从不同的流中聚合信息，将信息合并和集中到运营数据中。

* **转换**：使用流处理可将多个输入主题中的数据合并到一个或多个输出主题中，丰富其内容。

## <a name="next-steps"></a>后续步骤

使用以下链接了解如何使用 Apache Kafka on HDInsight：

* [快速入门：创建 Apache Kafka on HDInsight](apache-kafka-get-started.md)

* [教程：将 Apache Spark 与 Apache Kafka on HDInsight 配合使用](../hdinsight-apache-spark-with-kafka.md)

* [教程：将 Apache Storm 与 Apache Kafka on HDInsight 配合使用](../hdinsight-apache-storm-with-kafka.md)
