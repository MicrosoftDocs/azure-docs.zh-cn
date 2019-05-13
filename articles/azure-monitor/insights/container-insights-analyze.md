---
title: 使用适用于容器的 Azure Monitor 监视 AKS 群集性能 | Microsoft Docs
description: 本文介绍如何使用适用于容器的 Azure Monitor 查看和分析性能和日志数据。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/06/2019
ms.author: magoedte
ms.openlocfilehash: ed387f7038c5dee1a1685c918abcae49942cd55d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148847"
---
# <a name="understand-aks-cluster-performance-with-azure-monitor-for-containers"></a>使用适用于容器的 Azure Monitor 了解 AKS 群集性能 
借助适用于容器的 Azure Monitor，可以使用性能图表和运行状况从两个角度（直接从 AKS 群集查看，或是从 Azure Monitor 查看订阅中的所有 AKS 群集）查看 Azure Kubernetes 服务 (AKS) 群集的工作负载。 在监视特定 AKS 群集时，还可以查看 Azure 容器实例 (ACI)。

可以通过本文了解这两个角度的不同体验，以及如何有助于快速评估、调查和解决检测到的问题。

若要了解如何启用适用于容器的 Azure Monitor，请参阅[载入适用于容器的 Azure Monitor](container-insights-onboard.md)。

> [!IMPORTANT]
> 容器支持的 azure 监视器来监视运行 Windows Server 2019 的 AKS 群集当前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Azure Monitor 提供了一个多群集视图，显示所有受监视 AKS 群集中运行 Linux 和 Windows Server 2019 部署你的订阅中跨资源组的运行状况状态。  它显示发现的不受解决方案监视的 AKS 群集。 可以即时了解群集运行状况，并且可以从这里向下钻取到节点和控制器性能页，或者进行导航来查看群集的性能图表。  对于发现的标识为“不受监视”的 AKS 群集，可以随时为该群集启用监视功能。  

使用 Azure Monitor for 相比 Linux 群集的容器监视 Windows Server 群集的主要差异如下所示：

