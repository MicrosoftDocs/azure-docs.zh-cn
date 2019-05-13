---
title: Web API 调用 web Api （Api 调用）-Microsoft 标识平台
description: 了解如何构建 web API 的调用下游 web Api （调用 web API）。
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
ms.openlocfilehash: 1cd093cc68a21558dc326221eeaa8c034c24f1c2
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074720"
---
# <a name="web-api-that-calls-web-apis---call-an-api"></a>Web API 调用 web Api 的调用的 API

一旦您有一个令牌，就可以调用受保护的 web API。 这是从 ASP.NET/ASP.NET Core web API 控制器。

## <a name="controller-code"></a>控制器代码

下面是示例代码中所示的延续[受保护 web API 调用 web Api-获取令牌](scenario-web-api-call-api-acquire-token.md)、 被调用的 API 控制器操作，调用下游 API （名为 todolist）。

获取令牌后, 将其用作持有者令牌以调用下游 API。

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

// Once the token has been returned by MSAL, add it to the http authorization header, before making the call to access the To Do list service.
_httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the To Do list service.
HttpResponseMessage response = await _httpClient.GetAsync(TodoListBaseAddress + "/api/todolist");
...
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-web-api-call-api-production.md)
