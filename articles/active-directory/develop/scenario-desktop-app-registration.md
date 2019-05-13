---
title: 桌面应用程序调用 web Api （应用程序注册）-Microsoft 标识平台
description: 了解如何构建桌面应用调用 web Api （应用程序注册）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/18/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5da934709274d90668d94dfea3a9c223e191d032
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076055"
---
# <a name="desktop-app-that-calls-web-apis---app-registration"></a>桌面应用程序调用 web Api 的应用注册

本文包含桌面应用程序应用程序注册 specificities。

## <a name="supported-accounts-types"></a>受支持的帐户类型

支持桌面应用程序中的帐户类型取决于你想要启动的体验，因此在流上你希望使用。

### <a name="audience-for-interactive-token-acquisition"></a>交互式令牌获取的受众

如果桌面应用程序使用交互式身份验证，您可以从任何用户在登录[帐户类型](quickstart-register-app.md#register-a-new-application-using-the-azure-portal)

### <a name="audience-for-desktop-app-silent-flows"></a>对于桌面应用程序无提示流的受众

- 如果你想要使用集成 Windows 身份验证或用户名/密码，你的应用程序需要在自己的租户 （LOB 开发人员），或在 Azure Active directory 组织 （ISV 方案） 中的用户登录。 这些身份验证流不支持 Microsoft 个人帐户
- 如果你想要使用的设备代码流，你无法登录用户使用其 Microsoft 个人帐户尚未
- 如果具有传递 B2C 颁发机构和策略的社交标识登录在用户中，只能使用交互式和用户名和密码身份验证。

## <a name="redirect-uris"></a>重定向 URI

再次重定向 Uri 使用桌面应用程序中将取决于你想要使用的流。

- 如果使用交互式身份验证，你将想要使用`https://login.microsoftonline.com/common/oauth2/nativeclient`。 可通过单击相应的 URL 中实现此配置**身份验证**部分，了解你的应用程序
  
  > [!IMPORTANT]
  > 今天 MSAL.NET 默认情况下，在 Windows 上运行的桌面应用程序中使用另一个重定向 URI (`urn:ietf:wg:oauth:2.0:oob`)。 将来我们想要更改此默认值，并因此我们建议你使用 `https://login.microsoftonline.com/common/oauth2/nativeclient`

- 如果您的应用程序仅使用集成 Windows 身份验证，用户名/密码或设备代码流，不需要注册你的应用程序的重定向 URI。 实际上，这些流执行到 Microsoft 标识平台 v2.0 终结点的往返行程，并不会对任何特定的 URI 返回调用你的应用程序。 为了区分从机密客户端应用程序流，其不具有重定向 Uri 是 （客户端凭据流使用在守护程序应用程序中），您需要来表达你的应用程序是一个公共客户端应用程序。 此配置通过转到实现**身份验证**部分，为应用程序，以及在**高级设置**子节中，选择**是**，对问题**视为公共客户端应用程序**(在**默认客户端类型**段落)

  ![允许公共客户端](media/scenarios/default-client-type.png)

## <a name="api-permissions"></a>API 权限

桌面应用程序代表已登录用户调用 Api。 他们需要请求委托的权限。 它们不能请求应用程序权限 (这仅在中处理[守护程序应用程序](scenario-daemon-overview.md))

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [桌面应用程序-应用配置](scenario-desktop-app-configuration.md)
