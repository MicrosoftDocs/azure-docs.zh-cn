---
title: 数据库安全最佳实践-Microsoft Azure
description: 本文提供有关 Azure 数据库安全性的一套最佳做法。
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: b4744201f506730303e190783acc60bfaa383720
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409808"
---
# <a name="azure-database-security-best-practices"></a>Azure 数据库安全性最佳做法
本指南介绍了数据库安全性的最佳做法。

最佳做法以观点的共识以及 Azure 平台功能和特性集为基础。 随时间变化的观点和技术和定期以反映这些更改更新这篇文章。

## <a name="secure-databases"></a>确保数据库安全
安全性是管理数据库时的首要考虑因素，并且始终是 [Azure SQL 数据库](https://docs.microsoft.com/azure/sql-database/)的优先事务。 严格保护数据库有助于满足大部分法规或安全要求，包括 HIPAA、ISO 27001/27002 和 PCI DSS Level 1。 [Microsoft 信任中心站点](https://azure.microsoft.com/support/trust-center/services/)上提供了安全合规认证的最新列表。 也可以根据法规要求将数据库放置在特定的 Azure 数据中心。

## <a name="use-firewall-rules-to-restrict-database-access"></a>使用防火墙规则来限制数据库访问
Microsoft Azure SQL 数据库为 Azure 和其他基于 Internet 的应用程序提供关系数据库服务。 为确保访问安全性，SQL 数据库通过以下方式来控制访问：

- 采用通过 IP 地址限制连接的防火墙规则。
- 采用身份验证机制，要求用户表明其身份。
- 采用授权机制，限制用户执行特定操作，访问特定数据。

在指定哪些计算机具有访问权限之前，防火墙将禁止所有对数据库服务器的访问。 防火墙基于每个请求的起始 IP 地址授予数据库访问权限。

下图显示了在 SQL 数据库中设置服务器防火墙的位置：

![防火墙规则](./media/azure-database-security-best-practices/azure-database-security-best-practices-Fig1.png)

只能通过 TCP 端口 1433 使用 Azure SQL 数据库服务。 若要从计算机访问 SQL 数据库，请确保客户端计算机防火墙允许 TCP 端口 1433 上的传出 TCP 通信。 如果不需要将 TCP 端口 1433 上的入站连接用于其他应用，请使用防火墙规则阻止这些连接。

在连接过程中，来自 Azure 虚拟机的连接将重定向到每个辅助角色特有的 IP 地址和端口。 该端口号在 11000 到 11999 的范围内。 有关 TCP 端口的详细信息，请参阅[用于 ADO.NET 4.5 的非 1433 端口](../sql-database/sql-database-develop-direct-route-ports-adonet-v12.md)。

有关 SQL 数据库中的防火墙规则的详细信息，请参阅 [SQL 数据库防火墙规则](../sql-database/sql-database-firewall-configure.md)。

> [!Note]
> 除了 IP 规则外，防火墙还管理虚拟网络规则。 虚拟网络规则基于虚拟网络服务终结点。 在某些情况下，虚拟网络规则可能比 IP 规则更好。 若要了解详细信息，请参阅[虚拟网络服务终结点和 Azure SQL 数据库规则](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md)。

## <a name="enable-database-authentication"></a>启用数据库身份验证
SQL 数据库支持两种身份验证：SQL Server 身份验证和 Azure AD 身份验证。

### <a name="sql-server-authentication"></a>*SQL Server 身份验证*

包括如下优点：

- 这种身份验证可让 SQL 数据库支持包含混合操作系统的环境（其中的所有用户不是通过 Windows 域进行身份验证）。
- 可让 SQL 数据库支持旧式应用程序，以及合作伙伴提供的、需要 SQL Server 身份验证的应用程序。
- 可让用户从未知或不受信任的域进行连接。 例如，在某个应用程序中，指定的客户可以使用分配的 SQL Server 登录名建立连接，以接收其订单的状态。
- 可让 SQL 数据库支持基于 Web 的应用程序（其中的用户可创建自己的标识）。
- 可让软件开发人员基于已知的预设 SQL Server 登录名使用复杂的权限层次结构来分发应用程序。

> [!NOTE]
> SQL Server 身份验证无法使用 Kerberos 安全协议。

如果使用 SQL Server 身份验证，则必须：

- 自行管理强凭据。
- 保护连接字符串中的凭据。
- （可能需要）保护通过网络从 Web 服务器传递到数据库的凭据。 有关更多信息，请参阅[如何：在 ASP.NET 2.0 中使用 SQL 身份验证连接到 SQL Server](/previous-versions/msp-n-p/ff648340(v=pandp.10))。

### <a name="azure-active-directory-ad-authentication"></a>*Azure Active Directory (AD) 身份验证*
Azure AD 身份验证是使用 Azure AD 中的标识连接到 Azure SQL 数据库和 [SQL 数据仓库](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)的一种机制。 通过 Azure AD 身份验证，可以在一个中心位置中管理数据库用户和其他 Microsoft 服务的标识。 集中 ID 管理提供一个单一位置来管理数据库用户，并简化权限管理。

> [!NOTE]
> 我们建议使用 Azure AD 身份验证，而不使用 SQL Server 身份验证。

包括如下优点：

- 提供一个 SQL Server 身份验证的替代方法。
- 它帮助阻止用户标识在数据库服务器之间激增。
- 它允许在单一位置轮换密码。
- 客户可以使用外部 (Azure AD) 组管理数据库权限。
- 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- 它使用包含的数据库用户在数据库级别对标识进行身份验证。
- 它支持对连接到 SQL 数据库的应用程序进行基于令牌的身份验证。
- 它支持对本地 Azure Active Directory 实例进行 AD FS（域联合）或本机用户/密码身份验证，无需进行域同步。
- Azure AD 支持从 SQL Server Management Studio 进行连接，后者使用 Active Directory 通用身份验证，其中包括多重身份验证。 多重身份验证提供利用一系列验证选项进行的强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知。 有关详细信息，请参阅 [SQL 数据库和 SQL 数据仓库针对 Azure AD 多重身份验证的 SSMS 支持](../sql-database/sql-database-ssms-mfa-authentication.md)。

配置步骤包括配置和使用 Azure AD 身份验证的以下过程：

- 创建并填充 Azure AD。
- 可选：关联或更改当前与 Azure 订阅关联的 Active Directory 实例。
- 为 Azure SQL 数据库或 [Azure SQL 数据仓库](https://azure.microsoft.com/services/sql-data-warehouse/)创建 Azure Active Directory 管理员。
- 配置客户端计算机。
- 在映射到 Azure AD 标识的数据库中创建包含的数据库用户。
- 通过使用 Azure AD 标识连接到数据库。

有关详细信息，请参阅[将 Azure Active Directory 身份验证与 SQL 数据库、托管实例或 SQL 数据仓库结合使用](../sql-database/sql-database-aad-authentication.md)。

## <a name="protect-your-data-by-using-encryption-and-row-level-security"></a>使用加密和行级别安全性来保护数据
[Azure SQL 数据库透明数据加密](../sql-database/transparent-data-encryption-azure-sql.md)可帮助保护磁盘上的数据，并防止对硬件进行未经授权的访问。 它可执行静态数据库、关联备份和事务日志文件的实时加密和解密，无需更改应用程序。 透明数据加密使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。

即使整个存储都已加密，也一定要加密数据库本身。 这是数据保护的深度防御方法实现。 如果使用 Azure SQL 数据库，并想要保护敏感数据（例如信用卡号或社会安全号码），可以使用 FIPS 140-2 验证的 256 位 AES 加密来加密数据库。 此加密符合许多行业标准（例如 HIPAA 和 PCI）的要求。

使用透明数据加密来加密数据库时，未加密与[缓冲池扩展 (BPE)](https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension) 相关的文件。 必须对 BPE 相关文件使用文件系统级别的加密工具（例如 [BitLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc732774(v=ws.11))）或[加密文件系统 (EFS)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc749610(v%3dws.10))。

因为经过授权的用户（如安全管理员或数据库管理员）可以访问数据，所以即使已使用透明数据加密将数据库加密，也应遵循以下建议：

- 启用数据库级别的 SQL Server 身份验证。
- 通过 [RBAC 角色](../role-based-access-control/overview.md)使用 Azure AD 身份验证。
- 确保用户和应用程序使用不同的帐户进行身份验证。 这样，可以限制授予用户和应用程序的权限，并降低恶意活动的风险。
- 使用固定数据库角色（如 db_datareader 或 db_datawriter）实现数据库级别安全性。 也可以为你的应用程序创建自定义角色以向选定数据库对象授予显式权限。

若要通过其他方法保护数据，请考虑：

- 使用[单元格级加密](/sql/relational-databases/security/encryption/encrypt-a-column-of-data)，借助不同的加密密钥来加密特定的数据列甚至数据单元格。
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)，可让客户在客户端应用程序中加密敏感数据，永远不会向数据库引擎（SQL 数据库或 SQL Server）透露加密密钥。 因此，Always Encrypted 将数据所有者与数据管理者区分开来，前者可查看数据，而后者无权访问数据。
- [行级别安全性](/sql/relational-databases/security/row-level-security)，使客户能够根据执行查询的用户的特征，控制对数据库表中的行的访问。 （特征可以是组成员身份或执行上下文等。）

不使用数据库级别加密的组织可能更容易受到攻击，使 SQL 数据库中的数据面临透露的风险。

有关 SQL 数据库透明数据加密的详细信息，请参阅[使用 Azure SQL 数据库的透明数据加密](https://msdn.microsoft.com/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx)一文。

## <a name="enable-database-auditing"></a>启用数据库审核
审核 SQL Server 数据库引擎或单个数据库的实例涉及到跟踪和记录事件。 对于 SQL Server，可以创建包含服务器级别事件的规范和数据库级别事件的规范的审核。 可将审核的事件写入事件日志或审核文件。

SQL Server 有多个审核级别，具体取决于针对安装的政府要求或标准要求。 SQL Server 审核提供用于启用、存储和查看有关各个服务器和数据库对象的审核的工具和流程。

[Azure SQL 数据库审核](../sql-database/sql-database-auditing.md)跟踪数据库事件，并将事件写入 Azure 存储帐户中的审核日志。

审核可帮助你遵守法规、了解数据库活动，以及查出可以指明业务考量因素或安全违规的偏差和异常。 审核有助于遵从法规标准，但不能保证遵从法规。

若要了解有关数据库审核以及如何启用它的详细信息，请参阅 [SQL 数据库审核入门](../sql-database/sql-database-auditing.md)。

## <a name="enable-database-threat-detection"></a>启用数据库威胁检测
威胁防护超越了检测。 数据库威胁防护包括：

- 发现最敏感的数据并进行分类，以便可以保护数据。
- 在数据库上实现安全配置，以便可以保护数据库。
- 在出现潜在威胁时对其进行检测和响应，以便可以快速响应和修正。

**最佳做法**：发现数据库中的敏感数据并进行分类和标记。   
**详细信息**：通过启用 Azure SQL 数据库中的[数据发现和分类](../sql-database/sql-database-data-discovery-and-classification.md)对 SQL 数据库中的数据进行分类。 可以监视对 Azure 仪表板中的敏感数据的访问或下载报告。

**最佳做法**：跟踪数据库漏洞，以便可以主动提升数据库安全性。   
**详细信息**：使用 Azure SQL 数据库[漏洞评估](../sql-database/sql-vulnerability-assessment.md)服务，该服务可扫描潜在的数据库漏洞。 该服务采用标记安全漏洞的规则知识库，并显示与最佳做法的偏差，例如错误配置、过多权限和未受保护的敏感数据。

这些规则基于 Microsoft 的最佳做法，并专注于对数据库及其重要数据有最大风险的安全问题。 其中涵盖数据库级别的问题以及服务器级别的安全问题，如服务器防火墙设置和服务器级别的权限。 此外，这些规则也代表了监管机构的诸多要求，以满足其符合性标准。

**最佳做法**：启用威胁检测。  
**详细信息**：启用 Azure SQL 数据库[威胁检测](../sql-database/sql-database-threat-detection.md)以获取有关如何调查和缓解威胁的安全警报和建议。 你会收到有关可疑数据库活动、潜在漏洞、SQL 注入攻击以及异常数据库访问和查询模式的警报。

[高级威胁防护](../sql-database/sql-advanced-threat-protection.md)是高级 SQL 安全功能的统一程序包。 它包括前面提到的服务：数据发现和分类、漏洞评估和威胁检测。 它提供用于启用和管理这些功能的一个位置。

启用这些功能有助于：

- 满足数据隐私标准和法规符合性要求。
- 控制对数据库的访问并强化其安全性。
- 监视其中难以追踪变化的动态数据库环境。
- 检测和应对潜在威胁。

此外，威胁检测还会将警报与 Azure 安全中心集成，以便集中查看所有 Azure 资源的安全状态。

## <a name="next-steps"></a>后续步骤
有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com
