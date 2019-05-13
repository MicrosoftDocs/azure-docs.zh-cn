---
title: include 文件
description: include 文件
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 04/10/2019
ms.date: 05/10/2019
ms.author: v-junlch
ms.custom: include file
ms.openlocfilehash: ae6d590cdada24638ec2d24c83609b8e6addfaf0
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416259"
---
## <a name="set-up-your-project"></a>设置项目

本部分将创建新项目，用于演示如何将 Windows 桌面 .NET 应用程序 (XAML) 与“使用 Microsoft 登录”集成，使该应用程序能查询需要令牌的 Web API。

使用本指南创建的应用程序将显示一个用于调用图的按钮、一个用于在屏幕上显示结果的区域和一个注销按钮。

> [!NOTE]
> 想要改为下载此示例的 Visual Studio 项目？ [下载一个项目](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2/archive/msal3x.zip)，并且在执行该项目之前跳到[配置步骤](#register-your-application)来配置代码示例。
>

若要创建应用程序，请执行以下操作：

1. 在 Visual Studio 中，选择“文件” > “新建” > “项目”。
2. 在“模板”下，选择“Visual C#”。
3. 选择“WPF 应用(.NET Framework)”，具体取决于所使用的 Visual Studio 版本。

## <a name="add-msal-to-your-project"></a>将 MSAL 添加到项目

1. 在 Visual Studio 中，选择“工具” > “NuGet 包管理器”> “包管理器控制台”。
2. 在“包管理器控制台”窗口中，粘贴以下 Azure PowerShell 命令：

    ```powershell
    Install-Package Microsoft.Identity.Client -Pre
    ```

    > [!NOTE] 
    > 此命令将安装 Microsoft 身份验证库。 MSAL 负责获取、缓存和刷新用于访问受 Azure Active Directory v2.0 保护的 API 的用户令牌
    >

## <a name="add-the-code-to-initialize-msal"></a>添加代码以初始化 MSAL

此步骤将创建用于处理与 MSAL 的交互（例如令牌处理）的类。

1. 打开 *App.xaml.cs* 文件，将 MSAL 的引用添加到类：

    ```csharp
    using Microsoft.Identity.Client;
    ```
   <!-- Workaround for Docs conversion bug -->

2. 将 App 类更新为：

    ```csharp
    public partial class App : Application
    {
        static App()
        {
            _clientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AzureCloudInstance.AzureChina, Tenant)
                .Build();
        }

        // Below are the clientId (Application Id) of your app registration and the tenant information. 
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        // - Te content of Tenant by the information about the accounts allowed to sign-in in your application:
        //   - For Work or School account in your org, use your tenant ID, or domain
        //   - for any Work or School accounts, use `organizations`
        //   - for any Work or School accounts, or Microsoft personal account, use `common`
        //   - for Microsoft Personal account, use consumers
        private static string ClientId = "0b8b0665-bc13-4fdc-bd72-e0227b9fc011";

        private static string Tenant = "common";

        private static IPublicClientApplication _clientApp ;

        public static IPublicClientApplication PublicClientApp { get { return _clientApp; } }
    }
    ```

## <a name="create-the-application-ui"></a>创建应用程序 UI

本部分说明应用程序如何查询受保护的后端服务器，例如 Microsoft Graph。 

项目模板中应自动创建了 *MainWindow.xaml* 文件。 打开此文件，然后将应用程序的 *\<Grid>* 节点替换为以下代码：

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <Label Content="API Call Results" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <Label Content="Token Info" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

