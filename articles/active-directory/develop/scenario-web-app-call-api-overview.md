---
title: Web 应用调用 web Api （概述）-Microsoft 标识平台
description: 了解如何构建 web 应用调用 web Api （概述）
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
ms.openlocfilehash: ce45f11a697b72ebdd0fe01166a70e7b47aa8e9f
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076295"
---
# <a name="scenario-web-app-that-calls-web-apis"></a>场景：调用 web Api 的 web 应用

了解如何在 Microsoft 标识平台上构建的 web 应用登录用户并调用 web Api 代表已登录用户。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

此方案假设您已经历以下方案：

> [!div class="nextstepaction"]
> [登录用户的 web 应用](scenario-web-app-sign-user-overview.md)

## <a name="overview"></a>概述

将身份验证添加到 Web 应用，因此可以在用户登录并调用 web API 代表已登录用户。

![调用 web Api 的 web 应用](./media/scenario-webapp/web-app.svg)

Web 应用调用 web Api:

- 是机密客户端应用程序。
- 这就是为什么他们已注册到 Azure AD 机密 （应用程序密码或证书）。 此机密即传入过程调用 Azure AD，以获取令牌

## <a name="specifics"></a>详细信息

> [!NOTE]
> 将登录添加到 Web 应用不使用 MSAL 库，因为这是如何保护 Web 应用。 保护库来实现名为中间件的库。 这是在前面的方案对象[登录到 Web 应用的用户](scenario-web-app-sign-user-overview.md)
>
> 在从 Web 应用调用 web Api，需要为这些 web Api 获取访问令牌。 可以使用 MSAL 库来获取这些令牌。

因此，开发人员对此方案的端到端体验具有作为特定方面：

- 期间[应用程序注册](scenario-web-app-call-api-app-registration.md)、 将需要提供一个或多个 （如果将应用部署到多个位置时） 回复 Uri 时，机密，或需要与 Azure AD 共享证书。
- [应用程序配置](scenario-web-app-call-api-app-configuration.md)需要与 Azure AD 应用程序注册期间提供共享的客户端凭据

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-web-app-call-api-app-registration.md)
