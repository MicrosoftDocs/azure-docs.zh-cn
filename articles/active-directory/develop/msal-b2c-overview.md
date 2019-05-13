---
title: 了解如何使用 Microsoft 身份验证库 (MSAL) 与 Azure AD B2C 进行集成
description: 借助 Microsoft 身份验证库 (MSAL)，应用程序开发人员可以集成 Azure AD B2C，并获取令牌来调用受保护的 Web API。 这些 Web API 可以是 Microsoft Graph、其他 Microsoft API、第三方 Web API 或你自己的 Web API。
services: active-directory
documentationcenter: dev-center-name
author: negoe
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/04/2019
ms.author: negoe
ms.reviewer: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 87bd8834f840c2246bf3adc1d1f9cd9b8f635915
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191022"
---
# <a name="integrate-microsoft-authentication-library-msal-with-azure-active-directory-b2c"></a>将 Microsoft 身份验证库 (MSAL) 与 Azure Active Directory B2C 集成

借助 Microsoft 身份验证库 (MSAL)，应用程序开发人员可以使用 [Azure Active Directory (Azure AD) B2C](https://docs.microsoft.com/azure/active-directory-b2c/) 通过社交和本地标识对用户进行身份验证。 Azure Active Directory (Azure AD) B2C 是一项标识管理服务，用于自定义和控制客户在使用应用程序时的注册、登录和管理配置文件的方式。

通过 Azure Active Directory (Azure AD) B2C，你还可以为应用程序的用户界面 (UI) 提供标志并进行自定义，以便为你的客户提供无缝体验。

本教程展示了如何使用 Microsoft 身份验证库 (MSAL) 与 Azure Active Directory (Azure AD) B2C 进行集成。


## <a name="prerequisites"></a>先决条件

如果尚未创建自己的 [Azure AD B2C 租户](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant)，请立即创建一个。 可以使用现有的 Azure AD B2C 租户。 

## <a name="javascript"></a>Javascript

下面的步骤演示了单页应用程序可以如何使用 Azure AD B2C 执行用户注册和登录，以及如何调用受保护的 Web API。

### <a name="step-1-register-your-application"></a>步骤 1：注册应用程序

若要实现身份验证，首先需要注册你的应用程序。 若要注册应用，请遵循[注册应用程序](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp#step-4-register-your-own-web-application-with-azure-ad-b2c)来执行详细步骤。

### <a name="steps-2-download-applications"></a>步骤 2：下载应用程序

从 GitHub 下载 zip 文件或克隆该示例。
>git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git

### <a name="steps-3-authentication"></a>步骤 3：身份验证

1. 打开示例中的 index.html 文件。

2. 在注册应用程序时，使用以前记录的应用程序 ID 和密钥配置示例。 更改以下代码行，将值替换为你的目录和 API 的名称：

```javascript
// The current application coordinates were pre-registered in a B2C directory.

const msalConfig = {
    auth:{
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.com/tfp/<your-tenant-name>.onmicrosoft.com/<your-sign-in-sign-up-policy>",
        b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/hello/demo.read"],
        webApi: 'http://localhost:5000/hello',
  };

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(msalConfig);
```

本教程中使用的[用户流](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-policies)的名称为 B2C_1_signupsignin1。 如果你使用了不同的用户流名称，请在 authority 值中使用你的用户流名称。


### <a name="configure-application-to-use-b2clogincom"></a>将应用程序配置为使用 `b2clogin.com`

在你的 Azure Active Directory (Azure AD) B2C 应用程序中设置用于注册和登录的标识提供者时，你可以使用 `b2clogin.com` 代替 `login.microsoftonline.com` 作为重定向 URL。

**`b2clogin.com`** 即 
`https://your-tenant-name.b2clogin.com/your-tenant-guid` 用于以下项：

- Microsoft 服务在 cookie 标头中使用的空间就会减少。
- URL 不再包括对 Microsoft 的引用。 例如，你的 Azure AD B2C 应用程序可能会引用 login.microsoftonline.com


 若要使用“b2clogin.com”，你需要更新应用程序的配置。  

1. 更新 ValidateAuthority：将 **validateAuthority** 属性设置为 `false`。 当 **validateAuthority** 设置为 false 时，允许重定向到 b2clogin.com。

以下示例展示了如何设置此属性：
```javascript
// The current application coordinates were pre-registered in a B2C directory.

const msalConfig = {
    auth:{
        clientId: "Enter_the_Application_Id_here",
        authority: "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_signupsignin1",
        b2cScopes: ["https://contoso.onmicrosoft.com/demoapi/demo.read"],
        webApi: 'https://contosohello.azurewebsites.net/hello',
        validateAuthority: false;

};
// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(msalConfig);
```

> [!NOTE]
> 你的 Azure AD B2C 应用程序可能在多个地方引用了 login.microsoftonline.com，例如用户流引用和令牌终结点。 请确保授权终结点、令牌终结点和证书颁发者都已更新为使用 your-tenant-name.b2clogin.com。

请参考有关如何使用 JavaScript 的 Microsoft 身份验证库预览版 (msal.js) 的此 [MSAL JS 示例](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp#single-page-application-built-on-msaljs-with-azure-ad-b2c)来获取访问令牌并调用受 Azure AD B2C 保护的 API。

## <a name="next-steps"></a>后续步骤

了解有关以下方面的详细信息：

- [自定义策略](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview-custom)
- [用户界面自定义](https://docs.microsoft.com/azure/active-directory-b2c/customize-ui-overview)