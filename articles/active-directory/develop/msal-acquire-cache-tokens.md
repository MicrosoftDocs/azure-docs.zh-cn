---
title: 管理令牌（Microsoft 身份验证库）| Azure
description: 了解如何使用 Microsoft 身份验证库 (MSAL) 获取和缓存令牌。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0d32b56b28d9ce7425e782fc10fa9ffb67047ce0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139133"
---
# <a name="acquiring-and-caching-tokens-using-msal"></a>使用 MSAL 获取和缓存令牌
客户端可以使用[访问令牌](access-tokens.md)安全调用受 Azure 保护的 Web API。 可以使用 Microsoft 身份验证库 (MSAL) 通过多种方法获取令牌。 有些方法需要用户通过 Web 浏览器进行交互。 而有些方法则不需要任何用户交互。 一般情况下，获取令牌的方法取决于应用程序是公共客户端应用程序（桌面或移动应用）还是机密客户端应用程序（Web 应用、Web API，或类似于 Windows 服务的后台程序应用程序）。

MSAL 在获取令牌后会缓存令牌。  在通过其他方式获取令牌之前，应用程序代码应该先尝试以无提示方式（从缓存中）获取令牌。

你也可以清除令牌缓存，这可以通过从缓存中删除帐户来实现。 不过，这不会删除浏览器中的会话 Cookie。

## <a name="scopes-when-acquiring-tokens"></a>获取令牌时的范围
[范围](v2-permissions-and-consent.md)是客户端应用程序请求访问的 Web API 公开的权限。 在发出身份验证请求以获取用于访问 Web API 的令牌时，客户端应用程序将请求用户许可这些范围。 使用 MSAL 可以获取令牌来访问面向开发人员的 Azure AD (v1.0) 和 Microsoft 标识平台 (v2.0) API。 v2.0 协议在请求中使用范围而不是资源。 有关详细信息，请阅读 [v1.0 与 v2.0 的比较](active-directory-v2-compare.md)。 根据 v2.0 终结点接受的令牌版本的 Web API 配置，该终结点会将访问令牌返回到 MSAL。

有许多 MSAL 令牌获取方法都需要 *scopes* 参数。 此参数是一个简单的字符串列表，这些字符串声明所需的权限和请求的资源。 已知的范围是 [Microsoft Graph 权限](/graph/permissions-reference)。

在 MSAL 中也可以访问 v1.0 资源。 有关详细信息，请阅读 [v1.0 应用程序的范围](msal-v1-app-scopes.md)。

### <a name="request-specific-scopes-for-a-web-api"></a>请求 Web API 的特定范围
当应用程序需要使用资源 API 的特定权限请求令牌时，你需要采用以下格式传递包含 API 应用 ID URI 的范围：*&lt;应用 ID URI&gt;/&lt;范围&gt;*

Microsoft Graph API 的范围示例：`https://graph.microsoft.com/User.Read`

或者，自定义 Web API 的范围示例：`api://abscdefgh-1234-abcd-efgh-1234567890/api.read`

（仅适用于 Microsoft Graph API）范围值 `user.read` 将映射为 `https://graph.microsoft.com/User.Read` 格式，两者可以换用。

