---
title: Azure Active Directory Connect 常见问题解答 | Microsoft Docs
description: 本文解答有关 Azure AD Connect 的常见问题。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 4e47a087-ebcd-4b63-9574-0c31907a39a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/03/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2caca430de5ad666f4f4341e0723bc3173d6d91a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65137788"
---
# <a name="azure-active-directory-connect-faq"></a>Azure Active Directory Connect 常见问题解答

## <a name="general-installation"></a>常规安装

**问：如何强制执行我的 Azure AD Connect 服务器以减少安全攻击面？**

Microsoft 建议强化你的 Azure AD Connect 服务器以减少您的 IT 环境的此关键组件的安全攻击面。  遵循以下建议将降低你的组织的安全风险。

* 部署在加入域的服务器上的 Azure AD Connect，并向域管理员或其他受到严格控制的安全组限制管理访问权限

若要了解更多信息，请参阅以下文章： 

* [保护管理员组](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/appendix-g--securing-administrators-groups-in-active-directory)

* [确保内置管理员帐户的安全](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/appendix-d--securing-built-in-administrator-accounts-in-active-directory)

* [安全改进并减少攻击面 sustainment](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access#2-reduce-attack-surfaces )

* [减少 Active Directory 攻击面](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/reducing-the-active-directory-attack-surface)

**问：如果 Azure Active Directory (Azure AD) 全局管理员已启用双重身份验证 (2FA)，安装是否能够正常进行？**  
2016 年 2 月版开始支持此方案。

**问：Azure AD Connect 是否提供无人值守安装方法？**  
仅支持使用安装向导来安装 Azure AD Connect。 不支持无人值守的静默安装。

**问：我有一个林，但无法连接到其中的某个域。如何安装 Azure AD Connect？**  
2016 年 2 月版开始支持此方案。

**问：Azure Active Directory 域服务 (Azure AD DS) 运行状况代理是否能够在服务器核心上运行？**  
可以。 安装代理后，可以使用以下 PowerShell cmdlet 完成注册过程： 

`Register-AzureADConnectHealthADDSAgent -Credentials $cred`

**问：Azure AD Connect 是否支持从两个域同步到一个 Azure AD？**  
是的。支持此方案。 请参阅[多个域](how-to-connect-install-multiple-domains.md)。
 
**问：是否可对 Azure AD Connect 中的同一个 Active Directory 域使用多个连接器？**  
不支持对同一个 AD 域使用多个连接器。 

**问：是否可将 Azure AD Connect 数据库从本地数据库移到远程 SQL Server 实例？**   
是的，以下步骤提供了此操作的一般指导。 我们目前正在努力编写更详细的文档。
1. 备份 LocalDB ADSync 数据库。
最简单的方法就是使用 Azure AD Connect 所在的同一台计算机上安装的 SQL Server Management Studio。 连接到 *(LocalDb).\ADSync*，然后备份 ADSync 数据库。

2. 将 ADSync 数据库还原到远程 SQL Server 实例。

3. 针对现有的[远程 SQL 数据库](how-to-connect-install-existing-database.md)安装 Azure AD Connect。
   本文演示了如何改用本地 SQL 数据库。 如果改用远程 SQL 数据库，则在此过程的步骤 5 中，还必须输入用于运行 Windows 同步服务的现有服务帐户。 下面描述了此同步引擎服务帐户：
   
      **使用现有的服务帐户**：默认情况下，Azure AD Connect 将虚拟服务帐户用于为要使用的同步服务。 如果使用远程 SQL Server 实例或使用需要身份验证的代理，请使用托管服务帐户，或者使用域中的服务帐户并知道密码。 在这些情况下，请输入要使用的帐户。 确保运行安装的用户是 SQL 中的系统管理员，以便可以创建服务帐户的登录凭据。 有关详细信息，请参阅 [Azure AD Connect 帐户和权限](reference-connect-accounts-permissions.md#adsync-service-account)。 
   
      现在，在使用最新版本的情况下，可以由 SQL 管理员在带外进行数据库预配，然后由具有数据库所有者权限的 Azure AD Connect 管理员完成安装。 有关详细信息，请参阅[使用 SQL 委派的管理员权限安装 Azure AD Connect](how-to-connect-install-sql-delegation.md)。

为简单起见，我们建议安装 Azure AD Connect 的用户是 SQL 中的系统管理员。 但是，在最新的版本中，现在也可以根据[使用 SQL 委派的管理员权限安装 Azure AD Connect](how-to-connect-install-sql-delegation.md) 中所述，使用委派的 SQL 管理员。

**问：一些字段中的最佳实践是什么？**  

以下是支持的工程，提供了一些最佳做法的信息性文档，多年来开发了我们的顾问。  这是可以快速引用的项目符号列表中显示。  虽然此列表试图详尽，但可能有其他可能不具有其列表尚未建立的最佳做法。

- 如果使用完整的 SQL，则它应保留在本地与远程
    - 更少跃点
    - 更轻松地进行故障排除
    - 减少复杂性
    - 需要将指定的资源添加到 SQL，并允许 Azure AD Connect 和 OS 开销
- 如有可能不使用代理服务器，如果您不能绕过代理，则需要确保超时值大于 5 分钟。
- 如果代理是必需的则必须将代理添加到 machine.config 文件
- 要注意的本地 SQL 作业和维护以及它们如何影响 Azure AD Connect-尤其重新编制索引
- 确保不是 DNS 可以从外部解析
- 絋粄[服务器规范](how-to-connect-install-prerequisites.md#hardware-requirements-for-azure-ad-connect)是按每个建议使用物理或虚拟服务器
- 确保如果使用所需的资源都专用的虚拟服务器
- 确保你具有的磁盘和磁盘配置适用于 SQL Server 符合最佳实践
- 安装和配置 Azure AD Connect Health 进行监视
- 使用内置于 Azure AD Connect 的删除阈值。
- 请仔细查看版本更新做好准备的所有更改和可能添加的新特性
- 备份的所有内容
    - 备份密钥
    - 备份同步规则
    - 备份服务器配置
    - 备份 SQL 数据库
- 请确保没有备份而无需 SQL VSS 编写器 （在使用第三方快照的虚拟服务器公用） 的 SQL 没有第三方备份代理
- 限制使用添加复杂的自定义同步规则的量
- 将 Azure AD 连接的服务器，因为层 0 个服务器
- 是不愿意使用修改没有很好理解的影响，以及正确的业务驱动程序的云同步规则
- 请确保正确的 URL 以及防火墙端口已打开用于支持的 Azure AD Connect 和 Azure AD Connect Health
- 利用云筛选的属性以进行故障排除并防止幻影对象
- 使用暂存服务器，请确保使用 Azure AD Connect 配置文档的服务器之间的一致性
- 过渡服务器应位于独立的数据中心 （物理位置
- 过渡服务器并不是高可用性解决方案，但可以有多个暂存服务器
- 引入了"延迟"暂存服务器可能会缓解一些潜在的停机时间发生错误时
- 测试并首先验证所有升级过渡服务器上
- 始终验证导出之前切换到过渡 serverLeverage 完整导入和完全同步，以减少业务影响暂存服务器
- 保留尽可能多地版本 Azure AD Connect 服务器之间的一致性 

**问：允许 Azure AD Connect 将在工作组计算机上创建 Azure AD 连接器帐户？**
不。  为了使 Azure AD Connect 将自动创建的 Azure AD 连接器帐户，计算机必须已加入域的。  

## <a name="network"></a>网络
**问：我的防火墙、网络设备或其他软硬件会限制在网络上打开连接的时间。使用 Azure AD Connect 时，客户端超时阈值应设为多少？**  
所有网络软件、物理设备或其他软硬件限制最长连接时间的阈值应该至少为 5 分钟 (300 秒)，使装有 Azure AD Connect 客户端的服务器能够与 Azure Active Directory 连接。 此项建议同样适用于以前发布的 Microsoft 标识同步工具。

**问：是否支持单一标签域 (SLD)？**  
虽然我们强烈建议不要使用此网络配置（[请参阅相关文章](https://support.microsoft.com/help/2269810/microsoft-support-for-single-label-domains)），但只要单级域的网络配置正常发挥作用，将 Azure AD Connect 同步与单标签域配合使用就是受支持的。

**问：是否支持具有非连续 AD 域的林？**  
Azure AD Connect 不支持包含非连续命名空间的本地林。

**问：是否支持包含句点的 NetBIOS 名称？**  
Azure AD Connect 不支持 NetBIOS 名称包含点号 (.) 的本地林或域。

**问：是否支持纯 IPv6 环境？**  
Azure AD Connect 不支持纯 IPv6 环境。

**问：我有一个多林环境，两个林之间的网络使用 NAT（网络地址转换）。是否支持在这两个林之间使用 Azure AD Connect？**</br>
 否，不支持通过 NAT 使用 Azure AD Connect。 

## <a name="federation"></a>联合
**问：如果我收到一封电子邮件，要求我续订 Office 365 证书，我该怎么办？**  
有关续订证书的指导，请参阅[续订证书](how-to-connect-fed-o365-certs.md)。

**问：我为 Office 365 信赖方设置了“自动更新信赖方”。当我的令牌签名证书自动滚动更新时，我是否需要采取任何措施？**  
请参考[续订证书](how-to-connect-fed-o365-certs.md)一文中所述的指导。

## <a name="environment"></a>环境
**问：安装 Azure AD Connect 之后，是否支持重命名服务器？**  
不。 更改服务器名称将导致同步引擎无法连接到 SQL 数据库实例，并且服务将无法启动。

**问：已启用 FIPS 的计算机上支持下一代加密 (NGC) 同步规则？**  
不。  不支持。

## <a name="identity-data"></a>标识数据
**问：Azure AD 中的 userPrincipalName (UPN) 属性为何与本地 UPN 不匹配？**  
有关信息，请参阅以下文章：

* [Office 365、Azure 或 Intune 中的用户名与本地 UPN 或备用登录 ID 不匹配](https://support.microsoft.com/kb/2523192)
* [在将用户帐户的 UPN 更改为使用不同的联合域后，Azure Active Directory 同步工具未同步更改](https://support.microsoft.com/kb/2669550)

还可以根据 [Azure AD Connect 同步服务功能](how-to-connect-syncservice-features.md)中所述配置 Azure AD，以允许同步引擎更新 UPN。

**问：是否支持本地 Azure AD 组或联系人对象与现有 Azure AD 组或联系人对象的软匹配？**  
是，这种软匹配取决于 proxyAddress。 未启用邮件的组不支持软匹配。

**问：是否支持手动设置现有 Azure AD 组或联系人对象的 ImmutableId 属性，以将其硬匹配到本地 Azure AD 组或联系人对象？**  
目前不支持在现有的 Azure AD 组或联系人对象中手动设置 ImmutableId 属性，以硬匹配该对象。

## <a name="custom-configuration"></a>自定义配置
**问：在哪里可以找到 Azure AD Connect 的 PowerShell cmdlet 介绍？**  
仅支持客户使用本站点上介绍的 cmdlet，而不支持使用 Azure AD Connect 中的其他 PowerShell cmdlet。

**问：是否可以使用 Synchronization Service Manager 中的“服务器导出/服务器导入”选项在服务器之间移动配置？**  
不。 此选项不会检索所有配置设置，因此不应使用。 请改用向导在第二台服务器上创建基础配置，并使用同步规则编辑器生成 PowerShell 脚本，如此即可在服务器之间移动任何自定义规则。 有关详细信息，请参阅[交叉迁移](how-to-upgrade-previous-version.md#swing-migration)。

**问：是否可以为 Azure 登录页缓存密码，这是否会因为包含一个具有 *autocomplete = "false"* 属性的密码输入元素而阻止此缓存？**  
目前不支持修改“密码”字段的 HTML 属性，包括 autocomplete 标记。 我们目前正在开发一种功能，它将允许使用自定义 JavaScript 向“密码”字段添加任何属性。

**问：Azure 登录页会显示之前已成功登录的用户的用户名。是否可以关闭此行为？**  
目前不支持修改“密码”输入字段的 HTML 属性，包括 autocomplete 标记。 我们目前正在开发一种功能，它将允许使用自定义 JavaScript 向“密码”字段添加任何属性。

**问：是否有方法来阻止并发会话？**  
不。

## <a name="auto-upgrade"></a>自动升级

**问：使用自动升级有什么好处？其结果是什么？**  
建议所有客户为安装的 Azure AD Connect 启用自动升级。 好处是客户可以一直接收最新的修补程序，包括在 Azure AD Connect 中发现的漏洞的安全更新。 升级过程很轻松，只要有新版本发布就会自动进行。 每次发布新版本，成千上万的 Azure AD Connect 客户都会使用自动升级。

自动升级过程始终会先确定某个安装是否符合自动升级的条件。 如果符合条件，则会执行并测试升级。 此过程还包括查找对规则的自定义更改和特定的环境因素。 如果测试表明升级未成功，则会自动还原以前的版本。

根据环境大小，此过程可能需要数小时才能完成。 在升级过程中，不会在 Windows Server Active Directory 和 Azure AD 之间进行同步。

**问：我收到一封电子邮件，指出我的自动升级失效，需安装新版本。为什么我需要这样做？**  
我们去年发布了一个 Azure AD Connect 版本，该版本在特定情况下会禁用服务器上的自动升级功能。 Azure AD Connect 1.1.750.0 版中已修复此问题。 如果你受此问题的影响，可通过以下方式进行缓解：运行一个 PowerShell 脚本来修复此问题，或者手动升级到最新版本的 Azure AD Connect。 

若要运行该 PowerShell 脚本，请[下载该脚本](https://aka.ms/repairaadconnect)，并在 PowerShell 管理窗口中的 Azure AD Connect 服务器上运行该脚本。 若要了解如何运行该脚本，请[观看此简短视频](https://aka.ms/repairaadcau)。

若要手动进行升级，必须下载并运行最新版的 AADConnect.msi 文件。
 
-  如果当前版本低于 1.1.750.0，请[下载并升级到最新版本](https://www.microsoft.com/download/details.aspx?id=47594)。
- 如果 Azure AD Connect 版本为 1.1.750.0 或更高，则不需要采取其他措施。 所用的版本已包含自动升级修复程序。 

**问：我收到一封电子邮件，要求我升级到最新版本，以便重新启用自动升级。我使用的版本是 1.1.654.0，需要升级吗？**  
需要。需要升级到 1.1.750.0 或更高版本才能重新启用自动升级。 [下载并升级到最新版本](https://www.microsoft.com/download/details.aspx?id=47594)。

**问：我收到一封电子邮件，要求我升级到最新版本，以便重新启用自动升级。如果我已经通过 PowerShell 启用了自动升级，是否仍需安装最新版本？**  
是的，仍需要升级到 1.1.750.0 或更高版本。 通过 PowerShell 启用自动升级服务不会解决在 1.1.750.0 之前的版本中发现的自动升级问题。

**问：我想要升级到更高版本，但不确定谁安装了 Azure AD Connect，而且我们没有用户名和密码。我们需要该凭据吗？**
不需要知道最初用来升级 Azure AD Connect 的用户名和密码。 可以使用任何具有全局管理员角色的 Azure AD 帐户。

**问：如何确定所用 Azure AD Connect 的版本？**  
若要确定安装在服务器上的 Azure AD Connect 的具体版本，请转到“控制面板”，然后选择“程序” > “程序和功能”并找到已安装的 Microsoft Azure AD Connect 版本，如下所示：

![控制面板中的 Azure AD Connect 版本](./media/reference-connect-faq/faq1.png)

**问：如何升级到最新版本的 Azure AD Connect？**  
若要了解如何升级到最新版本，请参阅 [Azure AD Connect：从旧版升级到最新版本](how-to-upgrade-previous-version.md)。 

**问：我们去年已升级到最新版本的 Azure AD Connect。是否需要再次升级？**  
Azure AD Connect 团队会对该服务进行频繁的更新。 若要充分利用 Bug 修复、安全更新和新功能的优势，必须使用最新版本来保持服务器的最新状态。 如果启用自动升级，则会自动更新软件版本。 若要查找 Azure AD Connect 的版本发布历史记录，请参阅 [Azure AD Connect：版本发布历史记录](reference-connect-version-history.md)。

**问：执行升级需要多长时间？对我的用户有什么影响？**  
升级所需时间取决于租户大小。 对于大型组织而言，最好是在晚上或周末升级。 在升级期间不会发生同步活动。

**问：我认为我升级到了 Azure AD Connect，但在 Office 门户中，仍然显示 DirSync。为什么会这样？**  
Office 团队会更新 Office 门户，使之反映当前的产品名称。 它不会反映所用的同步工具。

**问：我的自动升级状态显示为“已暂停”。为什么是“已暂停”？我应该启用它吗？**  
在以前的版本中存在一个 Bug，该 Bug 在特定情况下会将自动升级状态设置为“已暂停”。 手动启用它在技术上是可行的，但需要执行多个复杂的步骤。 最好是安装最新版本的 Azure AD Connect。

**问：我的公司有严格的更改管理要求，而我希望控制它的推出时间，我能控制自动升级的启动时间吗？**  
不能。目前没有此类功能。 我们正在评估是否在将来的版本中推出此功能。

**问：如果自动升级失败，是否会通过电子邮件通知我？怎么才能知道升级成功？**  
你不会收到升级结果的通知。 我们正在评估是否在将来的版本中推出此功能。

**问：你们是否会发布一个关于何时计划推出自动升级的时间线？**  
自动升级是新版本发布过程的第一个步骤。 只要有新版本，我们就会自动推送升级。 新版 Azure AD Connect 会在 [Azure AD 路线图](../fundamentals/whats-new.md)中预先公告。

**问：自动升级是否同时升级 Azure AD Connect Health？**  
是的，自动升级也会升级 Azure AD Connect Health。

**问：你们是否也会自动升级暂存模式下的 Azure AD Connect 服务器？**  
是的，可以自动升级暂存模式下的 Azure AD Connect 服务器。

**问：如果自动升级失败而 Azure AD Connect 服务器无法启动，该怎么办？**  
Azure AD Connect 服务偶尔会在升级以后无法启动。 在这种情况下，重新启动服务器通常就会解决问题。 如果 Azure AD Connect 服务仍然无法启动，请开具支持票证。 有关详细信息，请参阅[创建服务请求以联系 Office 365 支持部门](https://blogs.technet.microsoft.com/praveenkumar/2013/07/17/how-to-create-service-requests-to-contact-office-365-support/)。 

**问：我不知道升级到新版 Azure AD Connect 后会有什么风险。你们能通过电话帮助我升级吗？**  
如果在升级到新版 Azure AD Connect 时需要帮助，请参阅[创建服务请求以联系 Office 365 支持部门](https://blogs.technet.microsoft.com/praveenkumar/2013/07/17/how-to-create-service-requests-to-contact-office-365-support/)开具支持票证。

## <a name="troubleshooting"></a>故障排除
**问：如何获取有关 Azure AD Connect 的帮助？**

[搜索 Microsoft 知识库 (KB)](https://www.microsoft.com/en-us/search/result.aspx?q=azure+active+directory+connect)

* 在知识库 (KB) 中搜索有关 Azure AD Connect 支持的常见故障维修服务问题的技术解决方案。

[Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=WindowsAzureAD)

* 转到 [Azure AD 社区](https://social.msdn.microsoft.com/Forums/azure/en-US/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)，搜索技术问题与答案，或提出自己的问题。

[获取 Azure AD 支持](https://docs.microsoft.com/azure/active-directory/active-directory-troubleshooting-support-howto)
