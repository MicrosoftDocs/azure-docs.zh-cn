---
title: Azure Application Insights 中的遥测采样 | Microsoft 文档
description: 如何使受控制的遥测数据的卷。
services: application-insights
documentationcenter: windows
author: cijothomas
manager: carmonm
ms.assetid: 015ab744-d514-42c0-8553-8410eef00368
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 03/14/2019
ms.reviewer: vitalyg
ms.author: cithomas
ms.openlocfilehash: d88de2bf660165022b39aaa0321ff5c62ea81cd3
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231846"
---
# <a name="sampling-in-application-insights"></a>在 Application Insights 中采样

采样是 [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) 中的一项功能。 建议用于降低遥测流量和存储，同时保留采用统计方式的应用程序数据的正确分析。 筛选器会选择相关项，以便可以在执行诊断调查时在不同项之间导航。
当指标计数显示在门户中时，它们将重新标准化以考虑采样。 执行此操作的统计信息的任何影响降至最低。

采样可以降低流量和数据成本，并可帮助用户避免限制。

## <a name="in-brief"></a>简单地说：

* 采样会保留 *n* 条记录中的 1 条并丢弃其余记录。 例如，它可能会保留一中五个事件，采样率为 20%。 
* 默认情况下，ASP.NET 和 ASP.NET Core 软件开发工具包 (Sdk) 的所有最新版本中启用自适应采样。
* 此外可以设置手动采样。 这可以在门户中配置，在*使用情况和预估的成本页*，ApplicationInsights.config 文件中的 ASP.NET sdk，通过代码的 ASP.NET Core sdk 或 ApplicationInsights.xml 中的 Java SDK 文件。
* 如果记录自定义事件和需要确保保留或放弃在一起的一组事件，事件必须具有相同的 OperationId 值。
* 采样除数 *n* 会在每个记录的属性 `itemCount` 中报告，在“搜索”中它出现在友好名称“请求计数”或“事件计数”下。 `itemCount==1`当采样不是在操作中。
* 如果要编写分析查询，应[考虑采样](../../azure-monitor/log-query/aggregations.md)。 特别是，应使用 `summarize sum(itemCount)`，而不是仅对记录进行计数。

## <a name="types-of-sampling"></a>采样类型

有三种备用采样方法：

* **自适应采样**自动调整从 ASP.NET/ASP.NET 核心应用程序中的 SDK 发送遥测数据量。 这是从 ASP.NET Web SDK v 2.0.0-beta3 及更高版本和 Microsoft.ApplicationInsights.AspNetCore SDK v 2.2.0-beta1 及更高版本默认采样。  自适应采样目前仅适用于 ASP.NET 服务器端遥测。

* **固定速率采样**减少从应用 ASP.NET 或 ASP.NET Core 或 Java 服务器和用户的浏览器发送遥测数据的量。 用户设定速率。 客户端和服务器将同步其采样，以便在“搜索”中可以在多个相关页面视图和请求之间导航。

* **引入采样**在 Azure 门户中的工作。 它会以设置的采样率丢弃一些来自应用的遥测数据。 它不会减少应用发送的遥测流量，但可帮助保持在每月配额内。 引入采样的主要优点是您可以无需重新部署您的应用程序设置的采样率。 引入采样统一适用于所有服务器和客户端。

如果自适应采样或固定速率采样正在运行，将禁用引入采样。


## <a name="adaptive-sampling-in-your-aspnetaspnet-core-web-applications"></a>ASP.NET/ASP.NET Core Web 应用程序中的自适应采样

自适应采样是适用于 Application Insights SDK ASP.NET v 2.0.0-beta3 及更高版本，Microsoft.ApplicationInsights.AspNetCore SDK v 2.2.0-beta1 及更高版本，并且默认情况下启用。

自适应采样会影响从 Web 服务器应用发送至 Application Insights 服务终结点的遥测量。 该卷自动调整以保持在指定的最大速率的流量，并通过设置控制`MaxTelemetryItemsPerSecond`。 如果应用程序会生成很少的遥测数据，如调试或由于低使用率，只要下面是卷，不会获取采样项`MaxTelemetryItemsPerSecond`。 与遥测数据会增加卷，以便实现目标量调整采样频率。

