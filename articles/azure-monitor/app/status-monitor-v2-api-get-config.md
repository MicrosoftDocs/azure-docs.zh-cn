---
title: Azure 状态监视器 v2 API 参考：获取配置 |Microsoft Docs
description: 状态监视器 v2 API 引用 Get-ApplicationInsightsMonitoringConfig。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
services: application-insights
documentationcenter: .net
author: MS-TimothyMothra
manager: alexklim
ms.assetid: 769a5ea4-a8c6-4c18-b46c-657e864e24de
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: tilee
ms.openlocfilehash: 28893e0782a7fa928757bbbba42662e6d3fa272c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153587"
---
# <a name="status-monitor-v2-api-get-applicationinsightsmonitoringconfig-v021-alpha"></a>状态监视器 v2 API:Get-ApplicationInsightsMonitoringConfig (v0.2.1-alpha)

本文档介绍了作为的成员提供的 cmdlet [Az.ApplicationMonitor PowerShell 模块](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/)。

> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="description"></a>描述

获取配置文件和打印到控制台的值。

> [!IMPORTANT] 
> 此 cmdlet 需要具有管理员权限的 PowerShell 会话。

## <a name="examples"></a>示例

```powershell
PS C:\> Get-ApplicationInsightsMonitoringConfig
```

## <a name="parameters"></a>parameters 

（没有所需的参数）

## <a name="output"></a>输出


#### <a name="example-output-from-reading-the-config-file"></a>读取配置文件的示例输出

```
RedfieldConfiguration:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
```

## <a name="next-steps"></a>后续步骤

  查看遥测：
 - [浏览指标](../../azure-monitor/app/metrics-explorer.md)，以便监视性能和使用情况
- [搜索事件和日志](../../azure-monitor/app/diagnostic-search.md)来诊断问题
- [分析](../../azure-monitor/app/analytics.md)，以便进行更高级的查询
- [创建仪表板](../../azure-monitor/app/app-insights-dashboards.md)
 
 添加更多遥测：
 - [创建 web 测试](monitor-web-app-availability.md)以确保你的站点保持活动状态。
- [添加 web 客户端遥测](../../azure-monitor/app/javascript.md)，查看网页代码中的异常，并将其插入跟踪调用。
- [将 Application Insights SDK 添加到你的代码](../../azure-monitor/app/asp-net.md)，以便您可以插入跟踪和日志调用
 
 用做更多状态监视器 v2:
 - 使用指南，了解如何[疑难解答](status-monitor-v2-troubleshoot.md)状态监视器 v2。
 - 此配置使用对进行的更改[设置配置](status-monitor-v2-api-set-config.md)cmdlet。
