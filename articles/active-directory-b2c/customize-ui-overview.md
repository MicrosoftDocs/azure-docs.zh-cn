---
title: 有关 Azure Active Directory B2C 中的用户界面自定义 | Microsoft Docs
description: 了解如何为使用 Azure Active Directory B2C 的应用程序自定义用户界面。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 26dea1b636fce9e7f5defddd5cf4bc4c7d78d5da
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64682021"
---
# <a name="about-user-interface-customization-in-azure-active-directory-b2c"></a>有关 Azure Active Directory B2C 中的用户界面自定义

通过 Azure Active Directory (Azure AD) B2C，可为应用程序的用户界面 (UI) 提供标志并进行自定义，这一点对于为你的客户提供无缝体验至关重要。 这些体验包括注册、登录、配置文件编辑和密码重置。 本文提供的信息有助于你自定义应用程序的 UI。

根据你对于这些体验的不同需求，可通过不同方式自定义应用程序的 UI。 例如：

- 如果使用[用户流](active-directory-b2c-reference-policies.md)在应用程序中提供注册或登录、密码重置或配置文件编辑体验，则可使用 [Azure 门户来自定义 UI](tutorial-customize-ui.md)。
- 如果使用的是 v2 用户流，则可以使用[页面布局模板](#page-layout-templates)来更改用户流页面的外观，无需进一步进行自定义。 例如，可以在用户流中的所有页面中应用海蓝或石板灰主题。
- 如果提供仅登录、其伴随的密码重置页面和验证电子邮件，请使用与 [Azure AD 登录页面](../active-directory/fundamentals/customize-branding.md)相同的自定义步骤。
- 如果客户在登录前尝试编辑其个人资料，则会将其重定向到使用与自定义 Azure AD 登录页面相同的步骤进行自定义的页面。
- 如果使用[自定义策略](active-directory-b2c-overview-custom.md)在应用程序中提供注册或登录、密码重置或配置文件编辑，则可使用[策略文件来自定义 UI](active-directory-b2c-ui-customization-custom.md)。
- 如果需要根据客户的决策提供动态内容，则可使用可根据查询字符串中发送的参数[更改页面内容的自定义策略](active-directory-b2c-ui-customization-custom-dynamic.md)。 例如，可以基于从 Web 或移动应用程序传递的参数，更改 Azure AD B2C 注册或登录页面上的背景图像。
- 可以在 Azure AD B2C [用户流](user-flow-javascript-overview.md)或[自定义策略](page-contract.md)中启用 JavaScript 客户端代码。

Azure AD B2C 在客户的浏览器中运行代码，并使用称为[跨域资源共享 (CORS)](https://www.w3.org/TR/cors/) 的新式方法。 在运行时，从在用户流或策略中指定的 URL 加载内容。 可以为不同的页面指定不同的 URL。 从 URL 加载的内容与从 Azure AD B2C 插入的 HTML 片段合并后，会向客户显示页面。

使用自己的 HTML 和 CSS 文件来自定义 UI 时，在开始之前请查看以下指南：

- 使用 Azure AD B2C 将 HTML 内容合并到页面中。 请勿复制并尝试更改 Azure AD B2C 提供的默认内容。 最好从头开始构建 HTML5 内容，将默认内容用作参考。
- 现在可以在自定义内容中包括 JavaScript。
- 支持的浏览器版本： 
    - Internet Explorer 11、10 和 Microsoft Edge
    - 对 Internet Explorer 9 和 8 的支持有限
    - Google Chrome 42.0 和更高版本
    - Mozilla Firefox 38.0 和更高版本
- 请确保在 HTML 中不包括窗体标记，因为这会干扰从 Azure AD B2C 注入的 HTML 生成的 POST 操作。

## <a name="page-layout-templates"></a>页面布局模板

对于 v2 用户流，你可以选择一个预先设计的模板，它为你的默认页面提供更好的外观，并用作你的自定义的良好基础。

在左侧菜单中，在“自定义”下选择“页面布局”。 然后，选择“模板(预览版)”。

![选择一个页面布局模板](media/customize-ui-overview/template.png)

从列表中选择一个模板。 例如，“海蓝”模板向用户流页面应用以下布局：

![“海蓝”模板](media/customize-ui-overview/ocean-blue.png)

选择某个模板时，选定的布局将应用于用户流中的所有页面，并且每个页面的 URI 将显示在“自定义页面 URI”字段中。

## <a name="where-do-i-store-ui-content"></a>在何处存储 UI 内容？

使用自己的 HTML 和 CSS 文件来自定义 UI 时，可以在任何位置托管 UI 内容，例如 [Azure Blob 存储](../storage/blobs/storage-blobs-introduction.md)、Web 服务器、CDN、AWS S3 或文件共享系统。 重要的一点是，可以在启用了 CORS 的公共 HTTPS 终结点上托管内容。 在内容中指定绝对 URL 时，必须使用绝对 URL。

## <a name="how-do-i-get-started"></a>如何入门？

请执行以下操作以自定义 UI：

- 使用位于 `<body>` 中某处的空 `<div id="api"></div>` 元素创建格式正确的 HTML 内容。 此元素标记插入 Azure AD B2C 内容的位置。 以下示例显示了一个最小化页面：

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>!Add your title here!</title>
        <link rel="stylesheet" href="https://mystore1.blob.core.windows.net/b2c/style.css">
      </head>
      <body>
        <h1>My B2C Application</h1>
        <div id="api"></div>   <!-- Leave this element empty because Azure AD B2C will insert content here. -->
      </body>
    </html>
    ```

- 在 HTTPS 终结点上托管内容（允许 CORS）。 配置 CORS 时必须启用 GET 和 OPTIONS 请求方法。
- 使用 CSS 设置 Azure AD B2C 插入页面的 UI 元素的样式。 以下示例显示了一个简单的 CSS 文件，其中还包含注册注入 HTML 元素的设置：

    ```css 
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center; 
    }
    .entry {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center; 
    }
    .create {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

- 创建或编辑策略以使用你创建的内容。

下表列出了 Azure AD B2C 合并到内容中 `<div id="api"></div>` 元素的 HTML 片段。

| 插入的页 | HTML 说明 |
| ------------- | ------------------- |
| 标识提供者选项 | 包含标识提供者的按钮列表，客户可在注册或登录时选择这些按钮。 这些按钮包括社交标识提供者（如 Facebook 和 Google）或本地帐户（基于电子邮件地址或用户名）。 |
| 本地帐户注册 | 包含一个窗体，用于基于电子邮件地址或用户名的本地帐户注册。 该窗体可以包含不同的输入控件，如文本输入框、密码输入框、单选按钮、单选下拉框和多选复选框。 |
| 社交帐户注册 | 使用社交标识提供者（如 Facebook 或 Google+）的现有帐户进行注册时，可能会显示此页面。 在必须使用注册窗体收集客户的其他信息时使用此页面。 |
| 统一注册或登录 | 处理客户的注册和登录，这些客户可以使用社交标识提供者（Facebook、Google）的帐户或本地帐户。 |
| 多重身份验证 | 用户可以在注册或登录期间（使用文字或语音）验证其电话号码。 |
| 错误 | 向客户提供错误信息。 |


## <a name="how-do-i-localize-content"></a>如何对内容进行本地化？

可通过在 Azure AD B2C 租户中启用[语言自定义](active-directory-b2c-reference-language-customization.md)来本地化 HTML 内容。 启用此功能可让 Azure AD B2C 将 OpenID Connect 参数 `ui-locales` 转发到终结点。 内容服务器可使用此参数提供特定语言的 HTML 页。

可以基于所用的区域设置从不同位置拉取内容。 在已启用 CORS 的终结点中，可以设置文件夹结构以托管特定语言的内容。 如果使用通配符值 {Culture:RFC5646}，则会调用正确的语言。 例如，自定义的页面 URI 可能类似于以下内容：`https://contoso.blob.core.windows.net/{Culture:RFC5646}/myHTML/unified.html`。 可通过从 `https://contoso.blob.core.windows.net/fr/myHTML/unified.html` 提取内容来加载法语页面

## <a name="examples"></a>示例

有关自定义示例，请下载并查看这些[示例模板文件](https://github.com/azureadquickstarts/b2c-azureblobstorage-client/archive/master.zip)。

## <a name="next-steps"></a>后续步骤

- 如果使用用户流，可以根据教程开始自定义 UI：[在 Azure Active Directory B2C 中自定义应用程序的用户界面](tutorial-customize-ui.md)。
- 如果使用自定义策略，可以根据本文开始自定义 UI：[使用 Azure Active Directory B2C 中的自定义策略自定义应用程序的用户界面](active-directory-b2c-ui-customization-custom.md)。

