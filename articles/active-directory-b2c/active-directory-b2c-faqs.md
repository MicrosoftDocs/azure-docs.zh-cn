---
title: 有关 Azure Active Directory B2C 的常见问题解答 | Microsoft Docs
description: 有关 Azure Active Directory B2C 的常见问题解答 (FAQ)。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/30/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 0c837ce66656c0e8a6a15d359c76459b03c2d0dd
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64722776"
---
# <a name="azure-ad-b2c-frequently-asked-questions-faq"></a>Azure AD B2C：常见问题 (FAQ) 
此页面解答了有关 Azure Active Directory (Azure AD) B2C 的常见问题。 请随时返回查看更新信息。

### <a name="why-cant-i-access-the-azure-ad-b2c-extension-in-the-azure-portal"></a>为什么我在 Azure 门户中无法访问 Azure AD B2C 扩展？
如果 Azure AD 扩展无法正常工作，通常有两个原因。  Azure AD B2C 要求你在目录中具备全局管理员的用户角色。  如果你认为应具有访问权限，请与管理员联系。  如果你拥有全局管理员权限，请确保处于 Azure AD B2C 目录（而不是 Azure Active Directory 目录）中。  可以查看有关[创建 Azure AD B2C 租户](tutorial-create-tenant.md)的说明。

### <a name="can-i-use-azure-ad-b2c-features-in-my-existing-employee-based-azure-ad-tenant"></a>我可以在基于员工的现有 Azure AD 租户中使用 Azure AD B2C 功能吗？
Azure AD 和 Azure AD B2C 是独立的产品/服务，不能在同一租户中共存。  Azure AD 租户表示组织。  Azure AD B2C 租户表示信赖方应用使用的标识集合。  通过自定义策略（在公共预览版中），Azure AD B2C 可以联合 Azure AD，允许对组织中的员工进行身份验证。

### <a name="can-i-use-azure-ad-b2c-to-provide-social-login-facebook-and-google-into-office-365"></a>我可以使用 Azure AD B2C 提供 Office 365 的社交登录（Facebook 和 Google+）吗？
Azure AD B2C 不用于 Microsoft Office 365 用户的身份验证。  Azure AD 是 Microsoft 针对管理员工对 SaaS 应用程序的访问权限的解决方案，它具有为此目的而设计的许可、条件访问等功能。  Azure AD B2C 提供用于生成 Web 和移动应用程序的标识和访问管理平台。  当 Azure AD B2C 配置为联合 Azure AD 租户时，Azure AD 租户将管理员工对依赖 Azure AD B2C 的应用程序的访问权限。

### <a name="what-are-local-accounts-in-azure-ad-b2c-how-are-they-different-from-work-or-school-accounts-in-azure-ad"></a>什么是 Azure AD B2C 中的本地帐户？ 它们与 Azure AD 中的工作或学校帐户有何不同？
在 Azure AD 租户中，属于租户的用户使用 `<xyz>@<tenant domain>` 形式的电子邮件地址登录。  `<tenant domain>` 是租户中已验证域之一或初始的 `<...>.onmicrosoft.com` 域。 此类型的帐户是工作或学校帐户。

在 Azure AD B2C 租户中，大多数应用都希望用户使用任意电子邮件地址（例如 joe@comcast.net、bob@gmail.com、sarah@contoso.com 或 jim@live.com）登录。 此类型的帐户是本地帐户。  我们还支持任意用户名作为本地帐户（例如，joe、bob、sarah 或 jim）。 在 Azure 门户中配置 Azure AD B2C 的标识提供者时，可以选择这两种本地帐户类型中的一种。 在 Azure AD B2C 租户中，单击“标识提供者”，然后选择“本地帐户”下的“用户名”。 

应用程序的用户帐户必须始终通过注册用户流、注册或登录用户流，或使用 Azure AD Graph API 创建。 在 Azure 门户中创建的用户帐户仅用于管理租户。

### <a name="which-social-identity-providers-do-you-support-now-which-ones-do-you-plan-to-support-in-the-future"></a>现在支持哪些社交标识提供者？ 计划在未来支持哪些？
我们当前支持 Facebook、Google+、LinkedIn、Amazon、Twitter（预览版）、微信（预览版）、微博（预览版）和 QQ（预览版）。 我们会根据客户需求添加对其他流行社交标志提供者的支持。

