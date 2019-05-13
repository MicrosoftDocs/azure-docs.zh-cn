---
title: Microsoft 标识平台的代码示例 |Microsoft Docs
description: 提供的索引可用的 Microsoft 标识平台 （V2 终结点） 的代码示例，按方案。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: a242a5ff-7300-40c2-ba83-fb6035707433
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/26/2018
ms.author: celested
ms.reviewer: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 09c5bd217f852f7130888d03e3368278af8abce5
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190878"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Microsoft 标识平台的代码示例 （v2.0 终结点）

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

可以使用 Microsoft 标识平台执行以下操作：

- 向 Web 应用程序和 Web API 添加身份验证和授权。
- 要求使用访问令牌来访问受保护的 Web API。

本文简要描述，并为您提供指向示例的 Microsoft 标识平台终结点。 这些示例将展示其工作原理以及可以在应用程序中使用的代码片段。 在代码示例页上，可以找到在要求、安装和设置方面提供帮助的详细自述主题。 代码中的注释可帮助你理解关键部分。

> [!NOTE]
> 如果对 v1.0 示例感兴趣，请参阅 [Azure AD 代码示例（v1.0 终结点）](sample-v1-code.md)。

若要了解每个示例类型的基本方案，请参阅[Microsoft 标识平台终结点的应用类型](v2-app-types.md)。

你也可以为 GitHub 上的示例做出补充。 若要了解如何操作，请参阅 [Microsoft Azure Active Directory 示例和文档](https://github.com/Azure-Samples?page=3&query=active-directory)。

## <a name="single-page-applications-spa"></a>单页应用程序 (SPA)

这些示例展示了如何编写使用 Microsoft 标识平台保护的单页面应用程序。 这些示例使用下列种类的 MSAL.js 之一：

* [适用于 JavaScript 的 Microsoft 身份验证库](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)
* [适用于 Angular 的 Microsoft 身份验证库](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)
* [适用于 AngularJS 的 Microsoft 身份验证库](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs)

| 平台 | 调用 Microsoft Graph |
| -------- | --------------------- |
| ![JavaScript](media/sample-v2-code/logo_js.png) JavaScript (msal.js)  | [javascript-graphapi-web-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-web-v2) |
| ![Angular JS](media/sample-v2-code/logo_angular.png) JavaScript (MSAL AngularJS) | [MsalAngularjsDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angularjs/samples/MsalAngularjsDemoApp)
| ![Angular](media/sample-v2-code/logo_angular.png) JavaScript (MSAL Angular) | [MSALAngularDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular/samples/MSALAngularDemoApp) |

## <a name="web-applications"></a>Web 应用程序

以下示例演示了将用户登录的 Web 应用。 一些示例还演示了使用用户标识调用 Microsoft Graph 或你自己的 Web API 的应用程序。

| 平台 | 仅让用户登录 | 让用户登录并调用 Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core 2.1 | [ASP.NET Core web 应用登录用户教程](https://aka.ms/aspnetcore-webapp-sign-in) | 在同一示例[ASP.NET Core Web 应用调用 Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph)阶段 |
| ![ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [ASP.NET 快速入门](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> |[msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)
| ![Node.js](media/sample-v2-code/logo_nodejs.png)  |                   | [Node.js 快速入门](https://github.com/azureadquickstarts/appmodelv2-webapp-openidconnect-nodejs) |
| ![Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |

## <a name="desktop-and-mobile-public-client-apps"></a>桌面和移动公共客户端应用

以下示例展示了以用户身份访问 Microsoft Graph API 或你自己的 Web API 的公共客户端应用程序（桌面/移动应用程序）。 所有这些客户端应用程序均使用 Microsoft 身份验证库 (MSAL)。

| 客户端应用程序 | 平台 | 流/授权 | 调用 Microsoft Graph | 调用 ASP.NET Core 2.0 Web API |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| 桌面 (WPF)      | ![.NET/C#](media/sample-v2-code/logo_NET.png) | [交互](msal-authentication-flows.md#interactive)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| 桌面（控制台）   | ![.NET/C#（桌面）](media/sample-v2-code/logo_NET.png) | [集成的 Windows 身份验证](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| 桌面（控制台）   | ![.NET/C#（桌面）](media/sample-v2-code/logo_NETcore.png) | [用户名/密码](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| 移动（Android、iOS、UWP）   | ![.NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [交互](msal-authentication-flows.md#interactive) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| 移动 (iOS)       | ![iOS / Objective C 或 swift](media/sample-v2-code/logo_iOS.png) | [交互](msal-authentication-flows.md#interactive) |[ios-swift-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| 移动 (Android)   | ![Android / Java](media/sample-v2-code/logo_Android.png) | [交互](msal-authentication-flows.md#interactive) |  [android-native-v2](https://github.com/azure-samples/active-directory-android-native-v2 ) |  |

## <a name="daemon-applications"></a>守护程序应用程序

下面的示例展示了一个应用程序，它使用自己的标识（没有用户）访问 Microsoft Graph API。

| 客户端应用程序 | 平台 | 流/授权 | 调用 Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| 控制台 | ![.NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [客户端凭据](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Web 应用 | ![ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [客户端凭据](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |

## <a name="headless-applications"></a>无外设应用程序

以下示例展示了在没有 Web 浏览器的设备上运行的公共客户端应用程序。 该应用可以是命令行工具，或者在 Linux/Mac 上运行，或者是 IoT 应用程序。 此示例提供了一个应用，以用户身份访问 Microsoft Graph API，该用户以交互方式在另一台设备上登录（例如移动电话）。 此客户端应用程序使用 MicroSoft 身份验证库 (MSAL)。

| 客户端应用程序 | 平台 | 流/授权 | 调用 Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| 桌面（控制台）   | ![.NET/C#（桌面）](media/sample-v2-code/logo_NETcore.png) | [设备代码流](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |

## <a name="web-apis"></a>Web API

下面的示例演示如何保护 web API 与 Microsoft 标识平台终结点。 此 API 由 WPF 应用程序执行，但可以由任何应用程序调用。 Web API 还会调用 Microsoft Graph。

| 平台 | 示例 |
| -------- | ------------------- |
| ![.NET/C#](media/sample-v2-code/logo_NET.png) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph) 的 WebAPI（服务） |

## <a name="other-microsoft-graph-samples"></a>其他 Microsoft Graph 示例

若要了解演示 Microsoft Graph API 的各种使用模式（包括向 Azure AD 进行身份验证）的[示例](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet)和教程，请参阅 [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples)（Microsoft Graph 社区示例和教程）。

## <a name="see-also"></a>另请参阅

- [Azure Active Directory (v1.0) 开发人员指南](v1-overview.md)
- [Azure AD 图形 API 概念和参考](https://msdn.microsoft.com/library/azure/hh974476.aspx)
- [Azure AD 图形 API 帮助程序库](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)
