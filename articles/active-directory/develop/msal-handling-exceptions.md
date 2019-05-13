---
title: 错误和异常 (MSAL) | Azure
description: 了解如何处理 MSAL 应用程序中的错误和异常、条件访问与声明质询。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/10/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 43c98181c926410bea2acf64bf1ed4d588c12616
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138964"
---
# <a name="handling-exceptions-and-errors-using-msal"></a>使用 MSAL 处理异常和错误
Microsoft 身份验证库 (MSAL) 中的异常旨在帮助应用开发人员进行故障排除，而不会向最终用户显示。 异常消息未经本地化。

处理异常和错误时，可以使用异常类型本身和错误代码来区分不同的异常。  有关错误代码的列表，请参阅[身份验证和授权错误代码](reference-aadsts-error-codes.md)。

## <a name="net-exceptions"></a>.NET 异常
处理异常时，可以使用异常类型本身和 `ErrorCode` 成员来区分不同的异常。 `ErrorCode` 的值是 [MsalError](/dotnet/api/microsoft.identity.client.msalerror?view=azure-dotnet#fields) 类型的常量。

也可以查看 [MsalClientException](/dotnet/api/microsoft.identity.client.msalexception?view=azure-dotnet#fields)、[MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet#fields) 和 [MsalUIRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet#fields) 字段。

如果引发了 [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet)，错误代码可能包含[身份验证和授权错误代码](reference-aadsts-error-codes.md)中提供的代码。

### <a name="common-exceptions"></a>常见异常
下面是可能引发的常见异常和一些可能的缓解措施。

| 异常 | 错误代码 | 缓解措施|
| --- | --- | --- |
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet) | AADSTS65001：用户或管理员尚未许可使用名为“{appName}”、ID 为“{appId}”的应用程序。 针对此用户和资源发送交互式授权请求。| 需要先获取用户的许可。 如果未使用 .NET Core（它没有任何 Web UI），请调用 `AcquireTokeninteractive`（仅一次）。 如果你使用 .NET Core 或者不希望执行 `AcquireTokenInteractive`，则用户可以导航到某个 URL 来提供许可： https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read。 若要调用 `AcquireTokenInteractive`，请使用 `app.AcquireTokenInteractive(scopes).WithAccount(account).WithClaims(ex.Claims).ExecuteAsync();`|
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet) | AADSTS50079：用户必须使用多重身份验证。| 无缓解措施 - 如果为租户配置了 MFA 并且 AAD 决定强制实施 MFA，则需要回退到 `AcquireTokenInteractive` 或 `AcquireTokenByDeviceCode` 等交互式流。|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet#fields) |AADSTS90010：不支持通过 */common* 或 */consumers* 终结点的授予类型。 请使用 */organizations* 或特定于租户的终结点。 使用了 */common*。| 根据 Azure AD 发出的消息中所述，颁发机构需要使用一个租户或 */organizations*。|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet#fields) | AADSTS70002：请求正文必须包含以下参数：“client_secret 或 client_assertion”。| 如果你的应用程序未注册为 Azure AD 中的公共客户端应用程序，则可能发生此问题。 在 Azure 门户中编辑应用程序的清单，并将 `allowPublicClient` 设置为 `true`。 |
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception?view=azure-dotnet)| unknown_user 消息：无法识别已登录的用户| 库无法查询当前的 Windows 已登录用户，或者此用户未加入 AD 或 AAD（不支持已加入工作区的用户）。 缓解措施 1：在 UWP 中，检查应用程序是否具有以下功能：企业身份验证、专用网络（客户端和服务器）、用户帐户信息。 缓解措施 2：实现自己的逻辑以提取用户名（例如 john@contoso.com），并使用 `AcquireTokenByIntegratedWindowsAuth` 表单来提取用户名。|
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception?view=azure-dotnet)|integrated_windows_auth_not_supported_managed_user| 此方法依赖于 Active Directory (AD) 公开的协议。 如果在 Azure Active Directory 中创建了一个用户但该用户不受 AD 的支持（“托管”用户），则此方法将会失败。 在 AD 中创建的且并受 AAD 支持的用户（“联合”用户）可以受益于这种非交互式身份验证方法。 缓解措施：使用交互式身份验证。|

