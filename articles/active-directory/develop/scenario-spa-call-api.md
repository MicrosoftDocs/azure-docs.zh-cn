---
title: 单页面应用程序 （调用 web API）-Microsoft 标识平台
description: 了解如何生成单页面应用程序 （调用 web API）
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/06/2019
ms.author: CelesteDG
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 01f141a5374c0e794b264f6e0135ca3e15ff8359
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074840"
---
# <a name="single-page-application---call-a-web-api"></a>单页面应用程序的一个 web API 的调用

我们建议您调用`acquireTokenSilent`方法获取，或调用 web API 之前续订访问令牌。 一旦您有一个令牌，就可以调用受保护的 web API。

## <a name="call-a-web-api"></a>调用 Web API

### <a name="javascript"></a>JavaScript

使用作为持有者 HTTP 请求中获取的访问令牌调用 Microsoft 图形 API 等任何 web API。 例如：

```javascript
    var headers = new Headers();
    var bearer = "Bearer " + access_token;
    headers.append("Authorization", bearer);
    var options = {
         method: "GET",
         headers: headers
    };
    var graphEndpoint = "https://graph.microsoft.com/v1.0/me";

    fetch(graphEndpoint, options)
        .then(function (response) {
             //do something with response
        }
```

### <a name="angular"></a>Angular

如中所述[获取令牌部分](scenario-spa-acquire-token.md)，MSAL Angular 包装器利用 HTTP 侦听器，以便自动以无提示方式获取访问令牌并将它们附加到 Api 收到的 HTTP 请求。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-spa-production.md)
