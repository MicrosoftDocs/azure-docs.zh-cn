---
title: 单一登录 （适用于 JavaScript 的 Microsoft 身份验证库） |Azure
description: 了解如何构建使用 JavaScript (MSAL.js) Microsoft 身份验证库的单一登录体验。
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a0fb0731b7ac46210294e3766b33bd6a239dcc2
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075845"
---
# <a name="single-sign-on-with-msaljs"></a>MSAL.js 与单一登录

单一登录 (SSO)，用户可以输入一次其凭据登录并建立会话而无需重新进行身份验证可重复使用跨多个应用程序。 这为用户提供无缝的体验，并减少重复提示你输入凭据。

Azure AD 通过用户进行身份验证的第一次时设置的会话 cookie 提供对应用程序的 SSO 功能。 MSAL.js 库允许应用程序可以利用这几种方式。

## <a name="sso-between-browser-tabs"></a>浏览器选项卡之间的 SSO

你的应用程序在多个选项卡中打开后，在一个选项卡上的用户首次登录，用户还登录的其他选项卡不会提示输入。 MSAL.js 缓存在浏览器中用户的 ID 令牌`localStorage`并将登录用户的其他打开的选项卡上的应用程序。

默认情况下，使用 MSAL.js`sessionStorage`它不允许会话选项卡之间共享。 若要获取选项卡之间的 SSO，请务必设置`cacheLocation`中为 MSAL.js `localStorage` ，如下所示。

```javascript
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    },
    cache: {
        cacheLocation: 'localStorage'
    }
}

const myMSALObj = new UserAgentApplication(config);
```

## <a name="sso-between-apps"></a>应用程序之间的 SSO

当用户身份验证时，在浏览器中的 Azure AD 域设置的会话 cookie。 MSAL.js 依赖于此会话 cookie 以不同的应用程序之间的用户提供 SSO。 MSAL.js 还将缓存的 ID 令牌和访问令牌的每个应用程序域的浏览器存储中的用户。 因此的 SSO 行为发生变化的不同用例：  

### <a name="applications-on-the-same-domain"></a>在同一个域上的应用程序

时的应用程序位于同一个域上，用户可以一次登录到应用，然后进行身份验证向无提示的其他应用。 MSAL.js 利用域上的用户提供 SSO 缓存的令牌。

### <a name="applications-on-different-domain"></a>另一域上的应用程序

MSAL.js 域 B 中时应用程序托管在不同的域，无法访问域 A 上缓存的令牌

这意味着，当用户登录域，导航到域 B 中的应用程序，它们将被重定向或与 Azure AD 页出现提示。 由于 Azure AD 仍具有用户会话 cookie，它将在用户登录，它们将不需要重新输入凭据。 如果用户在会话与 Azure AD 中具有多个用户帐户，则将提示用户选择相关的帐户登录时使用。

### <a name="automatically-select-account-on-azure-ad"></a>自动在 Azure AD 上选择帐户

在某些情况下，应用程序有权访问用户的身份验证上下文，并希望避免当多个帐户登录 Azure AD 帐户选择提示。  这可以完成多种不同的方式：

**使用会话 ID (SID)**

会话 ID 是[的可选声明](active-directory-optional-claims.md)，可以配置在 ID 令牌中。 此声明允许应用程序标识用户的 Azure AD 会话独立于用户的帐户名或用户名。 可以将 SID 传入的请求参数`acquireTokenSilent`调用。 这样，Azure AD，以绕过帐户选择。 SID 已绑定到会话 cookie，而且将不会跨浏览器上下文。

```javascript
var request = {
    scopes: ["user.read"],
    sid: sid
}

userAgentApplication.acquireTokenSilent(request).then(function(response) {
        const token = response.accessToken
    }
).catch(function (error) {  
        //handle error
});
```

> [!Note]
> SID 可以仅用于无提示身份验证请求所做的`acquireTokenSilent`MSAL.js 中调用。
您可以找到应用程序清单中配置可选声明的步骤[此处](active-directory-optional-claims.md)。

