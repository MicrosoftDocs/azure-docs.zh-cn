---
title: Web API 的调用下游 web Api （应用程序的代码配置）-Microsoft 标识平台
description: 了解如何构建 web API 调用 web Api （应用程序的代码配置）
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
ms.openlocfilehash: 3dedef2d22df9c8c81410296bdb0c4814bd98b80
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507133"
---
# <a name="web-api-that-calls-web-apis---code-configuration"></a>Web API 调用 web Api 的代码配置

注册你的 web API 后，可以配置应用程序的代码。

若要配置你的 web API，以便它将调用下游 web Api 的代码用于项目的 web API 的代码的基础上构建。 有关详细信息，请参阅[保护的 web API 的应用配置](scenario-protected-web-api-app-configuration.md)。

## <a name="code-subscribed-to-ontokenvalidated"></a>代码到 OnTokenValidated 订阅

基于任何受保护的 web Api 的代码配置，你需要与你的 API 调用时收到的持有者令牌验证订阅：

```CSharp
/// <summary>
/// Protects the web API with Microsoft Identity Platform v2.0 (AAD v2.0)
/// This supposes that the configuration files have a section named "AzureAD"
/// </summary>
/// <param name="services">Service collection to which to add authentication</param>
/// <param name="configuration">Configuration</param>
/// <returns></returns>
public static IServiceCollection AddProtectedApiCallsWebApis(this IServiceCollection services,
                                                             IConfiguration configuration,
                                                             IEnumerable<string> scopes)
{
    services.AddTokenAcquisition();
    services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        // When an access token for our own web API is validated, we add it 
        // to MSAL.NET's cache so that it can be used from the controllers.
        options.Events = new JwtBearerEvents();

        options.Events.OnTokenValidated = async context =>
        {
            context.Success();

            // Adds the token to the cache, and also handles the incremental consent 
            // and claim challenges
            AddAccountToCacheFromJwt(context, scopes);
            await Task.FromResult(0);
        };
    });
    return services;
}
```

## <a name="on-behalf-of-flow"></a>代理流

AddAccountToCacheFromJwt() 方法需要：

- 实例化的 MSAL 机密客户端应用程序。
- 调用`AcquireTokenOnBehalf`交换由 web API，针对同一用户，但对于我们的 API 来调用下游 API 的持有者令牌的客户端已获取的持有者令牌。

### <a name="instantiate-a-confidential-client-application"></a>实例化机密客户端应用程序

此流才可用在机密客户端流，因此受保护的 web API 客户端凭据 （客户端机密或证书） 提供给[ConfidentialClientApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.appconfig.confidentialclientapplicationbuilder?view=azure-dotnet-preview)通过`WithClientSecret`或`WithCertificate`方法，分别。

![image](https://user-images.githubusercontent.com/13203188/55967244-3d8e1d00-5c7a-11e9-8285-a54b05597ec9.png)

```CSharp
IConfidentialClientApplication app;

#if !VariationWithCertificateCredentials
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
           .WithClientSecret(config.ClientSecret)
           .Build();
#else
// Building the client credentials from a certificate
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
    .WithCertificate(certificate)
    .Build();
#endif
```

### <a name="how-to-call-on-behalf-of"></a>如何在代表的调用

通过调用完成上代表的 (OBO) 调用[AcquireTokenOnBehalf](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.apiconfig.acquiretokenonbehalfofparameterbuilder?view=azure-dotnet-preview)方法`IConfidentialClientApplication`接口。

`ClientAssertion`从 web API 从它自己的客户端收到的持有者令牌生成。 有[两个构造函数](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.clientcredential.-ctor?view=azure-dotnet)，另一个采用 JWT 持有者令牌，和一个接受任何类型的用户断言 (安全令牌，在名为的其他参数则指定哪种类型的另一种`assertionType`)。

![image](https://user-images.githubusercontent.com/13203188/37082180-afc4b708-21e3-11e8-8af8-a6dcbd2dfba8.png)

在实践中，OBO 流通常用于下游 api 获取令牌并将其存储在 MSAL.NET 用户令牌缓存中，以便 web API 的其他部分可以更高版本上调用[重写](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.clientapplicationbase.acquiretokensilent?view=azure-dotnet)的``AcquireTokenOnSilent``以调用下游 Api。 如果需要此项刷新令牌，的效果。

```CSharp
private void AddAccountToCacheFromJwt(IEnumerable<string> scopes, JwtSecurityToken jwtToken, ClaimsPrincipal principal, HttpContext httpContext)
{
 try
 {
  UserAssertion userAssertion;
  IEnumerable<string> requestedScopes;
  if (jwtToken != null)
  {
   userAssertion = new UserAssertion(jwtToken.RawData, "urn:ietf:params:oauth:grant-type:jwt-bearer");
   requestedScopes = scopes ?? jwtToken.Audiences.Select(a => $"{a}/.default");
  }
  else
  {
   throw new ArgumentOutOfRangeException("tokenValidationContext.SecurityToken should be a JWT Token");
  }

  // Create the application
  var application = BuildConfidentialClientApplication(httpContext, principal);

  // .Result to make sure that the cache is filled-in before the controller tries to get access tokens
  var result = application.AcquireTokenOnBehalfOf(requestedScopes.Except(scopesRequestedByMsalNet),
                                                  userAssertion)
                                        .ExecuteAsync()
                                        .GetAwaiter().GetResult();
 }
 catch (MsalException ex)
 {
  Debug.WriteLine(ex.Message);
  throw;
 }
}
```

## <a name="protocol"></a>Protocol

有关上代表的协议的详细信息，请参阅[Microsoft 标识平台和 OAuth 2.0 代理流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [获取应用程序令牌](scenario-web-api-call-api-acquire-token.md)