为了实现目标量，一些生成的遥测会被丢弃。 但与其他类型的采样一样，该算法会保留相关的遥测项。 例如，在“搜索”中检查遥测数据时，可以查找与特定异常相关的请求。

诸如请求速率和异常率等指标计数将进行调整以补偿采样率，以便它们在指标资源管理器中能够显示大约正确的值。

## <a name="configuring-adaptive-sampling-for-aspnet-applications"></a>配置 ASP.NET 应用程序的自适应采样

[了解](../../azure-monitor/app/sampling.md#configuring-adaptive-sampling-for-aspnet-core-applications)有关配置为 ASP.NET Core 应用程序的自适应采样。 

在 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 中，可以调整 `AdaptiveSamplingTelemetryProcessor` 节点中的多个参数。 显示的数字是默认值：

* `<MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>`
  
    自适应算法的目标速率旨在针对**每个服务器主机**。 如果在多台主机上运行 Web 应用，应减小此值以保留在 Application Insights 门户的目标流量速率以内。
* `<EvaluationInterval>00:00:15</EvaluationInterval>` 
  
    重新评估当前遥测速率的间隔。 评估以移动平均线形式进行执行。 可能想要缩短此间隔（如果遥测很容易就激增）。
* `<SamplingPercentageDecreaseTimeout>00:02:00</SamplingPercentageDecreaseTimeout>`
  
    当采样百分比值更改时，多久后我们可以再次降低采样百分比以便捕获更少数据。
* `<SamplingPercentageIncreaseTimeout>00:15:00</SamplingPercentageIncreaseTimeout>`
  
    当采样百分比值更改时，多久后我们可以再次增加采样百分比以便捕获更多数据。
* `<MinSamplingPercentage>0.1</MinSamplingPercentage>`
  
    随着采样百分比的变化，我们可以设置的最小值是多少。
* `<MaxSamplingPercentage>100.0</MaxSamplingPercentage>`
  
    随着采样百分比的变化，我们可以设置的最大值是多少。
* `<MovingAverageRatio>0.25</MovingAverageRatio>` 
  
    在移动平均线的计算中，权重分配给最新的值。 使用等于或小于 1 的值。 较小的值会使算法不易受突然的更改影响。
* `<InitialSamplingPercentage>100</InitialSamplingPercentage>`
  
    当应用已启动时分配的值。 在进行调试时，不减小值。

* `<ExcludedTypes>Trace;Exception</ExcludedTypes>`
  
    不要采样的分号分隔类型列表。 已识别的类型包括：Dependency、Event、Exception、PageView、Request、Trace。 指定类型的所有实例都会传输；对未指定的类型进行采样。

* `<IncludedTypes>Request;Dependency</IncludedTypes>`
  
    要采样的分号分隔类型列表。 已识别的类型包括：Dependency、Event、Exception、PageView、Request、Trace。 将对指定的类型进行采样；其他类型的所有实例始终都会传输。


**若要关闭**自适应采样，从 applicationinsights-config 中删除 AdaptiveSamplingTelemetryProcessor 节点。

### <a name="alternative-configure-adaptive-sampling-in-code"></a>备选： 在代码中配置自适应采样

除了在 .config 文件中设置采样参数以外，还可以编程方式设置这些值。

1. 删除所有`AdaptiveSamplingTelemetryProcessor`.config 文件中的节点。
2. 使用以下代码片段来配置自适应采样。

*C#*

```csharp

    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.Channel.Implementation;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    ...

    var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;
    // If you are on ApplicationInsights SDK v 2.8.0-beta2 or higher, use the following line instead
    // var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;

    // Enable AdaptiveSampling so as to keep overall telemetry volume to 5 items per second.
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();

```

（[了解遥测处理器](../../azure-monitor/app/api-filtering-sampling.md#filtering)。）

此外可以调整每个遥测类型采样率，或甚至可以从所有采样中排除特定类型。 

*C#*

```csharp
    // The following configures adaptive sampling with 5 items per second, and also excludes DependencyTelemetry from being subject to sampling.
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");
```

## <a name="configuring-adaptive-sampling-for-aspnet-core-applications"></a>配置为 ASP.NET Core 应用程序的自适应采样。

没有任何`ApplicationInsights.Config`对于 ASP.NET Core 应用程序，因此每一种配置是通过代码。
默认情况下会为所有 ASP.NET Core 应用程序启用自适应采样。 你可以禁用或自定义采样行为。

### <a name="turning-off-adaptive-sampling"></a>禁用自适应采样

可以在方法中添加 Application Insights 服务时禁用默认采样功能```ConfigureServices```，并使用```ApplicationInsightsServiceOptions```内`Startup.cs`文件：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetry(aiOptions);
    //...
}
```

上面的代码将禁用采样功能。 可以执行以下步骤使用更多自定义选项来添加采样。

### <a name="configure-sampling-settings"></a>配置采样设置

如下所示使用 ```TelemetryProcessorChainBuilder``` 的扩展方法来自定义采样行为。

> [!IMPORTANT]
> 如果使用此方法来配置采样，请确保将 aiOptions.EnableAdaptiveSampling = false; 设置与 AddApplicationInsightsTelemetry() 配合使用。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

    var builder = configuration .TelemetryProcessorChainBuilder;
    // version 2.5.0-beta2 and above should use the following line instead of above. (https://github.com/Microsoft/ApplicationInsights-aspnetcore/blob/develop/CHANGELOG.md#version-250-beta2)
    // var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;

    // Using adaptive sampling
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // Alternately, the following configures adaptive sampling with 5 items per second, and also excludes DependencyTelemetry from being subject to sampling.
    // builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");

    builder.Build();

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));
    // ...
}

```

**如果使用上述方法配置采样，请确保使用```aiOptions.EnableAdaptiveSampling = false;```AddApplicationInsightsTelemetry() 设置。**

## <a name="fixed-rate-sampling-for-aspnet-aspnet-core-and-java-websites"></a>ASP.NET、 ASP.NET Core 和 Java 网站的固定速率采样

固定速率采样会减少从 Web 服务器和 Web 浏览器发送的流量。 与自适应采样不同，它会按用户确定的固定速率来降低遥测。 它还将同步客户端和服务器采样，以便保留相关项，例如，如果在“搜索”中查看页面视图，可以查找其相关的请求。

像其他采样技术，这也会保留相关的项。 对于每个 HTTP 请求事件，请求和相关的事件会一起被丢弃或传输。

在指标资源管理器中，诸如请求和异常计数等速率将乘以某个系数来补偿采样率，以便它们大致正确。

### <a name="configuring-fixed-rate-sampling-in-aspnet"></a>使用 ASP.NET 配置固定速率采样

1. **禁用自适应采样**：在 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) 中，删除或注释掉 `AdaptiveSamplingTelemetryProcessor` 节点。

    ```xml

    <TelemetryProcessors>

    <!-- Disabled adaptive sampling:
      <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
        <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
      </Add>
    -->
    ```

