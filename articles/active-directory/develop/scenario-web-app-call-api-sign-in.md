---
title: Web 应用调用 web Api （登录）-Microsoft 标识平台
description: 了解如何构建一个 Web 应用，用于调用 web Api （登录）
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
ms.openlocfilehash: 663cea72eb620217ad5fa8925d3bb00eedbf890c
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074555"
---
# <a name="web-app-that-calls-web-apis---sign-in"></a>Web 应用调用 web Api-登录

您已经知道如何将登录添加到你的 web 应用。 了解这[Web 应用登录用户-添加登录](scenario-web-app-sign-user-sign-in.md)。

什么是此处不同，是当用户签出、 来自此应用程序，或从任何应用程序，您希望从令牌缓存，与用户关联的令牌中删除。

## <a name="intercepting-the-callback-after-sign-out---single-sign-out"></a>截获后注销的单一注销回调

你的应用程序可以拦截后`logout`事件，例如清除与已注销的帐户关联的令牌缓存的条目。我们将看到在本教程中 （有关调用 Web API 的 Web 应用） 的第二部分中，web 应用将在缓存中存储用户的访问令牌。 截获后`logout`回调可以将 web 应用程序从令牌缓存中删除用户。 此机制进行了阐释`AddMsal()`方法的[StartupHelper.cs L137 143](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/b87a1d859ff9f9a4a98eb7b701e6a1128d802ec5/Microsoft.Identity.Web/StartupHelpers.cs#L137-L143)

**注销 Url**为你的应用程序，用户可以实现单一注销已注册。Microsoft 标识平台`logout`终结点将调用**注销 URL**注册到应用程序。 如果从 web 应用，或从另一个 web 应用或浏览器已启动注销，则会发生此调用。 有关详细信息，请参阅[单一注销](https://docs.microsoft.com/azure/active-directory/develop/v2-protocols-oidc#single-sign-out)概念性文档中。

```CSharp
public static IServiceCollection AddMsal(this IServiceCollection services, IEnumerable<string> initialScopes)
{
    services.AddTokenAcquisition();

    services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
    {
     ...
        // Handling the sign-out: removing the account from MSAL.NET cache
        options.Events.OnRedirectToIdentityProviderForSignOut = async context =>
        {
            // Remove the account from MSAL.NET token cache
            var _tokenAcquisition = context.HttpContext.RequestServices.GetRequiredService<ITokenAcquisition>();
            await _tokenAcquisition.RemoveAccount(context);
        };
    });
    return services;
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [获取 web 应用的令牌](scenario-web-app-call-api-acquire-token.md)
