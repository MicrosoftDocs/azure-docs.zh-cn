---
title: 实例化包含选项 （适用于.NET 的 Microsoft 身份验证库） 的机密客户端应用程序 |Azure
description: 了解如何使用配置选项使用 Microsoft 身份验证库.NET (MSAL.NET) 实例化机密客户端应用程序。
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
ms.date: 04/30/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 60b3b2fa2cdb7808e45f9142f1c1a351f470af50
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65158830"
---
# <a name="instantiate-a-confidential-client-application-with-configuration-options-using-msalnet"></a>使用配置选项，使用 MSAL.NET 实例化机密客户端应用程序

本文介绍如何实例化[机密客户端应用程序](msal-client-applications.md)使用 Microsoft 身份验证库.NET (MSAL.NET)。  使用设置文件中定义的配置选项实例化应用程序。

在之前初始化应用程序，首先需要向[注册](quickstart-register-app.md)它，以便您的应用程序可以与 Microsoft 标识平台集成。 注册后，可能需要以下信息 （这可在 Azure 门户中找到）：

- 客户端 ID （表示 GUID 的字符串）
- （命名实例） 的标识提供程序 URL 和应用程序的登录受众。 这两个参数统称为颁发机构。
- 如果你正在编写业务线应用程序仅为你的组织 （也名为单租户应用程序） 租户的 ID。
- 应用程序机密 （客户端机密字符串） 或类型的证书 （的 X509Certificate2） 如果它是机密客户端应用程序。
- 对于 web 应用，以及有时公共客户端应用程序 （尤其是当您的应用程序需要使用代理时），也将具有设置重定向 Uri 其中标识提供程序将联系后使用安全令牌的应用程序。

## <a name="configure-the-application-from-the-config-file"></a>配置配置文件中的应用程序
MSAL.NET 中选项的属性的名称匹配的属性的名称`AzureADOptions`在 ASP.NET Core 中，因此你无需编写任何粘附代码。

中介绍了 ASP.NET Core 应用程序配置*appsettings.json*文件：

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "[Enter the domain of your tenant, e.g. contoso.onmicrosoft.com]",
    "TenantId": "[Enter 'common', or 'organizations' or the Tenant Id (Obtained from the Azure portal. Select 'Endpoints' from the 'App registrations' blade and use the GUID in any of the URLs), e.g. da41245a5-11b3-996c-00a8-4d99re19f292]",
    "ClientId": "[Enter the Client Id (Application ID obtained from the Azure portal), e.g. ba74781c2-53c2-442a-97c2-3d60re42f403]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc",

    "ClientSecret": "[Copy the client secret added to the app from the Azure portal]"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

从开始 MSAL.NET v3.x，可以配置从配置文件将机密客户端应用程序。 与应用程序配置相关的类都位于`Microsoft.Identity.Client.AppConfig`命名空间。

在要配置和实例化您的应用程序的类，您需要声明`ConfidentialClientApplicationOptions`对象。  绑定到应用程序选项的实例 （包括 appconfig.json 文件） 从源中读取的配置：

```csharp
using Microsoft.Identity.Client.AppConfig;

private ConfidentialClientApplicationOptions _applicationOptions;
_applicationOptions = new ConfidentialClientApplicationOptions();
configuration.Bind("AzureAD", _applicationOptions);
```

这使"AzureAD"部分中的内容*appsettings.json*要绑定到相应的属性文件`ConfidentialClientApplicationOptions`对象。  接下来，生成`ConfidentialClientApplication`对象：

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
        .Build();
```

## <a name="add-runtime-configuration"></a>添加运行时配置
在机密客户端应用程序中，通常具有每个用户的缓存。 因此需要获取与用户相关联的缓存，并通知应用程序生成器你想要使用它。 在相同的方式，您可能动态计算出的重定向 URI。 在这种情况下将为以下代码：

```csharp
IConfidentialClientApplication app;
var request = httpContext.Request;
var currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, _azureAdOptions.CallbackPath ?? string.Empty);
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
       .WithRedirectUri(currentUri)
       .Build();
TokenCache userTokenCache = _tokenCacheProvider.SerializeCache(app.UserTokenCache,httpContext, claimsPrincipal);
```

