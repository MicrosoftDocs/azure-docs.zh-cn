---
title: Azure Active Directory B2C 代码示例 | Microsoft Docs
description: Azure Active Directory B2C 移动、桌面、Web 和单页应用程序的代码示例。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.author: davidmu
ms.date: 01/23/2018
ms.custom: mvc
ms.topic: sample
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: ba3994f49f1d73aad1a5cf8211063f41d224c3d5
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64704083"
---
# <a name="azure-active-directory-b2c-code-samples"></a>Azure Active Directory B2C 代码示例

以下各表提供了应用程序示例的链接，包括 iOS、Android、.NET 和 Node.js。

## <a name="mobile-and-desktop-apps"></a>移动和桌面应用

| 示例 | 说明 |
|--------| ----------- |
| [ios-swift-native-msal](https://github.com/Azure-Samples/active-directory-b2c-ios-swift-native-msal) | 用 Swift 编写的一个 iOS 示例，它使用 OAuth 2.0 对 Azure AD B2C 用户进行身份验证并调用一个 API |
| [android-native-msal](https://github.com/Azure-Samples/active-directory-b2c-android-native-msal) | 一个简单的 Android 应用，它展示了如何使用 MSAL 来通过 Azure Active Directory B2C 对用户进行身份验证，并使用生成的令牌访问一个 Web API。 |
| [ios-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-ios-native-appauth) | 此示例展示了如何利用第三方库来使用 Objective-C 开发 iOS 应用程序，用以通过我们的 Azure AD B2C 标识服务对 Microsoft 标识用户进行身份验证。 |
| [android-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-android-native-appauth) | 此示例展示了如何利用第三方库来开发 Android 应用程序，用以通过我们的 B2C 标识服务对 Microsoft 标识用户进行身份验证并使用 OAuth 2.0 访问令牌调用一个 Web API。 |
| [dotnet-desktop](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) | 此示例展示了 Windows Desktop .NET (WPF) 应用程序如何使用 Azure AD B2C 完成用户登录，使用 MSAL.NET 获取访问令牌，并调用一个 API。 | 
| [xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | 一个简单的 Xamarin Forms 应用，它展示了如何使用 MSAL 来通过 Azure Active Directory B2C 对用户进行身份验证，并使用生成的令牌访问一个 Web API。 |

## <a name="web-apps-and-apis"></a>Web 应用和 API

| 示例 | 说明 |
|--------| ----------- |
| [dotnet-webapp-and-webapi](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) | 调用了一个 .NET Web API 的 .NET Web 应用程序的组合示例，两者都使用 Azure AD B2C 提供保护。 |
| [dotnetcore-webapp](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapp) | 一个 ASP.NET Core Web 应用程序，它可以使用 Azure AD B2C 完成用户登录，使用 MSAL.NET 获取访问令牌，并调用一个 API。 |
| [openidconnect-nodejs](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-NodeJS) | 一个 Node.js 应用，它提供了一种便捷的方法来使用 OpenID Connect 设置通过 Express 开发的 Web 应用程序。 |
| [javascript-nodejs-webapp](https://github.com/AzureADQuickStarts/active-directory-b2c-javascript-nodejs-webapp) | 一个 node.js 服务器，它提供了一种便捷的方法来使用 OAuth2 协议设置 REST API 服务。 |
| [javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) | 一个用于 Azure AD B2C 的小 node.js Web API，它展示了如何使用 passport.js 保护 Web API 并接受 B2C 访问令牌。 |

## <a name="single-page-apps"></a>单页应用

| 示例 | 说明 |
|--------| ----------- |
| [javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) | 调用了一个 Web API 的单页应用程序 (SPA)。 身份验证是利用 MSAL.js 通过 Azure AD B2C 执行的。 | 
| [javascript-hellojs-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-hellojs-singlepageapp) | 一个单页应用，其中实现了一个 ASP.NET Web API 后端，使用 Azure AD B2C 完成用户注册和登录，并使用 OAuth 2.0 访问令牌调用 Web API。 |
