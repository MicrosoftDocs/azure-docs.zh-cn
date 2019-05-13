---
title: 将数据从 StorSimple 5000-7000 系列迁移到 Azure 文件同步 | Microsoft Docs
description: 介绍如何将数据从 StorSimple 5000/7000 系列迁移到 Azure 文件同步 (AFS)。
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/19/2019
ms.author: alkohli
ms.openlocfilehash: b46e9ee8fc3e14981a01cc2425a8ce55d06c5a9a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150740"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-azure-file-sync"></a>将数据从 StorSimple 5000-7000 系列迁移到 Azure 文件同步

> [!IMPORTANT]
> 在 2019 年 7 月 9 日 StorSimple 5000/7000 系列将到达结束 (EOS) 的支持状态。 我们建议 StorSimple 5000/7000 系列客户迁移到本文档中所述的替代方案之一。

数据迁移是将数据从一个存储位置转移到另一个位置的过程。 这需要在组织中的一台设备上，创建另一台设备上的当前数据的完全相同的副本 — 最好能够避免中断或禁用活动的应用程序 — 然后将所有输入/输出 (I/O) 活动重定向到新设备。 

StorSimple 5000 和 7000 系列存储设备将在 2019 年 7 月退役。 这意味着，在 2019 年 7 月以后，Microsoft 不再能够支持 StorSimple 5000/7000 系列的硬件和软件。 使用这些设备的客户应将其 StorSimple 数据迁移到 Azure 上的其他混合存储解决方案。 本文介绍如何将数据从 StorSimple 5000/7000 系列设备迁移到 Azure 文件同步 (AFS)。

## <a name="intended-audience"></a>目标受众

本文面向负责在数据中心部署和管理 StorSimple 5000/7000 系列设备的信息技术 (IT) 专业人员和知识工作者。 对文件服务器工作负荷（包含 Windows Server）使用 StorSimple 设备的客户可能发现此迁移路径非常完美。 如果你觉得 Azure 文件同步的功能非常适合自己的组织，则本文可以帮助你了解如何从 StorSimple 转移到这些解决方案。

## <a name="migration-considerations"></a>迁移注意事项

此过程适用于已使用 StorSimple 卷作为存储配置了 Windows 文件共享的客户。 将数据从 StorSimple 5000/7000 迁移到 Azure 文件同步的过程涉及到：将该文件共享位置转换为[服务器终结点](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning)，然后使用本地附加的驱动器作为另一个终结点（该终结点随后会变成新位置）。 

转移到 AFS 时，应注意以下几点：

