---
title: Web 应用调用 web Api （调用 web API）-Microsoft 标识平台
description: 了解如何构建一个 Web 应用，调用 web Api （调用 web API）
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
ms.openlocfilehash: dd44dda06b2f6fc48538f2fb74c0bf8e04d0362b
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074630"
---
# <a name="web-app-that-calls-web-apis---call-a-web-api"></a>Web 应用调用 web Api 的调用 web API

现在，您有一个令牌，可以调用受保护的 web API。

## <a name="aspnet-core"></a>ASP.NET Core

下面是操作的简化的代码`HomeController`。 此代码获取令牌以调用 Microsoft Graph。 此时间代码已添加，显示了如何调用 Microsoft Graph 为 REST API。

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

 // Calls the web API (here the graph)
 HttpClient httpClient = new HttpClient();
 httpClient.DefaultRequestHeaders.Authorization =
     new AuthenticationHeaderValue(Constants.BearerAuthorizationScheme,accessToken);
 var response = await httpClient.GetAsync($"{webOptions.GraphApiUrl}/beta/me");

 if (response.StatusCode == HttpStatusCode.OK)
 {
  var content = await response.Content.ReadAsStringAsync();

  dynamic me = JsonConvert.DeserializeObject(content);
  return me;
 }

 ViewData["Me"] = me;
 return View();
}
```

> [!NOTE]
> 可以使用同样的原则来调用任何 web API。
>
> 大多数 Azure web Api 提供简化调用它的 SDK。 这也是 Microsoft Graph 的大小写。 将在下一篇文章中了解在哪里可以找到说明这些方面的教程。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-web-app-call-api-production.md)
