---
title: 为什么更新到 Microsoft 标识平台 (v2.0) |Azure
description: 了解 Microsoft 标识平台 (v2.0) 终结点和 Azure Active Directory (Azure AD) v1.0 终结点之间的差异，并了解更新到 v2.0 的优势。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: celested
ms.reviewer: saeeda, hirsin, jmprieur, sureshja, jesakowi, lenalepa, kkrishna, dadobali, negoe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: dba74735b4c703123f9ff89b63a57d53faa84fde
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138785"
---
# <a name="why-update-to-microsoft-identity-platform-v20"></a>更新到 Microsoft 标识平台 (v2.0) 的原因？

开发新程序时，务必要知道 Microsoft 标识平台 (v2.0) 和 Azure Active Directory (v1.0) 终结点之间的差异。 本文介绍如何将终结点与 Microsoft 标识平台的一些现有限制的主要差异。

> [!NOTE]
> Microsoft 标识平台终结点不支持所有 Azure AD 方案和功能。 若要确定是否应使用 Microsoft 标识平台终结点，阅读[Microsoft 标识平台限制](#limitations)。

## <a name="who-can-sign-in"></a>谁可以登录

![谁可以使用 v1.0 和 v2.0 终结点登录](media/azure-ad-endpoint-comparison/who-can-sign-in.svg)

* v1.0 终结点仅允许使用工作和学校帐户登录到应用程序 (Azure AD)
* Microsoft 标识平台终结点允许工作和学校帐户从 Azure AD 和个人 Microsoft 帐户 (MSA)，例如 hotmail.com、 outlook.com 和 msn.com，进行登录。
* 这两个终结点还接受的登录名*[来宾用户](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b)* 的应用程序配置为 Azure AD 目录*[单租户](single-and-multi-tenant-apps.md)* 或对于*多租户*应用程序配置为指向特定于租户的终结点 (`https://login.microsoftonline.com/{TenantId_or_Name}`)。

Microsoft 标识平台终结点，可编写的应用程序接受从个人 Microsoft 帐户和工作和学校帐户登录。 这样，你便可以编写完全不区分帐户的应用。 例如，如果应用调用 [Microsoft Graph](https://graph.microsoft.io)，则工作帐户可以使用某些附加功能和数据，如 SharePoint 站点或目录数据。 但对于许多操作（例如[读取用户的邮件](https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/user_list_messages)），相同的代码可以访问个人帐户以及工作和学校帐户的电子邮件。

对于 Microsoft 标识平台终结点，可以使用 Microsoft 身份验证库 (MSAL) 访问消费类、 教育、 环境和企业领域。 Azure AD v1.0 终结点仅接受工作和学校帐户的登录。

## <a name="incremental-and-dynamic-consent"></a>增量同意和动态同意

使用 Azure AD v1.0 终结点的应用需要提前指定所需的 OAuth 2.0 权限，例如：

![权限注册 UI](./media/azure-ad-endpoint-comparison/app_reg_permissions.png)

直接在应用程序注册中设置的权限是**静态的**。 尽管在 Azure 门户中定义应用的静态权限能保持代码的简洁性，但可能会给开发人员带来几个问题：

* 应用需要在用户首次登录时请求可能需要的权限。 这可能会导致冗长的权限列表，而让最终用户在初始登录时打消审批应用程序访问权限的念头。

* 应用需要事先知道可能访问的所有资源。 很难创建能够访问任意数目的资源的应用程序。

与 Microsoft 标识平台终结点，则可以忽略静态定义的权限在 Azure 门户和请求权限的应用程序注册信息中以增量方式相反，这意味着裸机的最小提前权限集的要求并不断增长的客户随着时间的推移更多使用更多的应用功能。 为此，可以在请求访问令牌时，通过在 `scope` 参数中包含新的范围指定应用所需的范围 - 无需在应用程序注册信息中预定义这些范围。 如果用户尚未许可添加到请求的新范围，则系统会提示他们仅许可新的权限。 有关详细信息，请参阅[权限、许可和范围](v2-permissions-and-consent.md)。

允许应用通过 `scope` 参数动态请求权限可让开发人员完全控制用户的体验。 还可以将许可体验提前，并在一个初始授权请求中请求所有的权限。 如果应用需要大量的权限，则你可以在用户尝试使用应用的某些功能过程中，以递增方式向用户收集这些权限。

代表组织执行的管理员许可仍然需要为应用注册的静态权限，因此，如果需要管理员代表整个组织提供许可，则应在应用注册门户中为应用设置这些权限。 这会减少组织管理员设置应用程序所需的周期数。

## <a name="scopes-not-resources"></a>范围而非资源

对于使用 v1.0 终结点的应用，应用可以充当**资源**或令牌接收者。 资源可定义它所了解的许多**范围**或 **oAuth2Permissions**，使客户端应用能够从该资源中为一组特定的范围请求令牌。 请考虑以 Azure AD 图形 API 作为资源的示例：

* 资源标识符，或 `AppID URI`：`https://graph.windows.net/`
* 范围或 `oAuth2Permissions`：`Directory.Read`、`Directory.Write` 等等。

这适用于 Microsoft 标识平台终结点。 应用仍可充当资源、定义范围并由 URI 标识。 客户端应用程序仍可请求访问这些范围。 但是，客户端请求这些权限的方式发生了变化。

对于 v1.0 终结点，Azure AD 的 OAuth 2.0 授权请求可能如下所示：

```text
GET https://login.microsoftonline.com/common/oauth2/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&resource=https://graph.windows.net/
...
```

此处的 **resource** 参数指示客户端应用请求授权的资源。 Azure AD 根据 Azure 门户中的静态设置计算应用程序所需的权限，并据以发出令牌。

对于使用 Microsoft 标识平台终结点的应用程序，相同的 OAuth 2.0 授权请求如下：

```text
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&scope=https://graph.windows.net/directory.read%20https://graph.windows.net/directory.write
...
```

此处的 **scope** 参数指示应用请求授权的资源和权限。 所需的资源仍然在请求中提供 - 它包含在 scope 参数的每个值中。 以这种方式使用 scope 参数允许 Microsoft 标识平台终结点更符合 OAuth 2.0 规范，并与常见的行业实践的更紧密地保持一致。 它还使应用执行的操作[增量许可](#incremental-and-dynamic-consent)-仅当应用程序需要它们而不是预先请求权限。

## <a name="well-known-scopes"></a>已知的范围

### <a name="offline-access"></a>脱机访问

使用 Microsoft 标识平台终结点的应用可能需要使用新的已知权限的应用程序-`offline_access`作用域。 如果应用程序需要长期表示用户访问资源，则所有应用程序都需要请求此权限，即使用户可能不主动使用此应用程序亦然。 在许可对话框中，`offline_access` 范围对用户显示为“随时访问数据”，而用户必须同意。 请求`offline_access`权限可让您的 web 应用程序从 Microsoft 标识平台终结点接收 OAuth 2.0 refresh_tokens。 刷新令牌属于长效令牌，可用于交换新的 OAuth 2.0 访问令牌以延长访问期限。

如果应用不会请求`offline_access`作用域，则收不到刷新令牌。 这意味着，在 OAuth 2.0 授权代码流中兑换授权代码时，只从 `/token` 终结点接收访问令牌。 该访问令牌短时间维持有效（通常是一小时），但最后终将过期。 到时，应用必须将用户重定向回到 `/authorize` 终结点以检索新的授权代码。 在此重定向期间，根据应用程序的类型，用户或许无需再次输入其凭据或重新同意权限。

若要详细了解 OAuth 2.0 `refresh_tokens`，并`access_tokens`，请查看[Microsoft 标识平台协议参考](active-directory-v2-protocols.md)。

### <a name="openid-profile-and-email"></a>OpenID、profile 和 email

从历史上看，最基本的 OpenID Connect 登录流与 Microsoft 标识平台将提供在随后出现用户的信息的大量*id_token*。 id_token 中的声明可以包含用户的名称、首选用户名、电子邮件地址和对象 ID 等等。

现在对 `openid` 范围允许应用访问的信息进行了限制。 `openid` 范围只允许应用将用户登录，并接收用户的应用特定标识符。 如果想要在应用中获取关于用户的个人数据，则应用需要向用户请求额外的权限。 两个新范围 `email` 和 `profile` 将允许请求额外的权限。

* 假设用户具有可寻址的电子邮件地址，则 `email` 范围允许应用通过 id_token 中的 `email` 声明访问用户的主要电子邮件地址。
* `profile`范围可让应用访问的所有其他基本用户的信息，如其名称、 首选用户名、 对象 ID，依次类推，id_token 中。

使用这些范围可在尽可以能透露最少信息的情况下为应用编写代码，因此，只能向用户请求应用执行其作业所需的信息集。 有关这些作用域的详细信息，请参阅[Microsoft 标识平台作用域引用](v2-permissions-and-consent.md)。

## <a name="token-claims"></a>令牌声明

Microsoft 标识平台终结点以保持较小的有效负载的默认情况下发出一小部分其令牌中的声明。 如果你的应用和服务在 v1.0 令牌中不再提供默认情况下，Microsoft 标识平台令牌中的特定声明上具有依赖关系，请考虑使用[可选声明](active-directory-optional-claims.md)功能包括该声明。

## <a name="limitations"></a>限制

有一些限制需要注意的时使用 Microsoft 标识平台。

生成与 Microsoft 标识平台集成的应用程序时，您需要确定 Microsoft 标识平台的终结点和身份验证协议是否满足你的需求。 V1.0 终结点和平台仍完全受支持而且，在某些方面，比 Microsoft 标识平台功能更丰富。 但是，Microsoft 标识平台[引入了显著的好处](azure-ad-endpoint-comparison.md)面向开发人员。

下面是为开发人员简化的建议现在：

* 如果想要或需要在应用程序中支持个人 Microsoft 帐户或正在编写新的应用程序，使用 Microsoft 标识平台。 但在此之前，请确保了解本文所述的限制。
* 如果要迁移或更新依赖于 SAML 的应用程序，则无法使用 Microsoft 标识平台。 相反，请参阅[Azure AD v1.0 指南](v1-overview.md)。

Microsoft 标识平台终结点不断发展，以消除此处列出的限制，使您只需要使用 Microsoft 标识平台终结点。 在此期间，使用本文来确定 Microsoft 标识平台终结点是否是最适合你。 我们将持续更新本文以反映 Microsoft 标识平台终结点的当前状态。 检查返回以重新评估您的要求和 Microsoft 标识平台功能。

### <a name="restrictions-on-app-registrations"></a>应用注册限制

对于每个你想要与 Microsoft 标识平台终结点集成的应用，可以在新创建的应用注册[**应用注册**体验](https://aka.ms/appregistrations)在 Azure 门户中。 现有 Microsoft 帐户应用不符合在门户中，但所有 Azure AD 应用是，无论何时或何处其注册。

支持工作和学校帐户以及个人帐户的应用注册的注意事项如下：

* 每个应用程序 ID 只允许有两个应用密码。
* 未在租户中注册的应用程序只能由注册它的帐户管理。 不能与其他开发人员共享该应用程序。 在应用注册门户中使用 Microsoft 个人帐户注册的大多数应用都是如此。 如果你想要与多个开发人员分享你的应用注册，在使用新的租户中注册应用程序**应用注册**Azure 门户的一部分。
* 允许的重定向 URL 格式存在一些限制。 有关重定向 URL 的详细信息，请参阅下一部分。

### <a name="restrictions-on-redirect-urls"></a>重定向 URL 的限制

已被注册为 Microsoft 标识平台的应用仅限使用一组有限的重定向 URL 值。 Web 应用和服务的重定向 URL 必须以方案 `https` 开头，并且所有重定向 URL 值必须共享一个 DNS 域。  注册系统会将现有重定向 URL 的完整 DNS 名称与要添加的重定向 URL 的 DNS 名称相比较。 也支持将 `http://localhost` 用作重定向 URL。  

如果满足以下任一条件，添加 DNS 名称的请求会失败：  

* 新的重定向 URL 的完整 DNS 名称与现有的重定向 URL 的 DNS 名称不匹配。
* 新重定向 URL 的完整 DNS 名称不是现有重定向 URL 的子域。

#### <a name="example-1"></a>示例 1

如果应用的重定向 URL 为 `https://login.contoso.com`，则你可以添加 DNS 名称完全匹配的重定向 URL，如以下示例所示：

`https://login.contoso.com/new`

或者，可以引用 login.contoso.com 的 DNS 子域，如以下示例所示：

`https://new.login.contoso.com`

#### <a name="example-2"></a>示例 2

若要在应用中包含 `login-east.contoso.com` 和 `login-west.contoso.com` 作为重定向 URL，必须按以下顺序添加这些重定向 URL：

`https://contoso.com`  
`https://login-east.contoso.com`  
`https://login-west.contoso.com`  

您可以添加后两个，因为它们的第一个重定向 URL，子域 contoso.com。

你可以为特定应用程序只有 20 个答复 Url-此限制应用于所有应用类型注册支持 （单页面应用程序 (SPA)、 本机客户端、 web 应用和服务）。  

若要了解如何使用 Microsoft 标识平台注册以使用应用，请参阅[注册应用程序使用新的应用程序注册体验](quickstart-register-app.md)。

### <a name="restrictions-on-libraries-and-sdks"></a>库和 SDK 限制

目前，Microsoft 标识平台终结点的库支持是有限的。 如果你想要在生产应用程序中使用 Microsoft 标识平台终结点，您可以使用以下选项：

* 如果您正在构建的 web 应用程序，可安全地用于已公开发布的服务器端中间件执行登录和令牌验证。 其中包括适用于 ASP.NET 的 OWIN OpenID Connect 中间件和 Node.js Passport 插件。 使用 Microsoft 中间件的代码示例，请参阅[Microsoft 标识平台入门](v2-overview.md#getting-started)部分。
* 如果您正在构建桌面或移动应用程序，可以使用其中一个的 Microsoft 身份验证库 (MSAL)。 这些库已公开发布或在支持生产的预览版中，因此是安全的生产应用程序中使用它们。 有关预览版和可用库的术语的详细信息，请阅读[身份验证库参考](reference-v2-libraries.md)中的内容。
* 对于不受 Microsoft 库的平台，你可以通过直接发送和接收协议消息的应用程序代码与 Microsoft 标识平台终结点集成。 OpenID Connect 和 OAuth 协议[显式记录](active-directory-v2-protocols.md)可帮助你实现此类集成。
* 最后，可以使用开放源代码 OpenID Connect 和 OAuth 库来与 Microsoft 标识平台终结点集成。 Microsoft 标识平台终结点应该与无需更改许多开源协议库兼容。 这些类型的库的可用性根据语言和平台而有所差异。 [OpenID Connect](https://openid.net/connect/) 和 [OAuth 2.0](https://oauth.net/2/) 网站将维护一份热门实现列表。 有关详细信息，请参阅[Microsoft 标识平台和身份验证库](reference-v2-libraries.md)，和开放源代码客户端库和经过了 Microsoft 标识平台终结点的示例的列表。
* 有关参考， `.well-known` Microsoft 标识平台通用终结点的终结点是`https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`。 将 `common` 替换为你的租户 ID，以获取特定于你的租户的数据。  

### <a name="protocol-changes"></a>协议更改

Microsoft 标识平台终结点不支持 SAML 或 WS 联合身份验证;它仅支持 OpenID Connect 和 OAuth 2.0。  相比 v1.0 终结点，OAuth 2.0 协议的重大变化包括： 

* 如果配置了可选声明，**或者**在请求中指定了 scope=email，则返回 `email` 声明。 
* 现在支持使用 `scope` 参数来取代 `resource` 参数。  
* 许多响应已经过修改，因此更符合 OAuth 2.0 规范，例如，可正确返回整数而不是字符串形式的 `expires_in`。  

若要更好地了解在 Microsoft 标识平台终结点支持的协议功能范围，请参阅[OpenID Connect 和 OAuth 2.0 协议参考](active-directory-v2-protocols.md)。

#### <a name="saml-restrictions"></a>SAML 限制

如果你已在 Windows 应用程序使用 Active Directory 身份验证库 (ADAL)，则可能已利用 Windows 集成身份验证，使用安全断言标记语言 (SAML) 断言授予。 通过此授予，联合 Azure AD 租户的用户可以使用其本地 Active Directory 实例以无提示方式进行身份验证，而无需输入凭据。 Microsoft 标识平台终结点不支持 SAML 断言授予。
