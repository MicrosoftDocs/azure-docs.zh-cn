---
title: 守护程序应用程序调用 web Api （获取令牌的应用）-Microsoft 标识平台
description: 了解如何构建守护程序应用调用 web Api （获取令牌）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
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
ms.openlocfilehash: aa4f5dc7a5aceaf81f71eacd36d131471a57e5c0
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075365"
---
# <a name="daemon-app-that-calls-web-apis---acquire-a-token"></a>守护程序应用调用 web Api-获取令牌

机密客户端应用程序构造后，您可以通过调用获取的应用令牌``AcquireTokenForClient``，传递作用域，并强制实施或未刷新令牌。

## <a name="scopes-to-request"></a>作用域，以请求

作用域，以请求客户端凭据流是资源的名称后跟`/.default`。 此表示法告知 Azure AD 即可使用**应用程序级权限**应用程序注册过程中以静态方式声明。 此外，如上所述，API 授予这些权限必须由租户管理员

### <a name="net"></a>.NET

```CSharp
ResourceId = "someAppIDURI";
var scopes = new [] {  ResourceId+"/.default"};
```

### <a name="python"></a>Python

在 MSAL。Python，配置文件如以下代码片段所示：

```Python
{
    "authority": "https://login.microsoftonline.com/organizations",
    "client_id": "your_client_id",
    "secret": "This is a sample only. You better NOT persist your password."
    "scope": ["https://graph.microsoft.com/.default"]
}
```

### <a name="java"></a>Java

```Java
public final static String KEYVAULT_DEFAULT_SCOPE = "https://vault.azure.net/.default";
```

### <a name="all"></a>全部

使用客户端凭据的作用域应始终为 resourceId +"/.default"

### <a name="case-of-v10-resources"></a>用例的 1.0 版资源

> [!IMPORTANT]
> 对于要求接受 v1.0 访问令牌的资源的访问令牌 MSAL （v2.0 终结点），Azure AD 通过采用最后一个反斜杠之前的所有内容并将其用作资源标识符分析期望的受众，从所请求的范围。
> 因此如果，如 Azure SQL (**https://database.windows.net**) 资源需要结尾斜杠的受众 (Azure sql: `https://database.windows.net/`)，你将需要请求的范围为`https://database.windows.net//.default`（请注意双斜杠）。 另请参阅 MSAL.NET 问题[#747](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/747):省略资源 url 尾部反斜杠，导致 sql 身份验证失败。

## <a name="acquiretokenforclient-api"></a>AcquireTokenForClient API

### <a name="net"></a>.NET

```CSharp
using Microsoft.Identity.Client;

// With client credentials flows the scopes is ALWAYS of the shape "resource/.default", as the
// application permissions need to be set statically (in the portal or by PowerShell), and then granted by
// a tenant administrator
string[] scopes = new string[] { "https://graph.microsoft.com/.default" };

AuthenticationResult result = null;
try
{
 result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    // The application does not have sufficient permissions
    // - did you declare enough app permissions in during the app creation?
    // - did the tenant admin needs to grant permissions to the application.
}
catch (MsalServiceException ex) when (ex.Message.Contains("AADSTS70011"))
{
    // Invalid scope. The scope has to be of the form "https://resourceurl/.default"
    // Mitigation: change the scope to be as expected !
}
```

#### <a name="application-token-cache"></a>应用程序令牌缓存

在 MSAL.NET，`AcquireTokenForClient`使用**应用程序令牌缓存**（所有其他 AcquireTokenXX 方法使用用户令牌缓存），不要调用`AcquireTokenSilent`之前调用`AcquireTokenForClient`作为`AcquireTokenSilent`使用**用户**令牌缓存。 `AcquireTokenForClient` 检查**应用程序**令牌缓存本身并对其进行更新。

### <a name="python"></a>Python

```Python
# Firstly, looks up a token from cache
# Since we are looking for token for the current app, NOT for an end user,
# notice we give account parameter as None.
result = app.acquire_token_silent(config["scope"], account=None)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    result = app.acquire_token_for_client(scopes=config["scope"])
```

### <a name="java"></a>Java

```Java
ClientCredentialParameters parameters = ClientCredentialParameters
        .builder(Collections.singleton(KEYVAULT_DEFAULT_SCOPE))
        .build();

CompletableFuture<AuthenticationResult> future = cca.acquireToken(parameters);

// You can complete the future in many different ways. Here we use .get() for simplicity
AuthenticationResult result = future.get();
```

### <a name="protocol"></a>Protocol

如果您还没有你选择的语言的库，你可能想要直接使用协议：

#### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一种情况：使用共享机密访问令牌请求

```Text
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

#### <a name="second-case-access-token-request-with-a-certificate"></a>第二种情况：使用证书访问令牌请求

```Text
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

### <a name="learn-more-about-the-protocol"></a>了解有关协议的详细信息

有关详细信息，请参阅协议文档：[Azure Active Directory v2.0 和 OAuth 2.0 客户端凭据流](v2-oauth2-client-creds-grant-flow.md)。

## <a name="troubleshooting"></a>故障排除

### <a name="did-you-use-the-resourcedefault-scope"></a>未使用的资源/.default 范围？

如果收到错误消息，告诉您使用无效的作用域，则可能没有使用`resource/.default`作用域。

### <a name="did-you-forget-to-provide-admin-consent-daemon-apps-need-it"></a>您是否忘记提供管理员许可？ 守护程序应用需要它 ！

如果调用 API 时遇到错误**特权不足以完成该操作**，租户管理员需要向应用程序授予权限。 请参阅上述客户端应用注册到的第 6 步。
通常，会看到等错误显示以下错误说明：

```JSon
Failed to call the web API: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<guid>",
      "date": "<date>"
    }
  }
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [守护程序应用-调用 web API](scenario-daemon-call-api.md)