---
title: Azure 的 VMware 解决方案 CloudSimple 快速入门-创建私有云
description: 了解如何创建和配置具有 Azure CloudSimple VMware 解决方案的私有云
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e1fc7809ad94d589483b87c638d027a39098164e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209537"
---
# <a name="quickstart---configure-a-private-cloud-environment"></a>快速入门-配置私有云环境

在本文中，了解如何创建 CloudSimple 私有云和设置私有云环境。

## <a name="sign-in-to-azure"></a>登录 Azure
在 [https://portal.azure.com](https://portal.azure.com) 中登录 Azure 门户。

## <a name="create-a-private-cloud"></a>创建私有云

1. 选择“所有服务”。
2. 搜索**CloudSimple 服务**。
3. 选择你想要创建您的私有云的 CloudSimple 服务。
4. 从概述中，单击**创建私有云**打开新 CloudSimple 门户的浏览器选项卡。  如果系统提示，请使用你的 Azure 登录在凭据登录。  

    ![从 Azure 中创建私有云](media/create-private-cloud-from-azure.png)

5. 在 CloudSimple 门户中，提供您的私有云的名称
6. 选择**位置**的您的私有云
7. 选择**节点类型**购买在 Azure 上。  你可以选择[CS28 或 CS36 选项](cloudsimple-node.md#vmware-solution-by-cloudsimple-nodes-sku)。 后一种包含的最大的计算和内存容量。
8. 指定**节点计数**。  创建私有云所需的最少三个节点

    ![创建私有云-基本信息](media/create-private-cloud-basic-info.png)

9. 单击“下一步:**高级选项**。
10. 输入 vSphere/vSAN 子网的 CIDR 范围。 请确保 CIDR 范围不与任何本地或其他 Azure 子网重叠。

    ![创建私有云-高级选项](media/create-private-cloud-advanced-options.png)

11. 在完成时选择“下一步:**查看和创建**。
12. 查看设置。 如果需要更改任何设置，请单击**上一步**。
13. 单击**创建**。

私有云预配过程将会启动。  可能需要最多两个小时，将其预配的私有云。

## <a name="launch-cloudsimple-portal"></a>启动 CloudSimple 门户

可以从 Azure 门户来访问 CloudSimple 门户。  将你的 Azure 启动 CloudSimple 门户中使用单一登录 (SSO) 的凭据登录。  访问 CloudSimple 门户需要进行授权**CloudSimple 服务授权**应用程序。  有关授予权限的详细信息，请参阅[同意 CloudSimple 服务授权应用程序](https://docs.azure.cloudsimple.com/access-cloudsimple-portal/#consent-to-cloudsimple-service-authorization-application)

1. 选择“所有服务”。
2. 搜索**CloudSimple 服务**。
3. 选择你想要创建您的私有云的 CloudSimple 服务。
4. 从概述中，单击**转到 CloudSimple 门户**打开新 CloudSimple 门户的浏览器选项卡。  如果系统提示，请使用你的 Azure 登录在凭据登录。  

    ![启动 CloudSimple 门户](media/launch-cloudsimple-portal.png)

## <a name="create-point-to-site-vpn"></a>创建点到站点 VPN

点到站点 VPN 连接是从您的计算机连接到您的私有云的最简单方法。 如果要远程连接到私有云，请使用点到站点 VPN 连接。  您的私有云快速访问，请执行以下步骤。  可以从你的本地网络访问 CloudSimple 区域使用[站点到站点 VPN](https://docs.azure.cloudsimple.com/vpn-gateway/)或[Azure ExpressRoute](https://docs.azure.cloudsimple.com/on-premises-connection/)。

### <a name="create-gateway"></a>创建网关

1. 启动 CloudSimple 门户并选择**网络**。
2. 选择**VPN 网关**。
3. 单击**新的 VPN 网关**。

    ![创建 VPN 网关](media/create-vpn-gateway.png)

4. 有关**网关配置**，指定以下设置，然后单击**下一步**。

    * 选择**点到站点 VPN**作为网关类型。
    * 输入一个名称以标识该网关。
    * 选择部署 CloudSimple 服务所在的 Azure 位置。
    * 指定的点到站点网关的客户端子网。  在连接时，将来自此子网提供 DHCP 地址。

5. 有关**用户的连接/**，指定以下设置，然后单击**下一步**。

    * 若要自动允许所有当前和未来用户通过此点到站点网关访问私有云，请选择**自动添加的所有用户**。 如果选择此选项时，会自动选择的用户列表中的所有用户。 可以通过取消选中列表中的各个用户覆盖自动选项。
    * 若要选择只有单个用户，请单击用户列表中的复选框。

6. Vlan/子网部分允许您指定管理和网关和连接的用户 Vlan/子网。

    * **自动添加**选项将此网关的全局策略设置。 设置应用于当前网关。 设置可以在中重写**选择**区域。
    * 选择**添加管理 Vlan/子网的私有云**。 
    * 若要添加所有用户定义的 Vlan/子网，请单击**添加用户定义的 Vlan/子网**。 
    * **选择**设置重写下的全局设置**自动添加**。 

7. 单击**下一步**可查看的设置。 单击编辑图标，需进行任何更改。
8. 单击**创建**创建 VPN 网关。

### <a name="connect-to-cloudsimple-using-point-to-site-vpn"></a>连接到 CloudSimple 使用点到站点 VPN

VPN 客户端需要从您的计算机连接到 CloudSimple。  下载[OpenVPN 客户端](https://openvpn.net/community-downloads/)的 Windows 或[粘度](https://www.sparklabs.com/viscosity/download/)对于 macOS 和 OS X。

1. 启动 CloudSimple 门户并选择**网络**。
2. 选择**VPN 网关**。
3. 从 VPN 网关列表中，单击点到站点 VPN 网关。
4. 选择“用户”。
5. 单击**下载我的 VPN 配置**

    ![下载 VPN 配置](media/download-p2s-vpn-configuration.png)

6. 导入 VPN 客户端上的配置

    * 有关说明[导入 Windows 客户端上的配置](https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-windows/#openvpn-open-source-openvpn-gui-program)
    * 有关说明[导入在 macOS 或 OS X 上的配置](https://www.sparklabs.com/support/kb/article/getting-started-with-viscosity-mac/#creating-your-first-connection)

7. 连接到 CloudSimple

## <a name="create-a-vlan-for-your-workload-vms"></a>为你的工作负荷的 Vm 创建的 VLAN

创建私有云之后, 创建将在其中部署你的工作负荷/应用程序 Vm 的 VLAN。

1. 在 CloudSimple 门户中，选择**网络**。
2. 单击**VLAN/子网**。
3. 单击**创建 VLAN/子网**

    ![创建 VLAN/子网](media/create-new-vlan-subnet.png)

4. 选择**私有云**新 VLAN/子网。
5. 从列表中选择一个 VLAN ID。  
6. 输入用于标识子网的子网名称。
7. 指定的子网 CIDR 范围和掩码。  此范围不得与任何现有的子网重叠。
8. 单击“提交”。

    ![创建子网 VLAN/详细信息](media/create-new-vlan-subnet-details.png)

将创建的 VLAN/子网。  现在可以使用此 VLAN ID 在私有云 vCenter 上创建分布式的端口组。 

## <a name="connect-your-environment-to-an-azure-virtual-network"></a>将环境连接到 Azure 虚拟网络

CloudSimple 提供了一条 ExpressRoute 线路的私有云。 您可以在 Azure 上的将虚拟网络连接到 ExpressRoute 线路。 设置该连接的完整详细信息，请按照中的步骤[Azure 虚拟网络连接，使用 ExpressRoute](https://docs.azure.cloudsimple.com/cloudsimple-azure-network-connection/)

## <a name="sign-in-to-vcenter"></a>登录到 vCenter

您可以立即登录到 vCenter 设置虚拟机和策略。

1. 若要访问 vCenter，请从 CloudSimple 门户启动。 在主页页面下**常见任务**，单击**启动 vSphere 客户端**。  选择私有云，然后单击**启动 vSphere 客户端**私有云。

    ![启动 vSphere 客户端](media/launch-vcenter-from-cloudsimple-portal.png)

2. 选择首选的 vSphere 客户端访问 vCenter 和使用你的用户名和密码登录。  默认值如下：
    * 用户名： **CloudOwner@cloudsimple.local**
    * 密码:**CloudSimple123!**  

在下一步的过程中的 vCenter 屏幕都是从 vSphere (HTML5) 客户端。

## <a name="change-your-vcenter-password"></a>更改 vCenter 密码

CloudSimple 建议您在第一次登录到 vCenter 更改密码。  
设置的密码必须满足以下要求：

* 最长生存期：密码必须更改每个 365 天
* 限制重复使用：用户不能重复使用任何以前的五个密码
* 时长：8-20 个字符
* 特殊字符：至少 1 个特殊字符
* 字母字符：至少一个大写字符、 A-Z 和至少一个小写字符 a 到 z
* 数字：至少一个数字字符，0-9
* 最大相同的相邻字符：三

    示例：抄送或 CCC 可接受的密码，一部分，但不是 CCCC。

如果设置不符合要求的密码：

* 如果使用 vSphere Flash 客户端，则将报告错误
* 如果使用 HTML5 客户端，它不会报告错误。 客户端不会接受更改并继续正常运行的旧密码。

## <a name="change-nsx-administrator-password"></a>更改 NSX 管理员密码

使用默认密码，NSX manager 进行部署。  我们建议在创建您的私有云后更改密码。

   * 用户名：**管理员**
   * 密码:**CloudSimple123!**

可以在 CloudSimple 门户上找到的完全限定的域名 (FQDN) 和 NSX 管理器的 IP 地址。

1. 启动 CloudSimple 门户并选择**资源**。
2. 单击你想要使用私有云。
3. 选择**vSphere 管理网络**
4. 使用的 FQDN 或 IP 地址**NSX Manager** ，使用 web 浏览器进行连接。 

    ![查找 NSX Manager FQDN](media/private-cloud-nsx-manager-fqdn.png)

若要更改的密码，请按照中的说明[NSX 管理器安装](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.2/com.vmware.nsxt.install.doc/GUID-A65FE3DD-C4F1-47EC-B952-DEDF1A3DD0CF.html)。

## <a name="create-a-port-group"></a>创建端口组

在 vSphere 中创建的分布式的端口组：

1. 按照中的"添加分布式的端口组"中的说明[vSphere 网络指南](https://docs.vmware.com/en/VMware-vSphere/6.5/vsphere-esxi-vcenter-server-65-networking-guide.pdf)。
2. 设置时分布式的端口组，请提供中创建的 VLAN ID[为工作负荷的 Vm 创建 VLAN](#create-a-vlan-for-your-workload-vms)。

## <a name="next-steps"></a>后续步骤

* [使用 Azure 上的 VMware Vm](https://docs.azure.cloudsimple.com/quickstart-create-vmware-virtual-machine)
* [使用 Azure 上的 VMware Vm](quickstart-create-vmware-virtual-machine.md)
* [连接到使用 Azure ExpressRoute 的本地网络](https://docs.azure.cloudsimple.com/on-premises-connection/)
* [设置从本地站点到站点 VPN](https://docs.azure.cloudsimple.com/vpn-gateway/)
