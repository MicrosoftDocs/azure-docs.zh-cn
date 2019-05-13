---
title: 初始化客户端应用程序 （适用于 JavaScript 的 Microsoft 身份验证库） |Azure
description: 了解有关初始化客户端应用程序使用 JavaScript (MSAL.js) Microsoft 身份验证库。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/12/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7990566ca9cd93e79b8356cfd15fda03a7469695
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138304"
---
# <a name="initialize-client-applications-using-msaljs"></a>初始化使用 MSAL.js 的客户端应用程序
本文介绍了用户代理应用程序的实例与 JavaScript (MSAL.js) 的初始化 Microsoft 身份验证库。 用户代理应用程序是在其中的客户端代码执行如 web 浏览器用户代理中的公共客户端应用程序的窗体。 这些客户端不存储机密，因为浏览器上下文是可公开访问。 若要了解有关客户端应用程序类型和应用程序配置选项的详细信息，请阅读[概述](msal-client-applications.md)。

## <a name="prerequisites"></a>必备组件
在之前初始化应用程序，首先需要向[将其注册到 Azure 门户](scenario-spa-app-registration.md)，以便您的应用程序可以与 Microsoft 标识平台集成。 注册后，可能需要以下信息 （这可在 Azure 门户中找到）：

- 客户端 ID （表示应用程序的 GUID 字符串）
- （命名实例） 的标识提供程序 URL 和应用程序的登录受众。 这两个参数统称为颁发机构。
- 如果你正在编写的业务线应用程序仅为你的组织 （也名为单租户应用程序） 租户的 ID。
- 对于 web 应用，你必须还将 redirectUri 设置标识提供程序返回到应用程序的安全令牌的位置。

## <a name="initializing-applications"></a>初始化应用程序

在纯 JavaScript/Typescript 应用程序中，可以按如下所示使用 MSAL.js。 通过实例化初始化 MSAL 身份验证上下文`UserAgentApplication`使用配置对象。 最小所需的配置，以初始化 MSAL.js 是应从应用程序注册门户获取应用程序的 clientID。

为流重定向使用的身份验证方法 (`loginRedirect`并`acquireTokenRedirect`)，将需要显式注册成功或错误通过回调`handleRedirectCallback()`方法。 这需要，因为重定向流不会返回承诺的弹出窗口体验的方法一样。

```javascript
// Configuration object constructed
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    }
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

// (optional when using redirect methods) register redirect call back for Success or Error
myMSALObj.handleRedirectCallback(authCallback);
```

MSAL.js 设计为具有单个实例和配置`UserAgentApplication`来表示单一身份验证上下文。 因为它们会导致冲突的缓存条目和行为，并在浏览器中不建议使用多个实例。

## <a name="configuration-options"></a>配置选项

MSAL.js 具有的配置，如下所示的对象提供可用于创建实例的可配置选项的分组`UserAgentApplication`。

```javascript
type storage = "localStorage" | "sessionStorage";

// Protocol Support
export type AuthOptions = {
    clientId: string;
    authority?: string;
    validateAuthority?: boolean;
    redirectUri?: string | (() => string);
    postLogoutRedirectUri?: string | (() => string);
    navigateToLoginRequestUrl?: boolean;
};

// Cache Support
export type CacheOptions = {
    cacheLocation?: CacheLocation;
    storeAuthStateInCookie?: boolean;
};

// Library support
export type SystemOptions = {
    logger?: Logger;
    loadFrameTimeout?: number;
    tokenRenewalOffsetSeconds?: number;
};

// Developer App Environment Support
export type FrameworkOptions = {
    isAngular?: boolean;
    unprotectedResources?: Array<string>;
    protectedResourceMap?: Map<string, Array<string>>;
};

// Configuration Object
export type Configuration = {
    auth: AuthOptions,
    cache?: CacheOptions,
    system?: SystemOptions,
    framework?: FrameworkOptions
};
```

下面是总组的当前支持的配置对象中的可配置选项：

- **clientID**:必需。 你的应用程序的 clientID，你应从获取此应用程序注册门户。

