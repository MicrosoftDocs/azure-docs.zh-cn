---
title: 使用 ILogger 在 Azure Application Insights 中浏览 .NET 跟踪日志
description: ASP.NET Core 和控制台应用程序中使用 Azure Application Insights ILogger 提供程序的示例。
services: application-insights
author: cijothomas
manager: carmonm
ms.service: application-insights
ms.topic: conceptual
ms.date: 02/19/2019
ms.reviewer: mbullwin
ms.author: cithomas
ms.openlocfilehash: ca842ce46a58dafa87581b77bcbd802191f7fcd1
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65511034"
---
# <a name="applicationinsightsloggerprovider-for-net-core-ilogger-logs"></a>对于.NET Core ILogger 日志 ApplicationInsightsLoggerProvider

ASP.NET Core 支持适用于不同种类的内置和第三方日志记录提供程序的日志记录 API。 通过调用进行日志记录**log （)** 或其上的一个变体*ILogger*实例。 本文演示如何使用*ApplicationInsightsLoggerProvider*捕获 ILogger 控制台和 ASP.NET Core 应用程序中的日志。 本文还介绍了 ApplicationInsightsLoggerProvider 如何与其他 Application Insights 遥测数据相集成。
有关详细信息，请参阅[登录 ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/logging)。

## <a name="aspnet-core-applications"></a>ASP.NET Core 应用程序

默认情况下启用 ApplicationInsightsLoggerProvider [Microsoft.ApplicationInsights.AspNet SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore)版本 2.7.0-beta3 （及更高版本） 当你启用正则 Application Insights 监视通过标准的方法：
- 通过调用**UseApplicationInsights** IWebHostBuilder 扩展方法 
- 通过调用**AddApplicationInsightsTelemetry** IServiceCollection 的扩展方法

ILogger ApplicationInsightsLoggerProvider 捕获的日志可能会有所相同的配置收集的任何其他遥测数据。 它们具有一组相同的 TelemetryInitializers 和 TelemetryProcessors、 使用相同的 TelemetryChannel，和相关和相同的方式与其他遥测数据采样。 如果使用版本 2.7.0-beta3 或更高版本，则无需操作来捕获 ILogger 日志。