> [!NOTE]
> 某些 Web API（例如 Azure 资源管理器 API (https://management.core.windows.net/)）要求在访问令牌的受众声明 (aud) 中使用尾随的“/”。 在这种情况下，必须以 https://management.core.windows.net//user_impersonation 形式（请注意双斜杠）传递范围，使令牌在 API 中有效。

### <a name="request-dynamic-scopes-for-incremental-consent"></a>请求增量许可的动态范围
使用 v1.0 生成应用程序时，必须注册应用程序所需的完整权限集（静态范围），让用户在登录时许可这些权限。 在 v2.0 中，可根据需要使用范围参数请求其他权限。 这些范围称为动态范围，可让用户提供范围的增量许可。

例如，可以让用户先登录，但拒绝他们进行任何类型的访问。 然后，可以通过在令牌获取方法中请求日历范围来让用户阅读日历，并获取用户的许可。

例如 `https://graph.microsoft.com/User.Read` 和 `https://graph.microsoft.com/Calendar.Read`

## <a name="acquiring-tokens-silently-from-the-cache"></a>以无提示方式（从缓存中）获取令牌
MSAL 维护一个令牌缓存（对机密客户端应用程序维护两个），获取令牌后会缓存令牌。  在许多情况下，尝试以无提示方式获取令牌会根据缓存中的令牌获取具有更多范围的另一个令牌。 此外，当某个令牌即将过期时可以刷新该令牌（因为令牌缓存中也包含一个刷新令牌）。

### <a name="recommended-call-pattern-for-public-client-applications"></a>公共客户端应用程序的建议调用模式
应用程序代码应该先尝试以无提示方式（从缓存中）获取令牌。  如果方法调用返回“需要 UI”错误或异常，请尝试通过其他方式获取令牌。 

但是，在尝试以无提示方式获取令牌之前，应该**先**完成两个流：

- [客户端凭据流](msal-authentication-flows.md#client-credentials)：不使用用户令牌缓存，而是使用应用程序令牌缓存。 在将请求发送到 STS 之前，此方法负责验证此应用程序令牌缓存。
- Web 应用中的[授权代码流](msal-authentication-flows.md#authorization-code)：通过将用户登录并让他们许可更多范围，来兑换应用程序收到的代码。 由于代码是作为参数而不是帐户传递的，该方法在兑换代码之前无法查看缓存，这仍然需要调用服务。

### <a name="recommended-call-pattern-in-web-apps-using-the-authorization-code-flow"></a>使用授权代码流的 Web 应用中的建议调用模式 
对于使用 [OpenID Connect 授权代码流](v2-protocols-oidc.md)的 Web 应用程序，控制器中的建议模式为：

- 使用已经过自定义序列化的令牌缓存实例化某个机密客户端应用程序。 
- 使用授权代码流获取令牌

## <a name="acquiring-tokens"></a>获取令牌
一般情况下，获取令牌的方法取决于应用程序是公共客户端还是机密客户端应用程序。

### <a name="public-client-applications"></a>公共客户端应用程序
对于公共客户端应用程序（桌面或移动应用）：
- 通常是让用户通过 UI 或弹出窗口登录，以交互方式获取令牌。
- 如果桌面应用程序在已加入域或 Azure 的 Windows 计算机上运行，则可以使用 Windows 集成身份验证 (IWA/Kerberos) [以无提示方式获取已登录用户的令牌](msal-authentication-flows.md#integrated-windows-authentication)。
- 在 .NET Framework 桌面客户端应用程序中可以[使用用户名和密码获取令牌](msal-authentication-flows.md#usernamepassword)，但不建议这样做。 在机密客户端应用程序中不要使用用户名/密码。
- 在没有 Web 浏览器的设备上运行的应用程序中，可以通过[设备代码流](msal-authentication-flows.md#device-code)获取令牌。 系统将为用户提供 URL 和代码，然后，他们可以在另一台设备上打开 Web 浏览器、输入代码并登录。  然后，Azure AD 会将令牌发回到没有浏览器的设备。

### <a name="confidential-client-applications"></a>机密客户端应用程序 
对于机密客户端应用程序（Web 应用、Web API，或类似于 Windows 服务的后台程序应用程序）：
- 使用[客户端凭据流](msal-authentication-flows.md#client-credentials)获取**应用程序本身**而不是用户的令牌。 此方法可用于同步工具，或者处理普通用户而不是特定用户的工具。 
- 使用适用于 Web API 的[代表流](msal-authentication-flows.md#on-behalf-of)代表用户调用 API。 系统会使用客户端凭据标识应用程序，以根据用户断言（例如 SAML 或 JWT 令牌）获取令牌。 需要在服务到服务调用中访问特定用户的资源的应用程序使用此流。
- 在用户通过授权请求 URL 登录后，在 Web 应用中使用[授权代码流](msal-authentication-flows.md#authorization-code)获取令牌。 OpenID Connect 应用程序通常使用此机制，可让用户使用 Open ID Connect 登录，然后代表用户访问 Web API。


## <a name="authentication-results"></a>身份验证结果 
当客户端请求访问令牌时，Azure AD 还会返回一条身份验证结果，其中包含有关访问令牌的一些元数据。 此信息包含访问令牌的过期时间及其有效范围。 此数据可让应用执行访问令牌的智能缓存，而无需分析访问令牌本身。  身份验证结果公开：

- 由 Web API 用来访问资源的[访问令牌](access-tokens.md)。 这是一个字符串，它通常是一个 base64 编码的 JWT，但客户端不得查看访问令牌的内部信息。 不保证格式稳定，对于资源可将令牌加密。 在客户端上依赖访问令牌内容编写代码是最大的错误来源之一，并且会违反客户端逻辑。
- 用户的 [ID 令牌](id-tokens.md) (JWT)。
- 令牌过期时间，告知令牌的过期日期/时间。
- 租户 ID 包含用户所在的租户。 对于来宾用户（Azure AD B2B 方案），租户 ID 是来宾租户，而不是唯一的租户。 以用户的名义传送令牌时，身份验证结果还包含有关此用户的信息。 对于未使用用户帐户请求令牌的机密客户端流（适用于应用程序），此用户信息为 null。
- 令牌的颁发范围。
- 用户的唯一 ID。

## <a name="next-steps"></a>后续步骤
了解如何[处理错误和异常](msal-handling-exceptions.md)。 