- **颁发机构**:可选。 MSAL 可以请求令牌从一个目录，该值指示一个 URL。 默认值为 `https://login.microsoftonline.com/common`。
    * 在 Azure AD 中，它是窗体 https://&lt;实例&gt;/&lt;受众&gt;，其中&lt;实例&gt;是标识提供者域 (例如， `https://login.microsoftonline.com`) 和&lt;受众&gt;是代表登录对象的标识符。 这可以是以下值：
        * `https://login.microsoftonline.com/<tenant>`-租户是关联到租户，例如 contoso.onmicrosoft.com，或 GUID 表示的域`TenantID`属性仅用于特定组织的用户登录的目录。
        * `https://login.microsoftonline.com/common`-用于在用户的工作和学校帐户或 Microsoft 个人帐户登录。
        * `https://login.microsoftonline.com/organizations/`-用于在使用用户的工作和学校帐户登录。
        * `https://login.microsoftonline.com/consumers/` -用于在具有仅个人 Microsoft 帐户 (live) 的用户登录。
    * 在 Azure AD B2C 中，它是窗体的`https://<instance>/tfp/<tenant>/<policyName>/`，其中实例是 Azure AD B2C 域中，租户是 Azure AD B2C 租户的名称，策略名称是要应用的 B2C 策略的名称。


- **validateAuthority**:可选。  验证令牌的颁发者。 默认为 `true`。 对于 B2C 应用程序，由于颁发机构值已知，并且可以是每个策略，不同的颁发机构验证不起作用，必须设置为`false`。

- **redirectUri**:可选。  应用的重定向 URI，应用可通过此 URI 发送和接收身份验证响应。 其必须与门户中注册的其中一个重定向 URI 完全匹配，否则必须经过 URL 编码。 默认为 `window.location.href`。

- **postLogoutRedirectUri**:可选。  将重定向到用户`postLogoutRedirectUri`后注销。默认为 `redirectUri`。

- **navigateToLoginRequestUrl**:可选。 若要关闭默认导航后登录名启动页的功能。 默认值为 true。 这仅用于重定向流。

- **cacheLocation**:可选。  将浏览器存储设置为`localStorage`或`sessionStorage`。 默认为 `sessionStorage`。

- **storeAuthStateInCookie**:可选。  此标志中引入 MSAL.js v0.2.2 作为修补程序来[身份验证循环问题](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#1-issues-due-to-security-zones)上 Microsoft Internet Explorer 和 Microsoft Edge。 启用标志`storeAuthStateInCookie`为 true，则利用此修复。 当启用此功能时，MSAL.js 将存储的浏览器 cookie 中的身份验证流数据流验证所需的身份验证请求状态。 默认情况下，此标志设置为`false`。

- **记录器**:可选。  具有可以由开发人员来使用和自定义方式发布日志提供的回调实例的记录器对象。 将记录器对象传递的详细信息，请参阅[msal.js 的日志记录](msal-logging.md)。

- **loadFrameTimeout**:可选。  超时前应考虑从 Azure AD 的令牌的续订响应的不活动的毫秒数。默认值为 6 秒。

- **tokenRenewalOffsetSeconds**:可选。 这将续订的令牌过期前所需的偏移量的窗口设置的毫秒数。 默认值为 300 毫秒。

这些是仅适用于向下传递从 MSAL Angular 的包装器库：
- **unprotectedResources**:可选。  是不受保护的资源的 Uri 的数组。 MSAL 将附加到具有这些 URI 的传出请求的令牌。 默认为 `null`。

- **protectedResourceMap**:可选。  这映射到 MSAL 用于自动附加在 web API 调用中的访问令牌的作用域的资源。 单个访问令牌获取的资源。 以便您可以按如下所示映射特定的资源路径: {"https://graph.microsoft.com/v1.0/me"，"user.read"}，或作为资源的应用 URL: {"https://graph.microsoft.com/"，["user.read"，"mail.send"]}。 这是必需的 CORS 调用。 默认为 `null`。
