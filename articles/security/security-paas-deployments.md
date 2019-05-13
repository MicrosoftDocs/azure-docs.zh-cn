---
title: 安全 PaaS 部署的 Microsoft Azure 的最佳实践
description: 了解最佳实践设计、 构建和管理安全云在 Azure 上的应用程序和了解其他云服务模型与 PaaS 的安全优势。
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: techlake
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 9da7a3b91223b8a6fd25814a10a0cbafd645d132
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231117"
---
# <a name="securing-paas-deployments"></a>保护 PaaS 部署

参考本文中的信息，可以：

- 了解云中的托管应用程序的安全优势
- 评估平台即服务 (PaaS) 相比其他云服务模型的安全优势
- 将安全重心从以网络为中心的方案转换为以标识为中心的外围安全方案
- 实施一般的 PaaS 安全最佳实践建议

[开发安全应用程序在 Azure 上的](abstract-develop-secure-apps.md)是安全问题和控件时，应考虑在软件开发生命周期的每个阶段开发的云应用程序的常规指南。

## <a name="cloud-security-advantages"></a>云的安全优势
转移到云中可带来一定的安全优势。 在本地环境中，组织的可用资源可能有限，无法尽责在安全措施上投资，使得攻击者能够利用所有层中的漏洞。

![云时代的安全优势][1]

组织可以使用提供商的基于云的安全功能和云智能来改善其威胁检测和响应时间。  通过将责任转移到云提供商，组织可以扩大安全覆盖范围，为其他优先业务重新调配安全资源与预算。

## <a name="division-of-responsibility"></a>责任划分
请务必了解你与 Microsoft 之间的责任分工。 在本地，拥有整个堆栈，但迁移到云后，某些责任将转移到 Microsoft。 下面的责任矩阵展示了在 SaaS、PaaS 和 IaaS 部署中，分别由你和 Microsoft 负责的堆栈区域。

![责任区域][2]

对于所有云部署类型，拥有数据和标识。 需要负责保护由你控制的数据和标识、本地资源及云组件的安全（保护的项目因服务类型而异）。

不管部署类型为何，都必须负责的项目包括：

- 数据
- 终结点
- 帐户
- 访问管理

## <a name="security-advantages-of-a-paas-cloud-service-model"></a>PaaS 云服务模型的安全优势
让我们使用上面这个责任矩阵来了解一下 Azure PaaS 部署相比本地部署的安全优势。

![PaaS 的安全优势][3]

从堆栈的底部（即物理基础结构）开始，Microsoft 可以消除常见的风险和管理责任。 由于 Microsoft 云受到 Microsoft 的持续监视，因此很难攻破。 攻击者将 Microsoft 云当作攻击目标是不会得逞的。 他们往往会改换目标，除非他们有大量的金钱和资源。  

在堆栈的中间，PaaS 部署与本地之间没有差别。 在应用程序层和帐户与访问管理层，面临的风险是类似的。 本文的后续步骤部分将提供有关消除或尽量避免这些风险的最佳实践指导。

堆栈的顶层（即数据监管和权限管理）存在一种风险，不过可以使用密钥管理来缓解。 （最佳实践中介绍了密钥管理。）尽管密钥管理是一个附加的责任，但你不再需要管理 PaaS 部署中的某些区域，因此可将资源转移到密钥管理。

Azure 平台还使用各种基于网络的技术提供强大的 DDoS 保护。 但是，根据链路和数据中心的不同，所有类型的基于网络的 DDoS 保护方法都有自身的限制。 为了帮助避免大规模 DDoS 攻击造成的影响，可以利用 Azure 的核心云功能快速自动扩展，以防御 DDoS 攻击。 在建议的实践文章中，我们将更详细地介绍如何采取这种措施。

## <a name="modernizing-the-defenders-mindset"></a>革新防御者的思维方式
PaaS 部署为整体安全方案带来了变革。 事必躬亲的局面现在可以改为与 Microsoft 分担责任。

PaaS 与传统本地部署之间的另一个重大差别在于，前者为主要安全边界的界定因素提供了全新的视野。 一直以来，主要的本地安全边界就是网络，大多数本地安全设计都使用网络作为主要安全枢纽。 在 PaaS 部署中，可将标识视为主要安全边界，从而改善安全性。

