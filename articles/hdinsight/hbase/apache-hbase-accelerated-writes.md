---
title: Azure HDInsight 的 Apache HBase 加速写入
description: 概述 Azure HDInsight 加速将写入功能，使用高级托管磁盘来提高性能的 Apache HBase 写入继续操作日志。
services: hdinsight
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.topic: conceptual
ms.date: 4/29/2019
ms.openlocfilehash: 219899c2e336f544ff6572589cc79f84f555490d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233837"
---
# <a name="azure-hdinsight-accelerated-writes-for-apache-hbase"></a>Azure HDInsight 的 Apache HBase 加速写入

本文提供了背景**加速写入**Azure HDInsight 中的 Apache HBase 的功能和如何它可用于有效地提高写入性能。 **加速写入**使用[Azure 高级 SSD 托管磁盘](../../virtual-machines/linux/disks-types.md#premium-ssd)以提高性能的 Apache HBase 写入提前日志 (WAL)。 若要了解有关 Apache HBase 的详细信息，请参阅[什么是 HDInsight 中的 Apache HBase](apache-hbase-overview.md)。

## <a name="overview-of-hbase-architecture"></a>HBase 体系结构概述

在 HBase 中，**行**包含一个或多个**列**并由标识**行键**。 多个行构成**表**。 列包含**单元格**，这是该列中的值的时间戳的版本。 列分为**列系列**，和列系列中的所有列一起都存储在存储文件中名为**Hfile**。

**区域**在 HBase 用于进行数据处理负载平衡。 HBase 首先存储在单个区域中的表的行。 行可分布在多个区域的表会增加中的数据量。 **区域服务器**可以处理多个区域的请求。

## <a name="write-ahead-log-for-apache-hbase"></a>预写日志的 Apache HBase

HBase 首先将数据更新写入为一种名为编写提前日志 (WAL) 的提交日志类型。 更新存储在 WAL 后，它写入到内存中**MemStore**。 当内存中的数据达到其最大容量时，其写入磁盘， **HFile**。

如果**regionserver 内存**发生故障或变得不可用会刷新之前，请继续写入日志可用于重播的更新。 而无需 WAL，如果**regionserver 内存**刷新更新前发生了崩溃**HFile**，所有这些更新都将丢失。

## <a name="accelerated-writes-feature-in-azure-hdinsight-for-apache-hbase"></a>Apache HBase 的 Azure HDInsight 中的加速的写入功能

加速写入功能解决了导致通过使用预写日志中的云存储的更高的写入的延迟问题。  群集的 HDInsight 的 Apache HBase 的加速写入功能，请附加到每个 regionserver 内存 （辅助角色节点） 的高级 SSD 托管磁盘。 编写提前日志写入到 Hadoop 文件系统 (HDFS) 上这些高级托管磁盘而不是云存储装载。  高级托管磁盘使用固态硬盘 (Ssd)，并提供带容错功能的极好的 I/O 性能。  与非托管磁盘，如果一个存储单元出现故障，它不会影响同一个可用性集中其他存储单元。  因此，托管磁盘提供低写入延迟和更好地为您的应用程序的复原能力。 若要了解有关 Azure 托管磁盘的详细信息，请参阅[简介 Azure 托管磁盘](../../virtual-machines/windows/managed-disks-overview.md)。 

## <a name="how-to-enable-accelerated-writes-for-hbase-in-hdinsight"></a>如何为 HDInsight 中的 HBase 启用加速写入

若要使用加速写入功能创建新的 HBase 群集，请按照中的步骤[在 HDInsight 中设置群集](../hdinsight-hadoop-provision-linux-clusters.md)直至到达**步骤 3 中，存储**。 下**元存储设置**，单击旁边的复选框**启用加速写入 （预览版）**。 然后，继续创建群集的剩余步骤。

![启用 HDInsight Apache HBase 的加速的写入选项](./media/apache-hbase-accelerated-writes/accelerated-writes-cluster-creation.png)

## <a name="other-considerations"></a>其他注意事项

若要保留的数据持久性，请至少包含三个工作节点创建群集。 创建后，不能缩减到少于三个工作节点群集。

刷新或删除该群集之前禁用 HBase 表，不会丢失继续写入日志数据。

```
flush 'mytable'
```

```
disable 'mytable'
```

## <a name="next-steps"></a>后续步骤

* 上的官方 Apache HBase 文档[继续写入日志功能](https://hbase.apache.org/book.html#wal)
* 若要使用加速写入将 HDInsight 的 Apache HBase 群集升级，请参阅[将 Apache HBase 群集迁移到新版本](apache-hbase-migrate-new-version.md)。
