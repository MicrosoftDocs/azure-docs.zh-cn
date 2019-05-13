---
title: 自动缩放 Azure HDInsight 群集（预览）
description: 使用 HDInsight 自动缩放功能来自动缩放群集
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: hrasheed
ms.openlocfilehash: f8803a498e62958a5488f2ac8830137c37533e54
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413703"
---
# <a name="automatically-scale-azure-hdinsight-clusters-preview"></a>自动缩放 Azure HDInsight 群集（预览）

Azure HDInsight 群集自动缩放功能自动缩放辅助角色节点数在群集中向上和向下。 不能当前缩放其他类型的群集中的节点。  创建新 HDInsight 群集期间，可以设置最小和最大工作节点数。 然后，自动缩放监视 analytics 负载的资源要求，并向上或向下缩放辅助角色节点数。 没有此功能无任何额外费用。

## <a name="cluster-compatibility"></a>群集兼容性

> [!Important]
> 自动缩放功能仅适用于 2019 年 5 中功能的公开上市后创建的群集。 它不适用于预先存在的群集。

下表介绍的群集类型和自动缩放功能与兼容的版本。

| Version | Spark | Hive | LLAP | Hbase | Kafka | Storm | ML |
|---|---|---|---|---|---|---|---|
| HDInsight 3.6 而无需 ESP | “是” | 是 | 否 | 否 | 否 | 否 | “否” |
| HDInsight 4.0 而无需 ESP | “是” | 是 | 否 | 否 | 否 | 否 | “否” |
| 对于 ESP HDInsight 3.6 | “是” | 是 | 否 | 否 | 否 | 否 | “否” |
| 对于 ESP HDInsight 3.6 | “是” | 是 | 否 | 否 | 否 | 否 | “否” |

## <a name="how-it-works"></a>工作原理

可以选择基于负载的缩放或基于计划的缩放 HDInsight 群集。 基于负载的缩放会更改在群集中，将设置，以确保最佳 CPU 利用率并降低运行成本的范围内的节点数。

基于计划的缩放更改群集中的节点数根据特定时间才会生效的条件。 这些条件缩放到所需数目的节点群集。

### <a name="metrics-monitoring"></a>指标监视

自动缩放会持续监视群集并收集以下指标：

* **总待处理 CPU**：开始执行所有待处理容器所需的核心总数。
* **总待处理内存**：开始执行所有待处理容器所需的总内存（以 MB 为单位）。
* **总可用 CPU**：活动工作节点上所有未使用核心的总和。
* **总可用内存**：活动工作节点上未使用内存的总和（以 MB 为单位）。
* **每个节点的已使用内存**：工作节点上的负载。 使用了 10 GB 内存的工作节点的负载被认为比使用了 2 GB 内存的工作节点的负载更大。
* **每个节点的应用程序主机数**：在工作节点上运行的应用程序主机 (AM) 容器的数量。 托管两个 AM 容器的工作节点被认为比托管零个 AM 容器的工作节点更重要。

每 60 秒检查一次上述指标。 自动缩放可以纵向扩展和缩减决策基于这些指标。

### <a name="load-based-cluster-scale-up"></a>基于负载的群集向上扩展

检测到以下情况时，自动缩放将发出纵向扩展请求：

* 挂起的 CPU 总数大于总可用 CPU 的时间超过 3 分钟。
* 挂起的内存总数大于可用内存总量超过 3 分钟。

HDInsight 服务计算多少新辅助角色节点所需满足的当前 CPU 和内存需求，然后添加所需的节点数的不断增长的请求。

### <a name="load-based-cluster-scale-down"></a>基于负载的群集缩减

检测到以下情况时，自动缩放将发出纵向缩减请求：

* 总待处理 CPU 小于总可用 CPU 的时间超过 10 分钟。
* 总待处理内存小于总可用内存的时间超过 10 分钟。

根据每个节点的当前 CPU 和内存要求 AM 容器数，自动缩放发出一个请求将删除一定数量的节点。 该服务还检测它们节点是基于当前的作业执行的删除候选项。 缩放减少操作首先取消配置节点，，并从群集删除它们。

## <a name="get-started"></a>开始使用

### <a name="create-a-cluster-with-load-based-autoscaling"></a>创建具有基于负载的自动缩放的群集

若要启用基于负载的缩放的自动缩放功能，作为正常的群集创建过程的一部分完成以下步骤：

1. 选择“自定义(大小、设置、应用)”而非“快速创建”。
1. 上**自定义**第 5 步 (**群集大小**)，检查**辅助角色节点自动缩放**复选框。
1. 选择的选项**基于负载**下**自动缩放类型**。
1. 为以下属性输入所需的值：  

    * 初始工作节点数。  
    * 最小工作节点数。  
    * 最大工作节点数。  

    ![启用辅助角色节点基于负载的自动缩放选项](./media/hdinsight-autoscale-clusters/usingAutoscale.png)

工作节点的初始数量必须介于最小值和最大值之间（含最大值和最小值）。 此值定义创建群集时的群集初始大小。 最小工作节点数必须大于零。

### <a name="create-a-cluster-with-schedule-based-autoscaling"></a>创建具有基于计划的自动缩放的群集

若要启用基于计划的缩放的自动缩放功能，作为正常的群集创建过程的一部分完成以下步骤：