## <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>采用标识用作主要安全边界的策略
在云计算的五大基本特征中，一个特征就是网络访问范围广泛，这使得以网络为中心的理念显得有点毫不相干。 许多云计算解决方案的目标是不管用户身居何处，都能允许他们访问资源。 对于大多数用户而言，他们的位置就是 Internet 上的某个节点。

下图演示了安全边界从网络边界演进成标识边界的过程。 安全性越来越少地与如何保护网络相关，而更多地与如何保护数据，以及如何管理应用和用户的安全性相关。 两者的关键差别在于如何为公司的重要资产提供更多的安全保障。

![标识用作新的安全边界][4]

最初，Azure PaaS 服务（例如 Web 角色和 Azure SQL）提供的传统网络边界防护措施很少，或者根本不提供。 开发人员已认识到，设计元素的目的就是在 Internet 上公开（Web 角色），而身份验证可提供新的边界（例如 BLOB 或 Azure SQL）。

新式安全措施假设入侵者会突破网络边界。 因此，新式防护措施已转移到标识。 组织必须使用强身份验证和授权保护机制建立基于标识的安全边界（最佳实践）。

网络边界的原理和模式早在几十年前就已建立。 相比之下，行业在使用标识作为主要安全边界的经验相对缺乏。 正因如此，我们累积了足够的经验，乐于提供已在现场得到证实的、适用于几乎所有 PaaS 服务的一些普通建议。

下面是管理标识边界的最佳做法。

**最佳做法**：保护密钥和凭据以保护 PaaS 部署。   
**详细信息**：丢失密钥和凭据是一个常见问题。 可以使用集中式的解决方案，其中在硬件安全模块 (Hsm) 中存储的密钥和机密。 [Azure 密钥保管库](../key-vault/key-vault-whatis.md)通过加密身份验证密钥、 存储帐户密钥、 数据加密密钥、.pfx 文件和使用受 Hsm 保护的密钥的密码来保护你的密钥和机密。

**最佳做法**：不要将凭据和其他机密放入源代码或 GitHub。   
**详细信息**：唯一比丢失密钥和凭据更遭糕的事情是让未经授权的一方获取这些密钥和凭据的访问权限。 攻击者可以利用 bot 技术来查找 GitHub 等代码存储库中存储的密钥和机密。 请不要将密钥和机密放入这些公共代码存储库。

