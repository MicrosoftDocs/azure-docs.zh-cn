---
title: Azure 标识和访问安全最佳实践 | Microsoft 文档
description: 本文提供一系列有关使用内置 Azure 功能进行标识管理和访问控制的最佳实践。
services: security
documentationcenter: na
author: barclayn
manager: barbkess
editor: TomSh
ms.assetid: 07d8e8a8-47e8-447c-9c06-3a88d2713bc1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/03/2019
ms.author: barclayn
ms.openlocfilehash: 2a669f5b46db4d5de7d1d6863b94e6c117667aee
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153244"
---
# <a name="azure-identity-management-and-access-control-security-best-practices"></a>Azure 标识管理和访问控制安全最佳实践
本文介绍一系列 Azure 标识管理和访问控制安全最佳实践。 这些最佳做法衍生自我们的 [Azure AD](../active-directory/fundamentals/active-directory-whatis.md) 经验和客户经验。

对于每项最佳做法，本文将说明：

* 最佳实践是什么
* 为何要启用该最佳实践
* 如果无法启用该最佳实践，可能的结果是什么
* 最佳实践的可能替代方案
* 如何学习启用最佳实践

这篇 Azure 标识管理和访问控制安全最佳实践以共识以及 Azure 平台功能和特性集（因为在编写本文时已存在）为基础。 看法和技术将随着时间改变，本文会定期更新以反映这些更改。

本文中介绍的 Azure 标识管理和访问控制安全最佳实践包括：

* 将标识视为主要安全边界
* 集中化标识管理
* 管理已连接的租户
* 启用单一登录
* 启用条件访问
* 启用密码管理
* 对用户强制执行多重身份验证
* 使用基于角色的访问控制
* 降低特权帐户的泄露风险
* 控制资源所在的位置
* 使用 Azure AD 进行存储身份验证

## <a name="treat-identity-as-the-primary-security-perimeter"></a>将标识视为主要安全边界

