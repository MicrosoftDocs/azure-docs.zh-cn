---
title: Microsoft 标识平台 JavaScript 快速入门 | Azure
description: 了解 JavaScript 应用程序如何才能通过 Microsoft 标识平台调用需要访问令牌的 API。
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/11/2019
ms.author: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 605206682cb70d430773cdbf9ff746eabf594103
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190849"
---
# <a name="quickstart-sign-in-users-and-acquire-an-access-token-from-a-javascript-single-page-application-spa"></a>快速入门：通过 JavaScript 单页应用程序 (SPA) 登录用户并获取访问令牌

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

本快速入门介绍如何使用代码示例，该示例演示了 JavaScript 单页应用程序 (SPA) 登录个人、工作和学校帐户并获取访问令牌以调用 Microsoft Graph API 或任何 Web API 的方式。

![显示本快速入门生成的示例应用的工作原理](media/quickstart-v2-javascript/javascriptspa-intro.svg)

## <a name="prerequisites"></a>先决条件

需要以下设置才能完成本快速入门：
* 若要通过 Node.js 服务器运行此项目，请执行以下操作：
    * 安装 [Node.js](https://nodejs.org/en/download/)
    * 安装 [Visual Studio Code](https://code.visualstudio.com/download)，以便编辑项目文件
* 若要将该项目作为 Visual Studio 解决方案运行，请安装 [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)。

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>注册并下载快速入门应用程序
> 可以使用两个选项来启动快速入门应用程序：
> * [快速][选项 1：注册并自动配置应用，然后下载代码示例](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [手动][选项 2：注册并手动配置应用程序和代码示例](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>选项 1：注册并自动配置应用，然后下载代码示例
>
> 1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
> 1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
> 1. 转到新的 [Azure 门户 - 应用注册](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs)窗格。
> 1. 输入应用程序的名称，然后单击“注册”。
> 1. 遵照说明下载内容，并一键式自动配置新应用程序。
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>选项 2：注册并手动配置应用程序和代码示例
>
> #### <a name="step-1-register-your-application"></a>步骤 1：注册应用程序
>
> 1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
> 1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
> 1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)页。
> 1. 选择“新注册”。
> 1. “注册应用程序”页显示后，请输入应用程序的名称。
> 1. 在“支持的帐户类型”下，选择“任何组织目录中的帐户和个人 Microsoft 帐户”。
> 1. 选择“重定向 URI”部分下的“Web”平台，并将值设置为 `http://localhost:30662/`。
> 1. 完成后，选择“注册”。  在应用的“概述”页上，记下“应用程序(客户端) ID”值。
> 1. 本快速入门要求启用[隐式授权流](v2-oauth2-implicit-grant-flow.md)。 在已注册的应用程序的左侧导航窗格中，选择“身份验证”。
> 1. 在“高级设置”中的“隐式授权”下，同时勾选“ID 令牌”和“访问令牌”复选框。 由于此应用需要将用户登录并调用 API，因此需要 ID 令牌和访问令牌。
> 1. 选择“保存”。

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>步骤 1：在 Azure 门户中配置应用程序
> 为使本快速入门的代码示例正常运行，需将重定向 URI 添加为 `http://localhost:30662/` 并启用“隐式授权”。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [为我进行这些更改]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已配置](media/quickstart-v2-javascript/green-check.png) 应用程序已使用这些属性进行配置。

#### <a name="step-2-download-the-project"></a>步骤 2：下载项目

可以根据开发环境选择这些选项之一。
* [下载核心项目文件](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip)，以便使用 Node.js 在 Web 服务器中运行。 若要打开这些文件，请使用 [Visual Studio Code](https://code.visualstudio.com/) 之类的编辑器。

* （可选）[下载 Visual Studio 项目](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip)，以便与 IIS 服务器一起运行。 将 zip 文件提取到本地文件夹（例如，**C:\Azure-Samples**）。



#### <a name="step-3-configure-your-javascript-app"></a>步骤 3：配置 JavaScript 应用

> [!div renderon="docs"]
> 在 JavaScriptSPA 文件夹下，编辑 `index.html` 并在 `msalConfig` 下设置 `clientID` 和 `authority` 值。

> [!div class="sxs-lookup" renderon="portal"]
> 在 JavaScriptSPA 文件夹下，编辑 `index.html`，并将 `msalConfig` 替换为：

```javascript
var msalConfig = {
    auth: {
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Info_Here"
    },
    cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: true
    }
};

```
> [!div renderon="docs"]
>
> 其中：
> - `Enter_the_Application_Id_here` - 是已注册应用程序的**应用程序（客户端）ID**。
> - `Enter_the_Tenant_Info_Here` - 设置为以下选项之一：
>   - 如果应用程序支持“此组织目录中的帐户”，请将该值替换为**租户 ID** 或**租户名称**（例如 contoso.microsoft.com）
>   - 如果应用程序支持“任何组织目录中的帐户”，请将该值替换为 `organizations`
>   - 如果应用支持“任何组织目录中的帐户和个人 Microsoft 帐户”，请将此值替换为“`common`”。 若支持*仅限 Microsoft 个人帐户*，请将该值替换为 `consumers`。
>
> > [!TIP]
> > 若要查找“应用程序(客户端) ID”、“目录(租户) ID”和“支持的帐户类型”的值，请转到 Azure 门户中应用的“概述”页。
>

#### <a name="step-4-run-the-project"></a>步骤 4：运行项目

* 如果使用 [Node.js](https://nodejs.org/en/download/)：

    1. 通过项目的目录运行以下命令，以便启动服务器：

        ```batch
        npm install
        node server.js
        ```

    1. 打开 Web 浏览器并导航到 `http://localhost:30662/`。
    1. 单击“登录”按钮以开始登录，然后调用 Microsoft Graph API。


* 如果使用 [Visual Studio](https://visualstudio.microsoft.com/downloads/)，请确保选择项目解决方案，然后按 **F5** 来运行项目。

在浏览器加载应用程序后，单击“登录”。  首次登录时，系统会提示你同意允许应用程序访问你的个人资料并登录。 成功登录后，应该会看到自己的用户个人资料信息显示在页面上。

## <a name="more-information"></a>更多信息

### <a name="msaljs"></a>*msal.js*

MSAL 是一个库，用于用户登录和请求令牌，此类令牌用于访问受 Microsoft 标识平台保护的 API。 本快速入门的 *index.html* 包含对库的引用：

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.min.js"></script>
```
> [!TIP]
> 可以在 [MSAL.js 版本](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)下将上面的版本替换为最新发布的版本。


另外，如果已安装 Node，则可通过 npm 下载最新版本：

```batch
npm install msal
```

### <a name="msal-initialization"></a>MSAL 初始化

快速入门代码也演示了如何初始化库：

```javascript
var msalConfig = {
    auth: {
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Info_Here"
    },
    cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: true
    }
};

var myMSALObj = new Msal.UserAgentApplication(msalConfig);
```

> |其中  |  |
> |---------|---------|
> |`ClientId`     |Azure 门户中注册的应用程序的应用程序 ID|
> |`authority`    | （可选）如上面的配置部分所述，它是用于支持帐户类型的颁发机构 URL。 默认颁发机构为 `https://login.microsoftonline.com/common`。 |
> |`cacheLocation`  | （可选）这样可以针对身份验证状态设置浏览器存储。 默认为 sessionStorage。   |
> |`storeAuthStateInCookie`  | （可选）此库将存储身份验证请求状态，该状态是在浏览器 Cookie 中对身份验证流进行验证所需的。 这是针对 IE 和 Edge 浏览器设置的，目的是缓解某些[已知问题](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues)。 |

 请参阅 [wiki](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/MSAL-basics#configuration-options)，详细了解提供的可配置选项。

### <a name="sign-in-users"></a>用户登录

以下代码片段演示如何进行用户登录：

```javascript
var requestObj = {
    scopes: ["user.read"]
};

myMSALObj.loginPopup(requestObj).then(function (loginResponse) {
    //Login Success callback code here
}).catch(function (error) {
    console.log(error);
});
```

> |其中  |  |
> |---------|---------|
> | `scopes`   | （可选）包含在登录时为了获得用户同意而请求的作用域。 例如：`[ "user.read" ]` 适用于 Microsoft Graph，`[ "<Application ID URL>/scope" ]` 适用于自定义 Web API（即 `api://<Application ID>/access_as_user`）。 |

> [!TIP]
> 另外，你可能希望使用 `loginRedirect` 方法来将当前页面重定向到登录页面而非重定向到弹出窗口。

### <a name="request-tokens"></a>请求令牌

MSAL 有三种用于获取令牌的方法，即 `acquireTokenRedirect`、`acquireTokenPopup`、`acquireTokenSilent`

#### <a name="get-a-user-token-silently"></a>以无提示方式获取用户令牌

`acquireTokenSilent` 方法处理令牌获取和续订，无需进行任何用户交互。 首次执行 `loginRedirect` 或 `loginPopup` 方法后，通常使用 `acquireTokenSilent` 方法获取用于访问受保护资源的令牌，以便进行后续调用。 进行请求或续订令牌的调用时，以静默方式进行。

```javascript
var requestObj = {
    scopes: ["user.read"]
};

myMSALObj.acquireTokenSilent(requestObj).then(function (tokenResponse) {
    // Callback code here
    console.log(tokenResponse.accessToken);
}).catch(function (error) {
    console.log(error);
});
```

> |其中  |  |
> |---------|---------|
> | `scopes`   | 包含请求的需要在 API 的访问令牌中返回的作用域。 例如：`[ "user.read" ]` 适用于 Microsoft Graph，`[ "<Application ID URL>/scope" ]` 适用于自定义 Web API（即 `api://<Application ID>/access_as_user`）。|

#### <a name="get-a-user-token-interactively"></a>以交互方式获取用户令牌

有些情况下，需强制用户与 Microsoft 标识平台终结点交互。 例如：
* 由于密码已过期，用户可能需要重新输入凭据
* 应用程序正在请求访问用户需要同意的其他资源作用域
* 需要双重身份验证

建议用于大多数应用程序的常用模式是先调用 `acquireTokenSilent`，然后捕获异常，然后再调用 `acquireTokenPopup`（或 `acquireTokenRedirect`），以便启动交互式请求。

调用 `acquireTokenPopup` 将导致显示一个用于登录的弹出窗口（调用 `acquireTokenRedirect` 则会导致将用户重定向到 Microsoft 标识平台终结点），在这种情况下，用户需要进行交互，即确认凭据、同意所需资源或完成双重身份验证。

```javascript
var requestObj = {
    scopes: ["user.read"]
};

myMSALObj.acquireTokenPopup(requestObj).then(function (tokenResponse) {
    // Callback code here
    console.log(tokenResponse.accessToken);
}).catch(function (error) {
    console.log(error);
});
```
> [!NOTE]
> 当使用的浏览器是 Internet Explorer 时，本快速入门使用 `loginRedirect` 和 `acquireTokenRedirect` 方法，因为存在一个与处理 Internet Explorer 的弹出窗口相关的[已知问题](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues)。

## <a name="next-steps"></a>后续步骤

如需更详细的分步指南，以便了解如何生成适用于本快速入门的应用程序，请尝试下面的 JavaScript 教程。

### <a name="learn-the-steps-to-create-the-application-for-this-quickstart"></a>了解为本快速入门创建应用程序的步骤

> [!div class="nextstepaction"]
> [有关登录和调用 MS Graph 的教程](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-javascriptspa)

### <a name="browse-the-msal-repo-for-documentation-faq-issues-and-more"></a>浏览 MSAL 存储库中的文档、常见问题解答、问题等

> [!div class="nextstepaction"]
> [MSAL.js GitHub 存储库](https://github.com/AzureAD/microsoft-authentication-library-for-js)
