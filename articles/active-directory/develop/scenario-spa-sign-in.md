---
title: 单页面应用程序 （登录）-Microsoft 标识平台
description: 了解如何生成单页面应用程序 （登录）
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/06/2019
ms.author: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: fc9c46ae28960387e6f8efc1ade20afa1c77ef55
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138798"
---
# <a name="single-page-application---sign-in"></a>单页面应用程序的登录

了解如何将登录添加到单页面应用程序的代码。

可以获取令牌来访问你的应用程序中的 Api 之前，将需要身份验证的用户上下文。 MSAL.js 中应用程序在两种方法，可以在用户进行登录：

* [使用弹出窗口登录](#sign-in-with-a-pop-up-window)使用`loginPopup`方法
* [登录重定向](#sign-in-with-redirect)使用`loginRedirect`方法

可以选择性地传递需要用户在登录时接受的 Api 的作用域。

> [!NOTE]
> 如果应用程序已有权访问的身份验证的用户上下文或 id 令牌，可以跳过登录步骤，直接获取的令牌。 有关更多详细信息，请参阅[msal.js 登录名没有 sso](msal-js-sso.md#sso-without-msaljs-login)。

## <a name="choosing-between-a-pop-up-or-redirect-experience"></a>弹出窗口或重定向体验之间进行选择

不能在应用程序中使用弹出窗口和重定向方法的组合。 弹出窗口或重定向体验之间的选择取决于你的应用程序流。

* 如果不希望用户在身份验证期间通过导航离开您的主应用程序页，建议使用弹出窗口的方法。 由于身份验证重定向发生在弹出窗口中，将保留主应用程序的状态。

* 在某些情况下，您可能需要使用重定向方法。 如果应用程序的用户有浏览器约束或策略在禁用弹出窗口，您可以使用重定向方法。 对 Internet Explorer 浏览器中使用的重定向方法，因为在某些[已知问题与 Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser)时处理的弹出窗口。

## <a name="sign-in-with-a-pop-up-window"></a>使用弹出窗口登录

### <a name="javascript"></a>JavaScript

```javascript
const loginRequest = {
    scopes: ["user.read", "user.write"]
}

userAgentApplication.loginPopup(loginRequest).then(function (loginResponse) {
    //login success
    let idToken = loginResponse.idToken;
}).catch(function (error) {
    //login failure
    console.log(error);
});
```

### <a name="angular"></a>Angular

MSAL Angular 包装器允许您只需添加受保护应用程序中的特定路由`MsalGuard`到路由定义。 此防护将调用方法来访问该路由时登录。

```javascript
// In app.routes.ts
{ path: 'product', component: ProductComponent, canActivate : [MsalGuard],
    children: [
      { path: 'detail/:id', component: ProductDetailComponent  }
    ]
   },
  { path: 'myProfile' ,component: MsGraphComponent, canActivate : [MsalGuard] },
```

弹出窗口体验，启用`popUp`config 选项。 你还可以传递需要同意的情况下，如下所示的作用域：

```javascript
//In app.module.ts
@NgModule({
  imports: [ MsalModule.forRoot({
                clientID: 'your_app_id',
                popUp: true,
                consentScopes: ["user.read", "user.write"]
            })]
         })
```

## <a name="sign-in-with-redirect"></a>使用重定向登录

### <a name="javascript"></a>JavaScript

重定向方法不返回由于导航从主应用程序的承诺。 若要处理并访问返回的令牌，将需要调用重定向方法之前注册 success 和 error 回调。

```javascript
function authCallback(error, response) {
    //handle redirect response
}

userAgentApplication.handleRedirectCallback(authCallback);

const loginRequest = {
    scopes: ["user.read", "user.write"]
}

userAgentApplication.loginRedirect(loginRequest);
```

### <a name="angular"></a>Angular

下面的代码是上文所述下使用弹出窗口部分登录相同的。 默认流是重定向。

> [!NOTE]
> ID 令牌不包含已获许可的作用域，并且只代表经过身份验证的用户。 这将在下一步中获取访问令牌中返回的已获许可的作用域。

## <a name="sign-out"></a>注销

MSAL 库提供了`logout`将清除浏览器存储中的缓存，并将注销请求发送到 Azure AD 的方法。 注销之后, 它将重定向回应用程序起始页默认情况下。

你可以配置它应重定向到号后出通过设置的 URI `postLogoutRedirectUri`。 此 URI 也应注册为应用程序注册中的注销 URI。

### <a name="javascript"></a>JavaScript

```javascript
const config = {

    auth: {
        clientID: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const userAgentApplication = new UserAgentApplication(config);
userAgentApplication.logout();

```

### <a name="angular"></a>Angular

```javascript
//In app.module.ts
@NgModule({
  imports: [ MsalModule.forRoot({
                clientID: 'your_app_id',
                postLogoutRedirectUri: "your_app_logout_redirect_uri"
            })]
         })

// In app.component.ts
this.authService.logout();
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [获取应用程序令牌](scenario-spa-acquire-token.md)
