---
title: Azure VM 备份的 Azure 备份支持矩阵
description: 提供有关在使用 Azure 备份服务备份 Azure VM 时的支持设置和限制摘要。
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: raynew
ms.openlocfilehash: 2267a4e836fe1aff214f40e34afa830de50fa2d5
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471651"
---
# <a name="support-matrix-for-azure-vm-backup"></a>Azure VM 备份的支持矩阵
可以使用[Azure 备份服务](backup-overview.md)备份的本地计算机和工作负荷和 Azure 虚拟机 (Vm)。 使用 Azure 备份的 Azure Vm 备份时，本文汇总了支持设置和限制。

其他支持矩阵：

- [常规支持矩阵](backup-support-matrix.md)Azure 备份
- [支持矩阵](backup-support-matrix-mabs-dpm.md)Azure 备份服务器/system Center Data Protection Manager (DPM) 备份
- [支持矩阵](backup-support-matrix-mars-agent.md)使用 Microsoft Azure 恢复服务 (MARS) 代理进行备份

## <a name="supported-scenarios"></a>支持的方案

下面描述了使用 Azure 备份服务备份和还原 Azure VM 的方式。

**方案** | **备份** | **代理** |**Restore**
--- | --- | --- | ---
直接备份 Azure VM  | 备份整个虚拟机。  | 无需在 Azure VM 上安装代理。 Azure 备份安装和使用的扩展[Azure VM 代理](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows)在 VM 上运行。 | 按如下所述进行还原：<br/><br/> - **创建基本的 VM**。 这是很有用，如果 VM 不有多个 IP 地址等任何特殊配置。<br/><br/> - **还原 VM 磁盘**。 还原磁盘。 然后将其附加到现有 VM，或使用 PowerShell 从磁盘创建新的 VM。<br/><br/> - **替换 VM 磁盘**。 如果 VM 存在并使用托管磁盘（未加密），则你可以还原磁盘并使用它来替换 VM 上的现有磁盘。<br/><br/> - **还原特定的文件/文件夹**。 可以从 VM，而不是从整个 VM 还原文件/文件夹。
Azure Vm (仅 Windows) 的直接备份  | 备份特定的文件/文件夹/卷。 | 安装[Azure 恢复服务代理](backup-azure-file-folder-backup-faq.md)。<br/><br/> 可将 MARS 代理与适用于 Azure VM 代理的备份扩展一同运行，以便在文件/文件夹级别备份 VM。 | 还原特定的文件夹/文件。
将 Azure VM 备份到备份服务器  | 备份文件/文件夹/卷;系统状态/裸机金属文件;System Center DPM 或到 Microsoft Azure 备份服务器 (MABS) 的应用程序数据。<br/><br/> DPM/MABS 然后备份到备份保管库。 | 在 VM 上安装 DPM/MABS 保护代理。 在 DPM/MABS 上安装 MARS 代理。| 还原文件/文件夹/卷；系统状态/裸机文件；应用数据。