1. 选择“自定义(大小、设置、应用)”而非“快速创建”。
1. 上**自定义**第 5 步 (**群集大小**)，检查**辅助角色节点自动缩放**复选框。
1. 输入**辅助节点数**，它可以控制扩展群集的限制。
1. 选择的选项**计划基于**下**自动缩放类型**。
1. 单击**配置**以打开**自动缩放配置**窗口。
1. 选择你的时区，然后单击 **+ 添加条件**
1. 选择新的条件应该应用到一周的日期。
1. 编辑条件的生效时间及群集应缩放到的节点数的时间。
1. 如果需要，添加更多条件。

    ![启用辅助角色节点基于计划的自动缩放选项](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-schedule-creation.png)

节点数必须介于 1 和添加条件之前输入的辅助角色节点数之间。

### <a name="final-creation-steps"></a>最终创建步骤

对于基于负载和计划的缩放，请通过单击选择辅助角色节点的 VM 类型**辅助角色节点大小**并**头节点大小**。 选择每个节点类型的 VM 类型后，可以看到整个群集的估计的开销范围。 调整以适合你的预算的 VM 类型。

![启用辅助角色节点基于计划的自动缩放选项](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-node-size-selection.png)

你的订阅具有针对每个区域的容量配额。 头节点核心总数加最大工作节点数不能超过容量配额。 但是，此配额是软性限制；始终可创建支持票证来轻松地增加此配额。

> [!Note]  
> 如果你超出总核心配额限制，你将收到错误消息，指出最大节点超出此区域中的可用内核，请选择另一个区域或联系支持人员增加配额。

有关使用 Azure 门户创建 HDInsight 群集的详细信息，请参阅[使用 Azure 门户在 HDInsight 中创建基于 Linux 的群集](hdinsight-hadoop-create-linux-clusters-portal.md)。  

### <a name="create-a-cluster-with-a-resource-manager-template"></a>使用资源管理器模板创建群集

#### <a name="load-based-autoscaling"></a>基于负载的自动缩放

您可以创建 HDInsight 群集与基于负载的自动缩放的 Azure 资源管理器模板，通过添加`autoscale`到节点`computeProfile`  >  `workernode`的属性部分`minInstanceCount`和`maxInstanceCount`作为下面的 json 代码段中所示。

```json
{
  "name": "workernode",
  "targetInstanceCount": 4,
  "autoscale": {
      "capacity": {
          "minInstanceCount": 2,
          "maxInstanceCount": 10
      }
  },
  "hardwareProfile": {
      "vmSize": "Standard_D13_V2"
  },
  "osProfile": {
      "linuxOperatingSystemProfile": {
          "username": "[parameters('sshUserName')]",
          "password": "[parameters('sshPassword')]"
      }
  },
  "virtualNetworkProfile": null,
  "scriptActions": []
}
```

若要深入了解如何使用资源管理器模板创建群集，请参阅[在 HDInsight 中使用资源管理器模板创建 Apache Hadoop 群集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)。  

#### <a name="schedule-based-autoscaling"></a>基于计划的自动缩放

您可以创建 HDInsight 群集基于计划的自动缩放 Azure 资源管理器模板，通过添加`autoscale`到节点`computeProfile`  >  `workernode`部分。 `autoscale`节点包含`recurrence`具有`timezone`和`schedule`描述更改将时生效。

```json
{
  "autoscale": {
    "recurrence": {
      "timeZone": "Pacific Standard Time",
      "schedule": [
        {
          "days": [
            "Monday",
            "Tuesday",
            "Wednesday",
            "Thursday",
            "Friday"
          ],
          "timeAndCapacity": {
            "time": "11:00",
            "minInstanceCount": 10,
            "maxInstanceCount": 10
          }
        },
      ]
    }
  },
  "name": "workernode",
  "targetInstanceCount": 4,
}
```

### <a name="enable-and-disable-autoscale-for-a-running-cluster"></a>为正在运行的群集启用和禁用自动缩放

若要在正在运行的群集上启用自动缩放，请选择**群集大小**下**设置**。 然后单击**启用自动缩放**。 选择您想和输入的基于负载或基于计划的缩放选项的自动缩放的类型。 最后，单击“保存”。

![启用辅助角色节点基于计划的自动缩放选项](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-enable-running-cluster.png)

## <a name="monitoring"></a>监视

### <a name="cluster-status"></a>群集状态

在 Azure 门户中列出的群集状态可以帮助您监视自动缩放活动。

![启用辅助角色节点基于负载的自动缩放选项](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-cluster-status.png)

以下列表介绍了所有可能会看到群集状态消息。

| 群集状态 | 说明 |
|---|---|
| 正在运行 | 群集运行正常。 所有以前的自动缩放活动已成功完成。 |
| 正在更新  | 正在更新群集自动缩放配置。  |
| HDInsight 配置  | 群集增加或减少操作正在进行中。  |
| 更新时出错  | HDInsight 自动缩放配置更新过程中遇到问题。 客户可以选择以重试更新或禁用自动缩放。  |
| 错误  | 群集后，出错了，并且不可用。 删除此群集，并创建一个新。  |

若要查看在群集中当前的节点数，请转到**群集大小**图表上**概述**页上为群集，或单击**群集大小**下**设置**。

### <a name="operation-history"></a>运行历史记录

可查看群集指标中包含的群集增加和减少历史记录。 此外可以通过过去一天、 周或其他时间段内的列出所有的缩放操作。

选择**指标**下**监视**。 然后单击**添加的度量值**并**活动工作线程数**从**指标**下拉框中。 单击右上方，若要更改的时间范围中的按钮。

![启用辅助角色节点基于计划的自动缩放选项](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-chart-metric.png)


## <a name="next-steps"></a>后续步骤

* 阅读[缩放最佳做法](hdinsight-scaling-best-practices.md)，了解手动缩放群集的最佳做法