1. Azure 文件目前限制为每个共享 5 TB。 使用 Azure 文件同步将数据分散到多个 Azure 文件共享即可摆脱此限制。 详细信息，请查看 [Azure 文件部署的数据增长模式](https://docs.microsoft.com/azure/storage/files/storage-files-planning)。
2. 完成从本地设备的数据复制后，此迁移会将整个主要数据集下载到本地设备。 请确保有足够的带宽来完成此传输。
3. 此过程不会保留已创建的快照， 而只会迁移主要数据。 此过程也不会保留关联的带宽模板或备份策略。 在 Azure 文件共享中迁移数据后，[使用 Azure 备份](https://docs.microsoft.com/azure/backup/backup-azure-files)设置备份策略。
4. StorSimple 提供第一方硬件。 但是，借助 Azure 文件/Azure 文件同步，可将自己的本地 Windows Server 硬件用作本地缓存。 请务必提供足够的存储容量，以便在本地保存所选的数据集。 有关分层和设置必备可用空间目标的详细信息，请查看[在部署 Azure 文件同步时创建服务器终结点](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal)。 
5. 查看 [Azure 文件同步的定价](https://azure.microsoft.com/pricing/details/storage/files/)，因为它与 StorSimple 不同。 AFS 不像 StorSimple 那样提供重复数据删除和压缩功能。

## <a name="migration-prerequisites"></a>迁移先决条件

本部分提供将旧式 5000 或 7000 系列设备迁移到 Azure 文件同步所要满足的先决条件。在开始之前，请确保：

- 有权访问运行 v2.1.1.518 或更高软件版本的 StorSimple 5000/7000 系列设备。 不支持更早的版本。 StorSimple 设备 Web UI 的右上角应显示正在运行的软件版本。  
    如果设备不是运行 v2.1.1.518，请将系统升级到要求的最低版本。 有关详细说明，请参阅[将系统升级到 v2.1.1.518](http://onlinehelp.storsimple.com/111_Appliance/6_System_Upgrade_Guides/Current_(v2.1.1)/000_Software_Patch_Upgrade_Guide_v2.1.1.518)。
- 检查源设备上是否有任何活动的备份作业在运行。 这包括在 StorSimple 数据保护控制台主机上的作业。 等待当前作业完成。 
- 访问已连接到 StorSimple 5000-7000 系列设备的 Windows Server 主机。 该主机必须运行 [Azure 文件同步互操作性](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning)中所述的受支持 Windows Server 版本。
- StorSimple 卷已装载在主机上，并包含文件共享。
- 该主机具有足够的本地存储，可以保存本地缓存的数据。
- 对用于部署 Azure 文件同步的 Azure 订阅拥有所有者级别的访问权限。如果没有所有者或管理员级别的权限，则为同步组创建云终结点时可能会遇到问题。
- 有权访问包含所要同步到的 Azure 文件共享的[常规用途 v2 存储帐户](https://docs.microsoft.com/azure/storage/common/storage-account-overview)。 有关详细信息，请参阅[创建存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)。
  - 如何[创建 Azure 文件共享](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share)。

## <a name="migration-process"></a>迁移过程

将数据从 StorSimple 迁移到 AFS 5000-7000 的过程包括两个步骤：
1.  从本地文件服务器（其中的 StorSimple 卷已装载到 Azure 文件共享）复制数据。  复制通过安装的 AFS 代理完成。
2.  断开连接 StorSimple 设备。 然后，本地磁盘将充当本地缓存。

### <a name="migration-steps"></a>迁移步骤

执行以下步骤，将 StorSimple 卷上配置的 Windows 文件共享迁移到 Azure 文件同步共享。 
1.  在同一台 Windows Server 主机（其中的 StorSimple 卷已装载或使用不同的系统）上执行这些步骤。 
    - [准备 Windows Server 用于 Azure 文件同步](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#prepare-windows-server-to-use-with-azure-file-sync)。
    - [安装 Azure 文件同步代理](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#install-the-azure-file-sync-agent)。
    - [部署存储同步服务](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#deploy-the-storage-sync-service)。 
    - [将 Windows Server 注册到存储同步服务](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#register-windows-server-with-storage-sync-service)。 
    - [创建同步组和云终结点](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#create-a-sync-group-and-a-cloud-endpoint)。 对于需要从主机迁移的每个 Windows 文件共享，请创建同步组。
    - [创建服务器终结点](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal#create-a-server-endpoint)。 将路径指定为包含文件共享数据的 StorSimple 卷的路径。 例如，如果 StorSimple 卷是驱动器 `J`，而数据驻留在 `J:/<myafsshare>` 中，则将此路径添加为服务器终结点。 将“分层”保留为“已禁用”。
2.  等到文件服务器同步完成。 对于给定同步组中的每个服务器，请确保：
    - 上传和下载会话的“上次尝试同步”时间戳是最近的时间。
    - 上传和下载会话的状态为绿色。
    - “同步活动”显示的要同步的剩余文件数很少或者为零。
    - 上传和下载会话的“未同步的文件数”值为 0。
    有关服务器同步何时完成的详细信息，请转到[排查 Azure 文件同步的问题](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#how-do-i-know-if-my-servers-are-in-sync-with-each-other)。同步所需的时间为几个小时到几天不等，具体取决于数据大小和带宽。 同步完成后，所有数据将安全保存在 Azure 文件共享中。 
3.  转到 StorSimple 卷上的共享。 选择一个共享，单击右键，然后选择“属性”。 注意“安全性”下面的共享权限。 在稍后的步骤中，需将这些权限手动应用到新共享。
4.  根据使用的是相同还是不同的 Windows Server 主机，后续步骤会有所不同。

    如果使用不同的 Windows Server 主机，请跳过此步骤并转到下一步骤。 如果对 AFS 使用相同的 Windows 文件服务器，则现在会出现几分钟的停机时间。 
    - **停机时间开始** - 删除在“步骤 1F”中创建的服务器终结点。 
    - 使用数据接下来要驻留到的路径创建新的服务器终结点。
    - 当服务器终结点显示为“正常”（这可能需要几分钟时间）后，新位置中会出现这些数据。 现在，可将 Windows Server 主机配置为从此新位置提供文件。 - **停机时间结束**。
5.  如果对 Azure 文件同步使用另一个 Windows 文件服务器，则不会出现任何停机时间。 
    - 使用准备用作缓存的本地存储的路径添加另一个服务器终结点，以代替 StorSimple 设备。 
    - 片刻之后，即可在新服务器中看到文件。 随时可以在主机上从 StorSimple 设备切换到此新位置。

    > [!TIP] 
    > 请考虑使用此新文件共享所代替的设备的相同名称和路径来配置此共享，以尽量减少中断时间。 如果使用 DFS-N，可能需要在配置中进行更改。
6.  根据“步骤 3”中所述重新配置共享权限。

如果在数据迁移期间遇到任何问题，请[联系 Microsoft 支持部门](storsimple-8000-contact-microsoft-support.md)。 



## <a name="next-steps"></a>后续步骤

如果 AFS 不适合你的解决方案，请了解如何[将数据从 StorSimple 5000-7000 系列迁移到 8000 系列设备](storsimple-8000-migrate-from-5000-7000.md)。

