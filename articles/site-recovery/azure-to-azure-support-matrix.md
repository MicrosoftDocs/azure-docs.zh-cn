---
title: 使用 Azure Site Recovery 的 Azure 区域之间的 Azure Vm 灾难恢复的支持矩阵 |Microsoft Docs
description: 总结了先决条件和支持的 Azure Vm 从一个区域到另一个使用 Azure Site Recovery 灾难恢复
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 04/29/2019
ms.author: raynew
ms.openlocfilehash: 1118d1de72ca7cd44844a0b526efd85eb419bc67
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412762"
---
# <a name="support-matrix-for-replicating-azure-vms-from-one-region-to-another"></a>从一个区域的 Azure Vm 复制到另一个支持矩阵

本文汇总了支持和必备组件时设置的灾难恢复 Azure Vm 从一个 Azure 区域到另一个，并使用[Azure Site Recovery](site-recovery-overview.md)服务。


## <a name="deployment-method-support"></a>部署方法支持

**部署** |  **支持**
--- | ---
**Azure 门户** | 。
**PowerShell** | 。 [了解详细信息](azure-to-azure-powershell.md)
**REST API** | 。
**CLI** | 目前不支持


## <a name="resource-support"></a>资源支持

**资源操作** | **详细信息**
--- | --- | ---
**跨资源组移动保管库** | 不支持
**跨资源组移动计算/存储/网络资源** | 不支持。<br/><br/> 如果在 VM 复制后移动 VM 或相关组件（如存储/网络），则需要禁用并重新启用 VM 的复制。
**将 Azure VM 从一个订阅复制到另一个订阅以进行灾难恢复** | 在同一 Azure Active Directory 租户中受支持。
**在受支持的地理群集内跨区域迁移虚拟机（订阅内和跨订阅）** | 在同一 Azure Active Directory 租户中受支持。
**在同一区域内迁移 VM** | 不支持。

## <a name="region-support"></a>区域支持

可以在同一地理群集中的任意两个区域之间复制和恢复 VM。 在定义地理群集时请注意数据延迟和主权。


**地理群集** | **Azure 区域**
-- | --
美洲 | 加拿大东部、加拿大中部、美国中南部、美国中西部、美国东部、美国东部 2、美国西部、美国西部 2、美国中部、美国中北部
欧洲 | 英国西部、 英国南部、 北欧、 西欧、 法国中部、 法国南部、 南非西部、 南非北部
亚洲 | 印度南部、印度中部、东南亚、东亚、日本东部、日本西部、韩国中部、韩国南部
澳大利亚   | 澳大利亚东部、澳大利亚东南部、澳大利亚中部、澳大利亚中部 2
Azure 政府    | US Gov 弗吉尼亚州、US Gov 爱荷华州、US Gov 亚利桑那州、US Gov 德克萨斯州、US DOD 东部、US DOD 中部
德国 | 德国中部、德国东北部
中国 | 中国东部、中国北部、中国北部 2、中国东部 2

>[!NOTE]
>
> - 有关**巴西南部**，可以进行复制和故障转移到这些区域：美国中南部、 美国中西部、 美国东部、 美国东部 2、 美国西部、 美国西部 2 和美国中北部。
> - 巴西南部目前仅可用作 Vm 可以从其复制使用 Site Recovery 的源区域。 它不能充当目标区域。 这是因为延迟问题，因为地理距离。
> - 您可以对其具有适当的访问权限的区域中。
> - 如果您要在其中创建保管库的区域中未显示，请确保你的订阅具有在该区域中创建资源的访问权限。
> - 如果在启用复制时，您无法查看地理群集内的区域，，请确保你的订阅有权在该区域中创建 Vm。



## <a name="cache-storage"></a>缓存存储

此表汇总了对复制期间 Site Recovery 使用的缓存存储帐户的支持。

**设置** | **支持** | **详细信息**
--- | --- | ---
常规用途 V2 存储帐户（热存储层和冷存储层） | 不支持。 | 因为 V2 的事务成本远高于 V1 存储帐户，因此缓存存储存在限制。
虚拟网络的 Azure 存储防火墙  | 支持 | 如果你使用的是启用了防火墙的缓存存储帐户或目标存储帐户，请确保“[允许受信任的 Microsoft 服务](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)”。


