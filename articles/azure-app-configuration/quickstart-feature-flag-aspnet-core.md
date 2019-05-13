---
title: 有关将功能标志添加到 ASP.NET Core 的快速入门 | Microsoft Docs
description: 有关将功能标志添加到 ASP.NET Core 应用以及在 Azure 应用程序配置中管理这些标志的快速入门
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: quickstart
ms.tgt_pltfrm: ASP.NET Core
ms.workload: tbd
ms.date: 04/19/2019
ms.author: yegu
ms.openlocfilehash: 95f702b1d85dc8fe22b1800df3f7b0ebc987bee5
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411833"
---
# <a name="quickstart-add-feature-flags-to-an-aspnet-core-app"></a>快速入门：将功能标志添加到 ASP.NET Core 应用

可以通过将应用程序连接到 Azure 应用程序配置，来启用 ASP.NET Core 中的功能管理。 使用此托管服务可以存储所有功能标志，以及集中控制其状态。 本快速入门介绍如何将该服务整合到 ASP.NET Core Web 应用，以创建功能管理的端到端实现。

.NET Core 功能管理库使用全面的功能标志支持扩展了该框架。 这些库全部基于 .NET Core 配置系统。 它们可以通过其 .NET Core 配置提供程序无缝集成到应用程序配置。

