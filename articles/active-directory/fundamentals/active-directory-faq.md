---
title: 常见问题解答 (FAQ) - Azure Active Directory | Microsoft Docs
description: 有关 Azure 和 Azure Active Directory、密码管理以及应用程序访问的常见问题和解答。
services: active-directory
author: eross-msft
manager: daveba
ms.assetid: b8207760-9714-4871-93d5-f9893de31c8f
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 11/12/2018
ms.author: lizross
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: f29ab7b908184533885929040ca8eccf56cda92d
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464419"
---
# <a name="frequently-asked-questions-about-azure-active-directory"></a>有关 Azure Active Directory 的常见问题
Azure Active Directory (Azure AD) 是综合性的标识即服务 (IDaaS) 解决方案，涉及到标识、访问管理和安全的方方面面。

有关详细信息，请参阅[什么是 Azure Active Directory？](active-directory-whatis.md)。


## <a name="access-azure-and-azure-active-directory"></a>访问 Azure 和 Azure Active Directory
**问：尝试在 Azure 门户中访问 Azure AD 时，为何出现“找不到订阅”错误？**

**答:** 若要访问 Azure 门户，每个用户都需要 Azure 订阅的权限。 如果订阅为付费型 Office 365 订阅或 Azure AD 订阅，请访问 [https://aka.ms/accessAAD](https://aka.ms/accessAAD)，了解一次性激活步骤。 否则需激活免费型 [Azure 帐户](https://azure.microsoft.com/pricing/free-trial/)或某个付费型订阅。

有关详细信息，请参阅：

* [Azure 订阅与 Azure Active Directory 的关联方式](active-directory-how-subscriptions-associated-directory.md)

- - -
**问：Azure AD、Office 365 与 Azure 之间的关系是什么？**

**答:** Azure AD 为所有 Web 服务提供通用的标识和访问功能。 不管使用的是 Office 365、Microsoft Azure、Intune 还是其他服务，都是在使用 Azure AD 为上述所有服务启用登录和访问管理。

可以将所有已设置为使用 Web 服务的用户定义为一个或多个 Azure AD 实例中的用户帐户。 可以在设置这些帐户时启用免费的 Azure AD 功能，例如云应用程序访问。

Azure AD 付费型服务（例如企业移动性 + 安全性）可通过综合性的企业级管理和安全解决方案来弥补其他 Web 服务（例如 Office 365 和 Microsoft Azure）的不足。

- - -

**问：所有者与全局管理员之间的差异是什么？**

**答:** 默认情况下，系统会将注册 Azure 订阅的人员指派为 Azure 资源的所有者角色。 所有者可以使用 Microsoft 帐户，也可以使用 Azure 订阅与之关联的目录中的工作或学校帐户。  此角色有权管理 Azure 门户中的服务。

如果其他人需要使用同一个订阅登录和访问服务，则可向他们分配相应的[内置角色](../../role-based-access-control/built-in-roles.md)。 有关其他信息，请参阅[使用 RBAC 和 Azure 门户管理访问权限](../../role-based-access-control/role-assignments-portal.md)。

默认情况下，系统会将注册 Azure 订阅的人员指派为目录的全局管理员角色。 全局管理员有权访问所有 Azure AD 目录功能。 Azure AD 提供一组不同的管理员角色，用于管理目录和标识相关的功能。 这些管理员将有权访问 Azure 门户中的各种功能。 管理员的角色决定了其所能执行的操作，例如创建或编辑用户、向其他用户分配管理角色、重置用户密码、管理用户许可证，或者管理域。  有关 Azure AD 目录管理员及其角色的其他信息，请参阅[在 Azure Active Directory 中向用户分配管理员角色](active-directory-users-assign-role-azure-portal.md)和[在 Azure Active Directory 中分配管理员角色](../users-groups-roles/directory-assign-admin-roles.md)。

另外，Azure AD 付费型服务（例如企业移动性 + 安全性）可通过综合性的企业级管理和安全解决方案来弥补其他 Web 服务（例如 Office 365 和 Microsoft Azure）的不足。

- - -
**问：是否可以通过报表来查看我的 Azure AD 用户许可证何时会过期？**

**答:** 不。  此功能目前不可用。

- - -

## <a name="get-started-with-hybrid-azure-ad"></a>混合 Azure AD 入门


**问：如果我已被添加为协作者，该如何离开原来的租户？**

**答:** 如果被作为协作者添加到另一组织的租户，可使用右上角的“租户切换器”在租户之间切换。  目前还无法主动离开邀请组织，Microsoft 正致力于提供该功能。  在该功能可用之前，可以要求邀请阻止你将从其租户中删除。
- - -
**问：如何将我的本地目录连接到 Azure AD？**

**答:** 可以使用 Azure AD Connect 将本地目录连接到 Azure AD。

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](../hybrid/whatis-hybrid-identity.md)。

