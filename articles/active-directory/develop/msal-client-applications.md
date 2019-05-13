---
title: 客户端应用程序 （Microsoft 身份验证库） |Azure
description: 了解有关公共客户端和机密客户端应用程序中 Microsoft 身份验证库 (MSAL)。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/25/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: f21aa62bae7599cf586ccf6f885ad9f58c148d1f
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077135"
---
# <a name="public-client-and-confidential-client-applications"></a>公共客户端和机密客户端应用程序
Microsoft 身份验证库 (MSAL) 定义了两种类型的客户端： 公共客户端和机密客户端。 通过使用授权服务器的安全地进行身份验证和维护其凭据机密性的客户端能力来区分两个客户端类型。  与此相反，Azure AD 身份验证库 (ADAL) 提供身份验证上下文 （即连接到 Azure AD） 的概念。

- **机密客户端应用程序**是在服务器 （Web 应用、 Web API 或甚至服务/守护程序应用程序） 运行的应用程序。 它们被视为访问权限，很难，因此能够保持应用程序密码。 机密客户端都能够保存时间机密的配置。 客户端的每个实例都有不同的配置 （包括 clientId 和机密）。 这些值是很难使最终用户可提取。 Web 应用是最常见的机密客户端。 通过 web 浏览器中，公开的客户端 ID，但仅在返回通道中传递并永远不会直接公开机密。

    机密客户端应用程序： <BR>
    ![Web 应用](media/msal-client-applications/web-app.png) ![Web API](media/msal-client-applications/web-api.png) ![守护程序/服务](media/msal-client-applications/daemon-service.png)

- **公共客户端应用程序**是在设备或桌面机上或在 web 浏览器中运行的应用程序。 它们不受信任，可以安全地保留应用程序机密，并因此只能访问 Web Api 代表用户 （它们只支持公共客户端流）。 公共客户端将无法保存配置一次性机密，并因此具有任何客户端机密。

    公共客户端应用程序： <BR>
    ![桌面应用程序](media/msal-client-applications/desktop-app.png) ![Browserless API](media/msal-client-applications/browserless-app.png) ![移动应用](media/msal-client-applications/mobile-app.png)

> [!NOTE]
> MSAL.js，在公共和机密客户端应用程序没有区别。  MSAL.js 表示客户端应用程序为基于用户代理的应用程序，如 web 浏览器用户代理中，客户端代码执行所在的公共客户端。  这些客户端不存储机密，因为浏览器上下文是可公开访问。

## <a name="comparing-the-client-types"></a>比较客户端类型
有一些共性和公共客户端与机密客户端之间的差异的应用程序：

- 这两种类型的应用程序维护用户令牌缓存，并且可以获取令牌以无提示方式 （在该符号已在令牌缓存中的情况下）。 机密客户端应用程序还具有适用于应用本身的令牌的应用令牌缓存。
- 同时管理用户帐户和可以从用户令牌缓存中获取帐户、 从其标识符获取帐户或删除帐户。
- 公共客户端应用程序具有四个获取令牌 （四个身份验证流），而机密客户端应用程序具有三个 （和一种方法来计算的标识提供者 URL 的授权终结点） 的方式。 有关详细信息，请参阅方案以及获取令牌。

如果在过去使用 ADAL，您可能注意到，与 ADAL 的身份验证上下文，相反 MSAL 中客户端 ID （也称为应用程序 ID 或应用程序 ID） 一次传递在构造应用程序，并将不再需要在获取令牌时，重复。 这种情况这两个公共和机密客户端应用程序。 机密客户端应用程序的构造函数还传递客户端凭据： 它们与标识提供程序共享的机密。

## <a name="next-steps"></a>后续步骤
学习内容：
- [客户端应用程序配置选项](msal-client-application-configuration.md)
- [实例化客户端应用程序使用 MSAL.NET](msal-net-initializing-client-applications.md)。
- [实例化客户端应用程序使用 MSAL.js](msal-js-initializing-client-applications.md)。
