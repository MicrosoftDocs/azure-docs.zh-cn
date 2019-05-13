---
title: Xamarin iOS 注意事项 （适用于.NET 的 Microsoft 身份验证库） |Azure
description: 使用 Xamarin iOS 和 Microsoft 身份验证库.NET (MSAL.NET) 时，了解有关特定注意事项。
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
ms.openlocfilehash: bb07fa00b9e1d917cb64df18fff6466dd5c0193d
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406992"
---
# <a name="xamarin-ios-specific-considerations-with-msalnet"></a>使用 MSAL.NET Xamarin 特定于 iOS 的注意事项
在 Xamarin iOS 上有几个使用 MSAL.NET 时必须考虑的注意事项

- [IOS 12 和身份验证的已知的问题](#known-issues-with-ios-12-and-authentication)
- [重写并实现`OpenUrl`中的函数 `AppDelegate`](#implement-openurl)
- [启用密钥链组](#enable-keychain-groups)
- [启用令牌缓存共享](#enable-token-cache-sharing-across-ios-applications)
- [启用密钥链访问](#enable-keychain-access)

## <a name="known-issues-with-ios-12-and-authentication"></a>IOS 12 和身份验证的已知的问题
Microsoft 已发布[安全公告](https://github.com/aspnet/AspNetCore/issues/4647)以提供有关 iOS12 和某些类型的身份验证之间不兼容性信息。 不兼容性中断社会、 WSFed 和 OIDC 登录名。 此建议还指导开发人员可以执行的操作若要删除当前由 ASP.NET 添加到他们的应用程序变得与 iOS12 兼容的安全限制。  

在开发时 MSAL.NET Xamarin iOS 上的应用程序，尝试从 iOS 12 登录到网站时，可能会看到一个无限循环 (类似于以下[ADAL 问题](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/issues/1329)。 

您可能还会看到在 ASP.NET Core OIDC 身份验证中使用的 Io 12 中断 Safari 中所述[WebKit 问题](https://bugs.webkit.org/show_bug.cgi?id=188165)。

## <a name="implement-openurl"></a>实现 OpenUrl

首先您需要重写`OpenUrl`方法`FormsApplicationDelegate`派生类，并调用`AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs`。

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

您还需要定义 URL 方案，需要为你的应用调用另一个应用、 特定形式的重定向 URL，并且注册中的此重定向 URL 的权限[Azure 门户](https://portal.azure.com)。

## <a name="enable-keychain-groups"></a>启用密钥链组

为了使令牌缓存的工作，并具有`AcquireTokenSilentAsync`方法的工作，必须执行多个步骤：
1. 启用密钥链访问中的您*`* Entitlements.plist* 文件，并指定**密钥链组**中捆绑标识符。
2. 选择*`*Entitlements.plist*`* 中的文件**自定义权利**字段中 iOS 项目选项窗口**捆绑包签名视图**。
3. XCode 时签名证书，请确保使用相同的 Apple id。

## <a name="enable-token-cache-sharing-across-ios-applications"></a>启用在 iOS 应用程序之间共享的令牌缓存

从 MSAL 2.x 中，可以指定要用于将令牌缓存保留在多个应用程序的密钥链安全组。 这使您能够共享之间具有同一密钥链安全组包括与开发的多个应用程序的令牌缓存[ADAL.NET](https://aka.ms/adal-net)，MSAL.NET Xamarin.iOS 应用程序和开发的本机 iOS 应用程序与[ADAL.objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc)或[MSAL.objc](https://github.com/AzureAD/microsoft-authentication-library-for-objc))。

共享令牌缓存允许单一登录 (SSO) 之间的所有使用相同的密钥链安全组应用程序。

若要启用单一登录，需要设置`PublicClientApplication.iOSKeychainSecurityGroup`中所有应用程序的相同值的属性。

此示例使用 MSAL v3.x 应为：
```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.msalrocks")
     .Build();
```

此示例使用 MSAL v2.7.x 应为：

```csharp
PublicClientApplication.iOSKeychainSecurityGroup = "com.microsoft.msalrocks";
```

> [!NOTE]
> `KeychainSecurityGroup`属性已弃用。 以前，在 MSAL 2.x 中，开发人员不得不对其使用时包括团队 Id 前缀`KeychainSecurityGroup`属性。 
> 
> 现在，从 MSAL 2.7.x，MSAL 将解决在团队 Id 前缀在运行时期间使用时`iOSKeychainSecurityGroup`属性。 当使用此属性，值不应包含团队 Id 前缀。 
> 
> 使用新`iOSKeychainSecurityGroup`属性，它不需要开发人员能够提供 teamid 的值。 `KeychainSecurityGroup`属性现已过时。 

## <a name="enable-keychain-access"></a>启用密钥链访问

在 MSAL 2.x 和 ADAL 4.x teamid 的值用来访问密钥链，将启用身份验证库，以提供单一登录 (SSO) 的同一发布者的应用程序之间。 

什么是[TeamIdentifierPrefix](/xamarin/ios/deploy-test/provisioning/entitlements?tabs=vsmac) （teamid 的值）？ 它是在应用商店中的唯一标识符 （公司或个人）。 应用程序标识是唯一的应用程序。 如果有多个应用，适用于所有应用 teamid 的值将是相同的但 AppId 将会不同。 密钥链访问组加 teamid 的值自动为每个组由系统。 它是如何 OS 强制执行来自同一发布者的应用可以访问的共享密钥链。 

初始化时`PublicClientApplication`，如果你收到`MsalClientException`并显示消息： `TeamId returned null from the iOS keychain...`，将需要执行以下操作在 iOS 的 Xamarin 应用：

1. 在 VS 中调试选项卡，请转到 nameOfMyApp.iOS 属性...
2. 然后转到 iOS 捆绑签名 
3. 自定义权利下单击...从您的应用程序选择 Entitlements.plist 文件
4. 在 csproj 文件的 iOS 应用程序中，应包含现在包含以下行： `<CodesignEntitlements>Entitlements.plist</CodesignEntitlements>`
5. **重新生成**项目。

这是*此外*到启用密钥链访问中的`Entitlements.plist`文件，使用以下访问组或你自己：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>keychain-access-groups</key>
  <array>
    <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
  </array>
</dict>
</plist>
```

## <a name="next-steps"></a>后续步骤

中提供了更多详细信息[iOS 具体注意事项](https://github.com/azure-samples/active-directory-xamarin-native-v2#ios-specific-considerations)段落的以下示例的 readme.md 文件：

示例 | 平台 | 描述 
------ | -------- | -----------
[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS、Android、UWP | 简单的 Xamarin Forms 应用，它展示了如何使用 MSAL MSA 和 AAD V2.0 终结点，通过 Azure AD 进行身份验证和访问 Microsoft Graph 使用生成的令牌。 <br>![拓扑](media/msal-net-xamarin-ios-considerations/topology.png)