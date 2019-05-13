---
title: Web 应用登录用户 （代码配置）-Microsoft 标识平台
description: 了解如何生成 web 应用登录用户 （代码配置）
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
ms.openlocfilehash: b2204fe3e08b3c4b909ddc8b7ade4cec219d34fb
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406632"
---
# <a name="web-app-that-signs-in-users---code-configuration"></a>Web 应用登录用户的代码配置

了解如何配置该登录用户的 Web 应用程序的代码。

## <a name="libraries-used-to-protect-web-apps"></a>用于保护 Web 应用库

<!-- This section can be in an include for Web App and Web APIs -->
是用于保护 Web 应用 （和 Web API） 的库：

| 平台 | 库 | 描述 |
|----------|---------|-------------|
| ![.NET](media/sample-v2-code/logo_net.png) | [适用于.NET 的标识模型扩展](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) | 使用直接通过 ASP.NET 和 ASP.NET Core，适用于.NET 的 Microsoft 标识扩展提供了一组.NET Framework 和.NET Core 上运行的 Dll。 从 ASP.NET/ASP.NET Core Web 应用程序，您可以控制令牌验证使用**TokenValidationParameters**类 （尤其是在某些 ISV 方案） |

## <a name="aspnet-core-configuration"></a>ASP.NET Core 配置

### <a name="application-configuration-files"></a>应用程序配置文件

在 ASP.NET Core Web 应用程序登录用户的 Microsoft 标识平台配置通过`appsettings.json`文件。 您需要填写的设置如下：

- 在云中`Instance`如果你想在国家云中运行您的应用程序
- 中的访问群体 `tenantId`
- `clientId`为应用程序，如从 Azure 门户复制。

```JSon
{
  "AzureAd": {
    // Azure Cloud instance among:
    // "https://login.microsoftonline.com/" for Azure Public cloud.
    // "https://login.microsoftonline.us/" for Azure US government.
    // "https://login.microsoftonline.de/" for Azure AD Germany
    // "https://login.chinacloudapi.cn/" for Azure AD China operated by 21Vianet
    "Instance": "https://login.microsoftonline.com/",

    // Azure AD Audience among:
    // - the tenant Id as a a GUID obtained from the azure portal to sign-in users in your organization
    // - "organizations" to sign-in users in any work or school accounts
    // - "common" to sign-in users with any work and school account or Microsoft personal account
    // - "consumers" to sign-in users with Microsoft personal account only
    "TenantId": "[Enter the tenantId here]]",

    // Client Id (Application ID) obtained from the Azure portal
    "ClientId": "[Enter the Client Id]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc"
  }
}
```

在 ASP.NET Core 中，是包含 URL 的另一个文件 (`applicationUrl`) 和 SSL 端口 (`sslPort`) 为你的应用程序，以及各种配置文件。

```JSon
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3110/",
      "sslPort": 44321
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "webApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:3110/"
    }
  }
}
```

在 Azure 门户中，答复你需要注册中的 Uri**身份验证**应用程序页必须与匹配这些 Url; 也就是说，对于上面的两个配置文件，它们将是`https://localhost:44321/signin-oidc`作为 applicationUrl是`http://localhost:3110`但`sslPort`是指定 (44321) 和`CallbackPath`是`/signin-oidc`中定义`appsettings.json`。
  
同样，在注销 URI 将设置为`https://localhost:44321/signout-callback-oidc`。

### <a name="initialization-code"></a>初始化代码

在 ASP.NET Core Web 应用 （和 Web Api） 中，执行应用程序初始化的代码位于`Startup.cs`文件，并且，若要添加使用 Microsoft 标识平台 (以前称为 Azure AD) v2.0 身份验证，你将需要添加以下代码。 在代码中的注释应该浅显易懂。

```CSharp
 services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
         .AddAzureAD(options => configuration.Bind("AzureAd", options));

 services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
 {
  // The ASP.NET core templates are currently using Azure AD v1.0, and compute
  // the authority (as {Instance}/{TenantID}). We want to use the Microsoft Identity Platform v2.0 endpoint
  options.Authority = options.Authority + "/v2.0/";

  // If you want to restrict the users that can sign-in to specific organizations
  // Set the tenant value in the appsettings.json file to 'organizations', and add the
  // issuers you want to accept to options.TokenValidationParameters.ValidIssuers collection.
  // Otherwise validate the issuer
  options.TokenValidationParameters.IssuerValidator = AadIssuerValidator.ForAadInstance(options.Authority).ValidateAadIssuer;

  // Set the nameClaimType to be preferred_username.
  // This change is needed because certain token claims from Azure AD v1.0 endpoint
  // (on which the original .NET core template is based) are different in Azure AD v2.0 endpoint.
  // For more details see [ID Tokens](https://docs.microsoft.com/azure/active-directory/develop/id-tokens)
  // and [Access Tokens](https://docs.microsoft.com/azure/active-directory/develop/access-tokens)
  options.TokenValidationParameters.NameClaimType = "preferred_username";
  ...
```

## <a name="aspnet-configuration"></a>ASP.NET 配置

在 ASP.NET 中，将应用程序配置通过`Web.Config`文件

```XML
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, please visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:ClientId" value="[Enter your client ID, as obtained from the app registration portal]" />
    <add key="ida:ClientSecret" value="[Enter your client secret, as obtained from the app registration portal]" />
    <add key="ida:AADInstance" value="https://login.microsoftonline.com/{0}{1}" />
    <add key="ida:RedirectUri" value="https://localhost:44326/" />
    <add key="vs:EnableBrowserLink" value="false" />
  </appSettings>
```

为 ASP.NET Web 应用中的身份验证相关的代码 / Web Api 位于`App_Start/Startup.Auth.cs`文件。

```CSharp
 public void ConfigureAuth(IAppBuilder app)
 {
  app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

  app.UseCookieAuthentication(new CookieAuthenticationOptions());

  app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
     // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0
     // The `Scope` describes the initial permissions that your app will need.
     //  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/
     ClientId = clientId,
     Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0"),
     RedirectUri = redirectUri,
     Scope = "openid profile",
     PostLogoutRedirectUri = redirectUri,
    });
 }
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [登录和注销](scenario-web-app-sign-user-sign-in.md)