**使用登录提示**

如果没有 SID 声明配置或需要绕过交互式身份验证的调用中的帐户选择提示符下，就可以做到，从而`login_hint`中的请求参数和可选`domain_hint`作为`extraQueryParameters`MSAL.js 中交互式方法 (`loginPopup`， `loginRedirect`，`acquireTokenPopup`和`acquireTokenRedirect`)。 例如：

```javascript
var request = {
    scopes: ["user.read"],
    loginHint: preferred_username,
    extraQueryParameters: {domain_hint: 'organizations'}
}

userAgentApplication.loginRedirect(request);
```

您可以通过读取用户的 ID 令牌中返回的声明来获取 login_hint 和 domain_hint 的值。

* **loginHint**应设置为`preferred_username`ID 令牌中声明。

* **domain_hint**仅在需要时即使用 /common 来传递颁发机构。 域提示取决于租户 ID(tid)。  如果`tid`ID 令牌中的声明是`9188040d-6c67-4c5b-b112-36a304b66dad`是使用者。 否则，它是组织。

读取[此处](v2-oauth2-implicit-grant-flow.md)的登录提示和域提示的值的详细信息。

> [!Note]
> 不能将 SID 和 login_hint 传递一次。 这将导致错误响应。

## <a name="sso-without-msaljs-login"></a>不带 MSAL.js 登录名的 SSO

根据设计，MSAL.js 需要调用 login 方法时获取的 Api 的令牌之前建立用户上下文。 由于登录方法是交互式的用户会看到一条提示。

某些情况下，在其中应用程序有权访问经过身份验证的用户的上下文或通过身份验证 ID 令牌中另一个应用程序启动，并且想要利用 SSO 来获取的令牌，而无需通过 MSAL.js 的第一个签名。

此示例是：用户登录到承载作为外接程序或插件运行的其他 JavaScript 应用程序的父 web 应用程序。

在此方案中的 SSO 体验可以按如下所示：

传递`sid`如果可用 (或`login_hint`和 （可选） `domain_hint`) 作为请求参数到 MSAL.js`acquireTokenSilent`调用，如下所示：

```javascript
var request = {
    scopes: ["user.read"],
    loginHint: preferred_username,
    extraQueryParameters: {domain_hint: 'organizations'}
}

userAgentApplication.acquireTokenSilent(request).then(function(response) {
        const token = response.accessToken
    }
).catch(function (error) {  
        //handle error
});
```

## <a name="sso-in-adaljs-to-msaljs-update"></a>在 ADAL.js 中 MSAL.js 更新到的 SSO

MSAL.js 带来了使用 ADAL.js 的 Azure AD 身份验证方案的功能奇偶校验。 若要轻松迁移从 ADAL.js MSAL.js 并避免提示用户重新登录，库会读取在 ADAL.js 缓存中，表示用户的会话的 ID 令牌和无缝登录 MSAL.js 中的用户。  

若要利用单一登录 (SSO) 行为，ADAL.js 从更新时，将需要确保使用的库`localStorage`用于缓存令牌。 设置`cacheLocation`到`localStorage`MSAL.js 和 ADAL.js 在初始化，如下所示的配置中：


```javascript
// In ADAL.js
window.config = {
   clientId: 'g075edef-0efa-453b-997b-de1337c29185',
   cacheLocation: 'localStorage'
};

var authContext = new AuthenticationContext(config);


// In latest MSAL.js version
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    },
    cache: {
        cacheLocation: 'localStorage'
    }
}

const myMSALObj = new UserAgentApplication(config);
```

此配置后，将能够读取在 ADAL.js 中经过身份验证的用户的缓存的状态并使用它来提供 SSO 中 MSAL.js MSAL.js。

## <a name="next-steps"></a>后续步骤

详细了解如何[单一登录会话和令牌生存期](active-directory-configurable-token-lifetimes.md)Azure AD 中的值。
