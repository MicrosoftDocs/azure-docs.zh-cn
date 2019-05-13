---
title: 了解支持的 Microsoft 标识平台的授权协议 |Azure
description: 支持的 Microsoft 标识平台终结点的 OAuth 2.0 和 OpenID Connect 协议的指南。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 5fb4fa1b-8fc4-438e-b3b0-258d8c145f22
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/11/2019
ms.author: celested
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8d509e9f3cabcd588d539d42b8cce022e17a1222
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139211"
---
# <a name="microsoft-identity-platform-protocols"></a>Microsoft 标识平台协议

Microsoft 标识平台的终结点标识为-服务与行业标准协议，OpenID Connect 和 OAuth 2.0。 尽管此服务与标准兼容，但这些协议的两个实现之间仍然存在微妙的差异。 如果选择通过直接发送和处理 HTTP 请求，或使用第三方开放源代码库来编写代码，而不是使用我们的其中一个[开放源代码库](reference-v2-libraries.md)，则可以参考此处提供的有用信息。

> [!NOTE]
> Microsoft 标识平台终结点支持不是所有 Azure AD 方案和功能。 若要确定是否应使用 Microsoft 标识平台终结点，阅读[Microsoft 标识平台限制](active-directory-v2-limitations.md)。

## <a name="the-basics"></a>基础知识

几乎在所有的 OAuth 2.0 和 OpenID Connect 流中，都有四个参与交换的对象：

![OAuth 2.0 角色](./media/active-directory-v2-flows/protocols-roles.svg)

* **授权服务器**是 Microsoft 标识平台终结点，并负责确保用户的标识，授予和撤消访问资源，以及发出令牌。 授权服务器也称为标识提供者 - 它可安全处理与用户信息、用户访问权，以及流中各方彼此间信任关系有关的任何项目。
* 资源所有者通常是最终用户。 它是拥有数据的一方，并且有权允许第三方访问该数据或资源。
* OAuth 客户端是应用，按照其应用程序 ID 进行标识。 OAuth 客户端通常是与最终用户交互的对象，并向授权服务器请求令牌。 客户端必须获得资源所有者授权才能访问资源。
* **资源服务器**是资源或数据所在的位置。 它信任授权服务器便可以安全地进行身份验证和授权 OAuth 客户端，并使用持有者访问令牌来确保可以授予对资源的访问。

## <a name="app-registration"></a>应用注册

要接受同时个人和工作或学校帐户的每个应用程序必须通过注册**应用注册**方面的经验[Azure 门户](https://aka.ms/appregistrations)它可以注册中使用 OAuth 2.0 的这些用户之前或 OpenID Connect。 应用注册进程会收集一些值并将其分配到应用：

* 用于唯一标识应用的**应用程序 ID**
* 用于将响应定向回应用的**重定向 URI** 或**包标识符**
* 其他一些特定于方案的值。

请了解如何 [注册应用](quickstart-register-app.md)获取详细信息。

## <a name="endpoints"></a>终结点

注册后，应用将通过将请求发送到终结点与 Microsoft 标识平台进行通信：

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

其中 `{tenant}` 可以接受以下四个不同值之一：

| 值 | 描述 |
| --- | --- |
| `common` | 允许用户使用个人 Microsoft 帐户和工作/学校帐户从 Azure AD 登录应用程序。 |
| `organizations` | 仅允许用户使用工作/学校帐户从 Azure AD 登录应用程序。 |
| `consumers` | 仅允许用户使用个人的 Microsoft 帐户 (MSA) 登录应用程序。 |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.onmicrosoft.com` | 仅允许用户使用工作/学校帐户从特定的 Azure AD 租户登录应用程序。 可以使用 Azure AD 租户的友好域名或租户的 GUID 标识符。 |

要了解如何与这些终结点进行交互，请在[协议](#protocols)部分中选择特定的应用类型，然后通过访问链接获取更多信息。

> [!TIP]
> 在 Azure AD 中注册任何应用程序可以使用 Microsoft 标识平台终结点，即使他们不在个人帐户登录。  这样一来，您可以将迁移到 Microsoft 标识平台的现有应用程序和[MSAL](reference-v2-libraries.md)而无需重新创建你的应用程序。  

## <a name="tokens"></a>令牌

Microsoft 标识平台实现的 OAuth 2.0 和 OpenID Connect 进行广泛使用持有者令牌，包括表示为 Jwt 的持有者令牌。 持有者令牌是一种轻型安全令牌，它授予对受保护资源的“持有者”访问权限。 从这个意义上来说，“持有者”是可以提供令牌的任何一方。 参与方必须首先进行身份验证 Microsoft 标识平台，才能收到持有者令牌中，如果不采取必要的步骤来保护传输和存储中的令牌，但它可以截获并由不速之客。 虽然某些安全令牌具有内置机制来防止未经授权方使用它们，但是持有者令牌没有这一机制，因此必须在安全的通道（例如传输层安全 (HTTPS)）中进行传输。 如果持有者令牌以明文形式传输，则恶意方可以使用中间人攻击来获取令牌并将其用于对受保护资源进行未经授权的访问。 当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。 请始终确保应用以安全的方式传输和存储持有者令牌。 有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](https://tools.ietf.org/html/rfc6750)。

更多详细信息的不同类型的标识平台终结点现已推出 Microsoft 中使用的标记[Microsoft 标识平台终结点令牌参考](v2-id-and-access-tokens.md)。

## <a name="protocols"></a>协议

如果已准备好查看部分示例请求，请从下列教程之一开始。 每个教程对应于特定的身份验证方案。 如果需要帮助确定哪一个适当的流，请查看[的类型的应用可以使用 Microsoft 标识平台构建](v2-app-types.md)。

* [使用 OAuth 2.0 构建移动和本机应用程序](v2-oauth2-auth-code-flow.md)
* [使用 OpenID Connect 构建 Web 应用](v2-protocols-oidc.md)
* [使用 OAuth 2.0 隐式流构建单页应用](v2-oauth2-implicit-grant-flow.md)
* [使用 OAuth 2.0 客户端凭据流构建守护程序或服务器端进程](v2-oauth2-client-creds-grant-flow.md)
* [使用 OAuth 2.0 代理流在 Web API 中获取令牌](v2-oauth2-on-behalf-of-flow.md)