2. **启用固定速率采样模块。** 将以下片段添加到 [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md)：
   
    ```XML
   
    <TelemetryProcessors>
     <Add  Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
   
      <!-- Set a percentage close to 100/N where N is an integer. -->
     <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
      <SamplingPercentage>10</SamplingPercentage>
      </Add>
    </TelemetryProcessors>
   
    ```
   ### <a name="alternative-enable-fixed-rate-sampling-in-your-server-code"></a>备选：启用服务器代码中的固定速率采样
    
    除了在 .config 文件中设置采样参数以外，还可以编程方式设置这些值。 

*C#*

```csharp

    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    ...

    var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;
    // If you are on ApplicationInsights SDK v 2.8.0-beta2 or higher, use the following line instead
    // var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;

    builder.UseSampling(10.0); // percentage

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();

```
（[了解遥测处理器](../../azure-monitor/app/api-filtering-sampling.md#filtering)。）

### <a name="configuring-fixed-rate-sampling-in-aspnet-core"></a>在 ASP.NET Core 中配置固定速率采样

1. **禁用自适应采样**：可以在方法中进行更改```ConfigureServices```，并使用```ApplicationInsightsServiceOptions```:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
    // ...

        var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
        aiOptions.EnableAdaptiveSampling = false;
        services.AddApplicationInsightsTelemetry(aiOptions);
    //...
    }
    ```

2. **启用固定速率采样模块。** 可以在方法中进行更改```Configure```如下代码段中所示：

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

        var builder = configuration.TelemetryProcessorChainBuilder;
        // version 2.5.0-beta2 and above should use the following line instead of above. (https://github.com/Microsoft/ApplicationInsights-aspnetcore/blob/develop/CHANGELOG.md#version-250-beta2)
        // var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;

        // Using fixed rate sampling   
        double fixedSamplingPercentage = 10;
        builder.UseSampling(fixedSamplingPercentage);

        builder.Build();

        // ...
    }

    ```

