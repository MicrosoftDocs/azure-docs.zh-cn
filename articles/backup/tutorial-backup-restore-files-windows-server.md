---
title: 将文件从 Azure 恢复到 Windows Server
description: 本教程详细介绍了如何将项目从 Azure 恢复到 Windows Server。
services: backup
author: saurabhsensharma
manager: shivamg
keywords: windows server 备份; 还原文件 windows server; 备份和灾难恢复
ms.service: backup
ms.topic: tutorial
ms.date: 2/14/2018
ms.author: saurse
ms.custom: mvc
ms.openlocfilehash: b01811d9c933802263e975b23b5d40cd77303766
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60722853"
---
# <a name="recover-files-from-azure-to-a-windows-server"></a>将文件从 Azure 恢复到 Windows Server

使用 Azure 备份可以从 Windows Server 备份中恢复个别项目。 如果必须快速还原意外删除的文件，恢复个别文件会很有帮助。 本教程介绍如何使用 Microsoft Azure 恢复服务 (MARS) 代理从在 Azure 中执行的备份中恢复项目。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 启动个别项目的恢复 
> * 选择恢复点 
> * 从恢复点还原项目

本教程假定已经执行了[将 Windows Server 备份到 Azure](backup-configure-vault.md) 的步骤，并且在 Azure 中拥有至少一个 Windows Server 文件的备份。

## <a name="initiate-recovery-of-individual-items"></a>启动个别项目的恢复

使用 Microsoft Azure 恢复服务 (MARS) 代理安装名为“Microsoft Azure 备份”的有用用户界面向导。 Microsoft Azure 备份向导与 Microsoft Azure 恢复服务 (MARS) 代理协同工作，从存储在 Azure 中的恢复点检索备份数据。 使用 Microsoft Azure 备份向导确定要还原到 Windows Server 的文件或文件夹。 

1. 打开“Microsoft Azure 备份”管理单元。 可以通过在计算机中搜索 **Microsoft Azure 备份**找到该代理。

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/mars.png)

2. 在向导中，单击代理控制台的“操作窗格”中的“恢复数据”，启动“恢复数据”向导。

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/mars-recover-data.png)

3. 在“入门”页上，选择“此服务器(服务器名称)”，然后单击“下一步”。

4. 在“选择恢复模式”页上，选择“个别文件和文件夹”，然后单击“下一步”以开始恢复点选择过程。
 
5. 在“选择卷和日期”页上，选择包含想要还原的文件或文件夹的卷，然后单击“装载”。 选择日期，然后从下拉菜单中选择与恢复点相对应的时间。 以**粗体**显示的日期指示当天至少有一个可用的恢复点。

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/mars-select-date.png)
 
    单击“装载”时，Azure 备份会将恢复点用作磁盘。 从磁盘浏览和恢复文件。

## <a name="restore-items-from-a-recovery-point"></a>从恢复点还原项目

1. 装载恢复卷后，单击“浏览”打开 Windows 资源管理器，并查找希望恢复的文件和文件夹。 

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/mars-browse-recover.png)

    可以从恢复卷直接打开文件，并验证这些文件。

2. 在 Windows 资源管理器中，复制想要还原的文件和/或文件夹，然后将它们粘贴到服务器的任何所需位置。

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/mars-final.png)

3. 完成还原文件和/或文件夹后，在“恢复数据”向导的“浏览和恢复文件”页上，单击“卸载”。 

    ![等待中的备份](./media/tutorial-backup-restore-files-windows-server/unmount-and-confirm.png)

4.  单击“是”，确认要卸载该卷。

    卸载快照后，代理控制台的“作业”窗格中将显示“作业已完成”。

## <a name="next-steps"></a>后续步骤

这将完成有关将 Windows Server 数据备份和还原到 Azure 的教程。 若要了解有关 Azure 备份的详细信息，请参阅备份加密虚拟机的 PowerShell 示例。

> [!div class="nextstepaction"]
> [备份加密 VM](./scripts/backup-powershell-sample-backup-encrypted-vm.md)