仅*警告*或更高版本 （从所有类别） 的 ILogger 日志默认情况下发送到 Application Insights。 但你可以[应用筛选器来修改此行为](#control-logging-level)。 其他步骤所需 ILogger 从捕获日志**Program.cs**或**Startup.cs**。 (请参阅[捕获 ILogger 在 ASP.NET Core 应用程序中记录从 Startup.cs 和 Program.cs](#capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps)。)

如果使用早期版本的 Microsoft.ApplicationInsights.AspNet SDK 或者想要只需使用 ApplicationInsightsLoggerProvider 而无需任何其他 Application Insights 监视，使用以下过程：

1. 安装 NuGet 包：

   ```xml
       <ItemGroup>
         <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.9.1" />  
       </ItemGroup>
   ```

1. 修改**Program.cs**如下所示：

   ```csharp
   using Microsoft.AspNetCore;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.Extensions.Logging;

   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
         WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>()
         .ConfigureLogging(
               builder =>
               {
                   // Providing an instrumentation key here is required if you're using
                   // standalone package Microsoft.Extensions.Logging.ApplicationInsights
                   // or if you want to capture logs from early in the application startup
                   // pipeline from Startup.cs or Program.cs itself.
                   builder.AddApplicationInsights("ikey");

                   // Optional: Apply filters to control what logs are sent to Application Insights.
                   // The following configures LogLevel Information or above to be sent to
                   // Application Insights for all categories.
                   builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                                    ("", LogLevel.Information);
               }
           );
   }
   ```

在步骤 2 中的代码配置`ApplicationInsightsLoggerProvider`。 下面的代码演示一个示例控制器类，该类使用`ILogger`发送日志。 Application Insights 捕获日志。

```csharp
public class ValuesController : ControllerBase
{
    private readonly `ILogger` _logger;

    public ValuesController(ILogger<ValuesController> logger)
    {
        _logger = logger;
    }

    // GET api/values
    [HttpGet]
    public ActionResult<IEnumerable<string>> Get()
    {
        // All the following logs will be picked up by Application Insights.
        // and all of them will have ("MyKey", "MyValue") in Properties.
        using (_logger.BeginScope(new Dictionary<string, object> { { "MyKey", "MyValue" } }))
            {
                _logger.LogWarning("An example of a Warning trace..");
                _logger.LogError("An example of an Error level message");
            }
        return new string[] { "value1", "value2" };
    }
}
```

### <a name="capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps"></a>捕获 Startup.cs 和 Program.cs 中 ASP.NET Core 应用中的 ILogger 日志

新 ApplicationInsightsLoggerProvider 可以捕获从尽早在应用程序启动管道中的日志。 尽管 ApplicationInsightsLoggerProvider 自动启用 Application Insights （从版本 2.7.0-beta3 开始） 中，但是它没有在管道中的更高版本才设置检测密钥。 因此，仅记录来自**控制器**/ 将捕获的其他类。 若要捕获从每个日志**Program.cs**并**Startup.cs**本身，你必须显式启用检测密钥的 ApplicationInsightsLoggerProvider。 此外， *TelemetryConfiguration*未完全设置时记录来自**Program.cs**或**Startup.cs**本身。 因此这些日志将具有使用 InMemoryChannel、 任何采样，并没有标准遥测初始值设定项或处理器的最小配置。

下面的示例演示此功能**Program.cs**并**Startup.cs**。

#### <a name="example-programcs"></a>示例 Program.cs

```csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Logging;

public class Program
{
    public static void Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();
        var logger = host.Services.GetRequiredService<ILogger<Program>>();
        // This will be picked up by AI
        logger.LogInformation("From Program. Running the host now..");
        host.Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureLogging(
        builder =>
            {
            // Providing an instrumentation key here is required if you're using
            // standalone package Microsoft.Extensions.Logging.ApplicationInsights
            // or if you want to capture logs from early in the application startup 
            // pipeline from Startup.cs or Program.cs itself.
            builder.AddApplicationInsights("ikey");

            // Adding the filter below to ensure logs of all severity from Program.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Program).FullName, LogLevel.Trace);

            // Adding the filter below to ensure logs of all severity from Startup.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Startup).FullName, LogLevel.Trace);
            }
        );
}
```

#### <a name="example-startupcs"></a>示例 Startup.cs

```csharp
public class Startup
{
    private readonly `ILogger` _logger;

    public Startup(IConfiguration configuration, ILogger<Startup> logger)
    {
        Configuration = configuration;
        _logger = logger;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();

        // The following will be picked up by Application Insights.
        _logger.LogInformation("Logging from ConfigureServices.");
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            // The following will be picked up by Application Insights.
            _logger.LogInformation("Configuring for Development environment");
            app.UseDeveloperExceptionPage();
        }
        else
        {
            // The following will be picked up by Application Insights.
            _logger.LogInformation("Configuring for Production environment");
        }

        app.UseMvc();
    }
}
```

## <a name="migrate-from-the-old-applicationinsightsloggerprovider"></a>从旧 ApplicationInsightsLoggerProvider 迁移

Microsoft.ApplicationInsights.AspNet SDK 版本之后才 2.7.0-beta2 支持的日志记录提供程序现已过时。 此提供程序通过启用**AddApplicationInsights()** ILoggerFactory 的扩展方法。 我们建议你将迁移到新的提供程序，这涉及到两个步骤：

1. 删除*ILoggerFactory.AddApplicationInsights()* 从调用**Startup.Configure()** 方法可避免双日志记录。
2. 重新应用任何筛选规则在代码中，因为它们不会考虑新的提供程序。 重载*ILoggerFactory.AddApplicationInsights()* 花费了最小日志级别或筛选器函数。 使用新的提供程序，筛选是日志记录框架本身的一部分。 它不是由 Application Insights 提供程序。 通过提供因此任何筛选器*ILoggerFactory.AddApplicationInsights()* 重载应删除。 和筛选规则应提供按照[控制日志记录级别](#control-logging-level)说明。 如果您使用*appsettings.json*若要筛选日志记录，它将继续使用新的提供程序，因为两者都使用相同的提供程序别名*ApplicationInsights*。

您仍然可以使用旧的提供程序。 （仅在主版本更改为 3 中将删除。*xx*。)但我们建议迁移到新的提供程序由于以下原因：

- 之前的提供程序缺少对的支持[日志作用域](https://docs.microsoft.com/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#log-scopes)。 在新的提供程序，从作用域的属性将自动添加为自定义属性到收集的遥测。
- 现在可以更早地在应用程序启动管道捕获日志。 从日志**程序**并**启动**类现在可以将捕获。
- 使用新的提供程序进行筛选在框架级别本身。 您可以在用于其他提供程序，包括控制台中，调试，如内置提供程序的相同方式筛选日志传输到 Application Insights 提供程序等。 您还可以应用于多个提供程序相同的筛选器。
- ASP.NET Core 中 （2.0 及更高版本） 的推荐方式[启用日志记录提供程序](https://github.com/aspnet/Announcements/issues/255)是上 ILoggingBuilder 中使用扩展方法**Program.cs**本身。

> [!Note]
> 新的提供程序是可用于面向 NETSTANDARD2.0 的应用程序或更高版本。 如果应用程序面向较旧的.NET Core 版本，如.NET Core 1.1 中，或者如果面向.NET Framework，继续使用旧的提供程序。

## <a name="console-application"></a>控制台应用程序

下面的代码演示的示例控制台应用程序配置为向 Application Insights 发送 ILogger 跟踪。

已安装的包：

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="2.1.0" />  
  <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.9.1" />
  <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="2.1.0" />
</ItemGroup>
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        // Create the DI container.
        IServiceCollection services = new ServiceCollection();

        // Channel is explicitly configured to do flush on it later.
        var channel = new InMemoryChannel();
        services.Configure<TelemetryConfiguration>(
            (config) =>
            {
                config.TelemetryChannel = channel;
            }
        );

        // Add the logging pipelines to use. We are using Application Insights only here.
        services.AddLogging(builder =>
        {
            // Optional: Apply filters to configure LogLevel Trace or above is sent to
            // Application Insights for all categories.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             ("", LogLevel.Trace);
            builder.AddApplicationInsights("--YourAIKeyHere--");
        });

        // Build ServiceProvider.
        IServiceProvider serviceProvider = services.BuildServiceProvider();

        ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

        // Begin a new scope. This is optional.
        using (logger.BeginScope(new Dictionary<string, object> { { "Method", nameof(Main) } }))
        {
            logger.LogInformation("Logger is working"); // this will be captured by Application Insights.
        }

        // Explicitly call Flush() followed by sleep is required in Console Apps.
        // This is to ensure that even if application terminates, telemetry is sent to the back-end.
        channel.Flush();
        Thread.Sleep(1000);
    }
}
```

此示例使用的独立包`Microsoft.Extensions.Logging.ApplicationInsights`。 默认情况下，此配置使用"最低"将数据发送到 Application Insights 的 TelemetryConfiguration。 最低要求意味着 InMemoryChannel 是使用的通道。 没有任何采样和标准 TelemetryInitializers。 可以为控制台应用程序，如以下示例所示重写此行为。

安装此附加包：

```xml
<PackageReference Include="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" Version="2.9.1" />
```

以下部分介绍如何使用重写默认的 TelemetryConfiguration**服务。配置<TelemetryConfiguration>（)** 方法。 此示例设置`ServerTelemetryChannel`和采样。 它将自定义 ITelemetryInitializer 添加到的 TelemetryConfiguration。

```csharp
    // Create the DI container.
    IServiceCollection services = new ServiceCollection();
    var serverChannel = new ServerTelemetryChannel();
    services.Configure<TelemetryConfiguration>(
        (config) =>
        {
            config.TelemetryChannel = serverChannel;
            config.TelemetryInitializers.Add(new MyTelemetryInitalizer());
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder.UseSampling(5);
            serverChannel.Initialize(config);
        }
    );

    // Add the logging pipelines to use. We are adding Application Insights only.
    services.AddLogging(loggingBuilder =>
    {
        loggingBuilder.AddApplicationInsights();
    });

    ........
    ........

    // Explicitly calling Flush() followed by sleep is required in Console Apps.
    // This is to ensure that even if the application terminates, telemetry is sent to the back end.
    serverChannel.Flush();
    Thread.Sleep(1000);
```

## <a name="control-logging-level"></a>控制日志记录级别

ASP.NET Core *ILogger*基础结构具有内置的机制来应用[日志筛选](https://docs.microsoft.com/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#log-filtering)。 这样可以控制发送到每个已注册的提供程序，包括 Application Insights 提供程序的日志。 筛选可进行配置中 (通常通过使用*appsettings.json*文件) 或在代码中。 由框架本身提供此功能。 它并不特定于 Application Insights 提供程序。

下面的示例对 ApplicationInsightsLoggerProvider 应用筛选器规则。

### <a name="create-filter-rules-in-configuration-with-appsettingsjson"></a>使用 appsettings.json 配置中创建筛选器规则

对于 ApplicationInsightsLoggerProvider，提供程序别名是`ApplicationInsights`。 以下部分*appsettings.json*配置为日志*警告*及更高版本所有类别和*错误*及更高版本以开头的类别"Microsoft"发送到`ApplicationInsightsLoggerProvider`。

```json
{
  "Logging": {
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "Warning",
        "Microsoft": "Error"
      }
    },
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="create-filter-rules-in-code"></a>在代码中创建筛选器规则

以下代码段配置日志*警告*及更高版本以及所有类别*错误*及更高版本开头"Microsoft"发送到的类别中`ApplicationInsightsLoggerProvider`。 此配置是与在上一节中的相同*appsettings.json*。

```csharp
    WebHost.CreateDefaultBuilder(args)
    .UseStartup<Startup>()
    .ConfigureLogging(logging =>
      logging.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("", LogLevel.Warning)
             .AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("Microsoft", LogLevel.Error);
```

## <a name="frequently-asked-questions"></a>常见问题

### <a name="what-are-the-old-and-new-versions-of-applicationinsightsloggerprovider"></a>ApplicationInsightsLoggerProvider 的新旧版本有哪些？

[Microsoft.ApplicationInsights.AspNet SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore)包括已启用通过内置 ApplicationInsightsLoggerProvider (Microsoft.ApplicationInsights.AspNetCore.Logging.ApplicationInsightsLoggerProvider) **ILoggerFactory**扩展方法。 此提供程序是标记为已从版本 2.7.0-beta2 过时。 它将在下一步的主版本更改中完全删除。 [Microsoft.ApplicationInsights.AspNetCore 2.6.1](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore)包本身不是已过时。 需要启用监视的请求、 依赖项，等等。

建议的替代方法是新的独立包[Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights)，其中包含一个改进的 ApplicationInsightsLoggerProvider （Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider) 和 ILoggerBuilder 上启用它的扩展方法。

