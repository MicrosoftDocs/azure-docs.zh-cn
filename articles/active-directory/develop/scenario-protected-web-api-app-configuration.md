---
title: 受保护的 web API 的应用程序代码配置 |Azure
description: 了解如何构建一个受保护的 Web API 和配置应用程序的代码。
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
ms.openlocfilehash: e206cb29338445e30a7462bcbaf0079236e75510
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074960"
---
# <a name="protected-web-api---code-configuration"></a>受保护的 web API 的代码配置

若要成功配置受保护 web API 的代码，您需要了解什么使受保护的 Api，需要配置的持有者令牌，以及如何验证令牌。

## <a name="what-makes-aspnetaspnet-core-apis-protected"></a>如何成为受保护的 ASP.NET/ASP.NET Core Api？

正如在 web 应用，ASP.NET/ASP.NET 核心的 web Api"受保护"，因为它们的控制器操作都带有前缀`[Authorize]`属性。 因此，如果使用已授权的标识来调用 API 可以仅调用控制器操作。

请考虑以下问题：

- Web API 如何知道调用它 （只是应用程序可以调用 web API） 的应用程序的标识？
- 如果应用程序调用 web API 代表某个用户，用户的标识是什么？

## <a name="bearer-token"></a>持有者令牌

有关该应用程序的标识以及用户的信息 （除非 web 应用程序接受来自守护程序应用程序的服务到服务调用），会保留在调用应用程序时设置的标头中的持有者令牌。

下面是C#显示的客户端获取与 MSAL.NET 令牌后调用该 API 的代码示例。

```CSharp
var scopes = new[] {$"api://.../access_as_user}";
var result = await app.AcquireToken(scopes)
                      .ExecuteAsync();

httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
```

> [!IMPORTANT]
> 到 Microsoft 标识平台终结点的客户端应用程序请求的持有者令牌**web API 的**。 Web API 是唯一的应用程序，应验证令牌，并查看其包含的声明。 客户端应用程序应永远不会查看令牌中的声明 (web API 可能会决定，在将来，它要求对令牌进行加密，这将中断可以破解的客户端应用程序打开的访问令牌)。

## <a name="jwtbearer-configuration"></a>JwtBearer 配置

本部分介绍需要配置的持有者令牌。

### <a name="config-file"></a>配置文件

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    /*
      You need specify the TenantId only if you want to accept access tokens from a single tenant
     (line of business app)
      Otherwise you can leave them set to common
      This can be:
      - A guid (Tenant ID = Directory ID)
      - 'common' (Any organization and personal accounts)
      - 'organizations' (any organization)
      - 'consumers' (Microsoft personal accounts)
    */
    "TenantId": "common"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="code-initialization"></a>代码初始化

当应用程序调用控制器操作保存在`[Authorize]`属性，ASP.NET/ASP.NET 核心查看调用的请求的 Authorization 标头中的持有者令牌并提取访问令牌，然后转发到 JwtBearer中间件，调用用于.NET 的 Microsoft 标识模型扩展。

在 ASP.NET Core 中，此中间件中初始化`Startup.cs`文件。

```CSharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
```

中间件添加到 web API 通过以下指令：

```CSharp
 services.AddAzureAdBearer(options => Configuration.Bind("AzureAd", options));
```

 目前，ASP.NET Core 模板创建 Azure AD v1.0 web Api。 但是，您可以轻松地更改它们以通过添加以下代码中的使用 Microsoft 标识平台终结点`Startup.cs`文件。

```CSharp
services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, options =>
{
    // This is a Microsoft identity platform v2.0 web API
    options.Authority += "/v2.0";

    // The web API accepts as audiences are both the Client ID (options.Audience) and api://{ClientID}
    options.TokenValidationParameters.ValidAudiences = new []
    {
     options.Audience,
     $"api://{options.Audience}"
    };

    // Instead of using the default validation (validating against a single tenant,
    // as we do in line of business apps),
    // we inject our own multi-tenant validation logic (which even accepts both V1 and V2 tokens)
    options.TokenValidationParameters.IssuerValidator = AadIssuerValidator.ValidateAadIssuer;
});
```

## <a name="token-validation"></a>令牌验证

通过定向 JwtBearer 中间件，如在 web 应用中的 OpenID Connect 中间件`TokenValidationParameters`来验证令牌。 该令牌进行解密 （如果需要）、 提取声明，并验证签名。 然后，通过检查以下数据来验证令牌：

- 其目标是 web API （受众）
- 它被颁发的应用程序可以调用 web API (sub)
- 它已签发由可信任安全令牌服务器 (STS) （颁发者）
- 令牌的生存期处于范围内 （过期）
- 它未被篡改 （签名）

也可以是特殊的验证。 例如，就可以验证签名密钥 （当嵌入在令牌中） 是受信任和令牌不在重播。 最后，某些协议需要特定的验证。

### <a name="validators"></a>验证程序

验证步骤捕获到验证程序，全部位于[适用于.NET 的 Microsoft 标识模型扩展](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet)开放源代码库，在一个源文件中：[Microsoft.IdentityModel.Tokens/Validators.cs](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/master/src/Microsoft.IdentityModel.Tokens/Validators.cs)

下表所述验证程序：

| 验证程序 | 描述 |
|---------|---------|
| `ValidateAudience` | 确保该令牌对令牌进行验证 （对我来说） 的应用程序。 |
| `ValidateIssuer` | 可确保令牌已由受信任的 STS 颁发 （来自我信任的人）。 |
| `ValidateIssuerSigningKey` | 验证令牌的信任关系的应用程序可确保用于签署令牌 （其中该密钥嵌入在令牌中，通常不需要特殊用例） 的密钥。 |
| `ValidateLifetime` | 确保仍 （或已） 有效令牌。 通过检查令牌的生存期 (`notbefore`，`expires`声明) 处于范围。 |
| `ValidateSignature` | 确保该标记尚未被篡改。 |
| `ValidateTokenReplay` | 可确保不重播令牌 （对于某些一次性使用协议的特殊情况）。 |

验证程序包括所有关联的属性与`TokenValidationParameters`类本身的 ASP.NET/ASP.NET 核心配置初始化。 在大多数情况下，您无需更改的参数。 一个异常不是单个租户的应用程序 （这是接受来自任何组织或个人 Microsoft 帐户的用户的 web 应用）--在这种情况下，必须验证颁发者。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [迁移到生产环境](scenario-protected-web-api-production.md)
