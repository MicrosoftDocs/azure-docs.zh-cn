---
title: Azure 状态监视器 v2 API 参考：启用检测引擎 |Microsoft Docs
description: 状态监视器 v2 API 引用启用-InstrumentationEngine。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
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
ms.openlocfilehash: d886aa364ca928d32100c570689f13beb0c682c9
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143421"
---
# <a name="status-monitor-v2-api-enable-instrumentationengine-v021-alpha"></a>状态监视器 v2 API:启用 InstrumentationEngine (v0.2.1-alpha)

本文档介绍了作为的成员提供的 cmdlet [Az.ApplicationMonitor PowerShell 模块](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/)。

> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="description"></a>描述

此 cmdlet 将通过设置一些注册表项启用检测引擎。
重新启动 IIS 以实现这些更改才会生效。

检测引擎可以补充由.NET Sdk 收集的数据。
事件和消息，将收集描述托管进程的执行。 包括但不是限于对依赖项结果代码、 HTTP 谓词和 SQL 命令文本。 

如果，启用检测引擎：
- 使用启用 cmdlet 监视已启用但未启用 InstrumentationEngine。
- 你已手动检测应用程序使用.NET Sdk 并想要收集的其他遥测数据。

> [!IMPORTANT] 
> 此 cmdlet 需要具有管理员权限的 PowerShell 会话。

> [!NOTE] 
> 此 cmdlet 将要求您查看并接受我们的许可协议和隐私声明语句。

> [!NOTE] 
> 检测引擎会增加系统开销，并默认处于关闭状态。

## <a name="examples"></a>示例

```powershell
PS C:\> Enable-InstrumentationEngine
```

## <a name="parameters"></a>parameters 

### <a name="-acceptlicense"></a>-AcceptLicense
**可选。** 使用此开关以接受许可协议和隐私声明语句在无外设安装中。

### <a name="-verbose"></a>-Verbose
**通用参数。** 使用此开关以输出详细的日志。

## <a name="output"></a>输出


#### <a name="example-output-from-successfully-enabling-the-instrumentation-engine"></a>已成功启用检测引擎的示例输出

```
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
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
 - [获取配置](status-monitor-v2-api-get-config.md)以确认已正确记录你的设置。
 - [获取状态](status-monitor-v2-api-get-status.md)检查监视。