### <a name="configuring-fixed-rate-sampling-in-java"></a>使用 JAVA 配置固定速率采样 ###

1. 使用最新的 [Application Insights Java SDK](../../azure-monitor/app/java-get-started.md)下载并配置 Web 应用程序

2. 通过将下面的代码片段添加到 ApplicationInsights.xml 文件，启用固定速率采样模块。

    ```XML
        <TelemetryProcessors>
            <BuiltInProcessors>
                <Processor type = "FixedRateSamplingTelemetryProcessor">
                    <!-- Set a percentage close to 100/N where N is an integer. -->
                    <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
                    <Add name = "SamplingPercentage" value = "50" />
                </Processor>
            </BuiltInProcessors>
        <TelemetryProcessors/>
    ```

3. 可以使用处理器标记“FixedRateSamplingTelemetryProcessor”中的以下标记，在采样中包括或排除特定类型的遥测
    ```XML
        <ExcludedTypes>
            <ExcludedType>Request</ExcludedType>
        </ExcludedTypes>

        <IncludedTypes>
            <IncludedType>Exception</IncludedType>
        </IncludedTypes>
    ```

可以包括或排除从采样的遥测数据类型为：依赖项、 事件、 异常、 页面视图、 请求和跟踪。

> [!NOTE]
> 对于采样百分比，请选择一个接近于 100/N 的百分比，其中 N 是整数。  当前采样不支持其他值。
> 
> 

<a name="other-web-pages"></a>



## <a name="ingestion-sampling"></a>引入采样

当来自 Web 服务器、浏览器和设备的遥测达到 Application Insights 服务终结点时，将运行此采样形式。 尽管它未减少从应用发送的遥测流量，但它确实减少了由 Application Insights 处理和保留（并收费）的遥测量。

如果应用经常超过其每月配额，且没有使用任一种基于 SDK 的采样类型的选项，请使用此种类型的采样。 

在“使用情况和预估成本”页面中设置采样率：

![从应用程序“概述”边栏选项卡中，依次单击“设置”、“配额”、“示例”，选择采样率，并单击“更新”。](./media/sampling/04.png)

与其他类型的采样一样，该算法会保留相关的遥测项。 例如，在“搜索”中检查遥测数据时，可以查找与特定异常相关的请求。 诸如请求速率和异常率等指标计数会正确保留。

被采样丢弃的数据点将无法在任何 Application Insights 功能（如[连续导出](../../azure-monitor/app/export-telemetry.md)）中使用。

当运行基于 SDK 的自适应采样或固定速率采样时，不会运行引入采样。 在 Visual Studio 中启用 ASP.NET/ASP.NET Core SDK 或将其启用 Azure Web 应用扩展中或通过使用状态监视器，并将禁用引入采样时，默认情况下启用自适应采样。 如果 SDK 的采样率小于 100%（即 项正在采样） 就会忽略您设置引入采样率。

