---
title: 用于处理自定义事件和指标的 Application Insights API | Microsoft 文档
description: 在设备、桌面应用、网页或服务中插入几行代码，即可跟踪使用情况和诊断问题。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 80400495-c67b-4468-a92e-abf49793a54d
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/16/2018
ms.author: mbullwin
ms.openlocfilehash: aac5010ca6b0ed958a849bf203f1d2f80bcdb81c
ms.sourcegitcommit: 818d3e89821d101406c3fe68e0e6efa8907072e7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2019
ms.locfileid: "54119810"
---
# <a name="application-insights-api-for-custom-events-and-metrics"></a>用于处理自定义事件和指标的 Application Insights API

在应用程序中插入几行代码，即可了解用户在该应用程序中执行的操作或帮助诊断问题。 可以从设备和桌面应用、Web 客户端和 Web 服务器发送遥测数据。 使用 [Visual Studio Application Insights](../../azure-monitor/app/app-insights-overview.md) 核心遥测 API 发送自定义事件和指标，以及自己的标准遥测版本。 此 API 与标准 Application Insights 数据收集器使用的 API 相同。

> [!NOTE]
> `TrackMetric()` 不再是用于为基于 .NET 的应用程序发送自定义指标的首选方法。 Application Insights .NET SDK 的[版本 2.60-beta 3](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/develop/CHANGELOG.md#version-260-beta3) 中引入了一个新方法：[`TelemetryClient.GetMetric()`](https://docs.microsoft.com/dotnet/api/microsoft.applicationinsights.telemetryclient.getmetric?view=azure-dotnet)。 从 Application Insights .NET SDK [版本 2.72](https://docs.microsoft.com/dotnet/api/microsoft.applicationinsights.telemetryclient.getmetric?view=azure-dotnet) 开始，此功能现在是稳定版本的一部分。

## <a name="api-summary"></a>API 摘要

核心 API 在所有平台中是统一的，只有少许差异，例如 `GetMetric`（仅限 .NET）。

| 方法 | 用途 |
| --- | --- |
| [`TrackPageView`](#page-views) |页面、屏幕、边栏选项卡或窗体。 |
| [`TrackEvent`](#trackevent) |用户操作和其他事件。 用于跟踪用户行为或监视性能。 |
| [`GetMetric`](#getmetric) |零维度和多维度指标，集中配置的聚合，仅限 C#。 |
| [`TrackMetric`](#trackmetric) |性能度量，例如与特定事件不相关的队列长度。 |
| [`TrackException`](#trackexception) |记录诊断的异常。 跟踪与其他事件的相关性，以及检查堆栈跟踪。 |
| [`TrackRequest`](#trackrequest) |记录服务器请求的频率和持续时间以进行性能分析。 |
| [`TrackTrace`](#tracktrace) |诊断日志消息。 还可以捕获第三方日志。 |
| [`TrackDependency`](#trackdependency) |记录对应用依赖的外部组件的调用持续时间和频率。 |

可以[将属性和指标附加到](#properties)其中的大多数遥测调用。

## <a name="prep"></a>开始之前

如果还没有 Application Insights SDK 引用：

* 将 Application Insights SDK 添加到项目：

  * [ASP.NET 项目](../../azure-monitor/app/asp-net.md)
  * [Java 项目](../../azure-monitor/app/java-get-started.md)
  * [Node.js 项目](../../azure-monitor/app/nodejs.md)
  * [每个网页中的 JavaScript](../../azure-monitor/app/javascript.md) 
* 在设备或 Web 服务器代码中包含以下内容：

    *C#：* `using Microsoft.ApplicationInsights;`

    Visual Basic：`Imports Microsoft.ApplicationInsights`

    *Java：* `import com.microsoft.applicationinsights.TelemetryClient;`

    *Node.js：*`var applicationInsights = require("applicationinsights");`

## <a name="get-a-telemetryclient-instance"></a>获取 TelemetryClient 实例

获取 `TelemetryClient` 的实例（网页中的 JavaScript 除外）：

*C#*

```csharp
private TelemetryClient telemetry = new TelemetryClient();
```

Visual Basic

```vb
Private Dim telemetry As New TelemetryClient
```

*Java*

```java
private TelemetryClient telemetry = new TelemetryClient();
``` 

*Node.js*

```javascript
var telemetry = applicationInsights.defaultClient;
```

TelemetryClient 是线程安全的。

对于 ASP.NET 和 Java 项目，可自动捕获传入的 HTTP 请求。 可能需要为应用的其他模块创建 TelemetryClient 的其他实例。 例如，可以在中间件类中使用一个 TelemetryClient 实例报告业务逻辑事件。 可以设置属性（如 UserId 和 DeviceId）来标识计算机。 此信息将附加到实例发送的所有事件中。 

*C#*

```csharp
TelemetryClient.Context.User.Id = "...";
TelemetryClient.Context.Device.Id = "...";
```

*Java*

```java
telemetry.getContext().getUser().setId("...");
telemetry.getContext().getDevice().setId("...");
```

在 Node.js 项目中，可以使用 `new applicationInsights.TelemetryClient(instrumentationKey?)` 创建新实例，但这建议仅用于需要与单一实例 `defaultClient` 隔离的配置的方案。

## <a name="trackevent"></a>TrackEvent

在 Application Insights 中，自定义事件是一个数据点，它可在[指标资源管理器](../../azure-monitor/app/metrics-explorer.md)中显示为聚合计数，在[诊断搜索](../../azure-monitor/app/diagnostic-search.md)中显示为单个事件。 （它与 MVC 或其他框架“事件”不相关。）

在代码中插入 `TrackEvent` 调用来统计各种事件。 用户选择特定功能的频率、实现特定目标的频率，或可能制造特定类型的错误的频率。

例如，在游戏应用中，每当用户获胜时会发送事件：

*JavaScript*

```javascript
appInsights.trackEvent("WinGame");
```

*C#*

```csharp
telemetry.TrackEvent("WinGame");
```

Visual Basic

```vb
telemetry.TrackEvent("WinGame")
```

*Java*

```java
telemetry.trackEvent("WinGame");
```

*Node.js*

```javascript
telemetry.trackEvent({name: "WinGame"});
```

### <a name="custom-events-in-analytics"></a>Analytics 中的自定义事件

[Application Insights Analytics](analytics.md) 的 `customEvents` 表格提供了遥测。 每行表示对应用中 `trackEvent(..)` 的调用。

如果正在进行[采样](../../azure-monitor/app/sampling.md)，那么 itemCount 属性将显示大于 1 的值。 例如，itemCount==10 表明对 trackEvent() 调用了 10 次，采样进程只传输其中一次。 若要获取自定义事件的正确计数，应使用 `customEvents | summarize sum(itemCount)` 之类的代码。

## <a name="getmetric"></a>GetMetric

### <a name="examples"></a>示例

*C#*

```csharp
#pragma warning disable CA1716  // Namespace naming

namespace User.Namespace.Example01
{
    using System;
    using Microsoft.ApplicationInsights;
    using TraceSeveretyLevel = Microsoft.ApplicationInsights.DataContracts.SeverityLevel;

    /// <summary>
    /// Most simple cases are one-liners.
    /// This is all possible without even importing an additional namespace.
    /// </summary>

    public class Sample01
    {
        /// <summary />
        public static void Exec()
        {
            // *** SENDING METRICS ***

            // Recall how you send custom telemetry with Application Insights in other cases, e.g. Events.
            // The following will result in an EventTelemetry object to be sent to the cloud right away.
            TelemetryClient client = new TelemetryClient();
            client.TrackEvent("SomethingInterestingHappened");

            // Metrics work very similar. However, the value is not sent right away.
            // It is aggregated with other values for the same metric, and the resulting summary (aka "aggregate" is sent automatically every minute.
            // To mark this difference, we use a pattern that is similar, but different from the established TrackXxx(..) pattern that sends telemetry right away:

            client.GetMetric("CowsSold").TrackValue(42);

            // *** MULTI-DIMENSIONAL METRICS ***

            // The above example shows a zero-dimensional metric.
            // Metrics can also be multi-dimensional.
            // In the initial version we are supporting up to 2 dimensions, and we will add support for more in the future as needed.
            // Here is an example for a one-dimensional metric:

            Metric animalsSold = client.GetMetric("AnimalsSold", "Species");

            animalsSold.TrackValue(42, "Pigs");
            animalsSold.TrackValue(24, "Horses");

            // The values for Pigs and Horses will be aggregated separately from each other and will result in two distinct aggregates.
            // You can control the maximum number of number data series per metric (and thus your resource usage and cost).
            // The default limits are no more than 1000 total data series per metric, and no more than 100 different values per dimension.
            // We discuss elsewhere how to change them.
            // We use a common .Net pattern: TryXxx(..) to make sure that the limits are observed.
            // If the limits are already reached, Metric.TrackValue(..) will return False and the value will not be tracked. Otherwise it will return True.
            // This is particularly useful if the data for a metric originates from user input, e.g. a file:

            Tuple<int, string> countAndSpecies = ReadSpeciesFromUserInput();
            int count = countAndSpecies.Item1;
            string species = countAndSpecies.Item2;

            if (!animalsSold.TrackValue(count, species))

            {
                client.TrackTrace($"Data series or dimension cap was reached for metric {animalsSold.Identifier.MetricId}.", TraceSeveretyLevel.Error);
            }

            // You can inspect a metric object to reason about its current state. For example:
            int currentNumberOfSpecies = animalsSold.GetDimensionValues(1).Count;
        }

        private static void ResetDataStructure()
        {
            // Do stuff
        }

        private static Tuple<int, string> ReadSpeciesFromUserInput()
        {
            return Tuple.Create(18, "Cows");
        }

        private static int AddItemsToDataStructure()
        {
            // Do stuff
            return 5;
        }
    }
}
```

## <a name="trackmetric"></a>TrackMetric

> [!NOTE]
> Microsoft.ApplicationInsights.TelemetryClient.TrackMetric 在 .NET SDK 中已弃用。 在发送之前，应当始终对一段时间内的指标进行预聚合。 使用 GetMetric(..) 重载之一获取用于访问 SDK 预聚合功能的指标对象。 如果实现你自己的预聚合逻辑，则可以使用 Track(ITelemetry metricTelemetry) 方法来发送生成的聚合。 如果应用程序需要在每种场合下发送单独的遥测项而不需要在整个时间段上进行聚合，那么你可能就有了一个事件遥测用例；请参阅 TelemetryClient.TrackEvent (Microsoft.Applicationlnsights.DataContracts.EventTelemetry)。

Application Insights 可绘制未附加到特定事件的指标。 例如，可以定期监视队列长度。 对指标而言，变化和趋势比单个度量值更具价值，因此统计图表非常实用。

若要将指标发送到 Application Insights，可以使用 `TrackMetric(..)` API。 可通过两种方式发送指标：

* 单个值。 每次在应用中执行测量时，会发送相应的值到 Application Insights。 例如，假设有个指标用于描述容器中项的数量。 在特定时间段，先将 3 个项放入容器中，再从容器中移除 2 个项。 相应地，将会调用 `TrackMetric` 两次：首先传递值 `3`，然后传递值 `-2`。 Application Insights 会替你存储这两个值。

* 聚合。 使用指标时，每个单次测量几乎无关紧要。 反而特定时间段内发生活动的摘要很重要。 此类摘要名为 _聚合_ 。 在上一示例中，该时间段的聚合指标总数为 `1`，同时指标值的计数为 `2`。 使用聚合方法时，每个时间段只调用一次 `TrackMetric` 并发送聚合值。 建议采用此方法是因为它可以通过发送更少的数据点到 Application Insights 同时仍然收集所有相关信息来显著降低成本和性能开销。

### <a name="examples"></a>示例

#### <a name="single-values"></a>单个值

发送单一指标值：

*JavaScript*

 ```javascript
appInsights.trackMetric("queueLength", 42.0);
 ```

*C#*

```csharp
var sample = new MetricTelemetry();
sample.Name = "metric name";
sample.Value = 42.3;
telemetryClient.TrackMetric(sample);
```

*Java*

```java
telemetry.trackMetric("queueLength", 42.0);
```

*Node.js*

 ```javascript
telemetry.trackMetric({name: "queueLength", value: 42.0});
 ```

### <a name="custom-metrics-in-analytics"></a>分析中的自定义指标

[Application Insights Analytics](analytics.md) 的 `customMetrics` 表格提供了遥测。 每行表示对应用中 `trackMetric(..)` 的调用。

* `valueSum` - 这是度量值的总和。 若要获取平均值，请除以 `valueCount`。
* `valueCount` - 聚合到此 `trackMetric(..)` 调用中的度量值个数。

## <a name="page-views"></a>页面视图

在设备或网页应用中，加载每个屏幕或页面时默认将发送页面视图遥测数据。 但是，可以更改为在其他时间或不同时间跟踪页面视图。 例如，在显示选项卡或边栏选项卡的应用中，可以在用户每次打开新边栏选项卡时跟踪一个页面。

用户和会话数据与页面视图作为属性一起发送，以便在有页面视图遥测数据时显示用户与会话图表。

### <a name="custom-page-views"></a>自定义页面视图

*JavaScript*

```javascript
appInsights.trackPageView("tab1");
```

*C#*

```csharp
telemetry.TrackPageView("GameReviewPage");
```

Visual Basic

```vb
telemetry.TrackPageView("GameReviewPage")
```

*Java*

```java
telemetry.trackPageView("GameReviewPage");
```

如果不同的 HTML 网页中有多个选项卡，还可以指定 URL：

```javascript
appInsights.trackPageView("tab1", "http://fabrikam.com/page1.htm");
```

### <a name="timing-page-views"></a>计时页面视图

默认情况下，报告为“页面视图加载时间”的时间测量是从浏览器发送请求开始、调用浏览器的页面加载事件为止的时间。

可以：

* 在 [trackPageView](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md#trackpageview) 调用中设置显式持续时间：`appInsights.trackPageView("tab1", null, null, null, durationInMilliseconds);`。
* 使用页面视图计时调用 `startTrackPage` 和 `stopTrackPage`。

*JavaScript*

```javascript
// To start timing a page:
appInsights.startTrackPage("Page1");

...

// To stop timing and log the page:
appInsights.stopTrackPage("Page1", url, properties, measurements);
```

用作第一个参数的名称会将开始调用与停止调用相关联。 该名称默认为当前页面名称。

显示在指标资源管理器中的最终页面加载持续时间派生自开始调用与停止调用的间隔时间。 实际时间间隔由你决定。

### <a name="page-telemetry-in-analytics"></a>Analytics 中的页面遥测

[Analytics](analytics.md)中有两个表展示了浏览器操作的数据：

* `pageViews` 表包含关于 URL 和页标题的数据
* `browserTimings` 表包含关于客户端性能的数据，例如处理传入数据所用的时间

查找浏览器处理不同页面需要的时间：

```kusto
browserTimings
| summarize avg(networkDuration), avg(processingDuration), avg(totalDuration) by name
```

发现不同浏览器的热门程度：

```kusto
pageViews
| summarize count() by client_Browser
```

若要将页面视图关联到 AJAX 调用，请联接依赖项：

```kusto
pageViews
| join (dependencies) on operation_Id 
```

## <a name="trackrequest"></a>TrackRequest

服务器 SDK 使用 TrackRequest 记录 HTTP 请求。

如果想要在没有 Web 服务模块运行的上下文中模拟请求，也可以自行调用。

但是，发送请求遥测的建议方式是将请求用作<a href="#operation-context">操作上下文</a>。

## <a name="operation-context"></a>操作上下文

可以通过将遥测项与操作上下文关联来将遥测项关联在一起。 标准的请求跟踪模块针对在处理 HTTP 请求时发送的异常和其他事件执行此操作。 在[搜索](../../azure-monitor/app/diagnostic-search.md)和[分析](analytics.md)中，可以使用此操作 ID 轻松找到与请求关联的任何事件。

有关关联的更多详细信息，请参阅 [Application Insights 中的遥测关联](../../azure-monitor/app/correlation.md)。

手动跟踪遥测时，使用此模式确保遥测关联的最简单方法：

*C#*

```csharp
// Establish an operation context and associated telemetry item:
using (var operation = telemetryClient.StartOperation<RequestTelemetry>("operationName"))
{
    // Telemetry sent in here will use the same operation ID.
    ...
    telemetryClient.TrackTrace(...); // or other Track* calls
    ...

    // Set properties of containing telemetry item--for example:
    operation.Telemetry.ResponseCode = "200";

    // Optional: explicitly send telemetry item:
    telemetryClient.StopOperation(operation);

} // When operation is disposed, telemetry item is sent.
```

除了设置操作上下文之外，`StartOperation` 还会创建一个所指定类型的遥测项。 在处理操作时，或如果显式调用 `StopOperation`，它将发送遥测项。 如果使用 `RequestTelemetry` 作为遥测类型，其持续时间将设置为开始与停止的间隔时间。

在操作范围内报告的遥测项将成为此类操作的“子级”。 操作上下文可以嵌套。

在搜索中，操作上下文可用于创建“相关项”列表：

![相关项](./media/api-custom-events-metrics/21.png)

有关自定义操作跟踪的详细信息，请参阅[使用 Application Insights .NET SDK 跟踪自定义操作](../../azure-monitor/app/custom-operations-tracking.md)。

### <a name="requests-in-analytics"></a>Analytics 中的请求

在 [Application Insights Analytics](analytics.md) 中，请求出现在 `requests` 表中。

如果正在进行[采样](../../azure-monitor/app/sampling.md)，那么 itemCount 属性将会显示大于 1 的值。 例如，itemCount==10 表明对 trackRequest() 调用了 10 次，采样进程只传输其中一次。 若要按请求名称获取正确的请求数和平均持续时间，请使用如下所示的代码：

```kusto
requests
| summarize count = sum(itemCount), avgduration = avg(duration) by name
```

## <a name="trackexception"></a>TrackException

将异常发送到 Application Insights：

* 用于[对其计数](../../azure-monitor/app/metrics-explorer.md)，作为问题发生频率的指示。
* 用于[检查单个事件](../../azure-monitor/app/diagnostic-search.md)。

报告包含堆栈跟踪。

*C#*

```csharp
try
{
    ...
}
catch (Exception ex)
{
    telemetry.TrackException(ex);
}
```

*Java*

```java
try {
    ...
} catch (Exception ex) {
    telemetry.trackException(ex);
}
```

*JavaScript*

```javascript
try
{
    ...
}
catch (ex)
{
    appInsights.trackException(ex);
}
```

*Node.js*

```javascript
try
{
    ...
}
catch (ex)
{
    telemetry.trackException({exception: ex});
}
```

SDK 会自动捕获许多异常，因此不一定需要显式调用 TrackException。

* ASP.NET：[编写代码来捕获异常](../../azure-monitor/app/asp-net-exceptions.md)。
* J2EE：[自动捕获异常](../../azure-monitor/app/java-get-started.md#exceptions-and-request-failures)。
* JavaScript：自动捕获异常。 若要禁用自动收集，请在插入网页的代码片段中添加一行：

```javascript
({
    instrumentationKey: "your key",
    disableExceptionTracking: true
})
```

### <a name="exceptions-in-analytics"></a>Analytics 中的异常

在 [Application Insights Analytics](analytics.md) 中，异常出现在 `exceptions` 表中。

如果正在进行[采样](../../azure-monitor/app/sampling.md)，那么 `itemCount` 属性将显示大于 1 的值。 例如，itemCount==10 表明对 trackException() 调用了 10 次，采样进程只传输其中一次。 若要按异常类型获取正确的异常数，请使用如下所示的代码：

```kusto
exceptions
| summarize sum(itemCount) by type
```

虽然大部分重要的堆叠信息已提取到了单独的变量中，但可以扩展 `details` 结构，获取更多信息。 由于这个结构是动态的，应将结果转换为预期的类型。 例如：

```kusto
exceptions
| extend method2 = tostring(details[0].parsedStack[1].method)
```

若要将异常与相关请求关联，请使用联接：

```kusto
exceptions
| join (requests) on operation_Id
```

## <a name="tracktrace"></a>TrackTrace

使用 TrackTrace 可以通过将“痕迹导航跟踪”发送到 Application Insights 来帮助诊断问题。 可以发送诊断数据区块，并在[诊断搜索](../../azure-monitor/app/diagnostic-search.md)中检查。

使用 .NET 时，[日志适配器](../../azure-monitor/app/asp-net-trace-logs.md)使用此 API 将第三方日志发送到门户。

使用 Java 时，[标准记录器（如 Log4J、Logback）](../../azure-monitor/app/java-trace-logs.md)使用 Application Insights Log4j 或 Logback Appenders 将第三方日志发送到门户。

*C#*

```csharp
telemetry.TrackTrace(message, SeverityLevel.Warning, properties);
```

*Java*

```java
telemetry.trackTrace(message, SeverityLevel.Warning, properties);
```

*Node.js*

```javascript
telemetry.trackTrace({
    message: message,
    severity: applicationInsights.Contracts.SeverityLevel.Warning,
    properties: properties
});
```

可以搜索消息内容，但是（不同于属性值）无法在其中进行筛选。

`message` 上的大小限制比属性上的限制高得多。
TrackTrace 的一个优势是可将相对较长的数据放置在消息中。 例如，可在此处对 POST 数据进行编码。  

此外，可向消息添加严重性级别。 并像其他遥测一样，可以添加属性值以帮助筛选或搜索不同跟踪集。 例如：

*C#*

```csharp
var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
telemetry.TrackTrace("Slow database response",
                SeverityLevel.Warning,
                new Dictionary<string,string> { {"database", db.ID} });
```

*Java*

```java
Map<String, Integer> properties = new HashMap<>();
properties.put("Database", db.ID);
telemetry.trackTrace("Slow Database response", SeverityLevel.Warning, properties);
```

在[搜索](../../azure-monitor/app/diagnostic-search.md)中，可轻松筛选出与特定数据库相关的所有特定严重性级别的消息。

### <a name="traces-in-analytics"></a>Analytics 中的跟踪

在 [Application Insights Analytics](analytics.md) 中，对 TrackTrace 的调用出现在 `traces` 表中。

如果正在进行[采样](../../azure-monitor/app/sampling.md)，那么 itemCount 属性将显示大于 1 的值。 例如，itemCount==10 表明对 `trackTrace()` 调用了 10 次，采样进程只传输其中一次。 若要获取正确的跟踪调用数，应使用 `traces | summarize sum(itemCount)` 之类的代码。

## <a name="trackdependency"></a>TrackDependency

可使用 TrackDependency 调用跟踪响应时间以及调用外部代码片段的成功率。 结果会显示在门户上的依赖项图表中。

*C#*

```csharp
var success = false;
var startTime = DateTime.UtcNow;
var timer = System.Diagnostics.Stopwatch.StartNew();
try
{
    success = dependency.Call();
}
finally
{
    timer.Stop();
    telemetry.TrackDependency("myDependency", "myCall", startTime, timer.Elapsed, success);
     // The call above has been made obsolete in the latest SDK. The updated call follows this format:
     // TrackDependency (string dependencyTypeName, string dependencyName, string data, DateTimeOffset startTime, TimeSpan duration, bool success);
}
```

*Java*

```java
boolean success = false;
long startTime = System.currentTimeMillis();
try {
    success = dependency.call();
}
finally {
    long endTime = System.currentTimeMillis();
    long delta = endTime - startTime;
    RemoteDependencyTelemetry dependencyTelemetry = new RemoteDependencyTelemetry("My Dependency", "myCall", delta, success);
    telemetry.setTimeStamp(startTime);
    telemetry.trackDependency(dependencyTelemetry);
}
```

*JavaScript*

```javascript
var success = false;
var startTime = new Date().getTime();
try
{
    success = dependency.Call();
}
finally
{
    var elapsed = new Date() - startTime;
    telemetry.trackDependency({
        dependencyTypeName: "myDependency",
        name: "myCall",
        duration: elapsed,
        success: success
    });
}
```

请记住，服务器 SDK 包含[依赖项模块](../../azure-monitor/app/asp-net-dependencies.md)，用于自动发现和跟踪特定的依赖项调用（例如，数据库和 REST API）。 必须在服务器上安装一个代理才能让模块正常运行。 

使用 Java 时，可以使用 [Java 代理](../../azure-monitor/app/java-agent.md)自动跟踪某些依赖项调用。

如果想要跟踪自动跟踪未捕获的调用，或不想安装代理，可以使用此调用。

要关闭 C# 中的标准依赖项跟踪模块，请编辑 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 并删除对 `DependencyCollector.DependencyTrackingTelemetryModule` 的引用。 使用 Java 时，如果不希望自动收集标准依赖项，请勿安装 java 代理。

### <a name="dependencies-in-analytics"></a>Analytics 中的依赖项

在 [Application Insights Analytics](analytics.md) 中，trackDependency 调用出现在 `dependencies` 表中。

如果正在进行[采样](../../azure-monitor/app/sampling.md)，那么 itemCount 属性将显示大于 1 的值。 例如，itemCount==10 表明对 trackDependency() 调用了 10 次，采样进程只传输其中一次。 若要按目标组件获取正确的依赖项数，请使用如下所示的代码：

```kusto
dependencies
| summarize sum(itemCount) by target
```

若要将依赖项与相关请求关联，请使用联接：

```kusto
dependencies
| join (requests) on operation_Id
```

## <a name="flushing-data"></a>刷新数据

通常，SDK 在选定的时间发送数据，以便最大程度地降低对用户的影响。 但是，在某些情况下，可能需要刷新缓冲区，例如，在关闭的应用程序中使用 SDK 时。

*C#*

 ```csharp
telemetry.Flush();
// Allow some time for flushing before shutdown.
System.Threading.Thread.Sleep(5000);
```

*Java*

```java
telemetry.flush();
//Allow some time for flushing before shutting down
Thread.sleep(5000);
```

*Node.js*

```javascript
telemetry.flush();
```

请注意，[服务器遥测通道](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/)的函数是异步的。

理想情况下，应在应用程序的关闭活动中使用 flush() 方法。

## <a name="authenticated-users"></a>经过身份验证的用户

在 Web 应用中，默认按 Cookie 标识用户。 如果用户从不同的计算机或浏览器访问应用或删除 Cookie，则可能会多次统计它们。

如果用户登录到应用，可以通过在浏览器代码中设置经过身份验证的用户 ID 来获取更准确的计数：

*JavaScript*

```javascript
// Called when my app has identified the user.
function Authenticated(signInId) {
    var validatedId = signInId.replace(/[,;=| ]+/g, "_");
    appInsights.setAuthenticatedUserContext(validatedId);
    ...
}
```

例如，在 ASP.NET Web MVC 应用程序中：

*Razor*

```cshtml
@if (Request.IsAuthenticated)
{
    <script>
        appInsights.setAuthenticatedUserContext("@User.Identity.Name
            .Replace("\\", "\\\\")"
            .replace(/[,;=| ]+/g, "_"));
    </script>
}
```

不需要使用用户的实际登录名。 只需使用该用户的唯一 ID 即可。 该 ID 不能包含空格或任何 `,;=|` 字符。

还可在会话 Cookie 中设置用户 ID 并将其发送到服务器。 如果安装了服务器 SDK，则经过身份验证的用户 ID 将作为客户端和服务器遥测上下文属性的一部分发送。 可对其进行筛选和搜索。

如果应用将用户分组到帐户，则还可以传递该帐户的标识符（具有相同的字符限制）。

```javascript
appInsights.setAuthenticatedUserContext(validatedId, accountId);
```

在[指标资源管理器](../../azure-monitor/app/metrics-explorer.md)中，可以创建统计“经身份验证的用户”和“用户帐户”的图表。

还可以[搜索](../../azure-monitor/app/diagnostic-search.md)具有特定用户名和帐户的客户端数据点。

## <a name="properties"></a>使用属性筛选、搜索和细分数据

可以将属性和度量值附加到事件（以及指标、页面视图、异常和其他遥测数据）。

*属性*是可以在使用情况报告中用来筛选遥测数据的字符串值。 例如，如果应用提供多种游戏，可以将游戏的名称附加到每个事件，了解哪些游戏更受欢迎。

字符串长度限制为 8192。 （如果想要发送大型数据区块，请使用消息参数 [TrackTrace](#track-trace)。）

*指标*是能够以图形方式呈现的数字值。 例如，可以查看玩家的分数是否逐渐增加。 可以根据连同事件一起发送的属性对图表进行分段，以便获取不同游戏的独立图形或堆积图。

这些值应大于或等于 0，以便正确显示指标值。

可[对属性、属性值和指标的数目使用一些限制](#limits)。

*JavaScript*

```javascript
appInsights.trackEvent
    ("WinGame",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );

appInsights.trackPageView
    ("page name", "http://fabrikam.com/pageurl.html",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );
```

*C#*

```csharp
// Set up some properties and metrics:
var properties = new Dictionary <string, string>
    {{"game", currentGame.Name}, {"difficulty", currentGame.Difficulty}};
var metrics = new Dictionary <string, double>
    {{"Score", currentGame.Score}, {"Opponents", currentGame.OpponentCount}};

// Send the event:
telemetry.TrackEvent("WinGame", properties, metrics);
```

*Node.js*

```javascript
// Set up some properties and metrics:
var properties = {"game": currentGame.Name, "difficulty": currentGame.Difficulty};
var metrics = {"Score": currentGame.Score, "Opponents": currentGame.OpponentCount};

// Send the event:
telemetry.trackEvent({name: "WinGame", properties: properties, measurements: metrics});
```

Visual Basic

```vb
' Set up some properties:
Dim properties = New Dictionary (Of String, String)
properties.Add("game", currentGame.Name)
properties.Add("difficulty", currentGame.Difficulty)

Dim metrics = New Dictionary (Of String, Double)
metrics.Add("Score", currentGame.Score)
metrics.Add("Opponents", currentGame.OpponentCount)

' Send the event:
telemetry.TrackEvent("WinGame", properties, metrics)
```

*Java*

```java
Map<String, String> properties = new HashMap<String, String>();
properties.put("game", currentGame.getName());
properties.put("difficulty", currentGame.getDifficulty());

Map<String, Double> metrics = new HashMap<String, Double>();
metrics.put("Score", currentGame.getScore());
metrics.put("Opponents", currentGame.getOpponentCount());

telemetry.trackEvent("WinGame", properties, metrics);
```

> [!NOTE]
> 请注意不要在属性中记录个人身份信息。
>
>

### <a name="alternative-way-to-set-properties-and-metrics"></a>设置属性和指标的替代方法

如果更方便的话，可以收集不同对象中的事件的参数：

```csharp
var event = new EventTelemetry();

event.Name = "WinGame";
event.Metrics["processingTime"] = stopwatch.Elapsed.TotalMilliseconds;
event.Properties["game"] = currentGame.Name;
event.Properties["difficulty"] = currentGame.Difficulty;
event.Metrics["Score"] = currentGame.Score;
event.Metrics["Opponents"] = currentGame.Opponents.Length;

telemetry.TrackEvent(event);
```

> [!WARNING]
> 请不要重复使用相同的遥测项实例（本示例中为 `event`）来调用 Track*() 多次。 这可能会导致使用不正确的配置发送遥测数据。
>
>

### <a name="custom-measurements-and-properties-in-analytics"></a>在 Analytics 中自定义度量值和属性

在 [Analytics](analytics.md) 中，自定义指标和属性显示在每个遥测记录的 `customMeasurements` 和 `customDimensions` 属性中。

例如，如果已向请求遥测添加名为“game”的属性，此查询将计算“game”不同值的出现次数，同时显示自定义指标“score”的平均值：

```kusto
requests
| summarize sum(itemCount), avg(todouble(customMeasurements.score)) by tostring(customDimensions.game)
```

请注意：

* 从 customDimensions 或 customMeasurements JSON 中提取值的时候，会有动态类型，所以必须将其转换为 `tostring` 或 `todouble`。
* 考虑到[采样](../../azure-monitor/app/sampling.md)的可能性，需要使用 `sum(itemCount)` 而非 `count()`。

## <a name="timed"></a>计时事件

有时，需要绘制图表来呈现执行某个操作花费了多少时间。 例如，你可能想要知道用户在游戏中考虑如何选择时花费了多少时间。 为此，可以使用度量参数。

*C#*

```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();

// ... perform the timed action ...

stopwatch.Stop();

var metrics = new Dictionary <string, double>
    {{"processingTime", stopwatch.Elapsed.TotalMilliseconds}};

// Set up some properties:
var properties = new Dictionary <string, string>
    {{"signalSource", currentSignalSource.Name}};

// Send the event:
telemetry.TrackEvent("SignalProcessed", properties, metrics);
```

*Java*

```java
long startTime = System.currentTimeMillis();

// Perform timed action

long endTime = System.currentTimeMillis();
Map<String, Double> metrics = new HashMap<>();
metrics.put("ProcessingTime", endTime-startTime);

// Setup some properties
Map<String, String> properties = new HashMap<>();
properties.put("signalSource", currentSignalSource.getName());

// Send the event
telemetry.trackEvent("SignalProcessed", properties, metrics);
```

## <a name="defaults"></a>自定义遥测的默认属性

如果想要为编写的一些自定义事件设置默认属性值，可以在 TelemetryClient 实例中设置。 这些值将附加到从该客户端发送的每个遥测项。

*C#*

```csharp
using Microsoft.ApplicationInsights.DataContracts;

var gameTelemetry = new TelemetryClient();
gameTelemetry.Context.Properties["Game"] = currentGame.Name;
// Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame");
```

Visual Basic

```vb
Dim gameTelemetry = New TelemetryClient()
gameTelemetry.Context.Properties("Game") = currentGame.Name
' Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame")
```

*Java*

```java
import com.microsoft.applicationinsights.TelemetryClient;
import com.microsoft.applicationinsights.TelemetryContext;
...


TelemetryClient gameTelemetry = new TelemetryClient();
TelemetryContext context = gameTelemetry.getContext();
context.getProperties().put("Game", currentGame.Name);

gameTelemetry.TrackEvent("WinGame");
```

*Node.js*

```javascript
var gameTelemetry = new applicationInsights.TelemetryClient();
gameTelemetry.commonProperties["Game"] = currentGame.Name;

gameTelemetry.TrackEvent({name: "WinGame"});
```

单个遥测调用可以重写其属性字典中的默认值。

*对于 JavaScript Web 客户端*，请[使用 JavaScript 遥测初始值设定项](#js-initializer)。

若要向所有遥测数据（包括来自标准收集模块的数据）添加属性，请[实现 `ITelemetryInitializer`](../../azure-monitor/app/api-filtering-sampling.md#add-properties)。

## <a name="sampling-filtering-and-processing-telemetry"></a>采样、筛选和处理遥测数据

可以先通过编写代码来处理遥测数据，再从 SDK 发送该数据。 处理包括从标准遥测模块（如 HTTP 请求收集和依赖项收集）发送的数据。

通过实现 `ITelemetryInitializer` [将属性添加到](../../azure-monitor/app/api-filtering-sampling.md#add-properties)遥测。 例如，可添加版本号或从其他属性计算得出的值。

[筛选](../../azure-monitor/app/api-filtering-sampling.md#filtering)可以先修改或丢弃遥测数据，然后通过实现 `ITelemetryProcesor` 从 SDK 发送遥测数据。 可以控制要发送或丢弃的项，但必须考虑到这会给指标造成怎样的影响。 根据丢弃项的方式，有时你可能无法在相关项之间导航。

[采样](../../azure-monitor/app/api-filtering-sampling.md)是减少从应用发送到门户的数据量的打包解决方案。 它不会影响显示的指标。 且不影响通过在相关项（如异常、请求和页面视图）之间导航来诊断问题。

[了解详细信息](../../azure-monitor/app/api-filtering-sampling.md)。

## <a name="disabling-telemetry"></a>禁用遥测

*动态停止和启动*收集与传输遥测数据：

*C#*

```csharp
using  Microsoft.ApplicationInsights.Extensibility;

TelemetryConfiguration.Active.DisableTelemetry = true;
```

*Java*

```java
telemetry.getConfiguration().setTrackingDisabled(true);
```

若要*禁用选定的标准收集器*（例如性能计数器、HTTP 请求或依赖项），请删除或注释掉 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 中的相关行。例如，如果想要发送自己的 TrackRequest 数据，则可以这样做。

*Node.js*

```javascript
telemetry.config.disableAppInsights = true;
```

若要*禁用所选的标准收集器*（例如，性能计数器、HTTP 请求或依赖项），初始化时请将配置方法链接到 SDK 初始化代码：

```javascript
applicationInsights.setup()
    .setAutoCollectRequests(false)
    .setAutoCollectPerformance(false)
    .setAutoCollectExceptions(false)
    .setAutoCollectDependencies(false)
    .setAutoCollectConsole(false)
    .start();
```

若要在初始化后禁用这些收集器，请使用配置对象：`applicationInsights.Configuration.setAutoCollectRequests(false)`

## <a name="debug"></a>开发人员模式

在调试期间，通过管道加速遥测会很有效，这样可以立即看到结果。 此外，还可以获得其他消息来帮助跟踪任何遥测问题。 在生产环境中请关闭此模式，因为它可能会拖慢应用。

*C#*

```csharp
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = true;
```

Visual Basic

```vb
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = True
```

## <a name="ikey"></a>设置所选自定义遥测的检测密钥

*C#*

```csharp
var telemetry = new TelemetryClient();
telemetry.InstrumentationKey = "---my key---";
// ...
```

## <a name="dynamic-ikey"></a> 动态检测密钥

若要避免混合来自开发、测试和生产环境的遥测，可以[创建单独的 Application Insights 资源](../../azure-monitor/app/create-new-resource.md )，并根据环境更改其密钥。

无需从配置文件获取检测密钥，可以在代码中设置密钥。 在初始化方法中设置密钥，如 ASP.NET 服务中的 global.aspx.cs：

*C#*

```csharp
protected void Application_Start()
{
    Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey =
        // - for example -
        WebConfigurationManager.Settings["ikey"];
    ...
}
```

*JavaScript*

```javascript
appInsights.config.instrumentationKey = myKey;
```

在网页中，可能需要通过 Web 服务器的状态设置密钥，而不是以文本方式将它编码到脚本中。 例如，在 ASP.NET 应用中生成的网页中：

*使用 Razor 的 JavaScript*

```cshtml
<script type="text/javascript">
// Standard Application Insights webpage script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
    // Generate from server property:
    @Microsoft.ApplicationInsights.Extensibility.
        TelemetryConfiguration.Active.InstrumentationKey;
}) // ...
```

## <a name="telemetrycontext"></a>TelemetryContext

TelemetryClient 具有上下文属性，其中包含与所有遥测数据一起发送的值。 它们通常由标准遥测模块设置，但你也可以自行设置。 例如：

```csharp
telemetry.Context.Operation.Name = "MyOperationName";
```

如果自行设置这些值，请考虑从 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 中删除相关的代码行，以便值与标准值不会造成混淆。

* **Component**：应用及其版本。
* **Device**：有关正在运行应用的设备的数据。 （在 Web 应用中，是指从其中发送遥测的服务器或客户端设备。）
* **InstrumentationKey**：Azure 中显示遥测数据的 Application Insights 资源。 通常可从 ApplicationInsights.config 中选择。
* **Location**：设备的地理位置。
* **Operation**：在 Web 应用中，为当前的 HTTP 请求。 在其他应用类型中，可将此属性设置为将事件分组在一起。
  * **Id**：一个生成的值，它将不同的事件关联在一起，以便在诊断搜索中检查任何事件时，可以发现相关项。
  * **Name**：一个标识符，通常是 HTTP 请求的 URL。
  * **SyntheticSource**：如果不为 null 或空，则此字符串表示请求的源已标识为机器人或 Web 测试。 默认情况下，该属性会从指标资源管理器的计算中排除。
* **Properties**：与所有遥测数据一起发送的属性。 可在单个 Track* 调用中重写。
* **会话一致性**：用户的会话。 ID 设置为生成的值，当用户有一段时间处于非活动状态时，此值会更改。
* **User**：用户信息。

## <a name="limits"></a>限制

[!INCLUDE [application-insights-limits](../../../includes/application-insights-limits.md)]

若要避免达到数据速率限制，请使用[采样](../../azure-monitor/app/sampling.md)。

若要确定保留数据的时间期限，请参阅[数据保留和隐私](../../azure-monitor/app/data-retention-privacy.md)。

## <a name="reference-docs"></a>参考文档

* [ASP.NET 参考](https://msdn.microsoft.com/library/dn817570.aspx)
* [Java 参考](http://dl.windowsazure.com/applicationinsights/javadoc/)
* [JavaScript 参考](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md)
* [Android SDK](https://github.com/Microsoft/ApplicationInsights-Android)
* [iOS SDK](https://github.com/Microsoft/ApplicationInsights-iOS)

## <a name="sdk-code"></a>SDK 代码

* [ASP.NET Core SDK](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [ASP.NET 5](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Windows Server 包](https://github.com/Microsoft/applicationInsights-dotnet-server)
* [Java SDK](https://github.com/Microsoft/ApplicationInsights-Java)
* [Node.js SDK](https://github.com/Microsoft/ApplicationInsights-Node.js)
* [JavaScript SDK](https://github.com/Microsoft/ApplicationInsights-JS)
* [所有平台](https://github.com/Microsoft?utf8=%E2%9C%93&query=applicationInsights)

## <a name="questions"></a>问题

* *Track_() 调用可能会引发哪些异常？*

    无。 不需要将它们包装在 try-catch 子句中。 如果 SDK 遇到问题，它会在调试控制台输出中记录消息，如果消息已传入，可在诊断搜索中查看。
* *是否可以使用某个 REST API 从门户获取数据？*

    是的，可以使用[数据访问 API](https://dev.applicationinsights.io/)。 提取数据的其他方法包括[从 Analytics 导出到 Power BI](../../azure-monitor/app/export-power-bi.md ) 和[连续导出](../../azure-monitor/app/export-telemetry.md)。

## <a name="next"></a>后续步骤

* [搜索事件和日志](../../azure-monitor/app/diagnostic-search.md)
* [故障排除](../../azure-monitor/app/troubleshoot-faq.md)