**最佳做法**：通过使用可以直接远程管理这些 VM 的管理接口来保护混合 PaaS 和 IaaS 服务上的 VM 管理接口。   
**详细信息**：可以使用 [SSH](https://en.wikipedia.org/wiki/Secure_Shell)、[RDP](https://support.microsoft.com/kb/186607) 和 [PowerShell 远程处理](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.core/enable-psremoting)等远程管理协议。 通常，我们建议不要从 Internet 启用对 VM 的直接远程访问。

如果可以，请使用替代方法，例如在 Azure 虚拟网络中使用虚拟专用网络。 如果无法使用替代方法，请确保使用复杂的通行短语，并使用双重身份验证（例如 [Azure 多重身份验证](../active-directory/authentication/multi-factor-authentication.md)）。

**最佳做法**：使用强身份验证和授权平台。   
**详细信息**：在 Azure AD 而不是自定义用户存储中使用联合标识。 使用联合标识时，可以利用基于平台的方法，将已获授权的标识的管理权限委托给合作伙伴。 如果员工离职后，需要通过多个标识和授权系统反映该信息，则联合标识方法就特别重要。

使用平台提供的身份验证和授权机制，而不要使用自定义代码。 原因是开发自定义身份验证代码可能很容易出错。 大部分开发人员都不是安全专家，不太可能会注意到身份验证和授权的细微之处与最新开发情况。 商业代码（例如 Microsoft 编写的代码）通常会接受广泛的安全性评审。

使用双重身份验证。 双重身份验证是最新的身份验证和授权标准，它避免了用户名与密码类型的身份验证所固有的安全漏洞。 需要访问 Azure 管理界面（门户/远程 PowerShell）和面向客户的服务的应用程序应设计并配置为使用 [Azure 多重身份验证](../active-directory/authentication/multi-factor-authentication.md)。

使用 OAuth2 和 Kerberos 等标准身份验证协议。 这些协议经过广泛的同行评审，有时可实现为平台库的一部分用于身份验证和授权。

## <a name="use-threat-modeling-during-application-design"></a>在应用程序设计期间使用威胁建模
Microsoft [安全开发生命周期](https://www.microsoft.com/en-us/sdl)指定团队应在设计阶段参与名为威胁建模的过程。 为了帮助简化此过程，Microsoft 已创建 [SDL 威胁建模工具](../security/azure-security-threat-modeling-tool.md)。 对应用程序设计进行建模以及跨所有信任边界枚举 [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) 威胁可以在早期捕获设计错误。

下表列出了 STRIDE 威胁，并提供了一些使用 Azure 功能的示例缓解措施。 这些缓解措施并非在每种情况下都起作用。

| 威胁 | 安全属性 | 潜在的 Azure 平台迁移 |
| --- | --- | --- |
| 欺骗 | Authentication | 需要 HTTPS 连接。 |
| 篡改 | 完整性 | 验证 SSL 证书。 |
| 否认性 | 不可否认性 | 启用 Azure [监视和诊断](https://docs.microsoft.com/azure/architecture/best-practices/monitoring)。 |
| 信息泄露 | 机密性 | 使用[服务证书](https://docs.microsoft.com/rest/api/appservice/certificates)加密静态敏感数据。 |
| 拒绝服务 | 可用性 | 监视潜在拒绝服务条件的性能指标。 实现连接筛选器。 |
| 权限提升 | 授权 | 使用[特权标识管理](../active-directory/privileged-identity-management/subscription-requirements.md)。 |

## <a name="develop-on-azure-app-service"></a>在 Azure 应用服务上开发
[Azure App Service](../app-service/overview.md) 是一个 PaaS 产品，可创建适用于任何平台或设备的 Web 和移动应用，并可连接到云中或本地任何位置的数据。 应用服务所包括的 Web 功能和移动功能是以前作为 Azure 网站和 Azure 移动服务单独交付的。 它还包括各种新功能，可以实现业务流程的自动化，并可托管云 API。 应用服务以单个集成服务的形式为 Web、移动和集成方案提供一组丰富的功能。

下面是使用应用服务的最佳做法。

**最佳做法**：[通过 Azure Active Directory 进行身份验证](../app-service/overview-authentication-authorization.md)。   
**详细信息**：应用服务为标识提供者提供 OAuth 2.0 服务。 OAuth 2.0 注重简化客户端开发人员的工作，同时为 Web 应用程序、桌面应用程序和移动电话提供特定的授权流。 Azure AD 使用 OAuth 2.0，可让你授予移动和 Web 应用程序的访问权限。

**最佳做法**：根据“需要知道”和“最低权限”安全原则限制访问。   
**详细信息**：对于想要实施数据访问安全策略的组织，限制访问是必须要做的事。 可以使用 RBAC 向特定范围的用户、组和应用程序分配权限。 若要了解有关向用户授予应用程序访问权限的详细信息，请参阅[访问管理入门](../role-based-access-control/overview.md)。

**最佳做法**：保护密钥。   
**详细信息**：Azure Key Vault 可帮助保护云应用程序和服务使用的加密密钥和机密。 通过 Key Vault，可以使用受硬件安全模块 (HSM) 保护的密钥，来加密密钥和机密（例如身份验证密钥、存储帐户密钥、数据加密密钥、.PFX 文件和密码）。 为了提升可靠性，可以在 HSM 中导入或生成密钥。 请参阅 [Azure Key Vault](../key-vault/key-vault-whatis.md) 了解详细信息。 还可以使用 Key Vault 和自动续订来管理 TLS 证书。

**最佳做法**：限制传入的源 IP 地址。   
**详细信息**：[应用服务环境](../app-service/environment/intro.md)提供虚拟网络集成功能，可帮助你通过网络安全组限制传入的源 IP 地址。 使用虚拟网络可将 Azure 资源置于可以控制其访问权限但无法通过 Internet 路由的网络中。 若要了解详细信息，请参阅[将应用与 Azure 虚拟网络集成](../app-service/web-sites-integrate-with-vnet.md)。

**最佳做法**：监视应用服务环境的安全状态。   
**详细信息**：使用 Azure 安全中心监视应用服务环境。 在安全中心识别潜在的安全漏洞时，它会创建一些[建议](../security-center/security-center-virtual-machine-recommendations.md)，这些建议会指导完成配置所需控件的过程。

> [!NOTE]
> 监视应用服务的功能以预览版提供，仅适用于安全中心的[标准层](../security-center/security-center-pricing.md)。
>
>

## <a name="install-a-web-application-firewall"></a>安装 Web 应用程序防火墙
Web 应用程序已逐渐成为利用常见已知漏洞的恶意攻击的目标。 这些攻击中最常见的攻击包括 SQL 注入攻击、跨站点脚本攻击等。 防止应用程序代码中的此类攻击颇具挑战性，可能需要在应用程序拓扑的多个层进行严格的维护、修补和监视。 集中式 Web 应用程序防火墙有助于大幅简化安全管理，为抵卸威胁或入侵的应用程序管理员提供更好的保障。 相较保护每个单独的 Web 应用程序，WAF 解决方案还可通过在中央位置修补已知漏洞，更快地响应安全威胁。 可将现有应用程序网关轻松转换为支持 Web 应用程序防火墙的应用程序网关。

[Web 应用程序防火墙 (WAF)](../application-gateway/waf-overview.md) 是应用程序网关的功能，可以对 Web 应用程序进行集中保护，避免其受到常见的攻击和漏洞危害。 WAF 基于 [开放 Web 应用程序安全项目 (OWASP) 核心规则集](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 或 2.2.9 中的规则。

## <a name="monitor-the-performance-of-your-applications"></a>监视应用程序的性能
监视是一种数据收集和分析操作，用于确定应用程序的性能、运行状况及可用性。 有效的监视策略有助于了解应用程序组件的详细运行状况， 它有助于向你发送关键情况的通知，让你在这些情况成为问题之前解决它们，从而提高运行时间。 它还有助于检测可能与安全相关的异常。

使用 [Azure Application Insights](https://azure.microsoft.com/documentation/services/application-insights) 监视应用程序的可用性、性能和使用情况，不管其是托管在云中还是在本地。 通过使用 Application Insights，可以快速确定并诊断应用程序中的错误，而无需等待用户报告这些错误。 利用所收集的信息，可作出有关应用程序维护和优化的明智抉择。

Application Insights 提供各种可以与所收集的数据交互的工具。 Application Insights 在公用存储库中存储其数据。 它可以充分利用各种共享功能，如警报、 仪表板，并使用 Kusto 查询语言进行深度分析。

## <a name="perform-security-penetration-testing"></a>执行安全渗透测试
验证安全防御措施是与测试其他功能一样重要。 请[渗透测试](azure-security-pen-testing.md)生成和部署过程的标准组成部分。 计划定期安全测试和漏洞扫描已部署应用程序，并监视打开的端口、 终结点和攻击。

模糊测试是一种方法用于通过提供程序接口 （入口点） 的分析和使用此数据的格式不正确输入的数据中查找程序故障 （错误代码）。 [Microsoft 安全风险检测](https://www.microsoft.com/en-us/security-risk-detection/)是一个基于云的工具，可用于部署到 Azure 之前查找 bug 和您的软件中的其他安全漏洞。 该工具用于在部署软件，因此无需修补 bug、 处理崩溃，或后发布该软件的攻击做出响应之前捕获的漏洞。


## <a name="next-steps"></a>后续步骤
本文重点介绍了 Azure PaaS 部署的安全优势以及云应用程序的最佳安全做法。 接下来，请阅读有关使用特定 Azure 服务保护 PaaS Web 和移动解决方案的建议做法。 首先，我们介绍如何保护 Azure 应用服务、Azure SQL 数据库和 Azure SQL 数据仓库，以及 Azure 存储。 随着适用于其他 Azure 服务的建议做法文章的发布，我们会在以下列表中提供相应的链接：

- [Azure 应用服务](security-paas-applications-using-app-services.md)
- [Azure SQL 数据库和 Azure SQL 数据仓库](security-paas-applications-using-sql.md)
- [Azure 存储](security-paas-applications-using-storage.md)
- 用于 Redis 的 Azure 缓存
- Azure 服务总线
- Web 应用程序防火墙

请参阅[开发安全应用程序在 Azure 上的](abstract-develop-secure-apps.md)安全问题和控件时，应考虑在软件开发生命周期的每个阶段开发的云应用程序。

有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com

<!--Image references-->
[1]: ./media/security-paas-deployments/advantages-of-cloud.png
[2]: ./media/security-paas-deployments/responsibility-zones.png
[3]: ./media/security-paas-deployments/advantages-of-paas.png
[4]: ./media/security-paas-deployments/identity-perimeter.png
