---
title: 教程 - 在单页应用程序中启用身份验证 - Azure Active Directory B2C | Microsoft Docs
description: 有关如何使用 Azure Active Directory B2C 为单页应用程序 (JavaScript) 提供用户登录功能的教程。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.author: davidmu
ms.date: 02/04/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 3773efdbe47c045dccf74ace3e1b53936c9400fe
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190307"
---
# <a name="tutorial-enable-authentication-in-a-single-page-application-using-azure-active-directory-b2c"></a>教程：使用 Azure Active Directory B2C 在单页应用程序中启用身份验证

本教程展示了如何在单页应用程序 (SPA) 中使用 Azure Active Directory (Azure AD) B2C 执行用户登录和注册。 应用程序可以使用 Azure AD B2C 通过开放式标准协议对社交帐户、企业帐户和 Azure Active Directory 帐户进行身份验证。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 在 Azure AD B2C 中更新此应用程序
> * 将示例配置为使用此应用程序
> * 使用用户流进行登录

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

* [创建用户流](tutorial-create-user-flows.md)，以便在应用程序中启用用户体验。 
* 安装带有 ASP.NET 和 Web 开发工作负荷的 [Visual Studio 2019](https://www.visualstudio.com/downloads/)。
* 安装 [.NET Core 2.0.0 SDK](https://www.microsoft.com/net/core) 或更高版本
* 安装 [Node.js](https://nodejs.org/en/download/)

## <a name="update-the-application"></a>更新应用程序

在按照先决条件完成的教程中，已在 Azure AD B2C 中添加了 Web 应用程序。 若要与教程中的示例通信，需添加一个指向 Azure AD B2C 中的应用程序的重定向 URI。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 请确保使用包含 Azure AD B2C 租户的目录，方法是单击顶部菜单中的“目录和订阅筛选器”，然后选择包含租户的目录。
3. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“Azure AD B2C”。
4. 选择“应用程序”，然后选择“webapp1”应用程序。
5. 在“回复 URL”下添加 `http://localhost:6420`。
6. 选择“保存”。
7. 在属性页上，记录你在配置 Web 应用程序时要使用的应用程序 ID。
8. 依次选择“密钥”、“生成密钥”、“保存”。 记录你在配置 Web 应用程序时要使用的密钥。

## <a name="configure-the-sample"></a>配置示例

在本教程中，将配置一个可从 GitHub 下载的示例。 该示例演示单页应用程序可以如何使用 Azure AD B2C 执行用户注册和登录，以及如何调用受保护的 Web API。

从 GitHub [下载 zip 文件](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip)或克隆该示例。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
```

若要更改设置，请执行以下操作：

1. 在示例中打开 `index.html` 文件。
2. 使用以前记录的应用程序 ID 和密钥配置示例。 更改以下代码行，将值替换为你的目录和 API 的名称：

    ```javascript
    // The current application coordinates were pre-registered in a B2C directory.
    var applicationConfig = {
        clientID: '<Application ID>',
        authority: "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_signupsignin1",
        b2cScopes: ["https://contoso.onmicrosoft.com/demoapi/demo.read"],
        webApi: 'https://contosohello.azurewebsites.net/hello',
    };
    ```

    在本教程中使用的用户流的名称为 **B2C_1_signupsignin1**。 如果你使用了不同的用户流名称，请在 `authority` 值中使用你的用户流名称。

## <a name="run-the-sample"></a>运行示例

1. 启动 Node.js 命令提示符。
2. 切换到包含 Node.js 示例的目录。 例如 `cd c:\active-directory-b2c-javascript-msal-singlepageapp`
3. 运行以下命令：

    ```
    npm install && npm update
    node server.js
    ```

    控制台窗口将显示承载着该应用的位置的端口号。
    
    ```
    Listening on port 6420...
    ```

4. 使用浏览器导航到地址 `http://localhost:6420` 来查看该应用程序。

示例支持注册、登录、配置文件编辑和密码重置。 本教程重点介绍用户如何使用电子邮件地址注册。

### <a name="sign-up-using-an-email-address"></a>使用电子邮件地址注册

1. 单击“登录”，注册为应用程序用户。 这将使用在前面的步骤中定义的 **B2C_1_signupsignin1** 用户流。
2. Azure AD B2C 会显示带注册链接的登录页面。 由于你还没有帐户，因此请单击“立即注册”链接。 
3. 注册工作流会显示一个页面，用于收集用户的标识并通过电子邮件地址对其进行验证。 注册工作流还收集用户的密码和请求的属性（在用户流中定义）。

    请使用有效的电子邮件地址，并使用验证码进行验证。 设置密码。 输入请求的属性的值。 

    ![注册工作流](media/active-directory-b2c-tutorials-desktop-app/sign-up-workflow.png)

4. 单击“创建”，在 Azure AD B2C 目录中创建本地帐户。

现在，用户可以使用电子邮件地址登录并使用 SPA 应用了。

> [!NOTE]
> 登录后，应用会显示“权限不足”错误。 收到此错误是因为你正在尝试从演示目录访问资源。 因为你的访问令牌仅对你的 Azure AD 目录有效，所以该 API 调用未经授权。 请继续学习下一教程，了解如何为目录创建受保护的 Web API。

## <a name="next-steps"></a>后续步骤

本文介绍了如何执行以下操作：

> [!div class="checklist"]
> * 在 Azure AD B2C 中更新此应用程序
> * 将示例配置为使用此应用程序
> * 使用用户流进行登录

> [!div class="nextstepaction"]
> [教程：从单页应用使用 Azure Active Directory B2C 授予对 ASP.NET Core Web API 的访问权限](active-directory-b2c-tutorials-spa-webapi.md)
