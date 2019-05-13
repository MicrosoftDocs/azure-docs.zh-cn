---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 01/27/2019
ms.author: cynthn
ms.openlocfilehash: ac400c86af8236ff5d67b8b6fbf99f6f4b1d36c9
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65404902"
---
通过收集、查看和分析诊断与日志数据，可以利用很多机会来监视 VM。 若要执行简单的 VM [监视](../articles/azure-monitor/overview.md)，可以在 Azure 门户中使用 VM 的“概述”屏幕。 可以使用[扩展](../articles/virtual-machines/windows/extensions-features.md)配置 VM 的诊断以收集更多指标数据。 还可以使用更多高级监视选项，如 [Application Insights](../articles/azure-monitor/app/app-insights-overview.md) 和 [Log Analytics](../articles/azure-monitor/log-query/log-query-overview.md)。

## <a name="diagnostics-and-metrics"></a>诊断和指标 

可以在 Azure 门户、Azure CLI、Azure PowerShell 和编程应用程序编程接口 (API) 中使用[指标](../articles/monitoring-and-diagnostics/monitoring-overview-metrics.md)来设置和监视[诊断数据](https://docs.microsoft.com/cli/azure/vm/diagnostics)收集。 例如，可以：

- **观察 VM 的基本指标。** Azure 门户的“概述”屏幕上显示的基本指标包括 CPU 使用率、网络使用情况、总磁盘字节数以及每秒的磁盘操作数。

- **使用 Azure 门户启用并查看启动诊断数据收集。** 将自己的映像加载到 Azure 或者启动某个平台映像时，可能会因为许多原因而导致 VM 进入无法启动状态。 创建 VM 时，针对“设置”屏幕的“监视”部分下的“启动诊断”单击“已启用”，即可轻松启用启动诊断。

    VM 启动时，启动诊断代理将捕获启动输出并将其存储在 Azure 存储中。 此数据可以用于排查 VM 启动问题。 从命令行工具创建 VM 时，不会自动启用启动诊断。 在启用启动诊断之前，需要创建一个存储帐户来存储启动日志。 如果在 Azure 门户中启用启动诊断，则会自动创建一个存储帐户。

    如果未在创建 VM 时启用启动诊断，可在以后随时使用 [Azure CLI](https://docs.microsoft.com/cli/azure/vm/boot-diagnostics)、[Azure PowerShell](https://docs.microsoft.com/powershell/module/az.compute/set-azvmbootdiagnostic) 或 [Azure 资源管理器模板](../articles/virtual-machines/windows/extensions-diagnostics-template.md)启用它。

- **启用来宾 OS 诊断数据收集。** 创建 VM 时，可以在“设置”屏幕上启用来宾 OS 诊断。 如果确实启用了诊断数据收集，[用于 Linux 的 IaaSDiagnostics 扩展](../articles/virtual-machines/linux/diagnostic-extension.md)或[用于 Windows 的 IaaSDiagnostics 扩展](../articles/virtual-machines/windows/ps-extensions-diagnostics.md)将添加到 VM，使你可以收集更多的磁盘、CPU 和内存数据。

    使用收集的诊断数据，可以为 VM 配置自动缩放。 还可以配置日志，以便存储数据并设置警报，从而在性能不正常时通知你。

## <a name="alerts"></a>警报

可以根据特定的性能指标创建[警报](../articles/azure-monitor/platform/alerts-overview.md)。 举例来说，可以根据以下问题生成警报，平均 CPU 使用率超过特定的阈值，或者可用磁盘空间低于特定的空间量。 可以在 [Azure 门户](../articles/azure-monitor/platform/alerts-classic-portal.md)中或者使用 [Azure PowerShell](../articles/azure-monitor/platform/alerts-classic-portal.md#with-powershell) 或 [Azure CLI](../articles/azure-monitor/platform/alerts-classic-portal.md#with-azure-cli) 来配置警报。

## <a name="azure-service-health"></a>Azure 服务运行状况

[Azure 服务运行状况](../articles/service-health/service-health-overview.md)会在 Azure 服务问题影响你时提供个性化的指导和支持，并且会帮助你为即将到来的计划内维护做好准备。 Azure 服务运行状况使用具有针对性和灵活性的通知提醒你和你的团队。

## <a name="azure-resource-health"></a>Azure 资源运行状况

[Azure 资源运行状况](../articles/service-health/resource-health-overview.md)有助于在 Azure 问题影响资源时进行诊断和获取支持。 它通知你有关资源的当前和过去运行状况的信息，并帮助你缓解问题。 在需要有关 Azure 服务问题的帮助时，资源运行状况将提供技术支持。

## <a name="azure-activity-log"></a>Azure 活动日志

[Azure 活动日志](../articles/azure-monitor/platform/activity-logs-overview.md)是一种方便用户深入了解 Azure 中发生的订阅级别事件的订阅日志。 该日志包括从 Azure 资源管理器操作数据到服务运行状况事件更新的一系列数据。 可以在 Azure 门户中单击“活动日志”查看 VM 的日志。

可以对活动日志执行的部分操作包括：

- [根据活动日志事件创建警报](../articles/azure-monitor/platform/activity-logs-overview.md)。
- [将活动日志流式传输到事件中心](../articles/azure-monitor/platform/activity-logs-stream-event-hubs.md)，方便第三方服务或自定义分析解决方案（例如 PowerBI）引入。
- 在 PowerBI 中使用 [PowerBI 内容包](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs/)分析活动日志。
- [将活动日志保存到存储帐户](../articles/azure-monitor/platform/archive-activity-log.md)进行存档或手动检查。 可以使用“日志配置文件”指定保留时间（天）。

还可以通过使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.insights/)、[Azure CLI](https://docs.microsoft.com/cli/azure/monitor) 或[监视 REST API](https://docs.microsoft.com/rest/api/monitor/) 访问活动日志数据。

[Azure 诊断日志](../articles/azure-monitor/platform/diagnostic-logs-overview.md)是 VM 发出的日志，其中提供与该 VM 的操作相关的各种频繁生成的数据。 不同于活动日志，诊断日志提供有关在 VM 中执行的操作的见解。

可以对诊断日志执行的部分操作包括：

- 将诊断日志保存到[存储帐户](../articles/azure-monitor/platform/archive-diagnostic-logs.md)进行审核或手动检查。 可以使用“资源诊断设置”指定保留时间（天）。
- [将诊断日志流式传输到事件中心](../articles/azure-monitor/platform/diagnostic-logs-stream-event-hubs.md)，方便第三方服务或自定义分析解决方案（例如 PowerBI）引入。
- 使用 [Log Analytics](../articles/log-analytics/log-analytics-azure-storage.md) 对诊断日志进行分析。

## <a name="advanced-monitoring"></a>高级监视

- [Azure Monitor](../articles/azure-monitor/overview.md) 是一个服务，用于监视云和本地环境，使其保持较高的可用性和性能。 它提供了一个全面的解决方案，用于从云和本地环境收集、分析和处理遥测数据。 它可以帮助你了解应用程序的性能，并主动识别影响应用程序及其所依赖资源的问题。 可以在 [Linux VM](../articles/virtual-machines/linux/extensions-oms.md) 或 [Windows VM](../articles/virtual-machines/windows/extensions-oms.md) 上安装一个用于安装 Log Analytics 代理的扩展，以便收集日志数据并将其存储在 Log Analytics 工作区中。

    对于 Windows 和 Linux VM，建议安装 Log Analytics 代理来收集日志。 在 VM 上安装 Log Analytics 代理的最简单方法是通过 [Log Analytics VM 扩展](../articles/log-analytics/log-analytics-azure-vm-extension.md)安装。 使用扩展可简化安装流程，并可自动配置代理，以将数据发送至指定的 Log Analytics 工作区。 代理还会自动升级，以确保拥有最新的功能和修补程序。

- 使用[网络观察程序](../articles/network-watcher/network-watcher-monitoring-overview.md)可以在 VM 及其关联资源与所在网络相关时对其进行监视。 可以在 [Linux VM](../articles/virtual-machines/linux/extensions-nwa.md) 或 [Windows VM](../articles/virtual-machines/windows/extensions-nwa.md) 上安装网络观察程序代理扩展。

- [用于 VM 的 Azure Monitor](../articles/azure-monitor/insights/vminsights-overview.md) 分析 Windows 和 Linux VM 的性能与运行状况，包括其不同的进程以及与其他资源和外部进程之间的相互依赖关系，可以大规模监视 Azure 虚拟机 (VM)。 

## <a name="next-steps"></a>后续步骤
- 逐步完成[使用 Azure PowerShell 监视 Windows 虚拟机](../articles/virtual-machines/windows/tutorial-monitoring.md)或[使用 Azure CLI 监视 Linux 虚拟机](../articles/virtual-machines/linux/tutorial-monitoring.md)中的步骤。
- 了解更多有关[监视和诊断](https://docs.microsoft.com/azure/architecture/best-practices/monitoring)的最佳做法的信息。
