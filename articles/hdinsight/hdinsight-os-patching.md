---
title: 为基于 Linux 的 HDInsight 群集配置 OS 修补计划 - Azure
description: 了解如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划。
author: omidm1
ms.author: omidm
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/24/2019
ms.openlocfilehash: 5b8ed75863087e077d483c792ac4134a0c3e1eb0
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203644"
---
# <a name="os-patching-for-hdinsight"></a>针对 HDInsight 的 OS 修补 

> [!IMPORTANT]
> Ubuntu 映像可在发布后的 3 个月内用于创建新的 HDInsight 群集。 自 2019 年 1 月起，系统**不**会自动修补正在运行的群集。 客户必须使用脚本操作或其他机制来修补正在运行的群集。 新创建的群集将始终包含最新的可用更新，其中包括最新的安全修补程序。

## <a name="how-to-configure-the-os-patching-schedule-for-linux-based-hdinsight-clusters"></a>如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划
需不定期重启 HDInsight 群集中的虚拟机，以便安装重要的安全修补程序。 

使用本文中描述的脚本操作，可以按如下所示修改 OS 修补计划：
1. 安装完整的 OS 更新或安装安全更新程序
2. 重新启动 VM

> [!NOTE]  
> 此脚本操作仅适用于 2016 年 8 月 1 日后创建的基于 Linux 的 HDInsight 群集。 仅在重启 VM 后，修补程序才生效。 此脚本不会自动应用更新的所有将来的更新周期。 运行每个时间的新更新需要安装更新并重启 VM 才能应用该脚本。

## <a name="how-to-use-the-script"></a>如何使用脚本 

使用此脚本需要以下信息：
1. 脚本位置： https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv02/os-patching-reboot-config.sh。HDInsight 使用此 URI 在群集中的所有虚拟机上查找并运行脚本。
  
2. 应用该脚本的群集节点类型：头节点、辅助节点、zookeeper。 此脚本必须应用于群集中的所有节点类型。 如果它不应用于一个节点类型，则将不会更新该节点类型的虚拟机。


3.  参数：此脚本接受一个数字参数：

    | 参数 | 定义 |
    | --- | --- |
    | 安装完整的 OS 更新/安装仅安全更新 |0 或 1。 值为 0 1 表示安装完整的 OS 更新时才安装安全更新。 如果没有提供任何参数，默认值为 0。 |

> [!NOTE]  
> 将其应用于现有群集时，必须将此脚本标记为持久化。 否则，通过缩放操作创建的任何新节点都将使用默认修补计划。  如果在群集创建过程中应用该脚本，则其会自动持久化。

## <a name="next-steps"></a>后续步骤

若要了解使用脚本操作的具体步骤，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](hdinsight-hadoop-customize-cluster-linux.md)中的以下部分：

* [在创建群集期间使用脚本操作](hdinsight-hadoop-customize-cluster-linux.md#use-a-script-action-during-cluster-creation)
* [将脚本操作应用到正在运行的群集](hdinsight-hadoop-customize-cluster-linux.md#apply-a-script-action-to-a-running-cluster)
