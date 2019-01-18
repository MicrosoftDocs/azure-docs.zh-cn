---
title: Azure Monitor 中的数据的来源 | Microsoft Docs
description: 介绍用于监视 Azure 资源及其上运行的应用程序的运行状况和性能的数据。
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/13/2018
ms.author: bwren
ms.openlocfilehash: 39229bbd120ca52f31a0bd54446e49990d952c9d
ms.sourcegitcommit: 25936232821e1e5a88843136044eb71e28911928
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/04/2019
ms.locfileid: "54020126"
---
# <a name="sources-of-data-in-azure-monitor"></a>Azure Monitor 中的数据的来源
本文介绍了由 Azure Monitor 收集的用于监视资源及其上运行的应用程序的运行状况和性能的数据的来源。 这些资源可能在 Azure 中，在其他云中或在本地。  有关此数据如何存储以及如何查看它的详细信息，请参阅 [Azure Monitor 收集的数据](data-collection.md)。

Azure 中的监视数据来自各种来源，这些来源可以组织为层，最高的层是你的应用程序和任何操作系统，较低的层是 Azure 平台的组件。 下图对此进行了描述，每个层在以下各部分详细说明。

![监视数据的层](media/data-sources/monitoring-tiers.png)

## <a name="azure-tenant"></a>Azure 租户
与你的 Azure 租户相关的遥测数据是从租户级服务（例如 Azure Active Directory）中收集的。

![Azure 租户集合](media/data-sources/tenant-collection.png)

### <a name="azure-active-directory-audit-logs"></a>Azure Active Directory 审核日志
[Azure Active Directory 报告](../../active-directory/reports-monitoring/overview-reports.md)包含特定租户中的登录活动历史记录和更改审核日志。 可以将这些审核日志写入到 Azure Monitor 日志中，以便将其与其他日志数据一起进行分析。


## <a name="azure-platform"></a>Azure 平台
与 Azure 本身的运行状况和操作相关的遥测数据包含 Azure 订阅的操作和管理数据。 它包括存储在 Azure 活动日志中的服务运行状况数据，以及 Azure Active Directory 中的审核日志。

![Azure 订阅集合](media/data-sources/azure-collection.png)

### <a name="azure-service-health"></a>Azure 服务运行状况
[Azure 服务运行状况](../../azure-monitor/platform/service-notifications.md)提供订阅中应用程序和资源所依赖的 Azure 服务的运行状况信息。 可以根据当前的以及预期的可能会影响应用程序的关键问题创建警报，以便到时获得通知。 服务运行状况记录存储在 [Azure 活动日志](../../azure-monitor/platform/activity-logs-overview.md)中，你可以在活动日志资源管理器中查看，并将其复制到 Azure Monitor 日志中。

### <a name="azure-activity-log"></a>Azure 活动日志
[Azure 活动日志](../../azure-monitor/platform/activity-logs-overview.md)包含服务运行状况记录，以及对 Azure 资源所做的配置更改的记录。 活动日志可供所有 Azure 资源使用，代表其 _外部_ 视图。 活动日志中记录的具体类型在 [Azure 活动日志事件架构](../../azure-monitor/platform/activity-log-schema.md)中介绍。

可以在 Azure 门户中特定资源的页面上查看其活动日志，或在[活动日志资源管理器](../../azure-monitor/platform/activity-logs-overview.md)中的多个资源处查看日志。 可以将日志项目复制到 Azure Monitor 中，与其他监视数据组合在一起，这特别有用。 也可使用[事件中心](../../azure-monitor/platform/activity-logs-stream-event-hubs.md)将其发送到其他位置。



## <a name="azure-services"></a>Azure 服务
指标和资源级诊断日志提供 Azure 资源的 _内部_ 操作的信息。 这些适用于大多数 Azure 服务。管理解决方案提供特定服务的更多见解。

![Azure 资源收集](media/data-sources/azure-resource-collection.png)


