---
title: 客户端应用程序配置 （Microsoft 身份验证库） |Azure
description: 了解应用程序中 Microsoft 身份验证库 (MSAL) 公共客户端和机密客户端的配置选项。
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
ms.openlocfilehash: 3a48eea9fedd2d82f44693d58b31ee0d5c8c288d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138534"
---
# <a name="application-configuration-options"></a>应用程序配置选项

在代码中，初始化新的公共或机密客户端 （或 MSAL.js 的用户代理） 应用程序进行身份验证并获取令牌。  有多种不同的配置初始化 MSAL 中的客户端应用程序时可以设置的选项。 这些选项可分为两个组：

- 注册选项，包括：
    - [颁发机构](#authority)(标识提供程序的组成[实例](#cloud-instance)和登录[受众](#application-audience)针对应用程序，或租户 ID)。
    - [客户端 ID](#client-id)
    - [重定向 URI](#redirect-uri)
    - [客户端机密](#client-secret)（针对机密客户端应用程序）。
- [日志记录选项](#logging)，包括日志级别、 个人数据的控件和组件使用库的名称。

## <a name="authority"></a>颁发机构
颁发机构是一个 URL，该值指示 MSAL 可以请求令牌从一个目录。 常用的颁发机构是：

- https://login.microsoftonline.com/&lt租户&gt;/，其中&lt;租户&gt;是 Azure AD 租户或域与此 Azure AD 租户相关联的租户 ID。  仅用于特定组织的用户登录。
- https://login.microsoftonline.com/common/。 用于在用户的工作和学校帐户或 Microsoft 个人帐户登录。
- https://login.microsoftonline.com/organizations/。 用于在使用用户的工作和学校帐户登录。
- https://login.microsoftonline.com/consumers/。 用于在具有仅个人 Microsoft 帐户 (live) 的用户登录。

颁发机构设置必须与在应用程序注册门户中声明的内容一致。

机构 URL 组成的实例和受众。

颁发机构可以是：
- Azure Active directory 云证书颁发机构
- Azure AD B2C 颁发机构。 请参阅[B2C 细节](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics)。
- ADFS 颁发机构。 请参阅[ADFS 支持](https://aka.ms/msal-net-adfs-support)。

Azure AD 云颁发机构具有两个部分：
- 标识提供者**实例**
- 单一登录**受众**应用程序

可以连接在一起并作为颁发机构 URL 提供的实例和受众。 在版本的 MSAL 之前 MSAL.NET 3.x 中，您必须自己依赖于您希望目标，并在登录受众云组合了的权限。  下图显示了如何构成的颁发机构 URL。

![颁发机构](media/msal-client-application-configuration/authority.png)

## <a name="cloud-instance"></a>云实例
**实例**用来指定你的应用程序签名从 Microsoft Azure 公有云，或从国家/地区云的用户。 在代码中使用 MSAL，Azure 云实例可以设置使用枚举或通过将传递到 URL[国家/地区云实例](authentication-national-cloud.md#azure-ad-authentication-endpoints)作为`Instance`成员 （如果知道）。

如果这两个，MSAL.NET 将引发的显式异常`Instance`和`AzureCloudInstance`指定。 

如果未指定实例，您的应用程序将面向 Azure 公有云实例 (实例 URL `https://login.onmicrosoftonline.com`)。

## <a name="application-audience"></a>应用程序访问群体

在登录受众取决于你的应用程序的业务需求：
- 如果您是业务线 (LOB) 开发人员的行，将可能产生单租户应用程序，将仅在你的组织中使用。 在这种情况下，您需要指定此组织是什么，通过其租户 ID (ID 的 Azure Active Directory) 或与此 Azure Active Directory 关联的域名。
- 如果你是 ISV，你可能想要登录用户使用其工作和学校帐户中任何组织中，或某些组织 （多租户应用程序），但您可能还想要让用户使用其 Microsoft 个人帐户登录。

### <a name="how-to-specify-the-audience-in-your-codeconfiguration"></a>如何在你的代码/配置中指定的受众
在代码中使用 MSAL，通过使用指定的受众：
- 是 Azure AD 颁发机构的受众的枚举。 
- 或租户 ID，这可以是：
  - 一个 GUID，(Azure Active Directory 的 ID) 的单租户应用程序
  - 与 Azure Active Directory （也适用于单租户应用程序） 相关联的域名
- 或其中一个作为租户 ID 来替代 Azure AD 颁发机构受众枚举这些占位符：
    - `organizations` 对于多租户应用程序
    - `consumers` 若要仅使用其个人帐户的用户登录
    - `common` 若要在用户使用其工作和学校帐户或 Microsoft 个人帐户登录

MSAL 将引发有意义的异常，如果指定 Azure AD 颁发机构受众和租户 id。 

如果未指定受众应用的目标 Azure AD 和作为受众的个人 Microsoft 帐户 (即`common`)。

### <a name="effective-audience"></a>有效的受众
你的应用程序的有效受众将最小值 （如果没有交集） 的应用程序中设置的用户和应用程序注册中指定的受众。 实际上，应用程序注册体验 ([应用注册](https://aka.ms/appregistrations)) 允许您指定应用程序的受众 （支持的帐户类型）。 请参阅[快速入门：注册一个应用程序的 Microsoft 标识平台](quickstart-register-app.md)有关详细信息。

目前，在使用 Microsoft 个人帐户的用户登录应用程序的唯一方法是设置：
- 将应用注册受众设置为"工作和学校帐户和个人帐户"，
- 和在代码中设置受众 / 配置到`AadAuthorityAudience.PersonalMicrosoftAccount`(或`TenantID `="使用者")

## <a name="client-id"></a>客户端 ID
由 Azure AD 注册应用程序时分配给您的应用程序的唯一的应用程序 （客户端） ID。

## <a name="redirect-uri"></a>重定向 URI
重定向 URI 是标识提供程序将返回的安全令牌的 URI。 

### <a name="redirect-uri-for-public-client-applications"></a>公共客户端应用程序的重定向 URI
公共客户端应用程序开发人员使用 MSAL:
- 不需要传递``RedirectUri``自动计算出的 MSAL。 此重定向 URI 设置为具体取决于平台的以下值：

- ``urn:ietf:wg:oauth:2.0:oob`` 所有 Windows 平台
- ``msal{ClientId}://auth`` 适用于 Xamarin Android 和 iOS

但是，重定向 URI 需要为其配置中[应用注册](https://aka.ms/appregistrations)。

![在门户中的重定向 URI](media/msal-client-application-configuration/redirect-uri.png)

可以重写重定向 URI 使用`RedirectUri`属性，例如如果你使用代理。 下面这种情况下是重定向 Uri 的一些示例：

- `RedirectUriOnAndroid` = "msauth-5a434691-ccb2-4fd1-b97b-b64bcfbc03fc://com.microsoft.identity.client.sample";
- `RedirectUriOnIos` = $"msauth.{Bundle.ID}://auth";

有关详细信息，请参阅 Android 细节和[iOS 具体信息](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Leveraging-the-broker-on-iOS)

### <a name="redirect-uri-for-confidential-client-applications"></a>机密客户端应用程序的重定向 URI
对于 web 应用，重定向 URI （或回复 URI） 是在其 Azure AD 将联系返回具有令牌的应用程序的 URI。 这可以是 Web 应用程序的 URL / Web API 如果机密就是其中之一。  此重定向 URI，需要在应用注册中注册。 部署最初本地测试应用程序时，这一点尤其重要。 然后需要在应用程序注册门户中添加已部署应用程序的答复 URL。

对于后台程序应用程序，不需要指定重定向 URI。

## <a name="client-secret"></a>客户端机密
机密客户端应用程序客户端密码。 此密钥 （应用程序密码） 为提供的应用程序注册门户，或到 Azure AD 应用程序注册期间提供与 azure Ad PowerShell、 PowerShell AzureRM 或 Azure CLI。

## <a name="logging"></a>日志记录
其他选项启用日志记录和故障排除。 有关详细信息，请参阅[日志记录](msal-logging.md)有关如何使用它们的详细信息页。

## <a name="next-steps"></a>后续步骤
了解如何[实例化客户端应用程序使用 MSAL.NET](msal-net-initializing-client-applications.md)。

了解如何[实例化客户端应用程序使用 MSAL.js](msal-js-initializing-client-applications.md)。
