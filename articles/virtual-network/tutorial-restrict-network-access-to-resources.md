---
title: 限制对 PaaS 资源的网络访问 - 教程 - Azure 门户 | Microsoft Docs
description: 本教程介绍如何使用 Azure 门户通过虚拟网络服务终结点限制对 Azure 资源（例如 Azure 存储和 Azure SQL 数据库）的网络访问。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 08/23/2018
ms.author: kumud
ms.openlocfilehash: 4d3fd152782c65c7f63e459a1c35dee6ae764361
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64708842"
---
# <a name="tutorial-restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-portal"></a>教程：使用 Azure 门户通过虚拟网络服务终结点限制对 PaaS 资源的网络访问

通过虚拟网络服务终结点，可将某些 Azure 服务资源限制为仅允许某个虚拟网络子网通过网络进行访问。 还可以删除对资源的 Internet 访问。 服务终结点提供从虚拟网络到受支持 Azure 服务的直接连接，使你能够使用虚拟网络的专用地址空间访问 Azure 服务。 通过服务终结点发往 Azure 资源的流量始终保留在 Microsoft Azure 主干网络上。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建包含一个子网的虚拟网络
> * 添加子网并启用服务终结点
> * 创建 Azure 资源并且仅允许从一个子网对其进行网络访问
> * 将虚拟机 (VM) 部署到每个子网
> * 确认从某个子网对资源的访问
> * 确认已拒绝从某个子网和 Internet 来访问资源

如果你愿意，可以使用 [Azure CLI](tutorial-restrict-network-access-to-resources-cli.md) 或 [Azure PowerShell](tutorial-restrict-network-access-to-resources-powershell.md) 完成本教程中的步骤。

