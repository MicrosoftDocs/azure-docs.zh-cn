---
title: 单页面应用程序 （获取令牌以调用的 API）-Microsoft 标识平台
description: 了解如何生成单页面应用程序 （获取令牌以调用的 API）
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: f4c842db8a0874d3619e0dc59b90aa12226cb984
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138816"
---
# <a name="single-page-application---acquire-a-token-to-call-an-api"></a>单页面应用程序-获取令牌以调用的 API

用于获取令牌，以先尝试无提示令牌请求使用与 MSAL.js Api 为模式`acquireTokenSilent`方法。 调用此方法时，库将首先检查是否存在有效的令牌，并将其返回的浏览器存储中的缓存。 如果在缓存中没有有效的令牌，它将发送无提示令牌请求到 Azure Active Directory (Azure AD) 从隐藏的 iframe。 此方法还使库能够以续订令牌。 有关单一登录会话和 Azure AD 中的令牌生存期值的更多详细信息，请参阅[令牌生存期](active-directory-configurable-token-lifetimes.md)。

对 Azure AD 的无提示令牌请求可能会失败的一些原因，例如过期的 Azure AD 会话或更改密码。 在这种情况下，您可以调用 （它将提示用户） 的交互式方法之一获取的令牌。

* [获取与一个弹出窗口的令牌](#acquire-token-with-a-pop-up-window)使用 `acquireTokenPopup`
* [获取令牌的重定向](#acquire-token-with-redirect)使用 `acquireTokenRedirect`

**弹出窗口或重定向体验之间进行选择**

 不能在应用程序中使用弹出窗口和重定向方法的组合。 弹出窗口或重定向体验之间的选择取决于你的应用程序流。

* 如果不希望用户在身份验证期间通过导航离开您的主应用程序页，建议使用弹出窗口的方法。 由于身份验证重定向发生在弹出窗口中，将保留主应用程序的状态。

* 在某些情况下，您可能需要使用重定向方法。 如果应用程序的用户有浏览器约束或策略在禁用弹出窗口，您可以使用重定向方法。 它还建议对 Internet Explorer 浏览器中使用的重定向方法，因为在某些[已知问题与 Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser)时处理的弹出窗口。

您可以设置你想要生成访问令牌请求时包含的访问令牌的 API 作用域。 请注意，所有请求的作用域不应授予访问令牌中，并且取决于用户的同意。

## <a name="acquire-token-with-a-pop-up-window"></a>获取与一个弹出窗口的令牌

### <a name="javascript"></a>JavaScript

上面的模式，使用弹出窗口体验的方法：

```javascript
const accessTokenRequest = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, send an interactive request.
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // Acquire token interactive success
        }).catch(function(error) {
            // Acquire token interactive failure
            console.log(error);
        });
    }
    console.log(error);
});
```

### <a name="angular"></a>Angular

MSAL Angular 包装器提供了添加 HTTP 侦听器的便利`MsalInterceptor`这将自动以无提示方式获取访问令牌，并将它们附加到 Api 收到的 HTTP 请求。

你可以指定在 api 范围`protectedResourceMap`MsalInterceptor 将请求时自动获取令牌的配置选项。

```javascript
//In app.module.ts
@NgModule({
  imports: [ MsalModule.forRoot({
                clientID: 'your_app_id',
                protectedResourceMap: {"https://graph.microsoft.com/v1.0/me", ["user.read", "mail.send"]}
            })]
         })

providers: [ ProductService, {
        provide: HTTP_INTERCEPTORS,
        useClass: MsalInterceptor,
        multi: true
    }
   ],
```

成功与失败的无提示令牌获取，MSAL Angular 提供您可以订阅的回调。 它是还需要记住取消订阅。

```javascript
// In app.component.ts
 ngOnInit() {
    this.subscription=  this.broadcastService.subscribe("msal:acquireTokenFailure", (payload) => {
    });
}

ngOnDestroy() {
   this.broadcastService.getMSALSubject().next(1);
   if(this.subscription) {
     this.subscription.unsubscribe();
   }
 }
```

或者，你可以显式获取令牌核心 MSAL.js 库中所述使用获取令牌方法。

## <a name="acquire-token-with-redirect"></a>获取令牌的重定向

### <a name="javascript"></a>JavaScript

模式是为上述但若要以交互方式获取令牌的重定向方法所示。 请注意，你将需要注册重定向回叫，上文所述。

```javascript
function authCallback(error, response) {
    //handle redirect response
}

userAgentApplication.handleRedirectCallback(authCallback);

const accessTokenRequest: AuthenticationParameters = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, send an interactive request.
    console.log(error);
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenRedirect(accessTokenRequest);
    }
});
```

### <a name="angular"></a>Angular

这是与上面所述相同。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [调用 Web API](scenario-spa-call-api.md)