## <a name="javascript-errors"></a>JavaScript 错误

MSAL.js 提供 Error 对象用于抽象化和分类不同类型的常见错误，并提供一个接口用于访问具体的错误详细信息，例如用于适当处理错误的错误消息。

**Error 对象**

```javascript                                
export class AuthError extends Error {
    // This is a short code describing the error
    errorCode: string;
    // This is a descriptive string of the error,
    // and may also contain the mitigation strategy
    errorMessage: string;
    // Name of the error class
    this.name = "AuthError";
}
```                
通过扩展 error 类可以访问以下属性：
* **AuthError.message：** 此属性与 errorMessage 相同。
* **AuthError.stack：** 引发的错误的堆栈跟踪。 允许跟踪错误来源位置。

**错误类型**

可用的错误类型如下：

* *AuthError：* MSAL.js 库的基本 error 类，也可用于意外错误。

* *ClientAuthError：* 指示客户端身份验证问题的 Error 类。 来自库的大多数错误都是 ClientAuthError。 这些错误可能包括正在登录时调用 login 方法、用户取消登录等操作时出现的错误。

* *ClientConfigurationError：* 当给定的用户配置参数格式不当或缺失时，扩展发出请求之前所引发的 ClientAuthError 的 Error 类。

* *ServerError：* 用于表示身份验证服务器发送的错误字符串的 Error 类。 这些错误可能包括：请求格式或参数无效，或者阻止服务器对用户进行身份验证或授权的任何其他错误。

* *InteractionRequiredAuthError：* 扩展 ServerError，表示需要交互式调用的服务器错误的 Error 类。 如果用户必须与服务器交互才能提供凭据或许可身份验证/授权，则 `acquireTokenSilent` 会引发此错误。 错误代码包括“interaction_required”、“login_required”和“consent_required”。

若要使用重定向方法（`loginRedirect`、`acquireTokenRedirect`）在身份验证流中处理错误，需要按如下所示注册回调，重定向后，系统会使用 `handleRedirectCallback()` 结合成功或失败状态调用该回调：

```javascript
function authCallback(error, response) {
    //handle redirect response
}


var myMSALObj = new Msal.UserAgentApplication(msalConfig);

// Register Callbacks for redirect flow
myMSALObj.handleRedirectCallback(authCallback);

myMSALObj.acquireTokenRedirect(request);
```

弹出体验的方法（`loginPopup`、`acquireTokenPopup`）会返回约定，因此你可以使用约定模式（.then 和 .catch）来处理这些错误，如下所示：

```javascript
myMSALObj.acquireTokenPopup(request).then(
    function (response) {
        // success response
    }).catch(function (error) {
        console.log(error);
    });
```

### <a name="interaction-required-errors"></a>“需要交互”错误

需要 UI 交互时，会返回错误。 这意味着你已尝试使用非交互式方法来获取令牌（例如 `acquireTokenSilent`），但 MSAL 无法以无提示方式执行该操作。 可能的原因包括：

* 需要登录
* 需要许可
* 需要经历多重身份验证体验。

补救措施是调用 `acquireTokenPopup` 或 `acquireTokenRedirect` 等交互式方法：

```javascript
// Request for Access Token
myMSALObj.acquireTokenSilent(request).then(function (response) {
    // call API
}).catch( function (error) {
    // call acquireTokenPopup in case of acquireTokenSilent failure
    // due to consent or interaction required
    if (error.errorCode === "consent_required"
    || error.errorCode === "interaction_required"
    || error.errorCode === "login_required") {
        myMSALObj.acquireTokenPopup(request).then(
            function (response) {
                // call API
            }).catch(function (error) {
                console.log(error);
            });
    }
});
```

## <a name="conditional-access-and-claims-challenges"></a>条件访问和声明质询
以无提示方式获取令牌时，如果你尝试访问的 API 需要[条件访问声明质询](conditional-access-dev-guide.md)（例如 MFA 策略），则应用程序可能会收到错误。

处理此错误的模式是使用 MSAL 以交互方式获取令牌。 以交互方式获取令牌会提示用户，并使他们能够满足所需的条件访问策略。