> [!WARNING]
> 磁贴上显示的值指示为引入采样所设定的值。 如果 SDK 采样不在运行，它并不表示实际采样率。
>
>
## <a name="sampling-for-web-pages-with-javascript"></a>使用 JavaScript 的网页采样
可以针对任何服务器中的固定速率采样配置网页。 

[为 Application Insights 配置网页](../../azure-monitor/app/javascript.md)时，修改从 Application Insights 门户获取的 JavaScript 代码片段。 （在 ASP.NET 应用中，代码段通常出现在 _Layout.cshtml 中。）在检测密钥前插入类似于 `samplingPercentage: 10,` 的行：

    <script>
    var appInsights= ... 
    }({ 


    // Value must be 100/N where N is an integer.
    // Valid examples: 50, 25, 20, 10, 5, 1, 0.1, ...
    samplingPercentage: 10, 

    instrumentationKey:...
    }); 

    window.appInsights=appInsights; 
    appInsights.trackPageView(); 
    </script> 

对于采样百分比，请选择一个接近于 100/N 的百分比，其中 N 是整数。  当前采样不支持其他值。

如果还在服务器上启用了固定速率采样，客户端和服务器将同步，以便在“搜索”中可以在多个相关页面视图和请求之间导航。

## <a name="when-to-use-sampling"></a>何时使用采样？

自适应采样会自动启用最新.NET 和.NET Core Sdk 中。 无论使用何种 SDK 版本，都可以启用引入采样，以允许 Application Insights 对收集的数据采样。

默认情况下，Java SDK 中不启用任何采样。 目前，它仅支持固定速率采样。 Java SDK 不支持自适应采样。

一般情况下，对于大多数小型和中型应用程序，不需要采样。 通过收集所有用户活动的数据，获取最有用的诊断信息和最准确的统计。 

采样的主要优势在于：

* 当应用以短时间间隔发送速率很高的遥测时，Application Insights 服务会删除（“限制”）数据点。 
* 保持在定价层的数据点[配额](../../azure-monitor/app/pricing.md)以内。 
* 减少遥测集合中的网络流量。 

### <a name="which-type-of-sampling-should-i-use"></a>应该使用哪种类型的采样？

**在以下情况下使用引入采样：**

* 遥测经常超出每月配额。
* 使用的 SDK 版本不支持采样，例如版本早于 2 的 ASP.NET 版本。
* 可将过多遥测数据从用户的 web 浏览器。

**在以下情况下使用固定速率采样：**

* 使用 Application Insights SDK for ASP.NET Web 服务版本 2.0.0 或更高版本或者 Java SDK v2.0.1 或更高版本，并且
* 希望同步客户端和服务器之间的采样，因此，调查 [搜索](../../azure-monitor/app/diagnostic-search.md) 中的事件时，可以在客户端和服务器上的相关事件之间导航，例如页面视图和 http 请求。
* 对应用的适当采样百分比有信心。 它应该足够高以获取准确指标，但应低于超出定价配额和限制的速率。 

**使用自适应采样：**

如果使用其他采样格式的条件不适用，我们建议使用自适应采样。 ASP.NET/ASP.NET 核心服务器 SDK 中的默认情况下启用此设置。 它不会将流量减少到特定最低速率，因此不会影响使用率较低的站点。

## <a name="how-do-i-know-whether-sampling-is-in-operation"></a>如何知道采样是否在运行？

若要查找实际采样率（无论是否已应用），请使用如下所示的[分析查询](../../azure-monitor/app/analytics.md)：

```
union requests,dependencies,pageViews,browserTimings,exceptions,traces
| where timestamp > ago(1d)
| summarize RetainedPercentage = 100/avg(itemCount) by bin(timestamp, 1h), itemType
```

如果任何类型的 RetainedPercentage 小于 100，该项是进行采样。

**Application Insights 不会采样会话中，度量值和性能计数器中上述任何采样方法的遥测类型。这些类型始终排除采样中，如降低精度可能非常有必要为这些遥测类型**

## <a name="how-does-sampling-work"></a>采样如何运行？

