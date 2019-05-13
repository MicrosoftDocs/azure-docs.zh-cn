---
title: 什么是 Azure Active Directory？ - Azure Active Directory | Microsoft Docs
description: 有关 Azure Active Directory 的概述和概念信息，包括术语、可用的许可证、相关功能的列表及其详细信息链接。
services: active-directory
author: eross-msft
manager: daveba
ms.service: active-directory
ms.topic: overview
ms.date: 05/08/2019
ms.author: lizross
ms.custom: it-pro, seodec18, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: ce8fad7f0dc76aad306e0f2a8e26692ec997952c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470363"
---
# <a name="what-is-azure-active-directory"></a>什么是 Azure Active Directory？

Azure Active Directory (Azure AD) 是 Microsoft 推出的基于云的标识和访问管理服务，可帮助员工登录及访问以下位置的资源：

- 外部资源，例如 Microsoft Office 365、Azure 门户以及成千上万的其他 SaaS 应用程序。

- 内部资源，例如公司网络和 Intranet 上的应用，以及由自己的组织开发的任何云应用。

可以通过各种[面向企业架构师的 Microsoft 云系列](https://docs.microsoft.com/office365/enterprise/microsoft-cloud-it-architecture-resources#identity)海报更好地了解 Azure、Azure AD 和 Office 365 中的核心标识服务。

## <a name="who-uses-azure-ad"></a>谁在使用 Azure AD？

Azure AD 适用于：

- **IT 管理员。** 作为 IT 管理员，你可以使用 Azure AD 根据业务要求控制用户对你的应用和应用资源的访问。 例如，可以使用 Azure AD 要求用户在访问重要的组织资源时进行多重身份验证。 另外，还可以使用 Azure AD 在现有 Windows Server AD 和云应用（包括 Office 365）之间自动完成用户预配。 最终可以利用 Azure AD 提供的强大工具自动保护用户标识和凭据，实现访问管理要求。 若要开始尝试，请注册 [30 天 Azure Active Directory Premium 免费试用版](https://azure.microsoft.com/trial/get-started-active-directory/)。

- **应用开发人员。** 作为应用开发人员，你可以通过 Azure AD 以基于标准的方式向应用添加单一登录 (SSO) 功能，使之能够与用户的预先存在的凭据配合使用。 另外还可以通过 Azure AD 提供的 API 来构建个性化应用体验，充分使用现有的组织数据。 若要开始尝试，请注册 [30 天 Azure Active Directory Premium 免费试用版](https://azure.microsoft.com/trial/get-started-active-directory/)。 有关详细信息，还可以参阅[针对开发人员的 Azure Active Directory](../develop/index.yml)。

- **Microsoft 365、Office 365、Azure 或 Dynamics CRM Online 订阅者。** 作为订阅者，你已在使用 Azure AD。 每个 Microsoft 365、Office 365、Azure 和 Dynamics CRM Online 租户都会自动成为 Azure AD 租户。 你可以立即开始管理用户对集成云应用的访问。

## <a name="what-are-the-azure-ad-licenses"></a>什么是 Azure AD 许可证？

Microsoft Online 业务服务（例如 Office 365 或 Microsoft Azure）要求通过 Azure AD 来完成登录操作并进行标识保护。 如果订阅任何 Microsoft Online 业务服务，则会自动获得 Azure AD 并且能够访问所有免费功能。

为了增强 Azure AD 实现，还可以通过升级到 Azure Active Directory Basic、Premium P1 或 Premium P2 许可证添加付费功能。 Azure AD 付费许可证建立在现有免费目录基础之上，提供自助服务、增强型监视、安全报告和移动用户安全访问。

>[!Note]
>有关这些许可证的定价选项，请参阅 [Azure Active Directory 定价](https://azure.microsoft.com/pricing/details/active-directory/)。
>
>中国地区目前不支持 Azure Active Directory Premium P1、Premium P2 和 Azure Active Directory Basic。 有关 Azure AD 定价的详细信息，请访问 [Azure Active Directory 论坛](https://azure.microsoft.com/support/community/?product=active-directory)。

- **Azure Active Directory Free。** 跨 Azure、Office 365 和许多常用的 SaaS 应用提供用户和组管理、本地目录同步、基本报表以及单一登录功能。

- **Azure Active Directory Basic。** 除了 Free 版的功能，Basic 版还提供以云为中心的应用访问、基于组的访问管理、用于云应用的自助密码重置，以及 Azure AD 应用程序代理（使用 Azure AD 发布本地 Web 应用）功能。

- **Azure Active Directory Premium P1。** 除了 Free 和 Basic 版功能，P1 还允许混合用户访问本地资源和云资源。 它还支持高级管理，例如动态组、自助服务组管理、Microsoft Identity Manager（一个本地标识与访问管理套件），以及允许本地用户进行自助密码重置的云写回功能。

- **Azure Active Directory Premium P2。** 除了 Free、Basic 和 P1 版功能，P2 还提供 [Azure Active Directory Identity Protection](../identity-protection/enable.md)，用于对应用和重要的公司数据进行基于风险的条件访问，以及提供 [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md)，用于发现、限制和监视管理员及其对资源的访问，并在需要时提供实时访问。

- **“即用即付”功能许可证。** 也可获取其他功能许可证，例如 Azure Active Directory 企业对客户 (B2C) 许可证。 可以通过 B2C 为面向客户的应用提供标识和访问管理解决方案。 有关详细信息，请参阅 [Azure Active Directory B2C 文档](../../active-directory-b2c/index.yml)。

有关将 Azure 订阅关联到 Azure AD 的详细信息，请参阅[如何：将 Azure 订阅关联或添加到 Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)以及有关将许可证分配给用户的详细信息，请参阅[如何：分配或删除 Azure Active Directory 许可证](license-users-groups.md)。

## <a name="terminology"></a>术语

为了更好地理解 Azure AD 及其文档，我们建议查看以下术语。

|术语或概念|描述|
|---------------|-----------|
|Azure 订阅| 用于为 Azure 云服务付费。 可以有多个订阅，这些订阅与一张信用卡关联。|
|Azure 租户| 组织在注册 Microsoft Azure、Microsoft Intune 或 Office 365 等 Microsoft 云服务订阅时自动创建的专用且受信任的 Azure AD 实例。 一个 Azure 租户表示一个组织。|
|单租户| 可以将访问专用环境中的其他服务的 Azure 租户视为单租户。|
|多租户| 可以将访问共享环境中的其他服务的 Azure 租户（跨多个组织）视为多租户。|
|Azure AD 目录|每个 Azure 租户都有一个专用且受信任的 Azure AD 目录。 Azure AD 目录包括租户的用户、组和应用，用于针对租户资源执行标识和访问管理功能。|
|Azure AD 帐户 | 通过 Azure AD 或其他 Microsoft 云服务（例如 Office 365）创建的标识。 标识存储在 Azure AD 中，可供组织的云服务订阅访问。 此帐户有时也称为工作或学校帐户。|
|自定义域|每个新的 Azure AD 目录都附带了初始域名 domainname.onmicrosoft.com。 除了该初始名称，还可以向列表添加组织的域名，其中包括用来开展业务的名称以及用户用来访问组织资源的名称。 添加自定义域名有助于创建用户所熟悉的用户名，例如 alain@contoso.com。|
|帐户管理员|从概念上讲，此经典订阅管理员角色是订阅的账单所有者。 此角色可以访问 [Azure 帐户中心](https://account.azure.com/Subscriptions)，用于管理一个帐户中的所有订阅。 有关详细信息，请参阅[经典订阅管理员角色、Azure 基于角色的访问控制 (RBAC) 角色和 Azure AD 管理员角色](../../role-based-access-control/rbac-and-directory-admin-roles.md)。|
|服务管理员|此经典订阅管理员角色用于管理所有 Azure 资源，包括访问权限。 此角色拥有在订阅范围内分配有“所有者”角色的用户的等效访问权限。 有关详细信息，请参阅[经典订阅管理员角色、Azure RBAC 角色和 Azure AD 管理员角色](../../role-based-access-control/rbac-and-directory-admin-roles.md)。|
|所有者|此角色有助于管理所有 Azure 资源，包括访问权限。 此角色在称为基于角色的访问控制 (RBAC) 的较新授权系统上构建，该系统可提供对 Azure 资源的精细访问管理。 有关详细信息，请参阅[经典订阅管理员角色、Azure RBAC 角色和 Azure AD 管理员角色](../../role-based-access-control/rbac-and-directory-admin-roles.md)。|
|Azure AD 全局管理员|此管理员角色自动分配给创建 Azure AD 租户的人员。 全局管理员可以执行 Azure AD 以及与 Azure AD 联合的任意服务（例如 Exchange Online、SharePoint Online 和 Skype for Business Online）的所有管理功能。 可以有多个全局管理员，但只有全局管理员才能向用户分配管理员角色（包括分配其他全局管理员）。<br><br>**注意**<br>此管理员角色在 Azure 门户中称为“全局管理员”，但在 Microsoft Graph API、Azure AD Graph API 和 Azure AD PowerShell 中称为“公司管理员”。<br><br>有关各种管理员角色的详细信息，请参阅 [Azure Active Directory 中的管理员角色权限](../users-groups-roles/directory-assign-admin-roles.md)。|
|Microsoft 帐户（也称 MSA）|个人帐户，用于访问面向使用者的 Microsoft 产品和云服务，例如 Outlook、OneDrive、Xbox LIVE 或 Office 365。 Microsoft 帐户在由 Microsoft 运行的 Microsoft 使用者标识帐户系统中创建和存储。|

## <a name="which-features-work-in-azure-ad"></a>哪些功能可以在 Azure AD 中使用？

在选择 Azure AD 许可证以后，即可访问下面这些适用于组织的部分或所有功能：

|类别|描述|
|-------|-----------|
|应用程序管理|使用应用程序代理、单一登录、“我的应用”门户（也称“访问面板”）和软件即服务 (SaaS) 应用来管理云应用和本地应用。 有关详细信息，请参阅[如何提供对本地应用程序的安全远程访问](../manage-apps/application-proxy.md)和[应用程序管理文档](../manage-apps/index.yml)。|
|Authentication|管理 Azure Active Directory 自助密码重置、多重身份验证、自定义禁止密码列表和智能锁定。 有关详细信息，请参阅 [Azure AD 身份验证文档](../authentication/index.yml)。|
|企业对企业 (B2B)|管理来宾用户和外部合作伙伴，同时保持对自己公司数据的控制。 有关详细信息，请参阅 [Azure Active Directory B2B 文档](../b2b/index.yml)。|
|企业对客户 (B2C)|自定义并控制用户在使用应用时如何注册、登录并管理其配置文件。 有关详细信息，请参阅 [Azure Active Directory B2C 文档](../../active-directory-b2c/index.yml)。|
|条件性访问|管理对云应用进行的访问。 有关详细信息，请参阅 [Azure AD 条件访问文档](../conditional-access/index.yml)。|
|针对开发人员的 Azure Active Directory|生成应用，以便进行所有 Microsoft 标识的登录，以及获取令牌来调用 Microsoft Graph、其他 Microsoft API 或自定义 API。 有关详细信息，请参阅 [Microsoft 标识平台（针对开发人员的 Azure Active Directory）](../develop/index.yml)。|
|设备管理|管理云设备或本地设备访问企业数据的方式。 有关详细信息，请参阅 [Azure AD 设备管理文档](../devices/index.yml)。|
|域服务|在不使用域控制器的情况下将 Azure 虚拟机加入域。 有关详细信息，请参阅 [Azure AD 域服务文档](../../active-directory-domain-services/index.yml)。|
|企业用户|使用组和管理员角色管理许可证分配、访问应用以及设置委托。 有关详细信息，请参阅 [Azure Active Directory 用户管理文档](../users-groups-roles/index.yml)。|
|混合标识|使用 Azure Active Directory Connect 和 Connect Health 提供单一用户标识，以便针对所有资源进行身份验证和授权，而不考虑位置（云或本地）。 有关详细信息，请参阅[混合标识文档](../hybrid/index.yml)。|
|标识治理|通过员工、业务合作伙伴、供应商、服务和应用访问控制管理组织的标识。 还可执行访问评审。 有关详细信息，请参阅 [Azure AD 标识治理文档](../governance/identity-governance-overview.md)和 [Azure AD 访问评审](../governance/access-reviews-overview.md)。|
|标识保护|检测影响组织标识的潜在漏洞，配置用于响应可疑操作的策略，然后采取相应的解决措施。 有关详细信息，请参阅 [Azure AD Identity Protection](../identity-protection/index.yml)。|
|Azure 资源的托管标识|在 Azure AD 中为 Azure 服务提供可以对任何 Azure AD 支持的身份验证服务（包括 Key Vault）进行身份验证的自动托管标识。 有关详细信息，请参阅[什么是 Azure 资源的托管标识？](../managed-identities-azure-resources/overview.md)。|
|Privileged Identity Management (PIM)|管理、控制和监视组织内的访问。 此功能包括访问 Azure AD、Azure 和其他 Microsoft Online Services（例如 Office 365 或 Intune）中的资源。 有关详细信息，请参阅 [Azure AD Privileged Identity Management](../privileged-identity-management/index.yml)。|
|报表和监视|了解环境中的安全性和使用模式。 有关详细信息，请参阅 [Azure Active Directory 报表和监视](../reports-monitoring/index.yml)。|

## <a name="next-steps"></a>后续步骤

- [注册 Azure Active Directory Premium](active-directory-get-started-premium.md)

- [将 Azure 订阅关联到 Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

- [访问 Azure Active Directory 并创建新租户](active-directory-access-create-new-tenant.md)

- [Azure Active Directory Premium P2 功能部署清单](active-directory-deployment-checklist-p2.md)
