---
title: include 文件
description: include 文件
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: nacanuma
ms.custom: include file
ms.openlocfilehash: 387adcdf8bdabf90bc1e691a7a8ec9ae0a8e90dc
ms.sourcegitcommit: abeefca6cd5ca01c3e0b281832212aceff08bf3e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "64993282"
---
## <a name="register-your-application"></a>注册应用程序

1. 登录到 [Azure 门户](https://portal.azure.com/)以注册应用程序。
1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)页。
1. “注册应用程序”页显示后，请输入应用程序的名称。
1. 在“支持的帐户类型”下，选择“任何组织目录中的帐户和个人 Microsoft 帐户”。
1. 在“重定向 URI”部分下选择“Web”平台，并根据 Web 服务器将值设置为应用程序的 URL。 有关如何在 Visual Studio 和 Node 中设置和获取重定向 URL 的说明，请参阅以下部分。
1. 完成后，选择“注册”。
1. 在应用的“概述”页上，记下“应用程序(客户端) ID”值。
1. 本快速入门要求启用[隐式授权流](../articles/active-directory/develop/v2-oauth2-implicit-grant-flow.md)。 在已注册的应用程序的左侧导航窗格中，选择“身份验证”。
1. 在“高级设置”中的“隐式授权”下，同时勾选“ID 令牌”和“访问令牌”复选框。 由于此应用需要将用户登录并调用 API，因此需要 ID 令牌和访问令牌。
1. 选择“保存”。

> #### <a name="setting-the-redirect-url-for-node"></a>设置 Node 的重定向 URL
> 对于 Node.js，可以在 *server.js* 文件中设置 Web 服务器端口。 本教程使用端口 30662 作为参考，但可以使用任何其他可用的端口。 请按照以下说明在应用程序注册信息中设置重定向 URL：<br/>
> - 切换回“应用程序注册”，并将 `http://localhost:30662/` 设置为 `Redirect URL`，或者，如果你使用的是自定义 TCP 端口（其中“[端口]”为自定义 TCP 端口号），则使用 `http://localhost:[port]/`。

<p>

> #### <a name="visual-studio-instructions-for-obtaining-the-redirect-url"></a>用于获取重定向 URL 的 Visual Studio 说明
> 按照以下步骤获取重定向 URL：
> 1. 在解决方案资源管理器中，选择项目并查看“属性”窗口。 如果没有看到“属性”窗口，请按 **F4**。
> 2. 将 **URL** 中的值复制到剪贴板：<br/> ![项目属性](media/active-directory-develop-guidedsetup-javascriptspa-configure/vs-project-properties-screenshot.png)<br />
> 3. 切换回“应用程序注册”，并将值设置为某个**重定向 URL**。

#### <a name="configure-your-javascript-spa"></a>配置 JavaScript SPA

1. 在设置项目期间创建的 `index.html` 文件中，添加应用程序注册信息。 在 `index.html` 文件正文中的 `<script></script>` 标记顶部添加以下代码：

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

    其中：
    - `Enter_the_Application_Id_here` - 是已注册应用程序的**应用程序（客户端）ID**。
    - `Enter_the_Tenant_Info_Here` - 设置为以下选项之一：
       - 如果应用支持“此组织目录中的帐户”，请将此值替换为“租户 ID”或“租户名称”（例如，contoso.microsoft.com）
       - 如果应用程序支持“任何组织目录中的帐户”，请将该值替换为 `organizations`
       - 如果应用支持“任何组织目录中的帐户和个人 Microsoft 帐户”，请将此值替换为“`common`”。 若要限制对“仅限个人 Microsoft 帐户”的支持，请将此值替换为“`consumers`”。