## <a name="replicated-machine-operating-systems"></a>复制的计算机操作系统

Site Recovery 支持复制那些运行本节中所列操作系统的 Azure VM。

### <a name="windows"></a>Windows

**操作系统** | **详细信息**
--- | ---
Windows Server 2019 |
Windows Server 2016  | 服务器核心、带桌面体验的服务器
Windows Server 2012 R2 |
Windows Server 2012 |
Windows Server 2008 R2 | 运行 SP1 或更高版本

#### <a name="linux"></a>Linux

**操作系统** | **详细信息**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6  
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6
Ubuntu 14.04 LTS Server | [受支持的内核版本](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Ubuntu 16.04 LTS Server | [受支持的内核版本](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Ubuntu 服务器如果使用基于密码的身份验证和登录、 和的 cloud-init 包配置云虚拟机，可能会基于密码的登录中上 （具体取决于 cloudinit 配置） 的故障转移的已禁用。 基于密码的登录中可以通过从支持的密码重置虚拟机上重新启用 > 故障排除 > 设置菜单 （在故障转移的 VM 在 Azure 门户中。
Debian 7 | [受支持的内核版本](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | [受支持的内核版本](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1、 SP2、 SP3、 SP4。 [（受支持的内核版本）](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | SP3<br/><br/> 不支持将复制计算机从 SP3 升级到 SP4。 如果已升级复制的计算机，则需要禁用复制并在升级后重新启用复制。
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5 <br/><br/> 运行与 Red Hat 兼容的内核或 Unbreakable Enterprise Kernel Release 3 (UEK3)。


#### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Azure 虚拟机支持的 Ubuntu 内核版本

**版本** | **移动服务版本** | **内核版本** |
--- | --- | --- |
14.04 LTS | 9.24 | 3.13.0-24-generic 到 3.13.0-167-generic，<br/>3.16.0-25-generic 到 3.16.0-77-generic、<br/>3.19.0-18-generic 到 3.19.0-80-generic、<br/>4.2.0-18-generic 到 4.2.0-42-generic、<br/>4.4.0-21-generic 到 4.4.0-143-generic，<br/>到 4.15.0-1040-azure 4.15.0-1023-azure |
14.04 LTS | 9.23 | 3.13.0-24-generic 到 3.13.0-165-generic，<br/>3.16.0-25-generic 到 3.16.0-77-generic、<br/>3.19.0-18-generic 到 3.19.0-80-generic、<br/>4.2.0-18-generic 到 4.2.0-42-generic、<br/>4.4.0-21-generic 到 4.4.0-142-generic，<br/>到 4.15.0-1037-azure 4.15.0-1023-azure |
14.04 LTS | 9.22 | 3.13.0-24-generic 到 3.13.0-164-generic、<br/>3.16.0-25-generic 到 3.16.0-77-generic、<br/>3.19.0-18-generic 到 3.19.0-80-generic、<br/>4.2.0-18-generic 到 4.2.0-42-generic、<br/>4.4.0-21-generic 到 4.4.0-140-generic、<br/>4.15.0-1023-azure 到 4.15.0-1036-azure |
14.04 LTS | 9.21 | 3.13.0-24-generic 到 3.13.0-163-generic、<br/>3.16.0-25-generic 到 3.16.0-77-generic、<br/>3.19.0-18-generic 到 3.19.0-80-generic、<br/>4.2.0-18-generic 到 4.2.0-42-generic、<br/>4.4.0-21-generic 到 4.4.0-140-generic、<br/>4.15.0-1023-azure 到 4.15.0-1035-azure |
|||
16.04 LTS | 9.24 | 4.4.0-21-generic 到 4.4.0-143-generic，<br/>4.8.0-34-generic 到 4.8.0-58-generic、<br/>4.10.0-14-generic 到 4.10.0-42-generic、<br/>4.11.0-13-generic 到 4.11.0-14-generic、<br/>4.13.0-16-generic 到 4.13.0-45-generic、<br/>到 4.15.0-46-generic 4.15.0-13-generic<br/>4.11.0-1009-azure 到 4.11.0-1018-azure，<br/>4.13.0-1005-azure 到 4.13.0-1018-azure <br/>到 4.15.0-1040-azure 4.15.0-1012-azure|
16.04 LTS | 9.23 | 4.4.0-21-generic 到 4.4.0-142-generic，<br/>4.8.0-34-generic 到 4.8.0-58-generic、<br/>4.10.0-14-generic 到 4.10.0-42-generic、<br/>4.11.0-13-generic 到 4.11.0-14-generic、<br/>4.13.0-16-generic 到 4.13.0-45-generic、<br/>到 4.15.0-45-generic 4.15.0-13-generic<br/>4.11.0-1009-azure 到 4.11.0-1016-azure、<br/>4.13.0-1005-azure 到 4.13.0-1018-azure <br/>到 4.15.0-1037-azure 4.15.0-1012-azure|
16.04 LTS | 9.22 | 4.4.0-21-generic 到 4.4.0-140-generic、<br/>4.8.0-34-generic 到 4.8.0-58-generic、<br/>4.10.0-14-generic 到 4.10.0-42-generic、<br/>4.11.0-13-generic 到 4.11.0-14-generic、<br/>4.13.0-16-generic 到 4.13.0-45-generic、<br/>4.15.0-13-generic 到 4.15.0-43-generic<br/>4.11.0-1009-azure 到 4.11.0-1016-azure、<br/>4.13.0-1005-azure 到 4.13.0-1018-azure <br/>4.15.0-1012-azure 到 4.15.0-1036-azure|
16.04 LTS | 9.21 | 4.4.0-21-generic 到 4.4.0-140-generic、<br/>4.8.0-34-generic 到 4.8.0-58-generic、<br/>4.10.0-14-generic 到 4.10.0-42-generic、<br/>4.11.0-13-generic 到 4.11.0-14-generic、<br/>4.13.0-16-generic 到 4.13.0-45-generic、<br/>4.15.0-13-generic 到 4.15.0-42-generic<br/>4.11.0-1009-azure 到 4.11.0-1016-azure、<br/>4.13.0-1005-azure 到 4.13.0-1018-azure <br/>4.15.0-1012-azure 到 4.15.0-1035-azure|

#### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Azure 虚拟机支持的 Debian 内核版本

**版本** | **移动服务版本** | **内核版本** |
--- | --- | --- |
Debian 7 | 9.21,9.22,9.23,9.24 | 3.2.0-4-amd64 到 3.2.0-6-amd64、3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | 9.21, 9.22, 9.23, 9.24 | 3.16.0-4-amd64 到 3.16.0-7-amd64、4.9.0-0.bpo.4-amd64 到 4.9.0-0.bpo.8-amd64 |

#### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Azure 虚拟机支持的 SUSE Linux Enterprise Server 12 内核版本

**版本** | **移动服务版本** | **内核版本** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 （SP1、 SP2、 SP3、 SP4） | 9.24 | SP1 3.12.49-11-default 到 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 到 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default 到 4.4.120-92.70-default</br></br>到 4.4.121-92.101-default SP2(LTSS) 4.4.121-92.73-default</br></br>到 4.4.175-94.79-default SP3 4.4.73-5-default</br></br>到 4.12.14-95.6-default SP4 4.12.14-94.41-default |
SUSE Linux Enterprise Server 12 （SP1、 SP2、 SP3、 SP4） | 9.23 | SP1 3.12.49-11-default 到 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 到 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default 到 4.4.120-92.70-default</br></br>到 4.4.121-92.101-default SP2(LTSS) 4.4.121-92.73-default</br></br>SP3 4.4.73-5-default 到 4.4.162-94.69-default</br></br>到 4.12.14-95.6-default SP4 4.12.14-94.41-default |
SUSE Linux Enterprise Server 12 (SP1、 SP2、 SP3) | 9.22 | SP1 3.12.49-11-default 到 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 到 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default 到 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default 到 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default 到 4.4.162-94.72-default |
SUSE Linux Enterprise Server 12（SP1、SP2、SP3） | 9.21 | SP1 3.12.49-11-default 到 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 到 3.12.74-60.64.107-default</br></br> SP2 4.4.21-69-default 到 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default 到 4.4.121-92.98-default</br></br>SP3 4.4.73-5-default 到 4.4.162-94.72-default |

## <a name="replicated-machines---linux-file-systemguest-storage"></a>复制的计算机 - Linux 文件系统/来宾存储

* 文件系统：ext3、ext4、ReiserFS（仅限 Suse Linux Enterprise Server）和 XFS、BTRFS
* 卷管理器：LVM2
* 多路径软件：设备映射器


## <a name="replicated-machines---compute-settings"></a>复制的计算机 - 计算设置

**设置** | **支持** | **详细信息**
--- | --- | ---
大小 | 具有至少 2 个 CPU 内核和 1 GB RAM 的任意 Azure VM 大小 | 验证 [Azure 虚拟机大小](../virtual-machines/windows/sizes.md)。
可用性集 | 支持 | 如果你启用复制 Azure VM 中使用的默认选项，可用性集根据源区域设置自动创建。 可以修改这些设置。
可用性区域 | 支持 |
混合使用权益 (HUB) | 支持 | 如果源 VM 启用了 HUB 许可证，则测试故障转移或故障转移 VM 也使用 HUB 许可证。
VM 规模集 | 不支持 |
Azure 库映像 — 由 Microsoft 发布 | 支持 | 如果 VM 在受支持的操作系统上运行，则支持该配置。
Azure 库映像 - 由第三方发布 | 支持 | 如果 VM 在受支持的操作系统上运行，则支持该配置。
自定义映像 - 由第三方发布 | 支持 | 如果 VM 在受支持的操作系统上运行，则支持该配置。
使用 Site Recovery 迁移的 VM | 支持 | 如果使用 Site Recovery 将 VMware VM 或物理计算机迁移到 Azure，则需要卸载计算机上运行的旧版移动服务，并在重启计算机后将该计算机复制到另一个 Azure 区域。
RBAC 策略 | 不支持 | 基于角色的访问控制 (RBAC) 策略在 Vm 上的不会复制到目标区域中的 VM 故障转移。
扩展 | 不支持 | 扩展不复制到目标区域中的 VM 故障转移。 它需要故障转移后手动安装。

## <a name="replicated-machines---disk-actions"></a>复制的计算机 - 磁盘操作

**Action** | **详细信息**
-- | ---
调整复制的 VM 上的磁盘大小 | 支持
将磁盘添加到复制的 VM | 不支持。<br/><br/> 需要为 VM 禁用复制，添加磁盘，然后再次启用复制。

## <a name="replicated-machines---storage"></a>复制的计算机 - 存储

此表汇总了对 Azure VM OS 磁盘、数据磁盘和临时磁盘的支持。

- 请务必遵循适用于 [Linux](../virtual-machines/linux/disk-scalability-targets.md) 和 [Windows](../virtual-machines/windows/disk-scalability-targets.md) VM 的 VM 磁盘限制以及目标，以避免任何性能问题。
- 如果使用默认设置进行部署，Site Recovery 会根据源设置自动创建磁盘和存储帐户。
- 如果自定义，请确保遵循指南。

组件 | **支持** | **详细信息**
--- | --- | ---
OS 磁盘的最大大小 | 2048 GB | [深入了解 ](../virtual-machines/windows/managed-disks-overview.md)VM 磁盘相关信息。
临时磁盘 | 不支持 | 始终从复制中排除临时磁盘。<br/><br/> 请勿在临时磁盘上存储任何持久性数据。 [了解详细信息](../virtual-machines/windows/managed-disks-overview.md)。
数据磁盘的最大大小 | 4095 GB |
数据磁盘的最大数量 | 最多为 64，根据对特定的 Azure VM 大小的支持而定 | [深入了解 ](../virtual-machines/windows/sizes.md)VM 大小相关信息。
数据磁盘更改率 | 每个高级存储的磁盘最大为 10 MBps。 每个标准存储的磁盘最大为 2 MBps。 | 如果磁盘上的平均数据更改率持续高于最大值，复制将跟不上。<br/><br/>  但是，如果偶尔超出最大值，则复制可跟上，但可能会看到稍有延迟的恢复点。
数据磁盘 - 标准存储帐户 | 支持 |
数据磁盘 - 高级存储帐户 | 支持 | 如果 VM 将磁盘分散在高级和标准存储帐户上，则可以为每个磁盘选择不同的目标存储帐户，以确保在目标区域中具有相同的存储配置。
托管磁盘 - 标准 | 在支持 Azure Site Recovery 的 Azure 区域中受支持。 |
托管磁盘 - 高级 | 在支持 Azure Site Recovery 的 Azure 区域中受支持。 |
标准 SSD | 支持 |
冗余 | LRS 和 GRS 受支持。<br/><br/> ZRS 不受支持。
冷存储和热存储 | 不支持 | 冷存储和热存储不支持 VM 磁盘
存储空间 | 支持 |
静态加密 (SSE) | 支持 | SSE 是存储帐户的默认设置。   
适用于 Windows OS 的 Azure 磁盘加密 (ADE) | 支持为[使用 Azure AD 应用的加密](https://aka.ms/ade-aad-app)启用的 VM |
适用于 Linux OS 的 Azure 磁盘加密 (ADE) | 不支持 |
热添加 | 支持 | 启用复制的数据磁盘添加到复制的 Azure VM 使用托管的磁盘的 Vm 支持。
热删除磁盘 | 不支持 | 如果删除 VM 上的数据磁盘，需要禁用复制并再次为 VM 启用复制。
排除磁盘 | 支持。 必须使用[Powershell](azure-to-azure-exclude-disks.md)配置。 |  默认情况下排除临时磁盘。
存储空间直通  | 崩溃一致恢复点支持。 不支持应用程序一致恢复点。 |
横向扩展文件服务器  | 崩溃一致恢复点支持。 不支持应用程序一致恢复点。 |
LRS | 支持 |
GRS | 支持 |
RA-GRS | 支持 |
ZRS | 不支持 |
冷存储和热存储 | 不支持 | 冷存储和热存储不支持虚拟机磁盘
虚拟网络的 Azure 存储防火墙  | 支持 | 如果将虚拟网络访问限制到存储帐户，请启用[允许受信任的 Microsoft 服务](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)。
常规用途 V2 存储帐户（冷热存储层） | “否” | 与常规用途 V1 存储帐户相比，事务成本显著增加

>[!IMPORTANT]
> 若要避免性能问题，请确保遵循的 VM 磁盘的可伸缩性和性能目标[Linux](../virtual-machines/linux/disk-scalability-targets.md)或[Windows](../virtual-machines/windows/disk-scalability-targets.md) Vm。 如果使用默认设置，Site Recovery 会创建所需的磁盘和存储帐户，基于源配置。 如果自定义和选择自己的设置，请遵循源 Vm 的磁盘可伸缩性和性能目标。

## <a name="limits-and-data-change-rates"></a>限制和数据更改率

下表汇总了 Site Recovery 限制。

- 这些限制基于我们的测试，但显然不涵盖所有可能的应用程序 I/O 组合。
- 实际结果可以根据应用程序 I/O 组合而有所不同。
- 有两个需要考虑每个磁盘数据变动量和每个虚拟机数据变动量的限制。
- 例如，如果我们使用高级 P20 磁盘下，表中所述 Site Recovery 可以处理 5 Mb 的每个磁盘，改动与最多五个此类磁盘，每个 VM 由于 25 MB/秒总变动量的每个 VM 的限制。

**存储目标** | **平均源磁盘 I/O** |**平均源磁盘数据变动量** | **每天的总源磁盘数据变动量**
---|---|---|---
标准存储 | 8 KB | 2 MB/秒 | 每个磁盘 168 GB
高级 P10 或 P15 磁盘 | 8 KB  | 2 MB/秒 | 每个磁盘 168 GB
高级 P10 或 P15 磁盘 | 16 KB | 4 MB/秒 |  每个磁盘 336 GB
高级 P10 或 P15 磁盘 | 至少 32 KB | 8 MB/秒 | 每个磁盘 672 GB
高级 P20、P30、P40 或 P50 磁盘 | 8 KB    | 5 MB/秒 | 每个磁盘 421 GB
高级 P20、P30、P40 或 P50 磁盘 | 至少 16 KB |20 MB/秒 | 每个磁盘 1684 GB

## <a name="replicated-machines---networking"></a>复制的计算机 - 网络
**设置** | **支持** | **详细信息**
--- | --- | ---
NIC | 特定 Azure VM 大小支持的最大数量 | 在故障转移期间创建 VM 时会创建 NIC。<br/><br/> 故障转移 VM 上的 NIC 数目取决于启用复制时源 VM 上的 NIC 数目。 如果在启用复制后添加或删除 NIC，不会影响故障转移后复制的 VM 上的 NIC 数目。 此外请注意，故障转移后的 Nic 顺序并不保证是原始顺序相同。
内部负载均衡器 | 支持 | 在恢复计划中使用 Azure 自动化脚本关联预配置的负载均衡器。
内部负载均衡器 | 支持 | 在恢复计划中使用 Azure 自动化脚本关联预配置的负载均衡器。
公用 IP 地址 | 支持 | 将现有的公共 IP 地址与 NIC 关联。 或者，在恢复计划中使用 Azure 自动化脚本创建公共 IP 地址并将其与 NIC 关联。
NIC 上的 NSG | 支持 | 在恢复计划中使用 Azure 自动化脚本将 NSG 与 NIC 关联。
子网上的 NSG | 支持 | 在恢复计划中使用 Azure 自动化脚本将 NSG 与子网关联。
保留（静态）IP 地址 | 支持 | 如果源 VM 上的 NIC 具有静态 IP 地址，并且目标子网具有相同的可用 IP 地址，则会将它分配给故障转移 VM。<br/><br/> 如果目标子网没有相同的可用 IP 地址，则为 VM 保留子网中的某个可用 IP 地址。<br/><br/> 此外可以在“复制的项” > “设置” > “计算和网络” > “网络接口”中指定固定的 IP 地址和子网。
动态 IP 地址 | 支持 | 如果源上的 NIC 具有动态 IP 地址，故障转移 VM 上的 NIC 也默认为动态。<br/><br/> 如果有需要，可以将其修改为固定的 IP 地址。
多个 IP 地址 | 不支持 | 当故障转移了多个 IP 地址的 NIC 的 VM 时，保留只能的主要 IP 地址的源区域中的 NIC。 若要分配多个 IP 地址，你可以添加到 Vm[恢复计划](recovery-plan-overview.md)并附加一个脚本来将其他 IP 地址分配给计划，或您可以进行更改手动或使用脚本故障转移后。 
流量管理器     | 支持 | 可以预配置流量管理器，这样在正常情况下，流量路由到源区域中的终结点；发生故障转移时，流量路由到目标区域中的终结点。
Azure DNS | 支持 |
自定义 DNS  | 支持 |
未经身份验证的代理 | 支持 | [了解详细信息]。(site-recovery-azure-to-azure-networking-guidance.md)   
经过身份验证的代理 | 不支持 | 如果 VM 对出站连接使用经过身份验证的代理，则不能使用 Azure Site Recovery 复制该 VM。    
VPN 站点到站点连接到本地<br/><br/>（带或不带 ExpressRoute）| 支持 | 请确保 Site Recovery 流量不路由到本地的这种方式配置 Udr 和 Nsg。 [了解详细信息](site-recovery-azure-to-azure-networking-guidance.md)    
VNET 到 VNET 连接 | 支持 | [了解详细信息](site-recovery-azure-to-azure-networking-guidance.md)  
虚拟网络服务终结点 | 支持 | 若要限制对存储帐户的虚拟网络访问，请确保允许受信任的 Microsoft 服务访问存储帐户。
更快的网络连接 | 支持 | 必须在源 VM 上启用加速网络。 [了解详细信息](azure-vm-disaster-recovery-with-accelerated-networking.md)。



## <a name="next-steps"></a>后续步骤
- 读取[网络连接指南](site-recovery-azure-to-azure-networking-guidance.md)Azure Vm 复制。
- 通过[复制 Azure VM](site-recovery-azure-to-azure.md) 来部署灾难恢复。
