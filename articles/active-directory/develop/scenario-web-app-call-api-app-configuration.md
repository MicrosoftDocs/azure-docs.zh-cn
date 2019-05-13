---
title: Web 应用调用 web Api （代码配置）-Microsoft 标识平台
description: 了解如何构建 Web 应用调用 web Api （应用程序的代码配置）
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
ms.openlocfilehash: 95a5e1ed89b6330a0b6a49cb20d8bf0ef3587d48
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074735"
---
# <a name="web-app-that-calls-web-apis---code-configuration"></a>Web 应用调用 web Api 的代码配置

如中所示[Web 应用登录用户的情况下](scenario-web-app-sign-user-overview.md)，并假设登录用户委托给 Open ID connect (OIDC) 中间件，你想要挂钩 OIDC 进程中。 执行此操作的方法是根据框架不同，使用 （此处 ASP.NET 和 ASP.NET Core），但最后，您将订阅到中间件 OIDC 事件。 原则是：

- 希望您告诉 ASP.NET 或 ASP.NET core 请求授权代码。 通过这样做此 ASP.NET/ASP.NET core 将让用户登录并同意，
- 通过 Web 应用，将订阅接收授权代码。
- 收到的身份验证代码后，将使用 MSAL 库来兑换代码和生成的访问令牌和刷新令牌缓存中的令牌存储。 在这里，缓存可以用于在应用程序的其他部分以无提示方式获取其他令牌。

## <a name="libraries-supporting-web-app-scenarios"></a>库支持的 Web 应用方案

有关在 Web 应用支持的授权代码流的库是：

| MSAL 库 | 描述 |
|--------------|-------------|
| ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | 支持的平台是.NET Framework 和.NET Core 平台 （不 UWP、 Xamarin.iOS 和 Xamarin.Android 为这些平台用于生成公共客户端应用程序） |
| ![Python](media/sample-v2-code/logo_python.png) <br/> MSAL.Python | 进度-公共预览版中的开发 |
| ![Java](media/sample-v2-code/logo_java.png) <br/> MSAL.Java | 进度-公共预览版中的开发 |

## <a name="aspnet-core-configuration"></a>ASP.NET Core 配置

在 ASP.NET Core 中，在发生情况`Startup.cs`文件。 你将想要订阅`OnAuthorizationCodeReceived`打开 ID 为连接事件，并通过此事件，调用 MSAL。NET 的方法`AcquireTokenFromAuthorizationCode`该效果将存储在令牌缓存、 请求作用域的访问令牌和刷新令牌将用于刷新访问令牌即将到期时或若要获取代表相同的用户令牌但对于不同的资源。

下面的代码中的注释将帮助你了解某些困难方面的编辑 MSAL.NET 和 ASP.NET Core

```CSharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
  {
   // Response type. We ask ASP.NET to request an Auth Code, and an IDToken
   options.ResponseType = OpenIdConnectResponseType.CodeIdToken;

   // This "offline_access" scope is needed to get a refresh token when users sign in with
   // their Microsoft personal accounts
   // (it's required by MSAL.NET and automatically provided by Azure AD when users
   // sign in with work or school accounts, but not with their Microsoft personal accounts)
   options.Scope.Add(OidcConstants.ScopeOfflineAccess);
   options.Scope.Add("user.read"); // for instance

   // Handling the auth redemption by MSAL.NET so that a token is available in the token cache
   // where it will be usable from Controllers later (through the TokenAcquisition service)
   var handler = options.Events.OnAuthorizationCodeReceived;
   options.Events.OnAuthorizationCodeReceived = async context =>
   {
    // As AcquireTokenByAuthorizationCode is asynchronous we want to tell ASP.NET core
    // that we are handing the code even if it's not done yet, so that it does 
    // not concurrently call the Token endpoint.
    context.HandleCodeRedemption();

    // Call MSAL.NET AcquireTokenByAuthorizationCode
    var application = BuildConfidentialClientApplication(context.HttpContext,
                                                         context.Principal);
    var result = await application.AcquireTokenByAuthorizationCode(scopes.Except(scopesRequestedByMsalNet), context.ProtocolMessage.Code)
                                  .ExecuteAsync();

    // Do not share the access token with ASP.NET Core otherwise ASP.NET will cache it
    // and will not send the OAuth 2.0 request in case a further call to
    // AcquireTokenByAuthorizationCodeAsync in the future for incremental consent 
    // (getting a code requesting more scopes)
    // Share the ID Token so that the identity of the user is known in the application (in 
    // HttpContext.User)
    context.HandleCodeRedemption(null, result.IdToken);

    // Call the previous handler if any
    await handler(context);
   };
```

在 ASP.NET Core 中，生成机密客户端应用程序使用 HttpContext 中的信息。 了解有关 Web 应用，并已登录用户 URL 此 HttpContext (在`ClaimsPrincipal`)。 它还使用 ASP.NET Core 配置，其中包含"AzureAD"部分中，并且其绑定到`_applicationOptions`数据结构。 最后该应用程序需要维护令牌缓存。

