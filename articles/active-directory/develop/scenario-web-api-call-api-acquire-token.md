---
title: Web API 调用其他 web 的 Api （获取应用程序的令牌）-Microsoft 标识平台
description: 了解如何构建 web API 调用其他 web Api （获取应用令牌）。
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
ms.openlocfilehash: 986e2e0f8a481d61dc870af2548290658b44d2d3
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231110"
---
# <a name="web-api-that-calls-web-apis---acquire-a-token-for-the-app"></a>Web API 调用 web Api-获取应用程序的令牌

构建客户端应用程序后对象，请使用它来获取可用来调用 web API 的令牌。

## <a name="code-in-the-controller"></a>在控制器中的代码

下面是代码的将调用的调用 （名为 todolist） 的下游 API 的 API 控制器操作中示例。

```CSharp
private async Task GetTodoList(bool isAppStarting)
{
 ...
 //
 // Get an access token to call the To Do service.
 //
 AuthenticationResult result = null;
 try
 {
  app = BuildConfidentialClient(HttpContext, HttpContext.User);
  result = await app.AcquireTokenSilent(Scopes, account)
                     .ExecuteAsync()
                     .ConfigureAwait(false);
 }
...
}
```

`BuildConfidentialClient()` 类似于在本文中所看到的内容[Web API 调用 web Api 的应用配置](scenario-web-api-call-api-app-configuration.md)。 `BuildConfidentialClient()` 实例化`IConfidentialClientApplication`包含仅一个帐户信息的缓存。 该帐户提供的`GetAccountIdentifier`方法。

`GetAccountIdentifier`方法使用与 web API 收到的 JWT 的用户的标识关联的声明：

```CSharp
public static string GetMsalAccountId(this ClaimsPrincipal claimsPrincipal)
{
 string userObjectId = GetObjectId(claimsPrincipal);
 string tenantId = GetTenantId(claimsPrincipal);

 if (    !string.IsNullOrWhiteSpace(userObjectId)
      && !string.IsNullOrWhiteSpace(tenantId))
 {
  return $"{userObjectId}.{tenantId}";
 }

 return null;
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [调用 web API](scenario-web-api-call-api-call-api.md)