如果没有 Azure 订阅，请在开始之前创建一个 [免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="log-in-to-azure"></a>登录 Azure

通过 https://portal.azure.com 登录到 Azure 门户。

## <a name="create-a-virtual-network"></a>创建虚拟网络

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“网络”，然后选择“虚拟网络”。
3. 输入或选择以下信息，然后选择“创建”：

   |设置|值|
   |----|----|
   |名称| myVirtualNetwork |
   |地址空间| 10.0.0.0/16|
   |订阅| 选择订阅|
   |资源组 | 选择“新建”，并输入 myResourceGroup|
   |位置| 选择“美国东部” |
   |子网名称| 公共|
   |子网地址范围| 10.0.0.0/24|
   |服务终结点| 已禁用|

   ![输入虚拟网络的基本信息](./media/tutorial-restrict-network-access-to-resources/create-virtual-network.png)

## <a name="enable-a-service-endpoint"></a>启用服务终结点

每个服务、每个子网均启用服务终结点。 创建子网并为该子网启用服务终结点。

1. 在门户顶部的“搜索资源、服务和文档”框中，输入 *myVirtualNetwork*。 当“myVirtualNetwork”出现在搜索结果中时，将其选中。
2. 将子网添加到虚拟网络。 在“设置”下面选择“子网”，然后选择“+ 子网”，如下图中所示：

    ![添加子网](./media/tutorial-restrict-network-access-to-resources/add-subnet.png) 

3. 在“添加子网”下，选择或输入以下信息，然后选择“确定”：

    |设置|值|
    |----|----|
    |名称| 专用 |
    |地址范围| 10.0.1.0/24|
    |服务终结点| 在“服务”下选择“Microsoft.Storage”|

> [!CAUTION]
> 在为其中有资源的现有子网启用服务终结点之前，请参阅[更改子网设置](virtual-network-manage-subnet.md#change-subnet-settings)。

## <a name="restrict-network-access-for-a-subnet"></a>限制子网的网络访问

默认情况下，子网中的所有虚拟机都可以与所有资源通信。 可以通过创建网络安全组并将其关联到子网来限制与子网中所有资源的通信。

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 依次选择“网络”、“网络安全组”。
3. 在“创建网络安全组”下，输入或选择以下信息，然后选择“创建”：

    |设置|值|
    |----|----|
    |名称| myNsgPrivate |
    |订阅| 选择订阅|
    |资源组 | 选择“使用现有资源组”，再选择“myResourceGroup”。|
    |位置| 选择“美国东部” |

4. 创建网络安全组后，在门户顶部的“搜索资源、服务和文档”框中输入 *myNsgPrivate*。 当“myNsgPrivate”出现在搜索结果中时，将其选中。
5. 在“设置”下，选择“出站安全规则”。
6. 选择“+ 添加”。
7. 创建一条允许出站通信到 Azure 存储服务的规则。 输入或选择以下信息，然后选择“添加”：

    |设置|值|
    |----|----|
    |源| 选择“VirtualNetwork” |
    |源端口范围| * |
    |目标 | 选择“服务标记”|
    |目标服务标记 | 选择“存储”|
    |目标端口范围| * |
    |协议|任意|
    |操作|允许|
    |优先度|100|
    |名称|Allow-Storage-All|

8. 创建另一条出站安全规则，拒绝到 Internet 的通信。 此规则将覆盖所有网络安全组中允许出站 Internet 通信的默认规则。 使用以下值再次完成步骤 5-7：

    |设置|值|
    |----|----|
    |源| 选择“VirtualNetwork” |
    |源端口范围| * |
    |目标 | 选择“服务标记”|
    |目标服务标记| 选择“Internet”|
    |目标端口范围| * |
    |协议|任意|
    |操作|拒绝|
    |优先度|110|
    |名称|Deny-Internet-All|

9. 在“设置”下，选择“入站安全规则”。
10. 选择“+ 添加”。
11. 创建一个允许从任何位置向该子网发送远程桌面协议 (RDP) 流量的入站安全规则。 该规则将替代拒绝来自 Internet 的所有入站流量的默认安全规则。 允许与子网建立远程桌面连接，以便可以在后续步骤中测试连接。 在“设置”下选择“入站安全规则”，然后选择“+ 添加”，输入以下值，然后选择“添加”：

    |设置|值|
    |----|----|
    |源| 任意 |
    |源端口范围| * |
    |目标 | 选择“VirtualNetwork”|
    |目标端口范围| 3389 |
    |协议|任意|
    |操作|允许|
    |优先度|120|
    |名称|Allow-RDP-All|

12. 在“设置”下选择“子网”。
13. 选择“+ 关联”
14. 在“关联子网”下，选择“虚拟网络”，然后在“选择虚拟网络”下选择“myVirtualNetwork”。
15. 在“选择子网”下选择“专用”，然后选择“确定”。

## <a name="restrict-network-access-to-a-resource"></a>限制对资源的网络访问

对于通过为服务终结点启用的 Azure 服务创建的资源，限制对其的网络访问时所需的步骤因服务而异。 请参阅各个服务的文档来了解适用于每个服务的具体步骤。 作为示例，本教程的剩余部分包括了针对 Azure 存储帐户限制网络访问的步骤。

### <a name="create-a-storage-account"></a>创建存储帐户

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“存储”，然后选择“存储帐户 - blob、文件、表、队列”。
3. 输入或选择以下信息，接受剩下的默认设置，然后选择“创建”：

    |设置|值|
    |----|----|
    |名称| 输入在所有 Azure 位置中唯一的、长度为 3-24 个字符且仅使用数字和小写字母的名称。|
    |帐户类型|StorageV2（常规用途 v2）|
    |位置| 选择“美国东部” |
    |复制| 本地冗余存储 (LRS)|
    |订阅| 选择订阅|
    |资源组 | 选择“使用现有资源组”，再选择“myResourceGroup”。|

### <a name="create-a-file-share-in-the-storage-account"></a>在存储帐户中创建文件共享

1. 创建存储帐户后，在门户顶部的“搜索资源、服务和文档”框中输入该存储帐户的名称。 该存储帐户的名称出现在搜索结果中时，请选择它。
2. 选择“文件”，如下图所示：

   ![存储帐户](./media/tutorial-restrict-network-access-to-resources/storage-account.png) 

3. 选择“+ 文件共享”。
4. 在“名称”下输入 *my-file-share*，然后选择“确定”。
5. 关闭“文件服务”框。

### <a name="restrict-network-access-to-a-subnet"></a>限制对子网的网络访问

默认情况下，存储帐户接受来自任何网络（包括 Internet）中的客户端的网络连接。 除 *myVirtualNetwork* 虚拟网络中的“专用”子网之外，拒绝来自 Internet 以及所有虚拟网络中的所有其他子网的网络访问。

1. 在存储帐户的“设置”下，选择“防火墙和虚拟网络”。
2. 选择“所选网络”。
3. 选择“+添加现有虚拟网络”。
4. 在“添加网络”下选择以下值，然后选择“添加”：

    |设置|值|
    |----|----|
    |订阅| 选择订阅。|
    |虚拟网络|选择“虚拟网络”下的“myVirtualNetwork”|
    |子网| 选择“子网”下的“专用”|

    ![防火墙和虚拟网络](./media/tutorial-restrict-network-access-to-resources/storage-firewalls-and-virtual-networks.png)

5. 选择“保存”。
6. 选中“防火墙和虚拟网络”框。
7. 在存储帐户的“设置”下，选择“访问密钥”，如下图所示：

      ![防火墙和虚拟网络](./media/tutorial-restrict-network-access-to-resources/storage-access-key.png)

8. 记下“密钥”值，因为在后续步骤中将文件共享映射到 VM 中的驱动器号时，需要手动输入该值。

## <a name="create-virtual-machines"></a>创建虚拟机

若要测试对存储帐户的网络访问，请向每个子网部署 VM。

### <a name="create-the-first-virtual-machine"></a>创建第一个虚拟机

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“计算”，然后选择“Windows Server 2016 Datacenter”。
3. 输入或选择以下信息，然后选择“确定”：

   |设置|值|
   |----|----|
   |名称| myVmPublic|
   |用户名|输入所选用户名。|
   |密码| 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
   |订阅| 选择订阅。|
   |资源组| 选择“使用现有资源组”，再选择“myResourceGroup”。|
   |位置| 选择“美国东部”。|

   ![输入虚拟机的基本信息](./media/tutorial-restrict-network-access-to-resources/virtual-machine-basics.png)
4. 选择虚拟机的大小，然后选择“选择”。
5. 在“设置”下选择“网络”，然后选择“myVirtualNetwork”。 然后依次选择“子网”、“公共”，如下图中所示：

   ![选择虚拟网络](./media/tutorial-restrict-network-access-to-resources/virtual-machine-settings.png)

6. 在“网络安全组”下，选择“高级”。 门户会自动为你创建一个网络安全组，该组允许端口 3389。此端口需保持打开状态，然后才能在后面的步骤中连接到虚拟机。 在“设置”页中，选择“确定”。
7. 在“摘要”页上，选择“创建”以启动虚拟机部署。 部署 VM 需要几分钟时间，但在创建 VM 期间，可以继续执行下一步骤。

### <a name="create-the-second-virtual-machine"></a>创建第二个虚拟机

再次完成步骤 1-7，但在步骤 3 中，请将虚拟机命名为 *myVmPrivate*，在步骤 5 中选择“专用”子网。

部署 VM 需要几分钟时间。 只有在创建完 VM 并且其设置已在门户中打开后，才继续下一步。

## <a name="confirm-access-to-storage-account"></a>确认对存储帐户的访问

1. 创建完 *myVmPrivate* VM 之后，Azure 会打开其设置。 选择“连接”按钮连接到 VM，如下图所示：

   ![连接到虚拟机](./media/tutorial-restrict-network-access-to-resources/connect-to-virtual-machine.png)

2. 选择“连接”按钮后将创建一个远程桌面协议 (.rdp) 文件，该文件会被下载到你的计算机。  
3. 打开下载的 rdp 文件。 出现提示时，选择“连接”。 输入在创建 VM 时指定的用户名和密码。 可能需要选择“更多选择”，然后选择“使用其他帐户”，以指定在创建 VM 时输入的凭据。 
4. 选择“确定”。
5. 你可能会在登录过程中收到证书警告。 如果收到警告，请选择“是”或“继续”以继续连接。
6. 在 *myVmPrivate* VM 上，使用 PowerShell 将 Azure 文件共享映射到驱动器 Z。 在运行以下命令之前，请将 `<storage-account-key>` 和 `<storage-account-name>` 替换为在[创建存储帐户](#create-a-storage-account)中提供或检索的值。

   ```powershell
   $acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
   $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
   New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
   ```

   PowerShell 将返回类似于以下示例的输出：

   ```powershell
   Name           Used (GB)     Free (GB) Provider      Root
   ----           ---------     --------- --------      ----
   Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
   ```

   Azure 文件共享已成功映射到驱动器 Z。

7. 通过命令提示符确认 VM 没有与 Internet 建立出站连接：

   ```
   ping bing.com
   ```

   不会收到回复，因为关联到“专用”子网的网络安全组不允许与 Internet 建立出站访问。

8. 关闭与 *myVmPrivate* VM 建立的远程桌面会话。

## <a name="confirm-access-is-denied-to-storage-account"></a>确认已拒绝对存储帐户的访问

1. 在门户顶部的“搜索资源、服务和文档”框中，输入 *myVmPublic*。
2. 当“myVmPublic”出现在搜索结果中时，将其选中。
3. 针对 *myVmPublic* VM 完成[确认对存储帐户的访问](#confirm-access-to-storage-account)中的步骤 1-6。

   稍等片刻，你会收到 `New-PSDrive : Access is denied` 错误。 访问被拒绝，因为 *myVmPublic* VM 部署在“公共”子网中。 “公共”子网没有为 Azure 存储启用服务终结点。 存储帐户仅允许从“专用”子网访问网络，而不允许从“公共”子网访问。

4. 关闭与 *myVmPublic* VM 建立的远程桌面会话。

5. 在计算机上浏览到 Azure [门户](https://portal.azure.com)。
6. 在“搜索资源、服务和文档”框中输入创建的存储帐户的名称。 该存储帐户的名称出现在搜索结果中时，请选择它。
7. 选择“文件”。
8. 会收到下图所示的错误：

   ![访问被拒绝错误](./media/tutorial-restrict-network-access-to-resources/access-denied-error.png)

   访问被拒绝，因为计算机不在 *MyVirtualNetwork* 虚拟网络的“专用”子网中。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，可将资源组及其包含的所有资源一并删除：

1. 在门户顶部的“搜索”框中输入“myResourceGroup”。 当在搜索结果中看到“myResourceGroup”时，将其选中。
2. 选择“删除资源组”。
3. 对于“键入资源组名称:”，输入“myResourceGroup”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

在本教程中，我们为虚拟网络子网启用了服务终结点。 我们已了解，可为通过多个 Azure 服务部署的资源启用服务终结点。 已创建了一个 Azure 存储帐户并将该存储帐户限制为仅可供某个虚拟网络子网中的资源进行网络访问。 若要详细了解服务终结点，请参阅[服务终结点概述](virtual-network-service-endpoints-overview.md)和[管理子网](virtual-network-manage-subnet.md)。

如果帐户中有多个虚拟网络，可将两个虚拟网络连接到一起，使每个虚拟网络中的资源可以相互通信。 若要了解如何连接虚拟网络，请继续学习下一教程。

> [!div class="nextstepaction"]
> [连接虚拟网络](./tutorial-connect-virtual-networks-portal.md)
