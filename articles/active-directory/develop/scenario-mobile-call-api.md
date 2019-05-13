---
title: 移动应用程序调用 web Api 的调用的 web API |Microsoft 标识平台
description: 了解如何构建一个移动应用，调用 Web Api （调用 Web API）
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2fd65b9f97c373c55a3486e06e83fca7cf824cad
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075110"
---
# <a name="mobile-app-that-calls-web-apis---call-a-web-api"></a>调用 web Api-移动应用程序调用 web API

一旦您的应用程序具有已注册用户并接收令牌，MSAL 将公开有关用户、 其环境，以及颁发的令牌的信息的多个片段。 您的应用程序可以使用这些值来调用 web API 或向用户显示一条欢迎消息。

首先，我们将探讨 MSAL 结果，然后介绍如何使用中的访问令牌`AuthenticationResult`或`result`以调用受保护的 web API。

## <a name="msal-result"></a>MSAL 结果

- `AccessToken`：用于调用受保护的 web Api 中的持有者 HTTP 请求。
- `IdToken`：包含有关登录的用户，如其名称、 主租户和唯一标识符进行存储的有用声明。
- `ExpiresOn`： 该令牌的过期时间。 MSAL 负责应用的自动刷新。
- `TenantId`：用于登录的用户的租户标识符。 对于来宾用户 (Azure AD B2B)，这将是用户登录的不是其宿主租户的租户。  
- `Scopes`： 授予了与你的令牌的作用域。 这可能是请求的内容的子集。

此外，MSAL 还提供抽象以`Account`。 客户表示当前用户的登录帐户。

- `HomeAccountIdentifier`：用户的主租户的标识符。
- `UserName`：用户的首选的用户名。 这可能是空的 Azure AD B2C 用户。
- `AccountIdentifier`：已登录用户的标识符。 这将是相同`HomeAccountIdentifier`在大多数情况下除非该用户是另一个租户中的来宾。

## <a name="calling-an-api"></a>调用 API

准备好的访问令牌后，可以很轻松调用 web API。 您的应用程序将需要此令牌，构造 HTTP 请求，并执行它。

### <a name="android"></a>Android

```Java
        RequestQueue queue = Volley.newRequestQueue(this);
        JSONObject parameters = new JSONObject();

        try {
            parameters.put("key", "value");
        } catch (Exception e) {
            // Error when constructing
        }
        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
                parameters,new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // Successfully called graph, process data and send to UI 
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                // Error
            }
        }) {
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                
                // Put Access Token in HTTP request 
                headers.put("Authorization", "Bearer " + authResult.getAccessToken());
                return headers;
            }
        };

        request.setRetryPolicy(new DefaultRetryPolicy(
                3000,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        queue.add(request);
```

### <a name="ios"></a>iOS

```swift
        let url = URL(string: kGraphURI)
        var request = URLRequest(url: url!)

        // Put Access token in HTTP Request
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }
            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {
                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }

            // Successfully got data from Graph
            self.updateLogging(text: "Result from Graph: \(result))")
        }.resume()
```

### <a name="xamarin"></a>Xamarin

```CSharp
httpClient = new HttpClient();

// Put Access token in HTTP request 
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call Graph
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
...
}
```

## <a name="making-several-api-requests"></a>发出几个 API 请求

如果你需要多次调用相同的 API 或多个 Api 构建您的应用程序时有一些其他注意事项：

- ***增量许可***:Microsoft 标识平台允许应用程序来获取用户同意的情况下，如权限是必需的而不是所有前期。 每次您的应用程序已准备好调用 API，它应请求仅它要使用的作用域。
- ***条件性访问***:在某些情况下，可能会发出多个 API 请求时收到条件性访问的其他要求。 若要处理这种情况，请确保捕获错误与无提示请求并准备好进行交互式请求。 如果第一个请求具有没有应用的条件性访问策略，并且您的应用程序尝试以静默方式访问需要条件性访问的新 API，可以发生这种情况。 若要了解详细信息，请参阅[条件性访问指南](conditional-access-dev-guide.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-mobile-production.md)
