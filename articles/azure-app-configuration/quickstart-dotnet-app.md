---
title: 将 Azure 应用配置与 .NET Framework 结合使用的快速入门| Microsoft Docs
description: 将 Azure 应用配置与 .NET Framework 应用结合使用的快速入门
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: quickstart
ms.tgt_pltfrm: .NET
ms.workload: tbd
ms.date: 02/24/2019
ms.author: yegu
ms.openlocfilehash: acf1608257bc5416ff52ed9fbd4209008712f2d7
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408713"
---
# <a name="quickstart-create-a-net-framework-app-with-azure-app-configuration"></a>快速入门：使用 Azure 应用配置创建 .NET Framework 应用

Azure 应用配置是 Azure 中的托管配置服务。 借助它，无需代码即可在一个位置轻松存储和管理所有应用程序设置。 本快速入门将介绍如何将服务合并到基于 .NET Framework 的Windows 桌面控制台应用。

![在本地完成该快速入门](./media/quickstarts/dotnet-fx-app-run.png)

## <a name="prerequisites"></a>先决条件

要完成此快速入门，请安装 [Visual Studio 2017](https://visualstudio.microsoft.com/vs) 和 [.NET Framework 4.7.1](https://dotnet.microsoft.com/download) 或更高版本（如果尚未安装）。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-app-configuration-store"></a>创建应用配置存储区

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. 选择“配置资源管理器” > “+创建”来添加以下键值对：

    | 密钥 | 值 |
    |---|---|
    | TestApp:Settings:Message | Azure 应用配置的数据 |

    暂时将“标签”和“内容类型”保留为空。

## <a name="create-a-net-console-app"></a>创建 .NET 控制台应用

1. 启动 Visual Studio，并选择“文件” > “新建” > “项目”。

2. 在“新建项目”中，选择“已安装” > “Visual C#” > “Windows 桌面版”。 选择“控制台应用 (.NET Framework)”，并输入项目的名称。 选择“.NET Framework 4.7.1”或更高版本，并选择“确定”。

## <a name="connect-to-an-app-configuration-store"></a>连接到应用程序配置存储区

1. 右键单击项目，然后选择“管理 NuGet 包”。 在“浏览”选项卡中，搜索以下 NuGet 包并将其添加到项目中。 如果无法找到，请选中“包括预发行版”复选框。

    ```
    Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration 1.0.0 preview or later
    Microsoft.Configuration.ConfigurationBuilders.Environment 2.0.0 preview or later
    ```

2. 如下所述，更新项目的 App.config 文件：

    ```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>

    <configBuilders>
        <builders>
            <add name="MyConfigStore" mode="Greedy" connectionString="${ConnectionString}" type="Microsoft.Configuration.ConfigurationBuilders.AzureAppConfigurationBuilder, Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration" />
            <add name="Environment" mode="Greedy" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
        </builders>
    </configBuilders>

    <appSettings configBuilders="Environment,MyConfigStore">
        <add key="AppName" value="Console App Demo" />
        <add key="ConnectionString" value ="Set via an environment variable - for example, dev, test, staging, or production connection string." />
    </appSettings>
    ```

   从环境变量 `ConnectionString` 中读取应用程序配置存储区的连接字符串。 在 `appSettings` 部分的 `configBuilders` 属性中，在 `MyConfigStore` 之前添加 `Environment` 配置生成器。

3. 打开 Program.cs 并更新 `Main` 方法，以通过调用 `ConfigurationManager` 来使用应用程序配置。

    ```csharp
    static void Main(string[] args)
    {
        string message = ConfigurationManager.AppSettings["TestApp:Settings:Message"];

        Console.WriteLine(message);
    }
    ```

## <a name="build-and-run-the-app-locally"></a>在本地生成并运行应用

1. 将名为 ConnectionString 的环境变量设置为应用配置存储区的连接字符串。 如果使用 Windows 命令提示符，请运行以下命令：

        setx ConnectionString "connection-string-of-your-app-configuration-store"

    如果使用 Windows PowerShell，请运行以下命令：

        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"

2. 重启 Visual Studio 以便使所做更改生效。 按 Ctrl+F5 生成并运行控制台应用。

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已经创建了一个新的应用程序配置存储区，并将其用于 .NET Framework 控制台应用。 若要深入了解如何使用应用程序配置，请继续学习下一个教程，其中将介绍如何进行身份验证。

> [!div class="nextstepaction"]
> [托管标识集成](./howto-integrate-azure-managed-service-identity.md)