在某些情况下调用需要条件访问的 API 时，API 返回的错误中可能会包含声明质询。 例如，如果条件访问策略要求使用托管设备 (Intune)，则错误将类似于 [AADSTS53000:需要管理你的设备才能访问此资源](reference-aadsts-error-codes.md)。 在这种情况下，可以在令牌获取调用中传递声明，使系统提示用户，以满足相应的策略。

### <a name="net"></a>.NET
从 MSAL.NET 调用需要条件访问的 API 时，应用程序需要处理声明质询异常。 此错误将显示为 [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet)，其中的 [Claims](/dotnet/api/microsoft.identity.client.msalserviceexception.claims?view=azure-dotnet) 属性不为空。

若要处理声明质询，需要使用 `PublicClientApplicationBuilder` 类的 `.WithClaim()` 方法。

### <a name="javascript"></a>JavaScript
使用 MSAL.js 以无提示方式获取令牌时（使用 `acquireTokenSilent`），如果你尝试访问的 API 需要[条件访问声明质询](conditional-access-dev-guide.md)（例如 MFA 策略），则应用程序可能会收到错误。

处理此错误的模式是发出交互式调用（例如 `acquireTokenPopup` 或 `acquireTokenRedirect`）以获取 MSAL.js 中的令牌，如以下示例所示：

```javascript
myMSALObj.acquireTokenSilent(accessTokenRequest).then(function (accessTokenResponse) {
    // call API
}).catch( function (error) {
    // call acquireTokenPopup in case of acquireTokenSilent failure
    myMSALObj.acquireTokenPopup(accessTokenRequest).then(
        function (accessTokenResponse) {
            // call API
        }).catch(function (error) {
            console.log(error);
        });
});
```

以交互方式获取令牌会提示用户，并使他们能够满足所需的条件访问策略。

调用需要条件访问的 API 时，API 返回的错误中可能会包含声明质询。 在这种情况下，可以在调用中以 `extraQueryParameters` 形式传递错误中返回的声明，使系统提示用户，以满足相应的策略：

```javascript
var request = {
    scopes: ["user.read"],
    extraQueryParameters: {claims: claims}
}

myMSALObj.acquireTokenPopup(request);
```

## <a name="retrying-after-errors-and-exceptions"></a>出现错误和异常后重试

### <a name="http-error-codes-500-600"></a>HTTP 错误代码 500-600
对于 HTTP 错误代码为 500-600 的错误，MSAL.NET 实现一个简单的重试一次机制。

### <a name="http-429"></a>HTTP 429
如果服务令牌服务器 (STS) 由于“请求过多”而变得过于繁忙，它会返回 HTTP 错误 429，并提示何时可以重试（Retry-After 响应字段），重试时间是以秒为单位的延迟或某个日期。

#### <a name="net"></a>.NET
[MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) 异常以 `namedHeaders` 属性的形式公开 `System.Net.Http.Headers.HttpResponseHeaders`。 因此，可以利用错误代码中的附加信息来提高应用程序的可靠性。 对于前面所述的场景，可以使用 `RetryAfterproperty`（类型为 `RetryConditionHeaderValue`）并计算重试时间。

下面是适用于守护程序应用程序的一个示例（使用客户端凭据流），但此示例也适用于任何令牌获取方法。

```csharp
do
{
 retry = false;
 TimeSpan? delay;
 try
 {
  result = await publicClientApplication.AcquireTokenForClient(scopes, account)
                                        .ExecuteAsync();
 }
 catch (MsalServiceException serviceException)
 {
  if (ex.ErrorCode == "temporarily_unavailable")
  {
   RetryConditionHeaderValue retryAfter = serviceException.Headers.RetryAfter;
   if (retryAfter.Delta.HasValue)
   {
    delay = retryAfter.Delta;
   }
   else if (retryAfter.Date.HasValue)
   {
    delay = retryAfter.Date.Value.Offset;
   }
  }
 }

    …
 if (delay.HasValue)
 {
  Thread.Sleep((int)delay.Value.TotalMilliseconds); // sleep or other
  retry = true;
 }
} while (retry);
```
