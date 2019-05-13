---
title: Azure 状态监视器 v2 API 参考：设置配置 |Microsoft Docs
description: 状态监视器 v2 API 引用集-ApplicationInsightsMonitoringConfig。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
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
ms.openlocfilehash: 953edcb98de6ea705721aef0922562d23e18f0f5
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148228"
---
# <a name="status-monitor-v2-api-set-applicationinsightsmonitoringconfig-v021-alpha"></a>状态监视器 v2 API:Set-ApplicationInsightsMonitoringConfig (v0.2.1-alpha)

本文档介绍了作为的成员提供的 cmdlet [Az.ApplicationMonitor PowerShell 模块](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/)。

> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="description"></a>描述

设置配置文件而无需重复完全重新安装。 重新启动 IIS 以实现所做的更改才会生效。

> [!IMPORTANT] 
> 此 cmdlet 需要具有管理员权限的 PowerShell 会话。


## <a name="examples"></a>示例

### <a name="example-with-single-instrumentation-key"></a>具有单个检测密钥的示例
在此示例中，在当前计算机上的所有应用程序将分配单个检测密钥。

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### <a name="example-with-instrumentation-key-map"></a>使用检测键映射示例
在此示例中， 
- `MachineFilter` 将当前的计算机使用的匹配`'.*'`通配符。
- `AppFilter='WebAppExclude'` 提供了`null`InstrumentationKey。 不会检测此应用。
- `AppFilter='WebAppOne'` 将此特定应用分配唯一的检测密钥。
- `AppFilter='WebAppTwo'` 此外会将分配此特定应用唯一检测键。
- 最后，`AppFilter`还使用`'.*'`通配符来匹配所有其他 web 应用不由早期规则匹配，并将分配默认检测密钥。
- 为便于阅读仅添加空格。

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap 
    @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
      @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'},
      @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'},
      @{MachineFilter='.*';AppFilter='.*';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'})

```


## <a name="parameters"></a>parameters 

### <a name="-instrumentationkey"></a>-InstrumentationKey
**必需。** 使用此参数由目标计算机上的所有应用程序提供使用单个 iKey。

### <a name="-instrumentationkeymap"></a>-InstrumentationKeyMap
**必需。** 使用此参数提供多个 ikey 和要使用哪个 ikey 的应用的映射。 通过设置 MachineFilter，可以创建多台计算机的单独的安装脚本。 

> [!IMPORTANT] 
> 应用程序将根据规则向其提供的顺序匹配。 这种情况下应首先指定的最具体的规则和最后一个最一般的规则。

#### <a name="schema"></a>架构
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'})`

- **MachineFilter**是所需的 c# 正则表达式的计算机或虚拟机名称。
    - 。 * 将匹配所有
    - ComputerName 将匹配只有具有相同名称的计算机。
- **AppFilter**是所需的 c# 正则表达式的计算机或虚拟机名称。
    - 。 * 将匹配所有
    - 应用程序名称将匹配仅 IIS 应用程序使用的确切名称。
- **InstrumentationKey**需启用与上述两个筛选器匹配的应用程序的监视。
    - 保留此值为 null，如果你想要定义规则来监视中排除


### <a name="-verbose"></a>-Verbose
**通用参数。** 使用此开关以输出详细的日志。


## <a name="output"></a>输出

默认情况下没有输出。

#### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkey"></a>示例详细输出设置-InstrumentationKey 通过配置文件

```
VERBOSE: Operation: InstallWithIkey
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

#### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkeymap"></a>示例详细输出设置-InstrumentationKeyMap 通过配置文件

```
VERBOSE: Operation: InstallWithIkeyMap
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
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
