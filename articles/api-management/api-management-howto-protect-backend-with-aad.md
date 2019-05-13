---
title: 结合 Azure Active Directory 和 API 管理使用 OAuth 2.0 保护 API | Microsoft Docs
description: 了解如何结合使用 Azure Active Directory 和 API 管理来保护 Web API 后端。
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2018
ms.author: apimpm
ms.openlocfilehash: b5467711f06380ca61b4a9d5150b66c3f945c08c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141078"
---
# <a name="protect-an-api-by-using-oauth-20-with-azure-active-directory-and-api-management"></a>结合 Azure Active Directory 和 API 管理使用 OAuth 2.0 保护 API

本指南介绍如何结合 Azure Active Directory (Azure AD) 使用 OAuth 2.0 协议配置 Azure API 管理实例，以保护 API。 

## <a name="prerequisites"></a>必备组件
若要执行本文中的步骤，必须提供：
* API 管理实例
* 使用 API 管理实例发布的 API
* Azure AD 租户

## <a name="overview"></a>概述

下面是简要的步骤概述：

1. 在 Azure AD 中注册一个应用程序（后端应用）用于表示 API。
2. 在 Azure AD 中注册另一个应用程序（客户端应用），用于表示需要调用 API 的客户端应用程序。
3. 在 Azure AD 中授予权限，使客户端应用能够调用后端应用。
4. 将开发人员控制台配置为使用 OAuth 2.0 用户授权。
5. 添加 **validate-jwt** 策略以验证每个传入请求的 OAuth 令牌。

## <a name="register-an-application-in-azure-ad-to-represent-the-api"></a>在 Azure AD 中注册一个应用程序用于表示 API

若要使用 Azure AD 保护 API，首先需要在 Azure AD 中注册一个表示该 API 的应用程序。 

1. 浏览到你的 Azure AD 租户，并浏览到**应用注册 （旧版）**。

2. 选择“新建应用程序注册”。 

3. 提供应用程序的名称。 （本示例使用名称 `backend-app`。）  

4. 为“应用程序类型”选择“Web 应用/API”。 

5. 对于“登录 URL”，可以使用 `https://localhost` 作为占位符。

6. 选择“创建”。

创建应用程序后，记下“应用程序 ID”，以便在后续步骤中使用。 

## <a name="register-another-application-in-azure-ad-to-represent-a-client-application"></a>在 Azure AD 中注册另一个应用程序用于表示客户端应用程序

需要调用该 API 的每个客户端应用程序也必须注册到 Azure AD 中。 在本示例中，示例客户端应用程序是 API 管理开发人员门户中的开发人员控制台。 下面介绍如何在 Azure AD 中注册另一个应用程序用于表示开发人员控制台。

1. 在处于**应用注册 （旧版）**，选择**新应用程序注册**。 

2. 提供应用程序的名称。 （本示例使用名称 `client-app`。）

3. 为“应用程序类型”选择“Web 应用/API”。  

4. 对于“登录 URL”，可以使用 `https://localhost` 作为占位符，或使用 API 管理实例的登录 URL。 （本示例使用 URL `https://contoso5.portal.azure-api.net/signin`。）

5. 选择“创建”。

创建应用程序后，记下“应用程序 ID”，以便在后续步骤中使用。 

现在，请创建此应用程序的客户端机密，以便在后续步骤中使用。

1. 再次选择“设置”并转到“密钥”。

2. 在“密码”下面提供**密钥说明**。 选择密钥过期时间，然后选择“保存”。

记下密钥值。 

## <a name="grant-permissions-in-azure-ad"></a>在 Azure AD 中授予权限

注册用于表示 API 和开发人员控制台的两个应用程序之后，需要授予权限，使客户端应用能够调用后端应用。  

1. 浏览到**应用程序注册 （旧版）**。 

2. 选择 `client-app` 并转到“设置”。

3. 选择“所需的权限” > “添加”。

4. 选择“选择 API”并搜索 `backend-app`。

5. 在“委托的权限”下面选择 **。**`Access backend-app` 

6. 依次选择“选择”、“完成”。 

> [!NOTE]
> 如果“Azure Active Directory”未在“对其他应用程序的权限”下列出，请选择“添加”并从列表中添加该项。

## <a name="enable-oauth-20-user-authorization-in-the-developer-console"></a>在开发人员门户中启用 OAuth 2.0 用户授权