### <a name="metrics"></a>度量值
大多数 Azure 服务会生成反映其性能和运行的[平台指标](data-collection.md#metrics)。 具体[指标因资源类型而异](../../azure-monitor/platform/metrics-supported.md)。  这些指标可以在指标资源管理器中访问，并且可以复制到 Log Analytics 中进行趋势推断和其他分析。


### <a name="resource-diagnostic-logs"></a>资源诊断日志
活动日志提供在 Azure 资源上执行的操作的信息，而资源级[诊断日志](../../azure-monitor/platform/diagnostic-logs-overview.md)则可用于了解资源本身的操作。   这些日志的配置要求和内容[因资源类型而异](../../azure-monitor/platform/tutorial-dashboards.md)。

不能直接在 Azure 门户中查看诊断日志，但可以[将它们发送到 Azure 存储进行存档](../../azure-monitor/platform/archive-diagnostic-logs.md)，然后将它们导出到[事件中心](../../event-hubs/event-hubs-about.md)以便重定向到其他服务，或者导出[到 Log Analytics](../../azure-monitor/platform/diagnostic-logs-stream-log-store.md) 进行分析。 某些资源可以直接写入到 Log Analytics 中，而另一些资源则只能先写入到存储帐户，然后再[导入 Log Analytics 中](../../azure-monitor/platform/azure-storage-iis-table.md#use-the-azure-portal-to-collect-logs-from-azure-storage)。

### <a name="monitoring-solutions"></a>监视解决方案
 [监视解决方案](../insights/solutions.md)通过收集数据来提供针对特定服务或应用程序的更多操作见解。 这些解决方案将数据收集到 Azure Monitor 日志中，以便在其中使用[查询语言](../log-query/log-query-overview.md)或通常包括在解决方案中的[视图](view-designer.md)对这些数据进行分析。

## <a name="guest-operating-system"></a>来宾操作系统
Azure 中的、其他云中的以及本地的计算资源都有要监视的来宾操作系统。 安装一个或多个代理以后，即可将来宾的遥测数据收集到与 Azure 服务本身相同的监视工具中。

![Azure 计算资源收集](media/data-sources/compute-resource-collection.png)

### <a name="azure-diagnostic-extension"></a>Azure 诊断扩展
使用 Azure 诊断扩展，可以从 Azure 计算资源的客户端操作系统收集日志和性能数据，从而提供基本级别的监视。   

### <a name="log-analytics-agent"></a>Log Analytics 代理
Windows 或 Linux 虚拟机或物理计算机的全面监视和管理是通过 Log Analytics 代理提供的。 虚拟机可以在 Azure 中、其他云中或在本地运行，代理直接或通过 System Center Operations Manager 连接到 Azure Monitor，并允许从你配置的[数据源](agent-data-sources.md)收集数据，或者从[监视解决方案](../insights/solutions.md)收集数据，监视解决方案提供有关在虚拟机上运行的应用程序的更多见解。

### <a name="dependency-agent"></a>依赖关系代理
[服务映射](../insights/service-map.md)和[用于 VM 的 Azure Monitor](../insights/vminsights-overview.md) 要求在 Windows 和 Linux 虚拟机上有一个依赖项代理。 它集成了 Log Analytics 代理，用于收集在虚拟机上运行的进程的已发现数据，以及外部进程依赖项的已发现数据。 它将此数据存储在 Azure Monitor 中，并将发现的互连组件可视化。  

若要进一步了解各个代理之间的差异以及根据你的监视要求应使用哪些代理，请参阅[监视代理概述](agents-overview.md)。

## <a name="applications"></a>应用程序
除了可以通过应用程序将遥测数据写入到来宾操作系统中，还可以通过 [Application Insights](https://docs.microsoft.com/azure/application-insights/) 进行详细的应用程序监视。 Application Insights 可以从各种平台上运行的应用程序处收集数据。 此应用程序可以在 Azure 中运行，也可以在其他云中或本地运行。

![应用程序数据收集](media/data-sources/application-collection.png)


### <a name="application-data"></a>应用程序数据
通过安装检测包为应用程序启用 Application Insights 后，它就会收集与应用程序的性能和运行相关的指标 和日志。 这包括有关页面视图、应用程序请求和异常的详细信息。 Application Insights 将它收集的数据存储在 Azure Monitor 中。 它包括各种用于分析此数据的工具，但是也可使用指标分析和日志分析之类的工具分析来自其他源的数据。

也可使用 Application Insights [创建自定义指标](../../azure-monitor/app/api-custom-events-metrics.md)。  这样就可以定义你自己的逻辑，以便计算出一个数字值，然后将该值与其他可以从指标资源管理器访问的指标一起存储，用于[自动缩放](../../azure-monitor/platform/autoscale-custom-metric.md)和指标警报。

### <a name="dependencies"></a>依赖项
若要监视应用程序的不同逻辑操作，必须[跨多个组件收集遥测数据](../../azure-monitor/app/transaction-diagnostics.md)。 Application Insights 支持[分布式遥测关联](../../azure-monitor/app/correlation.md)，该关联用于确定组件之间的依赖关系，这样就可以将它们放在一起进行分析。

### <a name="availability-tests"></a>可用性测试
使用 Application Insights 中的[可用性测试](../../azure-monitor/app/monitor-web-app-availability.md)，可以从公共 Internet 的不同位置测试应用程序的可用性和响应能力。 可以通过简单的 ping 测试来验证应用程序是否处于活动状态，也可以使用 Visual Studio 创建一个模拟用户方案的 Web 测试。  可用性测试不需应用程序中有任何检测。

## <a name="custom-sources"></a>自定义来源
除了应用程序的标准层之外，还可能需要监视具有不能与其他数据源一起收集的遥测数据的其他资源。 对于这些资源，你需要使用 Azure Monitor API 为此数据进行编码。

![自定义收集数据](media/data-sources/custom-collection.png)

### <a name="data-collector-api"></a>数据收集器 API
Azure Monitor 可以使用[数据收集器 API](data-collector-api.md) 从任何 REST 客户端收集日志数据。 这样，便可以创建自定义监视方案，并将监视范围扩展到不通过其他源公开遥测数据的资源。

## <a name="next-steps"></a>后续步骤

- 详细了解 [Azure Monitor 收集的监视数据的类型](data-collection.md)以及如何查看和分析此数据。