Azure AD B2C 还增加了对[自定义策略](active-directory-b2c-overview-custom.md)的支持。  这些[自定义策略](active-directory-b2c-overview-custom.md)允许开发人员使用支持 [OpenID 连接](https://openid.net/specs/openid-connect-core-1_0.html)或 SAML 的任何标识提供者创建自己的策略。 

查看我们的[自定义策略初学者包](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack)，开始使用自定义策略。

### <a name="can-i-configure-scopes-to-gather-more-information-about-consumers-from-various-social-identity-providers"></a>我可以配置范围，从各种社交标识提供者收集更多使用者的相关信息吗？
不。 一组受支持的社交标识提供者使用的默认范围是：

* Facebook：电子邮件
* Google+：电子邮件
* Microsoft 帐户：openid 电子邮件配置文件
* Amazon：配置文件
* LinkedIn：r_emailaddress、r_basicprofile

### <a name="does-my-application-have-to-be-run-on-azure-for-it-work-with-azure-ad-b2c"></a>必须在 Azure 上运行应用程序才能将其与 Azure AD B2C 一起使用吗？
不，可以在任何位置（在云中或本地）托管应用程序。 只要能在公共可访问的端点上发送和接收 HTTP 请求，它就可以与 Azure AD B2C 进行交互。

### <a name="i-have-multiple-azure-ad-b2c-tenants-how-can-i-manage-them-on-the-azure-portal"></a>我有多个 Azure AD B2C 租户。 如何在 Azure 门户上管理它们？
在 Azure 门户的左侧菜单中打开“Azure AD B2C”之前，必须切换到要管理的目录。  通过单击 Azure 门户右上方的标识切换目录，然后在出现的下拉列表中选择目录。

### <a name="how-do-i-customize-verification-emails-the-content-and-the-from-field-sent-by-azure-ad-b2c"></a>如何自定义 Azure AD B2C 发送的验证电子邮件（内容和“发件人:”字段）？
可以使用[公司品牌功能](../active-directory/fundamentals/customize-branding.md)来自定义验证电子邮件的内容。 具体来说，可以自定义电子邮件的下列两个元素：

* **横幅徽标**：显示在右下角。
* **背景色**：显示在顶部。

    ![自定义的验证电子邮件的屏幕截图](./media/active-directory-b2c-faqs/company-branded-verification-email.png)

电子邮件签名包含首次创建 Azure AD B2C 租户时提供的 Azure AD B2C 租户名称。 可以使用以下说明更改名称：

1. 以全局管理员身份登录到 [Azure 门户](https://portal.azure.com/)。
1. 打开“Azure Active Directory”边栏选项卡。
1. 单击“属性”选项卡。
1. 更改“名称”字段。
1. 单击页顶部的“保存”。

目前没有办法更改电子邮件中的“发件人:”字段。

### <a name="how-can-i-migrate-my-existing-user-names-passwords-and-profiles-from-my-database-to-azure-ad-b2c"></a>如何将我现有的用户名、密码和配置文件从数据库迁移到 Azure AD B2C？
可以使用 Azure AD 图形 API 编写迁移工具。 有关详细信息，请参阅[用户迁移指南](active-directory-b2c-user-migration.md)。

### <a name="what-password-user-flow-is-used-for-local-accounts-in-azure-ad-b2c"></a>Azure AD B2C 中的本地帐户使用什么密码用户流？
本地帐户的 Azure AD B2C 密码用户流以 Azure AD 的策略为基础。 Azure AD B2C 的注册、注册或登录和密码重置用户流使用“强”密码强度，并且不会让任何密码过期。 有关详细信息，请阅读 [Azure AD 密码策略](/previous-versions/azure/jj943764(v=azure.100))。 有关帐户锁定和密码的信息，请参阅[管理对 Azure Active Directory B2C 中资源和数据的威胁](active-directory-b2c-reference-threat-management.md)。

### <a name="can-i-use-azure-ad-connect-to-migrate-consumer-identities-that-are-stored-on-my-on-premises-active-directory-to-azure-ad-b2c"></a>我可以使用 Azure AD Connect 将存储在本地 Active Directory 中的使用者标识迁移到 Azure AD B2C 吗？
不可以，Azure AD Connect 不是为与 Azure AD B2C 一起使用而设计的。 请考虑使用 [Azure AD 图形 API](active-directory-b2c-devquickstarts-graph-dotnet.md) 进行用户迁移。  有关详细信息，请参阅[用户迁移指南](active-directory-b2c-user-migration.md)。

### <a name="can-my-app-open-up-azure-ad-b2c-pages-within-an-iframe"></a>我的应用是否可在 iFrame 中打开 Azure AD B2C 页？
不可以，出于安全的考虑，无法在 iFrame 中打开 Azure AD B2C 页。  我们的服务将与浏览器通信以禁止 iFrame。  由于点击劫持的风险，安全社区和 OAUTH2 规范一般建议不要使用 iFrame 进行标识体验。

### <a name="does-azure-ad-b2c-work-with-crm-systems-such-as-microsoft-dynamics"></a>Azure AD B2C 是否可以与 Microsoft Dynamics 之类的 CRM 系统一起使用？
与 Microsoft Dynamics 365 门户的集成可用。  请参阅[配置 Dynamics 365 门户以使用 Azure AD B2C 进行身份验证](https://docs.microsoft.com/dynamics365/customer-engagement/portals/azure-ad-b2c)。

### <a name="does-azure-ad-b2c-work-with-sharepoint-on-premises-2016-or-earlier"></a>Azure AD B2C 是否可以与 SharePoint 本地 2016 或更早版本一起使用？
Azure AD B2C 不适用于 SharePoint 外部合作伙伴共享的情况；请改为参阅 [Azure AD B2B](https://cloudblogs.microsoft.com/enterprisemobility/2015/09/15/learn-all-about-the-azure-ad-b2b-collaboration-preview/)。

### <a name="should-i-use-azure-ad-b2c-or-b2b-to-manage-external-identities"></a>我应该使用 Azure AD B2C 还是 B2B 来管理外部标识？
请阅读有关[外部标识](../active-directory/active-directory-b2b-compare-external-identities.md)的文章，了解有关将适当功能应用于外部标识方案的详细信息。

### <a name="what-reporting-and-auditing-features-does-azure-ad-b2c-provide-are-they-the-same-as-in-azure-ad-premium"></a>Azure AD B2C 提供哪些报告和审核功能？ 它们是否与 Azure AD Premium 中提供的功能相同？
否，Azure AD B2C 不支持与 Azure AD Premium 相同的报告集。 但是，有许多共性：

* 登录报告提供每次登录的记录以及简短的详细信息。
* 审核报告包括管理活动和应用程序活动。 
* 使用情况报告包括用户数、登录次数和 MFA 次数。 

### <a name="can-i-localize-the-ui-of-pages-served-by-azure-ad-b2c-what-languages-are-supported"></a>我可以本地化 Azure AD B2C 所提供页面的 UI 吗？ 支持哪些语言？
 能！  请阅读公共预览版中的[语言自定义](active-directory-b2c-reference-language-customization.md)。  我们提供 36 种语言的翻译版本，并且你可以根据需要替代任何字符串。

### <a name="can-i-use-my-own-urls-on-my-sign-up-and-sign-in-pages-that-are-served-by-azure-ad-b2c-for-instance-can-i-change-the-url-from-loginmicrosoftonlinecom-to-logincontosocom"></a>我可以在 Azure AD B2C 提供的注册和登录页面上使用自己的 URL 吗？ 例如，可以将 URL 从 login.microsoftonline.com 更改为 login.contoso.com 吗？
目前不可以。 该功能在我们的计划之中。 在 Azure 门户上的“域”选项卡中验证域并不能实现此目标。

### <a name="how-do-i-delete-my-azure-ad-b2c-tenant"></a>如何删除 Azure AD B2C 租户？
请按照以下步骤删除 Azure AD B2C 租户：

1. 删除所有用户流 （策略） 在 Azure AD B2C 租户中。
1. 删除 Azure AD B2C 租户中注册的所有应用程序。
1. 现在，以订阅管理员身份登录到 [Azure 门户](https://portal.azure.com/)。 （使用在注册 Azure 时使用的同一工作或学校帐户，或同一 Microsoft 帐户。）
1. 切换到要删除的 Azure AD B2C 租户。
2. 导航到左侧的 Active Directory 菜单。
3. 选择“用户和组”。
4. 依次选择每个用户（排除当前以订阅管理员身份登录的用户）。 单击页面底部的“删除”，并在出现提示时单击“是”。
5. 单击“应用注册”。
6. 选择名为“b2c-extensions-app”的应用程序。 单击“删除”，出现提示时，单击“是”。
7. 选择“概述”。
8. 单击“删除目录”。 要完成该过程，请按照屏幕上的说明进行操作。

### <a name="can-i-get-azure-ad-b2c-as-part-of-enterprise-mobility-suite"></a>我可以将 Azure AD B2C 作为企业移动性套件的一部分吗？
不，Azure AD B2C 是即用即付 Azure 服务，不是企业移动套件的一部分。

### <a name="how-do-i-report-issues-with-azure-ad-b2c"></a>如何报告 Azure AD B2C 存在的问题？
请参阅[提出针对 Azure Active Directory B2C 的支持请求](active-directory-b2c-support.md)。
