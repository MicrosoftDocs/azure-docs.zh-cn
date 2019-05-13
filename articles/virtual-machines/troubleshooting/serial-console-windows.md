---
title: Windows azure 串行控制台 |Microsoft Docs
description: 为 Azure 虚拟机和虚拟机规模集的双向串行控制台。
services: virtual-machines-windows
documentationcenter: ''
author: asinn826
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 5/1/2019
ms.author: harijay
ms.openlocfilehash: 31811d3d48f791e11bd6b709f59f5f6aaa18be0e
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442109"
---
# <a name="azure-serial-console-for-windows"></a>Windows azure 串行控制台

在 Azure 门户中的串行控制台提供基于文本的控制台访问的 Windows 虚拟机 (Vm) 和虚拟机规模集实例。 此串行连接连接到 COM1 串行端口的 VM 或虚拟机规模集实例，提供对它独立于网络或操作系统状态的访问。 串行控制台只能使用 Azure 门户访问，并允许仅为这些用户具有访问权限角色的参与者或更高版本到 VM 或虚拟机规模集。

串行控制台以相同方式适用于虚拟机和虚拟机规模集实例。 在此文档中所有提及到的 Vm 将隐式都包括虚拟机规模集实例，除非另有说明。

有关 Linux 虚拟机和虚拟机规模集的串行控制台文档，请参阅[适用于 Linux 的 Azure 串行控制台](serial-console-linux.md)。

> [!NOTE]
> 在全球 Azure 区域中，串行控制台是已正式发布。 串行控制台目前不可用于 Azure 政府云或 Azure 中国云。


## <a name="prerequisites"></a>必备组件

* VM 或虚拟机规模集实例必须使用资源管理部署模型。 不支持经典部署。

- 你使用串行控制台的帐户必须具有[虚拟机参与者角色](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)vm 并[启动诊断](boot-diagnostics.md)存储帐户

