---
title: Azure Site Recovery 新增功能 | Microsoft Docs
description: 汇总 Azure Site Recovery 中引入的新功能
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: raynew
ms.openlocfilehash: e2145fbbb5fa09aa3321742ca8a786822f6f0641
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148651"
---
# <a name="whats-new-in-site-recovery"></a>Site Recovery 中的新增功能

[Azure Site Recovery](site-recovery-overview.md) 服务会持续进行更新和改进。 本文介绍最新版本、新功能和新内容，让你始终了解最新动态。 此页会定期更新。

如果有关于 Site Recovery 功能的建议，请[提供反馈](https://feedback.azure.com/forums/256299-site-recovery)。


## <a name="updates-march-2019"></a>更新 (2019 年 3 月)

### <a name="update-rollup-35"></a>更新汇总 35

[更新汇总 35](https://support.microsoft.com/en-us/help/4494485/update-rollup-35-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 更新 Site Recovery 代理和提供程序 （如所述汇总）
**问题修复/改进** | 多个修复和改进 （如所述汇总）

#### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/物理服务器灾难恢复
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**托管磁盘** | 直接向 Azure 中的托管磁盘现已在本地 VMware Vm 和物理服务器复制。 在本地数据发送到缓存存储帐户在 Azure 中，以及在托管磁盘的目标位置中创建的恢复点。 这可确保无需管理多个目标存储帐户。
**配置服务器** | Site Recovery 现在支持具有多个 Nic 配置服务器。 配置服务器注册到保管库中之前，你必须将其他适配器添加到配置服务器 VM。 如果添加之后，您需要重新注册到保管库中的服务器。


## <a name="updates-february-2019"></a>更新 (2019 年 2 月)

### <a name="update-rollup-34"></a>更新汇总 34 

[更新汇总 34](https://support.microsoft.com/help/4490016/update-rollup-34-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。


### <a name="update-rollup-33"></a>更新汇总 33 

[更新汇总 33](https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。


#### <a name="azure-vm-disaster-recovery"></a>Azure VM 灾难恢复 
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**网络映射** | 对于 Azure VM 灾难恢复，你现在可以使用任何可用的目标网络时启用复制。 
**标准 SSD** | 你现在可以设置的 Azure Vm 使用灾难恢复[标准 SSD 磁盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-standard-ssd)。
**存储空间直通** | 您可以设置使用的 Azure VM 应用上运行的应用的灾难恢复[存储空间直通](https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview)以实现高可用性。  使用存储空间直通 (S2D) 与 Site Recovery 联合提供全面保护的 Azure VM 工作负荷。 S2D 可让你托管在 Azure 中的来宾群集。 当 VM 托管了关键的应用程序，如 SAP ASCS 层、 SQL Server 或横向扩展文件服务器，这是特别有用。


#### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/物理服务器灾难恢复
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux BRTFS 文件系统** | Site Recovery 现在支持 VMware Vm 复制使用 BRTFS 文件系统。 如果不支持复制：<br/><br/>-启用复制后更改了 BTRFS 文件系统子卷。<br/><br/>文件系统分散在多个磁盘。<br/><br/>-BTRFS 文件系统支持 RAID。
**Windows Server 2019** | 添加适用于运行 Windows Server 2019 计算机的支持。


## <a name="updates-january-2019"></a>更新 (2019 年 1 月)

### <a name="accelerated-networking-azure-vms"></a>加速的网络 (Azure Vm)

使用加速网络可以实现对 VM 的单根 I/O 虚拟化 (SR-IOV)，提升网络性能。 为 Azure VM 启用复制时，Site Recovery 会检测是否启用了加速网络。 如果启用了加速网络，Site Recovery 会在故障转移后自动在目标副本 Azure VM 上针对 [Windows](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell#enable-accelerated-networking-on-existing-vms) 和 [Linux](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli#enable-accelerated-networking-on-existing-vms) 配置加速网络。

[了解详细信息](azure-vm-disaster-recovery-with-accelerated-networking.md)。

### <a name="update-rollup-32"></a>更新汇总 32 

[更新汇总 32](https://support.microsoft.com/help/4485985/update-rollup-32-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。

#### <a name="azure-vm-disaster-recovery"></a>Azure VM 灾难恢复

在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 有关 Ubuntu、 Debian 和 SUSE RedHat 工作站 7 月 6 日，以及新的内核版本添加了支持。
**存储空间直通** | Site Recovery 支持使用存储空间直通 (S2D) 的 Azure Vm。

#### <a name="vmware-vmsphysical-servers-replication"></a>VMware Vm/物理服务器复制 
**功能** | **详细信息**
--- | ---
**Linux 支持** | 有关 Ubuntu、 Debian 和 SUSE 的 Redhat Enterprise Linux 7.6、 RedHat 工作站 7 月 6 日，Oracle Linux 6.10/7.6 和新的内核版本添加了支持。


### <a name="update-rollup-31"></a>更新汇总 31 

[更新汇总 31](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。

#### <a name="vmware-vmsphysical-servers-replication"></a>VMware Vm/物理服务器复制 
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 添加了支持以 Oracle Linux 6.8 和 6.9/7.0 以及 UEK5 内核。
**LVM** | LVM 和 LVM2 卷添加的支持。<br/><br/> 现在支持 /boot directory 和 LVM 卷上的磁盘分区。
**Directories** | 添加对这些目录将设置为单独的分区或不在同一系统磁盘的文件系统的支持：<br/><br/> /(root), /boot, /usr, /usr/local, /var, /etc.
**Windows Server 2008** | 添加了对动态磁盘的支持。
**故障转移** | Storvsc 和 vsbus 在不启动驱动程序的 VMware Vm 的改进了故障转移时间。
**UEFI 支持** | Azure Vm 不支持引导类型 UEFI。 现在可以使用 Site Recovery Azure 到迁移的本地物理服务器使用 UEFI。 站点恢复通过在迁移之前将引导类型转换为 BIOS 迁移服务器。 站点恢复以前此转换 Vm 仅支持。 支持是可用于物理服务器运行 Windows Server 2012 或更高版本。

#### <a name="azure-vm-disaster-recovery"></a>Azure VM 灾难恢复
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 支持添加了对 Oracle Linux 6.8 和 6.9/7.0;和 UEK5 内核。
**Linux BRTFS 文件系统** | Azure Vm 支持。
**可用性区域中的 azure Vm** | 可以启用部署在可用性区域中的 Azure Vm 的复制到另一个区域。 现在可以为 Azure VM 启用复制功能，并设置一个目标，以便故障转移到单个 VM 实例、可用性集中的 VM 或可用性区域中的 VM。 此设置不影响复制。 [阅读](https://azure.microsoft.com/blog/disaster-recovery-of-zone-pinned-azure-virtual-machines-to-another-region/)公告。
**启用了防火墙的存储 (门户/PowerShell)** | 添加了对支持[启用了防火墙的存储帐户](https://docs.microsoft.com/azure/storage/common/storage-network-security)。<br/><br/> 可以使用为另一个 Azure 区域的灾难恢复的启用防火墙的存储帐户上的非托管磁盘复制 Azure Vm。<br/><br/> 可以对非托管磁盘作为目标存储帐户中使用启用了防火墙的存储帐户。<br/><br/> 在受支持的门户，并使用 PowerShell。

## <a name="updates-december-2018"></a>更新 (2018 年 12 月)

### <a name="automatic-updates-for-the-mobility-service-azure-vms"></a>移动服务 (Azure Vm) 的自动更新

Site Recovery 增加了一个选项，可以针对移动服务扩展进行自动更新。 移动服务扩展安装在每个通过 Site Recovery 复制的 Azure VM 上。 启用复制时，请选择是否允许 Site Recovery 管理对扩展的更新。

更新不需 VM 重启，且不影响复制。 [了解详细信息](azure-to-azure-autoupdate.md)。

### <a name="pricing-calculator-for-azure-vm-disaster-recovery"></a>针对 Azure VM 灾难恢复的价格计算器

灾难恢复的 Azure Vm 会产生 VM 许可成本，以及网络和存储成本。 Azure 提供[定价计算器](https://aka.ms/a2a-cost-estimator)，用于计算这些费用。 Site Recovery 现在提供[定价估算](https://aka.ms/a2a-cost-estimator)功能，可以根据三层应用的情况对示例部署定价。该应用使用 6 个 VM，其中包含 12 个标准 HDD 磁盘和 6 个高级 SSD 磁盘。

- 该示例假定一个标准的日期和高级版为 20 GB 的 10 GB 的数据更改。
- 对于特定的部署，可以更改变量来估算费用。
- 可以指定 VM 的数目、托管磁盘的数目和类型，以及预期的跨 VM 总数据更改率。
- 另外，可以通过应用一个压缩因素来估算带宽费用。

[阅读](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/)公告。


## <a name="updates-october-2018"></a>更新 (2018 年 10 月)

### <a name="update-rollup-30"></a>更新汇总 30 

[更新汇总 30](https://support.microsoft.com/help/4468181/azure-site-recovery-update-rollup-30)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。

#### <a name="azure-vm-disaster-recovery"></a>Azure VM 灾难恢复
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**区域支持** | Site Recovery 支持为澳大利亚中部 1 和澳大利亚中部 2 添加了。
**对磁盘加密的支持** | 添加了对 Azure 磁盘加密 (ADE) 加密与 Azure AD 应用的 Azure Vm 的灾难恢复的支持。 [了解详细信息](azure-to-azure-how-to-enable-replication-ade-vms.md)。
**磁盘排除** | 未初始化的磁盘现在会在 Azure VM 复制期间自动排除。
**启用了防火墙的存储 (PowerShell)** | 添加了对支持[启用了防火墙的存储帐户](https://docs.microsoft.com/azure/storage/common/storage-network-security)。<br/><br/> 可以使用为另一个 Azure 区域的灾难恢复的启用防火墙的存储帐户上的非托管磁盘复制 Azure Vm。<br/><br/> 可以对非托管磁盘作为目标存储帐户中使用启用了防火墙的存储帐户。<br/><br/> 支持仅使用 PowerShell。


### <a name="update-rollup-29"></a>更新汇总 29 

[更新汇总 29](https://support.microsoft.com/help/4466466/update-rollup-29-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。


## <a name="updates-august-2018"></a>更新 (2018 年 8 月)

### <a name="update-rollup-28"></a>更新汇总 28 

[更新汇总 28](https://support.microsoft.com/help/4460079/update-rollup-28-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。

#### <a name="azure-vms-disaster-recovery"></a>Azure Vm 灾难恢复 
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 添加受支持的 RedHat Enterprise Linux 6.10;CentOS 6.10。<br/><br/>
**云支持** | 支持在德国云中的 Azure Vm 灾难恢复。
**跨订阅灾难恢复** | 将一个区域中的 Azure Vm 复制到在不同订阅中，同一 Azure Active Directory 租户中的另一个区域的支持。 [了解详细信息](https://aka.ms/cross-sub-blog)。

#### <a name="vmware-vmphysical-server-disaster-recovery"></a>VMware VM/物理服务器灾难恢复 
在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 添加对 RedHat Enterprise Linux 6.10，CentOS 6.10 支持。<br/><br/> 基于 Linux 的虚拟机现在支持使用在传统的 BIOS 兼容性模式中的 GUID 分区表 (GPT) 分区样式。 审阅[Azure VM 常见问题解答](https://docs.microsoft.com/azure/virtual-machines/linux/faq-for-disks)有关详细信息。 
**迁移后的 Vm 的灾难恢复** | 支持为启用到次要区域的灾难恢复的本地 VMware VM 迁移到 Azure，而无需卸载移动服务在 VM 上启用复制之前。
**Windows Server 2008** | 对迁移的支持计算机正在运行 Windows Server 2008 R2/2008年 64 位和 32 位。<br/><br/> 仅适用于迁移 （复制和故障转移）。 不支持故障回复。

## <a name="updates-july-2018"></a>更新 (2018 年 7 月)

### <a name="update-rollup-27-july-2018"></a>更新汇总 27 (2018 年 7 月)

[更新汇总 27](https://support.microsoft.com/help/4055712/update-rollup-27-for-azure-site-recovery)提供了以下更新。

**更新** | **详细信息**
--- | ---
**提供程序和代理** | 对 Site Recovery 代理和提供程序 （如所述汇总） 的更新。
**问题修复/改进** | 修复和改进 （如所述汇总） 数。

#### <a name="azure-vms-disaster-recovery"></a>Azure Vm 灾难恢复 

在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 添加了对 Red Hat Enterprise Linux 7.5 的支持。

#### <a name="vmware-vmphysical-server-disaster-recovery"></a>VMware VM/物理服务器灾难恢复 

在更新中添加的新功能。

**功能** | **详细信息**
--- | ---
**Linux 支持** | 添加了对 Red Hat Enterprise Linux 7.5，SUSE Linux Enterprise Server 12 的支持。



## <a name="next-steps"></a>后续步骤

在 [Azure 更新](https://azure.microsoft.com/updates/?product=site-recovery)页上时刻了解更新。