了解有关备份的详细信息[使用备份服务器](backup-architecture.md#architecture-back-up-to-dpmmabs)而是有关[支持要求](backup-support-matrix-mabs-dpm.md)。

## <a name="supported-backup-actions"></a>支持的备份操作

**Action** | **支持**
--- | ---
创建 Windows Azure VM 时启用备份 | 支持：Windows Server 2019 （数据中心/数据中心核心）、 Windows Server 2016 （数据中心/数据中心核心）;Windows Server 2012 R2 数据中心;Windows Server 2008 R2 （RTM 和 SP1）
创建 Linux VM 时启用备份 | 支持：<br/><br/> - Ubuntu Server：18.04、 17.10、 17.04，16.04 (LTS)、 14.04 (LTS)<br/><br/> - Red Hat：RHEL 6.7、6.8、6.9、7.2、7.3、7.4<br/><br/> - SUSE Linux Enterprise Server：11 SP4，12 SP2，12 SP3，15 <br/><br/> - Debian：8、9<br/><br/> - CentOS：6.9、7.3<br/><br/> -Oracle Linux:6.7、6.8、6.9、7.2、7.3
备份 VM 关闭/脱机 VM | 。<br/><br/> 仅创建崩溃一致性快照，不会创建应用一致性快照。
备份磁盘后迁移到托管磁盘 | 。<br/><br/> 备份将继续工作。 无需任何操作。
启用资源组锁定后备份托管磁盘 | 不支持。<br/><br/> Azure 备份不能删除较旧的资源点，并且备份将开始失败时达到的还原点的最大限制。
修改 VM 的备份策略 | 。<br/><br/> 将新策略中使用的计划和保留期设置备份 VM。 如果保留期设置已延长，则会标记并保留现有的恢复点。 如果它们的操作已简化，将在下一清理作业中修剪并最终删除现有的恢复点。
取消备份作业 | 在快照过程中受支持。<br/><br/> 快照正在传输到保管库时不受支持。
将 VM 备份到其他区域或订阅 |  不支持。
每日备份（通过 Azure VM 扩展） | 每日进行一次计划的备份。<br/><br/> 您可以进行每日最多四个按需备份。
每日备份（通过 MARS 代理） | 每日进行三次计划的备份。
每日备份（通过 DPM/MABS） | 每日进行两次计划的备份。
每月/每年备份   | 使用 Azure VM 扩展备份时不受支持。 仅支持每日和每周备份。<br/><br/> 可将策略设置为按每月/每年保留期保留每日/每周备份。
自动时钟调整 | 不支持。<br/><br/> 备份 VM 时，azure 备份不会自动调整为夏时制更改。<br/><br/>  请根据需要手动修改策略。
[混合备份的安全功能](https://docs.microsoft.com/azure/backup/backup-azure-security-feature) |  禁用安全功能不受支持。

## <a name="operating-system-support-windows"></a>操作系统支持 (Windows)

下表汇总了支持的操作系统，Windows Azure Vm 备份。

**方案** | **OS 支持**
--- | ---
使用 Azure VM 代理扩展进行备份 | Windows 客户端：不支持<br/><br/> Windows Server 2019 （数据中心/数据中心核心）、 Windows Server 2016 （数据中心/数据中心核心）;Windows Server 2012 R2 数据中心;Windows Server 2008 R2 （RTM 和 SP1）
使用 MARS 代理进行备份 | [支持](backup-support-matrix-mars-agent.md#support-for-direct-backups)的操作系统。
使用 DPM/MABS 备份 | 使用 [MABS](backup-mabs-protection-matrix.md) 和 [DPM](https://docs.microsoft.com/system-center/dpm/dpm-protection-matrix?view=sc-dpm-1807) 备份时支持的操作系统。

## <a name="support-for-linux-backup"></a>Linux 备份支持

下面是在需要备份 Linux 计算机的情况下的支持项目。

**Action** | **支持**
--- | ---
使用 Linux Azure VM 代理备份 Linux Azure VM | 文件一致性备份。<br/><br/> 使用[自定义脚本](backup-azure-linux-app-consistent.md)的应用一致性备份。<br/><br/> 还原过程中，可以创建新的 VM，还原磁盘并使用它来创建 VM，或还原磁盘并使用它来替换现有的 VM 上的磁盘。 还可以还原单个文件和文件夹。
使用 MARS 代理备份 Linux Azure VM | 不支持。<br/><br/> MARS 代理只能安装在 Windows 计算机上。
使用 DPM/MABS 备份 Linux Azure VM | 不支持。

## <a name="operating-system-support-linux"></a>操作系统支持 (Linux)

对于 Azure VM Linux 备份，Azure 备份支持 [Azure 认可的 Linux 分发版](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)列表。 注意以下事项：

- Azure 备份不支持 Core OS Linux。
- Azure 备份不支持 32 位操作系统。
- 其他自带您自己的 Linux 发行版可能正常工作，只要[适用于 Linux 的 Azure VM 代理](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux)可用的 VM 上，并且只要支持 Python。
- Azure 备份不支持代理配置 Linux VM，如果它没有 Python 2.7 安装版。


## <a name="backup-frequency-and-retention"></a>备份频率和保留期

**设置** | **限制**
--- | ---
每个受保护实例 （机工作负荷） 的最大恢复点 | 9999.
恢复点的最长过期时间 | 没有限制。
备份到保管库时的最高备份频率（Azure VM 扩展） | 每天一次。
备份到保管库时的最高备份频率（MARS 代理） | 每日三次备份。
备份到 DPM/MABS 时的最高备份频率 | SQL Server 每隔 15 分钟。<br/><br/> 每小时一次的其他工作负荷。
恢复点保留期 | 每日、 每周、 每月和每年。
最长数据保留期 | 取决于备份频率。
DPM/MABS 磁盘上的恢复点数 | 对于文件服务器和应用程序服务器为 448 为 64。<br/><br/> 对于本地 DPM 来说，磁带恢复点没有限制。

## <a name="supported-restore-methods"></a>支持的还原方法

**还原方法** | **详细信息**
--- | ---
创建新 VM | 可以在还原过程中创建 VM。 <br/><br/> 此选项可以启动并运行一个基本的 VM。 可以指定 VM 名称、资源组、虚拟网络、子网和存储。  
还原磁盘 | 可以还原磁盘并使用它来创建 VM。<br/><br/> 如果选择此选项，Azure 备份会将保管库中的数据复制到所选的存储帐户。 还原作业生成一个模板。 可以下载此模板，使用它来指定 VM 的自定义设置，并创建 VM。<br/><br/> 与前面用于创建 VM 的选项相比，使用此选项可以指定更多的设置。<br/><br/>
替换现有磁盘 | 可以还原某个磁盘，然后使用还原的磁盘来替换当前 VM 上的磁盘。
还原文件 | 可以从所选的恢复点恢复文件。 下载一个脚本，用于从恢复点装载 VM 磁盘。 您然后浏览要查找你想要恢复完成后卸载磁盘文件/文件夹的磁盘卷。

## <a name="support-for-file-level-restore"></a>文件级还原支持

**Restore** | **支持**
--- | ---
跨操作系统还原文件 | 可以在与备份的 VM 使用相同（或兼容）OS 的任何计算机上还原文件。 请参阅[兼容的 OS 表](backup-azure-restore-files-from-vm.md#system-requirements)。
还原经典 VM 上的文件 | 不支持。
还原已加密 VM 中的文件 | 不支持。
还原网络受限的存储帐户中的文件 | 不支持。
使用 Windows 存储空间还原 VM 上的文件 | 不支持在同一 VM 上还原。<br/><br/> 应在兼容的 VM 上还原文件。
使用 LVM/RAID 阵列还原 Linux VM 上的文件 | 不支持在同一 VM 上还原。<br/><br/> 在兼容的 VM 上还原。
使用特殊网络设置还原文件 | 不支持在同一 VM 上还原。 <br/><br/> 在兼容的 VM 上还原。

## <a name="support-for-vm-management"></a>VM 管理支持

下表汇总了在 VM 管理任务，例如添加或替换 VM 磁盘备份的支持。

**Restore** | **支持**
--- | ---
跨订阅/区域/局部区域还原。 | 不支持。
还原到现有 VM | 使用“替换磁盘”选项。
在为存储帐户启用了 Azure 存储服务加密 (SSE) 的情况下还原磁盘 | 不支持。<br/><br/> 还原到未启用 SSE 的帐户。
还原到混合存储帐户 | 不支持。<br/><br/> 根据存储帐户类型，所有已还原的磁盘将是高级或标准类型，而不是混合类型。
使用区域冗余存储 (ZRS) 还原到存储帐户 | 支持 (将备份在 2019 年 1 月之后的 vm 以及在何处[可用性区域](https://azure.microsoft.com/global-infrastructure/availability-zones/)可用)
将 VM 直接还原到可用性集 | 对于托管磁盘，可以将磁盘还原并在模板中使用可用性组选项。<br/><br/> 不支持非托管磁盘。 对于非托管磁盘，可以还原磁盘，然后在可用性集中创建 VM。
非托管 Vm 的备份还原后升级到托管 VM| 。<br/><br/> 可以还原磁盘，然后创建托管 VM。
在将 VM 迁移到托管磁盘之前将 VM 还原到还原点 | 。<br/><br/> 还原到非托管磁盘（默认设置），将已还原的磁盘转换为托管磁盘，然后使用托管磁盘创建 VM。
还原已删除的 VM。 | 。<br/><br/> 可以从恢复点还原 VM。
通过门户还原属于多 DC 配置的域控制器 (DC) VM | 如果还原磁盘并使用 PowerShell 创建虚拟机，支持。
在不同虚拟网络中还原 VM |   。<br/><br/> 虚拟网络必须位于同一订阅和区域中。

## <a name="vm-compute-support"></a>VM 计算支持

**计算** | **支持**
--- | ---
VM 大小 |   至少有 2 个 CPU 核心和 1-GB RAM 的任意 Azure VM 大小。<br/><br/> [了解详细信息。](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)
备份[可用性集](https://docs.microsoft.com/azure/virtual-machines/windows/regions-and-availability#availability-sets)中的 VM | 。<br/><br/> 使用选项来快速创建 VM，不能还原中提供的一组 VM。 相反，在还原 VM 时，还原磁盘并使用它来部署 VM，或还原磁盘并使用它来替换现有磁盘。
备份[可用性区域](https://docs.microsoft.com/azure/availability-zones/az-overview)中的 VM |  不支持。
使用部署的 Vm 备份[混合使用权益 (HUB)](https://docs.microsoft.com/azure/virtual-machines/windows/hybrid-use-benefit-licensing) | 。
在中部署的 Vm 备份[规模集](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview) |  不支持。
从部署的 Vm 备份[Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?filters=virtual-machine-images)<br/><br/> （由 Microsoft、 第三方发布） |  。<br/><br/> VM 必须运行受支持的操作系统。<br/><br/> 恢复 VM 上的文件时，可以仅还原到兼容的 OS（不是早期版本或更高版本的 OS）。
备份自定义映像 （第三方） 中部署的 Vm |   。<br/><br/> VM 必须运行受支持的操作系统。<br/><br/> 恢复 VM 上的文件时，可以仅还原到兼容的 OS（不是早期版本或更高版本的 OS）。
迁移到 Azure 的 Vm 备份  | 。<br/><br/> 若要备份 VM，必须在迁移的计算机上安装 VM 代理。
备份多 VM 一致性 | Azure 备份不提供跨多个 Vm 的数据和应用程序一致性。
备份与[诊断设置](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-logs-overview)  | 不受支持。 <br/><br/> 如果使用触发使用诊断设置的 Azure VM 还原[创建新](backup-azure-arm-restore-vms.md#create-a-vm)选项则还原将失败。


## <a name="vm-storage-support"></a>VM 存储支持

组件 | **支持**
--- | ---
Azure VM 数据磁盘 | 备份包含 16 个或更少数据磁盘的 VM。 <br/><br/> 最大支持 4 TB 的磁盘。
数据磁盘大小 | 单个磁盘最大可以为 4095 GB。<br/><br/> 如果将保管库运行最新版本的 Azure 备份 （称为即时还原），磁盘大小，最多 4 TB 受支持。 [了解详细信息](backup-instant-restore-capability.md)。  
存储类型 | 标准 HDD，标准的 SSD，高级 SSD。 <br/><br/> 如果将保管库升级到最新版本的 Azure VM 备份 （称为即时还原），则支持标准 SSD。 [了解详细信息](backup-instant-restore-capability.md)。
托管磁盘 | 。
加密的磁盘 | 。<br/><br/> 可以备份使用 Azure 磁盘加密启用的 azure Vm （有或没有 Azure AD 应用程序）。<br/><br/> 无法在文件/文件夹级别恢复已加密的 VM。 你必须恢复整个虚拟机。<br/><br/> 可以在已受 Azure 备份保护的 VM 上启用加密。
已启用写入加速器的磁盘 | 不支持。<br/><br/> 如果运行最新版本的 Azure VM 备份（称为[即时还原](backup-instant-restore-capability.md)），则可以从备份中排除已启用写入加速器的磁盘。
备份已删除重复数据的磁盘 | 不支持。
将磁盘添加到受保护的 VM | 。
调整受保护 VM 上的磁盘大小 | 。
共享的存储| 不建议使用群集共享卷 (CSV) 或横向扩展文件服务器的 Vm 备份。 CSV 编写器均可能会在备份期间失败。 在恢复时，包含 CSV 卷的磁盘可能不附带的最高。

> [!NOTE]
> Azure 备份不建议重设磁盘大小。


## <a name="vm-network-support"></a>VM 网络支持

组件 | **支持**
--- | ---
网络接口 (Nic) 数 | 特定 Azure VM 大小支持最大数量的 NIC。<br/><br/> NIC 是在还原过程中创建 VM 时创建的。<br/><br/> 已还原 VM 上的 NIC 数目与启用保护时 VM 上的 NIC 数目相同。 删除 Nic，在启用保护后不会影响该计数。
内部/外部负载均衡器 |   。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
多个保留的 IP 地址 |    。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
具有多个网络适配器的 VM  | 。 <br/><br/> [详细了解](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations)如何还原使用特殊网络设置的 VM。
具有公共 IP 地址的 VM    | 。<br/><br/> 将现有的公共 IP 地址与 NIC 关联或创建一个地址并完成还原后将其与 NIC 关联。
NIC/子网上的网络安全组 (NSG)。 |   。
保留的 IP 地址（静态） | 不支持。<br/><br/> 不能备份使用保留的 IP 地址和未定义终结点的 VM。
动态 IP 地址 |    。<br/><br/> 如果在源上的 NIC VM 使用动态 IP 寻址，默认情况下已还原的 VM 上的 NIC 将使用它过。
Azure 流量管理器   | 。<br/><br/>如果备份 VM，流量管理器中手动将已还原的 VM 添加到同一个流量管理器实例。
Azure DNS | 。
自定义 DNS |    。
通过 HTTP 代理建立出站连接 | 。<br/><br/> 不支持经过身份验证的代理。
虚拟网络服务终结点   | 。<br/><br/> 防火墙和虚拟网络的存储帐户设置应允许来自所有网络访问。



## <a name="vm-security-and-encryption-support"></a>VM 安全和加密支持

Azure 备份支持针对传输中数据和静态数据的加密：

发往 Azure 的网络流量：

- 从服务器到恢复服务保管库备份流量使用高级加密标准 256 进行加密。
- 备份数据通过安全 HTTPS 链接进行发送。
- 备份数据以加密格式存储在恢复服务保管库中。
- 只有你有解锁此数据的通行短语。 Microsoft 无法解密任何恢复点的备份数据。

  > [!WARNING]
  > 设置保管库后，你只能访问加密密钥。 Microsoft 不保留副本，且没有访问该密钥的权限。 如果客户丢失了密钥，Microsoft 无法恢复备份数据。

数据安全：

- 当备份 Azure Vm，需要设置加密*内*虚拟机。
- Azure 备份支持 Azure 磁盘加密，在 Windows 虚拟机和我们使用 BitLocker **dm crypt** Linux 虚拟机上。
- 在后端，Azure 备份使用 [Azure 存储服务加密](../storage/common/storage-service-encryption.md)来保护静态数据。


**计算机** | **传输中** | **静态**
--- | --- | ---
没有 DPM/MABS 的本地 Windows 计算机 | ![“是”][green] | ![“是”][green]
Azure VM | ![“是”][green] | ![“是”][green]
本地计算机/装有 DPM 的 Azure VM | ![“是”][green] | ![“是”][green]
本地计算机/装有 MABS 的 Azure VM | ![“是”][green] | ![“是”][green]



## <a name="vm-compression-support"></a>VM 压缩支持

下表中进行了总结，backup 支持备份流量的压缩。 注意以下事项：

- 对于 Azure Vm，VM 扩展读取的数据通过存储网络直接从 Azure 存储帐户中。 不需要优化此流量。
- 如果你使用 DPM 或 MABS，可以通过它备份到 DPM/MABS 之前将数据压缩来节省带宽。

**计算机** | **压缩到 MABS/DPM (TCP)** | **压缩到保管库 (HTTPS)**
--- | --- | ---
没有 DPM/MABS 的本地 Windows 计算机 | NA | ![“是”][green]
Azure VM | NA | NA
本地计算机/装有 DPM 的 Azure VM | ![“是”][green] | ![“是”][green]
本地计算机/装有 MABS 的 Azure VM | ![“是”][green] | ![“是”][green]


## <a name="next-steps"></a>后续步骤

- [Azure Vm 备份](backup-azure-arm-vms-prepare.md)。
- 不使用备份服务器[直接备份 Windows 计算机](tutorial-backup-windows-server-to-azure.md)。
- [设置 MABS](backup-azure-microsoft-azure-backup.md) 以备份到 Azure，然后将工作负荷备份到 MABS。
- [设置 DPM](backup-azure-dpm-introduction.md) 以备份到 Azure，然后将工作负荷备份到 DPM。

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
