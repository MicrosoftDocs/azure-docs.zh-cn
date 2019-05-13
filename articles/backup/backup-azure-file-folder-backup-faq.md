---
title: 备份文件和使用 Azure 备份的文件夹时的常见问题
description: 解决了有关备份文件和文件夹使用 Azure 备份常见问题的解决方案。
author: dcurwin
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: dacurwin
ms.openlocfilehash: 5dbd4fefd5c5e1acd7e12ace547ddb8866b7f081
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148592"
---
# <a name="common-questions-about-backing-up-files-and-folders"></a>有关备份文件和文件夹的常见问题 

这篇文章提供解答常见问题出现了备份文件和文件夹中的 Microsoft Azure 恢复服务 (MARS) 代理[Azure 备份](backup-overview.md)服务。

## <a name="general"></a>常规

### <a name="why-does-the-mars-agent-need-net-framework-452-or-higher"></a>MARS 代理为什么需要.NET framework 4.5.2 或更高版本？

中提供的新功能[即时还原](backup-azure-restore-windows-server.md#use-instant-restore-to-recover-data-to-the-same-machine)需要.NET Framework 4.5.2 或更高版本。

## <a name="configure-backups"></a>配置备份

### <a name="where-can-i-download-the-latest-version-of-the-mars-agent"></a>其中可以下载最新版本的 MARS 代理？ 
最新的 MARS 代理备份 Windows Server 计算机、 System Center DPM 和 Microsoft Azure 备份服务器时使用，可用于[下载](https://aka.ms/azurebackup_agent)。 

### <a name="how-long-are-vault-credentials-valid"></a>有效的保管库凭据是多长时间？
保管库凭据会在 48 小时后过期。 如果凭据文件过期，请再次下载文件从 Azure 门户。

### <a name="from-what-drives-can-i-back-up-files-and-folders"></a>从哪些驱动器可以备份文件和文件夹？ 

无法备份以下类型的驱动器和卷：

* 可移动介质：所有备份项源必须报告为“固定”。
* 只读卷：为使卷影复制服务 (VSS) 起作用，卷必须是可写的。
* 脱机卷：为使 VSS 起作用，卷必须是联机的。
* 网络共享：卷必须位于需要使用联机备份进行备份的服务器本地。
* BitLocker 保护的卷：必须先解锁卷，才能备份。
* 文件系统标识：NTFS 是受支持的唯一文件系统。

### <a name="what-file-and-folder-types-are-supported"></a>支持哪些文件和文件夹类型？

支持以下类型：

* 加密
* 压缩
* 稀疏
* 压缩 + 稀疏
* 硬链接：不支持，跳过
* 重分析点：不支持，跳过
* 加密 + 稀疏：不支持，跳过
* 压缩流：不支持，跳过
* 重新分析点，包括 DFS 链接和交接点


### <a name="can-i-use-the-mars-agent-to-back-up-files-and-folders-on-an-azure-vm"></a>可以使用 MARS 代理以备份 Azure VM 上的文件和文件夹？  
可以。 Azure 备份可以使用适用于 Azure VM 代理 VM 扩展的 Azure Vm 提供 VM 级别的备份。 如果你想要备份文件和文件夹在 VM 上的来宾 Windows 操作系统上，可以安装 MARS 代理要执行此操作。 

### <a name="can-i-use-the-mars-agent-to-back-up-files-and-folders-on-temporary-storage-for-the-azure-vm"></a>可以使用 MARS 代理以备份文件和文件夹上的临时存储的 Azure vm？ 
可以。 安装 MARS 代理，并将备份文件和文件夹在来宾 Windows 操作系统上，对临时存储。 -备份作业失败有关的数据擦除临时存储。
- 如果删除临时存储数据，则可以仅还原到非易失性存储。

### <a name="how-do-i-register-a-server-to-another-region"></a>如何注册到另一个区域的服务器？

备份数据发送到在其中注册服务器的保管库的数据中心。 更改数据中心的最简单方法是卸载并重新安装代理，，然后将计算机注册到新的保管库中所需的区域

### <a name="does-the-mars-agent-support-windows-server-2012-deduplication"></a>MARS 代理支持 Windows Server 2012 重复数据删除？
可以。 准备备份操作时，MARS 代理转换为常规数据已删除重复的数据。 然后，再来优化备份的数据、 加密数据，并将此加密的数据发送到保管库。

## <a name="manage-backups"></a>管理备份

### <a name="what-happens-if-i-rename-a-windows-machine-configured-for-backup"></a>如果我重命名为备份配置的 Windows 计算机，则会发生什么情况？

Windows 计算机进行重命名，则会停止所有当前配置的备份。

- 您需要向备份保管库注册新的计算机名称。
- 注册到保管库的新名称，第一个操作时，*完整*备份。
- 如果需要恢复备份到采用旧服务器名称的保管库数据，使用选项将还原到备用位置恢复数据向导中。 [了解详细信息](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine)。 

### <a name="what-is-the-maximum-file-path-length-for-backup"></a>什么是备份的最大文件路径长度？
MARS 代理依赖于 NTFS，并使用文件路径长度规范受限于[Windows API](/windows/desktop/FileIO/naming-a-file#fully_qualified_vs._relative_paths)。 如果想要保护的文件超过允许的值，则备份父文件夹或磁盘驱动器。  

### <a name="what-characters-are-allowed-in-file-paths"></a>在文件路径中允许哪些字符？

MARS 代理依赖于 NTFS，并允许[受支持的字符](/windows/desktop/FileIO/naming-a-file#naming_conventions)中文件的名称/路径。

### <a name="the-warning-azure-backups-have-not-been-configured-for-this-server-appears"></a>在"Azure 备份具有尚未配置此服务器"会出现警告。
即使在本地服务器上存储的备份计划设置不是存储在备份保管库中的设置相同时，已配置备份策略，可以显示此警告。
- 当服务器或设置具有已恢复到已知的良好状态时，备份计划可能会变得不同步。
- 如果你收到此警告[配置](backup-azure-manage-windows-server.md)同样，备份策略，然后运行按需备份本地服务器与 Azure 重新同步。


## <a name="manage-the-backup-cache-folder"></a>管理备份的缓存文件夹

### <a name="whats-the-minimum-size-requirement-for-the-cache-folder"></a>针对缓存文件夹的最小大小要求是什么？
缓存文件夹的大小由你正在备份的数据量确定。
- 缓存文件夹的卷应具有等于至少 5-10%的备份数据的总大小的可用空间。
- 如果卷上有可用空间低于 5%，增加卷的大小，或将缓存文件夹移动到具有足够空间的卷。
- 
### <a name="how-do-i-change-the-cache-location-for-the-mars-agent"></a>如何更改 MARS 代理的缓存位置？


1. 在提升的命令提示符停止备份引擎中运行以下命令：

    ```PS C:\> Net stop obengine```

2. 不将文件移动。 相反，将缓存空间文件夹复制到另一个驱动器具有足够的空间。
3. 使用新的缓存文件夹的路径更新以下注册表项。<br/>

    | 注册表路径 | 注册表项 | 值 |
    | --- | --- | --- |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config` |ScratchLocation |*新缓存文件夹位置* |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider` |ScratchLocation |*新缓存文件夹位置* |

4. 重启备份引擎在提升的命令提示符：

    ```PS C:\> Net start obengine```

5. 使用新位置成功完成备份后，可以删除原始缓存文件夹。


### <a name="where-should-the-cache-folder-be-located"></a>其中的缓存文件夹应位于？

不建议将缓存文件夹放在以下位置：

* 网络共享/可移动介质：缓存文件夹必须位于需要使用联机备份进行备份的服务器本地。 不支持网络位置或可移动媒体，例如 U 盘
* 脱机卷：缓存文件夹必须联机才能使用 Azure 备份代理执行预期的备份

### <a name="are-there-any-attributes-of-the-cache-folder-that-arent-supported"></a>是否有不支持缓存文件夹的任何属性？
缓存文件夹不支持以下属性或其组合：

* 加密
* 已删除重复数据
* 压缩
* 稀疏
* 重分析点

缓存文件夹和元数据 VHD 没有 Azure 备份代理所需的属性。

### <a name="is-there-a-way-to-adjust-the-amount-of-bandwidth-used-for-backup"></a>是否有办法调整备份所使用的带宽量？
 
是的可以使用**更改属性**MARS 代理中的选项可调整带宽和时间。 [了解详细信息](backup-configure-vault.md#enable-network-throttling)* *。

## <a name="restore"></a>还原

### <a name="what-happens-if-i-cancel-an-ongoing-restore-job"></a>如果取消正在进行的还原作业，会发生什么情况？

如果取消正在进行的还原作业，则会停止还原过程。 在取消前还原所有文件都保留在已配置的目标 （原始或备用位置） 中而无需任何回滚。


## <a name="next-steps"></a>后续步骤

[了解](tutorial-backup-windows-server-to-azure.md)如何备份 Windows 计算机。