- - -
**问：如何设置本地目录与云应用程序之间的 SSO？**

**答:** 只需在本地目录与 Azure AD 之间设置单一登录 (SSO)。 只要通过 Azure AD 访问云应用程序，该服务就会自动让用户使用其本地凭据正确进行身份验证。

可以通过联合身份验证解决方案（例如 Active Directory 联合身份验证服务 (AD FS)）或通过配置密码哈希同步，轻松地从本地实现 SSO。可以使用 Azure AD Connect 配置向导轻松部署这两个选项。

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](../hybrid/whatis-hybrid-identity.md)。

- - -
**问：Azure AD 是否为组织中的用户提供自助服务门户？**

**答:** 是的，Azure AD 提供 [Azure AD 访问面板](https://myapps.microsoft.com)，方便用户使用自助服务以及进行应用程序访问。 如果你是 Office 365 客户，你可以找到许多中相同的功能[Office 365 门户](https://portal.office.com)。

有关详细信息，请参阅[访问面板简介](../user-help/active-directory-saas-access-panel-introduction.md)。

- - -
**问：Azure AD 是否可以帮助我管理本地基础结构？**

**答:** 可以。 Azure AD Premium Edition 提供 Azure AD Connect Health。 Azure AD Connect Health 可帮助你监视和深入了解本地标识基础结构和同步服务。  

有关详细信息，请参阅[在云中监视本地标识基础结构和同步服务](../hybrid/whatis-hybrid-identity-health.md)。  

- - -
## <a name="password-management"></a>密码管理
**问：是否可以使用 Azure AD 密码写回但不使用密码同步？（在这种情况下，是否可以结合密码写回使用 Azure AD 自助密码重置 (SSPR)，而不将密码存储在云中？）**

**答:** 无需将 Active Directory 密码同步到 Azure AD 即可启用写回。 在联合环境中，Azure AD 单一登录 (SSO) 依赖本地目录对用户进行身份验证。 在这种情况下，并不需要在 Azure AD 中跟踪本地密码。

- - -
**问：需要多长时间才能将密码写回到 Active Directory 本地？**

**答:** 密码写回实时进行。

有关详细信息，请参阅[密码管理入门](../authentication/quickstart-sspr.md)。

- - -
**问：是否可以对管理员管理的密码使用密码写回？**

**答:** 可以。如果已启用密码写回，管理员执行的密码操作将写回到用户的本地环境。  

有关密码相关问题的详细解答，请参阅[密码管理常见问题](../authentication/active-directory-passwords-faq.md)。
- - -
**问：如果我在尝试更改 Office 365/Azure AD 密码时忘记了现有的密码，该怎么办？**

**答:** 对于这种情况，有几个选项。  在可行的情况下，使用自助服务密码重置 (SSPR)。  SSPR 是否适用取决于其配置方式。  有关详细信息，请参阅[密码重置门户的工作原理](../authentication/howto-sspr-deployment.md)。

对于 Office 365 用户，管理员可以使用 [Reset user passwords](https://support.office.com/article/Admins-Reset-user-passwords-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C?ui=en-US&rs=en-US&ad=US)（重置用户密码）中概述的步骤重置密码。

对于 Azure AD 帐户，管理员可以使用以下选项之一重置密码：

- [在 Azure 门户中重置帐户](active-directory-users-reset-password-azure-portal.md)
- [使用 PowerShell](/powershell/module/msonline/set-msoluserpassword?view=azureadps-1.0)


- - -
## <a name="security"></a>安全
**问：帐户在经过特定次数的失败尝试后被锁定还是使用了更复杂的策略？**

我们使用更复杂的策略来锁定帐户。  这基于请求的 IP 和输入的密码。 锁定的持续时间也会根据存在攻击的可能性而延长。  

**问：某些（通用）密码会被拒绝并且显示消息“此密码已使用了许多次”，这是否是指当前 Active Directory 中使用的密码？**

这指的是全局通用的密码，例如“Password”和“123456”的任何变体。

**问：B2C 租户中就会阻止来自可疑来源（僵尸网络、Tor 终结点）的登录请求还是需要使用基本或高级版租户才能阻止？**

我们有一个网关，它会筛选请求并针对僵尸网络提供一定的防护，它适用于所有 B2C 租户。

## <a name="application-access"></a>应用程序访问

**问：在哪里可以找到与 Azure AD 预先集成的应用程序及其功能的列表？**

**答:** Azure AD 中包含 Microsoft、应用程序服务提供商和合作伙伴提供的 2600 多个预先集成的应用程序。 所有预先集成的应用程序都支持单一登录 (SSO)。 SSO 允许用户使用组织凭据来访问应用。 某些应用程序还支持自动预配和自动取消预配。

有关预先集成的应用程序的完整列表，请参阅 [Active Directory 市场](https://azure.microsoft.com/marketplace/active-directory/)。

- - -
**问：如果 Azure AD 市场中没有我需要的应用程序怎么办？**

**答:** 使用 Azure AD Premium，可以添加和配置所需的任何应用程序。 可以根据应用程序的功能和喜好来配置 SSO 和自动预配。  

有关详细信息，请参阅：

* [针对不在 Azure Active Directory 应用程序库中的应用程序配置单一登录](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配](../manage-apps/use-scim-to-provision-users-and-groups.md)

- - -
**问：用户如何使用 Azure AD 来登录应用程序？**

**答:** Azure AD 提供多种方式供用户查看和访问其应用程序，例如：

* Azure AD 访问面板
* Office 365 应用程序启动器
* 直接登录联合应用
* 联合、基于密码或现有的应用的深层链接

有关详细信息，请参阅[应用程序的最终用户体验](../manage-apps/end-user-experiences.md)。

- - -
**问：Azure AD 可通过哪些不同的方式来启用对应用程序的身份验证和单一登录？**

**答:** Azure AD 支持使用多种标准化协议（例如 SAML 2.0、OpenID Connect、OAuth 2.0 和 WS 联合身份验证）进行身份验证和授权。 对于仅支持基于窗体的身份验证的应用程序，Azure AD 还支持密码保管和自动化登录功能。  

有关详细信息，请参阅：

* [Azure AD 的身份验证方案](../develop/authentication-scenarios.md)
* [Active Directory 身份验证协议](https://msdn.microsoft.com/library/azure/dn151124.aspx)
* [Azure AD 中应用程序的单一登录](../manage-apps/what-is-single-sign-on.md)

- - -
**问：是否可以添加本地运行的应用程序？**

**答:** Azure AD 应用程序代理可让你轻松安全地访问所选的本地 Web 应用程序。 可以像访问 Azure AD 中的软件即服务 (SaaS) 应用一样访问这些应用程序。 不需要设置 VPN 或更改网络基础结构。  

有关详细信息，请参阅[如何提供对本地应用程序的安全远程访问](../manage-apps/application-proxy.md)。

- - -
**问：如何要求访问特定应用程序的用户进行多重身份验证？**

**答:** 使用 Azure AD 条件访问，可以针对每个应用程序分配独特的访问策略。 可以在策略中要求用户始终进行多重身份验证，或者在未连接到本地网络时才进行。  

有关详细信息，请参阅[保护对 Office 365 和其他连接到 Azure Active Directory 的应用的访问](../active-directory-conditional-access-azure-portal.md)。

- - -
**问：什么是 SaaS 应用的自动化用户预配？**

**答:** 使用 Azure AD 可以在许多流行的云 (SaaS) 应用中自动创建、维护和删除用户标识。

有关详细信息，请参阅[使用 Azure Active Directory 自动预配和取消预配 SaaS 应用程序的用户](../manage-apps/user-provisioning.md)。

- - -
**问：是否可以通过 Azure AD 设置安全的 LDAP 连接？**

**答:** 不。 Azure AD 不支持轻型目录访问协议 (LDAP) 协议。 但是，就可以使用 Azure AD 域服务 (Azure AD DS) 与通过 Azure 网络的正确配置的网络安全组来实现 LDAP 连接。 有关详细信息，请参阅 https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-admin-guide-configure-secure-ldap 。