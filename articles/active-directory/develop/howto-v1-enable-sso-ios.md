---
title: 如何使用 ADAL 在 iOS 上启用跨应用 SSO | Microsoft Docs
description: 如何使用 ADAL SDK 的功能跨应用程序启用单一登录。
services: active-directory
author: CelesteDG
manager: mtillman
ms.assetid: d042d6da-7503-4e20-bb55-06917de01fcd
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: brandwe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 73d3c2846ed97b725d745cc76714738ac34ed1a0
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190210"
---
# <a name="how-to-enable-cross-app-sso-on-ios-using-adal"></a>如何：使用 ADAL 在 iOS 上启用跨应用 SSO

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

单一登录 (SSO) 允许用户仅输入一次其凭据，并让这些凭据跨应用程序以及跨其他应用程序可能使用的平台（例如 Microsoft 帐户，或 Microsoft 365 中的工作帐户）自动工作，无论发行商是谁。

使用 Microsoft 的标识平台以及各种 SDK，可以轻松在你自己的应用套件中启用 SSO，或者使用中转站功能和 Authenticator 应用程序在整个设备中启用 SSO。

在本操作指南中，你将学习如何在应用程序中配置 SDK，以便向客户提供 SSO。

本操作指南适用于：

* Azure Active Directory (Azure Active Directory)
* Azure Active Directory B2C
* Azure Active Directory B2B
* Azure Active Directory 条件访问

## <a name="prerequisites"></a>必备组件

本操作指南假定你知道如何：

