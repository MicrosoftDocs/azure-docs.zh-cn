---
title: 用于 SysRq 和 NMI 调用的 Azure 串行控制台 | Microsoft Docs
description: 在 Azure 虚拟机中使用串行控制台发出 SysRq 和 NMI 调用。
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: ad29bbd038c8982778f2dbca63756f6995077dce
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204913"
---
# <a name="use-serial-console-for-sysrq-and-nmi-calls"></a>使用串行控制台发出 SysRq 和 NMI 调用

## <a name="system-request-sysrq"></a>系统请求 (SysRq)
SysRq 是 Linux 操作系统内核识别的按键序列，可以触发一组预定义的操作。 （例如，如果 VM 未响应） 不能通过传统管理执行虚拟机进行故障排除或恢复时，通常使用这些命令。 使用 Azure 串行控制台的 SysRq 功能时，将会模拟 SysRq 按键，以及物理键盘上的字符输入。

传送 SysRq 序列后，内核配置将控制系统的响应方式。 有关启用和禁用 SysRq 的信息，请参阅 *SysRq 管理员指南* [text](https://aka.ms/kernelorgsysreqdoc) | [markdown](https://aka.ms/linuxsysrq)。  

可以使用下面显示的命令栏上的键盘图标通过 Azure 串行控制台向 Azure 虚拟机发送 SysRq。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg)

选择“发送 SysRq 命令”会打开一个对话框，其中提供了常用 SysRq 选项，或者可以接受该对话框中输入的 SysRq 命令序列。  这样，SysRq 序列便可以执行某个高级别的操作，例如，使用 `REISUB` 执行安全重新启动。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-sysreq_UI.png)

不能在已停止或者其内核处于无响应状态的虚拟机上使用 SysRq 命令。 （例如内核崩溃）。

### <a name="enable-sysrq"></a>启用 SysRq 
可按前面提到的 *SysRq 管理员指南*中所述对 SysRq 进行配置，使所有、没有任何或者只有特定的命令可用。 可使用以下步骤启用所有 SysRq 命令，但重新启动后，此配置将会失效。
```
echo "1" >/proc/sys/kernel/sysrq
```
若要永久保留 SysReq 配置，可执行以下命令来启用所有 SysRq 命令
1. 将此行添加到 */etc/sysctl.conf* <br>
    `kernel.sysrq = 1`
1. 运行以下命令来重新启动或更新 sysctl <br>
    `sysctl -p`

### <a name="command-keys"></a>命令键 
在上述 SysRq 管理员指南中：

|命令| 函数
| ------| ----------- |
|``b``  |   将立即重新启动系统，且不会同步或卸载磁盘。
|``c``  |   将通过 NULL 指针取消引用来执行系统崩溃。 将执行故障转储（如果已配置）。
|``d``  |   显示保持的所有锁。
|``e``  |   向所有进程（init 除外）发送 SIGTERM。
|``f``  |   将调用 oom 终止程序来终止占用大量内存的进程，但如果没有任何进程可终止，则不会造成崩溃。
|``g``  |   由 kgdb（内核调试程序）使用
|``h``  |   将显示帮助（使用此处所列以外的其他任何键也会显示帮助，但 ``h`` 很容易记住）
|``i``  |    向所有进程（init 除外）发送 SIGKILL。
|``j``  |    强制“将其解冻”- FIFREEZE ioctl 冻结的文件系统。
|``k``  |    安全访问键 (SAK) 会终止当前虚拟控制台上的所有程序。 注意:请参阅下面 SAK 部分中的重要注释。
|``l``  |    显示所有活动 CPU 的堆栈回溯。
|``m``  |    将当前内存信息转储到控制台。
|``n``  |    用于顺利完成 RT 任务
|``o``  |    将关闭系统（如果已配置并受支持）。
|``p``  |    将当前寄存器和标志转储到控制台。
|``q``  |    将转储所有已装备的 hrtimer（但不包括普通的 timer_list 计时器）的 CPU 列表和有关所有 clockevent 设备的详细信息。
|``r``  |    关闭键盘原始模式，并将其设置为 XLATE。
|``s``  |    将尝试同步所有已装载的文件系统。
|``t``  |    将当前任务的列表及其信息转储到控制台。
|``u``  |    将尝试重新装载所有已装载的只读文件系统。
|``v``  |    强制还原帧缓冲区控制台
|``v``  |    导致 ETM 缓冲区转储 [特定于 ARM]
|``w``  |    转储处于不可间断（阻塞）状态的任务。
|``x``  |    由 ppc/powerpc 平台上的 xmon 接口使用。 显示 sparc64 上的全局 PMU 寄存器。 转储 MIPS 上的所有 TLB 项。
|``y``  |    显示全局 CPU 寄存器 [特定于 SPARC-64]
|``z``  |    转储 ftrace 缓冲区
|``0``-``9`` | 设置控制台日志级别，控制要在控制台上列显哪些内核消息。 （例如，指定 ``0`` 只会在控制台上列显 PANIC 或 OOPS 等紧急消息。）