[Microsoft.ApplicationInsights.AspNet SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore)版本 2.7.0-beta3 依赖于新的包，并自动启用 ILogger 捕获。

### <a name="why-are-some-ilogger-logs-shown-twice-in-application-insights"></a>为什么某些 ILogger 日志显示在 Application Insights 中两次？

如果您有旧版的情况下调用启用 ApplicationInsightsLoggerProvider （现在已过时），则会发生重复`AddApplicationInsights`上`ILoggerFactory`。 如果检查你**配置**方法具有以下内容，并将其删除：

```csharp
 public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
 {
     loggerFactory.AddApplicationInsights(app.ApplicationServices, LogLevel.Warning);
     // ..other code.
 }
```

如果从 Visual Studio 调试时遇到双日志记录，请设置`EnableDebugLogger`到*false*中启用 Application Insights 中，按如下所示的代码。 在调试应用程序时，此重复和修补程序才适用。

```csharp
 public void ConfigureServices(IServiceCollection services)
 {
     ApplicationInsightsServiceOptions options = new ApplicationInsightsServiceOptions();
     options.EnableDebugLogger = false;
     services.AddApplicationInsightsTelemetry(options);
     // ..other code.
 }
```

### <a name="i-updated-to-microsoftapplicationinsightsaspnet-sdkhttpswwwnugetorgpackagesmicrosoftapplicationinsightsaspnetcore-version-270-beta3-and-logs-from-ilogger-are-captured-automatically-how-do-i-turn-off-this-feature-completely"></a>我已更新到[Microsoft.ApplicationInsights.AspNet SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore)自动捕获版本 2.7.0-beta3 和 ILogger 中的日志。 如何关闭此功能完全？