此时，我们已在 Azure AD 中创建应用程序，并已授予适当的权限来让客户端应用调用后端应用。 

在本示例中，开发人员控制台是 client-app。 以下步骤说明如何在开发人员门户中启用 OAuth 2.0 用户授权 

1. 在 Azure 门户中，浏览到你的 API 管理实例。

2. 选择“OAuth 2.0” > “添加”。

3. 提供“显示名称”和“说明”。

4. 对于“客户端注册页 URL”，请输入占位符值，如 `http://localhost`。 “客户端注册页 URL”指向供用户针对 OAuth 2.0 提供程序创建和配置其自己的帐户的页面。 在此示例中，用户不创建和配置自己的帐户，因此使用了占位符。

5. 选择“授权代码”作为“授权类型”。

6. 指定“授权终结点 URL”和“令牌终结点 URL”。 可以从 Azure AD 租户中的“终结点”页检索这些值。 再次浏览到“应用注册”页，并选择“终结点”。

    >[!NOTE]
    > 在此处使用 **v1** 终结点

7. 复制“OAuth 2.0 授权终结点”，并将其粘贴到“授权终结点 URL”文本框。

8. 复制“OAuth 2.0 令牌终结点”，并将其粘贴到“令牌终结点 URL”文本框。 除了在令牌终结点中粘贴上述值以外，请添加名为 **resource** 的附加正文参数。 使用后端应用的“应用程序 ID”作为该参数的值。

9. 接下来，指定客户端凭据。 这是客户端应用的凭据。

10. 对于“客户端 ID”，请使用客户端应用的“应用程序 ID”。

11. 对于“客户端机密”，请使用前面为客户端应用创建的密钥。 

12. 紧接在客户端机密的后面，是授权代码授权类型的 **redirect_url**。 记下此 URL。

13. 选择“创建”。

14. 返回客户端应用的“设置”页。

15. 选择“回复 URL”，并在第一行中粘贴 **redirect_url**。 在本示例中，我们已将 `https://localhost` 替换为第一行中的 URL。  

配置 OAuth 2.0 授权服务器后，开发人员控制台可从 Azure AD 获取访问令牌。 

下一步是为 API 启用 OAuth 2.0 用户授权。 这使得开发人员控制台知道在调用 API 之前，需要代表用户获取访问令牌。

1. 浏览到 API 管理实例，并转到“API”。

2. 选择要保护的 API。 本示例使用 `Echo API`。

3. 转到“设置”。

4. 在“安全性”下，选择“OAuth 2.0”并选择前面配置的 OAuth 2.0 服务器。 

5. 选择“保存”。

## <a name="successfully-call-the-api-from-the-developer-portal"></a>从开发人员门户成功调用 API

> [!NOTE]
> 本部分不适用于“消耗”层，该层不支持开发人员门户。

在 `Echo API` 中启用 OAuth 2.0 用户授权后，开发人员控制台在调用 API 之前，将会代表用户获取访问令牌。

1. 在开发人员门户中浏览到 `Echo API` 下的任一操作，并选择“试用”。 随后会转到开发人员控制台。

2. 可以看到，“授权”部分出现了一个与刚刚添加的授权服务器对应的新项。

3. 从授权下拉列表中选择“授权代码”。系统会提示登录到 Azure AD 租户。 如果已使用帐户登录，则可能不会出现提示。