### <a name="distribution-specific-documentation"></a>特定于分发版的文档 ###
有关 SysRq 的特定于分发版的文档，以及配置 Linux 以便在收到 SysRq“Crash”命令时创建故障转储的步骤，请参阅以下链接：

#### <a name="ubuntu"></a>Ubuntu ####
 - [内核故障转储](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

#### <a name="red-hat"></a>Red Hat ####
- [SysRq 工具是什么，如何使用它？](https://access.redhat.com/articles/231663)
- [如何使用 SysRq 工具从 RHEL 服务器收集信息](https://access.redhat.com/solutions/2023)

#### <a name="suse"></a>SUSE ####
- [配置内核核心转储捕获](https://www.suse.com/support/kb/doc/?id=3374462)

#### <a name="coreos"></a>CoreOS ####
- [收集崩溃日志](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="non-maskable-interrupt-nmi"></a>不可屏蔽的中断 (NMI) 
不可屏蔽的中断 (NMI) 旨在创建虚拟机上的软件不会忽略的信号。 过去，NMI 用来监视要求实现特定响应时间的系统上的硬件问题。  通常，今天、 程序员和系统管理员使用 NMI 作为一种机制，若要调试或故障排除系统未响应。

可以使用下面显示的命令栏上的键盘图标通过串行控制台向 Azure 虚拟机发送 NMI。 传送 NMI 后，虚拟机配置将控制系统的响应方式。  可将 Linux 操作系统配置为在收到 NMI 时发生崩溃并创建内存转储。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg) <br>

### <a name="enable-nmi"></a>启用 NMI
对于支持使用 sysctl 配置内核参数的 Linux 系统，可以使用以下步骤来启用收到此 NMI 时发生崩溃：
1. 将此行添加到 */etc/sysctl.conf* <br>
    `kernel.panic_on_unrecovered_nmi=1`
1. 运行以下命令来重新启动或更新 sysctl <br>
    `sysctl -p`

有关 Linux 内核配置的详细信息，包括 `unknown_nmi_panic`、`panic_on_io_nmi` 和 `panic_on_unrecovered_nmi`，请参阅：[/proc/sys/kernel/* 的文档](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt)。 有关 NMI 的特定于分发版的文档，以及配置 Linux 以便在收到 NMI 时创建故障转储的步骤，请参阅以下链接：
 
### <a name="ubuntu"></a>Ubuntu 
 - [内核故障转储](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

### <a name="red-hat"></a>Red Hat 
 - [NMI 是什么，它有什么用途？](https://access.redhat.com/solutions/4127)
 - [如何将系统配置为在按下 NMI 开关时发生崩溃？](https://access.redhat.com/solutions/125103)
 - [故障转储管理员指南](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/kernel_crash_dump_guide/kernel-crash-dump-guide.pdf)

### <a name="suse"></a>SUSE 
- [配置内核核心转储捕获](https://www.suse.com/support/kb/doc/?id=3374462)

### <a name="coreos"></a>CoreOS 
- [收集崩溃日志](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="next-steps"></a>后续步骤
* 主要串行控制台 Linux 文档页位于[此处](serial-console-linux.md)。
* 使用串行控制台启动到 [GRUB 并进入单用户模式](serial-console-grub-single-user-mode.md)
* 串行控制台也适用于 [Windows](serial-console-windows.md) VM
* 详细了解[启动诊断](boot-diagnostics.md)