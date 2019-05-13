---
title: 与 Microsoft 标识平台集成 |Azure
description: 了解最佳做法和常见疏忽时与 Microsoft 标识平台 (v2.0) 集成。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: celested
ms.reviewer: lenalepa, sureshja, jesakowi
ms.custom: aaddev
ms.openlocfilehash: 1f5a6d7501fd387548b111fcb0888515c8f8db25
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076400"
---
# <a name="microsoft-identity-platform-integration-checklist"></a>Microsoft 标识平台集成清单

Microsoft 标识平台集成清单旨在引导您找到高质量和安全的集成。 它将突出显示最佳实践，并与 Microsoft 标识平台集成时的常见疏忽因此查看定期进行以确保您维护的质量和安全性的应用程序的集成的标识平台上的列表。 检查表并不旨在查看整个应用程序。 当我们进行改进了平台，该清单的内容会有所更改。

如果你刚入门，请查看[文档](index.yml)若要了解有关身份验证基础知识、 Microsoft 标识平台中的应用程序方案和的详细信息。

## <a name="testing-your-integration"></a>测试你的集成

使用以下清单确保有效地结合您的应用程序[Microsoft 标识平台](https://docs.microsoft.com/legal/mdsa)。

### <a name="basics"></a>基础

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 阅读和理解[Microsoft 平台策略](https://docs.microsoft.com/legal/mdsa)。 请确保你的应用程序符合所述，因为它们旨在保护用户和平台的条款。 |

### <a name="ownership"></a>所有权

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 请确保您用来注册和管理应用与帐户关联的信息是最新。 |

### <a name="branding"></a>品牌

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 遵守[品牌的应用程序的准则](howto-add-branding-in-azure-ad-apps.md)。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 为你的应用程序中提供有意义的名称和徽标。 此信息将出现在应用程序的许可提示。 请确保你的名称和徽标是公司产品的代表，以便用户可以做出明智的决策。 请确保您在不违反任何商标。 |

### <a name="privacy"></a>隐私

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 提供指向你的应用的条款的服务和隐私声明的链接。 |

### <a name="security"></a>安全

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 保持所有重定向 Uri 的所有权并确保它们的 DNS 记录保持最新状态。 不要在 Uri 中使用通配符 （*）。 对于 web 应用，请确保所有 Uri 安全并且经过加密 （例如，使用 https 方案）。 对于公共客户端，使用特定于平台的重定向 Uri 可能的话 （主要用于 iOS 和 Android）。 否则，使用重定向 Uri 使用大量的随机性，以避免冲突时调用返回到您的应用程序。 如果你的应用正在使用从独立的 web 代理，则可能使用 https://login.microsoftonline.com/nativeclient。 查看和剪裁所有未使用或不必要的重定向 Uri 定期进行。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 如果在目录中注册您的应用程序，则最小化和手动监视应用程序注册所有者的列表。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 不支持[OAuth2 隐式授权流](v2-oauth2-implicit-grant-flow.md)除非明确要求。 了解有关有效的方案[此处](v1-oauth2-implicit-grant-flow.md#suitable-scenarios-for-the-oauth2-implicit-grant)。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 不要使用[资源所有者密码凭据流 (ROPC)](v2-oauth-ropc.md)，它直接处理用户的密码。 此流需要很高的信任和用户风险，并仅用于时不能使用其他、 更安全的流。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 保护和管理你的应用程序凭据。 使用[证书凭据](active-directory-certificate-credentials.md)，而不是密码凭据 （客户端机密）。 如果必须使用密码凭据，请不要将其手动。 不要将凭据存储在代码或配置，并永远不会允许他们要由人类。 如果可能，请使用[托管于 Azure 资源的标识](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)或[Azure 密钥保管库](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)来存储和定期轮换凭据。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 请确保你的应用程序请求最少特权权限。 只向请求应用程序绝对需要的权限和时，才需要。 了解不同[类型的权限](v1-permissions-and-consent.md#types-of-permissions)。 仅使用所需; 如果应用程序权限在可能的情况，请使用委派的权限。 有关 Microsoft Graph 权限的完整列表，请参阅此[权限参考](https://docs.microsoft.com/graph/permissions-reference)。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 如果您是要保护使用 Microsoft 标识平台的 API，认真考虑应公开的权限。 请考虑什么是正确的粒度，让你的解决方案以及哪些权限需要管理员同意。 任何授权决策之前检查传入令牌中的预期权限。 |

### <a name="implementation"></a>实现

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 使用新式身份验证解决方案 (OAuth 2.0 [OpenID Connect](v2-protocols-oidc.md)) 来安全地登录用户。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 不自行实现协议-使用[Microsoft 支持的身份验证库](reference-v2-libraries.md)(MSAL，服务器中间件)。 请确保你正在使用已与集成的身份验证库的最新版本。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 如果您的应用程序所需的数据是可通过[Microsoft Graph](https://developer.microsoft.com/graph)，请求的 Microsoft Graph 终结点而不是单个 API 使用此数据的权限。 |

### <a name="end-user-experience"></a>最终用户体验

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | [了解同意体验](application-consent-experience.md)和配置应用程序的许可提示的各个部分，以便最终用户和管理员具有足够的信息来确定它们是否信任您的应用程序。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 最大程度减少用户需要通过无提示尝试使用您的应用程序时输入登录凭据的次数之前交互式流的身份验证 （无提示令牌获取）。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 不要使用"提示 = 同意"来每次登录。 仅使用提示符 = 同意的情况下，如果您已确定你需要请求其他权限 （例如，如果你已更改你的应用所需的权限） 的许可。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 如果适用，丰富应用程序与用户数据。 使用[Microsoft Graph API](https://developer.microsoft.com/graph)是执行此操作的简单办法。 [图形资源管理器](https://developer.microsoft.com/graph/graph-explorer)可以帮助您入门的工具。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 注册您的应用程序需要，因此管理员可以轻松地授予同意的情况下，到其租户的权限的完整集。 使用[增量许可](azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent)在运行时以帮助用户了解其中的原因，您的应用程序正在请求可能涉及或使用户请求第一次启动时感到困惑的权限。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 实现[清理单一注销体验](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut)。 它是隐私和安全要求，并使以获得良好的用户体验。 |

### <a name="testing"></a>测试

|   |   |
|---|---|
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 用于测试[条件性访问策略](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut)，可能会影响你的用户能够使用你的应用程序。 |
| ![复选框](./media/active-directory-integration-checklist/checkbox-two.svg) | 测试应用程序与您计划支持 （有关示例、 工作或学校帐户、 个人 Microsoft 帐户、 子女帐户和主权帐户） 的所有可能的帐户。 |

## <a name="additional-resources"></a>其他资源

浏览有关 v2.0 的深入信息：

* [Microsoft 标识平台 （v2.0 概述）](v2-overview.md)
* [Microsoft 标识平台协议参考](active-directory-v2-protocols.md)
* [访问令牌参考](access-tokens.md)
* [ID 令牌参考](id-tokens.md)
* [身份验证库参考](reference-v2-libraries.md)
* [权限和 Microsoft 标识平台中的许可](v2-permissions-and-consent.md)
* [Microsoft 图形 API](https://developer.microsoft.com/graph)