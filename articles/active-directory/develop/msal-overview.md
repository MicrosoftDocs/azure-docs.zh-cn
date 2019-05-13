---
title: 了解 Microsoft 身份验证库 (MSAL) | Azure
description: 借助 Microsoft 身份验证库 (MSAL)，应用程序开发人员可以获取令牌，以调用受保护的 Web API。 这些 Web API 可以是 Microsoft Graph、其他 Microsoft API、第三方 Web API 或你自己的 Web API。 MSAL 支持多个应用程序体系结构和平台。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/25/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2198a5efe276ce872487cde7b2ddb5cb00c58c50
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079735"
---
# <a name="overview-of-microsoft-authentication-library-msal"></a>Microsoft 身份验证库 (MSAL) 的概述
借助 Microsoft 身份验证库 (MSAL)，开发人员能够从 Microsoft 标识平台终结点获取[令牌](developer-glossary.md#security-token)，以访问受保护的 Web API。 这些 Web API 可以是 Microsoft Graph、其他 Microsoft API、第三方 Web API 或你自己的 Web API。 MSAL 适用于 .NET、JavaScript、Android 和 iOS，支持许多不同的应用程序体系结构和平台。

MSAL 为你提供了许多获取令牌的方法，将一致的 API 用于许多平台。 使用 MSAL 具有以下好处：

* 无需直接在应用程序中对协议使用 OAuth 库或代码。
* 代表用户或代表应用程序获取令牌（如果适用于平台）。
* 维护令牌缓存，并在即将过期时为你刷新令牌。 你不需要自行处理令牌过期。
* 帮助你指定你希望应用程序登录的受众（你的组织、一系列组织、工作和学校以及 Microsoft 个人帐户、Azure AD B2C 的社交身份、主权云和国家云中的用户）。
* 可帮助你通过配置文件设置应用程序。
* 可显示可操作异常、日志和遥测，从而帮助你对应用进行故障排除。

## <a name="application-types-and-scenarios"></a>应用程序类型和方案
通过使用 MSAL，可从许多应用程序类型获取令牌：Web 应用程序、Web API、单页应用 (JavaScript)、移动和本机应用程序，以及守护程序和服务器端应用程序。 

可以在许多应用程序方案中使用 MSAL，包含以下方案：

* [单页应用程序 (JavaScript)](scenario-spa-overview.md) 
* [Web 应用登录用户](scenario-web-app-sign-user-overview.md)
* [Web 应用程序登录用户并代表用户调用 web API](scenario-web-app-call-api-overview.md)
* [保护 Web API，因此仅通过身份验证的用户可以访问它](scenario-protected-web-api-overview.md)
* [Web API 代表已登录用户调用其他下游 Web API](scenario-web-api-call-api-overview.md)
* [桌面应用程序代表已登录用户调用 Web API](scenario-desktop-overview.md)
* [移动应用程序代表以交互方式登录的用户调用 Web API](scenario-mobile-overview.md)。
* [桌面/服务守护程序应用程序代表自己调用 Web API](scenario-daemon-overview.md)

## <a name="languages-and-frameworks"></a>语言和框架

| 库 | 支持的平台和框架|
| --- | --- | 
| ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/>[MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)| .NET Framework、.NET Core、Xamarin Android、Xamarin iOS、通用 Windows 平台|
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/>[MSAL.js（预览版）](https://github.com/AzureAD/microsoft-authentication-library-for-js)| JavaScript/TypeScript 框架：例如 AngularJS、Ember.js 或 Durandal.js|
| ![适用于 Android 的 MSAL](media/sample-v2-code/logo_Android.png) <br/>[适用于 Android 的 MSAL（预览版）](https://github.com/AzureAD/microsoft-authentication-library-for-android)|Android|
| ![适用于 iOS 的 MSAL](media/sample-v2-code/logo_iOS.png) <br/>[MSAL.Objective-C（预览版）](https://github.com/AzureAD/microsoft-authentication-library-for-objc)|iOS|

## <a name="differences-between-adal-and-msal"></a>ADAL 和 MSAL 之间的差异
Active Directory 身份验证库 (ADAL) 与适用于开发人员的 Azure AD (v1.0) 终结点集成，其中 MSAL 与 Microsoft 标识平台 (v2.0) 终结点集成。 V1.0 终结点支持工作帐户，但不支持个人帐户。 V2.0 终结点将 Microsoft 个人帐户和工作帐户统一到单个身份验证系统。 此外，借助 MSAL，还可以获取 Azure AD B2C 的身份验证。

有关更多具体信息，请参阅[从 ADAL.NET 迁移到 MSAL.NET](msal-net-migration.md) 和[从 ADAL.js 迁移到 MSAL.js](msal-compare-msal-js-and-adal-js.md)。

            