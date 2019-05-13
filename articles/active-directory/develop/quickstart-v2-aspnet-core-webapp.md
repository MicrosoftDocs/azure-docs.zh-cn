---
title: Microsoft 标识平台 ASP.NET Core Web 应用快速入门 | Azure
description: 了解如何使用 OpenID Connect 在 ASP.NET Core Web 应用上实现 Microsoft 登录
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
ms.openlocfilehash: e51437a99217316ead50d4075be52f089225e618
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190871"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-an-aspnet-core-web-app"></a>快速入门：向 ASP.NET Core Web 应用添加 Microsoft 登录功能

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

本快速入门介绍如何使用 ASP.NET Core Web 应用从任何 Azure Active Directory (Azure AD) 实例中登录个人帐户（hotmail.com、outlook.com 等）以及学校和工作帐户。

![显示本快速入门生成的示例应用的工作原理](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.svg)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>注册并下载快速入门应用
> 可以使用两个选项来启动快速入门应用程序：
> * [快速][选项 1：注册并自动配置应用，然后下载代码示例](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [手动][选项 2：注册并手动配置应用程序和代码示例](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>选项 1：注册并自动配置应用，然后下载代码示例
>
> 1. 访问 [Azure 门户 - 应用注册](https://aka.ms/aspnetcore2-1-aad-quickstart-v2)。
> 1. 输入应用程序的名称并选择“注册”。
> 1. 遵照说明下载内容，并一键式自动配置新应用程序。
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>选项 2：注册并手动配置应用程序和代码示例
>
> #### <a name="step-1-register-your-application"></a>步骤 1：注册应用程序
> 若要注册应用程序并将应用的注册信息手动添加到解决方案，请执行以下步骤：
>
> 1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
> 1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
> 1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)页。
> 1. 选择“新注册”。
> 1. 出现“注册应用程序”页后，请输入应用程序的注册信息：
>    - 在“名称”部分输入一个会显示给应用用户的有意义的应用程序名称，例如 `AspNetCore-Quickstart`。
>    - 在“重定向 URI”中添加 `https://localhost:44321/`，然后选择“注册”。
> 1. 选择“身份验证”菜单，然后添加以下信息：
>    - 在“重定向 URI”中添加 `https://localhost:44321/signin-oidc`，然后选择“保存”。
>    - 在“高级设置”部分，将“注销 URL”设置为 `https://localhost:44321/signout-oidc`。
>    - 在“隐式授权”下，勾选“ID 令牌”。
>    - 选择“保存”。

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>步骤 1：在 Azure 门户中配置应用程序
> 为使此快速入门中的代码示例正常运行，需将回复 URL 添加为 `https://localhost:44321/` 和 `https://localhost:44321/signin-oidc`，将注销 URL 添加为 `https://localhost:44321/signout-oidc`，并请求将由授权终结点颁发的 ID 令牌。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [执行此更改]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已配置](media/quickstart-v2-aspnet-webapp/green-check.png) 应用程序已使用这些属性进行配置。

#### <a name="step-2-download-your-aspnet-core-project"></a>步骤 2：下载 ASP.NET Core 项目

- [下载 Visual Studio 2019 解决方案](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore2-2.zip)

#### <a name="step-3-configure-your-visual-studio-project"></a>步骤 3：配置 Visual Studio 项目

1. 将 zip 文件提取到根文件夹中的本地文件夹（例如，**C:\Azure-Samples**）
1. 如果使用 Visual Studio 2019，请在 Visual Studio 中打开该解决方案（可选）。
1. 编辑 **appsettings.json** 文件。 查找 `ClientId` 并使用你注册的应用程序的“应用程序 (客户端) ID”值更新 `ClientId` 的值。 

    ```json
    "ClientId": "Enter_the_Application_Id_here"
    "TenantId": "Enter_the_Tenant_Info_Here"
    ```

> [!div renderon="docs"]
> 其中：
> - `Enter_the_Application_Id_here` - 在 Azure 门户中注册的应用程序的**应用程序(客户端) ID**。 可以在应用的“概览”页中找到“应用程序(客户端) ID”。
> - `Enter_the_Tenant_Info_Here` - 以下选项之一：
>   - 如果应用程序支持“仅限此组织目录中的帐户”，请将该值替换为**租户 ID** 或**租户名称**（例如 contoso.microsoft.com）
>   - 如果应用程序支持“任何组织目录中的帐户”，请将该值替换为 `organizations`
>   - 如果应用程序支持“所有 Microsoft 帐户用户”，请将该值替换为 `common`
>
> > [!TIP]
> > 若要查找“应用程序(客户端) ID”、“目录(租户) ID”和“支持的帐户类型”的值，请转到 Azure 门户中应用的“概述”页。

## <a name="more-information"></a>详细信息

本部分概述了使用户登录所需的代码。 阅读本概述对于了解代码的工作原理、主要参数非常有用，如果你想要将登录功能添加到现有 ASP.NET Core 应用程序，阅读本概述也非常有用。

### <a name="startup-class"></a>Startup 类

Microsoft.AspNetCore.Authentication 中间件使用主机进程初始化时执行的 Startup 类：

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services.Configure<CookiePolicyOptions>(options =>
  {
    // This lambda determines whether user consent for non-essential cookies is needed for a given request.
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = SameSiteMode.None;
  });

  services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
          .AddAzureAD(options => Configuration.Bind("AzureAd", options));

  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
  {
    options.Authority = options.Authority + "/v2.0/";         // Microsoft identity platform

    options.TokenValidationParameters.ValidateIssuer = false; // accept several tenants (here simplified)
  });

  services.AddMvc(options =>
  {
     var policy = new AuthorizationPolicyBuilder()
                     .RequireAuthenticatedUser()
                     .Build();
     options.Filters.Add(new AuthorizeFilter(policy));
  })
  .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```

方法 `AddAuthentication` 配置该服务以添加基于 Cookie 的身份验证，此身份验证在浏览器方案中使用，并用于设置 OpenID Connect 质询。 

包含 `.AddAzureAd` 的行可向应用程序添加 Microsoft 标识平台身份验证。 然后，它会被配置为使用 Microsoft 标识平台终结点登录。

> |其中  |  |
> |---------|---------|
> | ClientId  | Azure 门户中注册的应用程序的应用程序（客户端）ID。 |
> | 颁发机构 | 用户要进行身份验证的 STS 终结点。 对于公有云，此项通常为 <https://login.microsoftonline.com/{tenant}/v2.0>，其中 {tenant} 是租户名称、租户 ID 或者引用常用终结点（用于多租户应用程序）的 common |
> | TokenValidationParameters | 用于令牌验证的参数列表。 在这种情况下，`ValidateIssuer` 设置为 `false`，以指示它可以接受来自任何个人或工作或学校帐户的登录。 |


> [!NOTE]
> 对本快速入门来说，设置 `ValidateIssuer = false` 是一种简化。 在实际应用程序中，需验证颁发者。
> 查看示例，了解如何执行该操作。

### <a name="protect-a-controller-or-a-controllers-method"></a>保护控制器或控制器的方法

可以使用 `[Authorize]` 属性保护控制器或控制器方法。 此属性通过仅允许经过身份验证的用户，来限制对控制器或方法的访问，这意味着如果用户未经身份验证，则可以启动身份验证质询来访问控制器。

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>后续步骤

请参阅此 ASP.NET Core 教程的 GitHub 存储库，了解有关详细信息，包括各种说明，即，如何向全新 ASP.NET Core Web 应用程序添加身份验证、如何调用 Microsoft Graph 和其他 Microsoft API、如何调用你自己的 API、如何添加授权、如何通过国家/地区云或社交标识等方式使用户登录：

> [!div class="nextstepaction"]
> [ASP.NET Core Web 应用教程](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
