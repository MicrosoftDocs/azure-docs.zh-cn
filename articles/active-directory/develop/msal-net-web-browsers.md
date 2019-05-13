---
title: 适用于.NET 的 web 浏览器中 Microsoft 身份验证库 |Azure
description: 使用 Xamarin Android Microsoft 身份验证库和.NET (MSAL.NET) 时，了解有关特定注意事项。
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
ms.date: 05/06/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d4b4c4cd4dbab10a9d4796a8393cc7f479b90cc4
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406767"
---
# <a name="using-web-browsers-in-msalnet"></a>在 MSAL.NET 中使用 web 浏览器
Web 浏览器所需的交互式身份验证。 默认情况下，支持 MSAL.NET[系统 web 浏览器](#system-web-browser-on-xamarinios-and-xamarinandroid)Xamarin.iOS 上和[Xamarin.Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/system-browser)。 但是[还可以启用嵌入式 Web 浏览器](#enable-embedded-webviews)具体取决于您的要求 （用户体验，实现单一登录 (SSO)、 安全需要） 进行[Xamarin.iOS](#choosing-between-embedded-web-browser-or-system-browser-on-xamarinios)并[Xamarin.Android](#choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid)应用程序。 甚至可以[动态选择](#detecting-the-presence-of-custom-tabs-on-xamarinandroid)哪些 web 浏览器使用基于是否存在 Chrome 或浏览器支持在 Android 中的 Chrome 自定义选项卡。

## <a name="web-browsers-in-msalnet"></a>MSAL.NET 中的 web 浏览器

请务必了解，当以交互方式获取令牌，不会提供对话框中的内容，由库，但由 STS （安全令牌服务）。 身份验证终结点发送回一些 HTML 和 JavaScript，控制交互过程，在 web 浏览器或 web 控件中呈现。 允许 STS 以处理 HTML 交互具有很多优点：

- 应用程序，也不身份验证库永远不会存储的密码 （如果其中一个已类型化）。
- 启用重定向到其他标识提供者 （例如登录名-在与工作学校帐户或个人帐户使用 MSAL，或使用与 Azure AD B2C 的社交帐户）。
- 允许控制条件性访问，例如，通过让在身份验证阶段 （输入 Windows Hello pin，或在电话上或其手机上的身份验证应用程序调用） 的用户执行多个身份验证 (MFA) 的 STS。 在其中需要的多重身份验证不它尚未设置的情况下，用户可以将其设置恰时相同的对话框中。  用户输入其手机号码，并指导来安装身份验证应用程序和扫描 QR 标记添加他们的帐户。 此服务器驱动的交互是很好的体验 ！
- 使用户可以更改此相同的对话框中其密码时的密码已过期 （提供旧密码和新密码的其他字段）。
- 启用受 Azure AD 租户管理员，在租户或应用程序 （映像） 的品牌 / 应用程序所有者。
- 使用户可以同意允许应用程序访问的资源/范围在身份验证后，只需在它们的名称。

## <a name="system-web-browser-on-xamarinios-and-xamarinandroid"></a>系统 web 浏览器上 Xamarin.iOS 和 Xamarin.Android

默认情况下，MSAL.NET Xamarin.iOS 和 Xamarin.Android 支持系统 web 浏览器。 用于提供 (即，不是.NET Core) 的用户界面的所有平台，由嵌入 Web 浏览器控件的库提供一个对话框。 MSAL.NET 还使用嵌入式的 web 视图进行.NET 桌面和 WAB 进行 UWP 平台。 但是，它利用默认情况下**系统 web 浏览器**适用于 Xamarin iOS 和 Xamarin Android 应用程序。 在 iOS 上，它甚至选择要具体取决于操作系统的版本使用的 web 视图 (iOS12，iOS11，及更早版本)。

使用系统浏览器具有的共享而无需 broker SSO 状态与其他应用程序以及 web 应用程序的一个明显优势 (公司门户 / 身份验证器)。 使用已在系统浏览器，默认情况下，在 Xamarin iOS 和 Xamarin Android 平台 MSAL.NET 因为、 在这些平台上系统 web 浏览器占据整个屏幕，，和用户体验是更好。 系统 web 视图不是一个对话框区分开来。 在 iOS 上，不过，用户可能需要浏览器回调也十分烦人的应用程序的同意。

### <a name="uwp-does-not-use-the-system-webview"></a>UWP 不使用 System Webview

对于桌面应用程序，但是，启动 System Webview 会导致比不上用户体验，因为该用户将看到浏览器中，其中它们可能已经有其他选项卡打开。 和用户时出现身份验证，获取一个页面，要求他们关闭此窗口。 如果用户未注意，它们可以关闭 （包括其他选项卡的身份验证不相关） 的整个过程。 利用在桌面上系统浏览器还需要打开本地端口和侦听，它可能需要高级的权限的应用程序。 您作为开发人员、 用户或管理员，可能是有关此要求不情愿。

## <a name="enable-embedded-webviews"></a>启用嵌入的 webviews 
此外可以启用 Xamarin.iOS 和 Xamarin.Android 应用程序中的嵌入的 webviews。 从开始 MSAL.NET 2.0.0-preview，MSAL.NET 还支持使用**嵌入**webview 选项。 对于 ADAL.NET，嵌入式 web 视图是支持的唯一选项。

作为使用 MSAL.NET 面向 Xamarin 的开发人员，您可以选择使用嵌入的 webview 或系统浏览器。 这是你选择，具体取决于你想要针对的用户体验和安全性问题。

目前，MSAL.NET 尚不支持 Android 和 iOS 代理人极其有用。 因此如果你需要提供单一登录 (SSO)，系统浏览器可能仍会将更好的选择。 支持使用嵌入式的 web 浏览器的代理是 MSAL.NET 积压工作上。

### <a name="differences-between-embedded-webview-and-system-browser"></a>嵌入式 web 视图和系统浏览器之间的差异 
有 MSAL.NET 嵌入式 web 视图和系统浏览器之间一些 visual 差异。

**交互式登录使用 MSAL.NET 使用嵌入式 web 视图：**

![嵌入](media/msal-net-web-browsers/embedded-webview.png)

**交互式登录使用 MSAL.NET 使用系统浏览器：**

![系统浏览器](media/msal-net-web-browsers/system-browser.png)

### <a name="developer-options"></a>开发人员选项

作为开发人员使用 MSAL.NET，有几个选项用于显示 STS 的交互式对话框：

- **系统浏览器。** 系统浏览器默认设置在库中。 如果使用 Android，请阅读[系统浏览器](msal-net-system-browser-android-considerations.md)进行身份验证支持哪些浏览器的特定信息。 当在 Android 中使用系统浏览器，我们建议该设备已支持 Chrome 自定义选项卡的浏览器。  否则，身份验证可能会失败。
- **嵌入式 web 视图。** 若要使用仅嵌入在 MSAL.NET，webview`AcquireTokenInteractively`参数生成器包含`WithUseEmbeddedWebView()`方法。

    iOS

    ```csharp
    AuthenticationResult authResult;
    authResult = app.AcquireTokenInteractively(scopes)
                    .WithUseEmbeddedWebView(useEmbeddedWebview)
                    .ExecuteAsync();
    ```

    Android：

    ```csharp
    authResult = app.AcquireTokenInteractively(scopes)
                .WithParentActivityOrWindow(activity)
                .WithUseEmbeddedWebView(useEmbeddedWebview)
                .ExecuteAsync();
    ```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinios"></a>嵌入式的 web 浏览器或系统在 Xamarin.iOS 上的浏览器之间进行选择

在 iOS 应用中，在`AppDelegate.cs`您可以初始化`ParentWindow`到`null`。 在 iOS 中不使用它

```csharp
App.ParentWindow = null; // no UI parent on iOS
```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid"></a>嵌入式的 web 浏览器或系统上 Xamarin.Android 的浏览器之间进行选择

在 Android 应用中，在`MainActivity.cs`可以设置父活动中，以便向其重新获取身份验证结果：

```csharp
 App.ParentWindow = this;
```

然后在`MainPage.xaml.cs`:

```csharp
authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(true)
                      .ExecuteAsync();
```

#### <a name="detecting-the-presence-of-custom-tabs-on-xamarinandroid"></a>检测存在自定义选项卡上 Xamarin.Android

如果你想要使用系统 web 浏览器在浏览器中运行的应用启用 SSO 但担心不具有自定义选项卡支持的浏览器的 Android 设备的用户体验，您可以选择决定通过调用`IsSystemWebViewAvailable()`中的方法 < c2 1> `IPublicClientApplication` 。 此方法返回`true`如果 PackageManager 检测到自定义选项卡和`false`如果它们未检测到在设备上。

根据此方法，而且您的要求返回的值，可以做出决策：

- 您可以自定义错误消息返回给用户。 例如:"请安装 Chrome 继续进行身份验证"-或-
- 可以故障回复到嵌入式 web 视图选项并启动作为嵌入式 web 视图 UI。

下面的代码显示了嵌入式 web 视图选项：

```csharp
bool useSystemBrowser = app.IsSystemWebviewAvailable();

authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(!useSystemBrowser)
                      .ExecuteAsync();
```

## <a name="net-core-does-not-support-interactive-authentication-out-of-the-box"></a>.NET core 不支持现成的交互式身份验证

对于.NET Core，令牌的获取以交互方式不可用。 实际上，.NET Core 不尚未提供 UI。 如果你想要为.NET Core 应用程序提供交互式登录，你可以让应用程序向用户显示一个代码和跳转以交互方式登录 URL (请参阅[设备代码流](msal-authentication-flows.md#device-code))。

或者可以实现[IWithCustomUI](scenario-desktop-acquire-token.md#withcustomwebui)接口，并提供你自己的浏览器