请参阅[控制日志记录级别](../../azure-monitor/app/ilogger.md#control-logging-level)部分，以查看如何筛选记录一般情况下。 到加以 ApplicationInsightsLoggerProvider 使用`LogLevel.None`:

**在代码中：**

```csharp
    builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                      ("", LogLevel.None);
```

**在配置：**

```json
{
  "Logging": {
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "None"
      }
}
```

### <a name="why-do-some-ilogger-logs-not-have-the-same-properties-as-others"></a>为什么某些 ILogger 日志没有与其他人相同的属性？

Application Insights 捕获，并使用相同的 TelemetryConfiguration 用于每个其他遥测数据将发送 ILogger 日志。 但出现异常。 默认情况下的 TelemetryConfiguration 未完全设置时记录来自**Program.cs**或**Startup.cs**。 从这些位置的日志不会具有默认配置中，因此它们不会运行所有 TelemetryInitializers 和 TelemetryProcessors。

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-i-want-to-log-some-additional-custom-telemetry-manually-how-should-i-do-that"></a>我使用的独立包 Microsoft.Extensions.Logging.ApplicationInsights，并且我想要手动记录一些附加的自定义遥测。 我应该如何做的？

当使用独立包`TelemetryClient`不注入到 DI 容器，因此需要创建的新实例`TelemetryClient`和使用相同的配置和记录器提供程序使用，如以下代码所示。 这可确保相同的配置用于所有自定义遥测数据以及来自 ILogger 的遥测数据。

```csharp
public class MyController : ApiController
{
   // This telemtryclient can be used to track additional telemetry using TrackXXX() api.
   private readonly TelemetryClient _telemetryClient;
   private readonly ILogger _logger;

   public MyController(IOptions<TelemetryConfiguration> options, ILogger<MyController> logger)
   {
        _telemetryClient = new TelemetryClient(options.Value);
        _logger = logger;
   }  
}
```

> [!NOTE]
> 如果使用的 Microsoft.ApplicationInsights.AspNetCore 包以启用 Application Insights，修改此代码以获取`TelemetryClient`直接在构造函数中。 有关示例，请参阅[此常见问题](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core-no-visualstudio#frequently-asked-questions)。


### <a name="what-application-insights-telemetry-type-is-produced-from-ilogger-logs-or-where-can-i-see-ilogger-logs-in-application-insights"></a>Application Insights 遥测类型产生的 ILogger 日志？ 或者，可以在何处查看 ILogger 中 Application Insights 的日志？

ApplicationInsightsLoggerProvider 捕获 ILogger 的日志，并根据这些创建 TraceTelemetry。 如果异常对象传递给**log （)** ILogger，方法*ExceptionTelemetry*创建而不是 TraceTelemetry。 应用程序见解，包括门户、 分析或 Visual Studio 本地调试器，可以为任何其他 TraceTelemetry 或 ExceptionTelemetry 时所在的位置中找到这些遥测项。

如果想要始终发送 TraceTelemetry，使用此代码片段： ```builder.AddApplicationInsights((opt) => opt.TrackExceptionsAsExceptionTelemetry = false);```

### <a name="i-dont-have-the-sdk-installed-and-i-use-the-azure-web-apps-extension-to-enable-application-insights-for-my-aspnet-core-applications-how-do-i-use-the-new-provider"></a>我没有安装 SDK，并且使用的 Azure Web 应用程序扩展来为我的 ASP.NET Core 应用程序启用 Application Insights。 如何使用新的提供程序？ 

Application Insights 扩展在 Azure Web 应用中的使用旧的提供程序。 您可以修改中的筛选规则*appsettings.json*应用程序文件。 若要充分利用新的提供程序，使用通过将 SDK 上的 NuGet 依赖项生成时检测。 扩展名切换为使用新的提供程序时，将更新这篇文章。

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-enabling-application-insights-provider-by-calling-builderaddapplicationinsightsikey-is-there-an-option-to-get-an-instrumentation-key-from-configuration"></a>我使用的独立包 Microsoft.Extensions.Logging.ApplicationInsights 并启用 Application Insights 提供程序通过调用**生成器。AddApplicationInsights("ikey")**。 是否有从配置获取检测密钥的选项？


修改 Program.cs 和 appsettings.json，如下所示：

   ```csharp
   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
           WebHost.CreateDefaultBuilder(args)
               .UseStartup<Startup>()
               .ConfigureLogging((hostingContext, logging) =>
               {
                   // hostingContext.HostingEnvironment can be used to determine environments as well.
                   var appInsightKey = hostingContext.Configuration["myikeyfromconfig"];
                   logging.AddApplicationInsights(appInsightKey);
               });
   }
   ```

   从相关部分`appsettings.json`:

   ```json
   {
     "myikeyfromconfig": "putrealikeyhere"
   }
   ```

此代码是必需的仅当你使用独立的日志记录提供程序。 有关正则 Application Insights 监视的检测密钥自动从加载的配置路径*ApplicationInsights:Instrumentationkey*。 Appsettings.json 应如下所示：

   ```json
   {
     "ApplicationInsights":
       {
           "Instrumentationkey":"putrealikeyhere"
       }
   }
   ```

## <a name="next-steps"></a>后续步骤

了解有关以下方面的详细信息：

* [在 ASP.NET Core 中的日志记录](https://docs.microsoft.com/aspnet/core/fundamentals/logging)
* [在 Application Insights 中的.NET 跟踪日志](../../azure-monitor/app/asp-net-trace-logs.md)
