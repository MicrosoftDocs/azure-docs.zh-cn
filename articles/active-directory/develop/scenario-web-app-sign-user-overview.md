---
title: 登录用户 （概述）-Microsoft 标识平台的 web 应用
description: 了解如何生成 web 应用登录用户 （概述）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: b46df5e4ec8352b47f744a507fad702c37aa5fba
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075095"
---
# <a name="scenario-web-app-that-signs-in-users"></a>场景：用户登录的 web 应用

了解您需要构建 web 应用登录用户的 Microsoft 标识平台。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>入门

如果你想要在用户中创建第一个可移植 (ASP.NET Core) web 应用进行签名，请按照本快速入门：

> [!div class="nextstepaction"]
> [快速入门：ASP.NET Core web 应用登录用户](quickstart-v2-aspnet-core-webapp.md)

如果想要随时了解 ASP.NET，试用以下教程：

> [!div class="nextstepaction"]
> [快速入门：ASP.NET Core web 应用登录用户](quickstart-v2-aspnet-webapp.md)

## <a name="overview"></a>概述

将身份验证添加到 web 应用，以便它可以在用户登录。 添加身份验证后，web 应用可访问有限的配置文件信息，例如自定义提供给它的用户体验。 Web 应用进行身份验证的 web 浏览器中的用户。 在此方案中，Web 应用程序指示用户的浏览器让用户登录到 Azure AD 中。 Azure AD 通过用户的浏览器返回一个登录响应，该响应在一个安全令牌中包含了关于用户的声明。 登录用户利用[Open ID Connect](./v2-protocols-oidc.md)标准协议本身通过中间件使用简化[库](scenario-web-app-sign-user-app-configuration.md#libraries-used-to-protect-web-apps)。

![Web 应用登录用户](./media/scenario-webapp/scenario-webapp-signs-in-users.svg)

作为第二个阶段还可以启用应用程序以代表已登录用户调用 Web Api。 此下一阶段是不同的方案，您会发现在[Web 应用程序调用 Web Api](scenario-web-app-call-api-overview.md)

> [!NOTE]
> 添加在登录到 web 应用程序都是为了防止 web 应用和验证的用户令牌，这是什么**中间件**库执行操作。 此方案不需要，但 Microsoft 身份验证库 (MSAL)，这是有关获取令牌以调用受保护的 Api。 当 web 应用需要调用 web Api，则仅将后续方案中引入了身份验证库。

## <a name="specifics"></a>详细信息

- 在应用程序注册期间将需要提供一个或多个 （如果将应用部署到多个位置时） 回复 Uri。 在某些情况下 （ASP.NET/ASP.NET 核），你将需要启用 IDToken。 最后您将想要设置注销 URI，以便你的应用程序对用户签名扩展做出反应。
- 在你的应用程序代码中，你将需要提供 web 应用程序委托登录到的颁发机构。 您可能想要自定义令牌验证 （尤其是在 ISV 方案）。
- Web 应用程序支持的任何帐户类型。 有关详细信息，请参阅[支持的帐户类型](v2-supported-account-types.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-web-app-sign-user-app-registration.md)
