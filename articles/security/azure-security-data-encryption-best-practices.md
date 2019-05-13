---
title: 数据安全与加密最佳实践-Microsoft Azure
description: 本文提供一系列有关使用内置 Azure 功能实现数据安全与加密的最佳实践。
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 17ba67ad-e5cd-4a8f-b435-5218df753ca4
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 9955450b468ef38ba456d7ee73d9681de677494d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190713"
---
# <a name="azure-data-security-and-encryption-best-practices"></a>Azure 数据安全与加密最佳实践
本文介绍数据安全与加密最佳的实践。

最佳做法以观点的共识以及 Azure 平台功能和特性集为基础。 随时间变化的观点和技术和定期以反映这些更改更新这篇文章。

## <a name="protect-data"></a>保护数据
为了帮助保护云中的数据，需要考虑数据可能出现的状态以及可用于该状态的控件。 Azure 数据安全与加密的最佳做法与以下数据状态相关：

- 静态：包括物理媒体（磁盘或光盘）上以静态方式存在的所有信息存储对象、容器和类型。
- 传输中：在各组件、位置或程序间传输数据时，数据处于“传输中”状态。 例如通过网络、通过服务总线（从本地到云，反之亦然，包括诸如 ExpressRoute 的混合连接）进行传输，或在输入/输出过程。

## <a name="choose-a-key-management-solution"></a>选择密钥管理解决方案

保护密钥对保护云中的数据至关重要。

[Azure Key Vault](../key-vault/key-vault-overview.md) 可帮助保护云应用程序和服务使用的加密密钥和机密。 Key Vault 简化了密钥管理过程，可让我们控制用于访问和加密数据的密钥。 开发人员可以在几分钟内创建用于开发和测试的密钥，然后将其迁移到生产密钥。 安全管理员可以根据需要授予（和吊销）密钥权限。

可以使用 Key Vault 创建多个安全容器（称为保管库）。 这些保管库受 HSM 支持。 保管库可以集中存储应用程序机密，降低安全信息意外丢失的可能性。 Key vault 还控制并记录外界对其所存储内容的访问。 Azure Key Vault 负责处理传输层安全性 (TLS) 证书的请求和续订事宜。 它为可靠的证书生命周期管理解决方案提供相关功能。

Azure Key Vault 旨在支持应用程序密钥和机密。 Key Vault 不应用于存储用户密码。

以下是使用 Key Vaul 的安全最佳做法。

**最佳做法**：向特定范围内的用户、组和应用程序授予访问权限。   
**详细信息**：使用 RBAC 的预定义角色。 例如，要向用户授予管理密钥保管库的访问权限，需要将预定义的角色[密钥保管库参与者](../role-based-access-control/built-in-roles.md)分配给位于特定范围内的此用户。 在此情况下，该范围可以是订阅、资源组，或只是特定的密钥保管库。 如果预定义角色不符合需求，可以[定义自己的角色](../role-based-access-control/custom-roles.md)。

**最佳做法**：控制用户有权访问的内容。   
**详细信息**：可通过以下两个独立接口来控制对密钥保管库的访问：管理平面和数据平面。 管理平面和数据平面访问独立控制工作。

使用 RBAC 控制用户有权访问的内容。 例如，如果想要授予应用程序使用密钥保管库中的密钥的访问权限，只需使用密钥保管库访问策略授予数据平面访问权限，而无需授予此应用程序的管理平面访问权限。 相反，如果希望用户能够读取保管库属性和标记，但不让其具有任何访问密钥、机密或证书的权限，则可以使用 RBAC 向此用户授予“读取”访问权限，而无需授予数据平面访问权限。

**最佳做法**：将证书存储在密钥保管库中。 证书的价值很高。 如果落入他人之手，应用程序或数据的安全性可能会受到损害。   
**详细信息**：Azure 资源管理器可以在部署 VM 时，将存储在 Azure Key Vault 中的证书安全地部署到 Azure VM。 通过为密钥保管库设置适当的访问策略，还可以控制有权访问证书的人员。 另一个好处是，可以在 Azure Key Vault 中的一个位置管理所有证书。 有关详细信息，请参阅[将证书从客户托管的 Key Vault 部署到 VM](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/)。