4. 成功登录后，会将一个 `Authorization` 标头添加到请求，该标头包含从 Azure AD 获取的访问令牌。 下面是一个采用 Base64 编码的示例令牌：

   ```
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCIsImtpZCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCJ9.eyJhdWQiOiIxYzg2ZWVmNC1jMjZkLTRiNGUtODEzNy0wYjBiZTEyM2NhMGMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC80NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgvIiwiaWF0IjoxNTIxMTUyNjMzLCJuYmYiOjE1MjExNTI2MzMsImV4cCI6MTUyMTE1NjUzMywiYWNyIjoiMSIsImFpbyI6IkFWUUFxLzhHQUFBQUptVzkzTFd6dVArcGF4ZzJPeGE1cGp2V1NXV1ZSVnd1ZXZ5QU5yMlNkc0tkQmFWNnNjcHZsbUpmT1dDOThscUJJMDhXdlB6cDdlenpJdzJLai9MdWdXWWdydHhkM1lmaDlYSGpXeFVaWk9JPSIsImFtciI6WyJyc2EiXSwiYXBwaWQiOiJhYTY5ODM1OC0yMWEzLTRhYTQtYjI3OC1mMzI2NTMzMDUzZTkiLCJhcHBpZGFjciI6IjEiLCJlbWFpbCI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiSmlhbmciLCJnaXZlbl9uYW1lIjoiTWlhbyIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpcGFkZHIiOiIxMzEuMTA3LjE3NC4xNDAiLCJuYW1lIjoiTWlhbyBKaWFuZyIsIm9pZCI6IjhiMTU4ZDEwLWVmZGItNDUxMS1iOTQzLTczOWZkYjMxNzAyZSIsInNjcCI6InVzZXJfaW1wZXJzb25hdGlvbiIsInN1YiI6IkFGaWtvWFk1TEV1LTNkbk1pa3Z3MUJzQUx4SGIybV9IaVJjaHVfSEM1aGciLCJ0aWQiOiI0NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgiLCJ1bmlxdWVfbmFtZSI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsInV0aSI6ImFQaTJxOVZ6ODBXdHNsYjRBMzBCQUEiLCJ2ZXIiOiIxLjAifQ.agGfaegYRnGj6DM_-N_eYulnQdXHhrsus45QDuApirETDR2P2aMRxRioOCR2YVwn8pmpQ1LoAhddcYMWisrw_qhaQr0AYsDPWRtJ6x0hDk5teUgbix3gazb7F-TVcC1gXpc9y7j77Ujxcq9z0r5lF65Y9bpNSefn9Te6GZYG7BgKEixqC4W6LqjtcjuOuW-ouy6LSSox71Fj4Ni3zkGfxX1T_jiOvQTd6BBltSrShDm0bTMefoyX8oqfMEA2ziKjwvBFrOjO0uK4rJLgLYH4qvkR0bdF9etdstqKMo5gecarWHNzWi_tghQu9aE3Z3EZdYNI_ZGM-Bbe3pkCfvEOyA
   ```

5. 选择“发送”，然后即可成功调用 API。


## <a name="configure-a-jwt-validation-policy-to-pre-authorize-requests"></a>配置 JWT 验证策略以对请求进行预授权

此时，当用户尝试从开发人员控制台发出调用时，系统会提示其登录。 开发人员控制台会代表该用户获取访问令牌。

如果有人调用我们的 API 但未提供令牌或者提供无效的令牌，会发生什么情况？ 例如，你可以尝试删除 `Authorization` 标头，然后会发现仍可调用该 API。 原因是 API 管理暂时不会验证访问令牌。 它只是将 `Authorization` 标头传递给后端 API。

可以使用[验证 JWT](api-management-access-restriction-policies.md#ValidateJWT) 策略通过验证每个传入请求的访问令牌，对 API 管理中的请求进行预授权。 如果某个请求没有有效的令牌，API 管理会阻止该请求。 例如，我们可以在 `Echo API` 的 `<inbound>` 策略部分中添加以下策略。 它会检查访问令牌中的受众声明，如果令牌无效，则会返回一条错误消息。 有关如何配置策略的信息，请参阅[设置或编辑策略](set-edit-policies.md)。

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/{aad-tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud">
            <value>{Application ID of backend-app}</value>
        </claim>
    </required-claims>
</validate-jwt>
```

## <a name="build-an-application-to-call-the-api"></a>生成应用程序来调用 API

在本指南中，我们使用了 API 管理中的开发者控制台作为示例客户端应用程序来调用由 OAuth 2.0 保护的 `Echo API`。 若要详细了解如何生成应用程序并实现 OAuth 2.0，请参阅 [Azure Active Directory 代码示例](../active-directory/develop/sample-v1-code.md)。

## <a name="next-steps"></a>后续步骤
* 详细了解 [Azure Active Directory 和 OAuth2.0](../active-directory/develop/authentication-scenarios.md)。
* 观看有关 API 管理的更多[视频](https://azure.microsoft.com/documentation/videos/index/?services=api-management)。
* 有关保护后端服务的其他方法，请参阅[使用证书进行相互身份验证](api-management-howto-mutual-certificates.md)。

* [创建 API 管理服务实例](get-started-create-service-instance.md)。

* [管理第一个 API](import-and-publish.md)。
