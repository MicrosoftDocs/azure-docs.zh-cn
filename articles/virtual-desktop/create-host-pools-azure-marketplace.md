---
title: 使用 Azure 市场创建 Windows 虚拟桌面预览版主机池 - Azure
description: 如何使用 Azure 市场创建 Windows 虚拟桌面预览版主机池。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: tutorial
ms.date: 04/05/2019
ms.author: helohr
ms.openlocfilehash: c4ab209d63a01d9e1089eec351eff64fc116f403
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64924978"
---
# <a name="tutorial-create-a-host-pool-with-azure-marketplace"></a>教程：通过 Azure 市场创建主机池

主机池是 Windows 虚拟桌面预览版租户环境中一个或多个相同虚拟机的集合。 每个主机池可以包含一个应用组，用户可以像在物理桌面上一样与该应用组交互。

本文介绍如何使用 Microsoft Azure 市场套餐在 Windows 虚拟桌面租户中创建主机池。 这包括在 Windows 虚拟桌面中创建主机池，使用 Azure 订阅中的 VM 创建资源组，将这些 VM 加入 Active Directory 域，然后将 VM 注册到 Windows 虚拟桌面。

在开始之前，如果尚未这样做，请[下载并导入 Windows 虚拟桌面 PowerShell 模块](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview)，以便在 PowerShell 会话中使用。

## <a name="sign-in-to-azure"></a>登录 Azure

通过 <https://portal.azure.com> 登录到 Azure 门户。

## <a name="run-the-azure-marketplace-offering-to-provision-a-new-host-pool"></a>运行 Azure 市场套餐以预配新的主机池

若要运行 Azure 市场套餐以预配新的主机池，请执行以下操作：

1. 选择 **+** 或“+ 创建资源”。
2. 在市场搜索窗口中输入“Windows 虚拟桌面”。
3. 依次选择“Windows 虚拟桌面 - 预配主机池”、“创建”。

遵照指导在相应的边栏选项卡中输入信息。

### <a name="basics"></a>基础

在“基本信息”边栏选项卡中执行以下操作：

1. 输入主机池的名称，该名称在 Windows 虚拟桌面租户中必须唯一。
2. 选择适合个人桌面的选项。 如果选择“是”，则连接到此主机池的每个用户将永久分配到某个虚拟机。
3. 输入在运行完 Azure 市场套餐后可以登录到 Windows 虚拟桌面客户端并可以访问桌面的用户的逗号分隔列表。 例如，若要向 user1@contoso.com 和 user2@contoso.com 分配访问权限，请输入“user1@contoso.com,user2@contoso.com”。
4. 选择“新建”，并为新资源组提供一个名称。
5. 对于“位置”，请选择连接到 Active Directory 服务器的虚拟网络所在的同一位置。
6. 选择“确定”。

### <a name="configure-virtual-machines"></a>配置虚拟机

在“配置虚拟机”边栏选项卡中：

1. 接受默认设置，或自定义 VM 的数量和大小。
2. 输入虚拟机名称的前缀。 例如，如果输入“prefix”作为名称，则虚拟机的名称将是“prefix-0”、“prefix-1”，依此类推。
3. 选择“确定”。

### <a name="virtual-machine-settings"></a>虚拟机设置

在“虚拟机设置”边栏选项卡中：

1. 选择“映像源”，并输入有关如何找到和存储该映像的相应信息。 如果选择不使用托管磁盘，请选择包含 .vhd 文件的存储帐户。
2. 输入用于将 VM 加入 Active Directory 域的域帐户的用户主体名称和密码。 在虚拟机上将会创建与此相同的用户名和密码作为本地帐户。 以后可以重置这些本地帐户。
3. 选择连接到 Active Directory 服务器的虚拟网络，然后选择一个子网用于托管虚拟机。
4. 选择“确定”。

