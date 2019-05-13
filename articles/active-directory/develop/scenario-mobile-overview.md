---
title: 移动应用程序调用 web Api-概述 |Microsoft 标识平台
description: 了解如何构建移动应用程序调用 web Api （概述）
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: eb0acd1534bab11eb57a7aa0e695f192b5999ed2
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076490"
---
# <a name="scenario-mobile-application-that-calls-web-apis"></a>场景：移动应用程序调用 web Api

了解您需要构建一个移动应用，调用 web Api。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Prerequisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>入门

创建第一个移动应用程序并试用快速入门 ！

> [!div class="nextstepaction"]
> [快速入门：获取令牌，然后从 Android 应用调用 Microsoft Graph API](./quickstart-v2-android.md)
>
> [快速入门：获取令牌，然后从 iOS 应用调用 Microsoft Graph API](./quickstart-v2-ios.md)
>
> [快速入门：获取令牌，然后从 Xamarin iOS 和 Android 应用调用 Microsoft Graph API](https://github.com/Azure-Samples/active-directory-xamarin-native-v2)

## <a name="overview"></a>概述

在创建移动应用时，个性化的、 无缝的最终用户体验至关重要。  Microsoft 标识平台，移动开发人员实现此适用于 iOS 和 Android 用户。 可以在 Azure AD 中，个人 Microsoft 帐户，请登录你的应用程序和 Azure AD B2C 用户并获取令牌以代表自己调用 web API。 若要实现这些流，我们将使用 Microsoft 身份验证库 (MSAL) 实现的行业标准[OAuth2.0 授权代码流](v2-oauth2-auth-code-flow.md)。

![守护程序应用](./media/scenarios/mobile-app.svg)

移动应用的注意事项：

- ***关键在于用户体验***:允许用户查看您的应用程序的值之前要求登录，并仅请求所需的权限。
- ***支持所有用户配置***:许多移动业务用户是在条件性访问和设备符合性策略。 请确保支持这些重要方案。
- ***单一登录 (SSO) 实现***:MSAL 和 Microsoft 标识平台简化启用单一登录到设备的浏览器或 Microsoft 验证器 （和 Android 上的 Intune 公司门户）。

## <a name="specifics"></a>详细信息

Microsoft 标识平台上构建移动应用时, 中的端到端体验有几个注意事项：

- 根据平台，而无需任何交互登录不可能在首次登录。 iOS 中，例如，需要可通过 Microsoft Authenticator （和 Android 上的 Intune 公司门户） 获取第一个 SSO 时间时显示用户交互的应用。
- 在 iOS 和 Android 上，MSAL 还可以使用外部浏览器 （这可能会显示在您的应用程序之上） 来登录用户。 这可以自定义以改为使用应用程序内 WebViews。
- 永远不会在移动应用程序中使用机密，它将为所有用户访问。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-mobile-app-registration.md)