你可使用任意代码编辑器来执行该快速入门中的步骤。 [Visual Studio Code](https://code.visualstudio.com/) 是 Windows、macOS 和 Linux 平台上提供的一个卓越选项。

## <a name="prerequisites"></a>先决条件

要完成本快速入门，请安装 [.NET Core SDK](https://dotnet.microsoft.com/download)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-app-configuration-store"></a>创建应用配置存储区

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. 选择“功能管理器” > “+ 创建”以添加以下功能标志：

    | 密钥 | 状态 |
    |---|---|
    | Beta | 关闭 |

## <a name="create-an-aspnet-core-web-app"></a>创建一个 ASP.NET Core Web 应用

使用 [.NET Core命令行接口 (CLI)](https://docs.microsoft.com/dotnet/core/tools/) 创建新的 ASP.NET Core MVC Web 应用项目。 通过 Visual Studio 使用 .NET Core CLI 的优点是，它可用于 Windows、macOS 和 Linux 平台。

1. 为项目新建一个文件夹。 本快速入门将其命名为 *TestFeatureFlags*。

2. 在新文件夹中，运行以下命令，创建新的 ASP.NET Core MVC Web 应用项目：

        dotnet new mvc

## <a name="add-secret-manager"></a>添加机密管理器

向项目添加[机密管理器工具](https://docs.microsoft.com/aspnet/core/security/app-secrets)。 机密管理器工具存储敏感数据，以便用于项目树之外的开发工作。 此方法有助于防止意外共享源代码中的应用密码。

- 打开 *.csproj* 文件。 如下所示，添加 `UserSecretsId` 元素，将其值替换为你自己的值（通常为 GUID）。 保存文件。

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>netcoreapp2.1</TargetFramework>
        <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.App" />
        <PackageReference Include="Microsoft.AspNetCore.Razor.Design" Version="2.1.2" PrivateAssets="All" />
    </ItemGroup>

    </Project>
    ```

## <a name="connect-to-an-app-configuration-store"></a>连接到应用程序配置存储区

1. 运行以下命令添加对 `Microsoft.Extensions.Configuration.AzureAppConfiguration` 和 `Microsoft.FeatureManagement` NuGet 包的引用：

        dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration --version 1.0.0-preview-008520001

        dotnet add package Microsoft.FeatureManagement.AspNetCore --version 1.0.0-preview-008560001-910

2. 运行以下命令，还原项目包：

        dotnet restore

3. 将名为 ConnectionStrings:AppConfig 的机密添加到机密管理器。

    此机密包含用于访问应用程序配置存储区的连接字符串。 将以下命令中的值替换为应用程序配置存储区的连接字符串。

    必须在 .csproj 文件所在的同一目录中执行此命令。

        dotnet user-secrets set ConnectionStrings:AppConfig <your_connection_string>

    机密管理器仅用于本地测试 web 应用程序。 例如，将应用部署到 [Azure 应用服务](https://azure.microsoft.com/services/app-service/web)后，可以使用应用服务中的“连接字符串”应用程序设置，而无需使用机密管理器来存储连接字符串。

    此机密使用配置 API 进行访问。 在所有支持的平台上，冒号 (:) 可以在配置 API 的配置名称中使用。 请参阅[按环境进行的配置](https://docs.microsoft.com/aspnet/core/fundamentals/configuration/index?tabs=basicconfiguration&view=aspnetcore-2.0)。

4. 打开 *Program.cs*，并添加对 .NET Core 应用程序配置提供程序的引用。

    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

5. 通过调用 `config.AddAzureAppConfiguration()` 方法，更新 `CreateWebHostBuilder` 方法以使用应用配置。

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options => {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .UseFeatureFlags();
                });
            })
            .UseStartup<Startup>();
    ```

6. 打开 *Startup.cs*，并添加对 .NET Core 功能管理器的引用。

    ```csharp
    using Microsoft.FeatureManagement.AspNetCore;
    ```

7. 通过调用 `services.AddFeatureManagement()` 方法更新 `ConfigureServices` 方法以添加功能标志支持，并选择性地通过调用 `services.AddFeatureFilter<FilterType>()` 来包含与功能标志配合使用的任何筛选器：

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
    ```

8. 添加 *MyFeatureFlags.cs* 文件。

    ```csharp
    namespace TestFeatureFlags
    {
        public enum MyFeatureFlags
        {
            Beta
        }
    }
    ```

9. 将 *BetaController.cs* 添加到 Controllers 目录：

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.FeatureManagement.AspNetCore;

    namespace TestFeatureFlags.Controllers
    {
        public class BetaController: Controller
        {
            private readonly IFeatureManager _featureManager;

            public BetaController(IFeatureManagerSnapshot featureManager)
            {
                _featureManager = featureManager;
            }

            [Feature(MyFeatureFlags.Beta)]
            public IActionResult Index()
            {
                return View();
            }
        }
    }
    ```

10. 打开 Views 目录中的 *_ViewImports.cshtml*，并添加功能管理器标记帮助器：

    ```html
    @addTagHelper *, Microsoft.FeatureManagement.AspNetCore
    ```

11. 打开 Views > Shared 目录中的 *_Layout.cshtml*，并将 `<body>` > `<header>` 下的 `<nav>` 栏替换为以下代码：

    ```html
    <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
        <div class="container">
            <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">TestFeatureFlags</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
            aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
            </button>
            <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                <ul class="navbar-nav flex-grow-1">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                    </li>
                    <feature name="Beta">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Beta" asp-action="Index">Beta</a>
                    </li>
                    </feature>
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    ```

12. 在 Views 下创建一个 Beta 目录，并在其中添加 *Index.cshtml*：

    ```html
    @{
        ViewData["Title"] = "Beta Home Page";
    }

    <h1>
        This is the beta website.
    </h1>
    ```

## <a name="build-and-run-the-app-locally"></a>在本地生成并运行应用

1. 要通过使用 .NET Core CLI 生成应用，请在命令行界面中执行以下命令：

        dotnet build

2. 生成成功完成后，请运行以下命令以在本地运行 Web 应用：

        dotnet run

3. 启动浏览器窗口并转到 `https://localhost:5001`，即本地托管的 Web 应用的默认 URL。

    ![本地启动应用快速入门](./media/quickstarts/aspnet-core-feature-flag-local-before.png)

4. 登录到 [Azure 门户](https://aka.ms/azconfig/portal)。 选择“所有资源”，然后选择在快速入门中创建的应用程序配置存储区实例。

5. 选择“功能管理器”，将“Beta”值更改为“On”：

    | 密钥 | 状态 |
    |---|---|
    | Beta | 启用 |

6. 刷新浏览器页面，查看新的配置设置。

    ![本地启动应用快速入门](./media/quickstarts/aspnet-core-feature-flag-local-after.png)

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已创建一个新的应用配置存储区，并已使用它来通过[功能管理库](https://go.microsoft.com/fwlink/?linkid=2074664)管理 ASP.NET Core Web 应用中的功能。

* 详细了解[功能管理](./concept-feature-management.md)
* [管理功能标志](./manage-feature-flags.md)
* [在 ASP.NET Core 应用中使用功能标志](./use-feature-flags-dotnet-core.md)
