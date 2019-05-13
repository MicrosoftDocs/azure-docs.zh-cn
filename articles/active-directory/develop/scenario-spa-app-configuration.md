---
title: 单页面应用程序 （应用程序的代码配置）-Microsoft 标识平台
description: 了解如何生成单页面应用程序 （应用程序的代码配置）
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
ms.openlocfilehash: b71454fc553a0f81c26426a6a9588f15d5311e38
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406425"
---
# <a name="single-page-application---code-configuration"></a>单页面应用程序的代码配置

了解如何配置单页面应用程序 (SPA) 的代码。

## <a name="msal-libraries-supporting-implicit-flow"></a>MSAL 库支持隐式流

Microsoft 标识平台提供了 MSAL.js 库，用于支持隐式流使用行业建议的安全做法。  

支持隐式流的库是：

| MSAL 库 | 描述 |
|--------------|--------------|
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)  | 在使用 JavaScript 或 SPA 框架，如 Angular、 Vue.js、 React.js 等生成的任何客户端 web 应用中使用的纯 JavaScript 库。 |
| ![MSAL Angular](media/sample-v2-code/logo_angular.png) <br/> [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | 若要简化使用 Angular 框架构建的单页面应用程序中使用的核心 MSAL.js 库的包装器。 此库处于预览状态，具有[已知问题](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues?q=is%3Aopen+is%3Aissue+label%3Aangular)与特定 Angular 版本和浏览器。 |

## <a name="application-code-configuration"></a>应用程序代码配置

MSAL 库中的应用程序注册信息将作为库初始化过程中的配置。

### <a name="javascript"></a>JavaScript

```javascript
// Configuration object constructed.
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri" //defaults to application start page
    }
}

// create UserAgentApplication instance
const userAgentApplication = new UserAgentApplication(config);
```
有关可用的可配置选项的更多详细信息，请参阅[初始化应用程序与 MSAL.js](msal-js-initializing-client-applications.md)。

### <a name="angular"></a>Angular

```javascript
//In app.module.ts
@NgModule({
  imports: [ MsalModule.forRoot({
                clientId: 'your_app_id'
            })]
         })

  export class AppModule { }
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [登录和注销](scenario-spa-sign-in.md)
