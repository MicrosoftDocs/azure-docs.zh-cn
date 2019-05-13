---
title: 配置应用程序来访问 Web API - Microsoft 标识平台
description: 了解如何配置注册到 Microsoft 标识平台的应用程序，使之包括访问 Web API 所需的重定向 URI、凭据或权限。
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7b2f5331b22f6292b00c367c6abb8cd4f439a70c
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413977"
---
# <a name="quickstart-configure-a-client-application-to-access-web-apis"></a>快速入门：将客户端应用程序配置为访问 Web API

为使 Web/机密客户端应用程序能够参与要求身份验证的授权流程（以及获取访问令牌），必须建立安全凭据。 Azure 门户支持的默认身份验证方法为“客户端 ID + 机密密钥”。

此外，在客户端可以访问资源应用程序公开的 Web API（例如 Microsoft Graph API）之前，许可框架可确保客户端根据请求的权限获取所需的授权。 默认情况下，所有应用程序都可以从 Microsoft Graph API 选择权限。 默认选择[图形 API 的“登录并读取用户配置文件”权限](https://developer.microsoft.com/graph/docs/concepts/permissions_reference#user-permissions)。 可以从每个所需 Web API 的[两种类型的权限](developer-glossary.md#permissions)中进行选择：

* **应用程序权限** - 客户端应用程序需要亲自直接访问 Web API（无用户上下文）。 此类型的权限需要管理员同意，并且不可用于公共（桌面和移动）客户端应用程序。
* **委托的权限** - 客户端应用程序需要以登录用户的身份访问 Web API，但访问权限受所选权限的限制。 除非权限需要管理员许可，否则用户可以授予此类型的权限。

  > [!NOTE]
  > 将委托权限添加到应用程序不会自动向租户中的用户授予许可。 除非管理员代表所有用户授予许可，否则用户仍必须在运行时手动同意添加的委派权限。

本快速入门介绍如何将应用配置为执行以下操作：

* [将重定向 URL 添加到应用程序](#add-redirect-uris-to-your-application)
* [将凭据添加到 Web 应用程序](#add-credentials-to-your-web-application)
* [添加用于访问 Web API 的权限](#add-permissions-to-access-web-apis)

## <a name="prerequisites"></a>先决条件

若要开始，请确保满足下列先决条件：

* 了解支持的[权限和许可](v2-permissions-and-consent.md)，这是在生成其他用户或应用程序需要使用的应用程序时必须理解的。
* 拥有一个其中已注册了应用程序的租户。
  * 如果尚未注册应用，请[了解如何将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)。

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>登录到 Azure 门户，并选择应用

在配置应用之前，请执行以下步骤：

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务，然后选择“应用注册”。
1. 找到并选择要配置的应用程序。 选择应用以后，会看到应用程序的“概览”页或主注册页。
1. 按步骤将应用程序配置为访问 Web API： 
    * [将重定向 URL 添加到应用程序](#add-redirect-uris-to-your-application)
    * [将凭据添加到 Web 应用程序](#add-credentials-to-your-web-application)
    * [添加用于访问 Web API 的权限](#add-permissions-to-access-web-apis)

## <a name="add-redirect-uris-to-your-application"></a>将重定向 URL 添加到应用程序

[![为 Web 和公共客户端应用添加自定义重定向 URI](./media/quickstart-update-azure-ad-app-preview/authentication-redirect-uris-expanded.png)](./media/quickstart-update-azure-ad-app-preview/authentication-redirect-uris-expanded.png#lightbox)

若要将重定向 URI 添加到应用程序，请执行以下步骤：

1. 在应用的“概览”页中，选择“身份验证”部分。

1. 若要为 Web 和公共客户端应用程序添加自定义重定向 URI，请执行以下步骤：

   1. 找到“重定向 URI”部分。
   1. 选择要生成的应用程序的类型：“Web”或“公共客户端(移动和桌面)”。
   1. 输入应用程序的重定向 URI。
      * 对于 Web 应用程序，请提供应用程序的基 URL。 例如，`http://localhost:31544` 可以是本地计算机上运行的 Web 应用程序的 URL。 用户将使用此 URL 登录到 Web 客户端应用程序。
      * 对于公共应用程序，请提供 Azure AD 返回令牌响应时所用的 URI。 输入特定于应用程序的值，例如 https://MyFirstApp。

1. 若要从建议用于公共客户端（移动、桌面）的重定向 URI 中进行选择，请执行以下步骤：

    1. 找到“建议用于公共客户端(移动、桌面)的重定向 URI”部分。
    1. 通过复选框选择适用于应用程序的重定向 URI。

## <a name="add-credentials-to-your-web-application"></a>将凭据添加到 Web 应用程序

[![添加证书和客户端机密](./media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets-expanded.png)](./media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets-expanded.png#lightbox)

若要向 Web 应用程序添加凭据，请执行以下操作：

1. 在应用的“概览”页中，选择“证书和机密”部分。

1. 若要添加证书，请执行以下步骤：

    1. 选择“上传证书”。
    1. 选择要上传的文件。 它必须是以下文件类型之一：.cer、.pem、.crt。
    1. 选择 **添加** 。

1. 若要添加客户端机密，请执行以下步骤：

    1. 选择“新建客户端机密”。
    1. 添加客户端机密的说明。
    1. 选择持续时间。
    1. 选择 **添加** 。

> [!NOTE]
> 保存配置更改后，最右边的列会包含客户端机密值。 **请务必复制此值**，以便在客户端应用程序代码中使用，因为退出此页后将无法访问此密钥。

## <a name="add-permissions-to-access-web-apis"></a>添加用于访问 Web API 的权限

[![添加 API 权限](./media/quickstart-update-azure-ad-app-preview/api-permissions-expanded.png)](./media/quickstart-update-azure-ad-app-preview/api-permissions-expanded.png#lightbox)

若要添加从客户端访问资源 API 的权限，请执行以下操作：

1. 在应用的“概览”页中，选择“API 权限”部分。
1. 选择“添加权限”按钮。
1. 默认情况下，此视图允许从“Microsoft API”进行选择。 选择感兴趣的 API 部分。
    * **Microsoft API** - 用于选择 Microsoft API（例如 Microsoft Graph）的权限。
    * **组织使用的 API** - 用于选择由组织公开的 API 或组织已与之集成的 API 的权限。
    * **我的 API** - 用于选择你已经公开的 API 的权限。
1. 选择 API 后，会看到“请求 API 权限”页。 如果 API 公开托管的权限和应用程序权限，请选择应用程序需要哪种类型的权限。
1. 完成后，请选择“添加权限”。 此时会返回到“API 权限”页，其中的权限已保存并添加到表。

## <a name="next-steps"></a>后续步骤

了解下述其他相关的应用管理快速入门：

* [将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)
* [将应用程序配置为公开 Web API](quickstart-configure-app-expose-web-apis.md)
* [修改应用程序支持的帐户](quickstart-modify-supported-accounts.md)
* [删除注册到 Microsoft 标识平台的应用程序](quickstart-remove-app.md)

了解有关表示已注册应用程序和它们之间的关系的两个 Azure AD 对象的详细信息，请参阅[应用程序对象和服务主体对象](app-objects-and-service-principals.md)。

深入了解使用 Azure Active Directory 开发应用程序时应使用的品牌准则，请参阅[应用程序的品牌准则](howto-add-branding-in-azure-ad-apps.md)。
