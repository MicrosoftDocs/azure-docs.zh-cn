---
title: 通用 Windows 平台注意事项 （适用于.NET 的 Microsoft 身份验证库） |Azure
description: 使用通用 Windows 平台和 Microsoft 身份验证库.NET (MSAL.NET) 时，了解有关特定注意事项。
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
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: eb8076df5482c91942349e0a052794f3fe945a5f
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076505"
---
# <a name="universal-windows-platform-specific-considerations-with-msalnet"></a>使用 MSAL.NET 的通用 Windows 平台专属注意事项
Xamarin iOS 上有一些使用 MSAL.NET 时必须考虑的几个注意事项。

## <a name="the-usecorporatenetwork-property"></a>UseCorporateNetwork 属性
在 WinRT 平台中，`PublicClientApplication`具有以下布尔值属性``UseCorporateNetwork``。 此属性使 Win8.1 和 UWP 应用程序能够受益于集成 Windows 身份验证 （并因此与用户的 SSO 登录的操作系统） 如果用户是登录的帐户在联合 Azure AD 租户。 这利用 WAB （Web 身份验证代理）。 

> [!IMPORTANT]
> 此属性设置为 true，则假定应用程序开发人员已在应用程序中启用集成 Windows 身份验证 (IWA)。 为以下内容：
> - 在中``Package.appxmanifest``UWP 应用程序，在**功能**选项卡上，启用以下功能：
>   - 企业身份验证
>   - 专用网络(客户端和服务器)
>   - 共享的用户证书

IWA 未启用默认情况下，因为请求的企业身份验证或共享用户证书的功能的应用程序需要更高级别的验证到 Windows 应用商店中，接受并不是所有开发人员可能想要执行更高版本验证级别。 

在 UWP 平台 (WAB) 上的基础实现已启用条件访问的企业方案中都无法正常工作。 症状是用户尝试使用 Windows 你好，登录，并建议要选择未找到证书，但 pin 的证书，或用户选择它，但永远不会提示输入 Pin。 一种解决方法是使用一种替代方法 (用户名/密码 + 电话身份验证)，但不是好的体验。 

## <a name="next-steps"></a>后续步骤
下列示例中提供了更多详细信息：

示例 | 平台 | 描述 
|------ | -------- | -----------|
|[active-directory-dotnet-native-uwp-v2](https://github.com/azure-samples/active-directory-dotnet-native-uwp-v2) | UWP | 使用 msal.net，访问 Microsoft Graph 对用户进行身份验证与 Azure AD v2.0 终结点的通用 Windows 平台客户端应用程序。 <br>![拓扑](media/msal-net-uwp-considerations/topology-native-uwp.png)|
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/Azure-Samples/active-directory-xamarin-native-v2) | Xamarin iOS、Android、UWP | 简单的 Xamarin Forms 应用，它展示了如何使用 MSAL MSA 和 AAD v2.0 终结点，通过 Azure AD 进行身份验证和访问 Microsoft Graph 使用生成的令牌。 <br>![拓扑](media/msal-net-uwp-considerations/topology-xamarin-native.png)|