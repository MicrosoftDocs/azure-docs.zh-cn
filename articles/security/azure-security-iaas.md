---
title: Azure 中 IaaS 工作负荷的安全性最佳实践 | Microsoft Docs
description: " 将工作负荷迁移到 Azure IaaS 为重新评估设计带来了机会 "
services: security
documentationcenter: na
author: barclayn
manager: MBaldwin
editor: TomSh
ms.assetid: 02c5b7d2-a77f-4e7f-9a1e-40247c57e7e2
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/05/2019
ms.author: barclayn
ms.openlocfilehash: f4b2506781df5572ddaff8dda34bf3edab8987be
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145206"
---
# <a name="security-best-practices-for-iaas-workloads-in-azure"></a>Azure 中 IaaS 工作负荷的安全性最佳实践
本文介绍了 VM 和操作系统的安全最佳做法。

最佳做法以观点的共识以及 Azure 平台功能和特性集为基础。 由于观点和技术会随时改变，本文会进行更新以反映这些变化。

在大多数基础结构即服务 (IaaS) 方案中，[Azure 虚拟机 (VM)](https://docs.microsoft.com/azure/virtual-machines/) 是使用云计算的组织的主要工作负荷。 这种事实在[混合方案](https://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx)中十分明显，组织希望在混合方案中慢慢将工作负载迁移到云。 在这种方案中，应遵循 [IaaS 常规安全注意事项](https://social.technet.microsoft.com/wiki/contents/articles/3808.security-considerations-for-infrastructure-as-a-service-iaas.aspx)，并向所有 VM 应用安全最佳做法。

## <a name="shared-responsibility"></a>共担责任
对安全的责任取决于云服务的类型。 下图总结了 Microsoft 和客户的责任平衡：

![责任范围](./media/azure-security-iaas/sec-cloudstack-new.png)

安全要求取决于许多因素，包括不同类型的工作负载。 这些最佳实践没有一种可以单独保护系统。 就像安全中的其他内容，必须选择相应的选项，了解解决方案如何通过填补其他内容留下的缺口来相互补充。

## <a name="protect-vms-by-using-authentication-and-access-control"></a>通过身份验证和访问控制保护 VM
保护 VM 安全的第一步是确保只有授权用户才能设置新 VM 以及访问 VM。

> [!NOTE]
> 若要提高 Azure 上的 Linux Vm 的安全性，您可以通过 Azure AD 身份验证集成。 当你使用[适用于 Linux Vm 的 Azure AD 身份验证](../virtual-machines/linux/login-using-aad.md)，你可以集中控制并强制执行策略允许或拒绝对 Vm 的访问。
>
>

**最佳做法**：控制 VM 访问。   
**详细信息**：使用 [Azure 策略](../azure-policy/azure-policy-introduction.md)建立组织中的资源约定和创建自定义策略。 将这些策略应用于资源，如[资源组](../azure-resource-manager/resource-group-overview.md)。 属于该资源组的 VM 将继承该组的策略。

如果你的组织有多个订阅，则可能需要一种方法来高效地管理这些订阅的访问权限、策略和符合性。 [Azure 管理组](../azure-resource-manager/management-groups-overview.md)提供订阅上的作用域级别。 可将订阅组织到管理组（容器）中，并将管理条件应用到该组。 管理组中的所有订阅都将自动继承应用于该组的条件。 不管使用什么类型的订阅，管理组都能提供大规模的企业级管理。

**最佳做法**：减少 VM 的安装和部署的可变性。   
**详细信息**：使用 [Azure 资源管理器](../azure-resource-manager/resource-group-authoring-templates.md)模板增强部署选项，使其更易理解并清点环境中的 VM。

**最佳做法**：保护特权访问。   
**详细信息**：使用[最低特权方法](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models)和内置 Azure 角色使用户能够访问和设置 VM：

- [虚拟机参与者](../role-based-access-control/built-in-roles.md#virtual-machine-contributor)：可以管理 VM，但无法管理虚拟机连接的虚拟网络或存储帐户。
- [经典虚拟机参与者](../role-based-access-control/built-in-roles.md#classic-virtual-machine-contributor)：可管理使用经典部署模型创建的 VM，但无法管理这些 VM 连接到的虚拟网络或存储帐户。
- [安全管理员](../role-based-access-control/built-in-roles.md#security-admin)：仅在安全中心内：可以查看安全策略、查看安全状态、编辑安全策略、查看警报和建议、关闭警报和建议。
- [开发测试实验室用户](../role-based-access-control/built-in-roles.md#devtest-labs-user)：可以查看所有内容，以及连接、启动、重新启动和关闭 VM。

订阅管理员和共同管理员可更改此设置，使其成为订阅中所有 VM 的管理员。 请确保你信任所有订阅管理员和共同管理员，以登录你的任何计算机。

> [!NOTE]
> 建议将具有相同生命周期的 VM 合并到同一个资源组中。 使用资源组可以部署和监视资源，并统计资源的计费成本。
>
>

控制 VM 访问和设置的组织可改善其整体 VM 安全性。

## <a name="use-multiple-vms-for-better-availability"></a>使用多个 VM 提高可用性
如果 VM 运行需要具有高可用性的关键应用程序，我们强烈建议使用多个 VM。 为提高可用性，可使用[可用性集](../virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)。

可用性集是一种逻辑分组功能，在 Azure 中使用它可以确保将 VM 资源部署在 Azure 数据中心后，这些资源相互隔离。 Azure 确保可用性集中部署的 VM 能够跨多个物理服务器、计算机架、存储单元和网络交换机运行。 如果出现硬件或 Azure 软件故障，只有一部分 VM 会受到影响，整体应用程序仍可供客户使用。 如果想要构建可靠的云解决方案，可用性集是一项关键功能。

## <a name="protect-against-malware"></a>防范恶意软件
应安装反恶意软件保护，以帮助识别和删除病毒、间谍软件和其他恶意软件。 可安装 [Microsoft 反恶意软件](azure-security-antimalware.md)或 Microsoft 合作伙伴的终结点保护解决方案（[Trend Micro](https://help.deepsecurity.trendmicro.com/azure-marketplace-getting-started-with-deep-security.html)、[Symantec](https://www.symantec.com/products)、[McAfee](https://www.mcafee.com/us/products.aspx)、[Windows Defender](https://www.microsoft.com/search/result.aspx?q=Windows+defender+endpoint+protection) 和 [System Center Endpoint Protection](https://www.microsoft.com/search/result.aspx?q=System+Center+endpoint+protection)）。

Microsoft 反恶意软件包括实时保护、计划扫描、恶意软件修正、签名更新、引擎更新、示例报告和排除事件收集等功能。 对于与生产环境分开托管的环境，可以使用反恶意软件扩展来帮助保护 VM 和云服务。

可将 Microsoft 反恶意软件和合作伙伴解决方案与 [Azure 安全中心](https://docs.microsoft.com/azure/security-center/)集成，以方便部署和内置检测（警报和事件）。

**最佳做法**：安装反恶意软件解决方案，以防范恶意软件。   
**详细信息**：[安装 Microsoft 合作伙伴解决方案或 Microsoft 反恶意软件](../security-center/security-center-install-endpoint-protection.md)

**最佳做法**：将反恶意软件解决方案与安全中心集成，以监视保护状态。   
**详细信息**：[使用安全中心管理终结点保护问题](../security-center/security-center-partner-integration.md)

## <a name="manage-your-vm-updates"></a>管理 VM 更新
与所有本地 VM 一样，Azure VM 应由用户管理。 Azure 不会向他们推送 Windows 更新。 你需要管理 VM 更新。

**最佳做法**：使 VM 保持最新。   
**详细信息**：使用 Azure 自动化中的[更新管理](../automation/automation-update-management.md)解决方案，为部署在 Azure、本地环境或其他云提供程序中的 Windows 和 Linux 计算机管理操作系统更新。 可以快速评估所有代理计算机上可用更新的状态，并管理为服务器安装所需更新的过程。

由更新管理托管的计算机使用以下配置执行评估和更新部署：

- 用于 Windows 或 Linux 的 Microsoft 监视代理 (MMA)
- 用于 Linux 的 PowerShell 所需状态配置 (DSC)
- 自动化混合 Runbook 辅助角色
- 适用于 Windows 计算机的 Microsoft 更新或 Windows Server 更新服务 (WSUS)

若使用 Windows 更新，请启用 Windows 自动更新设置。

**最佳做法**：在部署时，确保构建的映像包含最新一轮的 Windows 更新。   
**详细信息**：每个部署的第一步应是检查和安装所有 Windows 更新。 在部署自己或库中提供的映像时，采用此措施就特别重要。 虽然默认情况下会自动更新 Azure 市场中的映像，但公开发布后可能会有延迟（最多几周）。

**最佳做法**：定期重新部署 VM 以强制刷新操作系统版本。   
**详细信息**：使用 [Azure 资源管理器模板](../azure-resource-manager/resource-group-authoring-templates.md)定义 VM，以便轻松地重新部署。 使用模板可在需要时提供已修补且安全的 VM。

**最佳做法**：快速应用于 Vm 的安全更新。   
**详细信息**：启用 Azure 安全中心 （免费层或标准层） 到[确定缺少安全更新并将其应用](../security-center/security-center-apply-system-updates.md)。

**最佳做法**：安装最新的安全更新。   
**详细信息**：客户移到 Azure 的部分首批工作负荷为实验室和面向外部的系统。 如果 Azure VM 托管需要访问 Internet 的应用程序或服务，则需要警惕修补。 修补不仅仅包括操作系统。 合作伙伴应用程序上未修补的漏洞还可能导致一些问题，而如果实施良好的修补程序管理，就可以避免这些问题。

**最佳做法**：部署并测试一个备份解决方案。   
**详细信息**：需要按照处理任何其他操作的相同方法处理备份。 这适合于属于扩展到云的生产环境的系统。

测试和开发系统必须遵循备份策略，这些策略可以根据用户的本地环境体验，提供与用户习惯的功能类似的存储功能。 如果可能，迁移到 Azure 的生产工作负荷应与现有的备份解决方案集成。 或者，可以使用 [Azure 备份](../backup/backup-azure-vms-first-look-arm.md)来帮助解决备份要求。

未实施软件更新策略的组织面临更多利用已修复的已知漏洞的威胁。 为了遵守行业法规，公司还必须证明他们在不断作出相应努力并使用正确的安全控制机制来帮助确保云中工作负载的安全性。

传统数据中心与 Azure IaaS 之间的软件更新最佳做法存在许多相似之处。 建议评估当前的软件更新策略，将位于 Azure 中的 VM 包含在内。

## <a name="manage-your-vm-security-posture"></a>管理 VM 安全状况
网络威胁不断加剧。 保护 VM 需要监视功能，以便快速检测威胁、防止有人未经授权访问资源、触发警报并减少误报。

若要监视 [Windows](../security-center/security-center-virtual-machine.md) 和 [Linux VM](../security-center/security-center-linux-virtual-machine.md) 的安全状况，可以使用 [Azure 安全中心](../security-center/security-center-intro.md)。 可以利用安全中心的以下功能来保护 VM：

- 应用包含建议的配置规则的 OS 安全设置。
- 识别并下载可能缺少的系统安全更新和关键更新。
- 部署终结点反恶意软件防护建议措施。
- 验证磁盘加密。
- 评估并修正漏洞。
- 检测威胁。

安全中心可主动监视威胁，并通过“安全警报”公开潜在的威胁。 关联的威胁将合并到名为“安全事件”的单个视图中。

安全中心将数据存储在[Azure Monitor 日志](../log-analytics/log-analytics-overview.md)。 Azure Monitor 日志提供了使你能够深入了解你的应用程序和资源的操作的查询语言和分析引擎。 数据也是从 [Azure Monitor](../monitoring-and-diagnostics/monitoring-overview.md)、管理解决方案以及安装在虚拟机（云中或本地）上的代理收集的数据。 可以通过此共享功能全面了解自己的环境。

没有为 VM 实施强大安全措施的组织将意识不到未经授权的用户可能试图绕过安全控制机制。

## <a name="monitor-vm-performance"></a>监视 VM 性能
如果 VM 进程消耗的资源多过实际所需的量，可能会造成资源滥用的问题。 VM 性能问题可能会导致服务中断，从而违反可用性安全原则。 这对于托管 IIS 或其他 Web 服务器的 VM 尤其重要，因为 CPU 或内存占用较高可能意味着遭到拒绝服务 (DoS) 攻击。 不仅要在出现问题时被动监视 VM 的访问，而且还要在正常运行期间针对基准性能进行主动监视。

我们还建议使用 [Azure Monitor](../monitoring-and-diagnostics/monitoring-overview-metrics.md) 来洞察资源的运行状况。 Azure Monitor 功能：

- [资源诊断日志文件](../azure-monitor/platform/diagnostic-logs-overview.md)：监视 VM 资源并识别可能会损害性能与可用性的潜在问题。
- [Azure 诊断扩展](../azure-monitor/platform/diagnostics-extension-overview.md)：在 Windows VM 上提供监视和诊断功能。 在 [Azure 资源管理器模板](../virtual-machines/windows/extensions-diagnostics-template.md)中包含该扩展即可启用这些功能。

不监视 VM 性能的组织无法确定性能模式的某些变化是正常还是异常。 若 VM 消耗的资源超过平常，可能意味着存在来自外部资源的攻击，或者此 VM 中有不安全的进程正在运行。

## <a name="encrypt-your-virtual-hard-disk-files"></a>加密虚拟硬盘文件
建议加密虚拟硬盘 (VHD)，以帮助保护存储中的静态启动卷和数据卷以及加密密钥和机密。

[Azure 磁盘加密](azure-security-disk-encryption-overview.md)用于加密 Windows 和 Linux IaaS 虚拟机磁盘。 Azure 磁盘加密利用 Windows 的行业标准 [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) 功能和 Linux 的 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 功能，为 OS 和数据磁盘提供卷加密。 该解决方案与 [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) 集成，帮助用户管理 Key Vault 订阅中的磁盘加密密钥和机密。 此解决方案还可确保虚拟机磁盘上的所有数据在 Azure 存储中静态加密。

下面是使用 Azure 磁盘加密的最佳做法：

**最佳做法**：在 VM 上启用加密。   
**详细信息**：Azure 磁盘加密将生成加密密钥并将其写入密钥保管库。 在 Key Vault 中管理加密密钥需要 Azure AD 身份验证。 为此，请创建 Azure AD 应用程序。 对于身份验证，可以使用基于客户端机密的身份验证或[基于客户端证书的 Azure AD 身份验证](../active-directory/active-directory-certificate-based-authentication-get-started.md)。

**最佳做法**：使用密钥加密密钥 (KEK) 来为加密密钥提供附加的安全层。 将 KEK 添加到密钥保管库。   
**详细信息**：使用[添加 AzKeyVaultKey](https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultkey) cmdlet 在密钥保管库中创建的密钥加密密钥。 还可从本地硬件安全模块 (HSM) 导入 KEK 以进行密钥管理。 有关详细信息，请参阅 [Key Vault 文档](../key-vault/key-vault-hsm-protected-keys.md)。 指定密钥加密密钥后，Azure 磁盘加密会使用该密钥包装加密机密，然后将机密写入 Key Vault。 在本地密钥管理 HSM 中保留此密钥的托管副本，提供额外的保护，防止意外删除密钥。

**最佳做法**：在加密磁盘之前创建[快照](../virtual-machines/windows/snapshot-copy-managed-disk.md)和/或备份。 如果加密期间发生意外故障，备份可提供恢复选项。   
**详细信息**：加密之前，需要备份包含托管磁盘的 VM。 执行备份后，可以使用**集 AzVMDiskEncryptionExtension** cmdlet 可通过指定加密托管的磁盘 *-skipVmBackup*参数。 有关如何备份和还原已加密 VM 的详细信息，请参阅 [Azure 备份](../backup/backup-azure-vms-encryption.md)一文。

**最佳做法**：为确保加密机密不会跨过区域边界，Azure 磁盘加密需要将密钥保管库和 VM 共置在同一区域。   
**详细信息**：在要加密的 VM 所在的同一区域中创建并使用密钥保管库。

Azure 磁盘加密可解决以下业务需求：

- 使用行业标准的加密技术轻松保护 IaaS VM，满足组织的安全性与合规性要求。
- IaaS VM 会根据客户控制的密钥和策略启动，客户可以在 Key Vault 中审核密钥和策略的使用方式。

## <a name="restrict-direct-internet-connectivity"></a>限制直接 internet 连接
监视并限制 VM 直接 internet 连接。 攻击者不断扫描公有云的打开的管理端口的 IP 范围，并尝试"简单"等常见密码和未打补丁的已知的漏洞的攻击。 下表列出了帮助保护对这些攻击的最佳做法：

**最佳做法**：防止无意中泄露网络路由和安全性。   
**详细信息**：使用 RBAC 来确保只为中心的网络组具有对网络资源的权限。

**最佳做法**：标识和修正允许从"任何"源 IP 地址进行访问的公开的 Vm。   
**详细信息**：使用 Azure 安全中心。 安全中心将建议限制通过面向 internet 的终结点的访问，如果任何网络安全组具有一个或多个入站的规则，允许从"任何"源 IP 地址进行访问。 安全中心将建议编辑这些入站的规则[限制访问](../security-center/security-center-restrict-access-through-internet-facing-endpoints.md)到实际需要访问的源 IP 地址。

**最佳做法**：限制管理端口 （RDP、 SSH）。   
**详细信息**：[实时 (JIT) VM 访问](../security-center/security-center-just-in-time.md)可以用来锁定发往 Azure Vm，降低遭受攻击，同时允许轻松访问要连接到 Vm 时所需入站流量。 启用 JIT 后，安全中心会通过创建网络安全组规则来锁定到 Azure Vm 的入站流量。 你需要选择要锁定 VM 上的哪些端口的入站流量。 这些端口控制 JIT 解决方案。

## <a name="next-steps"></a>后续步骤
有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com
