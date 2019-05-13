---
title: 在.NET Azure Functions 中使用依赖关系注入
description: 了解如何注册和使用服务在.NET 函数中使用依赖关系注入
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函数, 无服务器体系结构
ms.service: azure-functions
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/22/2019
ms.author: jehollan
ms.openlocfilehash: 2044718d2ec7a7acc58e1e7ba9ba04ec5caf16b3
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408448"
---
# <a name="use-dependency-injection-in-net-azure-functions"></a>在.NET Azure Functions 中使用依赖关系注入

Azure Functions 支持依赖关系注入 (DI) 软件设计模式，这种方法实现[控制反转 (IoC)](https://docs.microsoft.com/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion)类和其依赖项之间。

ASP.NET Core 依赖关系注入功能的基础上构建 azure 函数。  您应了解服务、 生命周期和的设计模式[ASP.NET Core 依赖关系注入](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection)之前在函数中使用它们。

## <a name="installing-dependency-injection-packages"></a>安装依赖项注入包

若要使用的依赖关系注入功能，您需要包括公开这些 Api 的 NuGet 包。

```powershell
Install-Package Microsoft.Azure.Functions.Extensions
```

## <a name="registering-services"></a>注册服务

若要注册服务，可以创建一个配置方法，并将组件添加到`IFunctionsHostBuilder`实例。  Azure Functions 主机创建`IFunctionsHostBuilder`并将其直接传入你配置的方法。

若要注册你配置方法，则必须添加一个指定的类型的程序集特性你配置的方法使用`FunctionsStartup`属性。

```csharp
[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();
            builder.Services.AddSingleton((s) => {
                return new CosmosClient(Environment.GetEnvironmentVariable("COSMOSDB_CONNECTIONSTRING"));
            });
            builder.Services.AddSingleton<ILoggerProvider, MyLoggerProvider>();
        }
    }
}
```

## <a name="service-lifetimes"></a>服务生存期

提供的 azure 函数应用中为相同的服务生存期[ASP.NET 依赖关系注入](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection#service-lifetimes)、 暂时、 作用域，和单一实例。

在函数应用中，作用域内的服务生存期匹配函数执行生存期。 每次执行一次创建作用域的服务。  对该服务在执行过程的更高版本请求重复使用该实例。  单一实例服务生存期匹配主机生存期，并在该实例上的函数执行重复使用。

单例生存期服务建议使用连接和客户端，例如`SqlConnection`， `CloudBlobClient`，或`HttpClient`。

查看或下载[示例的不同服务生存期](https://aka.ms/functions/di-sample)。

## <a name="logging-services"></a>日志记录服务

如果您需要自己的日志记录提供程序，建议的方法是注册`ILoggerProvider`。  Application Insights 的函数会将添加 Application Insights 会自动为您。  

> [!WARNING]
> 请不要添加`AddApplicationInsightsTelemetry()`到服务集合因为它将注册发生冲突的服务使用什么环境提供的。 
 
## <a name="function-app-provided-services"></a>函数提供应用服务

函数主机会注册许多服务本身。  下面是服务都依赖于安全的。  不支持其他主机服务以注册或依赖于。  如果没有其他服务，你想要创建一个依赖项，请[在 GitHub 上创建的问题和讨论](https://github.com/azure/azure-functions-host)。

|服务类型|生存期|描述|
|--|--|--|
|`Microsoft.Extensions.Configuration.IConfiguration`|单一实例|运行时配置|
|`Microsoft.Azure.WebJobs.Host.Executors.IHostIdProvider`|单一实例|负责提供的主机实例的 ID|

### <a name="overriding-host-services"></a>重写主机服务

目前不支持重写由主机提供服务。  如果要重写的服务，请[在 GitHub 上创建的问题和讨论](https://github.com/azure/azure-functions-host)。

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅以下资源：

* [如何监视 function app](functions-monitoring.md)
* [Functions 最佳做法](functions-best-practices.md)