- 内存 RSS 跃点数不能用于 Windows 节点和容器 
- 磁盘存储容量信息不是可用于 Windows 节点
- 除了 Windows 容器日志提供了实时日志支持。
- 仅监视环境的 pod，不 Docker 环境。
- 预览版本中，支持最多为 30 的 Windows Server 容器。 此限制不适用于 Linux 容器。  

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户
登录到 [Azure 门户](https://portal.azure.com)。 

## <a name="multi-cluster-view-from-azure-monitor"></a>从 Azure Monitor 获得的多群集视图 
若要查看已部署的所有 AKS 群集的运行状况，请在 Azure 门户的左窗格中选择“监视”。  在“见解”部分，选择“容器”。  

![Azure Monitor 多群集仪表板示例](./media/container-insights-analyze/azmon-containers-multiview.png)

在“受监视的群集”选项卡上，可以了解以下情况：

1. 多少群集处于严重或不正常状态？多少群集处于正常或未报告状态（也称未知状态）？
2. 我的所有 [Azure Kubernetes 引擎（AKS 引擎）](https://github.com/Azure/aks-engine)部署是否都正常？
3. 每个群集部署多少个节点、 用户和系统 pod？
4. 磁盘空间是否可用以及是否有容量问题？

包含的运行状况有： 

* **正常** - VM 没有检测到任何问题，并且按要求运行。 
* **严重** - 检测到一个或多个严重问题，需要解决这些问题才能按预期还原正常操作状态。
* **警告** - 检测到一个或多个需要解决的问题，不解决这些问题可能会导致运行状况变得严重。
* **未知** - 如果服务无法与节点或 Pod 建立连接，则状态将更改为未知状态。
* **找不到** - 工作区、资源组或包含此解决方案的工作区的订阅已删除。
* **未授权** - 用户没有读取工作区中的数据所需的权限。
*  - 尝试从工作区中读取数据时发生错误。
* **配置不正确** - 未在指定工作区中正确配置适用于容器的 Azure Monitor。
* **没有数据** - 在过去 30 分钟内未向工作区报告数据。

运行状况状态计算总体群集状态为*的最差*三个状态有一个例外-如果任何三个状态为*未知*，整体群集状态将显示**未知**.  

下表提供了计算明细，该计算控制多群集视图中受监视群集的运行状况。

| |状态 |可用性 |  
|-------|-------|-----------------|  
|**用户 Pod**| | |  
| |Healthy |100% |  
| |警告 |90 - 99% |  
| |严重 |<90% |  
| |Unknown |如果未在过去 30 分钟报告 |  
|**系统 Pod**| | |  
| |Healthy |100% |
| |警告 |不适用 |
| |严重 |<100% |
| |Unknown |如果未在过去 30 分钟报告 |
|**Node** | | |
| |Healthy |>85% |
| |警告 |60 - 84% |
| |严重 |<60% |
| |Unknown |如果未在过去 30 分钟报告 |

在群集列表中，可以通过单击群集名称向下钻取到“群集”页，通过单击该特定群集的“节点”列中的节点汇总向下钻取到“节点”性能页，或者通过单击“用户 Pod”或“系统 Pod”列的汇总向下钻取到“控制器”性能页。   

## <a name="view-performance-directly-from-an-aks-cluster"></a>直接从 AKS 群集查看性能
可以直接从 AKS 群集访问适用于容器的 Azure Monitor，只需从左窗格中选择“见解”即可。 分四个视角查看有关 AKS 群集的信息：

- 群集
- Nodes 
- 控制器  
- 容器

单击“见解”时打开的默认页为“群集”，其中包括四个性能折线图，显示群集的主要性能指标。 

![“群集”选项卡上的性能图表示例](./media/container-insights-analyze/containers-cluster-perfview.png)

性能图表显示四个性能指标：

- **节点 CPU 利用率&nbsp;%**：从聚合视角反映整个群集的 CPU 利用率。 若要筛选出相关结果的时间范围，可通过图表上的百分位选择器选择“Avg”、“Min”、“Max”、“50th”、“90th”、“95th”（单选或复选均可）。 
- **节点内存利用率&nbsp;%**：提供整个群集的内存利用率的聚合视角。 若要筛选出相关结果的时间范围，可通过图表上的百分位选择器选择“Avg”、“Min”、“Max”、“50th”、“90th”、“95th”（单选或复选均可）。 
- **节点计数**：Kubernetes 提供的节点计数和状态。 表示的群集节点状态为“全部”、“就绪”和“未就绪”，可通过图表上的选择器单独筛选或以组合方式进行筛选。 
- **活动 Pod 计数**：Kubernetes 提供的 Pod 计数和状态。 表示的 Pod 状态为“全部”、“挂起”、“正在运行”和“未知”，可通过图表上的选择器单独筛选或以组合方式进行筛选。 

可以使用向左键/向右键循环浏览图表上的每个数据点，使用向上键/向下键循环显示百分位线。

用于容器的 azure Monitor 还支持 Azure Monitor[指标资源管理器](../platform/metrics-getting-started.md)，可以创建您自己绘图图表，关联和调查趋势，并位置固定到仪表板。 在指标资源管理器中，还可以使用所设置的条件将指标可视化为[基于指标的警报规则](../platform/alerts-metric.md)的基础。  

## <a name="view-container-metrics-in-metrics-explorer"></a>在指标资源管理器中查看容器指标
在指标资源管理器中，可以通过适用于容器的 Azure Monitor 查看聚合的节点和 Pod 利用率指标。 下表汇总了详细信息，这些信息有助于你了解如何使用指标图表来可视化容器指标。

|命名空间 | 指标 |
|----------|--------|
| insights.container/nodes | |
| | cpuUsageMillicores |
| | cpuUsagePercentage |
| | memoryRssBytes |
| | memoryRssPercentage |
| | memoryWorkingSetBytes |
| | memoryWorkingSetPercentage |
| | nodesCount |
| insights.container/pods | |
| | PodCount |

可以应用指标[拆分](../platform/metrics-charts.md#apply-splitting-to-a-chart)，以便按维度来查看它，并以可视化方式表现其片段相互之间的不同之处。 对于节点，可以按“主机”维度将图表分段，并可在 Pod 中按以下维度将其分段：

* 控制器
* Kubernetes 命名空间
* 节点
* 阶段

## <a name="analyze-nodes-controllers-and-container-health"></a>分析节点、控制器和容器运行状况

切换到“节点”、“控制器”和“容器”选项卡时，页面右侧会自动显示“属性”窗格。 它显示所选项的属性，包括定义用于组织 Kubernetes 对象的标签。 选择在 Linux 节点后，它还演示了部分下**本地磁盘容量**可用磁盘空间和用于向节点显示每个磁盘的百分比。 单击窗格中的 >> 链接以查看\隐藏窗格。 

![示例 Kubernetes 透视属性窗格](./media/container-insights-analyze/perspectives-preview-pane-01.png)

在层次结构中展开对象时，属性窗格将根据所选对象进行更新。 在窗格中，还可通过单击窗格顶部的“查看 Kubernetes 事件日志”链接，查看具有预定义日志搜索的 Kubernetes 事件。 有关查看 Kubernetes 日志数据的详细信息，请参阅[搜索日志以分析数据](container-insights-log-search.md)。 时您正在查看群集资源，可以查看容器日志和实时事件。 有关此功能，来授予和控制访问所需的配置详细信息，请参阅[如何查看容器中的与 Azure Monitor 日志实时](container-insights-live-logs.md)。 

使用 **+ 添加筛选器**从页面顶部的选项以筛选的视图的结果**服务**，**节点**， **Namespace**，或**节点池**并选择筛选器作用域后, 您从选择中显示的值之一**选择值**字段。  筛选器在配置后会在用户查看任何视角的 AKS 群集时进行全局应用。  公式只支持等号。  可以在第一个筛选器的基础上添加更多的筛选器，进一步缩小结果范围。  例如，如果指定了一个按“节点”筛选的筛选器，则第二个筛选器只能选择“服务”或“命名空间”。  

![通过筛选器缩小结果范围的示例](./media/container-insights-analyze/add-filter-option-01.png)

在一个选项卡中指定一个筛选器后，如果又选择一个筛选器，则前者会继续应用。在单击指定筛选器旁边的 **x** 符号后，该筛选器会被删除。   

切换到“节点”选项卡，行层次结构遵循以群集节点开头的 Kubernetes 对象模型。 展开节点即可查看在节点上运行的一个或多个 Pod。 如果将多个容器分组到 Pod 中，它们将在层次结构中的最后一行显示。 还可查看有多少非 Pod 相关工作负荷在主机上运行（如果主机有处理器或内存使用压力）。

![性能视图中的示例 Kubernetes 节点层次结构](./media/container-insights-analyze/containers-nodes-view.png)

后面的所有基于 Linux 的节点列表中显示了运行 Windows Server 2019 操作系统的 Windows Server 容器。 当您展开 Windows Server 节点时，可以查看一个或多个 pod 和在节点上运行的容器。 选中一个节点后，属性窗格将显示版本信息，因为 Windows Server 节点不具有安装了代理，不包括的代理信息。  

![示例使用列出的 Windows Server 节点的节点层次结构](./media/container-insights-analyze/nodes-view-windows.png) 

运行 Linux 操作系统的 Azure 容器实例虚拟节点显示在列表中最后一个 AKS 群集节点之后。  展开 ACI 虚拟节点时，可以查看在节点上运行的一个或多个 ACI Pod 和容器。  不会为节点（只为 Pod）收集和报告指标。

![列出了容器实例的示例节点层次结构](./media/container-insights-analyze/nodes-view-aci.png)

从展开的节点中，你可以从在节点上运行的 pod 或容器向下钻取到控制器来查看针对该控制器筛选的性能数据。 单击特定节点的“控制器”列下的值。   
![性能视图中从节点到控制器的示例向下钻取](./media/container-insights-analyze/drill-down-node-controller.png)

可从页面顶部选择控制器或容器，查看这些对象的状态和资源使用率。  如果想要查看内存利用率，可在“指标”下拉列表中选择“内存 RSS”或“内存工作集”。 仅 Kubernetes 1.8 版和更高版本支持**内存 RSS**。 否则，看到的 **Min&nbsp;%** 值会显示为 *NaN&nbsp;%*，它表示未定义或无法表示的值的数值数据类型值。 

![容器节点性能视图](./media/container-insights-analyze/containers-node-metric-dropdown.png)

默认情况下，性能数据基于过去六个小时的数据，但可以使用左上角的“TimeRange”选项更改时间窗口。 若要在时间范围内筛选结果，还可通过百分位选择器选择“Avg”、“Min”、“Max”、“50th”、“90th”、“95th”。 

![用于数据筛选的百分位选择](./media/container-insights-analyze/containers-metric-percentile-filter.png)

当鼠标悬停在“趋势”列下的条形图上方时，每一条都显示 15 分钟示例期间内的 CPU 或内存使用情况（具体取决于所选指标）。 通过键盘选择趋势图后，可以使用 Alt + PageUp 或 Alt + PageDown 键单独循环浏览每一条，并获得与鼠标悬停操作相同的详细信息。

![“趋势”条形图悬停示例](./media/container-insights-analyze/containers-metric-trend-bar-01.png)    

在下一个示例中，请注意列表中的第一项 - 节点 aks-nodepool1-，其“容器”的值为 9，表示部署的容器汇总总数。

![单节点容器数汇总示例](./media/container-insights-analyze/containers-nodes-containerstotal.png)

它有助于快速确定群集中节点间的容器是否适当均衡。 

下表介绍查看节点时显示的信息：

| 列 | Description | 
|--------|-------------|
| 名称 | 主机的名称。 |
| 状态 | 节点状态的 Kubernetes 视图。 |
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 基于所选时段百分位的平均节点百分比。 |
| Avg、Min、Max、50th、90th | 基于所选时段的平均节点实际值。 平均值根据为节点设置的 CPU/内存限制进行计算；对于 Pod 和容器，平均值为主机报告的平均值。 |
| 容器 | 容器数量。 |
| 运行时间 | 表示节点启动或重启后的时间。 |
| 控制器 | 仅适用于容器和 Pod。 它显示驻留的控制器。 并非所有 Pod 都在控制器中，因此有些 Pod 可能会显示 **N/A**。 | 
| 趋势 Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 条形图趋势表示控制器的平均百分位指标百分比。 |

在选择器中，选择“控制器”。

![选择控制器视图](./media/container-insights-analyze/containers-controllers-tab.png)

可以在此处查看控制器和 ACI 虚拟节点控制器或未连接到控制器的虚拟节点 Pod 的性能运行状况。

![<名称>控制器性能视图](./media/container-insights-analyze/containers-controllers-view.png)

行层次结构以控制器开始，当展开控制器时，可以查看一个或多个 pod。  展开 Pod，最后一行显示分组到 Pod 的容器。 从展开的控制器中，你可以向下钻取到运行它的节点来查看针对该节点筛选的性能数据。 未连接到控制器的 ACI Pod 在列表中最后列出。

![列出了容器实例 Pod 的示例控制器层次结构](./media/container-insights-analyze/controllers-view-aci.png)

单击特定控制器的“节点”列下的值。   

![性能视图中从节点到控制器的示例向下钻取](./media/container-insights-analyze/drill-down-controller-node.png)

下表介绍查看控制器时显示的信息：

| 列 | Description | 
|--------|-------------|
| 名称 | 控制器的名称。|
| 状态 | 完成运行状态时容器的汇总状态，例如“正常”、“已终止”、“已失败”、“已停止”或“已暂停”。 如果容器仍在运行，但是状态未正确显示或者未被代理选择并且超出 30 分钟后仍未响应，则状态为“未知”。 下表中提供了状态图标的更多详细信息。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 汇总所选度量值和百分位数的每个实体的平均百分比的平均值。 |
| Avg、Min、Max、50th、90th  | 所选百分位的容器的平均 CPU millicore 或内存性能汇总。 平均值根据为 Pod 设置的 CPU/内存限制进行计算。 |
| 容器 | 控制器或 Pod 的容器总数。 |
| 重启数 | 容器重启计数汇总。 |
| 运行时间 | 表示容器启动后的时间。 |
| 节点 | 仅适用于容器和 Pod。 它显示驻留的控制器。 | 
| 趋势 Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;%| 条形图趋势表示控制器的平均百分位指标。 |

状态字段中的图标指示容器的联机状态：
 
| 图标 | 状态 | 
|--------|-------------|
| ![“就绪”运行状态图标](./media/container-insights-analyze/containers-ready-icon.png) | 正在运行（就绪）|
| ![“正在等待”或“已暂停”状态图标](./media/container-insights-analyze/containers-waiting-icon.png) | “正在等待”或“已暂停”|
| ![“上次报告正在运行”状态图标](./media/container-insights-analyze/containers-grey-icon.png) | 上次报告正在运行但已超过 30 分钟未响应|
| ![成功状态图标](./media/container-insights-analyze/containers-green-icon.png) | 成功停止或无法停止|

状态图标显示的计数基于 Pod 提供的数据。 它显示最差的两个状态。将鼠标悬停在状态上时，它显示容器中所有 Pod 的汇总状态。 如果没有就绪状态，状态值会显示 **(0)**。 

在选择器中，选择“容器”。

![选择容器视图](./media/container-insights-analyze/containers-containers-tab.png)

在此处可查看 Azure Kubernetes 和 Azure 容器实例容器的性能运行状况。  

![<名称>控制器性能视图](./media/container-insights-analyze/containers-containers-view.png)

从容器中，可以向下钻取到某个 pod 或节点来查看针对该对象筛选的性能数据。 单击特定容器的“Pod”或“节点”列下的值。   

![性能视图中从节点到控制器的示例向下钻取](./media/container-insights-analyze/drill-down-controller-node.png)

下表介绍查看容器时显示的信息：

| 列 | Description | 
|--------|-------------|
| 名称 | 控制器的名称。|
| 状态 | 容器状态（如果有）。 接下来的表格提供状态图标的更多详细信息。|
| Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 每个实体在选定指标和百分位的平均百分比汇总。 |
| Avg、Min、Max、50th、90th  | 容器在选定百分位的平均 CPU millicore 或内存性能汇总。 平均值根据为 Pod 设置的 CPU/内存限制进行计算。 |
| Pod | Pod 驻留的容器。| 
| 节点 |  容器驻留的节点。 | 
| 重启数 | 表示容器启动后的时间。 |
| 运行时间 | 表示容器启动或重启后的时间。 |
| 趋势 Avg&nbsp;%、Min&nbsp;%、Max&nbsp;%、50th&nbsp;%、90th&nbsp;% | 条形图趋势表示容器的平均百分位指标百分比。 |

状态字段中的图标指示 Pod 的联机状态，如下表所述：
 
| 图标 | 状态 |  
|--------|-------------|  
| ![“就绪”运行状态图标](./media/container-insights-analyze/containers-ready-icon.png) | 正在运行（就绪）|  
| ![“正在等待”或“已暂停”状态图标](./media/container-insights-analyze/containers-waiting-icon.png) | “正在等待”或“已暂停”|  
| ![“上次报告正在运行”状态图标](./media/container-insights-analyze/containers-grey-icon.png) | 上次报告正在运行但已超过 30 分钟未响应|  
| ![“已终止”状态图标](./media/container-insights-analyze/containers-terminated-icon.png) | 成功停止或无法停止|  
| ![“已失败”状态图标](./media/container-insights-analyze/containers-failed-icon.png) | “已失败”状态 |  

## <a name="next-steps"></a>后续步骤
- 审阅[适用于容器的 Azure Monitor 创建性能警报](container-insights-alerts.md)若要了解如何为高的 CPU 和内存利用率，以支持 DevOps 或操作流程和过程创建警报。 
- 视图[记录查询示例](container-insights-log-search.md#search-logs-to-analyze-data)若要查看预定义的查询和示例，以评估或自定义的警报、 可视化，或分析你的群集。
