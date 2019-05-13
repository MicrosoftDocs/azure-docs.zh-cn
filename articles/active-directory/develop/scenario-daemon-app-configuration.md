---
title: 守护程序应用程序调用 web Api （应用程序配置）-Microsoft 标识平台
description: 了解如何构建守护程序应用调用 web Api （应用程序配置）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d8d377db827a6548c380128624c21f4ae7896aff
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075320"
---
# <a name="daemon-app-that-calls-web-apis---code-configuration"></a>守护程序应用程序调用 web Api 的代码配置

了解如何配置守护程序应用程序调用 web Api 的代码。

## <a name="msal-libraries-supporting-daemon-apps"></a>MSAL 库支持守护程序应用

支持守护程序应用的 Microsoft 库包括：

  MSAL 库 | 描述
  ------------ | ----------
  ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | 支持的平台构建守护程序应用程序是.NET Framework 和.NET Core 平台 （不 UWP、 Xamarin.iOS 和 Xamarin.Android 为这些平台用于生成公共客户端应用程序）
  ![Python](media/sample-v2-code/logo_python.png) <br/> MSAL.Python | 进度-公共预览版中的开发
  ![Java](media/sample-v2-code/logo_java.png) <br/> MSAL.Java | 进度-公共预览版中的开发

## <a name="configuration-of-the-authority"></a>颁发机构的配置

考虑到守护程序应用程序不使用委派的权限，但应用程序权限，其*受支持的帐户类型*不能为*任何组织的目录中的帐户和个人 Microsoft 帐户 （例如，Skype，Xbox、 Outlook.com)*。 实际上，是没有租户管理员同意 Microsoft 个人帐户的守护程序应用程序。 将需要选择*我的组织中的帐户*或*任何组织中的帐户*。

因此在应用程序配置中指定的颁发机构应租户 ed （指定租户 ID 或与组织关联的域名）。 如果你是 ISV 并且想要提供多租户工具，则可以使用`organizations`。 但请记住，您还需要向客户说明如何授予管理员同意。 请参阅[请求整个租户的许可](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant)有关详细信息

## <a name="application-configuration-and-instantiation"></a>应用程序配置和实例化

在 MSAL 库中，客户端凭据 （密码或证书） 作为机密客户端应用程序构造的一个参数传递。

> [!IMPORTANT]
> 即使你的应用程序是控制台应用程序的后台应用程序是否运行作为一种服务，它需要机密客户端应用程序。

### <a name="msalnet"></a>MSAL.NET

添加[Microsoft.IdentityClient](https://www.nuget.org/packages/Microsoft.Identity.Client)到你的应用程序的 NuGet 包。

使用 MSAL.NET 命名空间

```CSharp
using Microsoft.Identity.Client;
```

后台应用程序将主讲 `IConfidentialClientApplication`

```CSharp
IConfidentialClientApplication app;
```

下面是生成与应用程序密码的应用程序的代码：

```CSharp
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
           .WithClientSecret(config.ClientSecret)
           .WithAuthority(new Uri(config.Authority))
           .Build();
```

下面是生成的证书的应用程序的代码：

```CSharp
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
    .WithCertificate(certificate)
    .WithAuthority(new Uri(config.Authority))
    .Build();
```

### <a name="msalpython"></a>MSAL.Python

```Python
# Create a preferably long-lived app instance which maintains a token cache.

app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential=config["secret"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache

    )
```

### <a name="msaljava"></a>MSAL.Java

```Java
PrivateKey key = getPrivateKey();
X509Certificate publicCertificate = getPublicCertificate();

// create clientCredential with public and private key
IClientCredential credential = ClientCredentialFactory.create(key, publicCertificate);

ConfidentialClientApplication cca = ConfidentialClientApplication
  .builder(CLIENT_ID, credential)
  .authority(AUTHORITY_MICROSOFT)
  .build();
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [守护程序应用-获取令牌的应用](./scenario-daemon-acquire-token.md)