* 使用 Azure AD 的旧门户预配你的应用。 有关详细信息，请参阅[注册应用](quickstart-register-app.md)
* 将你的应用程序与 [Azure AD iOS SDK](https://github.com/AzureAD/azure-activedirectory-library-for-objc) 进行集成。

## <a name="single-sign-on-concepts"></a>单一登录概念

### <a name="identity-brokers"></a>标识中转站

Microsoft 为每个移动平台提供了应用程序，可在来自不同供应商的应用程序之间桥接凭据，并支持需要一个安全位置来验证凭据的特殊增强功能。 这些被称为**中转站**。

在 iOS 和 Android 平台上，中转站通过可下载的应用程序提供，客户可以独立安装这些应用程序，也可以由负责管理其员工的部分或全部设备的公司推送到设备。 中转站支持仅为某些应用程序管理安全性，或者为整个设备管理安全性，具体取决于 IT 管理配置。 在 Windows 中，此功能由内置于操作系统的帐户选择器提供，在技术上被称为 Web 身份验证中转站。

### <a name="patterns-for-logging-in-on-mobile-devices"></a>在移动设备上登录的模式

在设备上访问凭据遵循两种基本模式：

* 非中转站辅助的登录
* 中转站辅助的登录

#### <a name="non-broker-assisted-logins"></a>无中转站辅助的登录

非中转站的辅助登录是应用程序内联的登录体验，使用该应用程序的设备上的本地存储。 此存储可以跨应用程序共享，但凭据紧密绑定到应用或使用该凭据的应用套件。 你最有可能在许多移动应用程序中有过此体验，当你在应用程序本身中输入用户名和密码时。

这种登录具有以下优点：

* 完全在应用程序存在的用户体验。
* 凭据可跨应用程序共享并由同一个证书签名，为应用程序套件提供单一登录体验。
* 围绕体验中的日志记录的控件提供给应用程序之前和之后登录。

这种登录具有以下缺点：

* 用户不能在使用 Microsoft 标识的所有应用上体验单一登录，而只能在应用程序已配置的这些 Microsoft 标识上体验单一登录。
* 应用程序不能用于更高级的业务功能，如条件访问，或使用 InTune 产品套件。
* 应用程序不能支持业务用户使用基于证书的身份验证。

下面介绍了 SDK 如何与应用程序的共享存储配合工作以启用 SSO：

```
+------------+ +------------+  +-------------+
|            | |            |  |             |
|   App 1    | |   App 2    |  |   App 3     |
|            | |            |  |             |
|            | |            |  |             |
+------------+ +------------+  +-------------+
| ADAL SDK  |  |  ADAL SDK  |  |  ADAL SDK   |
+------------+-+------------+--+-------------+
|                                            |
|            App Shared Storage              |
+--------------------------------------------+
```

#### <a name="broker-assisted-logins"></a>中转站辅助的登录

中转站辅助的登录是一种发生在中转站应用程序中的登录体验，它使用中转站的存储和安全性在设备上应用了标识平台的所有应用程序间共享凭据。 这意味着，应用程序依赖于中转站才能将用户登录。 在 iOS 和 Android 平台上，通过可下载的应用程序提供这些中转站。可以由客户独立安装这些应用程序，也可以由负责管理其用户设备的公司将这些应用程序推送到设备。 这种类型的应用程序示例是 iOS 上的 Microsoft Authenticator 应用程序。 在 Windows 内置于操作系统，已知技术作为 Web 身份验证中转站帐户选择器提供此功能。

具体的体验因平台而异，如果未正确管理，有时会给用户带来麻烦。 如果已安装 Facebook 应用程序并在另一个应用程序中使用 Facebook Connect，则可能很熟悉这种模式。 标识平台使用相同的模式。

对于 iOS，这会导致“过渡”动画，其中，应用程序将发送到后台，而 Microsoft Authenticator 应用程序将发送到前台，让用户选择他们登录时使用的帐户。

对于 Android 和 Windows，帐户选择器会显示在应用程序顶部，可减少对用户的干扰。

#### <a name="how-the-broker-gets-invoked"></a>如何调用中转站

如果在设备上安装了 Microsoft Authenticator 应用程序等兼容的中转站，当用户指出他们希望在标识平台上使用任何帐户登录时，SDK 会自动调用中转站。 此帐户可以是个人 Microsoft 帐户、工作或学校帐户，也可以是由你提供并使用我们的 B2C 和 B2B 产品托管在 Azure 中的帐户。

#### <a name="how-we-ensure-the-application-is-valid"></a>我们如何确保应用程序有效

确保调用中转站的应用程序标识有效的需求对于我们在中转站辅助的登录中提供的安全性至关重要。 iOS 和 Android 都不会强制实施仅对给定应用程序有效的唯一标识符，因此恶意应用程序可能“假冒”合法应用程序的标识符，并接收对合法应用程序适用的令牌。 为了确保在运行时我们始终与适当的应用程序进行通信，我们会要求开发人员在将其应用程序注册到 Microsoft 时提供自定义重定向 URI。 下面详细讨论了开发人员应如何设计此重定向 URI。 此自定义重定向 URI 包含应用程序的捆绑 ID，并由 Apple App Store 确保其对应用程序是唯一的。 当应用程序调用中转站时，中转站会请求 iOS 操作系统为其提供调用中转站的捆绑 ID。 在调用我们的标识系统时，中转站会向 Microsoft 提供此捆绑 ID。 如果应用程序的捆绑 ID 与开发人员在注册期间提供给我们的捆绑 ID 不匹配，我们将拒绝应用程序访问所请求的资源的令牌。 此检查可确保只有开发人员注册的应用程序收到令牌。

**开发人员可以选择 SDK 是调用中转站，还是使用非中转站辅助的流。** 但是，如果开发人员选择不使用中转站辅助的流，他们会失去使用用户可能已添加到设备的 SSO 凭据的优势，并阻止其应用程序使用 Microsoft 为客户提供的业务功能，例如条件访问、Intune 管理功能和基于证书的身份验证。

这种登录具有以下优点：

* 无论供应商是谁，用户都可以在所有应用程序中体验 SSO。
* 应用程序可以使用条件访问等更高级的业务功能，也可使用 Intune 产品套件。
* 应用程序可以支持业务用户使用基于证书的身份验证。
* 登录体验要安全得多，因为中转站应用程序将使用附加的安全算法和加密来验证应用程序与用户的标识。

这种登录具有以下缺点：

* 在 iOS 中用户时选择了凭据，则利用应用程序体验转换。
* 管理客户应用程序中的登录体验的能力的丢失。

下面介绍了该 SDK 如何与应用程序的中转站应用程序配合工作以启用 SSO：

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    Else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
| Azure SDK  | | Azure SDK  |   | Azure SDK   |
+-----+------+-+-----+------+-  +-------+-----+
      |              |                  |
      |       +------v------+           |
      |       |             |           |
      |       | Microsoft   |           |
      +-------> Broker      |^----------+
              | Application
              |             |
              +-------------+
              |             |
              |   Broker    |
              |   Storage   |
              |             |
              +-------------+
```

## <a name="enabling-cross-app-sso-using-adal"></a>使用 ADAL 启用跨应用 SSO

这里，我们使用 ADAL iOS SDK 执行以下操作：

* 为应用套件打开非中转站辅助的 SSO
* 打开对中转站辅助 SSO 的支持

### <a name="turning-on-sso-for-non-broker-assisted-sso"></a>打开对非中转站辅助 SSO 的 SSO

对于跨应用程序的非中转站辅助 SSO，使用 SDK 可以大大消除 SSO 的复杂性。 这包括在缓存中查找适当的用户，以及维护登录用户的列表供你查询。

要跨你拥有的应用程序启用 SSO，需要执行以下操作：

1. 确保所有应用程序使用的相同客户端 ID 或应用程序 id。
2. 确保所有应用程序共享来自 Apple 的相同签名证书，以便可以共享密钥链。
3. 请求每个应用程序的相同密钥链授权。
4. 告知 SDK 要使用的共享密钥链。

#### <a name="using-the-same-client-id--application-id-for-all-the-applications-in-your-suite-of-apps"></a>对应用套件中的所有应用程序使用相同的客户端 ID/应用程序 ID

为了让标识平台知道可以跨应用程序共享令牌，每个应用程序需要共享同一个客户端 ID 或应用程序 ID。 这是在门户中注册第一个应用程序时提供的唯一标识符。

如果应用使用相同的应用程序 ID，则重定向 URI 可以用来在 Microsoft 标识服务中标识不同的应用。 每个应用程序可以在登记门户中注册多个重定向 URI。 套件中的每个应用程序具有不同的重定向 URI。 下面显示了这种情况的示例：

App1 重定向 URI：`x-msauth-mytestiosapp://com.myapp.mytestapp`

App2 重定向 URI：`x-msauth-mytestiosapp://com.myapp.mytestapp2`

App3 重定向 URI：`x-msauth-mytestiosapp://com.myapp.mytestapp3`

....

这些应用嵌套在同一个客户端 ID/应用程序 ID 下，可以根据你在 SDK 配置中返回给我们的重定向 URI 来查找。

```
+-------------------+
|                   |
|  Client ID        |
+---------+---------+
          |
          |           +-----------------------------------+
          |           |  App 1 Redirect URI               |
          +----------^+                                   |
          |           +-----------------------------------+
          |
          |           +-----------------------------------+
          +----------^+  App 2 Redirect URI               |
          |           |                                   |
          |           +-----------------------------------+
          |
          +----------^+-----------------------------------+
                      |  App 3 Redirect URI               |
                      |                                   |
                      +-----------------------------------+

```

下面介绍了这些重定向 URI 的格式。 可以使用任何重定向 URI，除非你想要支持中转站，在这种情况下，它们必须如上所示*

#### <a name="create-keychain-sharing-between-applications"></a>创建在应用程序之间共享的密钥链

启用密钥链共享超出了本文档的范围，Apple 文档 [Adding Capabilities](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html)（添加功能）中作了具体介绍。 重要的是，需要决定密钥链的调用方式，并将该功能添加到所有应用程序。

如果正确设置了授权，应在项目目录中看到标题为 `entitlements.plist` 的文件，其中包含类似如下的内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.myapp.mytestapp</string>
        <string>$(AppIdentifierPrefix)com.myapp.mycache</string>
    </array>
</dict>
</plist>
```

后必须在每个应用程序中启用密钥链授权，并且已准备好使用 SSO，请告知标识 SDK 密钥链通过使用以下设置你`ADAuthenticationSettings`使用以下设置：

```
defaultKeychainSharingGroup=@"com.myapp.mycache";
```

> [!WARNING]
> 在应用程序之间共享密钥链之后，任何应用程序都可以删除用户，更糟的是，删除整个应用程序的所有令牌。 如果应用程序依赖于这些令牌来执行后台工作，则这是特别严重的后果。 要共享密钥链，就必须十分警惕通过标识 SDK 执行的任意和所有删除操作。

就这么简单！ 该 SDK 现在会在所有应用程序之间共享凭据。 此外还会在应用程序实例之间共享用户列表。

### <a name="turning-on-sso-for-broker-assisted-sso"></a>打开对中转站辅助 SSO 的 SSO

应用程序使用安装在设备上的任何中转站的功能**默认已关闭**。 要向中转站使用应用程序必须执行一些额外配置，并将一些代码添加到应用程序。

要遵循的步骤如下：

1. 启用应用程序代码调用到 MS SDK 中的中转站模式。
2. 建立新的重定向 URI，并为应用程序和应用程序的注册提供程序。
3. 正在注册 URL 方案。
4. 将权限添加到 info.plist 文件。

#### <a name="step-1-enable-broker-mode-in-your-application"></a>步骤 1：在应用程序中启用中转站模式

创建“上下文”或身份验证对象的初始安装时，应用程序使用了中转站的功能被打开的。 通过在代码中设置凭据类型中执行此操作：

```
/*! See the ADCredentialsType enumeration definition for details */
@propertyADCredentialsType credentialsType;
```
`AD_CREDENTIALS_AUTO` 设置允许 SDK 尝试调用中转站，而 `AD_CREDENTIALS_EMBEDDED` 阻止 SDK 调用中转站。

#### <a name="step-2-registering-a-url-scheme"></a>步骤 2：注册 URL 方案

标识平台使用 URL 来调用中转站，然后将控制权返回给应用程序。 要完成这种往返过程，需要为应用程序注册一个标识平台所知的 URL 方案。 此方案可以不同于以前注册到应用程序的其他应用方案。

> [!WARNING]
> 我们建议将 URL 方案保持相当高的独特性，以尽量避免其他应用使用同一个 URL 方案。 Apple 不强制在应用商店中注册的 URL 方案的唯一性。

下面是在项目配置中的显示方式示例。 也可以在 XCode 中执行此操作：

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.myapp.mytestapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>x-msauth-mytestiosapp</string>
        </array>
    </dict>
</array>
```

#### <a name="step-3-establish-a-new-redirect-uri-with-your-url-scheme"></a>步骤 3：使用 URL 方案建立新的重定向 URI

为了确保我们始终返回凭据令牌到正确的应用程序，我们需要确保我们回调到应用程序可以验证 iOS 操作系统的方式。 iOS 操作系统会向 Microsoft 中转站应用程序报告正在调用它的应用程序的捆绑 ID。 此证书不能受到恶意应用程序的欺骗。 因此，我们会利用这以及中转站应用程序以确保令牌返回到正确的应用程序的 URI。 我们要求在应用程序中建立此唯一重定向 URI 这两个并为我们的开发人员门户中的重定向 URI 进行设置。

重定向 URI 必须采用正确的格式：

`<app-scheme>://<your.bundle.id>`

例如：*x-msauth-mytestiosapp://com.myapp.mytestapp*

需要使用 [Azure 门户](https://portal.azure.com/)在应用注册中指定此重定向 URI。 有关 Azure AD 应用注册的详细信息，请参阅[与 Azure Active Directory 集成](active-directory-how-to-integrate.md)。

##### <a name="step-3a-add-a-redirect-uri-in-your-app-and-dev-portal-to-support-certificate-based-authentication"></a>步骤 3a：在应用和开发门户添加重定向 URI，以支持基于证书的身份验证

若要支持基于证书的身份验证，需要在应用程序和 [Azure 门户](https://portal.azure.com/)中再注册一个“msauth”，用于处理证书身份验证（如果想要在应用程序中添加该支持）。

`msauth://code/<broker-redirect-uri-in-url-encoded-form>`

例如：*msauth://code/x-msauth-mytestiosapp%3A%2F%2Fcom.myapp.mytestapp*

#### <a name="step-4-add-a-configuration-parameter-to-your-app"></a>步骤 4：将配置参数添加到应用

ADAL 使用 -canOpenURL: 来检查是否在设备上安装了中转站。 在 iOS 9 中，Apple 锁定了应用程序可以查询的方案。 需要将“msauth”添加到 `info.plist file` 的 LSApplicationQueriesSchemes 节。

```
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauth</string>
    </array>

```

### <a name="youve-configured-sso"></a>已配置 SSO！

现在，标识 SDK 会自动在应用程序之间共享凭据并调用中转站（如果在设备上存在）。

## <a name="next-steps"></a>后续步骤

* 了解[单一登录 SAML 协议](single-sign-on-saml-protocol.md)