许多人认为标识是主要安全边界。 这与以网络安全为重点的传统做法不同。 网络边界出现越来越多的漏洞，在 [BYOD](https://aka.ms/byodcg) 设备和云应用程序激增之前，该边界防御不似之前那样有效。

[Azure Active Directory (Azure AD)](../active-directory/active-directory-whatis.md) 是用于标识和访问管理的 Azure 解决方案。 Azure AD 是 Microsoft 提供的多租户、基于云的目录和标识管理服务。 它将核心目录服务、应用程序访问管理和标识保护融入一个解决方案中。

以下部分列出了使用 Azure AD 实现标识和访问安全性的最佳做法。

## <a name="centralize-identity-management"></a>集中化标识管理

在[混合标识](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?)方案中，我们建议集成本地目录和云目录。 集成使你的 IT 团队管理从一个位置，而不考虑创建一个帐户的帐户。 集成还可帮助用户通过提供一个通用标识用于访问在本地和云资源来提高工作效率。

**最佳做法**：建立一个 Azure AD 实例。 一致性和单个权威源将提高清晰度，并从人为失误和配置的复杂性，降低安全风险。
**详细信息**：指定单个 Azure AD 目录视为企业和组织帐户的权威来源。

**最佳做法**：将本地目录与 Azure AD 进行集成。  
**详细信息**：使用 [Azure AD Connect](../active-directory/connect/active-directory-aadconnect.md) 将本地目录与云目录同步。

> [!Note]
> 有[因素会影响性能的 Azure AD Connect](../active-directory/hybrid/plan-connect-performance-factors.md)。 请确保 Azure AD Connect 具有足够的容量来保持系统的阻碍安全性和工作效率的绩效不佳。 大型或复杂的组织 （组织预配 100,000 多个对象） 应遵循[建议](../active-directory/hybrid/whatis-hybrid-identity.md)来优化其 Azure AD Connect 实现。

**最佳做法**：不要同步到 Azure AD 在现有的 Active Directory 实例中具有高权限的帐户。
**详细信息**：不要更改默认值[Azure AD Connect 配置](../active-directory/hybrid/how-to-connect-sync-configure-filtering.md)会筛选掉这些帐户。 此配置应从云正在切换到本地资产 （这可能会产生重大事件） 的攻击者的风险。

**最佳做法**：启用密码哈希同步。  
**详细信息**：密码哈希同步是一项功能用来同步用户密码的哈希从本地 Active Directory 实例与基于云的 Azure AD 实例。 此同步帮助防范泄漏的凭据在重播从先前的攻击。

即使决定使用 Active Directory 联合身份验证服务 (AD FS) 或其他标识提供者进行联合身份验证，也可以选择性地设置密码哈希同步作为备用机制，以应对本地服务器发生故障或临时不可用的情况。 此同步使用户能够使用他们用于登录到其本地 Active Directory 实例的同一密码登录到该服务。 它还允许 Identity Protection 检测通过比较已同步的密码哈希与已知遭到入侵，如果用户未连接到 Azure AD 的其他服务中使用的同一电子邮件地址和密码的密码已泄露的凭据。

有关详细信息，请参阅[使用 Azure AD Connect 同步实现密码哈希同步](../active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization.md)。

**最佳做法**：开发新的应用程序使用 Azure AD 进行身份验证。
**详细信息**：使用正确的功能来支持身份验证：

  - Azure AD 的员工
  - [Azure AD B2B](https://docs.microsoft.com/azure/active-directory/b2b/)对来宾用户和外部合作伙伴
  - [Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/)来控制客户如何进行注册，登录并在使用您的应用程序时管理其配置文件

未将其本地标识与云标识集成的组织在管理帐户方面可能开销更大。 这种开销增加了出错和安全漏洞的可能性。

> [!Note]
> 需要选择哪些帐户将位于并使用管理工作站是通过新的云服务托管的或现有处理关键的目录。 使用现有的管理和标识预配进程可能会降低一些风险，但也可以创建攻击者破坏的本地帐户，并切换到云的风险。 您可能想要对不同的角色 （例如，业务单元管理员与 IT 管理员） 使用不同的策略。 可以使用两个选项。 第一个选项是创建未与你的本地 Active Directory 实例同步的 Azure AD 帐户。 将管理工作站加入到 Azure AD 中，你可以管理和使用 Microsoft Intune 的修补程序。 第二个选项是使用现有的管理员帐户通过与你的本地 Active Directory 实例同步。 使用 Active Directory 域中的现有工作站，管理和安全。

## <a name="manage-connected-tenants"></a>管理已连接的租户
组织安全需要来评估风险并确定你的组织和任何法规要求的策略是否也要遵循的可见性。 您应确保组织安全具有可见性连接到你的生产环境和网络的所有订阅 (通过[Azure ExpressRoute](../expressroute/expressroute-introduction.md)或[站点到站点 VPN](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md))。 一个[全局管理员/公司管理员](../active-directory/users-groups-roles/directory-assign-admin-roles.md#company-administrator)在 Azure AD 中可以提升到其访问权限[用户访问管理员](../role-based-access-control/built-in-roles.md#user-access-administrator)角色并查看所有订阅和管理的组连接到你的环境。

请参阅[提升访问权限来管理所有 Azure 订阅和管理组](../role-based-access-control/elevate-access-global-admin.md)以确保您和您的安全组可以查看所有订阅或管理组连接到你的环境。 评估风险后，应删除此提升的访问权限。

## <a name="enable-single-sign-on"></a>启用单一登录

在移动优先、云优先的世界中，你希望能够从任意位置实现对设备、应用和服务的单一登录 (SSO)，以便你的用户随时随地都能高效工作。 如果要管理多个标识解决方案，则不仅会给 IT 人员造成管理问题，而且用户还必须记住多个密码。

通过对所有应用和资源使用相同的标识解决方案，可以实现 SSO。 并且不论资源是位于本地还是云中，用户均可以使用相同凭据集登录和访问所需资源。

**最佳做法**：启用 SSO。  
**详细信息**：Azure AD [将本地 Active Directory](../active-directory/connect/active-directory-aadconnect.md) 扩展到云。 用户可以将他们的主要工作或学校帐户用于他们加入域的设备、公司资源以及完成工作所需的所有 Web 和 SaaS 应用程序。 用户无需记住多组用户名和密码，系统会根据组织的组成员身份和员工身份的状态，自动预配（或取消设置）应用程序访问权限。 可以针对库应用或者通过 [Azure AD 应用程序代理](../active-directory/active-directory-application-proxy-get-started.md)自行开发和发布的本地应用控制访问权限。

用户可使用 SSO 基于 Azure AD 中的工作或学校帐户访问 [SaaS 应用程序](../active-directory/active-directory-appssoaccess-whatis.md)。 这不仅适用于 Microsoft SaaS 应用，还适用于其他应用，例如 [Google Apps](../active-directory/active-directory-saas-google-apps-tutorial.md) 和 [Salesforce](../active-directory/active-directory-saas-salesforce-tutorial.md)。 应用程序可配置为使用 Azure AD 作为[基于 SAML 的标识](../active-directory/fundamentals-identity.md)提供者。 作为安全控制机制，Azure AD 不会发出允许用户登录应用程序的令牌，除非用户已通过 Azure AD 获取了访问权限。 可以直接或者通过用户所属的组授予访问权限。

如果组织没有通过创建通用标识来为用户和应用程序实现 SSO，那么用户拥有多个密码的情况就更容易出现。 这种情况增加了用户重复使用同一密码或使用弱密码的可能性。

## <a name="turn-on-conditional-access"></a>启用条件访问

用户可能会从任意位置使用各种设备和应用访问组织的资源。 作为 IT 管理员，你想要确保这些设备符合标准的安全性和符合性。 仅关注谁可以访问资源不再能满足需求。

若要平衡安全性和工作效率，您需要考虑之前您可以决定如何对访问控制访问资源的方式。 使用 Azure AD 条件访问便可处理该需求。 使用条件性访问，可以根据用于访问你的云应用的条件自动的访问控制决策。

**最佳做法**：管理和控制对公司资源的访问。  
**详细信息**：根据组、位置和应用程序敏感性为 SaaS 应用和 Azure AD 连接应用配置 Azure AD [条件访问](../active-directory/active-directory-conditional-access-azure-portal.md)。

**最佳做法**：阻止旧式身份验证协议。
**详细信息**：攻击者利用较旧的协议中的弱点每日，特别是对于密码喷射攻击。 配置条件访问阻止旧版协议。 请观看视频[Azure AD:注意事项](https://www.youtube.com/watch?v=wGk0J4z90GI)有关详细信息。

## <a name="enable-password-management"></a>启用密码管理

如果有多个租户或者你想要允许用户[重置自己的密码](../active-directory/active-directory-passwords-update-your-own-password.md)，则必须使用适当的安全策略来防止滥用。

**最佳做法**：为用户设置自助式密码重置 (SSPR)。  
**详细信息**：使用 Azure AD [自助式密码重置](../active-directory-b2c/active-directory-b2c-reference-sspr.md)功能。

**最佳做法**：监视是否在使用 SSPR 及其使用情况。  
**详细信息**：通过使用 Azure AD [密码重置注册活动报表](../active-directory/active-directory-passwords-get-insights.md)监视正在注册的用户。 Azure AD 提供的报表功能可帮助使用预生成的报表来回答问题。 如果有相应的授权，还可以创建自定义查询。

**最佳做法**：扩展到你的本地基础结构的基于云的密码策略。
**详细信息**：通过像对基于云的密码更改的本地密码更改执行相同检查增强你的组织中的密码策略。 安装[Azure AD 密码保护](../active-directory/authentication/concept-password-ban-bad.md)为 Windows Server Active Directory 代理在本地扩展到现有基础结构的阻止的密码列表。 用户和管理员更改，请设置，或重置的密码的本地所需遵循仅限云的用户相同的密码策略。

## <a name="enforce-multi-factor-verification-for-users"></a>对用户强制执行多重身份验证

建议对所有用户要求进行双重验证。 这包括组织中的管理员和其他人员，如果他们的帐户泄露，可能会产生重大影响（例如，财务官员）。

要求双重验证有多种选项。 最佳选项取决于你的目标、正在运行的 Azure AD 版本以及许可计划。 请参阅[如何要求对用户进行双重验证](../active-directory/authentication/howto-mfa-userstates.md)了解最佳选项。 有关许可和定价的详细信息，请参阅 [Azure AD](https://azure.microsoft.com/pricing/details/active-directory/) 和 [Azure 多重身份验证](https://azure.microsoft.com/pricing/details/multi-factor-authentication/)定价页。

以下是启用双重验证的选项和优势：

**选项 1**：[通过更改用户状态启用多重身份验证](../active-directory/authentication/howto-mfa-userstates.md)。   
**优势**：这是要求进行双重验证的传统方法。 它适用于[云中的 Azure 多重身份验证和 Azure 多重身份验证服务器](../active-directory/authentication/concept-mfa-whichversion.md)。 使用此方法要求用户每次登录时都执行双重验证并重写条件访问策略。

若要确定需要启用多因素身份验证，请参阅[最适合我的组织是哪个版本的 Azure MFA？](../active-directory/authentication/concept-mfa-whichversion.md)。

**选项 2**：[通过条件访问策略启用多重身份验证](../active-directory/authentication/howto-mfa-getstarted.md)。
**优势**：此选项允许使用[条件访问](../active-directory/active-directory-conditional-access-azure-portal.md)在特定条件下提示进行双重验证。 特定条件可以是用户从不同位置、不受信任的设备或你认为存在风险的应用程序登录。 定义要求双重验证的特定条件可以避免不断提示用户这种令人不快的用户体验。

这是为用户启用双重验证最灵活的方式。 启用条件访问策略仅适用于云中的 Azure 多重身份验证，并且是 Azure AD 的高级功能。 有关此方法的详细信息，请参阅[部署基于云的 Azure 多重身份验证](../active-directory/authentication/howto-mfa-getstarted.md)。

**选项 3**：通过评估 [Azure AD Identity Protection](../active-directory/authentication/tutorial-risk-based-sspr-mfa.md) 的用户和登录风险，通过条件访问策略启用多重身份验证。   
**优势**：此选项使你能够：

- 检测影响组织标识的潜在漏洞。
- 配置自动响应与组织标识相关的可疑操作。
- 调查可疑事件，并采取适当的措施进行解决。

此方法使用 Azure AD Identity Protection 风险评估来确定是否需要基于所有云应用程序的用户和登录风险进行双重验证。 此方法需要 Azure Active Directory P2 授权。 有关此方法的详细信息，请参阅 [Azure Active Directory Identity Protection](../active-directory/identity-protection/overview.md)。

> [!Note]
> 选项 1 通过更改用户状态启用多重身份验证来替代条件访问策略。 因为选项 2 和 3 使用条件访问策略，所以无法将选项 1 与其他两个选项结合使用。

未添加额外标识保护层（如双重验证）的组织将更容易受到凭据窃取攻击。 凭据窃取攻击可能导致数据泄漏。

## <a name="use-role-based-access-control"></a>使用基于角色的访问控制
云资源的访问管理的组织，使用云至关重要。 [基于角色的访问控制 (RBAC)](../role-based-access-control/overview.md)可帮助你管理谁有权访问 Azure 资源、 他们可以对这些资源执行哪些操作和他们有权访问哪些领域。

指定组或单个角色负责在 Azure 中的特定功能有助于避免混淆，可能会导致人为和创建的安全风险的自动化错误。 对于想要实施数据访问安全策略的组织而言，必须根据[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低权限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全策略限制访问权限。

安全团队需要 Azure 资源以评估和修正风险的可见性。 如果安全团队具有的运行责任，他们需要其他权限完成其工作。

可以使用[RBAC](../role-based-access-control/overview.md)将权限分配给用户、 组和特定范围内的应用程序。 角色分配的范围可以是订阅、资源组或单个资源。

**最佳做法**：对在团队中的职责进行分配，并向执行其作业所需的用户授予的访问量。 而不是向每个人提供 Azure 订阅或资源中不受限制的权限仅允许某些操作在特定范围内。
**详细信息**：使用[内置 RBAC 角色](../role-based-access-control/built-in-roles.md)在 Azure 中向用户分配权限。

> [!Note]
> 特定权限创建不必要的复杂性和混淆，累积到很难修复而无需担心弄坏东西"传统"配置。 避免特定于资源的权限。 相反，使用管理组的企业级权限和资源组以获取订阅中的权限。 避免特定于用户的权限。 相反，在 Azure AD 中向组分配访问权限。

**最佳做法**：授予安全团队与 Azure 的责任以便它们可以评估和修正风险，请参阅 Azure 资源的访问权限。
**详细信息**：向安全团队授予 RBAC[安全读取者](../role-based-access-control/built-in-roles.md#security-reader)角色。 可以使用根管理组或段管理组，具体取决于的职责范围：

- **根管理组**团队负责所有企业资源
- **分段管理组**的团队 （通常由于法规或其他组织边界） 限制的作用域

**最佳做法**：授予安全团队拥有直接操作职责到适当的权限。
**详细信息**：查看相应的角色分配的 RBAC 内置角色。 如果内置角色无法满足你的组织的特定需求，可以创建[Azure 资源的自定义角色](../role-based-access-control/custom-roles.md)。 如使用内置角色中，您可以将自定义角色分配到用户、 组和订阅、 资源组和资源范围内的服务主体。

**最佳做法**：授予 Azure 安全中心需要它的安全角色的访问。 安全中心，让安全团队能够快速识别和修正风险。
**详细信息**：将这些需求的安全团队添加到 RBAC[安全管理员](../role-based-access-control/built-in-roles.md#security-admin)角色以便他们可以查看安全策略、 查看安全状态、 编辑安全策略、 查看警报和建议，并关闭警报和建议。 您可以使用根管理组或段管理组，具体取决于的职责范围来执行此操作。

通过使用功能等 RBAC 可能会给分配超过需要它们的用户权限的组织未强制执行数据访问控制。 这可能会导致数据泄漏通过允许用户访问他们不应有的数据 （例如，高业务影响） 的类型。

## <a name="lower-exposure-of-privileged-accounts"></a>降低特权帐户的泄露风险

保护特权访问是保护业务资产的首要步骤。 减少拥有访问权限的人员以保护信息或资源安全，这样可以减小恶意用户获得访问权限，或者已授权用户无意中影响敏感资源的可能性。

特权帐户是指掌控和管理 IT 系统的帐户。 网络攻击者会攻击这些帐户来获取组织数据和系统的访问权限。 为了保护特权访问，应隔离此类帐户和系统，使其免受恶意用户的威胁。

建议制定并遵循一个路线图，防止特权访问受到网络攻击者的攻击。 有关创建详细路线图以保护在 Azure AD、Microsoft Azure、Office 365 和其他云服务中托管或报告的标识和访问的信息，请查看[确保 Azure AD 中混合部署和云部署的特权访问安全性](../active-directory/users-groups-roles/directory-admin-roles-secure.md)。

以下内容总结了[确保 Azure AD 中混合部署和云部署的特权访问安全性](../active-directory/users-groups-roles/directory-admin-roles-secure.md)中介绍的最佳做法：

**最佳做法**：管理、控制和监视对特权帐户的访问。   
**详细信息**：启用 [Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/active-directory-securing-privileged-access.md)。 启用 Privileged Identity Management 以后，会收到有关特权访问角色更改的通知电子邮件。 向目录中的高特权角色添加更多用户时，这些通知相当于早期警告。

**最佳做法**：请确保所有关键的管理帐户管理 Azure AD 帐户。
**详细信息**：从关键管理员角色 （例如，hotmail.com、 live.com 和 outlook.com 等 Microsoft 帐户） 中删除任何使用者帐户。

**最佳做法**：请确保所有关键的管理员角色可以用于管理任务的单独帐户以避免网络钓鱼和其他攻击破坏管理权限。
**详细信息**：创建单独的管理员帐户分配执行管理任务所需的权限。 阻止这些管理帐户用于每日的工作效率工具，例如 Microsoft Office 365 电子邮件或任意 web 浏览。

**最佳做法**：对特许权限高的角色中的帐户进行标识和分类。   
**详细信息**：启用 Azure AD Privileged Identity Management 后，请查看角色为全局管理员、特权角色管理员和其他高特权角色的用户。 请删除在这些角色中不再需要的任何帐户，并对剩余的分配给管理员角色的帐户分类：

- 单独分配给管理用户，可用于非管理性目的（例如，个人电子邮件）
- 单独分配给管理用户，按规定只能用于管理目的
- 跨多个用户共享
- 适用于紧急访问情况
- 适用于自动化脚本
- 适用于外部用户

**最佳做法**：实行“实时”(JIT) 访问可进一步降低特权的曝光时间，并提高对特权帐户使用情况的可见性。   
**详细信息**：利用 Azure AD Privileged Identity Management，可以：

- 限制用户只接受他们的权限 JIT。
- 分配时限更短的角色，确信权限会自动撤消。

**最佳做法**：定义至少两个紧急访问帐户。   
**详细信息**：可以使用紧急访问帐户来帮助组织限制现有 Azure Active Directory 环境中的特权访问。 这些帐户拥有极高的特权，不要将其分配给特定的个人。 紧急访问帐户只能用于不能使用正常管理帐户的情况。 组织必须将紧急账户的使用限制在必要时间范围内。

评估已经获得或有资格获得全局管理员角色的帐户。 如果使用 `*.onmicrosoft.com` 域（用于紧急访问）看不到任何仅限云的帐户，请创建此类帐户。 有关详细信息，请参阅[在 Azure AD 中管理紧急访问管理帐户](../active-directory/users-groups-roles/directory-emergency-access.md)。

**最佳做法**：在发生紧急情况下时的位置具有"不受限"过程。
**详细信息**：按照中的步骤[保护特权访问在 Azure AD 中混合和云部署](../active-directory/users-groups-roles/directory-admin-roles-secure.md)。

**最佳做法**：需要为无密码的所有关键的管理员帐户 （首选），或需要多重身份验证。
**详细信息**：使用[Microsoft Authenticator 应用](../active-directory/authentication/howto-authentication-phone-sign-in.md)而无需使用密码登录到任何 Azure AD 帐户。 像[Windows hello 企业版](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification)，Microsoft 验证器使用基于密钥的身份验证来启用它绑定到设备，并使用生物识别身份验证或 PIN 的用户凭据。

需要在登录的所有单个用户永久分配给一个或多个 Azure AD 管理员角色的 Azure 多重身份验证：全局管理员、 特权角色管理员、 Exchange Online 管理员和 SharePoint Online 管理员。 启用[针对管理员帐户的多重身份验证](../active-directory/authentication/howto-mfa-userstates.md)并确保管理员帐户用户都已注册。

**最佳做法**：对于关键的管理员帐户，具有管理工作站的生产任务不允许 （适用于示例、 浏览和电子邮件）。 这将从使用浏览和电子邮件，并显著降低出现重大事件的风险的攻击手段来保护管理员帐户。
**详细信息**：使用管理工作站。 选择工作站安全的级别：

- 高度安全的工作效率的设备提供用于浏览的高级的安全性和工作效率的其他任务。
- [特权访问工作站 (Paw)](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)提供专用的操作系统免受 internet 攻击和威胁向量为敏感任务。

**最佳做法**：当员工离开组织时，取消设置管理员帐户。
**详细信息**：在禁用或当员工离开组织时删除管理员帐户的位置有一个进程。

**最佳做法**：使用当前的攻击技术，定期测试管理员帐户。
**详细信息**：使用 Office 365 攻击模拟器或第三方产品/服务来运行你的组织中的实际攻击方案。 这可以帮助您找到易受攻击用户实际攻击发生之前。

**最佳做法**：采取措施来缓解最常用的攻击技术的冲击。  
**详细信息**：[确定管理角色中那些需要切换到工作或学校帐户的 Microsoft 帐户](../active-directory/users-groups-roles/directory-admin-roles-secure.md#identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts)  

[对于全局管理员帐户，请确保使用单独的用户帐户和邮件转发功能](../active-directory/users-groups-roles/directory-admin-roles-secure.md)  

[确保最近更改过管理帐户的密码](../active-directory/users-groups-roles/directory-admin-roles-secure.md#ensure-the-passwords-of-administrative-accounts-have-recently-changed)  

[启用密码哈希同步](../active-directory/users-groups-roles/directory-admin-roles-secure.md#turn-on-password-hash-synchronization)  

[要求对所有特权角色用户和公开的用户进行多重身份验证](../active-directory/users-groups-roles/directory-admin-roles-secure.md#require-multi-factor-authentication-mfa-for-users-in-all-privileged-roles-as-well-as-exposed-users)  

[获取 Office 365 安全功能分数（如果使用 Office 365）](../active-directory/users-groups-roles/directory-admin-roles-secure.md#obtain-your-office-365-secure-score-if-using-office-365)  

[查看 Office 365 安全性和符合性指南（如果使用 Office 365）](../active-directory/users-groups-roles/directory-admin-roles-secure.md#review-the-office-365-security-and-compliance-guidance-if-using-office-365)  

[配置 Office 365 活动监视（如果使用 Office 365）](../active-directory/users-groups-roles/directory-admin-roles-secure.md#configure-office-365-activity-monitoring-if-using-office-365)  

[确定事件/紧急情况响应计划所有者](../active-directory/users-groups-roles/directory-admin-roles-secure.md#establish-incidentemergency-response-plan-owners)  

[保护本地特权管理帐户](../active-directory/users-groups-roles/directory-admin-roles-secure.md#turn-on-password-hash-synchronization)

如果不保护特权访问，你可能会拥有过多高特权角色用户，并且更易受到攻击。 恶意操作者（包括网络攻击者）通常会以管理员帐户和特权访问的其他元素为目标，通过凭据窃取获得敏感数据和系统的访问权限。

## <a name="control-locations-where-resources-are-created"></a>控制创建资源的位置

非常重要的一点是，既要允许云操作员执行任务，同时又要防止他们违反管理组织资源所需的惯例。 想要控制创建资源的位置的组织应该对这些位置进行硬编码。

可以使用 [Azure 资源管理器](../azure-resource-manager/resource-group-overview.md)创建安全策略，其中的定义描述了会明确遭到拒绝的操作或资源。 可以在所需范围（例如订阅、资源组或是单个资源）分配这些策略定义。

> [!NOTE]
> 安全策略与 RBAC 不同。 它们实际上使用 RBAC 授权用户来创建这些资源。
>
>

无法控制资源创建方式的组织更容易因用户创建的资源超过所需数目，而产生滥用服务的情况。 强化资源创建过程是保护多租户方案的重要步骤。

## <a name="actively-monitor-for-suspicious-activities"></a>主动监视可疑活动

主动身份监视系统可以快速检测可疑行为并触发警报以进行进一步调查。 下表列出了两个可帮助组织监视其标识的 Azure AD 功能：

**最佳做法**：采用一种方法来确定：

- [未受跟踪](../active-directory/active-directory-reporting-sign-ins-from-unknown-sources.md)的登录尝试。
- 针对特定帐户的[暴力](../active-directory/active-directory-reporting-sign-ins-after-multiple-failures.md)攻击。
- 从多个位置的登录尝试。
- 从[受感染的设备](../active-directory/active-directory-reporting-sign-ins-from-possibly-infected-devices.md)登录。
- 可疑 IP 地址。

**详细信息**：使用 Azure AD Premium [异常报告](../active-directory/active-directory-view-access-usage-reports.md)。 制定相应的流程和过程，使 IT 管理员每天或按需（通常在事件响应方案中）运行这些报告。

**最佳做法**：安装一个主动监视系统，用于通知风险，并且可以根据业务需求调整风险等级（高、中或低）。   
**详细信息**：使用 [Azure AD Identity Protection](../active-directory/active-directory-identityprotection.md)，它会在自己的仪表板上标记当前风险并通过电子邮件发送每日摘要通知。 要帮助保护组织的标识，可以配置基于风险的策略，该策略可在达到指定风险级别时自动响应检测到的问题。

不主动监视其标识系统的组织将面临用户凭据泄露的风险。 如果不知道有人通过这些凭据实施可疑活动，组织就无法缓解这种类型的威胁。

## <a name="use-azure-ad-for-storage-authentication"></a>使用 Azure AD 进行存储身份验证
[Azure 存储](../storage/common/storage-auth-aad.md)支持 Blob 存储和队列存储的身份验证和授权与 Azure AD。 Azure AD 身份验证，可以使用 Azure 基于角色的访问控制向用户、 组和单个 blob 容器或队列的作用域下的应用程序授予特定权限。

我们建议你使用[Azure AD 进行身份验证访问存储](https://azure.microsoft.com/blog/azure-storage-support-for-azure-ad-based-access-control-now-generally-available/)。

## <a name="next-step"></a>后续步骤

有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。
