---
title: 初始化客户端应用程序 （适用于.NET 的 Microsoft 身份验证库） |Azure
description: 了解有关初始化公共客户端和使用.NET (MSAL.NET) 的 Microsoft 身份验证库的机密客户端应用程序。
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
ms.date: 04/12/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: c6197d472bdfaf03c9f99baa7691354e735cc91e
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075800"
---
# <a name="initialize-client-applications-using-msalnet"></a>初始化使用 MSAL.NET 的客户端应用程序
本指南介绍了初始化公共客户端和使用.NET (MSAL.NET) 的 Microsoft 身份验证库的机密客户端应用程序。  若要了解有关客户端应用程序类型和应用程序配置选项的详细信息，请阅读[概述](msal-client-applications.md)。

使用 MSAL.NET 3.x 中，实例化应用程序的建议的方法是通过使用应用程序构建者：`PublicClientApplicationBuilder`和`ConfidentialClientApplicationBuilder`。 它们可提供强大的机制来配置应用程序的代码，或从配置文件，或甚至通过混合使用这两种方法。

## <a name="prerequisites"></a>必备组件
在之前初始化应用程序，首先需要向[将其注册](quickstart-register-app.md)，以便您的应用程序可以与 Microsoft 标识平台集成。  注册后，可能需要以下信息 （这可在 Azure 门户中找到）：

- 客户端 ID （表示 GUID 的字符串）
- （命名实例） 的标识提供程序 URL 和应用程序的登录受众。 这两个参数统称为颁发机构。
- 如果你正在编写业务线应用程序仅为你的组织 （也名为单租户应用程序） 租户的 ID。
- 应用程序机密 （客户端机密字符串） 或类型的证书 （的 X509Certificate2） 如果它是机密客户端应用程序。
- 对于 web 应用，以及有时公共客户端应用程序 （尤其是当您的应用程序需要使用代理时），也将具有设置重定向 Uri 其中标识提供程序将联系后使用安全令牌的应用程序。

## <a name="ways-to-initialize-applications"></a>方法以初始化应用程序
有许多不同的方式来实例化客户端应用程序。

### <a name="initializing-a-public-client-application-from-code"></a>初始化来自代码的公共客户端应用程序

下面的代码实例化的公共客户端应用程序中，在 Microsoft Azure 公有云，与他们的工作和学校帐户，或者其个人 Microsoft 帐户登录用户。

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-code"></a>初始化代码中的机密客户端应用程序

同样，在下面的代码实例化机密的应用程序 (Web 应用位于`https://myapp.azurewebsites.net`) 处理从 Microsoft Azure 公有云，与他们的工作和学校帐户，或者其个人 Microsoft 帐户中的用户令牌。 应用程序标识通过共享的客户端机密的标识提供者：

```csharp
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

您可能知道，在生产环境中，而不使用客户端机密，您可能想要与 Azure AD 共用一个证书。 然后，代码应如下所示：

```csharp
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithRedirectUri(redirectUri )
    .Build();
```

### <a name="initializing-a-public-client-application-from-configuration-options"></a>正在初始化配置选项中的公共客户端应用程序

下面的代码实例化来自配置对象，这可能是已填写的以编程方式或从配置文件中读取的公共客户端应用程序：

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-configuration-options"></a>正在初始化配置选项中的机密客户端应用程序

相同类型的模式适用于机密客户端应用程序。 您还可以添加其他参数使用`.WithXXX`修饰符 （此处证书）。

```csharp
ConfidentialClientApplicationOptions options = GetOptions(); // your own method
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(options)
    .WithCertificate(certificate)
    .Build();
```

## <a name="builder-modifiers"></a>生成器修饰符

在代码片段中使用应用程序构建者的大量`.With`方法都可用作修饰符 (例如，`.WithCertificate`和`.WithRedirectUri`)。 

### <a name="modifiers-common-to-public-and-confidential-client-applications"></a>公共和机密客户端应用程序的公共修饰符

您可以设置公共客户端或机密客户端应用程序生成器修饰符有：

|参数 | 描述|
|--------- | --------- |
|`.WithAuthority()` 7 重写 | 到 Azure AD 颁发机构，并选择 Azure 云，受众，租户 （租户 ID 或域名称），或直接提供颁发机构 URI 可能设置了应用程序的默认权限。|
|`.WithAdfsAuthority(string)` | 设置了应用程序的默认权限为 ADFS 颁发机构。|
|`.WithB2CAuthority(string)` | 设置了应用程序的默认权限为 Azure AD B2C 颁发机构。|
|`.WithClientId(string)` | 重写客户端 id。|
|`.WithComponent(string)` | 设置使用 MSAL.NET （出于遥测原因） 的库的名称。 |
|`.WithDebugLoggingCallback()` | 如果调用，应用程序将调用`Debug.Write`只需启用调试跟踪。 请参阅[日志记录](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging)有关详细信息。|
|`.WithExtraQueryParameters(IDictionary<string,string> eqp)` | 设置的应用程序级别的额外查询参数，将所有身份验证请求中发送。 这是可重写在每个令牌获取方法级别 (具有相同`.WithExtraQueryParameters pattern`)。|
|`.WithHttpClientFactory(IMsalHttpClientFactory httpClientFactory)` | 可以让高级方案，例如配置 HTTP 代理，或若要强制 MSAL 为使用特定的 HttpClient （例如 ASP.NET Core web 应用/Api)。|
|`.WithLogging()` | 如果调用，该应用程序将调用具有调试跟踪的回调。 请参阅[日志记录](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging)有关详细信息。|
|`.WithRedirectUri(string redirectUri)` | 重写默认重定向 URI。 对于公共客户端应用程序，这会十分有用的方案涉及中转站。|
|`.WithTelemetry(TelemetryCallback telemetryCallback)` | 设置用于发送遥测数据的委托。|
|`.WithTenantId(string tenantId)` | 重写的租户 ID 或租户说明。|

### <a name="modifiers-specific-to-xamarinios-applications"></a>特定于 Xamarin.iOS 应用程序的修饰符

您可以设置在 Xamarin.iOS 上的公共客户端应用程序生成器修饰符有：

|参数 | 描述|
|--------- | --------- |
|`.WithIosKeychainSecurityGroup()` | **Xamarin.iOS 仅**:设置 iOS 密钥链安全组 （对于缓存暂留）。|

### <a name="modifiers-specific-to-confidential-client-applications"></a>机密客户端应用程序专用的修饰符

您可以设置机密客户端应用程序生成器修饰符有：

|参数 | 描述|
|--------- | --------- |
|`.WithCertificate(X509Certificate2 certificate)` | 设置用于标识与 Azure AD 应用程序的证书。|
|`.WithClientSecret(string clientSecret)` | 设置用于标识与 Azure AD 应用程序的客户端机密 （应用程序密码）。|

这些修饰符互相排斥。 如果你提供，MSAL 将引发有意义的异常。

### <a name="example-of-usage-of-modifiers"></a>修饰符的用法的示例

让我们假定您的应用程序为业务线应用程序，这是仅为你的组织。  然后，可以编写：

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzurePublic, tenantId)
        .Build();
```

它将变得有趣是国家/地区云编程现在简化了。 如果想要在国家/地区云的多租户应用程序应用程序，您可以编写，例如：

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment, AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

此外，还有 ADFS 的替代 （目前，不支持 ADFS 2019）：
```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

最后，如果你是 Azure AD B2C 开发人员，您可以指定你的租户如下：

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```
