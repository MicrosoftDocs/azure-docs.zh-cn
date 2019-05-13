---
title: 外围网络示例 – 具有外围网络包含防火墙、 UDR 和 Nsg 保护网络 |Microsoft Docs
description: 构建外围网络 (也称为 DMZ) 与防火墙、 用户定义路由 (UDR) 和网络安全组 (Nsg)。
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: dc01ccfb-27b0-4887-8f0b-2792f770ffff
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: jonor;sivae
ms.openlocfilehash: 0a7927868a9a4bebc80ec995baefbae4c45d747f
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410481"
---
# <a name="example-3-build-a-perimeter-network-to-protect-networks-with-a-firewall-udr-and-nsgs"></a>示例 3：构建外围网络以通过防火墙、 UDR 和 Nsg 保护网络

[返回安全边界最佳实践页面][HOME]

在此示例中，创建外围网络 （也称为 DMZ、 外围安全区域和外围子网）。 该示例实现防火墙、 四个 Windows 服务器、 用户定义的路由 (UDR)、 IP 转发和网络安全组 (Nsg)。 本文将引导您完成每个相关的命令，提供更深入地了解每个步骤。 流量方案部分还介绍了深度中流量如何流经外围网络中的防御层。 最后，参考部分包含的所有代码和说明如何构建此环境，以便您可以测试和试验各种方案。

![包含 NVA、NSG 和 UDR 的双向外围网络][1]

## <a name="environment-setup"></a>环境设置

此示例使用一个订阅包含以下组件：

* 三个云服务：SecSvc001、 FrontEnd001 和 BackEnd001
* 具有三个子网的虚拟网络 (CorpNetwork):SecNet、 FrontEnd 和后端
* 网络虚拟设备： 防火墙连接到 SecNet 子网
* 表示应用程序 web 服务器的 Windows server:IIS01
* 代表应用程序后端服务器的两个 Windows 服务器：AppVM01, AppVM02
* Windows server，一个代表 DNS 服务器：DNS01