```CSharp
/// <summary>
/// Creates an MSAL Confidential client application
/// </summary>
/// <param name="httpContext">HttpContext associated with the OIDC response</param>
/// <param name="claimsPrincipal">Identity for the signed-in user</param>
/// <returns></returns>
private IConfidentialClientApplication BuildConfidentialClientApplication(HttpContext httpContext, ClaimsPrincipal claimsPrincipal)
{
 var request = httpContext.Request;

 // Find the URI of the application)
 string currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, azureAdOptions.CallbackPath ?? string.Empty);

 // Updates the authority from the instance (including national clouds) and the tenant
 string authority = $"{azureAdOptions.Instance}{azureAdOptions.TenantId}/";

 // Instantiates the application based on the application options (including the client secret)
 var app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
               .WithRedirectUri(currentUri)
               .WithAuthority(authority)
               .Build();

 // Initialize token cache providers. In the case of Web applications, there must be one
 // token cache per user (here the key of the token cache is in the claimsPrincipal which
 // contains the identity of the signed-in user)
 if (this.UserTokenCacheProvider != null)
 {
  this.UserTokenCacheProvider.Initialize(app.UserTokenCache, httpContext, claimsPrincipal);
 }
 if (this.AppTokenCacheProvider != null)
 {
  this.AppTokenCacheProvider.Initialize(app.AppTokenCache, httpContext);
 }
 return app;
}
```

`AcquireTokenByAuthorizationCode` 真正兑换授权代码请求的 ASP.NET，并获取添加到 MSAL.NET 用户令牌缓存的令牌。 在这里，它们然后，在 ASP.NET Core 控制器中使用。

## <a name="aspnet-configuration"></a>ASP.NET 配置

ASP.NET 处理事情的方式非常相似，只不过 OpenIdConnect 和对订阅的配置`OnAuthorizationCodeReceived`事件发生在`App_Start\Startup.Auth.cs`文件。 您会发现类似的概念，只不过此处您将需要在配置文件中，这是低一些指定 RedirectUri 可靠：

```CSharp
private void ConfigureAuth(IAppBuilder app)
{
 app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

 app.UseCookieAuthentication(new CookieAuthenticationOptions { });

 app.UseOpenIdConnectAuthentication(
 new OpenIdConnectAuthenticationOptions
 {
  Authority = Globals.Authority,
  ClientId = Globals.ClientId,
  RedirectUri = Globals.RedirectUri,
  PostLogoutRedirectUri = Globals.RedirectUri,
  Scope = Globals.BasicSignInScopes, // a basic set of permissions for user sign in & profile access
  TokenValidationParameters = new TokenValidationParameters
  {
  // We'll inject our own issuer validation logic below.
  ValidateIssuer = false,
  NameClaimType = "name",
  },
  Notifications = new OpenIdConnectAuthenticationNotifications()
  {
   AuthorizationCodeReceived = OnAuthorizationCodeReceived,
  }
 });
}
```

```CSharp
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification context)
{
 // Upon successful sign in, get & cache a token using MSAL
 string userId = context.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
 IConfidentialClientApplication clientapp;
 clientApp = ConfidentialClientApplicationBuilder.Create(Globals.ClientId)
                  .WithClientSecret(Globals.ClientSecret)
                  .WithRedirectUri(Globals.RedirectUri)
                  .WithAuthority(new Uri(Globals.Authority))
                  .Build();

  EnablePersistence(HttpContext, clientApp.UserTokenCache, clientApp.AppTokenCache);

 AuthenticationResult result = await clientapp.AcquireTokenByAuthorizationCode(scopes, context.Code)
                                              .ExecuteAsync();
}
```

### <a name="msalnet-token-cache-for-a-aspnet-core-web-app"></a>ASP.NET (Core) Web 应用的 MSAL.NET 令牌缓存

在 web 应用 （或 web Api 作为一种事实） 的令牌缓存实现是不同于桌面应用程序令牌缓存实现 (通常[基于文件](scenario-desktop-acquire-token.md#file-based-token-cache)。 它可以使用 ASP.NET/ASP.NET 核心会话或的 Redis 缓存或数据库或甚至 Azure Blog 存储。 在代码中更高版本此代码片段是对象的`EnablePersistence(HttpContext, clientApp.UserTokenCache, clientApp.AppTokenCache);`方法调用中，这是将缓存服务的绑定。 下面是超出了本指南范围的方案，但下面提供了链接，会发生什么的详细信息。

> [!IMPORTANT]
> 非常重要的一点是，对于 web 应用和 web Api，应会每个用户 （每个帐户） 的一个令牌缓存。 需要序列化每个帐户的令牌缓存。

有关如何使用令牌缓存适用于 Web 应用和 web Api 的示例中有[ASP.NET Core Web 应用教程](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial)阶段[2-2 令牌缓存](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-2-TokenCache)。 为实现具有以下文件夹看[TokenCacheProviders](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Web/TokenCacheProviders)中[microsoft 身份验证的扩展-为-dotnet](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet)库 (在[Microsoft.Identity.Client.Extensions.Web](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Web)文件夹。

## <a name="next-steps"></a>后续步骤

此时，当用户登录令牌存储在令牌缓存。 我们来看它然后中如何使用 Web 应用的其他部分。

> [!div class="nextstepaction"]
> [登录到 Web 应用](scenario-web-app-call-api-sign-in.md)
