---
title: 在 Application Insights 中浏览 .NET 跟踪日志
description: 搜索由 Trace、 NLog 或 Log4Net 生成的日志。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 0c2a084f-6e71-467b-a6aa-4ab222f17153
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: mbullwin
ms.openlocfilehash: d366f363b7bd1d5306d598c9b38258eb78076b7c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472055"
---
# <a name="explore-netnet-core-trace-logs-in-application-insights"></a>在 Application Insights 中浏览 .NET/.NET Core 跟踪日志

从 ILogger、 NLog、 log4Net、 或 System.Diagnostics.Trace 到发送 ASP.NET/ASP.NET 核心应用程序的诊断跟踪日志[Azure Application Insights][start]。 然后，您可以浏览并搜索它们。 这些日志合并随其他日志文件从您的应用程序，因此您可以识别出与每个用户请求相关联并将它们与其他事件和异常报告相关联的跟踪。

> [!NOTE]
> 是否需要日志捕获模块？ 它是第三方记录器的有用适配器。 但如果您不已在使用 nlog、log4net 或 System.Diagnostics.Trace，请考虑直接调用[ **Application Insights tracktrace （)** ](../../azure-monitor/app/api-custom-events-metrics.md#tracktrace)直接。
>
>
## <a name="install-logging-on-your-app"></a>在应用上安装记录
在项目中，这应该产生 app.config 或 web.config 中的条目安装你所选的日志记录框架。

```XML
    <configuration>
      <system.diagnostics>
    <trace autoflush="true" indentsize="0">
      <listeners>
        <add name="myAppInsightsListener" type="Microsoft.ApplicationInsights.TraceListener.ApplicationInsightsTraceListener, Microsoft.ApplicationInsights.TraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
   </configuration>
```

## <a name="configure-application-insights-to-collect-logs"></a>配置 Application Insights 以收集日志
[将 Application Insights 添加到你的项目](../../azure-monitor/app/asp-net.md)如果尚未执行的操作。 会看到一个选项以包括日志收集器。

右键单击解决方案资源管理器中的项目或者**配置 Application Insights**。 选择**配置跟踪集合**选项。

> [!NOTE]
> 没有 Application Insights 菜单或日志收集器选项？ 尝试[故障排除](#troubleshooting)。

## <a name="manual-installation"></a>手动安装
如果项目类型（例如 Windows 桌面项目）不受 Application Insights 安装程序支持，请使用此方法。

1. 如果计划使用 log4Net 或 NLog，请在项目中安装它。
2. 在解决方案资源管理器，右键单击你的项目，然后选择**管理 NuGet 包**。
3. 搜索"Application Insights"。
4. 选择以下包之一：

   - 对于 ILogger：[Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.Extensions.Logging.ApplicationInsights.svg)](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
   - 对于 NLog：[Microsoft.ApplicationInsights.NLogTarget](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.NLogTarget.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
   - 对于 Log4Net：[Microsoft.ApplicationInsights.Log4NetAppender](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.Log4NetAppender.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
   - 对于 System.Diagnostics：[Microsoft.ApplicationInsights.TraceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.TraceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
   - [Microsoft.ApplicationInsights.DiagnosticSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.DiagnosticSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
   - [Microsoft.ApplicationInsights.EtwCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
[![NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EtwCollector.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
   - [Microsoft.ApplicationInsights.EventSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EventSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)

NuGet 包安装必要的程序集，并修改 web.config 或 app.config，如果这是适用。

## <a name="ilogger"></a>ILogger

使用控制台应用程序和 ASP.NET Core 中使用的应用程序见解 ILogger 实现的示例，请参阅[.NET Core ilogger ApplicationInsightsLoggerProvider 记录](ilogger.md)。

## <a name="insert-diagnostic-log-calls"></a>插入诊断日志调用
如果使用 System.Diagnostics.Trace，则典型的调用将是：

    System.Diagnostics.Trace.TraceWarning("Slow response - database01");

如果首选 log4net 或 NLog，请使用：

    logger.Warn("Slow response - database01");

## <a name="use-eventsource-events"></a>使用 EventSource 事件
可配置要作为跟踪发送到 Application Insights 的 [System.Diagnostics.Tracing.EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) 事件。 首先，安装 `Microsoft.ApplicationInsights.EventSourceListener` NuGet 包。 然后编辑 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 文件的 `TelemetryModules` 部分。

```xml
    <Add Type="Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule, Microsoft.ApplicationInsights.EventSourceListener">
      <Sources>
        <Add Name="MyCompany" Level="Verbose" />
      </Sources>
    </Add>
```

可对每个资源设置以下参数：
 * **名称**指定要收集的 EventSource 的名称。
 * **级别**指定要收集的日志记录级别：*关键*，*错误*，*条信息性*，*始终记录*，*详细*，或*警告*.
 * **关键字**（可选） 指定要使用的关键字组合的整数值。

## <a name="use-diagnosticsource-events"></a>使用 DiagnosticSource 事件
可配置要作为跟踪发送到 Application Insights 的 [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) 事件。 首先，安装 [`Microsoft.ApplicationInsights.DiagnosticSourceListener`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener) NuGet 包。 然后，编辑的"TelemetryModules"一节[ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md)文件。

```xml
    <Add Type="Microsoft.ApplicationInsights.DiagnosticSourceListener.DiagnosticSourceTelemetryModule, Microsoft.ApplicationInsights.DiagnosticSourceListener">
      <Sources>
        <Add Name="MyDiagnosticSourceName" />
      </Sources>
    </Add>
```

对于要跟踪的每个 DiagnosticSource，添加一个包含的项**名称**属性设置为项的名称。

## <a name="use-etw-events"></a>使用 ETW 事件
你可以配置要发送到 Application Insights 作为跟踪的事件跟踪 Windows (ETW) 事件。 首先，安装 `Microsoft.ApplicationInsights.EtwCollector` NuGet 包。 然后，编辑的"TelemetryModules"一节[ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md)文件。

> [!NOTE] 
> 如果是 Performance Log Users 或 Administrators 的成员身份运行的进程的宿主 SDK，才可收集 ETW 事件。

```xml
    <Add Type="Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule, Microsoft.ApplicationInsights.EtwCollector">
      <Sources>
        <Add ProviderName="MyCompanyEventSourceName" Level="Verbose" />
      </Sources>
    </Add>
```

可对每个资源设置以下参数：
 * **ProviderName**是要收集的 ETW 提供程序的名称。
 * **ProviderGuid**指定要收集的 ETW 提供程序的 GUID。 它可以使用而不是`ProviderName`。
 * **级别**设置要收集的日志记录级别。 它可以是*严重*，*错误*，*信息*，*始终记录*， *Verbose*，或*警告*。
 * **关键字**（可选） 设置要使用的关键字组合的整数值。

## <a name="use-the-trace-api-directly"></a>直接使用跟踪 API
可以直接调用 Application Insights 跟踪 API。 日志记录适配器使用此 API。

例如：

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow response - database01");

TrackTrace 的一个优势是可将相对较长的数据放置在消息中。 例如，可在此处对 POST 数据进行编码。

您还可以向您的消息添加严重性级别。 并像其他遥测一样，您可以添加属性值，以帮助筛选或搜索不同跟踪集。 例如:

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow database response",
                   SeverityLevel.Warning,
                   new Dictionary<string,string> { {"database", db.ID} });

这可使您轻松中筛选出[搜索][ diagnostic]特定严重级别的与特定数据库相关的所有消息。

## <a name="explore-your-logs"></a>浏览日志
在调试模式下运行你的应用或将其部署为实时。

在应用程序的概述窗格中，在[Application Insights 门户][portal]，选择[搜索][diagnostic]。

例如，可以：

* 筛选日志跟踪或具有特定属性的项。
* 检查详细信息中的特定项。
* 查找与同一用户请求与其他系统日志数据 （具有相同的 OperationId）。
* 将页面的配置保存到收藏夹。

> [!NOTE]
>如果你的应用程序发送大量数据，并且使用 Application Insights SDK 的 ASP.NET 版本 2.0.0-beta3 或更高版本，*自适应采样*功能可以正常运行，将仅为一部分遥测数据发送。 [了解有关采样的详细信息。](../../azure-monitor/app/sampling.md)
>

## <a name="troubleshooting"></a>故障排除
### <a name="how-do-i-do-this-for-java"></a>对于 Java，我该怎么做？
使用 [Java 日志适配器](../../azure-monitor/app/java-trace-logs.md)。

### <a name="theres-no-application-insights-option-on-the-project-context-menu"></a>项目上下文菜单上没有 Application Insights 选项
* 请确保 Developer Analytics Tools 安装在开发计算机上。 在 Visual Studio**工具** > **扩展和更新**，寻找**Developer Analytics Tools**。 如果在不是**已安装**选项卡上，打开**联机**选项卡并将其安装。
* 这可能是 Devloper 分析工具不支持的项目类型。 使用[手动安装](#manual-installation)。

### <a name="theres-no-log-adapter-option-in-the-configuration-tool"></a>配置工具中没有日志适配器选项
* 首先安装日志记录框架。
* 如果您使用的 System.Diagnostics.Trace，请确保将其[中配置*web.config*](https://msdn.microsoft.com/library/system.diagnostics.eventlogtracelistener.aspx)。
* 请确保具有最新版本的 Application Insights。 在 Visual Studio 中，转到**工具** > **扩展和更新**，并打开**更新**选项卡。如果**Developer Analytics Tools**是存在，则选择它以更新它。

### <a name="emptykey"></a>我收到"检测密钥不能为空"错误消息
你可能安装而无需安装 Application Insights 的日志记录适配器 Nuget 包。 在解决方案资源管理器中右键单击*ApplicationInsights.config*，然后选择**更新 Application Insights**。 系统将提示您登录到 Azure 并创建 Application Insights 资源或重复使用现有帐户。 它能解决问题。

### <a name="i-can-see-traces-but-not-other-events-in-diagnostic-search"></a>我可以看到跟踪，但未在诊断搜索中的其他事件
可能需要一段时间所有事件和请求都通过管道。

### <a name="limits"></a>保留多少数据？
以下几个因素会影响保留的数据量。 有关详细信息，请参阅[限制](../../azure-monitor/app/api-custom-events-metrics.md#limits)客户事件指标页部分。

### <a name="i-dont-see-some-log-entries-that-i-expected"></a>我看不到所需内容的某些日志条目
如果你的应用程序发送庞大的数据，并且要使用 Application Insights SDK 的 ASP.NET 版本 2.0.0-beta3 或更高版本，自适应采样功能可以正常运行，将仅为一部分遥测数据发送。 [了解有关采样的详细信息。](../../azure-monitor/app/sampling.md)

## <a name="add"></a>后续步骤

* [ASP.NET 中的诊断故障和异常][exceptions]
* [了解有关搜索的详细信息][diagnostic]
* [设置可用性和响应能力测试][availability]
* [故障排除][qna]

<!--Link references-->

[availability]: ../../azure-monitor/app/monitor-web-app-availability.md
[diagnostic]: ../../azure-monitor/app/diagnostic-search.md
[exceptions]: asp-net-exceptions.md
[portal]: https://portal.azure.com/
[qna]: ../../azure-monitor/app/troubleshoot-faq.md
[start]: ../../azure-monitor/app/app-insights-overview.md