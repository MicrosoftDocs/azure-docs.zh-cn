---
title: 登录用户 （登录）-Microsoft 标识平台的 web 应用
description: 了解如何构建 web 应用登录用户 （登录）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3fb7fbba7ec48da580d2a630ae51aa20b3307848
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074615"
---
# <a name="web-app-that-signs-in-users---sign-in"></a>Web 应用登录用户-登录

了解如何将登录添加到 web 应用程序的代码登录用户。

## <a name="sign-in"></a>登录

我们已在前一篇文章中的代码[应用程序的代码配置](scenario-web-app-sign-user-app-configuration.md)是需要实现注销所有。后用户已登录到你的应用，你可能想要使其能够注销。ASP.NET core 为你处理注销。

## <a name="what-sign-out-involves"></a>什么注销涉及

从 web 应用注销是多个 web 应用程序的状态从删除有关登录的帐户的信息。
Web 应用必须还将用户重定向到 Microsoft 标识平台 v2.0`logout`终结点才能注销。当你的 web 应用将用户重定向到`logout`终结点，此终结点将清除从浏览器的用户的会话。 如果您的应用程序未转到`logout`终结点，用户将重新进行身份验证到您的应用程序而无需输入其凭据，因为它们将具有有效的单一登录会话与 Microsoft 标识平台 v2.0 终结点。

若要了解详细信息，请参阅[发送注销请求](v2-protocols-oidc.md#send-a-sign-out-request)主题中[Microsoft 标识平台 v2.0 和 OpenID Connect 协议](v2-protocols-oidc.md)概念文档。

## <a name="application-registration"></a>应用程序注册

在应用程序注册期间你将注册**发布注销 URI**。 在本教程中，你注册`https://localhost:44321/signout-oidc`中**注销 URL**字段**高级设置**主题中**身份验证**页。 有关详细信息，请参阅[注册 web 应用的应用程序](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-1-MyOrg#register-the-webapp-app-webapp)

## <a name="aspnet-core-code"></a>ASP.NET Core 代码

### <a name="signout-button"></a>注销按钮

在中公开按钮注销`Views\Shared\_LoginPartial.cshtml`和身份验证的帐户 （这是当用户以前已登录时） 时，仅显示。

```html
@using Microsoft.Identity.Web
@if (User.Identity.IsAuthenticated)
{
    <ul class="nav navbar-nav navbar-right">
        <li class="navbar-text">Hello @User.GetDisplayName()!</li>
        <li><a asp-area="AzureAD" asp-controller="Account" asp-action="SignOut">Sign out</a></li>
    </ul>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li><a asp-area="AzureAD" asp-controller="Account" asp-action="SignIn">Sign in</a></li>
    </ul>
}
```

### <a name="signout-action-of-the-accountcontroller"></a>`Signout()` 操作 `AccountController`

按下**注销**web 应用触发器上的按钮`SignOut`上的操作`Account`控制器。 在以前版本的 ASP.NET core 模板，`Account`控制器已嵌入 web 应用，但这不再是这种情况，因为它现在是 ASP.NET Core 框架自身的一部分。 

代码`AccountController`是从 ASP.NET core 存储库在从可用[AccountController.cs](https://github.com/aspnet/AspNetCore/blob/master/src/Azure/AzureAD/Authentication.AzureAD.UI/src/Areas/AzureAD/Controllers/AccountController.cs)。 帐户控制中：

- 将 OpenID 重定向到 URI `/Account/SignedOut` ，以便在控制器调用 Azure AD 都没有执行过注销时
- 调用`Signout()`，它可让 OpenIdConnect 中间件，请联系 Microsoft 标识平台`logout`终结点的：

  - 清除浏览器中，从会话 cookie 和
  - 调用最后回拨**注销 URL**，后者) 默认情况下，显示出视图页的签名[SignedOut.html](https://github.com/aspnet/AspNetCore/blob/master/src/Azure/AzureAD/Authentication.AzureAD.UI/src/Areas/AzureAD/Pages/Account/SignedOut.cshtml)还提供了 ASP.NET Core 的一部分。

### <a name="intercepting-the-call-to-the-logout-endpoint"></a>截获调用`logout`终结点

ASP.NET Core OpenIdConnect 中间件让你的应用以截获调用到 Microsoft 标识平台`logout`终结点通过提供名为的 OpenIdConnect 事件`OnRedirectToIdentityProviderForSignOut`。 Web 应用使用它来尝试避免在注销时向用户显示选择帐户对话框。在中完成此拦截`AddAzureAdV2Authentication`的`Microsoft.Identity.Web`可重用的库。 请参阅[StartupHelpers.cs L58 L66](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/b87a1d859ff9f9a4a98eb7b701e6a1128d802ec5/Microsoft.Identity.Web/StartupHelpers.cs#L58-L66)

```CSharp
public static IServiceCollection AddAzureAdV2Authentication(this IServiceCollection services,
                                                            IConfiguration configuration)
{
    ...
    services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
    {
        ...
        options.Authority = options.Authority + "/v2.0/";
        ...
        // Attempt to avoid displaying the select account dialog when signing out
        options.Events.OnRedirectToIdentityProviderForSignOut = async context =>
        {
            var user = context.HttpContext.User;
            context.ProtocolMessage.LoginHint = user.GetLoginHint();
            context.ProtocolMessage.DomainHint = user.GetDomainHint();
            await Task.FromResult(0);
        };
    }
}
```

## <a name="aspnet-code"></a>ASP.NET Code

在 ASP.NET 中，从 signout （） 方法的控制器 (例如 AccountController) 上触发注销。 此方法不是 （与相反会发生什么情况在 ASP.NET Core 中），ASP.NET 框架的一部分，但不使用异步，而这正是它：

- 发送 OpenId 注销质询
- 清除的缓存
- 将重定向到它想要的页

```CSharp
/// <summary>
/// Send an OpenID Connect sign-out request.
/// </summary>
public void SignOut()
{
 HttpContext.GetOwinContext()
            .Authentication
            .SignOut(CookieAuthenticationDefaults.AuthenticationType);
 MsalAppBuilder.ClearUserTokenCache();
 Response.Redirect("/");
}
```

## <a name="protocol"></a>Protocol

如果不想要使用 ASP.NET Core 或 ASP.NET，则可以查看协议文档，可从[Open ID Connect](./v2-protocols-oidc.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-web-app-sign-user-production.md)
