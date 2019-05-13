---
title: Microsoft 标识平台 Windows UWP 快速入门 | Azure
description: 了解通用 Windows 平台 (XAML) 应用程序如何获取访问令牌并调用受 Microsoft 标识平台终结点保护的 API。
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
ms.date: 04/12/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: c45cd1627eb1eb98b2fc19f6663d5635b001ce0c
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067790"
---
# <a name="quickstart-call-the-microsoft-graph-api-from-a-universal-windows-platform-uwp-application"></a>快速入门：从通用 Windows 平台 (UWP) 应用程序调用 Microsoft Graph API

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

本快速入门包含了一个代码示例，该示例演示了通用 Windows 平台 (UWP) 应用程序如何让用户使用个人帐户或工作和学校帐户进行登录，如何获取访问令牌以及如何调用 Microsoft Graph API。

![显示本快速入门生成的示例应用的工作原理](media/quickstart-v2-uwp/uwp-intro.svg)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>注册并下载快速入门应用
> [!div renderon="docs" class="sxs-lookup"]
> 可以使用两个选项来启动快速入门应用程序：
> * [快速][选项 1：注册并自动配置应用，然后下载代码示例](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [手动][选项 2：注册并手动配置应用程序和代码示例](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>选项 1：注册并自动配置应用，然后下载代码示例
>
> 1. 转到新的 [Azure 门户 - 应用注册](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/UwpQuickstartPage/sourceType/docs)窗格。
> 1. 输入应用程序的名称，然后单击“注册”。
> 1. 遵照说明下载内容，并一键式自动配置新应用程序。
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>选项 2：注册并手动配置应用程序和代码示例
> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>步骤 1：注册应用程序
> 若要注册应用程序并将应用的注册信息添加到解决方案，请执行以下步骤：
> 1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
> 1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
> 1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://aka.ms/MobileAppReg)页。
> 1. 选择“新注册”。
> 1. 出现“注册应用程序”页后，请输入应用程序的注册信息：
>      - 在“名称”部分输入一个会显示给应用用户的有意义的应用程序名称，例如 `UWP-App-calling-MsGraph`。
>      - 在“支持的帐户类型”部分，选择“任何组织目录中的帐户和个人 Microsoft 帐户(例如 Skype、Xbox、Outlook.com)”。
>      - 选择“注册”以创建应用程序。
> 1. 在应用的页面列表中，选择“身份验证”。
> 1. 展开“桌面 + 设备”部分。  （如果“桌面 + 设备”不可见，首先请单击顶部的横幅，以便查看预览版身份验证体验）
> 1. 在“重定向 URI”部分下选择“添加 URI”。  键入 **urn:ietf:wg:oauth:2.0:oob**。
> 1. 选择“保存”。

> [!div renderon="portal" class="sxs-lookup alert alert-info"]
> #### <a name="step-1-configure-your-application"></a>步骤 1：配置应用程序
> 为使此快速入门中的代码示例正常运行，需要将重定向 URI 添加为 **urn:ietf:wg:oauth:2.0:oob**。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [执行此更改]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已配置](media/quickstart-v2-uwp/green-check.png) 应用程序已使用这些属性进行配置。

#### <a name="step-2-download-your-visual-studio-project"></a>步骤 2：下载 Visual Studio 项目

 - [下载 Visual Studio 项目](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip)

#### <a name="step-3-configure-your-visual-studio-project"></a>步骤 3：配置 Visual Studio 项目

1. 将 zip 文件提取到靠近磁盘根目录的本地文件夹，例如 **C:\Azure-Samples**。
1. 在 Visual Studio 中打开项目。 系统可能会提示你安装 UWP SDK。 在这种情况下，请接受。
1. 编辑 **MainPage.Xaml.cs**，替换 `ClientId` 字段的值：

    ```csharp
    private const string ClientId = "Enter_the_Application_Id_here";
    ```

> [!div renderon="docs"]
> 其中：
> - `Enter_the_Application_Id_here` - 是已注册应用程序的应用程序 ID。
>
> > [!TIP]
> > 若要查找“应用程序 ID”的值，请转到门户中的“概览”部分

#### <a name="step-4-run-your-application"></a>步骤 4：运行应用程序

若要在 Windows 计算机上尝试快速入门，请执行以下操作：

1. 在 Visual Studio 工具栏中，选择适当的平台（可能为 **x64** 或 **x86**，不是 ARM）。
   > 可以看到目标设备从“设备”更改为“本地计算机”。
1. 选择“调试”|“在不调试的情况下启动”

## <a name="more-information"></a>详细信息

此部分提供快速入门的详细信息。

### <a name="msalnet"></a>MSAL.NET

MSAL ([Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)) 是一个库，用于用户登录和请求安全令牌。 安全令牌用于访问受面向开发人员的 Microsoft 标识平台保护的 API。 可在 Visual Studio 的包管理器控制台中运行以下命令，以便安装 MSAL：

```powershell
Install-Package Microsoft.Identity.Client -IncludePrerelease
```

### <a name="msal-initialization"></a>MSAL 初始化

可以通过添加以下代码，为 MSAL 添加引用：

```csharp
using Microsoft.Identity.Client;
```

然后，系统将使用以下代码对 MSAL 进行初始化：

```csharp
public static IPublicClientApplication PublicClientApp;
PublicClientApp = new PublicClientApplicationBuilder.Create(ClientId)
                                                    .Build();
```

> |其中： ||
> |---------|---------|
> | `ClientId` | 是在 Azure 门户中注册的应用程序的**应用程序(客户端) ID**。 可以在 Azure 门户的应用的“概览”页中找到此值。 |

### <a name="requesting-tokens"></a>请求令牌

MSAL 有两种在 UWP 应用中获取令牌的方法：`AcquireTokenInteractive` 和 `AcquireTokenSilent`。

#### <a name="get-a-user-token-interactively"></a>以交互方式获取用户令牌

在某些情况下需要强制用户通过弹出窗口与 Microsoft 标识平台终结点进行交互，以验证其凭据或进行许可。 示例包括：

- 用户首次登录应用程序
- 由于密码已过期，用户可能需要重新输入凭据的情况
- 应用程序正在请求访问需要用户同意的资源时
- 需要双重身份验证的情况

```csharp
authResult = await App.PublicClientApp.AcquireTokenInteractive(scopes)
                      .ExecuteAsync();
```

> |其中：||
> |---------|---------|
> | `scopes` | 包含所请求的作用域，例如针对 Microsoft Graph 的 `{ "user.read" }` 或针对自定义 Web API 的 `{ "api://<Application ID>/access_as_user" }`。 |

#### <a name="get-a-user-token-silently"></a>以无提示方式获取用户令牌

使用 `AcquireTokenSilent` 方法可获取令牌，以在初始 `AcquireTokenAsync` 方法后访问受保护资源。 你不希望在用户每次需要访问资源时都要求其验证其凭据。 大多数时候，你希望在无需任何用户交互的情况下进行令牌获取和续订

```csharp
var accounts = await App.PublicClientApp.GetAccountsAsync();
var firstAccount = accounts.FirstOrDefault();
authResult = await App.PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                      .ExecuteAsync();
```

> |其中： ||
> |---------|---------|
> | `scopes` | 包含所请求的作用域，例如针对 Microsoft Graph 的 `{ "user.read" }` 或针对自定义 Web API 的 `{ "api://<Application ID>/access_as_user" }` |
> | `firstAccount` | 指定缓存中的第一个用户帐户（MSAL 支持在单个应用中使用多个用户） |

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>后续步骤

试用 Windows 桌面教程，了解有关构建应用程序和新功能的完整分布指南，包括本快速入门的完整说明。

> [!div class="nextstepaction"]
> [UWP - 调用 Graph API 教程](tutorial-v2-windows-uwp.md)