ASP.NET 版本 2.0.0 和 Java SDK 版本 2.0.1 及以上版本中 SDK 的固定速率采样功能。 自适应采样是 ASP.NET 版本（从 2.0.0 开始）中 SDK 的一项功能。 引入采样是 Application Insights 服务的一项功能，并且可以在 SDK 未执行采样时运行。

采样算法决定要删除哪些遥测项，以及要保留哪些遥测项（无论它是在 SDK 中还是在 Application Insights 服务中）。 采样决策取决于多个规则，目标是保留所有相互关联的数据点不变，同时在 Application Insights 中保持可操作和可靠而且即使有精简数据集的诊断体验。 例如，如果有失败的请求，应用汇发送其他遥测项（例如此请求记录的异常和跟踪），采样将不会拆分此请求和其他遥测数据。 采样会将它们一起保留或删除。 因此，在 Application Insights 中查看请求详细信息时，始终可以看到请求以及其关联的遥测项。 

采样决策取决于请求，以表示属于某一特定操作的所有遥测项不保留或删除的操作 ID。 不具有的操作的遥测项的 ID （适用于从且没有 http 上下文在异步线程报告的示例遥测项） 的集采样仅捕获每种类型的遥测项的百分比。 2.5.0-beta2.NET SDK 在和之前的 ASP.NET Core SDK 2.2.0-beta3，采样决定基于定义"用户"的应用程序的用户 ID 的哈希 (即，最典型的 web 应用程序)。 对于未定义用户 （如 web 服务） 的应用程序的类型，采样决定基于请求的操作 ID。

当遥测发回给用户时，Application Insights 服务会以收集时所使用的同一采样百分比来调整指标，以补偿缺失的数据点。 因此，当用户在 Application Insights 中查看遥测数据，会看到统计的正确近似值非常接近于实际值。

近似值的准确性很大程度上取决于配置的采样百分比。 此外，对于处理通常来自大量用户的类似请求的较大量的应用程序，准确性也会提升。 相反，对于不处理大量负载的应用程序，不需要采样，因为这些应用程序通常可以发送其所有遥测的同时保持在配额以内，而不会因限制而造成数据丢失。 

> [!WARNING]
> Application Insights 不会对指标和会话遥测类型采样。 对于这些遥测类型，可能非常有必要降低精度。
> 

### <a name="adaptive-sampling"></a>自适应采样

自适应采样添加用于监视当前 SDK 的传输速率的组件，并调整采样百分比，以便尝试保持在目标最大速率以内。 调整会定期重新计算，计算依据是传出的传输速率的移动平均线。

## <a name="sampling-and-the-javascript-sdk"></a>采样和 JavaScript SDK

客户端 (JavaScript) SDK 与服务器端 SDK 一同参与固定速率采样。 检测到的页面将仅从服务器端决定“纳入采样”的相同用户发送客户端遥测。 此逻辑的设计目的是维护跨客户端和服务器端的用户会话的完整性。 因此，通过 Application Insights 中的任何特定遥测项，可查找此用户或会话的所有其他遥测项。 

*我的客户端和服务器端遥测未显示上面描述的协调示例。*

* 确认已在服务器和客户端上启用固定速率采样。
* 确保 SDK 版本是 2.0 或更高版本。
* 检查在客户端和服务器上设置的采样百分比相同。

## <a name="frequently-asked-questions"></a>常见问题

*什么是 ASP.NET 和 ASP.NET Core SDK 中的默认采样行为？*

