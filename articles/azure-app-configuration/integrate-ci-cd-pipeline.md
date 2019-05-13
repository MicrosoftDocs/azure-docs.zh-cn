---
title: 使用 Azure 应用程序配置与持续集成和交付管道集成的教程 | Microsoft Docs
description: 本教程介绍了如何在持续集成和交付期间使用 Azure 应用程序配置中的数据生成配置文件
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 02/24/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: 26bd49af7245d6e6dde3162a2e1d95c54f13e35b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415941"
---
# <a name="integrate-with-a-cicd-pipeline"></a>与 CI/CD 管道集成

本文介绍如何通过不同的方式在持续集成和持续部署系统中使用来自 Azure 应用程序配置的数据。

## <a name="use-app-configuration-in-your-azure-devops-pipeline"></a>在 Azure DevOps 管道中使用应用程序配置

如果你有一个 Azure DevOps 管道，则可以从应用程序配置提取键值，并将其设置为任务变量。 [Azure 应用程序配置 DevOps 扩展](https://go.microsoft.com/fwlink/?linkid=2091063)是可以提供此功能的加载项模块。 只需遵照此扩展中的说明在生成或发布任务序列中使用它。

## <a name="deploy-app-configuration-data-with-your-application"></a>使用应用程序部署应用程序配置数据

如果应用程序依赖于 Azure 应用程序配置且无法访问它，则应用程序可能无法运行。 可以增强应用程序的复原能力来应对此类事件，不过，这种情况不太可能发生。 为此，请将当前配置数据打包到与应用程序一起部署的文件中，并在其启动期间在本地加载。 这种方法可保证应用程序至少有默认设置值。 当应用程序配置存储区可用时，这些值将被应用程序配置存储区中的任何新更改覆盖。

使用 Azure 应用程序配置的[导出](./howto-import-export-data.md#export-data)功能，可以将当前配置数据的检索过程自动化为单个文件。 然后将此文件嵌入到持续集成和持续部署 (CI/CD) 管道的生成或部署步骤中。

下面的示例演示如何将应用程序配置数据包含在快速入门中引入的 Web 应用程序的生成步骤中。 在继续操作之前，请先完成[使用应用程序配置创建 ASP.NET Core 应用](./quickstart-aspnet-core-app.md)。

你可以使用任何代码编辑器执行本教程中的步骤。 [Visual Studio Code](https://code.visualstudio.com/) 是 Windows、macOS 和 Linux 平台上提供的一个卓越选项。

### <a name="prerequisites"></a>先决条件

如果在本地生成，请下载并安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)（如果尚未安装）。

要进行云生成，例如使用 Azure DevOps，请确保在生成系统中安装了 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。

### <a name="export-an-app-configuration-store"></a>导出应用程序配置存储区

1. 打开 .csproj 文件，然后添加以下脚本：

    ```xml
    <Target Name="Export file" AfterTargets="Build">
        <Message Text="Export the configurations to a temp file. " />
        <Exec WorkingDirectory="$(MSBuildProjectDirectory)" Condition="$(ConnectionString) != ''" Command="az appconfig kv export -f $(OutDir)\azureappconfig.json --format json --separator : --connection-string $(ConnectionString)" />
    </Target>
    ```

    添加与应用程序配置存储区相关联的 ConnectionString 作为环境变量。

2. 打开 *Program.cs* 并更新 `CreateWebHostBuilder` 方法以通过调用 `config.AddJsonFile()` 方法使用导出的 JSON 文件。

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var directory = System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
                var settings = config.Build();

                config.AddJsonFile(Path.Combine(directory, "azureappconfig.json"));
                config.AddAzureAppConfiguration(settings["ConnectionStrings:AppConfig"]);
            })
            .UseStartup<Startup>();
    ```

### <a name="build-and-run-the-app-locally"></a>在本地生成并运行应用

1. 设置名为“ConnectionString”的环境变量，并将其设置为应用程序配置存储区的访问键。 如果使用 Windows 命令提示符，则请运行以下命令并重启命令提示符，这样更改才会生效：

        setx ConnectionString "connection-string-of-your-app-configuration-store"

    如果使用 Windows PowerShell，请运行以下命令：

        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"

    如果使用 macOS 或 Linux，则请运行以下命令：

        export ConnectionString='connection-string-of-your-app-configuration-store'

2. 要通过使用 .NET Core CLI 生成应用，请在命令行界面中执行以下命令：

        dotnet build

3. 生成成功完成后，请运行以下命令以在本地运行 Web 应用：

        dotnet run

4. 启动浏览器窗口并转到 `http://localhost:5000`，即本地托管的 Web 应用的默认 URL。

    ![本地启动应用快速入门](./media/quickstarts/aspnet-core-app-launch-local.png)

## <a name="next-steps"></a>后续步骤

在本教程中，导出了要在部署管道中使用的 Azure 应用程序配置数据。 若要了解有关如何使用应用程序配置的更多信息，请继续阅读 Azure CLI 示例。

> [!div class="nextstepaction"]
> [托管标识集成](./howto-integrate-azure-managed-service-identity.md)