### <a name="windows-virtual-desktop-preview-tenant-information"></a>Windows 虚拟桌面预览版租户信息

在“Windows 虚拟桌面租户信息”边栏选项卡中：

1. 输入包含你的租户的租户组的 **Windows 虚拟桌面租户组名称**。 如果你尚未想好具体的租户组名称，请将此字段保留默认值。
2. 输入要在其中创建此主机池的租户的 **Windows 虚拟桌面租户名称**。
3. 指定以 Windows 虚拟桌面租户 RDS 所有者身份进行身份验证时要使用的凭据类型。 如果选择“服务主体”，则还必须提供与服务主体关联的“Azure AD 租户 ID”。
4. 另请输入租户管理员帐户的凭据。 仅支持具有密码凭据的服务主体。
5. 选择“确定”。

## <a name="complete-setup-and-create-the-virtual-machine"></a>完成设置并创建虚拟机

对于最后两个边栏选项卡：

1. 在“摘要”边栏选项卡中检查设置信息。 如果需要更改设置，请返回相应的边栏选项卡进行更改，然后继续。 如果信息正确，请选择“确定”。
2. 在“购买”边栏选项卡中，检查有关在 Azure 市场中购买的内容的附加信息。
3. 选择“创建”以部署主机池。

此过程可能需要 30 分钟或更长时间才能完成，具体取决于创建的 VM 有多少。

## <a name="optional-assign-additional-users-to-the-desktop-application-group"></a>（可选）将其他用户分配到桌面应用程序组

运行完 Azure 市场套餐后，可将其他用户分配到桌面应用程序组，然后在虚拟机上开始测试完整的会话桌面。 如果已在 Azure 市场套餐中添加默认用户，并且不想要添加其他用户，则可以跳过本部分。

若要将用户分配到桌面应用程序组，首先必须打开 PowerShell 窗口。 然后，需要输入以下两个 cmdlet。

运行以下 cmdlet 登录到 Windows 虚拟桌面环境：

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

使用以下 cmdlet 将上下文设置为在 Azure 市场套餐中指定的 Windows 虚拟桌面租户组。 如果已在 Azure 市场套餐中将“Windows 虚拟桌面租户组”值保留为默认值，则可以跳过此步骤。

```powershell
Set-RdsContext -TenantGroupName <tenantgroupname>
```

完成这两项操作后，可使用以下 cmdlet 将用户添加到桌面应用程序组：

```powershell
Add-RdsAppGroupUser <tenantname> <hostpoolname> "Desktop Application Group" -UserPrincipalName <userupn>
```

用户的 UPN 应与该用户在 Azure Active Directory 中的标识（例如 user1@contoso.com）相匹配。 若要添加多个用户，必须针对每个用户运行此 cmdlet。

完成这些步骤后，添加到桌面应用程序组的用户可以使用支持的远程桌面客户端登录到 Windows 虚拟桌面，并且可以看到会话桌面的资源。

当前支持的客户端如下：

- [适用于 Windows 7 和 Windows 10 的远程桌面客户端](connect-windows-7-and-10.md)
- [Windows 虚拟桌面 Web 客户端](connect-web.md)

>[!IMPORTANT]
>为了帮助保护 Azure 中的 Windows 虚拟桌面环境，我们建议你不要在 VM 上打开入站端口 3389。 Windows 虚拟机不需要打开入站端口 3389，用户就可以访问主机池的 VM。 如果必须打开端口 3389 以进行故障排除，我们建议你使用[实时 VM 访问](https://docs.microsoft.com/azure/security-center/security-center-just-in-time)。

## <a name="next-steps"></a>后续步骤

创建主机池并分配可访问其桌面的用户后，还可以使用 RemoteApps 来填充主机池。 若要详细了解如何在 Windows 虚拟桌面中管理应用，请参阅“管理应用组”教程。

> [!div class="nextstepaction"]
> [管理应用组教程](./manage-app-groups.md)
