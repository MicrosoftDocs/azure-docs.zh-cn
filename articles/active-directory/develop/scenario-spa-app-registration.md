---
title: 单页面应用程序 （应用程序注册）-Microsoft 标识平台
description: 了解如何生成单页面应用程序 （应用程序注册）
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
ms.openlocfilehash: b1faf4669dca2aaaf3f873e66f859473ccd99f10
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074825"
---
# <a name="single-page-application---app-registration"></a>单页面应用程序-应用注册

此页说明单页面应用程序 (SPA) 应用程序注册详细信息。

请按照步骤进行[注册新的应用程序与 Microsoft 标识平台](quickstart-register-app.md)，然后选择你的应用程序的受支持的帐户。 SPA 方案可以在你的组织或任何组织和个人 Microsoft 帐户支持使用帐户进行身份验证。

接下来，了解适用于单页面应用程序的应用程序注册的特定方面。

## <a name="register-a-redirect-uri"></a>注册的重定向 URI

隐式流将在重定向令牌发送到 web 浏览器中运行的单页面应用程序。 因此，它是一项重要的要求，若要注册的重定向你的应用程序可以在其中接收令牌的 URI。 请确保重定向 URI 完全与你的应用程序的 URI 相匹配。

中[Azure 门户](https://go.microsoft.com/fwlink/?linkid=2083908)，导航到已注册应用程序，在**身份验证**应用程序，选择的页**Web**平台和输入的值中的应用程序的重定向 URI**重定向 URI**字段。

## <a name="enable-the-implicit-flow"></a>启用隐式流

在同一**身份验证**页面上，在**高级设置**，则还必须启用**隐式授权**。 如果应用程序仅执行登录的用户并获取 ID 令牌，则足以让**ID 令牌**复选框。

如果你的应用程序还需要获取访问令牌来调用 Api，请确保启用**访问令牌**也复选框。 有关详细信息，请参阅[ID 令牌](./id-tokens.md)并[的访问令牌](./access-tokens.md)。

## <a name="api-permissions"></a>API 权限

单页面应用程序可代表已登录用户调用 Api。 他们需要请求委托的权限。 有关详细信息，请参阅[添加权限以访问 web Api](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用程序的代码配置](scenario-spa-app-configuration.md)
