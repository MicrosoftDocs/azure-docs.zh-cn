---
title: 适用于 Linux 的 azure 串行控制台 |Microsoft Docs
description: 为 Azure 虚拟机和虚拟机规模集的双向串行控制台。
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
ms.date: 5/1/2019
ms.author: harijay
ms.openlocfilehash: 9577a81af3da98c6e8802c586ec468a6e44e46cf
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442044"
---
# <a name="azure-serial-console-for-linux"></a>适用于 Linux 的 azure 串行控制台

在 Azure 门户中的串行控制台提供基于文本的控制台访问的 Linux 虚拟机 (Vm) 和虚拟机规模集实例。 此串行连接连接到 COM1 串行端口的 VM 或虚拟机规模集实例，提供对它独立于网络或操作系统状态的访问。 串行控制台只能使用 Azure 门户访问，并允许仅为这些用户具有访问权限角色的参与者或更高版本到 VM 或虚拟机规模集。

串行控制台以相同方式适用于虚拟机和虚拟机规模集实例。 在此文档中所有提及到的 Vm 将隐式都包括虚拟机规模集实例，除非另有说明。

有关 Windows 的串行控制台文档，请参阅[串行控制台的 Windows](../windows/serial-console.md)。

> [!NOTE]
> 在全球 Azure 区域中，串行控制台是已正式发布。 串行控制台目前不可用于 Azure 政府云或 Azure 中国云。


## <a name="prerequisites"></a>必备组件

- VM 或虚拟机规模集实例必须使用资源管理部署模型。 不支持经典部署。

- 你使用串行控制台的帐户必须具有[虚拟机参与者角色](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)vm 并[启动诊断](boot-diagnostics.md)存储帐户