* 如果使用更高版本的 SDK 的最新版本，默认情况下，每秒的五个遥测项与启用自适应采样。
  有 2 个 AdaptiveSamplingTelemetryProcessors 默认情况下，添加一个在采样中包括事件类型和其他不采样中包括事件类型。 这种配置意味着 SDK 将尝试限制为五个遥测项的事件类型的遥测项和所有其他类型的组合，从而确保事件与其他遥测类型进行单独采样的五个遥测项。 事件通常用于业务遥测，并很可能应不会影响诊断遥测数据卷。
  
  下面显示了生成的默认 ApplicationInsights.Config 文件。 如所述，有两个单独的 AdaptiveSamplingTelemetryProcessor 节点添加了，其中一个不包括事件类型，另一个包括它。 在 ASP.NET Core 中，代码中启用了完全相同的默认行为。 使用在前面部分的文档的示例来更改此默认行为。

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <ExcludedTypes>Event</ExcludedTypes>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <IncludedTypes>Event</IncludedTypes>
        </Add>
    </TelemetryProcessors>
    ```

*可以一次采样遥测数据？*

* 不。 SamplingTelemetryProcessors 忽略中采样的注意事项，如果该项已采样的项。 同样适用于作为嗯，这不会将采样应用于 SDK 本身中这些项已引入采样。

*为何不采样简单“收集每个遥测类型 %X”？*

* 虽然此采样方法提供使用高级别的精度的指标近似值，它将会破坏功能将每个用户、 会话和请求，这很关键的诊断的诊断数据相关联。 因此，对于“收集 %X 应用用户的所有遥测项”或“收集 %X 应用请求的所有遥测”逻辑，采样的效果更佳。 回退到未与请求 （如后台异步处理） 关联的遥测项，是"收集 %x 的每个遥测类型的所有项。" 

*采样百分比是否随时间变化？*

* 是，自适应采样会根据当前观察到的遥测量，逐渐更改采样百分比。

*如果我使用固定速率采样，如何知道哪一个才样百分比最适合我的应用？*

* 一种方法是从自适应采样开始，了解应用选定的速率（请参阅上述问题），然后使用该速率切换到固定速率采样。 
  
    否则，就必须猜测。 分析 Application Insights 中当前遥测使用量、观察出现的任何限制，并估计所收集的遥测量。 这三个输入以及所选的定价层暗示你可能想要减少收集的遥测的数量。 但是，用户数量的增加或遥测量中的某些其他变化可能导致估计无效。

*如果我配置的采样百分比过低，会发生什么情况？*

* 当 Application Insights 尝试补偿数据量减少的数据可视化效果时，过低的采样百分比（过度采样）会降低近似值的准确性。 此外，诊断体验可能会受到负面影响，由于可能会采样某些很少出现的请求或缓慢请求。

*如果我配置的采样百分比过高，会发生什么情况？*

* 将配置过高的采样百分比（力度不够）会导致收集的遥测量减少不够。 仍可能会遇到与限制相关的遥测数据丢失，而使用 Application Insights 的成本可能因过高费用而超出预算。

*可以在哪些平台上使用采样？*

* 如果 SDK 未执行采样，则对于过于特定量的任何遥测，自动运行引入采样。 此配置将起作用，例如，如果使用的较旧版本的 ASP.NET SDK 或以前版本的 Java SDK(1.0.10 or before)。
* 如果使用的 ASP.NET SDK 版本 2.0.0 及更高版本或 ASP.NET CORE SDK 版本 2.2.0 及更高版本 （托管在 Azure 中或在你自己的服务器上），自适应采样默认情况下，但如上文所述，可以切换到固定速率。 使用固定速率采样，浏览器 SDK 会自动同步到示例相关的事件。 
* 如果使用的 Java SDK 版本 2.0.1 或更高版本，你可以配置 ApplicationInsights.xml 以打开固定速率采样。 默认情况下，采样处于关闭状态。 使用固定速率采样，浏览器 SDK 会自动同步到示例相关的事件。

*我总是想要查看某些罕见的事件。如何让它们通过采样模块？*

* 实现此目的的最佳方法是编写一个自定义[TelemetryProcessor](../../azure-monitor/app/api-filtering-sampling.md#filtering)，哪个集`SamplingPercentage`到 100 之间的遥测项要保留的如下所示。 这可确保所有采样技术将都忽略此项从任何采样注意事项。

```csharp
    if(somecondition)
    {
        ((ISupportSampling)item).SamplingPercentage = 100;
    }
```

## <a name="next-steps"></a>后续步骤

* [筛选](../../azure-monitor/app/api-filtering-sampling.md)可以对 SDK 发送的内容提供更严格地控制。