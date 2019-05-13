---
title: Microsoft 标识平台 Windows 桌面快速入门 | Microsoft Docs
description: 了解 Windows 桌面 .NET (XAML) 应用程序如何获取访问令牌并调用受 Microsoft 标识平台终结点保护的 API
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/11/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2d263014638329c454149af8ca922c1c5c6393a5
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413837"
---
# <a name="quickstart-acquire-a-token-and-call-microsoft-graph-api-from-a-windows-desktop-app"></a>快速入门：获取令牌并从 Windows 桌面应用中调用 Microsoft Graph API

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

本快速入门介绍如何编写 Windows 桌面.NET (WPF) 应用程序，该应用程序能够登录个人、工作和学校帐户，获取访问令牌以及调用 Microsoft Graph API。

![显示本快速入门生成的示例应用的工作原理](media/quickstart-v2-windows-desktop/windesktop-intro.svg)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>注册并下载快速入门应用
> 可以使用两个选项来启动快速入门应用程序：
> * [快速][选项 1：注册并自动配置应用，然后下载代码示例](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [手动][选项 2：注册并手动配置应用程序和代码示例](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>选项 1：注册并自动配置应用，然后下载代码示例
>
> 1. 转到新的 [Azure 门户 - 应用注册](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/WinDesktopQuickstartPage/sourceType/docs)。
> 1. 输入应用程序的名称并选择“注册”。
> 1. 遵照说明下载内容，并只需单击一下自动配置新应用程序。
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>选项 2：注册并手动配置应用程序和代码示例
>
> #### <a name="step-1-register-your-application"></a>步骤 1：注册应用程序
> 若要手动注册应用程序并将应用的注册信息添加到解决方案，请执行以下步骤：
>
> 1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
> 1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
> 1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://aka.ms/MobileAppReg)页。
> 1. 选择“新注册”。
>      - 在“名称”部分输入一个会显示给应用用户的有意义的应用程序名称，例如 `Win-App-calling-MsGraph`。
>      - 在“支持的帐户类型”部分，选择“任何组织目录中的帐户和个人 Microsoft 帐户(例如 Skype、Xbox、Outlook.com)”。
>      - 选择“注册”以创建应用程序。
> 1. 在应用的页面列表中，选择“身份验证”。
> 1. 展开“桌面 + 设备”部分。  （如果“桌面 + 设备”不可见，首先请单击顶部的横幅，以便查看预览版身份验证体验）
> 1. 在“重定向 URI”部分下选择“添加 URI”。  键入 **urn:ietf:wg:oauth:2.0:oob**。
> 1. 选择“保存”。

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-azure-portal"></a>步骤 1：在 Azure 门户中配置应用程序
> 要使此快速入门中的代码示例正常运行，需要将答复 URL 添加为 urn:ietf:wg:oauth:2.0:oob。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [执行此更改]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已配置](media/quickstart-v2-windows-desktop/green-check.png) 应用程序已使用这些属性进行配置。

#### <a name="step-2-download-your-visual-studio-project"></a>步骤 2：下载 Visual Studio 项目

[下载 Visual Studio 项目](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2/archive/msal3x.zip)

#### <a name="step-3-configure-your-visual-studio-project"></a>步骤 3：配置 Visual Studio 项目

1. 将 zip 文件提取到靠近磁盘根目录的本地文件夹，例如 **C:\Azure-Samples**。
1. 在 Visual Studio 中打开项目。
1. 编辑 **App.Xaml.cs** 并将字段 `ClientId` 和 `Tenant` 的值替换为以下代码：

    ```csharp
    private static string ClientId = "Enter_the_Application_Id_here";
    private static string Tenant = "Enter_the_Tenant_Info_Here";
    ```

> [!div renderon="docs"]
> 其中：
> - `Enter_the_Application_Id_here` - 是已注册应用程序的**应用程序（客户端）ID**。
> - `Enter_the_Tenant_Info_Here` - 设置为以下选项之一：
>   - 如果应用程序支持“此组织目录中的帐户”，请将该值替换为**租户 ID** 或**租户名称**（例如 contoso.microsoft.com）
>   - 如果应用程序支持“任何组织目录中的帐户”，请将该值替换为 `organizations`
>   - 如果应用程序支持“任何组织目录中的帐户和个人 Microsoft 帐户”，请将该值替换为 `common`
>
> > [!TIP]
> > 若要查找“应用程序(客户端) ID”、“目录(租户) ID”和“支持的帐户类型”的值，请转到 Azure 门户中应用的“概述”页。

## <a name="more-information"></a>详细信息

### <a name="msalnet"></a>MSAL.NET

MSAL ([Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)) 是一个库，用于用户登录和请求令牌，此类令牌用于访问受 Microsoft 标识平台保护的 API。 可在 Visual Studio 的包管理器控制台中运行以下命令，以便安装 MSAL：

```powershell
Install-Package Microsoft.Identity.Client -IncludePrerelease
```

### <a name="msal-initialization"></a>MSAL 初始化

可以通过添加以下代码，为 MSAL 添加引用：

```csharp
using Microsoft.Identity.Client;
```

然后，使用以下代码对 MSAL 进行初始化：

```csharp
public static IPublicClientApplication PublicClientApp;
PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AzureCloudInstance.AzurePublic, Tenant)
                .Build();
```

> |其中： ||
> |---------|---------|
> | `ClientId` | 是在 Azure 门户中注册的应用程序的**应用程序(客户端) ID**。 可以在 Azure 门户的应用的“概览”页中找到此值。 |

### <a name="requesting-tokens"></a>请求令牌

MSAL 有两种获取令牌的方法：`AcquireTokenInteractive` 和 `AcquireTokenSilent`。

#### <a name="get-a-user-token-interactively"></a>以交互方式获取用户令牌

某些情况下需要强制用户通过弹出窗口与 Microsoft 标识平台终结点进行交互，以验证其凭据或进行许可。 示例包括：

- 用户首次登录应用程序
- 由于密码已过期，用户可能需要重新输入凭据的情况
- 应用程序正在请求访问用户需要同意的资源的情况
- 需要双重身份验证的情况

```csharp
authResult = await App.PublicClientApp.AcquireTokenInteractive(_scopes)
                                      .ExecuteAsync();
```

> |其中：||
> |---------|---------|
> | `_scopes` | 包含所请求的作用域，例如针对 Microsoft Graph 的 `{ "user.read" }` 或针对自定义 Web API 的 `{ "api://<Application ID>/access_as_user" }`。 |

#### <a name="get-a-user-token-silently"></a>以无提示方式获取用户令牌

你不希望在用户每次需要访问资源时都要求其验证其凭据。 大多数情况下，你希望在无需任何用户交互的情况下进行令牌获取和续订。 可以使用 `AcquireTokenSilentAsync` 方法获取令牌，以在初始 `AcquireTokenInteractive` 方法后访问受保护资源：

```csharp
var accounts = await App.PublicClientApp.GetAccountsAsync();
var firstAccount = accounts.FirstOrDefault();
authResult = await App.PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                      .ExecuteAsync();
```

> |其中： ||
> |---------|---------|
> | `scopes` | 包含所请求的作用域，例如针对 Microsoft Graph 的 `{ "user.read" }` 或针对自定义 Web API 的 `{ "api://<Application ID>/access_as_user" }`。 |
> | `firstAccount` | 指定缓存中的第一个用户（MSAL 支持单个应用中的多个用户）。 |

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>后续步骤

试用 Windows 桌面教程，了解有关构建应用程序和新功能的完整分布指南，包括本快速入门的完整说明。

> [!div class="nextstepaction"]
> [调用 Graph API 教程](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-windesktop)