**最佳做法**：确保可以恢复删除的密钥保管库或密钥保管库对象。   
**详细信息**：删除密钥保管库或密钥保管库对象可以是无意或恶意的。 启用 Key Vault 的软删除和清除保护功能，尤其是对用于加密静态数据的密钥。 删除这些密钥相当于丢失数据，因此可以在需要时恢复已删除的保管库和保管库对象。 定期练习 Key Vault 恢复操作。

> [!NOTE]
> 如果用户具有密钥保管库管理平面的参与者权限 (RBAC)，则该用户可以通过设置密钥保管库访问策略来授予自己对数据平面的访问权限。 建议严格控制具有密钥保管库“参与者”权限的人员，以确保只有获得授权的人员可以访问和管理密钥保管库、密钥、机密和证书。
>
>

## <a name="manage-with-secure-workstations"></a>使用安全工作站进行管理

> [!NOTE]
> 订阅管理员或所有者应使用安全访问工作站或特权访问工作站。
>
>

因为绝大多数的攻击以最终用户为目标，所以终结点将成为主要攻击点之一。 入侵终结点的攻击者可以使用用户的凭据来访问组织的数据。 大多数终结点攻击都利用了用户是其本地工作站的管理员这一事实。

**最佳做法**：使用安全管理工作站来保护敏感帐户、任务和数据。   
**详细信息**：使用[特权访问工作站](https://technet.microsoft.com/library/mt634654.aspx)来减小工作站的受攻击面。 这些安全管理工作站可帮助减轻其中一些攻击，以确保数据更为安全。

**最佳做法**：确保实施终结点保护。   
**详细信息**：在用于使用数据的所有设备上强制实施安全策略（无论数据位于云中还是本地）。

## <a name="protect-data-at-rest"></a>保护静止的数据

[静态数据加密](https://cloudblogs.microsoft.com/microsoftsecure/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/)是实现数据隐私性、符合性和数据主权的必要措施。

**最佳做法**：使用磁盘加密来帮助保护数据。   
**详细信息**：使用 [Azure 磁盘加密](azure-security-disk-encryption.md)。 它使 IT 管理员能够加密 Windows 和 Linux IaaS VM 磁盘。 磁盘加密利用符合行业标准的 Windows BitLocker 功能和 Linux dm-crypt 功能为 OS 和数据磁盘提供卷加密。

Azure 存储和 Azure SQL 数据库默认对静态数据进行加密，并且许多服务都将加密作为选项提供。 可以使用 Azure Key Vault 来持续控制用于访问和加密数据的密钥。 有关详细信息，请参阅 [Azure 资源提供程序加密模型支持](azure-security-encryption-atrest.md#azure-resource-providers-encryption-model-support)。

**最佳做法**：使用加密来帮助降低与未经授权访问数据相关的风险。   
**详细信息**：在将敏感数据写入驱动器之前先将驱动器加密。

未实施数据加密的组织面临的数据保密性问题风险更大。 例如，未经授权的用户或恶意用户可能会窃取已入侵帐户中的数据，或者未经授权访问以明文格式编码的数据。 公司还必须证明，为了遵守行业法规，他们在不断作出相应努力并使用正确的安全控件来增强其数据安全性。

## <a name="protect-data-in-transit"></a>保护传输中的数据

保护传输中的数据应该是数据保护策略中不可或缺的部分。 由于数据将从许多位置来回移动，因此我们一般建议始终使用 SSL/TLS 协议来交换不同位置的数据。 在某些情况下，可能需要使用 VPN 隔离本地与云基础结构之间的整个信道。

对于在本地基础结构与 Azure 之间移动的数据，请考虑适当的防护措施，例如 HTTPS 或 VPN。 通过公共 Internet 在 Azure 虚拟网络和本地位置之间发送加密流量时，请使用 [Azure VPN 网关](https://docs.microsoft.com/azure/vpn-gateway/)。

以下是特定于使用 Azure VPN 网关、SSL/TLS 和 HTTPS 的最佳做法。

**最佳做法**：从位于本地的多个工作站安全访问 Azure 虚拟网络。   
**详细信息**：使用[站点到站点 VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。

**最佳做法**：从位于本地的单个工作站安全访问 Azure 虚拟网络。   
**详细信息**：使用[点到站点 VPN](../vpn-gateway/vpn-gateway-point-to-site-create.md)。

**最佳做法**：通过专用高速 WAN 链路移动大型数据集。   
**详细信息**：使用 [ExpressRoute](../expressroute/expressroute-introduction.md)。 如果选择使用 ExpressRoute，则还可以使用 [SSL/TLS](https://support.microsoft.com/kb/257591) 或其他协议在应用程序级别加密数据，以提供额外的保护。

**最佳做法**：通过 Azure 门户与 Azure 存储进行交互。   
**详细信息**：所有事务都通过 HTTPS 进行。 此外可以使用[存储 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)通过 HTTPS 与进行交互[Azure 存储](https://azure.microsoft.com/services/storage/)。

无法保护传输中数据的组织更容易遭受[中间人攻击](https://technet.microsoft.com/library/gg195821.aspx)、[窃听](https://technet.microsoft.com/library/gg195641.aspx)和会话劫持。 这些攻击可能是获取机密数据访问权限的第一步。

## <a name="secure-email-documents-and-sensitive-data"></a>保护电子邮件、文档和敏感数据

你希望控制并帮助保护在公司外部共享的电子邮件、文档和敏感数据。 [Azure 信息保护](https://docs.microsoft.com/azure/information-protection/)是基于云的解决方案，可帮助组织对其文档和电子邮件进行分类、标记和保护。 这可以由定义了规则和条件的管理员自动执行、由用户手动执行，或者以组合方式执行，在组合方式中，用户可获得建议。

分类始终是可标识的，而无论数据的存储位置或数据的共享人员。 标签包括视觉标记，如页眉、页脚或水印。 元数据以明文形式添加到文件和电子邮件标题中。 明文形式确保其他服务（如防止数据丢失的解决方案）可以识别分类并采取相应的操作。

保护技术使用 Azure Rights Management (Azure RMS)。 此技术与其他 Microsoft 云服务和应用程序（如 Office 365 和 Azure Active Directory）集成。 此保护技术使用加密、标识和授权策略。 通过 Azure RMS 应用的保护与文档和电子邮件保留在一起，不受位置影响，也无论是在组织、网络、文件服务器和应用程序内部还是外部。

此信息保护解决方案可用于控制数据，即使是与他人共享的数据，也可控制。 还可以将 Azure RMS 用于自己的业务线应用程序和软件供应商提供的信息保护解决方案，而无论这些应用程序和解决方案是在本地还是在云中。

建议：

- 为组织[部署 Azure 信息保护](https://docs.microsoft.com/azure/information-protection/deployment-roadmap)。
- 应用可反映业务需求的标签。 例如：将名为“高度机密”的标签应用于包含绝密数据的所有文档和电子邮件，以对这些数据进行分类和保护。 然后，只有授权的用户才能访问此数据，并具有指定的任何限制。
- 配置 [Azure RMS 的使用情况日志记录](https://docs.microsoft.com/azure/information-protection/log-analyze-usage)，以便监视组织使用保护服务的方式。

[数据分类](https://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf)和文件保护能力不佳的组织可能更容易遭到数据泄漏或数据滥用。 使用适当的文件保护，可以分析数据流，以深入了解业务、检测风险行为并采取纠正措施、跟踪对文档的访问等等。

## <a name="next-steps"></a>后续步骤

有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com