[引用部分](#references)包含生成的大多数环境此处所述的 PowerShell 脚本。 否则，这篇文章没有提供生成的虚拟机 (Vm) 和虚拟网络的详细的说明。

构建环境的方法如下：

1. 保存网络配置 XML 文件中包含[参考部分](#references)。 您需要将其更新名称、 位置和 IP 地址以匹配给定的方案。
1. 更新的完整脚本，根据特定环境 （例如，订阅、 服务名称等） 中的用户变量。
1. 在 PowerShell 中执行脚本。

> [!NOTE]
> PowerShell 脚本中指定区域必须与网络配置 XML 文件中指定的区域相匹配。

该脚本成功运行后，执行以下步骤：

1. 设置防火墙规则。 请参阅[防火墙规则](#firewall-rules)部分。
1. （可选） 使用参考部分中两个脚本来设置了一个 web 应用程序 web 服务器和应用程序服务器上，以便测试此外围网络配置。

## <a name="user-defined-routing"></a>用户定义的路由

默认情况下，以下系统路由定义为：

    Effective routes :
     Address Prefix    Next hop type    Next hop IP address Status   Source
     --------------    -------------    ------------------- ------   ------
     {10.0.0.0/16}     VNETLocal                            Active   Default
     {0.0.0.0/0}       internet                             Active   Default
     {10.0.0.0/8}      Null                                 Active   Default
     {100.64.0.0/10}   Null                                 Active   Default
     {172.16.0.0/12}   Null                                 Active   Default
     {192.168.0.0/16}  Null                                 Active   Default

VNETLocal 始终是该特定的虚拟网络定义的地址已。 例如，它将从虚拟网络更改到虚拟网络，具体取决于如何定义每个特定的虚拟网络。 其余系统路由是静态的默认值与所示。

在优先级路由处理通过最长前缀匹配 (LPM) 方法。 因此最具体的路由表中适用于给定的目标地址。

因此，流量适用于服务器如 DNS01 (10.0.2.4) 的本地网络 (10.0.0.0/16) 虚拟网络之间路由由于 10.0.0.0/16 路由。  对于 10.0.2.4，10.0.0.0/16 路由是最具体的路由。 此规则应用即使 10.0.0.0/8 和 0.0.0.0/0 也可能适用也是如此。 它们是更具体的但是，因此它们不会影响此流量。 发往 10.0.2.4 的流量路由到目标以便其下一跃点与具有本地虚拟网络。

例如，10.0.0.0/16 路由不适用于适用于 10.1.1.1 的流量。 10.0.0.0/8 系统路由是最具体的因此流量被丢弃或"黑洞化"因为下一跃点为 Null。

如果目标不适用于任何 Null 前缀或 VNETLocal 前缀，流量会遵循最不具体路由 (0.0.0.0/0)。 它被路由出 internet 作为下一跃点，并利用 Azure 的 internet 边缘。

如果路由表中有两个相同的前缀，则其优先顺序基于路由的源属性：

1. VirtualAppliance:用户定义路由手动添加到表中。
1. Vpn 网关：动态路由 (与混合网络一起使用时，则为 BGP) 添加由动态网络协议。 这些路由随时间改变，因为动态协议会自动反映对等网络中的更改。
1. 默认值：系统路由、 本地的虚拟网络和更高版本的路由表中所示的静态条目。

> [!NOTE]
> 你现在可以使用用户定义路由 (UDR) 与 ExpressRoute 和 VPN 网关以强制出站和入站跨界流量路由到网络虚拟设备 (NVA)。

### <a name="create-local-routes"></a>创建本地路由

此示例使用两个路由表，分别用于前端和后端子网。 已在每个表中加载了适用于给定子网的静态路由。 对于本示例而言，每个数据表包含三个路由：

1. 与任何定义的下一跃点的本地子网流量。 此路由允许本地子网流量绕过防火墙。
2. 下一跃点定义为防火墙的虚拟网络流量。 此路由覆盖允许本地虚拟网络流量直接路由的默认规则。
3. 使用下一跃点定义为防火墙的所有剩余流量 (0/0)。

路由表在创建后，将会绑定到其子网。 前端子网路由表应如下所示：

    Effective routes :
     Address Prefix    Next hop type       Next hop IP address  Status   Source
     --------------    ------------------  -------------------  ------   ------
     {10.0.1.0/24}     VNETLocal                                Active
     {10.0.0.0/16}     VirtualAppliance    10.0.0.4             Active
     {0.0.0.0/0}       VirtualAppliance    10.0.0.4             Active

此示例使用以下命令生成的路由表、 添加用户定义的路由，以及然后将路由表绑定到子网。 以开头的项`$`，如`$BESubnet`，是用户定义变量的脚本中的参考部分。

1. 首先，创建基础路由表。 以下代码段创建后端子网的表。 完整脚本还会创建前端子网的相应表。

   ```powershell
   New-AzureRouteTable -Name $BERouteTableName `
       -Location $DeploymentLocation `
       -Label "Route table for $BESubnet subnet"
   ```

1. 创建路由表后，可以添加特定的用户定义路由。 下面的代码段指定，通过虚拟设备路由所有流量 (0.0.0.0/0)。 变量`$VMIP[0]`用于传递之前在脚本中创建虚拟设备时分配的 IP 地址。 完整脚本还在前端的表中创建相应的规则。

   ```powershell
   Get-AzureRouteTable $BERouteTableName | `
       Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
       -NextHopType VirtualAppliance `
       -NextHopIpAddress $VMIP[0]
   ```

1. 以前的路由条目将替代默认"0.0.0.0/0"路由，但默认 10.0.0.0/16 规则仍允许在虚拟网络路由直接到目标，而不适用于网络虚拟设备路由流量。 若要更正此行为，您需要添加以下规则：

   ```powershell
   Get-AzureRouteTable $BERouteTableName | `
       Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
       -NextHopType VirtualAppliance `
       -NextHopIpAddress $VMIP[0]
   ```

1. 此时，您必须做出选择。 两个以前的规则将所有流量都路由到防火墙进行评估，包括中的单个子网的流量。 可能需要此行为。 如果不这样做，但是，可以允许在本地路由而防火墙参与一个子网的流量。 添加第三，直接路由的本地子网发往任何地址的特定规则 (NextHopType = VNETLocal)。

   ```powershell
   Get-AzureRouteTable $BERouteTableName | `
       Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
           -NextHopType VNETLocal
   ```

1. 最后，创建并填充用户定义的路由的路由表后，您需要将表绑定到子网。 下面的代码段将绑定后端子网的表。 完整脚本还将前端路由表绑定到前端子网。

   ```powershell
   Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
       -SubnetName $BESubnet `
       -RouteTableName $BERouteTableName
   ```

## <a name="ip-forwarding"></a>IP 转发

IP 转发是 UDR 的随附功能。 在虚拟设备上的通过此设置，它能够接收不发送到该设备的流量再将转发到其最终目标的流量。

例如，如果来自 AppVM01 的流量对 DNS01 服务器发出请求，UDR 会将流量路由到防火墙。 启用 IP 转发，与 DNS01 目标 (10.0.2.4) 的流量是被防火墙设备 (10.0.0.4) 所接受，并转发到其最终目标 (10.0.2.4)。 而无需在防火墙上启用 IP 转发，流量是不被设备所接受即使路由表将防火墙用作下一跃点。

> [!IMPORTANT]
> 请记住启用 IP 转发和用户定义路由。

在创建 VM 时，可以使用单个命令启用 IP 转发。 调用你的防火墙虚拟设备和启用 IP 转发的 VM 实例。 请注意，红色开头的项`$`，例如`$VMName[0]`，是从本文档的参考部分中的脚本的用户定义变量。 在方括号中，零`[0]`，代表 Vm 阵列中的第一个 VM。 对于示例脚本无须修改即可运行，第一个 VM (VM 0) 必须是防火墙。 在完整脚本中，相关代码片段进行分组所依据即将结束的 UDR 命令。

```powershell
Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] | `
    Set-AzureIPForwarding -Enable
```

## <a name="network-security-groups"></a>网络安全组

在此示例中，生成的网络安全组 (NSG)，然后将其加载使用单个规则。 然后，示例将 NSG 绑定仅到前端和后端的子网 (不 SecNet)。 加载到 NSG 的规则如下所示：

* 拒绝任何流量 （所有端口） 从 internet 到整个虚拟网络 （所有子网）

尽管在此示例中使用 Nsg，但其主要用途是防线的作为第二，以防手动配置不正确。 你想要阻止所有从 internet 到前端或后端子网的入站的流量。 流量应仅流经 SecNet 子网到防火墙，其后应获取仅相应流量路由到前端或后端子网。 此外，配置 UDR 规则将任何达到到防火墙的前端或后端子网的流量路由。 防火墙将它视为非对称流量，并丢弃出站流量。

三个安全层保护的前端和后端的子网：

1. FrontEnd001 和 BackEnd001 云服务上没有开放的终结点
1. Nsg 拒绝来自 internet 的流量
1. 防火墙会丢弃非对称流量

有关此示例中的 NSG 关注点是，它只包含一个规则，如下所示。 此规则将拒绝 internet 流往整个虚拟网络，包括安全子网流量。

```powershell
Get-AzureNetworkSecurityGroup -Name $NSGName | `
    Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet `
    from the internet" `
    -Type Inbound -Priority 100 -Action Deny `
    -SourceAddressPrefix INTERNET -SourcePortRange '*' `
    -DestinationAddressPrefix VIRTUAL_NETWORK `
    -DestinationPortRange '*' `
    -Protocol *
```

由于 NSG 只绑定到前端和后端的子网，不应用该规则的流量往安全子网的入站。 因此，流量流向安全子网。

```powershell
Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
    -SubnetName $FESubnet -VirtualNetworkName $VNetName

Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
    -SubnetName $BESubnet -VirtualNetworkName $VNetName
```

## <a name="firewall-rules"></a>防火墙规则

必须在防火墙上创建转发规则。 由于防火墙阻止或转发所有入站、 出站和虚拟网络内通信，因此需要许多防火墙规则。 此外，防火墙必须处理 （在不同的端口） 的安全服务公共 IP 地址的所有入站的流量。 若要避免以后出现返工，请通过关系图创建功能之前设置的子网和防火墙规则的逻辑流的最佳做法。 下图是此示例中防火墙规则的逻辑视图：

![防火墙规则的逻辑视图][2]

> [!NOTE]
> 管理端口会有所不同，具体取决于网络虚拟设备。 此示例演示 Barracuda NextGen 防火墙使用端口 22、 801 和 807。 请参阅设备供应商文档以查找可以管理设备的确切端口。

### <a name="logical-rule-description"></a>逻辑规则描述

上面的逻辑关系图不会显示安全子网，因为防火墙是该子网上的唯一资源。 下图显示了防火墙规则，如何在逻辑上允许或拒绝流量的流动，但不是实际的路由路径。 此外，远程桌面协议 (RDP) 流量选择的外部端口是较高范围的端口 (8014 – 8026)，选择要对齐的本地 IP 地址的最后两个八位字节以方便阅读。 例如，本地服务器地址 10.0.1.4 是与外部端口 8014 相关联。 但是，可以使用更高版本的任何非冲突端口。

对于此示例需要以下类型的规则：

* 外部规则针对入站流量：
  1. 防火墙管理规则： 允许流量传递到网络虚拟设备的管理端口。
  2. 每个 windows 服务器的 RDP 规则： 允许通过 RDP 单个服务器管理。  具体取决于网络虚拟设备的功能，您可能能够捆绑到一个规则。
  3. 应用程序流量规则： 一个用于前端 web 流量，一个用于后端流量 （例如，web 服务器到数据层）。 这些规则的配置取决于网络体系结构和流量流。

     * 第一个规则允许实际的应用程序流量抵达应用程序服务器。 与安全、 管理和等等的规则，不同应用程序规则允许外部用户或服务访问应用程序。 此示例中有单个 web 服务器上的端口 80，它允许单个防火墙应用程序规则目标为外部的 IP 地址，而是将路由到 web 服务器的内部 IP 地址的流量重定向。 重定向的流量会话通过 NAT 重新映射到内部服务器。
     * 第二个应用程序流量规则是后端规则，用于允许 web 服务器的任何端口用于将流量路由到 AppVM01 服务器，而不是属于 AppVM02 服务器。

* 内虚拟网络流量的内部规则：
  1. 出站到 internet 规则： 允许来自任何网络传递到所选的网络流量。 此规则通常是默认值在防火墙上，但处于禁用状态。 启用此规则对于此示例。
  2. DNS 规则： 只允许 DNS （端口 53） 流量传递到 DNS 服务器。 在此环境中，阻止了大部分由前端流往后端的流量。 此规则专门允许来自任何本地子网的 DNS。
  3. 子网到子网规则： 允许后端子网连接到前端子网，但不能反向的任何服务器上的任何服务器。

* 不满足上述条件的任何流量的防故障规则：
  1. 拒绝所有流量规则： 规则在优先级方面始终最后一个。 如果流量流动行为无法符合任何上述规则，此规则会阻止它。 这是默认规则。 因为它通常已激活，不需要修改。

> [!TIP]
> 在第二个应用程序流量规则，允许任何端口为了简化此示例。 在真实情况下，应使用特定端口和地址范围以减少此规则的攻击面。


> [!IMPORTANT]
> 创建规则后，务必查看每个规则，以确保允许或拒绝根据流量的优先级。 在本示例中，规则设置了优先顺序。 它很容易的规则错误顺序获取锁定在防火墙。 请确保将防火墙管理规则设置为绝对最高优先级。

### <a name="rule-prerequisites"></a>规则先决条件

公共终结点所需的运行防火墙的虚拟机。 必须打开这些公共终结点，以便在防火墙可以处理流量。 有三种类型的流量在此示例中：

1. 管理流量，用于控制防火墙和防火墙规则
1. 若要控制 windows 服务器的 RDP 流量
1. 应用程序流量

在上面的防火墙规则逻辑图的上半部分显示了流量类型。

> [!IMPORTANT]
> 请记住，*所有*流量是通过防火墙。 远程桌面连接到 IIS01 服务器，您需要连接到端口 8014 上的防火墙，并允许防火墙将 RDP 请求内部路由到 IIS01 RDP 端口。 在 Azure 门户**Connect**按钮不会起作用，因为有没有直接到 IIS01 的 RDP 路径，就可以看到在门户。 所有来自 internet 的连接将连往安全服务和端口 (例如 secscv001.cloudapp.net: xxxx)。 参考上图中的外部端口与内部 IP 和端口的映射。

在创建 VM 或生成后的时间，可以打开一个终结点。 示例脚本和下面的代码段打开一个终结点后创建的 VM。

请注意，项的开头`$`，如`$VMName[$i]`，是用户定义变量的脚本中的参考部分。 `[$i]`代表 Vm 阵列中特定 VM 的阵列编号。

```powershell
Add-AzureEndpoint -Name "HTTP" -Protocol tcp -PublicPort 80 -LocalPort 80 `
    -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | `
    Update-AzureVM
```

尽管它不清楚地显示此处由于变量，但只应该打开安全云服务上的终结点。 此预防措施有助于确保防火墙处理所有入站的流量，是否具有路由、 重新映射 NAT，或删除。

若要管理防火墙并创建必要的配置的 PC 上安装管理客户端。 有关如何管理您的防火墙或其他 NVA 的详细信息，请参阅供应商的文档。 本部分的其余部分并将**防火墙规则创建**部分描述了防火墙的配置。 使用供应商的管理客户端，而不是 Azure 门户或 PowerShell。

转到[Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin)下载的管理客户端并了解如何连接到 Barracuda 防火墙。

登录到防火墙后，请创建防火墙规则之前定义网络和服务对象。 这两个必备的对象类可以更轻松地创建规则。

对于本例，请为每个定义三个命名的网络对象：

* 前端子网
* 后端子网
* DNS 服务器 IP 地址

若要从 Barracuda NG Admin 客户端仪表板中创建命名的网络：

1. 转到**配置选项卡**。
1. 选择**Ruleset**中**操作配置**部分
1. 选择**网络**下**防火墙对象**菜单。
1. 选择**新建**中**编辑网络**菜单。
1. 通过添加名称和前缀创建网络对象：

   ![创建 front ed 网络对象][3]

在前面的步骤创建前端子网的命名的网络。 创建为后端子网以及类似的对象。 现在可以更轻松地在防火墙规则中按名称引用子网。

DNS 服务器对象：

![创建 DNS 服务器对象][4]

DNS 规则文档中的更高版本中使用此单个 IP 地址引用。

其他对象类包括服务对象，它表示每个服务器的 RDP 连接端口。 现有 RDP 服务对象绑定到默认 RDP 端口 3389。 因此，您可以创建新的服务，以允许来自外部端口 (8014-8026) 的流量。 您还可以向现有 RDP 服务添加新的端口。 但是，为了便于演示，你可以使单个规则的每个服务器。 若要从 Barracuda NG Admin 客户端仪表板创建新的服务器的 RDP 规则：

1. 转到**配置选项卡**。
1. 选择**Ruleset**中**操作配置**部分。
1. 选择**Services**下**防火墙对象**菜单。
1. 服务，然后选择列表中向下的滚动**RDP**。
1. 右键单击并选择复制，然后右键单击并选择粘贴。
1. 现可编辑的 RDP-Copy1 服务对象。 右键单击**RDP-Copy1** ，然后选择**编辑**。
1. **编辑服务对象**窗口弹出按如下所示：

   ![默认 RDP 规则的副本][5]

1. 编辑的值来表示特定服务器的 RDP 服务。 对于 AppVM01，默认 RDP 规则应修改以反映新的服务**名称**，**说明**，和在图 8 关系图中使用的外部 RDP 端口。 请记住为此特定服务器的外部端口，从 RDP 默认值 3389 更改端口。 例如，为 AppVM01 的外部端口为 8025。 已修改的服务规则如下所示：

   ![AppVM01 规则][6]

重复此过程以创建其余的服务器的 RDP 服务：AppVM02、 DNS01 和 IIS01。 这些服务使规则下一节中，若要创建更简单且更明显。

> [!NOTE]
> 不需要在防火墙的 RDP 服务，因为防火墙 VM 是基于 Linux 的映像，因此管理 VM 而不是 RDP 的端口 22 上使用 SSH。 此外，端口 22，其他两个端口允许进行管理连接。 请参阅**防火墙管理规则**下一节中。

### <a name="firewall-rules-creation"></a>创建防火墙规则

有三种类型的防火墙规则使用在此示例中，所有这些不同的图标：

应用程序重定向规则：![“应用程序重定向”图标][7]

目标 NAT 规则：![“目标 NAT”图标][8]

传递规则：![“传递”图标][9]

可以在 Barracuda 网站上找到这些规则的详细信息。

若要创建以下规则或验证现有的默认规则：

1. 从 Barracuda NG Admin 客户端仪表板中，转到**配置**选项卡。
1. 在中**操作配置**部分中，选择**Ruleset**。
1. **主规则**网格中显示的现有活动，然后停用此防火墙规则。 选择绿色**+** 中创建新规则的右上角。 如果防火墙处于锁定状态的更改，请参阅标记为的按钮**锁**和不能创建或编辑规则。 选择**锁**按钮以解除锁定规则集，并允许编辑。 右键单击你想要编辑，并选择某个的规则**编辑规则**。

创建或修改的所有规则后，将其推送到防火墙并激活它们。 否则，规则更改将不会生效。 推送和激活过程是中所述[规则激活](#rule-activation)。

以下是完成此示例中所需的每个规则的详细信息：

* **防火墙管理规则**:此应用重定向规则允许流量传递到网络虚拟设备，在此示例中 Barracuda NextGen 防火墙的管理端口。 管理端口是 801、 807，并可以选择 22。 外部和内部端口是相同的没有端口转换。 此规则称为设置管理访问权限。 它是默认规则，并在 Barracuda NextGen 防火墙版本 6.1 中默认启用。
  
    ![防火墙管理规则][10]

  > [!TIP]
  > 此规则中的源地址空间是**任何**。 如果已知的管理 IP 地址范围，减小此范围也将减少到的管理端口的攻击面。

* **RDP 规则**:这些目标 NAT 规则允许通过 RDP 单个服务器管理。 这些规则的重要字段包括：
  * 源。 若要允许从任何位置的 RDP，使用引用**任何**源字段中。
  * 服务。 使用前面创建的 RDP 服务对象：**AppVM01 RDP**。 外部端口将重定向到服务器的本地 IP 地址和默认 RDP 端口 3386。 此特定规则用于对 AppVM01 进行 RDP 访问。
  * 目标。 使用在防火墙上的本地端口：**DCHP 1 本地 IP**或**eth0**如果使用静态 Ip。 序号 （eth0、 eth1 等） 可能会不同，如果网络设备有多个本地接口。 防火墙使用此端口用于发送，并且它可能与接收端口相同。 实际的路由的目标处于**目标列表**字段。
  * 重定向。 配置，以告知虚拟设备最终将此流量重定向的位置。 最简单的重定向是放在目标列表字段中的 IP。 您还可以指定端口，和 NAT 网络端口和 IP 地址。 如果不指定端口，虚拟设备使用的目标端口入站请求。

    ![防火墙 RDP 规则][11]

    创建四个 RDP 规则：

    | 规则名称 | 服务器 | 服务 | 目标列表 |
    | --- | --- | --- | --- |
    | RDP-to-IIS01 |IIS01 |IIS01 RDP |10.0.1.4:3389 |
    | RDP-to-DNS01 |DNS01 |DNS01 RDP |10.0.2.4:3389 |
    | RDP-to-AppVM01 |AppVM01 |AppVM01 RDP |10.0.2.5:3389 |
    | RDP-to-AppVM02 |AppVM02 |AppVm02 RDP |10.0.2.6:3389 |

  > [!TIP]
  > 缩小源和服务字段的范围可以减少受攻击面。 使用允许功能的最小的范围。

* **应用程序流量规则**:有两个应用程序流量规则。 其中一个是前端 web 流量。 其他涵盖了如到数据层的 web 服务器的后端流量。 这些规则取决于网络体系结构和流量流动行为。
  
  * 前端 web 流量的规则：
  
    ![防火墙 Web 规则][12]
  
    此目标 NAT 规则允许实际的应用程序流量抵达应用程序服务器。 与安全、 管理和等的规则，不同应用程序规则允许外部用户或服务访问应用程序。 此示例中有单个 web 服务器上的端口 80，它允许单个防火墙应用程序规则目标为外部的 IP 地址，而是将路由到 web 服务器的内部 IP 地址的流量重定向。 重定向的流量会话通过 NAT 重新映射到内部服务器。

    > [!NOTE]
    > 没有在分配的端口**目标列表**字段。 因此，在 web 服务器的重定向使用的入站的端口 80 （或选择的服务为 443）。 如果 web 服务器正在侦听其他端口 8080 等，您可以将目标列表字段更新为 10.0.1.4: 8080 以允许端口重定向。
  
  * 后端规则允许 web 服务器与 AppVM01 服务器，而非 AppVM02，通过**任何**服务：
  
    ![防火墙 AppVM01 规则][13]
  
    此传递规则允许前端子网访问 AppVM01 上的任何 IIS 服务器 (10.0.2.5) 上使用任何协议，以便可以由 web 应用程序访问数据的任何端口。
  
    在此屏幕截图`<explicit-dest>`中使用**目标**字段来表示目标目标为 10.0.2.5。 可以按屏幕截图所示显式指定的 IP 地址。 此外可以在 DNS 服务器的先决条件中使用命名的网络对象等。 防火墙管理员可以选择要使用的方法。 若要将 10.0.2.5 添加为显式目标，请双击下方的第一个空白行`<explicit-dest>`并在打开的对话框中输入的地址。
  
    使用此传递规则时，不需要 NAT，因为它处理内部流量。 可以设置**连接方法**到`No SNAT`。
  
    > [!NOTE]
    > 此规则中的源网络是在前端子网上的任何资源，如果只有一个。 如果您的体系结构指定已知的多个 web 服务器，可以创建要为更与这些确切 IP 地址而不是整个前端子网的网络对象资源。

    > [!TIP]
    > 此规则使用服务**任何**轻松地设置和使用示例应用程序。 它允许 ICMPv4 (ping) 中的单个规则。 但是，为了降低攻击面跨越此边界，因此建议限制的端口和协议的服务可能允许应用程序的操作的最低。

    > [!TIP]
    > 尽管该示例规则使用`<explicit-dest>`引用，应使用一致的方法在整个防火墙配置。 我们建议使用已命名的网络对象，以便于可读性和可支持性。 `<explicit-dest>`此处显示仅用于显示备选的引用方法。 我们通常不建议，尤其是对于复杂的配置。

* **出站到 internet 规则**:此传递规则允许来自任何源网络传递到所选的目标网络的流量。 Barracuda NextGen 防火墙默认情况下，但处于禁用状态，通常将此规则具有"on"。 右键单击此规则以访问**激活规则**命令。 修改将后端和前端的子网的网络对象添加到此规则的源属性的屏幕截图中所示的规则。 在这篇文章的先决条件部分中创建这些网络对象。
  
    ![防火墙出站规则][14]

* **DNS 规则**:此传递规则只允许 DNS （端口 53） 流量传递到 DNS 服务器。 在此环境，以便此规则专门允许 DNS 流量被阻止大多数流量从前端到后端。
  
    ![防火墙 DNS 规则][15]
  
    > [!NOTE]
    > **连接方法**设置为`No SNAT`因为此规则用于内部 IP 到内部 IP 地址的流量，并且是必需的无重新路由功能可通过 NAT。

* **子网到子网规则**:默认传递规则已激活并修改，以允许后端子网连接到前端子网的任何服务器上的任何服务器。 此规则仅 coves 内部流量，因此**连接方法**可设置为`No SNAT`。

    ![防火墙 VNet 内部规则][16]
  
    > [!NOTE]
    > **双向**以便仅在一个方向中应用此规则，不此处选中复选框。 可以从后端子网到前端网络，但不能反向启动连接。 如果已选中该复选框，此规则将启用双向通信，我们已指定为逻辑图中。

* **拒绝所有流量规则**:此规则应始终为最终规则在优先级方面。 如果流量流不匹配任何上述规则，此规则后，即可删除。 默认情况下通常激活规则，因此不需要修改。
  
    ![防火墙拒绝规则][17]

> [!IMPORTANT]
> 创建所有前面的规则后，查看每个规则，以确保允许还是拒绝根据流量的优先级。 对于此示例，它们应在 Barracuda 管理客户端的转发规则主网格中显示的顺序列出的规则。

## <a name="rule-activation"></a>规则激活

已修改的规则集以满足逻辑图中的规范后，您需要将规则集上传到防火墙并将其激活。

![防火墙规则激活][18]

在管理客户端窗口的右上角中查找并选择**发送更改**若要将修改后的规则上传到防火墙。 选择“激活”。

在启用防火墙规则集时，此示例环境中已完成。

## <a name="traffic-scenarios"></a>流量方案

> [!IMPORTANT]
> 请记住，*所有*流量是通过防火墙。 远程桌面连接到 IIS01 服务器，您需要连接到端口 8014 上的防火墙，并允许防火墙将 RDP 请求内部路由到 IIS01 RDP 端口。 在 Azure 门户**Connect**按钮不会起作用，因为有没有直接到 IIS01 的 RDP 路径，就可以看到在门户。 所有来自 internet 的连接将连往安全服务和端口 (例如 secscv001.cloudapp.net: xxxx)。 参考上图中的外部端口与内部 IP 和端口的映射。

对于这些方案，应准备好以下防火墙规则：

1. 防火墙管理 (FW Mgmt)
2. 通过 RDP 访问 IIS01
3. 通过 RDP 访问 DNS01
4. 通过 RDP 访问 AppVM01
5. 通过 RDP 访问 AppVM02
6. 到 Web 的应用流量
7. 到 AppVM01 的应用流量
8. 出站到 internet
9. 前端到 DNS01
10. 子网内部流量（仅限后端到前端）
11. 全部拒绝

在实际的防火墙规则集将很可能需要比在此示例中的多个规则。 它们可能具有不同优先级的数字。 您应参考此列表和关联的编号为彼此相对优先级。 例如，"通过 RDP 访问 IIS01"规则可能是数字 5 上实际的防火墙，但只要它的下面的"防火墙管理"规则和上面"通过 RDP 访问 DNS01"规则，一组符合此列表的目的。 此列表还可帮助简化方案，请按照的说明。 例如，"防火墙规则 9 (DNS)。" 请注意，四个 RDP 规则统称为"RDP 规则"当流量方案与 RDP 无关时。

此外请记住，网络安全组 (Nsg) 是在前端和后端子网上的入站 internet 流量的位置。

### <a name="allowed-internet-to-web-server"></a>（允许）Internet 访问 web 服务器

1. Internet 用户从 SecSvc001.CloudApp.Net （面向 internet 的云服务） 请求 HTTP 页面。
1. 云服务将端口 80 上通过开放的终结点的流量传递到 10.0.0.4: 80 上的防火墙接口。
1. 无 NSG 分配到安全子网，因此系统 NSG 规则允许流量发往防火墙。
1. 流量抵达防火墙 (10.0.1.4) 的内部 IP 地址。
1. 防火墙执行规则处理：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 (应用：适用于 web)。 允许该流量。 防火墙重新确定通过 NAT 发送到 10.0.1.4 (IIS01) 的流量。
1. 前端子网执行处理入站的规则：
   1. NSG 规则 1 (阻止 internet) 不适用。 防火墙重新路由通过 NAT 此流量，因此源地址现在是防火墙。 由于防火墙位于安全子网，并且将显示为前端子网 NSG 的本地流量，允许流量。 转到下一规则。
   1. 默认的 NSG 规则允许子网到子网流量，因此允许此流量。 停止处理 NSG 规则。
1. IIS01 正在侦听的 web 流量。 它接收此请求，并开始处理请求。
1. IIS01 尝试启动 AppVM01 的 FTP 会话在后端子网。
1. 在前端子网的 UDR 路由使防火墙成为下一跃点。
1. 没有出站规则上的前端子网以便允许该流量。
1. 防火墙开始处理规则：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 (应用：Web) 不适用。 转到下一规则。
   1. 防火墙规则 7 (应用： 后端) 应用。 允许该流量。 防火墙将流量转发到 10.0.2.5 (AppVM01)。
1. 后端子网执行处理入站的规则：
    1. NSG 规则 1 (阻止 internet) 不适用。 转到下一规则。
    1. 默认的 NSG 规则允许子网到子网的流量。 允许该流量。 停止处理 NSG 规则。
1. AppVM01 接收请求，启动会话，并做出响应。
1. 后端子网上的 UDR 路由使防火墙成为下一跃点。
1. 没有出站的 NSG 规则在后端子网因此允许响应。
1. 由于它会返回已建立的会话上的流量，防火墙将响应返回给 web 服务器 (IIS01)。
1. 前端子网执行处理入站的规则：
    1. NSG 规则 1 (阻止 internet) 不适用。 转到下一规则。
    1. 默认 NSG 规则允许子网到子网流量，因此允许此流量。 停止处理 NSG 规则。
1. IIS 服务器接收响应，并完成 AppVM01 的事务。 然后在服务器完成构建 HTTP 响应，并将其发送给请求者。
1. 没有出站的 NSG 规则在前端子网上因此允许响应。
1. HTTP 响应抵达防火墙。 因为它是对已建立的 NAT 会话的响应，防火墙会接受它。
1. 防火墙将响应传回给 internet 用户重定向。
1. 没有任何出站 NSG 规则或在前端子网上的 UDR 跃点以便允许响应。 Internet 用户将收到请求的网页。

### <a name="allowed-internet-rdp-to-back-end"></a>（允许）Internet RDP 到后端

1. 在 internet 上的服务器管理员请求通过 secsvc001.cloudapp.net:8025 启动 appvm01 的 RDP 会话。 8025 是用户分配的端口号的防火墙规则 4 (RDP AppVM01)。
1. 云服务将端口 8025 上通过开放终结点的流量传递到 10.0.0.4: 8025 上的防火墙接口。
1. 无 NSG 分配到安全子网，因此系统 NSG 规则允许流量发往防火墙。
1. 防火墙执行规则处理：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 不能应用防火墙规则 2 (RDP IIS)。 转到下一规则。
   1. 防火墙规则 3 (RDP DNS01) 不适用。 转到下一规则。
   1. 防火墙规则 4 (RDP AppVM01) 适用因此允许流量。 防火墙重新确定其通过 NAT 到 10.0.2.5:3386 （AppVM01 上的 RDP 端口）。
1. 后端子网执行处理入站的规则：
   1. NSG 规则 1 (阻止 internet) 不适用。 防火墙重新路由通过 NAT 此流量，因此源地址现在是位于安全子网上的防火墙。 它被后端子网 NSG 是本地的流量，允许使用。 转到下一规则。
   1. 默认的 NSG 规则允许子网到子网流量，因此允许此流量。 停止处理 NSG 规则。
1. AppVM01 侦听 RDP 流量并做出响应。
1. 没有出站 NSG 规则以便将应用默认规则。 允许返回流量。
1. UDR 将出站流量路由到防火墙，为下一跃点。
1. 由于它会返回已建立的会话上的流量，防火墙将响应传回给 internet 用户。
1. 已启用 RDP 会话。
1. AppVM01 会提示输入用户名和密码。

### <a name="allowed-web-server-dns-lookup-on-dns-server"></a>（允许）DNS 服务器上的 web 服务器 DNS 查找

1. Web 服务器 IIS01 请求数据馈送在 http\:\/\/www.data.gov，但需要解析地址。
1. 网络配置的虚拟网络列表 DNS01 (后端子网上的 10.0.2.4) 作为主 DNS 服务器。 IIS01 将 DNS 请求发送到 DNS01。
1. UDR 将出站流量路由到防火墙，为下一跃点。
1. 没有出站的 NSG 规则会绑定到前端子网。 允许该流量。
1. 防火墙执行规则处理：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 和 7 （应用规则） 不适用。 转到下一规则。
   1. 防火墙规则 8 （到 internet) 不适用。 转到下一规则。
   1. 应用防火墙规则 9 (DNS)。 允许该流量。 防火墙将流量转发到 10.0.2.4 (DNS01)。
1. 后端子网执行处理入站的规则：
   1. NSG 规则 1 (阻止 internet) 不适用。 转到下一规则。
   1. 默认的 NSG 规则允许子网到子网的流量。 允许该流量。 停止处理 NSG 规则。
1. DNS 服务器接收请求。
1. DNS 服务器没有缓存的地址，在 internet 上请求的根 DNS 服务器。
1. UDR 将出站流量路由到防火墙，为下一跃点。
1. 有允许的流量的后端子网上没有出站 NSG 规则。
1. 防火墙执行规则处理：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 和 7 （应用规则） 不适用。 转到下一规则。
   1. 应用防火墙规则 8 （到 internet)。 允许该流量。 会话通过 SNAT 重新路由到 internet 上的根 DNS 服务器。
1. Internet DNS 服务器做出响应。 此会话是从防火墙发起，因此被防火墙接受响应。
1. 此会话已建立使防火墙将转发到发起端服务器 DNS01 的响应。
1. 后端子网执行处理入站的规则：
    1. NSG 规则 1 (阻止 internet) 不适用。 转到下一规则。
    1. 默认 NSG 规则，允许对此流量的子网到子网流量。 停止处理 NSG 规则。
1. DNS 服务器接收并缓存响应，并响应给 IIS01 的初始请求。
1. 后端子网上的 UDR 路由使防火墙成为下一跃点。
1. 没有出站的 NSG 规则存在在后端子网，因此允许该流量。
1. 此会话已在防火墙上建立，因此将防火墙重置回 IIS 服务器的响应。
1. 前端子网执行处理入站的规则：
    1. 因此没有任何 NSG 规则应用从后端子网到前端子网的入站流量没有 NSG 规则。
    1. 默认系统规则允许子网之间的通信。 允许该流量。
1. IIS01 从 DNS01 接收响应。

### <a name="allowed-back-end-server-to-front-end-server"></a>（允许）后端服务器到前端服务器

1. 管理员身份登录 AppVM02 通过 RDP 直接从 IIS01 服务器通过 windows 文件资源管理器请求文件。
1. 后端子网上的 UDR 路由使防火墙成为下一跃点。
1. 没有出站的 NSG 规则在后端子网因此允许响应。
1. 防火墙执行规则处理：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 和 7 （应用规则） 不适用。 转到下一规则。
   1. 防火墙规则 8 （到 internet) 不适用。 转到下一规则。
   1. 防火墙规则 9 (DNS) 不适用。 转到下一规则。
   1. 应用防火墙规则 10 （子网内部）。 允许该流量。 防火墙将流量传递到 10.0.1.4 (IIS01)。
1. 前端子网开始处理入站的规则：
   1. NSG 规则 1 (阻止 internet) 不适用，会转到下一规则
   1. 默认 NSG 规则允许子网到子网流量，因此允许该流量。 停止处理 NSG 规则。
1. 假设正确的身份验证和授权，IIS01 将接受请求和响应。
1. 在前端子网上的 UDR 路由使防火墙成为下一跃点。
1. 没有出站的 NSG 规则在前端子网上因此允许响应。
1. 此会话已在防火墙上存在，因此允许此响应。 防火墙将响应返回给 AppVM02。
1. 后端子网执行处理入站的规则：
    1. NSG 规则 1 (阻止 internet) 不适用。 转到下一规则。
    2. 默认 NSG 规则允许子网到子网流量，因此允许该流量。 停止处理 NSG 规则。
1. AppVM02 接收响应。

### <a name="denied-internet-direct-to-web-server"></a>（拒绝）Internet 直接连接到 web 服务器

1. Internet 用户尝试通过 FrontEnd001.CloudApp.Net 服务访问 IIS01 web 服务器。
1. 没有为 HTTP 流量开放的终结点，因此此流量不会通过云服务。 流量不会连接到服务器。
1. 如果出于某种原因，终结点处于打开状态，在前端子网上的 NSG (阻止 internet) 将阻止此流量。
1. 最后前, 端子网 UDR 路由将发送任何出站流量从 IIS01 到防火墙用作下一跃点。 防火墙将它视为非对称流量，并且将删除出站响应。

>因此，有三个独立防御 internet 和 IIS01 之间层。 云服务防止未经授权或不适当的访问。

### <a name="denied-internet-to-back-end-server"></a>（拒绝）Internet 访问后端服务器

1. Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务访问 AppVM01 上的文件。
2. 没有打开的文件共享，以便此请求不会通过云服务的终结点。 流量不会连接到服务器。
3. 如果出于某种原因处于打开状态的终结点，NSG (阻止 internet) 将阻止此流量。
4. 最后，UDR 路由将发送任何出站流量来自 AppVM01 到防火墙用作下一跃点。 防火墙将它视为非对称流量，并且将删除出站响应。

> 因此，有三个独立防御 internet 和 AppVM01 之间层。 云服务防止未经授权或不适当的访问。

### <a name="denied-front-end-server-to-back-end-server"></a>（拒绝）前端服务器到后端服务器

1. IIS01 遭到破坏，并且正在运行恶意代码尝试扫描后端子网服务器。
1. 前端子网 UDR 路由将发送任何出站流量从 IIS01 到防火墙用作下一跃点。 遭到入侵的 VM 不能更改此路由。
1. 防火墙处理流量。 如果请求与 AppVM01 或 DNS 查找的 DNS 服务器，则防火墙可能可能由于防火墙规则 7 和 9 允许的流量。 所有其他流量被防火墙规则 11 （全部拒绝） 阻止。
1. 如果您启用高级的威胁检测在防火墙上，无法阻止包含已知的签名或模式的标志的高级的威胁规则的流量。 此度量值可以工作，即使根据本文中讨论的基本转发规则允许该流量。 本文档中未介绍高级的威胁检测。 请参阅高级威胁功能了解特定网络设备的供应商文档。

### <a name="denied-internet-dns-lookup-on-dns-server"></a>（拒绝）在 DNS 服务器上执行 Internet DNS 查找

1. Internet 用户尝试查找 DNS01 上通过 BackEnd001.CloudApp.Net 服务的内部 DNS 记录。
1. 由于没有为 DNS 流量开放终结点，此流量不会通过云服务。 它不会到达服务器。
1. 如果出于某种原因，终结点处于打开状态，在前端子网上的 NSG 规则 (阻止 internet) 将阻止此流量。
1. 最后后, 端子网 UDR 路由将发送任何出站流量从 DNS01 到防火墙用作下一跃点。 防火墙将此视为非对称流量，并且删除出站响应。

> 因此，有三个独立防御 internet 和 DNS01 之间层。 云服务防止未经授权或不适当的访问。

#### <a name="denied-internet-to-sql-access-through-firewall"></a>（拒绝）Internet 通过防火墙访问 SQL

1. Internet 用户从 SecSvc001.CloudApp.Net 面向 internet 的云服务请求 SQL 数据。
1. 没有为 SQL 开放的终结点，因此此流量不会通过云服务。 应用程序不会在防火墙。
1. 如果出于某种原因，SQL 终结点处于打开状态，防火墙将执行处理规则：
   1. 防火墙规则 1 (FW Mgmt) 不适用。 转到下一规则。
   1. 防火墙规则 2-5 （RDP 规则） 不适用。 转到下一规则。
   1. 防火墙规则 6 和 7 （应用程序规则） 不适用。 转到下一规则。
   1. 防火墙规则 8 （到 internet) 不适用。 转到下一规则。
   1. 防火墙规则 9 (DNS) 不适用。 转到下一规则。
   1. 防火墙规则 10 （子网内部） 不适用。 转到下一规则。
   1. 应用防火墙规则 11 （全部拒绝）。 流量被阻止。 停止处理规则。

## <a name="references"></a>参考

本部分包含以下各项：

* 完整脚本。 将其保存在 PowerShell 脚本文件中。
* 网络配置。将其保存到名为 NetworkConf2.xml 的文件。

修改用户定义的变量中根据需要的文件。 运行该脚本，然后按照在本文前面列出的防火墙规则设置说明。

### <a name="full-script"></a>完整脚本

设置用户定义变量之后, 运行此脚本保存到：

1. 连接到 Azure 订阅
1. 创建新的存储帐户
1. 创建新的虚拟网络和网络配置文件中定义的三个子网
1. 生成五个虚拟机： 防火墙和四个 Windows Server Vm
1. 配置 UDR:
   1. 创建两个新的路由表
   1. 将路由添加到表
   1. 将表绑定到相应的子网
1. 在 NVA 上启用 IP 转发
1. 配置 NSG:
   1. 创建 NSG
   1. 添加规则
   1. 将 NSG 绑定到相应的子网

运行此 PowerShell 脚本在 internet 上的本地连接 PC 或服务器。

> [!IMPORTANT]
> 当运行此脚本时，警告或其他信息性消息可能会弹出在 PowerShell 中。 仅红色错误消息是令人担忧。

```powershell
    <#
     .SYNOPSIS
      Example of DMZ and User Defined Routing in an isolated network (Azure only, no hybrid connections)

     .DESCRIPTION
      This script will build out a sample DMZ setup containing:
       - A default storage account for VM disks
       - Three new cloud services
       - Three Subnets (SecNet, FrontEnd, and BackEnd subnets)
       - A Network Virtual Appliance (NVA), in this case a Barracuda NextGen Firewall
       - One server on the FrontEnd Subnet
       - Three Servers on the BackEnd Subnet
       - IP Forwarding from the FireWall out to the internet
       - User Defined Routing FrontEnd and BackEnd Subnets to the NVA

      Before running script, ensure the network configuration file is created in
      the directory referenced by $NetworkConfigFile variable (or update the
      variable to reflect the path and file name of the config file being used).

     .Notes
      Everyone's security requirements are different and can be addressed in a myriad of ways.
      Please be sure that any sensitive data or applications are behind the appropriate
      layer(s) of protection. This script serves as an example of some of the techniques
      that can be used, but should not be used for all scenarios. You are responsible to
      assess your security needs and the appropriate protections needed, and then effectively
      implement those protections.

      Security Service (SecNet subnet 10.0.0.0/24)
       myFirewall - 10.0.0.4

      FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
       IIS01      - 10.0.1.4

      BackEnd Service (BackEnd subnet 10.0.2.0/24)
       DNS01      - 10.0.2.4
       AppVM01    - 10.0.2.5
       AppVM02    - 10.0.2.6

    #>

    # Fixed Variables
        $LocalAdminPwd = Read-Host -Prompt "Enter Local Admin Password to be used for all VMs"
        $VMName = @()
        $ServiceName = @()
        $VMFamily = @()
        $img = @()
        $size = @()
        $SubnetName = @()
        $VMIP = @()

    # User Defined Global Variables
      # These should be changes to reflect your subscription and services
      # Invalid options will fail in the validation section

      # Subscription Access Details
        $subID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

      # VM Account, Location, and Storage Details
        $LocalAdmin = "theAdmin"
        $DeploymentLocation = "Central US"
        $StorageAccountName = "vmstore02"

      # Service Details
        $SecureService = "SecSvc001"
        $FrontEndService = "FrontEnd001"
        $BackEndService = "BackEnd001"

      # Network Details
        $VNetName = "CorpNetwork"
        $VNetPrefix = "10.0.0.0/16"
        $SecNet = "SecNet"
        $FESubnet = "FrontEnd"
        $FEPrefix = "10.0.1.0/24"
        $BESubnet = "BackEnd"
        $BEPrefix = "10.0.2.0/24"
        $NetworkConfigFile = "C:\Scripts\NetworkConf3.xml"

      # VM Base Disk Image Details
        $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
        $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}

      # UDR Details
        $FERouteTableName = "FrontEndSubnetRouteTable"
        $BERouteTableName = "BackEndSubnetRouteTable"

      # NSG Details
        $NSGName = "MyVNetSG"

    # User Defined VM Specific Config
        # Note: To ensure UDR and IP forwarding is setup
        # properly this script requires VM 0 be the NVA.

        # VM 0 - The Network Virtual Appliance (NVA)
          $VMName += "myFirewall"
          $ServiceName += $SecureService
          $VMFamily += "Firewall"
          $img += $FWImg
          $size += "Small"
          $SubnetName += $SecNet
          $VMIP += "10.0.0.4"

        # VM 1 - The Web Server
          $VMName += "IIS01"
          $ServiceName += $FrontEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.4"

        # VM 2 - The First Application Server
          $VMName += "AppVM01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.5"

        # VM 3 - The Second Application Server
          $VMName += "AppVM02"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.6"

        # VM 4 - The DNS Server
          $VMName += "DNS01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.4"

    # ----------------------------- #
    # No User Defined Variables or   #
    # Configuration past this point #
    # ----------------------------- #

      # Get your Azure accounts
        Add-AzureAccount
        Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
        Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop

      # Create Storage Account
        If (Test-AzureName -Storage -Name $StorageAccountName) {
            Write-Host "Fatal Error: This storage account name is already in use, please pick a different name." -ForegroundColor Red
            Return}
        Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan
              New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}

      # Update Subscription Pointer to New Storage Account
        Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
        Set-AzureSubscription –SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop

    # Validation
    $FatalError = $false

    If (-Not (Get-AzureLocation | Where {$_.DisplayName -eq $DeploymentLocation})) {
         Write-Host "This Azure Location was not found or available for use" -ForegroundColor Yellow
         $FatalError = $true}

    If (Test-AzureName -Service -Name $SecureService) { 
        Write-Host "The SecureService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}

    If (Test-AzureName -Service -Name $FrontEndService) { 
        Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use" -ForegroundColor Green}

    If (Test-AzureName -Service -Name $BackEndService) { 
        Write-Host "The BackEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The BackEndService service name is valid for use." -ForegroundColor Green}

    If (-Not (Test-Path $NetworkConfigFile)) { 
        Write-Host 'The network config file was not found, please update the $NetworkConfigFile variable to point to the network config xml file.' -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The network config file was found" -ForegroundColor Green
            If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
                Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation variable is correct and the network config file matches.' -ForegroundColor Yellow
                $FatalError = $true}
            Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}

    If ($FatalError) {
        Write-Host "A fatal error has occurred, please see the above messages for more information." -ForegroundColor Red
        Return}
    Else { Write-Host "Validation passed, now building the environment." -ForegroundColor Green}

    # Create VNET
        Write-Host "Creating VNET" -ForegroundColor Cyan 
        Set-AzureVNetConfig -ConfigurationPath $NetworkConfigFile -ErrorAction Stop

    # Create Services
        Write-Host "Creating Services" -ForegroundColor Cyan
        New-AzureService -Location $DeploymentLocation -ServiceName $SecureService -ErrorAction Stop
        New-AzureService -Location $DeploymentLocation -ServiceName $FrontEndService -ErrorAction Stop
        New-AzureService -Location $DeploymentLocation -ServiceName $BackEndService -ErrorAction Stop

    # Build VMs
        $i=0
        $VMName | Foreach {
            Write-Host "Building $($VMName[$i])" -ForegroundColor Cyan
            If ($VMFamily[$i] -eq "Firewall") 
                { 
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Linux -LinuxUser $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                # Set up all the EndPoints we'll need once we're up and running
                # Note: All traffic goes through the firewall, so we'll need to set up all ports here.
                #       Also, the firewall will be redirecting traffic to a new IP and Port in a forwarding
                #       rule, so all of these endpoint have the same public and local port and the firewall
                #       will do the mapping, NATing, and/or redirection as declared in the firewall rules.
                Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPWeb"    -Protocol tcp -PublicPort 8014 -LocalPort 8014 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPApp1"   -Protocol tcp -PublicPort 8025 -LocalPort 8025 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPApp2"   -Protocol tcp -PublicPort 8026 -LocalPort 8026 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPDNS01"  -Protocol tcp -PublicPort 8024 -LocalPort 8024 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                # Note: A SSH endpoint is automatically created on port 22 when the appliance is created.
                }
            Else
                {
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
                    Remove-AzureEndpoint -Name "RemoteDesktop" | `
                    Remove-AzureEndpoint -Name "PowerShell" | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                }
            $i++
        }

    # Configure UDR and IP Forwarding
        Write-Host "Configuring UDR" -ForegroundColor Cyan

      # Create the Route Tables
        Write-Host "Creating the Route Tables" -ForegroundColor Cyan 
        New-AzureRouteTable -Name $BERouteTableName `
            -Location $DeploymentLocation `
            -Label "Route table for $BESubnet subnet"
        New-AzureRouteTable -Name $FERouteTableName `
            -Location $DeploymentLocation `
            -Label "Route table for $FESubnet subnet"

      # Add Routes to Route Tables
        Write-Host "Adding Routes to the Route Tables" -ForegroundColor Cyan 
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
            -NextHopType VNETLocal
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $FEPrefix `
            -NextHopType VNETLocal

      # Associate the Route Tables with the Subnets
        Write-Host "Binding Route Tables to the Subnets" -ForegroundColor Cyan 
        Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
            -SubnetName $BESubnet `
            -RouteTableName $BERouteTableName
        Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
            -SubnetName $FESubnet `
            -RouteTableName $FERouteTableName

     # Enable IP Forwarding on the Virtual Appliance
        Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] `
            |Set-AzureIPForwarding -Enable

    # Configure NSG
        Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan

      # Build the NSG
        Write-Host "Building the NSG" -ForegroundColor Cyan
        New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"

      # Add NSG Rule
        Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 100 -Action Deny `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
            -Protocol *

      # Assign the NSG to two Subnets
        # The NSG is *not* bound to the Security Subnet. The result
        # is that internet traffic flows only to the Security subnet
        # since the NSG bound to the FrontEnd and BackEnd subnets
        # will Deny internet traffic to those subnets.
        Write-Host "Binding the NSG to two subnets" -ForegroundColor Cyan
        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $FESubnet -VirtualNetworkName $VNetName
        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $BESubnet -VirtualNetworkName $VNetName

    # Optional Post-script Manual Configuration
      # Configure Firewall
      # Install Test Web App (Run Post-Build Script on the IIS Server)
      # Install BackEnd resource (Run Post-Build Script on the AppVM01)
      Write-Host
      Write-Host "Build Complete!" -ForegroundColor Green
      Write-Host
      Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
      Write-Host " - Configure Firewall" -ForegroundColor Gray
      Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
      Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
      Write-Host
```

### <a name="network-config-file"></a>网络配置文件

使用更新的位置保存此 XML 文件。 更改`$NetworkConfigFile`变量完整上述脚本中要链接到已保存的网络配置文件。

```xml
    <NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
      <VirtualNetworkConfiguration>
        <Dns>
          <DnsServers>
            <DnsServer name="DNS01" IPAddress="10.0.2.4" />
            <DnsServer name="Level3" IPAddress="209.244.0.3" />
          </DnsServers>
        </Dns>
        <VirtualNetworkSites>
          <VirtualNetworkSite name="CorpNetwork" Location="Central US">
            <AddressSpace>
              <AddressPrefix>10.0.0.0/16</AddressPrefix>
            </AddressSpace>
            <Subnets>
              <Subnet name="SecNet">
                <AddressPrefix>10.0.0.0/24</AddressPrefix>
              </Subnet>
              <Subnet name="FrontEnd">
                <AddressPrefix>10.0.1.0/24</AddressPrefix>
              </Subnet>
              <Subnet name="BackEnd">
                <AddressPrefix>10.0.2.0/24</AddressPrefix>
              </Subnet>
            </Subnets>
            <DnsServersRef>
              <DnsServerRef name="DNS01" />
              <DnsServerRef name="Level3" />
            </DnsServersRef>
          </VirtualNetworkSite>
        </VirtualNetworkSites>
      </VirtualNetworkConfiguration>
    </NetworkConfiguration>
```

## <a name="next-steps"></a>后续步骤

你可以安装示例应用程序以帮助进行此外围网络示例。

> [!div class="nextstepaction"]
> [示例应用程序脚本](./virtual-networks-sample-app.md)

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3design.png "使用 NVA、NSG 和 UDR 的双向外围网络"
[2]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3firewalllogical.png "防火墙规则的逻辑视图"
[3]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectfrontend.png "创建前端网络对象"
[4]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectdns.png "创建 DNS 服务器对象"
[5]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpa.png "默认 RDP 规则的副本"
[6]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpb.png "AppVM01 规则"
[7]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconapplicationredirect.png "应用程序重定向图标"
[8]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/icondestinationnat.png "目标 NAT 图标"
[9]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconpass.png "传递图标"
[10]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulefirewall.png "防火墙管理规则"
[11]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulerdp.png "防火墙 RDP 规则"
[12]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleweb.png "防火墙 Web 规则"
[13]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleappvm01.png "防火墙 AppVM01 规则"
[14]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleoutbound.png "防火墙出站规则"
[15]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledns.png "防火墙 DNS 规则"
[16]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleintravnet.png "防火墙 VNet 内部规则"
[17]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledeny.png "防火墙拒绝规则"
[18]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/firewallruleactivate.png "防火墙规则激活"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