- VM 或虚拟机规模集实例必须有一个基于密码的用户。 可以使用 VM 访问扩展的[重置密码](https://docs.microsoft.com/azure/virtual-machines/extensions/vmaccess#reset-password)功能创建一个帐户。 在“支持 + 故障排除”部分选择“重置密码”。

* 你要在其中访问串行控制台的 VM 必须已启用[启动诊断](boot-diagnostics.md)。

    ![启动诊断设置](../media/virtual-machines-serial-console/virtual-machine-serial-console-diagnostics-settings.png)

## <a name="get-started-with-the-serial-console"></a>开始使用串行控制台
Vm 和虚拟机规模集的串行控制台可访问只能通过 Azure 门户：

### <a name="serial-console-for-virtual-machines"></a>虚拟机串行控制台
适用于 Vm 的串行控制台是与单击的简单**串行控制台**内**支持 + 故障排除**在 Azure 门户中的部分。
  1. 打开 [Azure 门户](https://portal.azure.com)。

  1. 导航到**的所有资源**选择虚拟机。 此时会打开该 VM 的概述页。

  1. 向下滚动到“支持 + 故障排除”部分，并选择“串行控制台”。 此时会打开一个包含串行控制台的新窗格，并启动连接。

### <a name="serial-console-for-virtual-machine-scale-sets"></a>虚拟机规模集的串行控制台
虚拟机规模集的每个实例基础上提供了串行控制台。 您将需要导航到虚拟机规模集的单个实例才会看到**串行控制台**按钮。 如果虚拟机规模集不具有已启用启动诊断，请确保更新虚拟机规模集模型以启用启动诊断，然后再升级所有实例到新模型才能访问串行控制台。
  1. 打开 [Azure 门户](https://portal.azure.com)。

  1. 导航到**的所有资源**和选择虚拟机规模集。 虚拟机规模集的概览页设置打开。

  1. 导航到**实例**

  1. 选择虚拟机规模集实例

  1. 从**支持 + 故障排除**部分中，选择**串行控制台**。 此时会打开一个包含串行控制台的新窗格，并启动连接。

## <a name="enable-serial-console-functionality"></a>启用串行控制台功能

> [!NOTE]
> 如果看不到串行控制台中的任何内容，请确保在 VM 或虚拟机规模集上启用启动诊断。

### <a name="enable-the-serial-console-in-custom-or-older-images"></a>在自定义或更低版本的映像中启用串行控制台
Azure 上较新的 Windows Server 映像默认情况下已启用[特殊管理控制台](https://technet.microsoft.com/library/cc787940(v=ws.10).aspx) (SAC)。 SAC 在服务器版本的 Windows 上受支持，但在客户端版本（例如 Windows 10、Windows 8 或 Windows 7）上不可用。

对于较旧的 Windows Server 映像（在 2018 年 2 月之前创建），可以通过 Azure 门户的运行命令功能自动启用串行控制台。 在 Azure 门户中，选择**运行命令**，然后选择名为命令**EnableEMS**从列表中。

![运行命令列表](./media/virtual-machines-serial-console/virtual-machine-windows-serial-console-runcommand.png)

或者，若要手动启用在 2018 年 2 月之前创建的 Windows Vm/虚拟机规模集的串行控制台，请按照下列步骤：

1. 通过使用远程桌面连接到 Windows 虚拟机
1. 从管理命令提示符运行以下命令：
    - `bcdedit /ems {current} on`
    - `bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200`
1. 重新启动系统以启用 SAC 控制台。

    ![SAC 控制台](./media/virtual-machines-serial-console/virtual-machine-windows-serial-console-connect.gif)

如果需要，也可以使 SAC 脱机：

1. 将想要为其配置 SAC 的 Windows 磁盘作为数据磁盘附加到现有 VM。

1. 从管理命令提示符运行以下命令：
   - `bcdedit /store <mountedvolume>\boot\bcd /ems {default} on`
   - `bcdedit /store <mountedvolume>\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

#### <a name="how-do-i-know-if-sac-is-enabled"></a>如何知道是否已启用 SAC？

如果未启用 [SAC](https://technet.microsoft.com/library/cc787940(v=ws.10).aspx)，则串行控制台将不会显示 SAC 提示符。 在某些情况下，会显示 VM 运行状况信息，而在其他情况下则为空白。 如果使用的是 2018 年 2 月之前创建的 Windows Server 映像，则可能无法启用 SAC。

### <a name="enable-the-windows-boot-menu-in-the-serial-console"></a>在串行控制台中启用 Windows 启动菜单

如果需要让 Windows 启动加载程序提示显示在串行控制台中，可以将以下附加选项添加到启动配置数据中。 有关详细信息，请参阅 [bcdedit](https://docs.microsoft.com/windows-hardware/drivers/devtest/bcdedit--set)。

1. 连接到 Windows VM 或虚拟机规模集实例使用远程桌面。

1. 从管理命令提示符运行以下命令：
   - `bcdedit /set {bootmgr} displaybootmenu yes`
   - `bcdedit /set {bootmgr} timeout 10`
   - `bcdedit /set {bootmgr} bootems yes`

1. 重新启动系统以启用启动菜单

> [!NOTE]
> 为显示的启动管理器菜单所设置的超时值会影响 OS 启动时间。 如果你认为 10 秒的超时值太短或过长，请将其设置为其他值。

## <a name="use-serial-console"></a>使用串行控制台

### <a name="use-cmd-or-powershell-in-serial-console"></a>在串行控制台中使用 CMD 或 PowerShell

1. 连接到串行控制台。 如果已成功连接，则提示符为“SAC>”：

    ![连接到 SAC](./media/virtual-machines-serial-console/virtual-machine-windows-serial-console-connect-sac.png)

1.  输入 `cmd` 以创建具有 CMD 实例的通道。

1.  输入 `ch -si 1` 以切换到正在运行 CMD 实例的通道。

1.  按 Enter，然后输入具有管理权限的登录凭据。

1.  输入有效凭据后，CMD 实例随即打开。

1.  要启动 PowerShell 实例，请在 CMD 实例中输入 `PowerShell`，然后按 Enter。

    ![打开 PowerShell 实例](./media/virtual-machines-serial-console/virtual-machine-windows-serial-console-powershell.png)

### <a name="use-the-serial-console-for-nmi-calls"></a>使用串行控制台进行 NMI 调用
不可屏蔽的中断 (NMI) 旨在创建虚拟机上的软件不会忽略的信号。 过去，NMI 用来监视要求实现特定响应时间的系统上的硬件问题。 通常，今天、 程序员和系统管理员使用 NMI 作为一种机制，若要调试或故障排除系统未响应。

可以使用命令栏上的键盘图标通过串行控制台向 Azure 虚拟机发送 NMI。 传送 NMI 后，虚拟机配置将控制系统的响应方式。 可以将 Windows 配置为在收到 NMI 时崩溃并创建内存转储文件。

![发送 NMI](../media/virtual-machines-serial-console/virtual-machine-windows-serial-console-nmi.png) <br>

有关配置 Windows 在收到 NMI 时创建故障转储文件的信息，请参阅[如何使用 NMI 生成故障转储文件](https://support.microsoft.com/help/927069/how-to-generate-a-complete-crash-dump-file-or-a-kernel-crash-dump-file)。

### <a name="use-function-keys-in-serial-console"></a>在串行控制台中使用函数密钥
针对 Windows VM 中的串行控制台的使用情况，将启用函数密钥。 利用串行控制台下拉列表中的 F8 可以很方便地进入高级启动设置菜单，但串行控制台与所有其他功能键兼容。 可能需要按**Fn** + **F1** (或 F2，F3，等等) 具体取决于计算机键盘上使用串行控制台。

### <a name="use-wsl-in-serial-console"></a>在串行控制台中使用 WSL
针对 Windows Server 2019 或更高版本，已启用 Windows Subsystem for Linux (WSL)，因此如果运行 Windows Server 2019 或更高版本，也可以启用 WSL 以用于串行控制台。 这对于熟悉 Linux 命令的用户可能有所帮助。 有关为 Windows Server 启用 WSL 的说明，请参阅[安装指南](https://docs.microsoft.com/windows/wsl/install-on-server)。

### <a name="restart-your-windows-vmvirtual-machine-scale-set-instance-within-serial-console"></a>重新启动串行控制台中的将 Windows VM/虚拟机规模集实例
导航到电源按钮，然后单击"重新启动 VM"，可以启动串行控制台中的重新启动。 这将发起 VM 重启，你将在 Azure 门户中看到有关重启的通知。

这是在你可能想要启动菜单访问而无需离开串行控制台体验的情况下很有用。

![Windows 串行控制台重启](./media/virtual-machines-serial-console/virtual-machine-serial-console-restart-button-windows.gif)

## <a name="disable-serial-console"></a>禁用串行控制台
默认情况下，所有订阅为所有 VM 启用了串行控制台访问。 可以在订阅级别或 VM 级别禁用串行控制台。

### <a name="vmvirtual-machine-scale-set-level-disable"></a>VM/虚拟机规模集级别禁用
可以禁用的特定虚拟机或虚拟机规模集通过禁用启动诊断设置的串行控制台。 关闭从 Azure 门户对 VM 或虚拟机规模集禁用串行控制台的启动诊断。 如果在虚拟机规模集上使用串行控制台，请确保虚拟机规模集实例升级到最新模型。

> [!NOTE]
> 若要为订阅启用或禁用串行控制台，必须具有订阅的写入权限。 这些权限包括但不限于管理员或所有者角色。 自定义角色也可能具有写入权限。

### <a name="subscription-level-disable"></a>订阅级禁用
可以通过[禁用控制台 REST API 调用](/rest/api/serialconsole/console/disableconsole)为整个订阅禁用串行控制台。 可以使用此 API 文档页面上提供的“试用”功能为订阅禁用和启用串行控制台。 在“subscriptionId”字段中输入自己的订阅 ID，在“default”字段中输入“default”，然后选择“运行”。 Azure CLI 命令目前不可用。

![试用 REST API](../media/virtual-machines-serial-console/virtual-machine-serial-console-rest-api-try-it.png)

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
> 这意味着，断开连接的用户尚未注销。断开连接后强制注销（使用 SIGHUP 或类似机制）的功能目前仍在规划中。 对于 Windows，SAC 中会启用自动超时；对于 Linux，可以配置终端超时设置。

## <a name="accessibility"></a>辅助功能
可访问性是 Azure 串行控制台的重点。 为此，我们已确保视听障碍者以及可能无法使用鼠标的用户能够访问串行控制台。

### <a name="keyboard-navigation"></a>键盘导航
使用键盘上的 **Tab** 键在 Azure 门户中的串行控制台界面上导航。 屏幕上会突出显示你的位置。 若要使焦点离开串行控制台窗口，请在键盘上按 **Ctrl**+**F6**。

### <a name="use-the-serial-console-with-a-screen-reader"></a>结合使用串行控制台与屏幕阅读器
串行控制台内置了屏幕阅读器支持。 在打开屏幕阅读器的情况下导航，屏幕阅读器可大声读出当前所选按钮的替换文字。

## <a name="common-scenarios-for-accessing-the-serial-console"></a>要访问串行控制台的常见场景

场景          | 串行控制台中的操作
:------------------|:-----------------------------------------
错误的防火墙规则 | 访问串行控制台，并修复 Windows 防火墙规则。
文件系统损坏/检查 | 访问串行控制台并恢复文件系统。
RDP 配置问题 | 访问串行控制台并更改设置。 有关详细信息，请参阅 [RDP 文档](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/remote-desktop-allow-access)。
网络锁定系统 | 通过 Azure 门户访问串行控制台以管理系统。 [Windows 命令：CMD 和 PowerShell](serial-console-cmd-ps-commands.md) 中列出了一些网络命令。
与引导加载程序交互 | 通过串行控制台访问 BCD。 有关详细信息，请参阅[在串行控制台中启用 Windows 启动菜单](#enable-the-windows-boot-menu-in-the-serial-console)。


## <a name="errors"></a>错误
大多数错误都是暂时性的，重试连接往往可以解决。 下表显示了一系列错误和缓解措施为两个 Vm 和虚拟机规模集实例。

错误                            |   缓解措施
:---------------------------------|:--------------------------------------------|
无法检索 *&lt;VMNAME&gt;* 的启动诊断设置。 若要使用串行控制台，请确保为此 VM 启用了启动诊断。 | 确保 VM 已启用[启动诊断](boot-diagnostics.md)。
VM 处于已停止/已解除分配状态。 启动 VM，然后重试串行控制台连接。 | 虚拟机必须处于已启动状态才能访问串行控制台
没有所需的权限，无法使用此 VM 的串行控制台。 请确保至少拥有“虚拟机参与者”角色权限。| 需要特定的权限才能访问串行控制台。 有关详细信息，请参阅[先决条件](#prerequisites)。
无法确定启动诊断存储帐户 *&lt;STORAGEACCOUNTNAME&gt;* 的资源组。 确认是否为此 VM 启用了启动诊断，以及是否有权访问此存储帐户。 | 需要特定的权限才能访问串行控制台。 有关详细信息，请参阅[先决条件](#prerequisites)。
访问此 VM 的启动诊断存储帐户时遇到“已禁止”响应。 | 请确保启动诊断没有帐户防火墙。 若要使串行控制台正常运行，需要一个可访问的启动诊断存储帐户。
Web 套接字已关闭或无法打开。 | 你可能需要将 `*.console.azure.com` 加入允许列表。 一种更详细但更冗长的方法是将 [Microsoft Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)加入允许列表，该范围相当规律地定期更改。
连接到 Windows VM 时，仅显示运行状况信息| 如果尚未为 Windows 映像启用特殊管理控制台，则会发生此错误。 有关如何在 Windows VM 上手动启用 SAC 的说明，请参阅[在自定义或更低版本的映像中启用串行控制台](#enable-the-serial-console-in-custom-or-older-images)。 有关详细信息，请参阅 [Windows 运行状况信号](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Windows_Health_Info.md)。

## <a name="known-issues"></a>已知问题
我们注意到串行控制台存在的一些问题。 下面是这些问题和缓解措施的列表。 这些问题和缓解措施适用于这两个 Vm 和虚拟机规模集实例。

问题                             |   缓解措施
:---------------------------------|:--------------------------------------------|
在出现连接标题后按 **Enter** 不会显示登录提示。 | 有关详细信息，请参阅[按 Enter 不起任何作用](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)。 如果运行的自定义 VM、强化设备或启动配置导致 Windows 无法正确连接到串行端口的启动配置，则可能会发生此错误。 如果运行的是 Windows 10 客户端 VM，也会发生此错误，因为只有 Windows Server VM 配置为启用 EMS。
如果已启用内核调试，则无法在 SAC 提示符下键入内容。 | 通过 RDP 连接到 VM，并从权限提升的命令提示符运行 `bcdedit /debug {current} off`。 如果无法建立 RDP 连接，可将 OS 磁盘附加到另一个 Azure VM，并且在该磁盘附加为数据磁盘时通过运行 `bcdedit /store <drive letter of data disk>:\boot\bcd /debug <identifier> off` 对其进行修改，然后换回磁盘。
如果原始内容具有重复的字符，则粘贴到 SAC 结果中的 PowerShell 将产生第三个字符。 | 解决方法是运行 `Remove-Module PSReadLine` 以从当前会话中卸载 PSReadLine 模块。 此操作不会删除或卸载该模块。
某些键盘输入会生成奇怪的 SAC 输出（例如 [A、[3~）。 | SAC 提示符不支持 [VT100](https://aka.ms/vtsequences) 转义序列。
无法粘贴长字符串。 | 串行控制台将粘贴到终端的字符串长度限制为 2048 个字符，以防止串行端口带宽过载。
串行控制台不能与存储帐户防火墙一起使用。 | 根据设计串行控制台无法与启动诊断存储帐户上启用的存储帐户防火墙一起使用。


## <a name="frequently-asked-questions"></a>常见问题

**问：如何发送反馈？**

A. 可以通过在 https://aka.ms/serialconsolefeedback 中创建 GitHub 问题来提供反馈。 也可以通过 azserialhelp@microsoft.com，或者 https://feedback.azure.com 上的虚拟机类别发送反馈（不太建议）。

**问：串行控制台是否支持复制/粘贴？**

A. 可以。 可以使用 **Ctrl**+**Shift**+**C** 和 **Ctrl**+**Shift**+**V** 复制并粘贴到终端。

**问：谁可以为我的订阅启用或禁用串行控制台？**

A. 若要在订阅范围级别启用或禁用串行控制台，必须拥有订阅的写入权限。 拥有写入权限的角色包括管理员或所有者角色。 自定义角色也可能具有写入权限。

**问：谁可以访问 VM 的串行控制台？**

A. 必须拥有 VM 的“虚拟机参与者”或更高的角色才能访问该 VM 的串行控制台。

**问：我的串口控制台未显示任何内容，该怎么办？**

A. 你的映像可能配置错误，无法进行串行控制台访问。 有关配置映像以启用串行控制台的详细信息，请参阅[在自定义或更低版本的映像中启用串行控制台](#enable-the-serial-console-in-custom-or-older-images)。

**问：串行控制台是否可用于虚拟机规模集？**

A. 目前，不支持访问虚拟机规模集实例的串行控制台。

## <a name="next-steps"></a>后续步骤
* 有关可以在 Windows SAC 中使用的 CMD 和 PowerShell 命令的深度指南，请参阅 [Windows 命令：CMD 和 PowerShell](serial-console-cmd-ps-commands.md)。
* 串行控制台也适用于 [Linux](serial-console-linux.md) VM。
* 详细了解[启动诊断](boot-diagnostics.md)。
