---
title: 桌面应用程序调用 web Api （调用 web API）-Microsoft 标识平台
description: 了解如何构建桌面应用调用 web Api （调用 web API）
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
ms.openlocfilehash: 6fb240b54c3d698ead9d3427f603acca2b53745a
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075920"
---
# <a name="desktop-app-that-calls-web-apis---call-a-web-api"></a>调用 web Api-的桌面应用调用 web API

现在，您有一个令牌，可以调用受保护的 web API。

## <a name="calling-a-web-api-from-net"></a>从.NET 调用 web API

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

<!--
More includes will come later for Python and Java
-->

## <a name="calling-several-apis---incremental-consent-and-conditional-access"></a>调用了几个 Api 的增量许可和条件性访问

如果你需要为同一用户调用了几个 Api 的第一个 API 获取令牌后，可以只需调用`AcquireTokenSilent`，并且你将获得一个令牌用于其他 Api 以无提示方式大部分时间。

```CSharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
```

需要交互的情况下以下情况：

- 用户许可的第一个 api，但现在需要表示同意的多个作用域 （增量许可）
- 第一个 API 不需要多个身份验证，但下一个。

```CSharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

try
{
 result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
 result = await app.AcquireTokenInteractive("scopeApi2")
                  .WithClaims(ex.Claims)
                  .ExecuteAsync();
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-desktop-production.md)
