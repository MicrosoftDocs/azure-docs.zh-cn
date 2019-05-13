---
title: Azure VM 上的 Windows 重启循环 | Microsoft Docs
description: 了解如何对 Windows 重启循环进行故障排除 |Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: cshepard
editor: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/15/2018
ms.author: genli
ms.openlocfilehash: 1c97b1da094b759ccf85f310ceec4c7abfd91b9b
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472298"
---
# <a name="windows-reboot-loop-on-an-azure-vm"></a>Azure VM 上的 Windows 重启循环
本文介绍在 Microsoft Azure 中的 Windows 虚拟机 (VM) 上可能遇到的重启循环。

## <a name="symptom"></a>症状

使用[启动诊断](./boot-diagnostics.md)获取 VM 的屏幕截图时，你发现虚拟机正在启动，但启动进程被中断并且即将重新开始。

![开始屏幕 1](./media/troubleshoot-reboot-loop/start-screen-1.png)

## <a name="cause"></a>原因

重启循环由以下原因导致：

### <a name="cause-1"></a>原因 1

存在标记为“重要”的第三方服务，并且该服务无法启动。 这将导致操作系统重启。

### <a name="cause-2"></a>原因 2

对操作系统进行了一些更改。 通常情况下，这些更改与更新安装、应用程序安装或新策略相关。 可能需要查看以下日志来了解更多详细信息：

- 事件日志
- CBS.logWindows
- Update.log

### <a name="cause-3"></a>原因 3

文件系统损坏可能导致此问题。 但是，很难诊断并确定导致操作系统损坏的更改。

## <a name="solution"></a>解决方案

要解决此问题，请[备份 OS 磁盘](../windows/snapshot-copy-managed-disk.md)，并[将 OS 磁盘附加到安全的 VM](../windows/troubleshoot-recovery-disks-portal.md)，然后按照相应地解决方案选项执行操作或逐个尝试解决方案。

### <a name="solution-for-cause-1"></a>原因 1 的解决方案

1. 将 OS 磁盘附加到有效的 VM 后，确保磁盘在磁盘管理控制台中标记为“联机”，并记下保存 \Windows 文件夹的分区的驱动器号。

2. 如果磁盘设置为“脱机”，则将其设置为“联机”。

3. 创建 \Windows\System32\config 文件夹的副本，以防需要回滚更改。

4. 在安全的 VM 中，打开 Windows 注册表编辑器 (regedit)。

5. 选择 HKEY_LOCAL_MACHINE 键，然后选择菜单中的“文件” > “加载配置单元”。

6. 浏览到 \Windows\System32\config 文件夹中的系统文件。

7. 选择“打开”，键入 BROKENSYSTEM 作为名称，展开 HKEY_LOCAL_MACHINE 键，然后将看到名为 BROKENSYSTEM 的附加键。

8. 检查计算机从哪个 ControlSet 重启。 你将在以下注册表项中看到键编号。

    `HKEY_LOCAL_MACHINE\BROKENSYSTEM\Select\Current`

9. 通过以下注册表项查看 VM 代理服务的重要程度。

    `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\RDAgent\ErrorControl`

10. 如果注册表项的值未设置为 2，则执行下一步缓解措施。

11. 如果注册表项的值设置为 2，则将值从 2 改为 1。

12. 如果存在以下键，并且其值为 2 或 3，则将这些值相应地改为 1：

    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupCoordinatorSvc\ErrorControl`
    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupInquirySvc\ErrorControl`
    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupPluginSvc\ErrorControl`

13. 选择 BROKENSYSTEM 键，然后选择菜单中的“文件” > “加载配置单元”。

14. 从故障排除 VM 中分离 OS 磁盘。

15. 从故障排除 VM 中删除该磁盘并等待大约 2 分钟，以便 Azure 释放此磁盘。

16. [从 OS 磁盘创建新的 VM](../windows/create-vm-specialized.md)。

17. 如果问题得到解决，则可能需要重新安装 [RDAgent](https://blogs.msdn.microsoft.com/mast/2014/04/07/install-the-vm-agent-on-an-existing-azure-vm/) (WaAppAgent.exe)。

### <a name="solution-for-cause-2"></a>原因 2 的解决方案

将 VM 还原到上一个已知的正确配置：按照[如何使用上一个已知的正确配置启动 Azure Windows VM](https://support.microsoft.com/help/4016731/) 中的步骤执行操作。

### <a name="solution-for-cause-3"></a>原因 3 的解决方案
>[!NOTE]
>以下过程应该仅作为最后的手段。 尽管从 regback 还原可以还原对计算机的访问权限，但 OS 并不稳定，因为注册表中将出现数据丢失（在配置单元和当天的时间戳之间）。 需要构建新的 VM 并制订数据迁移计划。

1. 将磁盘附加到故障排除 VM 后，请确保磁盘在磁盘管理控制台中标记为“联机”。

2. 创建 \Windows\System32\config 文件夹的副本，以防需要回滚更改。

3. 复制 \Windows\System32\config\regback 文件夹中的文件，并替换 \Windows\System32\config 文件夹中的文件。

4. 从故障排除 VM 中删除该磁盘并等待大约 2 分钟，以便 Azure 释放此磁盘。

5. [从 OS 磁盘创建新的 VM](../windows/create-vm-specialized.md)。


