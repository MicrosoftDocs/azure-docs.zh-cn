---
title: Web 应用调用 web Api （获取应用程序的令牌）-Microsoft 标识平台
description: 了解如何构建一个 Web 应用，调用 web Api （获取应用令牌）
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
ms.openlocfilehash: 653db995000308bb3ef78a9183696cd9d8ed1056
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074795"
---
# <a name="web-app-that-calls-web-apis---acquire-a-token-for-the-app"></a>Web 应用调用 web Api-获取应用程序的令牌

现在您已经生成客户端应用程序对象，将使用它来获取令牌，你将使用它来调用 web Api。 在 ASP.NET 或 ASP.NET Core 中，调用的 web API 然后就可以在控制器中。 它是关于：

- 获取 web API 使用令牌缓存的令牌。 为此，您调用 `AcquireTokenSilent`
- 调用受保护的 API 的访问令牌

## <a name="aspnet-core"></a>ASP.NET Core

控制器方法受`[Authorize]`强制用户从使用 Web 应用进行身份验证的属性。 下面是调用 Microsoft Graph 的代码

```CSharp
[Authorize]
public class HomeController : Controller
{
 ...
}
```

下面是操作的 HomeController，获取一个令牌以调用 Microsoft Graph 的简化的代码。

```CSharp
public async Task<IActionResult> Profile()
{
 var application = BuildConfidentialClientApplication(HttpContext, HttpContext.User);
 string accountIdentifier = claimsPrincipal.GetMsalAccountId();
 string loginHint = claimsPrincipal.GetLoginHint();

 // Get the account
 IAccount account = await application.GetAccountAsync(accountIdentifier);

 // Special case for guest users as the Guest iod / tenant id are not surfaced.
 if (account == null)
 {
  var accounts = await application.GetAccountsAsync();
  account = accounts.FirstOrDefault(a => a.Username == loginHint);
 }

 AuthenticationResult result;
 result = await application.AcquireTokenSilent(new []{"user.read"}, account)
                            .ExecuteAsync();
 var accessToken = result.AccessToken;
 ...
 // use the access token to call a web API
}
```

如果你想要了解详细信息中的总此方案所需的代码，请参阅第 2 阶段[2-1-Web 应用调用 Microsoft Graph](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-1-Call-MSGraph)的步骤[ms-标识-aspnetcore 的 web 应用的教程](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial)教程

有许多其他复杂性如所示：

- 为 Web 应用 （在本教程提供多个实现） 实现令牌缓存
- 当用户符号出从缓存中删除帐户
- 调用了几个 Api，包括需要增量许可

## <a name="aspnet"></a>ASP.NET

在 ASP.NET 中类似以下事项：

- 通过 [Authorize] 属性受保护的控制器操作将提取的租户 ID 和用户的用户 ID`ClaimsPrincipal`控制器的成员 (ASP.NET 使用`HttpContext.User`)
- 从中生成 MSAL.NET `IConfidentialClientApplication`
- IT 调用`AcquireTokenSilent`机密客户端应用程序的方法 

代码是类似于 ASP.NET Core 的所示的代码

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [调用 Web API](scenario-web-app-call-api-call-api.md)
