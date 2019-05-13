---
title: Azure 监视代理概述 | Microsoft Docs
description: 本文提供可用于在 Azure 或混合环境中支持监视虚拟机的 Azure 代理的详细概述。
services: azure-monitor
documentationcenter: azure-monitor
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 11/14/2018
ms.author: magoedte
ms.openlocfilehash: 58abe3a3973986ab489456be7958361ad8ab06f4
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64922830"
---
# <a name="overview-of-the-azure-monitoring-agents"></a>Azure 监视代理概述 
Microsoft Azure 提供多种方式用于从 Azure、数据中心或其他云提供商托管的，运行 Microsoft Windows 和 Linux 的虚拟机收集各种数据。 可用于监视 VM 的三种代理包括：

* Azure 诊断扩展
* 适用于 Linux 和 Windows 的 Log Analytics 代理
* 依赖关系代理

本文将会描述这些代理的差异及其功能，帮助你确定哪种代理能够为 IT 服务管理或一般性监视要求提供支持。  

## <a name="azure-diagnostic-extension"></a>Azure 诊断扩展
自 2010 年推出正式版以来，针对 Azure 云服务提供的 [Azure 诊断扩展](../../azure-monitor/platform/diagnostics-extension-overview.md)（通常称作 Windows Azure 诊断 (WAD) 扩展或 Linux Azure 诊断 (LAD) 扩展）代理以简单的方式从 Azure 计算资源（例如 VM）收集诊断数据，并将这些数据保存在 Azure 存储中。 将数据保存到存储中后，可以选择使用多种可用工具之一（例如 [Visual Studio 中的服务器资源管理器](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage)和 [Azure 存储资源管理器](../../vs-azure-tools-storage-manage-with-storage-explorer.md)）查看这些数据。

可以选择收集：

* 一组预定义的操作系统性能计数器和事件日志，或者指定要收集的数据。 
* 针对 IIS Web 服务器发出的所有请求和/或失败的请求
* .NET 应用程序跟踪输出日志
* Windows 事件跟踪 (ETW) 事件 
* 从 syslog 收集日志事件  
* 故障转储 

如果存在以下需求，应使用 Azure 诊断代理：

* 将日志和指标存档到 Azure 存储
* 将监视数据与第三方工具集成。 这些工具使用各种方法，包括查询存储帐户、转发到[事件中心](../../event-hubs/event-hubs-about.md)，或使用 [Azure 监视 REST API](../../azure-monitor/platform/rest-api-walkthrough.md) 进行查询
* 将数据上传到 Azure Monitor 以便在 Azure 门户中创建指标图表，或创建近实时的[指标警报](../../azure-monitor/platform/alerts-metric-overview.md)。 
* 基于来宾 OS 指标自动缩放虚拟机规模集和经典云服务。
* 使用[启动诊断](../../virtual-machines/troubleshooting/boot-diagnostics.md)调查 VM 启动问题。
* 使用 [Application Insights](../../azure-monitor/overview.md) 了解应用程序的性能，并主动识别影响应用程序的问题。
* 配置 Azure Monitor，导入指标和日志数据从云服务，经典 Vm，收集并存储在 Azure 存储帐户中 Service Fabric 节点。

## <a name="log-analytics-agent"></a>Log Analytics 代理
对于高级监视需要多个收集指标和日志的子集，则需要 （也简称为 Microsoft Monitoring Agent (MMA)） 的 Windows 和 Linux 的 Log Analytics 代理。 Log Analytics 代理旨在对本地物理机和虚拟机、System Center Operations Manager 监视的计算机以及其他云中托管的 VM 进行全面的管理。 Windows 和 Linux 代理连接到 Azure Monitor 收集监视基于解决方案的数据以及你配置的自定义数据源中的 Log Analytics 工作区。

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

如果存在以下需求，应使用 Log Analytics 代理：

* 从 Azure、其他云提供商和本地资源中的各种源收集数据。 
* 使用某种 Azure Monitor 监视解决方案，例如[用于 VM 的 Azure Monitor](../insights/vminsights-overview.md)、[用于容器的 Azure Monitor](../insights/container-insights-overview.md)，等等。  
* 使用其他某个 Azure 管理服务，例如 [Azure 安全中心](../../security-center/security-center-intro.md)、[Azure 自动化](../../automation/automation-intro.md)，等等。

以前，*Operations Management Suite* 由多个 Azure 服务捆绑而成，因此，Log Analytics 代理在 Azure 安全中心和 Azure 自动化等服务之间共享。  此套件包含这些服务提供的整套功能，可在 Azure VM 的整个生命周期内为其提供全面的管理。  这些功能的示例包括：

* 对操作系统更新进行 [Azure 自动化更新管理](../../automation/automation-update-management.md)。
* 使用 [Azure 自动化 Desired State Configuration](../../automation/automation-dsc-overview.md) 来保持一致的配置状态。
* 使用 [Azure 自动化更改跟踪和库存](../../automation/change-tracking.md)来跟踪配置更改。
* Azure 服务（例如 [Application Insights](https://docs.microsoft.com/azure/application-insights/) 和 [Azure 安全中心](https://docs.microsoft.com/azure/security-center/)）原生就会将其数据直接存储在 Log Analytics 中。  

## <a name="dependency-agent"></a>依赖关系代理
依赖关系代理是作为服务映射解决方案，最初并非由 Microsoft 开发的一部分开发的。 [服务映射](../insights/service-map.md)并[Vm 的 Azure Monitor](../insights/vminsights-overview.md)需要依赖关系代理在 Windows 和 Linux 上的虚拟机并进行了集成与 Log Analytics 代理来收集有关在虚拟运行的进程发现的数据计算机和外部进程依赖关系。 它将此数据存储在 Log Analytics 工作区中，并直观显示已发现的互连的组件。

可能需要以某种形式组合使用这些代理才能监视 VM。 这些代理可与 Azure 扩展并列安装，但在 Linux 上，必须先安装 Log Analytics 代理，否则安装将会失败。 

## <a name="next-steps"></a>后续步骤

- 请参阅 [Log Analytics 代理概述](../../azure-monitor/platform/log-analytics-agent.md)，了解将代理部署到 Auzre、数据中心或其他云环境中托管的计算机所要满足的要求和支持的方法。

