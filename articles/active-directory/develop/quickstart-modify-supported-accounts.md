---
title: 修改注册到 Microsoft 标识平台的应用程序支持的帐户 | Azure
description: 配置注册到 Microsoft 标识平台的应用程序，更改能够访问应用程序的人或具体帐户。
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
ms.openlocfilehash: e2c52bd0bf6c1e385248bd379a8001756d28ac2d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413584"
---
# <a name="quickstart-modify-the-accounts-supported-by-an-application"></a>快速入门：修改应用程序支持的帐户

在 Microsoft 标识平台中注册应用程序时，可能会希望系统只允许你组织中的用户访问你的应用程序。 或者，可能还希望应用程序可由外部组织中的用户访问，或者可由外部组织中的用户以及不一定属于某个组织的用户（个人帐户）访问。

本快速入门介绍如何修改应用程序的配置，更改能够访问应用程序的人或具体帐户。

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
1. 按步骤[更改应用程序注册以支持不同的帐户](#change-the-application-registration-to-support-different-accounts)。
1. 如果有单页应用程序，请[启用 OAuth 2.0 隐式授权](#enable-oauth-20-implicit-grant-for-single-page-applications)。

## <a name="change-the-application-registration-to-support-different-accounts"></a>更改应用程序注册以支持不同的帐户

如果正在编写一个要供组织外部的客户或合作伙伴使用的应用程序，则需要在 Azure 门户中更新应用程序定义。

> [!IMPORTANT]
> Azure AD 要求多租户应用程序的应用程序 ID URI 全局唯一。 应用 ID URI 是在协议消息中标识应用程序的方式之一。 就单租户应用程序而言，应用 ID URI 在该租户中保持唯一便已足够。 就多租户应用程序而言，该 URI 必须全局唯一，以便 Azure AD 能够在所有租户中找到该应用程序。 系统通过要求应用 ID URI 必须具有与已验证 Azure AD 租户域匹配的主机名，来强制实施全局唯一性。 例如，如果租户的名称为 contoso.onmicrosoft.com，则有效的应用 ID URI 为 https://contoso.onmicrosoft.com/myapp。 如果租户具有已验证的域 contoso.com，则有效的应用 ID URI 也是 https://contoso.com/myapp。 如果应用程序 ID URI 不遵循此模式，则将应用程序设置为多租户就会失败。

### <a name="to-change-who-can-access-your-application"></a>更改谁能够访问你的应用程序

1. 在应用的“概览”页中，选择“身份验证”部分，然后更改在“支持的帐户类型”下选择的值。
    * 若要生成业务线 (LOB) 应用程序，请选择“仅此目录中的帐户”。 如果未在目录中注册应用程序，则此选项不可用。
    * 若要以所有企业和教育客户为目标，请选择“任何组织目录中的帐户”。
    * 若要以最广泛的客户为目标，请选择“任何组织目录中的帐户和个人 Microsoft 帐户”。
1. 选择“保存”。

## <a name="enable-oauth-20-implicit-grant-for-single-page-applications"></a>为单页应用程序启用 OAuth 2.0 隐式授权

通常将单页应用程序 (SPA) 构建为一个在浏览器中运行的 JavaScript 重型前端，该前端调用应用程序的 Web API 后端来执行其业务逻辑。 对于托管在 Azure AD 中的 SPA，可以使用 OAuth 2.0 隐式授权对具有 Azure AD 的用户进行身份验证，并获取可用来保护从应用程序 JavaScript 客户端到其后端 Web API 的调用的令牌。

用户授予同意之后，可以使用同一个身份验证协议来获取令牌以保护客户端与针对应用程序配置的其他 Web API 资源之间的调用。 若要了解有关隐式授权的详细信息，并确定其是否适合应用程序方案，请了解 Azure AD [v1.0](v1-oauth2-implicit-grant-flow.md) 和 [v2.0](v2-oauth2-implicit-grant-flow.md) 中的 OAuth 2.0 隐式授权流。

默认情况下，为应用程序禁用了 OAuth 2.0 隐式授权。 可以执行下述步骤，为应用程序启用 OAuth 2.0 隐式授权。

### <a name="to-enable-oauth-20-implicit-grant"></a>启用 OAuth 2.0 隐式授权

1. 在应用的“概览”页中，选择“身份验证”部分。
1. 在“高级设置”下找到“隐式授权”部分。
1. 选择“ID 令牌”和/或“访问令牌”。
1. 选择“保存”。

## <a name="next-steps"></a>后续步骤

了解下述其他相关的应用管理快速入门：

* [将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)
* [将客户端应用程序配置为访问 Web API](quickstart-configure-app-access-web-apis.md)
* [将应用程序配置为公开 Web API](quickstart-configure-app-expose-web-apis.md)
* [删除注册到 Microsoft 标识平台的应用程序](quickstart-remove-app.md)

了解有关表示已注册应用程序和它们之间的关系的两个 Azure AD 对象的详细信息，请参阅[应用程序对象和服务主体对象](app-objects-and-service-principals.md)。

深入了解使用 Azure Active Directory 开发应用程序时应使用的品牌准则，请参阅[应用程序的品牌准则](howto-add-branding-in-azure-ad-apps.md)。
