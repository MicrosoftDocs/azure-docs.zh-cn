---
title: 在 Azure Monitor 中收集和分析性能计数器 | Microsoft Docs
description: 性能计数器由 Azure Monitor 收集，用于分析 Windows 和 Linux 代理的性能。  本文介绍了如何为 Windows 和 Linux 代理配置性能计数器收集、这些性能计数器在工作区中的存储详情和如何在 Azure 门户中对其进行分析。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: 20e145e4-2ace-4cd9-b252-71fb4f94099e
ms.service: log-analytics
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/28/2018
ms.author: magoedte
ms.openlocfilehash: 76f4061af816c59e644db99913193ed6fcf24d18
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205754"
---
# <a name="windows-and-linux-performance-data-sources-in-azure-monitor"></a>Azure Monitor 中的 Windows 和 Linux 性能数据源
Windows 和 Linux 中的性能计数器提供对硬件组件、操作系统和应用程序性能的见解。  除聚合性能数据以用于长期分析和报告外，Azure Monitor 还可以定期收集性能计数器以进行近实时 (NRT) 分析。

![性能计数器](media/data-sources-performance-counters/overview.png)

## <a name="configuring-performance-counters"></a>配置性能计数器
通过[“高级设置”中的“数据”菜单](agent-data-sources.md#configuring-data-sources)配置性能计数器。

首次为新的工作区配置 Windows 或 Linux 性能计数器时，可以选择快速创建几个通用的计数器。  将这些计数器在一个复选框中依次列出。  请确保已选中所有想要首先创建的计数器，并单击“**添加选定的性能计数器**。

对于 Windows 性能计数器，可以为每个性能计数器选择一个特定实例。 对于 Linux 性能计数器，选择的每个计数器的实例会应用于父计数器的所有子计数器。 下表显示 Linux 和 Windows 性能计数器的可用通用实例。

| 实例名称 | 描述 |
| --- | --- |
| \_Total |所有实例的总计 |
| \* |所有实例 |
| (/&#124;/var) |匹配命名的实例：/ 或 /var |

### <a name="windows-performance-counters"></a>Windows 性能计数器

![配置 Windows 性能计数器](media/data-sources-performance-counters/configure-windows.png)

遵循以下步骤添加要收集的新 Windows 性能计数器。

1. 按照 *object(instance)\counter* 格式在文本框中键入计数器的名称。  开始键入时，会显示通用计数器的匹配列表。  可以选择列表中的计数器或者键入自己的计数器。  还可以通过指定 *object\counter* 返回特定计数器的所有实例。  

    在从命名实例中收集 SQL Server 性能计数器时，所有命名实例计数器以 MSSQL$ 开头，并且后面接实例的名称。  例如，若要从命名 SQL 实例 INST2 的数据库性能对象收集所有数据库的“日志缓存命中率”计数器，请指定 `MSSQL$INST2:Databases(*)\Log Cache Hit Ratio`。

2. 单击 **+** 或按 **Enter** 将计数器添加到列表中。
3. 添加计数器后，计数器将把 10 秒作为“**采样间隔**”的默认时间。  如果想要降低收集的性能数据的存储要求，可以将此值更改为更高值，最高可达 1800 秒（30 分钟）。
4. 添加完计数器后，单击屏幕顶部的“**保存**”按钮保存配置。

### <a name="linux-performance-counters"></a>Linux 性能计数器

![配置 Linux 性能计数器](media/data-sources-performance-counters/configure-linux.png)

遵循以下步骤添加要收集的新 Linux 性能计数器。

1. 默认情况下，所有配置更改均会自动推送到所有代理。  对于 Linux 代理，配置文件发送到 Fluentd 数据收集器。  如果想在每个 Linux 代理上手动修改此文件，请取消选中“将下面的配置应用到我的 Linux 计算机”框并遵循下面的指南。
2. 按照 *object(instance)\counter* 格式在文本框中键入计数器的名称。  开始键入时，会显示通用计数器的匹配列表。  可以选择列表中的计数器或者键入自己的计数器。  
3. 单击 **+** 或按 **Enter** 将计数器添加到此对象的其他计数器列表中。
4. 一个对象的所有计数器使用相同的“**采样间隔**”。  默认为 10 秒。  如果想要降低收集的性能数据的存储要求，可以将此值更改为更高值，最高可达 1800 秒（30 分钟）。
5. 添加完计数器后，单击屏幕顶部的“**保存**”按钮保存配置。

#### <a name="configure-linux-performance-counters-in-configuration-file"></a>在配置文件中配置 Linux 性能计数器
可以不使用 Azure 门户配置 Linux 性能计数器，而是在 Linux 代理上编辑配置文件。  要收集的性能指标由 **/etc/opt/microsoft/omsagent/\<workspace id\>/conf/omsagent.conf** 中的配置进行控制。

要收集的性能指标的每个对象或类别应在配置文件中作为单个 `<source>` 元素进行定义。 语法遵循下面的模式。

    <source>
      type oms_omi  
      object_name "Processor"
      instance_regex ".*"
      counter_name_regex ".*"
      interval 30s
    </source>


下表介绍了此元素中的参数。

| parameters | 描述 |
|:--|:--|
| object\_name | 收集的对象名称。 |
| instance\_regex |  用于定义要收集的实例的*正则表达式*。 值 `.*` 指定所有实例。 要仅收集 \_Total 实例的处理器指标，可以指定 `_Total`。 要仅收集 crond 或 sshd 实例的进程指标，可以指定 `(crond\|sshd)`。 |
| counter\_name\_regex | 用于定义要收集的对象计数器的*正则表达式*。 要收集对象的所有计数器，请指定：`.*`。 例如，要仅收集内存对象的交换空间计数器，可以指定 `.+Swap.+` |
| interval | 收集对象计数器时采用的频率。 |


下表列出了可以在配置文件中指定的对象和计数器。  如[在 Azure Monitor 中收集 Linux 应用程序的性能计数器](data-sources-linux-applications.md)中所述，对于某些应用程序，还有其他计数器可用。

| 对象名称 | 计数器名称 |
|:--|:--|
| 逻辑磁盘 | 可用 Inode 百分比 |
| 逻辑磁盘 | 可用空间百分比 |
| 逻辑磁盘 | 已用 Inode 百分比 |
| 逻辑磁盘 | 已用空间百分比 |
| 逻辑磁盘 | 磁盘读取的字节数/秒 |
| 逻辑磁盘 | 磁盘读取数/秒 |
| 逻辑磁盘 | 磁盘传输数/秒 |
| 逻辑磁盘 | 磁盘写入的字节数/秒 |
| 逻辑磁盘 | 磁盘写入数/秒 |
| 逻辑磁盘 | 可用 MB 数 |
| 逻辑磁盘 | 逻辑磁盘字节数/秒 |
| 内存 | 可用内存百分比 |
| 内存 | 可用交换空间百分比 |
| 内存 | 已用内存百分比 |
| 内存 | 已用交换空间百分比 |
| 内存 | 可用内存 MB 数 |
| 内存 | 可用交换空间 MB 数 |
| 内存 | 页面读取数/秒 |
| 内存 | 页面写入数/秒 |
| 内存 | 页面数/秒 |
| 内存 | 已用交换空间 MB 数 |
| 内存 | 已用内存 MB 数 |
| 网络 | 已传输的字节数总计 |
| 网络 | 已接收的字节数总计 |
| 网络 | 字节数总计 |
| 网络 | 已传输的包数总计 |
| 网络 | 已接收的包数总计 |
| 网络 | Rx 错误数总计 |
| 网络 | Tx 错误数总计 |
| 网络 | 冲突数总计 |
| 物理磁盘 | 平均值磁盘秒数/读取 |
| 物理磁盘 | 平均值磁盘秒数/传输 |
| 物理磁盘 | 平均值磁盘秒数/写入 |
| 物理磁盘 | 物理磁盘字节数/秒 |
| 进程 | 特权时间百分比 |
| 进程 | 用户时间百分比 |
| 进程 | 已用内存 KB 数 |
| 进程 | 虚拟共享内存 |
| 处理器 | DPC 时间百分比 |
| 处理器 | 空闲时间百分比 |
| 处理器 | 中断时间百分比 |
| 处理器 | IO 等待时间百分比 |
| 处理器 | 良好时间百分比 |
| 处理器 | 特权时间百分比 |
| 处理器 | 处理器时间百分比 |
| 处理器 | 用户时间百分比 |
| 系统 | 可用物理内存 |
| 系统 | 分页文件中的可用空间 |
| 系统 | 可用虚拟内存 |
| 系统 | 进程 |
| 系统 | 分页文件中存储的大小 |
| 系统 | 运行时间 |
| 系统 | 用户 |


下面是性能指标的默认配置。

    <source>
      type oms_omi
      object_name "Physical Disk"
      instance_regex ".*"
      counter_name_regex ".*"
      interval 5m
    </source>

    <source>
      type oms_omi
      object_name "Logical Disk"
      instance_regex ".*
      counter_name_regex ".*"
      interval 5m
    </source>

    <source>
      type oms_omi
      object_name "Processor"
      instance_regex ".*
      counter_name_regex ".*"
      interval 30s
    </source>

    <source>
      type oms_omi
      object_name "Memory"
      instance_regex ".*"
      counter_name_regex ".*"
      interval 30s
    </source>

## <a name="data-collection"></a>数据收集
Azure Monitor 以指定的采样间隔在已安装相应计数器的所有代理上收集所有指定的性能计数器。  数据未聚合，原始数据在订阅指定的持续时间内在所有日志查询视图中提供。

## <a name="performance-record-properties"></a>性能记录属性
性能记录具有 **Perf** 类型，并且具有下表中的属性。

| 属性 | 描述 |
|:--- |:--- |
| Computer |从中收集事件的计算机。 |
| CounterName |性能计数器的名称 |
| CounterPath |性能计数器的完整路径，以 \\\\\<Computer>\\object(instance)\\counter 格式显示。 |
| CounterValue |计数器的数值。 |
| InstanceName |事件实例的名称。  无实例时为空。 |
| ObjectName |性能对象的名称 |
| SourceSystem |从中收集数据的代理类型。 <br><br>OpsManager – Windows 代理，直接连接或通过 SCOM 连接 <br> Linux - 所有 Linux 代理  <br> AzureStorage – Azure 诊断 |
| TimeGenerated |对数据进行采样的日期和时间。 |

## <a name="sizing-estimates"></a>大小估计值
 以 10 秒间隔收集特定计数器的粗略估计值约为每个实例每天 1 MB。  可以使用以下公式估计特定计数器的存储要求。

    1 MB x (number of counters) x (number of agents) x (number of instances)

## <a name="log-queries-with-performance-records"></a>使用性能记录的日志查询
下表提供了检索性能记录的不同日志查询的示例。

| Query | 描述 |
|:--- |:--- |
| 性能 |所有性能数据 |
| Perf &#124; where Computer == "MyComputer" |特定计算机中的所有性能数据 |
| Perf &#124; where CounterName == "Current Disk Queue Length" |特定计数器的所有性能数据 |
| 性能&#124;其中 ObjectName = ="Processor"and CounterName = ="%Processor Time"和 InstanceName = ="_Total"&#124;汇总 AVGCPU = avg （countervalue） 的计算机 |所有计算机的平均 CPU 使用率 |
| 性能&#124;其中 CounterName = ="%Processor Time"&#124;汇总 AggregatedValue = max(CounterValue) 计算机 |所有计算机的最大 CPU 使用率 |
| Perf &#124; where ObjectName == "LogicalDisk" and CounterName == "Current Disk Queue Length" and Computer == "MyComputerName" &#124; summarize AggregatedValue = avg(CounterValue) by InstanceName |指定计算机的所有实例上的当前磁盘队列平均长度 |
| Perf &#124; where CounterName == "Disk Transfers/sec" &#124; summarize AggregatedValue = percentile(CounterValue, 95) by Computer |每秒所有计算机上磁盘传输的第 95 百分位数 |
| Perf &#124; where CounterName == "% Processor Time" and InstanceName == "_Total" &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), Computer |每小时所有计算机 CPU 使用率的平均值 |
| Perf &#124; where Computer == "MyComputer" and CounterName startswith_cs "%" and InstanceName == "_Total" &#124; summarize AggregatedValue = percentile(CounterValue, 70) by bin(TimeGenerated, 1h), CounterName | 每小时特定计算机的每个 % 百分比计数器的第 70 百分位数 |
| Perf &#124; where CounterName == "% Processor Time" and InstanceName == "_Total" and Computer == "MyComputer" &#124; summarize ["min(CounterValue)"] = min(CounterValue), ["avg(CounterValue)"] = avg(CounterValue), ["percentile75(CounterValue)"] = percentile(CounterValue, 75), ["max(CounterValue)"] = max(CounterValue) by bin(TimeGenerated, 1h), Computer |每小时特定计算机的 CPU 使用率的平均值、最小值、最大值和第 75 百分位数 |
| Perf &#124; where ObjectName == "MSSQL$INST2:Databases" and InstanceName == "master" | 所有性能数据来自命名 SQL Server 实例 INST2 的 master 数据库的数据库性能对象。  




## <a name="next-steps"></a>后续步骤
* [从 Linux 应用程序收集性能计数器](data-sources-linux-applications.md)，包括 MySQL 和 Apache HTTP Server。
* 了解[日志查询](../log-query/log-query-overview.md)以便分析从数据源和解决方案中收集的数据。  
* 将收集的数据导出到 [Power BI](powerbi.md) 以进行其他可视化操作和分析。
