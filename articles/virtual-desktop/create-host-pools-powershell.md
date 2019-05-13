---
title: 使用 PowerShell 的 Azure 中创建 Windows 虚拟桌面预览主机池
description: 如何使用 PowerShell cmdlet 在 Windows 虚拟桌面预览中创建主机池。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/06/2019
ms.author: helohr
ms.openlocfilehash: 165980da2b78e7514ea1938bdbeb6fda82dd123d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236661"
---
# <a name="create-a-host-pool-with-powershell"></a>使用 PowerShell 创建主机池

主机池是 Windows 虚拟桌面预览版租户环境中一个或多个相同虚拟机的集合。 每个主机池可以包含一个应用组，用户可以像在物理桌面上一样与该应用组交互。

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>使用 PowerShell 客户端创建主机池

首先[下载并导入 Windows 虚拟桌面 PowerShell 模块](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview)（如果尚未这样做），以便在 PowerShell 会话中使用。

运行以下 cmdlet，以登录到 Windows 虚拟桌面环境

```powershell
Add-RdsAccount -DeploymentUrl https://rdbroker.wvd.microsoft.com
```

然后，运行以下 cmdlet，以将上下文设置为你的租户组。 如果你没有租户组的名称，你的租户是很可能出在"默认租户组中,"，因此可以跳过此 cmdlet。

```powershell
Set-RdsContext -TenantGroupName <tenantgroupname>
```

接下来，运行此 cmdlet 可在 Windows 虚拟桌面租户中创建新的主机池：

```powershell
New-RdsHostPool -TenantName <tenantname> -Name <hostpoolname>
```

运行下一个 cmdlet 以创建注册令牌以授权会话主机加入主机池并将其保存到本地计算机上的新文件。 您可以指定多长时间的注册令牌无效，通过使用-ExpirationHours 参数。

```powershell
New-RdsRegistrationInfo -TenantName <tenantname> -HostPoolName <hostpoolname> -ExpirationHours <number of hours> | Select-Object -ExpandProperty Token > <PathToRegFile>
```

然后，运行此 cmdlet 可将 Azure Active Directory 用户添加到主机池的默认桌面应用程序组。

```powershell
Add-RdsAppGroupUser -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName "Desktop Application Group" -UserPrincipalName <userupn>
```

**添加 RdsAppGroupUser** cmdlet 不支持添加安全组，并仅将一个用户一次添加到应用程序组。 如果你想要将多个用户添加到应用程序组，请重新运行 cmdlet 和相应的用户主体名称。

运行以下 cmdlet 将导出到一个变量，将更高版本中使用的注册令牌[注册到 Windows 虚拟机主机池的虚拟机](#register-the-virtual-machines-to-the-windows-virtual-desktop-preview-host-pool)。

```powershell
$token = (Export-RdsRegistrationInfo -TenantName <tenantname> -HostPoolName <hostpoolname>).Token
```

## <a name="create-virtual-machines-for-the-host-pool"></a>为主机池创建虚拟机

现在可以创建 Azure 虚拟机可以加入到 Windows 虚拟机主机池。

可以通过多种方式来创建虚拟机：

- [从 Azure 库映像创建的虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#create-virtual-machine)
- [从托管映像创建虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-generalized-managed)
- [从非托管映像创建虚拟机](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-preview-agent-installations"></a>对于 Windows 虚拟桌面预览代理安装准备的虚拟机

需要执行以下操作来准备虚拟机，然后可以安装的 Windows 虚拟桌面代理并注册到 Windows 虚拟机主机池的虚拟机：

- 必须在计算机加入域。 这样，传入的 Windows 虚拟桌面用户若要从其 Azure Active Directory 帐户映射到其 Active Directory 帐户，并成功地允许对虚拟机的访问。
- 如果虚拟机正在运行 Windows Server 操作系统，必须安装远程桌面会话主机 (RDSH) 角色。 RDSH 角色允许的 Windows 虚拟桌面代理才能正常安装。

已成功加入域，请执行以下操作在每个虚拟机上的操作：

1. [连接到虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine)时创建虚拟机时提供的凭据。
2. 虚拟机上启动**Control Panel** ，然后选择**系统**。
3. 选择**计算机名**，选择**更改设置**，然后选择**更改...**
4. 选择**域**，然后输入虚拟网络上的 Active Directory 域。
5. 使用有权限将计算机加入域的域帐户进行身份验证。

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-preview-host-pool"></a>注册到 Windows 虚拟桌面预览主机池的虚拟机

注册到 Windows 虚拟机主机池的虚拟机是作为安装的 Windows 虚拟桌面代理一样简单。

若要注册的 Windows 虚拟机代理，请执行以下操作在每个虚拟机上：

1. [连接到虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine)时创建虚拟机时提供的凭据。
2. 下载并安装 Windows 的虚拟桌面代理。
   - 下载[Windows 的虚拟桌面代理](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv)。
   - 右键单击下载的安装程序中，选择**属性**，选择**解除阻止**，然后选择**确定**。 这将允许您的系统信任安装程序。
   - 运行安装程序。 当安装程序要求您注册令牌时，输入时所获取的值**导出 RdsRegistrationInfo** cmdlet。
3. 下载并安装 Windows 的虚拟桌面代理启动程序。
   - 下载[Windows 虚拟桌面代理引导加载程序](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH)。
   - 右键单击下载的安装程序中，选择**属性**，选择**解除阻止**，然后选择**确定**。 这将允许您的系统信任安装程序。
   - 运行安装程序。

>[!IMPORTANT]
>为了帮助保护 Azure 中的 Windows 虚拟桌面环境，我们建议你不要在 VM 上打开入站端口 3389。 Windows 虚拟机不需要打开入站端口 3389，用户就可以访问主机池的 VM。 如果必须打开端口 3389 以进行故障排除，我们建议你使用[实时 VM 访问](https://docs.microsoft.com/azure/security-center/security-center-just-in-time)。

## <a name="next-steps"></a>后续步骤

现在，所做的主机池，可以用 RemoteApps 填充它。 若要详细了解如何在 Windows 虚拟桌面中管理应用，请参阅“管理应用组”教程。

> [!div class="nextstepaction"]
> [管理应用组教程](./manage-app-groups.md)