- VM 或虚拟机规模集实例必须有一个基于密码的用户。 可以使用 VM 访问扩展的[重置密码](https://docs.microsoft.com/azure/virtual-machines/extensions/vmaccess#reset-password)功能创建一个帐户。 在“支持 + 故障排除”部分选择“重置密码”。

- VM 或虚拟机规模集实例必须具有[启动诊断](boot-diagnostics.md)启用。

    ![启动诊断设置](./media/virtual-machines-serial-console/virtual-machine-serial-console-diagnostics-settings.png)

- 有关特定于 Linux 分发版的设置，请参阅[串行控制台 Linux 分发版可用性](#serial-console-linux-distribution-availability)。



## <a name="get-started-with-the-serial-console"></a>开始使用串行控制台
Vm 和虚拟机规模集的串行控制台可访问只能通过 Azure 门户：

### <a name="serial-console-for-virtual-machines"></a>虚拟机串行控制台
适用于 Vm 的串行控制台是与单击的简单**串行控制台**内**支持 + 故障排除**在 Azure 门户中的部分。
  1. 打开 [Azure 门户](https://portal.azure.com)。

  1. 导航到**的所有资源**选择虚拟机。 此时会打开该 VM 的概述页。

  1. 向下滚动到“支持 + 故障排除”部分，并选择“串行控制台”。 此时会打开一个包含串行控制台的新窗格，并启动连接。

     ![Linux 的串行控制台窗口](./media/virtual-machines-serial-console/virtual-machine-linux-serial-console-connect.gif)

### <a name="serial-console-for-virtual-machine-scale-sets"></a>虚拟机规模集的串行控制台
虚拟机规模集的每个实例基础上提供了串行控制台。 您将需要导航到虚拟机规模集的单个实例才会看到**串行控制台**按钮。 如果虚拟机规模集不具有已启用启动诊断，请确保更新虚拟机规模集模型以启用启动诊断，然后再升级所有实例到新模型才能访问串行控制台。
  1. 打开 [Azure 门户](https://portal.azure.com)。

  1. 导航到**的所有资源**和选择虚拟机规模集。 虚拟机规模集的概览页设置打开。

  1. 导航到**实例**

  1. 选择虚拟机规模集实例

  1. 从**支持 + 故障排除**部分中，选择**串行控制台**。 此时会打开一个包含串行控制台的新窗格，并启动连接。

     ![Linux 虚拟机规模集串行控制台](./media/virtual-machines-serial-console/vmss-start-console.gif)


> [!NOTE]
> 串行控制台需要一个配置了密码的本地用户。 虚拟机或虚拟机规模集仅使用 SSH 公共密钥配置将无法登录到串行控制台。 若要创建带有密码的本地用户，请使用 [VMAccess 扩展](https://docs.microsoft.com/azure/virtual-machines/linux/using-vmaccess-extension)（在 Azure 门户中选择“重置密码”即可访问它），然后创建带有密码的本地用户。
> 也可在帐户中重置管理员密码，方法是[使用 GRUB 启动进入单用户模式](./serial-console-grub-single-user-mode.md)。

## <a name="serial-console-linux-distribution-availability"></a>串行控制台 Linux 分发的可用性
要使串行控制台正常运行，必须将来宾操作系统配置为向串行端口读取和写入控制台消息。 大多数[认可的 Azure Linux 分发版](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)默认都已配置串行控制台。 在 Azure 门户的“支持 + 故障排除”部分选择“串行控制台”即可访问串行控制台。

分发组      | 串行控制台访问
:-----------|:---------------------
Red Hat Enterprise Linux    | 默认已启用串行控制台访问。
CentOS      | 默认已启用串行控制台访问。
Ubuntu      | 默认已启用串行控制台访问。
CoreOS      | 默认已启用串行控制台访问。
SUSE        | Azure 中提供的较新 SLES 映像默认已启用串行控制台访问。 如果在 Azure 上使用旧版（10 或更低）SLES，请参阅此[知识库文章](https://www.novell.com/support/kb/doc.php?id=3456486)启用串行控制台。
Oracle Linux        | 默认已启用串行控制台访问。
自定义 Linux 映像     | 若要为自定义 Linux VM 映像启用串行控制台，请在文件 */etc/inittab* 中启用控制台访问，以便在 `ttyS0` 上运行终端。 例如：`S0:12345:respawn:/sbin/agetty -L 115200 console vt102`。 有关正确创建自定义映像的详细信息，请参阅[在 Azure 中创建和上传 Linux VHD](https://aka.ms/createuploadvhd)。 如果你正在生成自定义内核，请考虑启用以下内核标志：`CONFIG_SERIAL_8250=y` 和 `CONFIG_MAGIC_SYSRQ_SERIAL=y`。 配置文件通常位于 */boot/* 路径。

> [!NOTE]
> 如果在串行控制台中没有看到任何内容，请确保在 VM 上启用了启动诊断。 达到**Enter**将通常修复问题的任何内容显示在串行控制台。

## <a name="common-scenarios-for-accessing-the-serial-console"></a>访问串行控制台的常见场景

场景          | 串行控制台中的操作
:------------------|:-----------------------------------------
*FSTAB* 文件受损 | 按 **Enter** 键继续，然后使用文本编辑器修复 *FSTAB* 文件。 可能需要在单用户模式下执行此操作。 有关详细信息，请参阅的串行控制台部分[如何修复 fstab 问题](https://support.microsoft.com/help/3206699/azure-linux-vm-cannot-start-because-of-fstab-errors)并[使用串行控制台访问 GRUB 和单用户模式下](serial-console-grub-single-user-mode.md)。
错误的防火墙规则 |  如果已配置 ip 表来阻止 SSH 连接，可以使用串行控制台进行与 VM 交互而无需 SSH。 更多详细信息，请参阅[iptables 手册页](https://linux.die.net/man/8/iptables)。<br>同样，如果你 firewalld 阻止 SSH 访问，可以通过串行控制台访问 VM，并重新配置 firewalld。 更多详细信息可在[firewalld 文档](https://firewalld.org/documentation/)。
文件系统损坏/检查 | 请参阅的串行控制台部分[Azure Linux VM 因文件系统错误而无法启动](https://support.microsoft.com/en-us/help/3213321/linux-recovery-cannot-ssh-to-linux-vm-due-to-file-system-errors-fsck)的详细故障排除的详细信息已损坏的文件系统使用串行控制台。
SSH 配置问题 | 访问串行控制台并更改设置。 串行控制台可以使用而不考虑 VM 的 SSH 配置，因为它不需要 VM 具有网络连接。 故障排除指南位于[排除使用 SSH 连接到 Azure Linux VM 的失败，出现错误，或被拒绝](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/troubleshoot-ssh-connection)。 更多详细信息位于[详细 SSH 故障排除连接到 Azure 中的 Linux VM 的问题的步骤](./detailed-troubleshoot-ssh-connection.md)
与引导加载程序交互 | 从串行控制台边栏选项卡中重启 VM 以访问 Linux VM 上的 GRUB。 有关更多详细信息和特定于发行版的信息，请参阅[使用串行控制台访问 GRUB 和单用户模式下](serial-console-grub-single-user-mode.md)。

## <a name="disable-the-serial-console"></a>禁用串行控制台
默认情况下，所有订阅都已启用串行控制台访问。 您可以禁用在订阅级别或 VM/虚拟机规模集级别的串行控制台。 请注意，必须使串行控制台中工作的 VM 上启用启动诊断。

### <a name="vmvirtual-machine-scale-set-level-disable"></a>VM/虚拟机规模集级别禁用
可以禁用的特定虚拟机或虚拟机规模集通过禁用启动诊断设置的串行控制台。 关闭从 Azure 门户对 VM 或虚拟机规模集禁用串行控制台的启动诊断。 如果在虚拟机规模集上使用串行控制台，请确保虚拟机规模集实例升级到最新模型。

> [!NOTE]
> 若要为订阅启用或禁用串行控制台，必须具有订阅的写入权限。 这些权限包括管理员或所有者角色。 自定义角色也可能具有写入权限。

### <a name="subscription-level-disable"></a>订阅级禁用
可以通过[禁用控制台 REST API 调用](/rest/api/serialconsole/console/disableconsole)为整个订阅禁用串行控制台。 可以使用此 API 文档页面上提供的“试用”功能为订阅禁用和启用串行控制台。 在“subscriptionId”字段中输入自己的订阅 ID，在“default”字段中输入“default”，然后选择“运行”。 Azure CLI 命令目前不可用。

![试用 REST API](./media/virtual-machines-serial-console/virtual-machine-serial-console-rest-api-try-it.png)

另外，可以在 Cloud Shell 中使用以下 bash 命令集来禁用、启用订阅的串行控制台，以及查看其禁用状态。

* 若要获取订阅的串行控制台禁用状态，请使用以下命令：
    ```azurecli-interactive
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"'))

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s | jq .properties
    ```
* 若要禁用订阅的串行控制台，请使用以下命令：
    ```azurecli-interactive
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"'))

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl -X POST "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default/disableConsole?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s -H "Content-Length: 0"
    ```
* 若要启用订阅的串行控制台，请使用以下命令：
    ```azurecli-interactive
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"'))

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl -X POST "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default/enableConsole?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s -H "Content-Length: 0"
    ```

## <a name="serial-console-security"></a>串行控制台安全性

### <a name="access-security"></a>访问安全性
只允许对虚拟机拥有[虚拟机参与者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)或更高权限访问角色的用户访问串行控制台。 如果 Azure Active Directory 租户需要多重身份验证 (MFA)，则访问串行控制台时也需要执行 MFA，因为串行控制台是通过 [Azure 门户](https://portal.azure.com)访问的。

### <a name="channel-security"></a>通道安全性
来回发送的所有数据在线路上经过加密。

### <a name="audit-logs"></a>审核日志
对串行控制台的所有访问目前都会记录在虚拟机的[启动诊断](https://docs.microsoft.com/azure/virtual-machines/linux/boot-diagnostics)日志中。 Azure 虚拟机管理员拥有并可控制这些日志的访问权限。

> [!CAUTION]
> 不会记录控制台的访问密码。 但是，如果在控制台中运行的命令包含或输出密码、机密、用户名或其他任何形式的个人身份信息 (PII)，则这些信息将写入到 VM 启动诊断日志。 这些信息是在实现串行控制台的回滚功能过程中连同其他所有可见文本一起写入的。 这些日志不断循环，只有对诊断存储帐户拥有读取权限的个人才能访问它们。 但是，我们建议遵循有关将远程桌面用于涉及机密和/或 PII 的任何操作的最佳做法。

### <a name="concurrent-usage"></a>并发使用
如果某个用户已连接到串行控制台，而另一个用户已成功请求访问同一个虚拟机，则第一个用户将断开连接，第二用户此时已连接到同一会话。

> [!CAUTION]
> 这意味着，断开连接的用户尚未注销。强制断开连接后的注销 （通过 SIGHUP 或类似机制） 的能力是仍在规划之中。 对于 Windows，特殊管理控制台 (SAC) 中会启用自动超时；但对于 Linux，可以配置终端超时设置。 为此，请将 `export TMOUT=600` 添加到用于登录控制台的用户的 *.bash_profile* 或 *.profile* 中。 此设置使会话在 10 分钟后超时。

## <a name="accessibility"></a>辅助功能
可访问性是 Azure 串行控制台的一个重要关注点。 为此，我们会确保串行控制台的完全可访问性。

### <a name="keyboard-navigation"></a>键盘导航
使用键盘上的 **Tab** 键在 Azure 门户中的串行控制台界面上导航。 屏幕上会突出显示你的位置。 若要使焦点离开串行控制台窗口，请在键盘上按 **Ctrl**+**F6**。

### <a name="use-serial-console-with-a-screen-reader"></a>与屏幕阅读器使用串行控制台
串行控制台内置了屏幕阅读器支持。 在打开屏幕阅读器的情况下导航，屏幕阅读器可大声读出当前所选按钮的替换文字。

## <a name="errors"></a>错误
大多数错误都是暂时性的，重试连接往往可以解决。 下表显示了错误和缓解措施的列表。 这些错误和缓解措施适用于这两个 Vm 和虚拟机规模集实例。

错误                            |   缓解措施
:---------------------------------|:--------------------------------------------|
无法检索 *&lt;VMNAME&gt;* 的启动诊断设置。 若要使用串行控制台，请确保为此 VM 启用了启动诊断。 | 确保 VM 已启用[启动诊断](boot-diagnostics.md)。
VM 处于已停止/已解除分配状态。 启动 VM，然后重试串行控制台连接。 | VM 必须处于已启动状态才能访问串行控制台。
没有所需的权限，无法使用此 VM 来访问串行控制台。 请确保至少拥有“虚拟机参与者”角色权限。| 需要特定的权限才能访问串行控制台。 有关详细信息，请参阅[先决条件](#prerequisites)。
无法确定启动诊断存储帐户 *&lt;STORAGEACCOUNTNAME&gt;* 的资源组。 确认是否为此 VM 启用了启动诊断，以及是否有权访问此存储帐户。 | 需要特定的权限才能访问串行控制台。 有关详细信息，请参阅[先决条件](#prerequisites)。
Web 套接字已关闭或无法打开。 | 你可能需要将 `*.console.azure.com` 加入允许列表。 一种更详细但更冗长的方法是将 [Microsoft Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)加入允许列表，该范围相当规律地定期更改。
访问此 VM 的启动诊断存储帐户时遇到“已禁止”响应。 | 请确保启动诊断没有帐户防火墙。 若要使串行控制台正常运行，需要一个可访问的启动诊断存储帐户。

## <a name="known-issues"></a>已知问题
我们注意到串行控制台存在的一些问题。 下面是这些问题和缓解措施的列表。 这些问题和缓解措施适用于这两个 Vm 和虚拟机规模集实例。

问题                           |   缓解措施
:---------------------------------|:--------------------------------------------|
在出现连接标题后按 **Enter** 不会显示登录提示。 | 有关详细信息，请参阅[按 Enter 不起任何作用](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)。 如果您运行的自定义 VM、 加强的设备或导致无法连接到串行端口的 linux 操作系统的 GRUB 配置，则可能出现此问题。
串行控制台文本仅占用屏幕大小的一部分（通常在使用文本编辑器后）。 | 串行控制台不支持协商窗口大小 ([RFC 1073](https://www.ietf.org/rfc/rfc1073.txt))，这意味着不会发送 SIGWINCH 信号来更新屏幕大小，因此 VM 不会了解终端的大小。 安装可提供 `resize` 命令的 xterm 或类似实用工具，然后运行 `resize`。
无法粘贴长字符串。 | 串行控制台将粘贴到终端的字符串长度限制为 2048 个字符，以防止串行端口带宽过载。
串行控制台不能与存储帐户防火墙一起使用。 | 根据设计串行控制台无法与启动诊断存储帐户上启用的存储帐户防火墙一起使用。


## <a name="frequently-asked-questions"></a>常见问题

**问：如何发送反馈？**

A. 可以通过在 https://aka.ms/serialconsolefeedback 中创建 GitHub 问题来提供反馈。 也可以通过 azserialhelp@microsoft.com，或者 https://feedback.azure.com 上的虚拟机类别发送反馈（不太建议）。

**问：串行控制台是否支持复制/粘贴？**

A. 可以。 可以使用 **Ctrl**+**Shift**+**C** 和 **Ctrl**+**Shift**+**V** 复制并粘贴到终端。

**问：是否可以使用串行控制台替代 SSH 连接？**

A. 虽然从技术上讲这种用法是可行的，但串行控制台主要用作无法通过 SSH 进行连接时的故障排除工具。 不建议使用串行控制台替代 SSH 的原因如下：

- 串行控制台的带宽不如 SSH 那样大。 由于它是仅限文本的连接，因此很难进行大量的 GUI 交互。
- 目前只能使用用户名和密码访问串行控制台。 由于 SSH 密钥比用户名/密码组合安全得多，因此，从登录安全性的立场，我们建议使用 SSH 而不是串行控制台。

**问：谁可以为我的订阅启用或禁用串行控制台？**

A. 若要在订阅范围级别启用或禁用串行控制台，必须拥有订阅的写入权限。 拥有写入权限的角色包括管理员或所有者角色。 自定义角色也可能具有写入权限。

**问：谁可以访问我的 VM/虚拟机规模集的串行控制台？**

A. 您必须具有虚拟机参与者角色或更高的虚拟机或虚拟机规模集来访问串行控制台。

**问：我的串口控制台未显示任何内容，该怎么办？**

A. 你的映像可能配置错误，无法进行串行控制台访问。 有关配置映像以启用串行控制台的详细信息，请参阅[串行控制台 Linux 分发版可用性](#serial-console-linux-distribution-availability)。

**问：串行控制台是否可用于虚拟机规模集？**

A. 是的，它是！ 请参阅[虚拟机规模集的串行控制台](#serial-console-for-virtual-machine-scale-sets)

**问：如果我设置我的 VM 或虚拟机规模集仅使用 SSH 密钥身份验证，我仍可用的串行控制台来连接到我的 VM/虚拟机规模集实例？**

A. 可以。 由于串行控制台不需要 SSH 密钥，因此你只需设置用户名/密码组合。 为此，可以在 Azure 门户中选择“重置密码”，然后使用这些凭据登录到串行控制台。

## <a name="next-steps"></a>后续步骤
* 使用串行控制台[访问 GRUB 和单用户模式](serial-console-grub-single-user-mode.md)。
* 使用串行控制台执行 [NMI 和 SysRq 调用](serial-console-nmi-sysrq.md)。
* 了解如何使用串行控制台[在各种分发版中启用 GRUB](https://blogs.msdn.microsoft.com/linuxonazure/2018/10/23/why-proactively-ensuring-you-have-access-to-grub-and-sysrq-in-your-linux-vm-could-save-you-lots-of-down-time/)。
* 串行控制台也适用于 [Windows VM](../windows/serial-console.md)。
* 详细了解[启动诊断](boot-diagnostics.md)。

