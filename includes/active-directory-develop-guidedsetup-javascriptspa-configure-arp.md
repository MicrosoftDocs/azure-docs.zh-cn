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
ms.openlocfilehash: 7a6fa45d0f4ea09fbc84d3185aa0e58db165ac19
ms.sourcegitcommit: ed66a704d8e2990df8aa160921b9b69d65c1d887
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64947342"
---
## <a name="add-the-applications-registration-information-to-your-app"></a>向应用添加应用程序的注册信息

在此步骤中，需要配置应用程序注册信息的重定向 URL，然后将应用程序 ID 添加到 JavaScript SPA 应用程序。

### <a name="configure-redirect-url"></a>配置重定向 URL

使用基于 Web 服务器的 index.html 页的 URL 配置 `Redirect URL` 字段，然后单击“更新”。

> #### <a name="visual-studio-instructions-for-obtaining-the-redirect-url"></a>用于获取重定向 URL 的 Visual Studio 说明
> 按照以下步骤获取重定向 URL：
> 1. 在解决方案资源管理器中，选择项目并查看“属性”窗口。 如果没有看到“属性”窗口，请按 **F4**。
> 2. 将 **URL** 中的值复制到剪贴板：<br/> ![项目属性](media/active-directory-develop-guidedsetup-javascriptspa-configure/vs-project-properties-screenshot.png)<br />
> 3. 将此值粘贴为此页顶部的“重定向 URL”，然后选择“更新”。

<p>

> #### <a name="setting-redirect-url-for-node"></a>设置 Node 的重定向 URL
> 对于 Node.js，可以在 *server.js* 文件中设置 Web 服务器端口。 本教程使用端口 30662 作为参考，但可以使用任何其他可用的端口。 请按照以下说明在应用程序注册信息中设置重定向 URL：<br/>
> 将此页面顶部的“重定向 URL”设置为 `http://localhost:30662/`，如果使用自定义 TCP 端口，则设置为 `http://localhost:[port]/`（其中 *[port]* 是自定义 TCP 端口号），然后单击“更新”

### <a name="configure-your-javascript-spa-application"></a>配置 JavaScript SPA 应用程序

1. 在设置项目期间创建的 `index.html` 文件中，添加应用程序注册信息。 在 `index.html` 文件正文中的 `<script></script>` 标记顶部添加以下代码：

```javascript
var applicationConfig = {
    clientID: "Enter_the_Application_Id_here",
    authority: "https://login.microsoftonline.com/common",
    graphScopes: ["user.read"],
    graphEndpoint: "https://graph.microsoft.com/v1.0/me"
};
```
