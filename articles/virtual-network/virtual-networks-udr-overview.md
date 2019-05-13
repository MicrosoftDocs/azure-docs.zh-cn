---
title: Azure 虚拟网络流量路由
titlesuffix: Azure Virtual Network
description: 了解 Azure 如何路由虚拟网络流量，以及你如何自定义 Azure 的路由。
services: virtual-network
documentationcenter: na
author: malopMSFT
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2017
ms.author: malop;kumud
ms.openlocfilehash: 011ce61f9ac0656db8804c203000f54a7146afe0
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408199"
---
# <a name="virtual-network-traffic-routing"></a>虚拟网络流量路由

了解 Azure 如何在 Azure 资源、本地资源和 Internet 资源之间路由流量。 Azure 自动为 Azure 虚拟网络中的每个子网创建一个路由表，然后向该表添加系统默认路由。 若要详细了解虚拟网络和子网，请参阅[虚拟网络概述](virtual-networks-overview.md)。 可以使用[自定义路由](#custom-routes)替代 Azure 的部分系统路由，并向路由表添加其他自定义路由。 Azure 根据子网的路由表中的路由，从子网路由出站流量。

## <a name="system-routes"></a>系统路由

Azure 自动创建系统路由，并将路由分配到虚拟网络中的每个子网。 你不能创建系统路由，也不能删除系统路由，但可以使用[自定义路由](#custom-routes)替代某些系统路由。 Azure 为每个子网创建默认系统路由，并向特定子网或每个子网添加其他[可选默认路由](#optional-default-routes)，前提是当你使用特定的 Azure 功能时。

### <a name="default"></a>默认

每个路由包含地址前缀和下一跃点类型。 将离开子网的流量发送到某个路由的地址前缀中的 IP 地址时，包含前缀的该路由是 Azure 使用的路由。 详细了解当多个路由包含相同的前缀或重叠前缀时，[Azure 如何选择路由](#how-azure-selects-a-route)。 只要创建了虚拟网络，Azure 就会自动为虚拟网络中的每个子网创建下述默认的系统路由：


|源 |地址前缀                                        |下一个跃点类型  |
|-------|---------                                               |---------      |
|默认|对虚拟网络唯一                           |虚拟网络|
|默认|0.0.0.0/0                                               |Internet       |
|默认|10.0.0.0/8                                              |无           |
|默认|172.16.0.0/12                                           |无           |
|默认|192.168.0.0/16                                          |无           |
|默认|100.64.0.0/10                                           |无           |

上表中列出的下一跃点类型表示 Azure 如何路由目标为所列地址前缀的流量。 下面是下一跃点类型的说明：

- **虚拟网络**：在虚拟网络的[地址空间](manage-virtual-network.md#add-or-remove-an-address-range)中的地址范围之间路由流量。 Azure 使用地址前缀创建路由，该前缀对应的每个地址范围是在虚拟网络的地址空间中定义的。 如果虚拟网络地址空间定义了多个地址范围，Azure 会为每个地址范围创建单个路由。 Azure 使用为每个地址范围创建的路由在子网之间自动路由流量。 不需为 Azure 定义在子网之间路由流量的网关。 虽然虚拟网络包含子网，且每个子网都有定义的地址范围，但 Azure 不为子网地址范围创建默认路由，因为每个子网地址范围都是在虚拟网络地址空间的地址范围之内。

- **Internet**：将地址前缀指定的流量路由到 Internet。 系统默认路由指定 0.0.0.0/0 地址前缀。 如果不替代 Azure 的默认路由，Azure 会将不是通过虚拟网络中的地址范围指定的地址的流量路由到 Internet，但有一个例外。 如果目标地址是用于某个 Azure 服务的，Azure 会将流量通过 Azure 的主干网络直接路由到该服务，而不是将流量路由到 Internet。 Azure 服务之间的流量不跨越 Internet，不管虚拟网络存在于哪个 Azure 区域，也不管 Azure 服务的实例部署在哪个 Azure 区域。 可以将 0.0.0.0/0 地址前缀对应的 Azure 默认系统路由替代为[自定义路由](#custom-routes)。

- **无**：系统会将路由到“无”下一跃点类型的流量删除，而不是将其路由到子网外。 Azure 自动为以下地址前缀创建默认路由：
  - **10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16**：在 RFC 1918 中保留专用。
  - **100.64.0.0/10**：在 RFC 6598 中保留。

    如果将上述任何地址范围分配到虚拟网络的地址空间中，Azure 会自动将路由的下一跃点类型从“无”更改为“虚拟网络”。 如果将地址范围分配到虚拟网络的地址空间时，该地址空间包括四个保留地址前缀中的一个，但与其并不相同，则 Azure 会删除该前缀对应的路由，为你所添加的地址前缀添加一个路由，并使用“虚拟网络”作为下一跃点类型。

### <a name="optional-default-routes"></a>可选默认路由

Azure 会针对不同的 Azure 功能添加其他默认的系统路由，但前提是你启用这些功能。 Azure 会根据功能将可选的默认路由添加到虚拟网络中的特定子网，或者添加到虚拟网络中的所有子网。 启用不同的功能时，Azure 可能添加的其他系统路由和下一跃点类型为：

|源                 |地址前缀                       |下一个跃点类型|向其添加路由的虚拟网络中的子网|
|-----                  |----                                   |---------                    |--------|
|默认                |对虚拟网络唯一，例如：10.1.0.0/16|VNet 对等                 |全部|
|虚拟网络网关|从本地通过 BGP 播发的前缀，或者在本地网关中配置的前缀     |虚拟网络网关      |全部|
|默认                |多个                               |虚拟网络服务终结点|仅为其启用服务终结点的子网。|

- **虚拟网络 (VNet) 对等互连**：在两个虚拟网络之间创建虚拟网络对等互连时，会为每个虚拟网络（已为其创建对等互连）的地址空间中的每个地址范围添加一个路由。 详细了解[虚拟网络对等互连](virtual-network-peering-overview.md)。  
- **虚拟网络网关**：向虚拟网络添加虚拟网关时，会添加一个或多个将“虚拟网关”列为下一跃点类型的路由。 源也是“虚拟网关”，因为网关向子网添加路由。 如果本地网关与 Azure 虚拟网关交换了边界网关协议 ([BGP](#border-gateway-protocol)) 路由，则会为每个从本地网关传播的路由添加一个路由。 建议尽可能汇总最大地址范围的本地路由，尽量减少传播到 Azure 虚拟网关的路由数。 传播到 Azure 虚拟网关的路由存在数量限制。 有关详细信息，请参阅 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)。
- **VirtualNetworkServiceEndpoint**：启用特定服务的服务终结点时，Azure 会将该服务的公共 IP 地址添加到路由表。 服务终结点是为虚拟网络中的单个子网启用的，因此仅将路由添加到为其启用了服务终结点的子网的路由表。 Azure 服务的公共 IP 地址定期更改。 当地址更改时，Azure 自动管理路由表中的地址。 详细了解[虚拟网络服务终结点](virtual-network-service-endpoints-overview.md)，以及可以为其创建服务终结点的服务。 

> [!NOTE]
> “VNet 对等互连”和“VirtualNetworkServiceEndpoint”下一跃点类型仅添加到通过 Azure 资源管理器部署模型创建的虚拟网络中子网的路由表。 下一跃点类型不添加到通过经典部署模型与虚拟网络子网关联的路由表。 详细了解 Azure [部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="custom-routes"></a>自定义路由

创建自定义路由有两种方式：一是创建[用户定义](#user-defined)路由，二是在本地网关和 Azure 虚拟网关之间交换[边界网关协议](#border-gateway-protocol) (BGP) 路由。 
 
### <a name="user-defined"></a>用户定义

可以在 Azure 中创建自定义或用户定义路由，以便替代 Azure 的默认系统路由，或者向子网的路由表添加其他路由。 可以在 Azure 中创建一个路由表，然后将该路由表关联到零个或零个以上的虚拟网络子网。 每个子网可以有一个与之关联的路由表，也可以没有。 若要了解可以添加到路由表的最大路由数，以及可以为每个 Azure 订阅创建的最大用户定义路由表数，请参阅 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)。 如果创建一个路由表并将其关联到子网，则其中的路由会与 Azure 默认情况下添加到子网的默认路由组合在一起，或者将其替代。

可以在创建用户定义路由时指定下面的下一跃点类型：

- **虚拟设备**：虚拟设备是通常运行防火墙等网络应用程序的虚拟机。 若要了解各种可在虚拟网络中部署的预配置网络虚拟设备，请参阅 [Azure 市场](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances)。 使用“虚拟设备”跃点类型创建路由时，也指定下一跃点 IP 地址。 IP 地址可以是：

  - 附加到虚拟机的网络接口的[专用 IP 地址](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses)。 如果网络接口附加到虚拟机，而虚拟机将网络流量转发到不是自己地址的地址，则该网络接口必须为其启用 Azure 选项“启用 IP 转发”。 此设置禁止 Azure 在源和目标中检查网络接口。 详细了解如何[为网络接口启用 IP 转发](virtual-network-network-interface.md#enable-or-disable-ip-forwarding)。 虽然“启用 IP 转发”是一项 Azure 设置，但你也可能需要在虚拟机的操作系统中启用 IP 转发，否则设备无法在分配到 Azure 网络接口的专用 IP 地址之间转发流量。 如果必须将流量路由到公共 IP 地址，则设备需通过代理来路由流量，或者通过网络地址转换将源的专用 IP 地址转换为其自己的专用 IP 地址，然后再由 Azure 将网络地址转换为公共 IP 地址，这样才能将流量发送到 Internet。 若要确定虚拟机中的必需设置，请参阅操作系统或网络应用程序的文档。 若要了解 Azure 中的出站连接，请参阅[了解出站连接](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

    > [!NOTE]
    > 将虚拟设备部署到子网时，该子网应不同于通过虚拟设备路由的资源所部署到的子网。 如果将虚拟设备部署到同一子网，然后将路由表应用到通过虚拟设备路由流量的子网，则可能导致路由循环，使流量无法离开子网。

  - Azure [内部负载均衡器](../load-balancer/load-balancer-get-started-ilb-arm-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的专用 IP 地址。 负载均衡器通常作为[网络虚拟设备的高可用性策略](/azure/architecture/reference-architectures/dmz/nva-ha?toc=%2fazure%2fvirtual-network%2ftoc.json)的一部分使用。

    可以在定义路由时，使用“0.0.0.0/0”作为地址前缀，使用“虚拟设备”作为下一跃点类型，这样设备就可以检查流量，并决定是转发流量还是丢弃流量。 若要创建包含 0.0.0.0/0 地址前缀的用户定义路由，请先阅读 [0.0.0.0/0 地址前缀](#default-route)。

- **虚拟网络网关**：需要将目标为特定地址前缀的流量路由到虚拟网关时，请指定此项。 创建虚拟网关时，类型必须为“VPN”。 不能在用户定义路由中指定将虚拟网关创建为“ExpressRoute”类型，因为类型为 ExpressRoute 时，必须对自定义路由使用 BGP。 可以定义一个路由，让其将目标为 0.0.0.0/0 地址前缀的流量定向到[基于路由](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#vpntype)的虚拟网关。 可以在本地设置一个设备，让其检查流量并决定是转发还是丢弃流量。 若要创建地址前缀为 0.0.0.0/0 的用户定义路由，请先阅读 [0.0.0.0/0 地址前缀](#default-route)。 可以通过 BGP 播发前缀为 0.0.0.0/0 的路由，而不必配置地址前缀为 0.0.0.0/0 的用户定义路由，前提是[为 VPN 虚拟网关启用 BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
- **无**：需要丢弃流向某个地址前缀的流量，而不是将该流量转发到目标时，请指定此项。 如果某项功能尚未完全配置好，Azure 可能会针对部分可选的系统路由列出“无”。 例如，如果看到“无”作为“下一跃点 IP 地址”列出，且“下一跃点类型”为“虚拟网关”或“虚拟设备”，则可能是因为设备未运行或未完全配置好。 Azure 为保留的地址前缀创建系统[默认路由](#default)，使用“无”作为下一跃点类型。
- **虚拟网络**：需要替代虚拟网络中的默认路由时，请指定此项。 请参阅[路由示例](#routing-example)，通过示例了解为何需创建跃点类型为“虚拟网络”的路由。
- **Internet**：需要将目标为某个地址前缀的流量显式路由到 Internet 时，或者需要将流量的目标设定为 Azure 服务，且公共 IP 地址始终位于 Azure 主干网络内时，请指定此项。

不能在用户定义路由中指定“VNet 对等互连”或“VirtualNetworkServiceEndpoint”作为下一跃点类型。 下一跃点类型为“VNet 对等互连”或“VirtualNetworkServiceEndpoint”的路由只能在配置虚拟网络对等互连或服务终结点时，通过 Azure 创建。

**各个 Azure 工具中的下一跃点类型**

下一跃点类型的显示和引用名称在 Azure 门户和命令行工具以及 Azure 资源管理器部署模型和经典部署模型中并不相同。 下表列出了在不同的工具和[部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)中引用每个下一跃点类型时所使用的名称：

|下一个跃点类型                   |Azure CLI 和 PowerShell（资源管理器） |Azure 经典 CLI 和 PowerShell（经典）|
|-------------                   |---------                                       |-----|
|虚拟网络网关         |VirtualNetworkGateway                           |VPNGateway|
|虚拟网络                 |VNetLocal                                       |VNETLocal（不适用于 asm 模式下的经典 CLI）|
|Internet                        |Internet                                        |Internet（不适用于 asm 模式下的经典 CLI）|
|虚拟设备               |VirtualAppliance                                |VirtualAppliance|
|无                            |无                                            |Null（不适用于 asm 模式下的经典 CLI）|
|虚拟网络对等互连         |VNet 对等                                    |不适用|
|虚拟网络服务终结点|虚拟网络服务终结点                   |不适用|

### <a name="border-gateway-protocol"></a>边界网关协议

本地网关可以通过边界网关协议 (BGP) 与 Azure 虚拟网关交换路由。 将 BGP 与 Azure 虚拟网关配合使用的前提是在创建网关时选择了适当的类型。 如果选择的类型是：

- **ExpressRoute**：必须使用 BGP 将本地路由播发到 Microsoft 边缘路由器。 如果将虚拟网关部署为 ExpressRoute 类型，则不能通过创建用户定义路由来强制流量到达 ExpressRoute 虚拟网关。 例如，可以使用用户定义的路由来强制来自 Express Route 的流量到达网络虚拟设备。
- **VPN**：可以选择性地使用 BGP。 有关详细信息，请参阅 [BGP 与站点到站点 VPN 连接配合使用](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

使用 BGP 与 Azure 交换路由时，系统会针对每个播发的前缀，将一个单独的路由添加到包含虚拟网络中所有子网的路由表。 添加路由时，会将*虚拟网关*列为源和下一跃点类型。 

可以使用路由表的一个属性，在子网上禁用 ER 和 VPN 网关路由传播。 使用 BGP 与 Azure 交换路由时，路由不会添加到禁用 BGP 传播的任何子网的路由表。 VPN 连接的连接性是通过下一跃点类型为“虚拟网关”的[自定义路由](#custom-routes)实现的。 有关详细信息，请参阅[如何禁用 BGP 路由传播](manage-route-table.md#create-a-route-table)。

## <a name="how-azure-selects-a-route"></a>Azure 如何选择路由

当出站流量是从子网发送时，Azure 使用最长前缀匹配算法，根据目标 IP 地址选择路由。 例如，一个路由表有两个路由：一个路由指定 10.0.0.0/24 地址前缀，另一个路由指定 10.0.0.0/16 地址前缀。 Azure 会将目标为 10.0.0.5 的流量路由到在路由中指定的地址前缀为 10.0.0.0/24 的下一跃点类型，因为前缀 10.0.0.0/24 比 10.0.0.0/16 长，虽然 10.0.0.5 处于这两个地址前缀范围内。 Azure 会将目标为 10.0.1.5 的流量路由到在路由中指定的地址前缀为 10.0.0.0/16 的下一跃点类型，因为 10.0.1.5 不包括在 10.0.0.0/24 地址前缀中，而地址前缀为 10.0.0.0/16 的路由具有相匹配的最长前缀。

如果多个路由包含同一地址前缀，Azure 根据以下优先级选择路由类型：

1. 用户定义的路由
2. BGP 路由
3. 系统路由

> [!NOTE]
> 即使 BGP 路由更具体，与虚拟网络、虚拟网络对等互连或虚拟网络服务终结点相关的流量的系统路由也仍是首选路由。

例如，路由表包含以下路由：


|源   |地址前缀  |下一个跃点类型           |
|---------|---------         |-------                 |
|默认  | 0.0.0.0/0        |Internet                |
|用户     | 0.0.0.0/0        |虚拟网络网关 |

当流量的目标 IP 地址位于路由表中任何其他路由的地址前缀之外时，Azure 选择源为“用户”的路由，因为用户定义路由的优先级高于系统默认路由。

请参阅[路由示例](#routing-example)，其中有一个全面的路由表，对多种路由进行了说明。

## <a name="default-route"></a>0.0.0.0/0 地址前缀

0.0.0.0/0 地址前缀的路由会指导 Azure 如何路由目标 IP 地址不在子网路由表中任何其他路由的地址前缀中的流量。 创建子网时，Azure 会创建地址前缀为“0.0.0.0/0”且下一跃点类型为“Internet”的[默认](#default)路由。 如果不替代此路由，Azure 会将目标 IP 地址不包括在任何其他路由的地址前缀中的所有流量路由到 Internet。 例外是，目标为 Azure 服务公共 IP 地址的流量仍保留在 Azure 主干网络中，不路由到 Internet。 如果使用[自定义](#custom-routes)路由来替代此路由，系统会将目标地址不在路由表中任何其他路由的地址前缀中的流量发送到网络虚拟设备或虚拟网关，具体取决于在自定义路由中指定了哪一个。

替代 0.0.0.0/0 地址前缀时，除了出站流量从子网流经虚拟网关或虚拟设备，还会在 Azure 的默认路由上发生以下变化： 

- Azure 将所有流量发送到路由中指定的下一跃点类型，包括目标为 Azure 服务公共 IP 地址的流量。 当地址前缀为 0.0.0.0/0 的路由的下一跃点类型为“Internet”时，从子网流向 Azure 服务公共 IP 地址的流量不会离开 Azure 的主干网络，不管虚拟网络或 Azure 服务资源存在于哪个 Azure 区域。 但在创建下一跃点类型为“虚拟网关”或“虚拟设备”的用户定义路由或 BGP 路由时，所有流量都会发送到路由中指定的下一跃点类型，包括发送到 Azure 服务（尚未为其启用[服务终结点](virtual-network-service-endpoints-overview.md)）公共 IP 地址的流量。 如果已为服务启用服务终结点，则不会将目标为该服务的流量路由到地址前缀为 0.0.0.0/0 的路由中的下一跃点类型，因为该服务的地址前缀是在启用服务终结点时 Azure 创建的路由中指定的，并且该服务的地址前缀比 0.0.0.0/0 长。
- 你将不再能够直接从 Internet 访问子网中的资源。 可以从 Internet 直接访问子网中的资源，前提是入站流量在到达虚拟网络中的资源之前，流经地址前缀为 0.0.0.0/0 的路由的下一跃点类型所指定的设备。 如果路由包含下一跃点类型的以下值：
    - **虚拟设备**：设备必须：
        - 可从 Internet 访问
        - 有分配的公共 IP 地址
        - 没有与阻止设备通信的网络安全组规则相关联
        - 不拒绝通信
        - 能够进行网络地址转换和转发，或者能够对流向子网中目标资源的流量进行代理，以及能够让流量返回 Internet。 
    - **虚拟网络网关**：如果网关为 ExpressRoute 虚拟网络网关，则连接了 Internet 的本地设备可以进行网络地址转换和转发，或者通过 ExpressRoute 的[专用对等互连](../expressroute/expressroute-circuit-peerings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#privatepeering)对流向子网中目标资源的流量进行代理。 

如果虚拟网络已连接到 Azure VPN 网关，请不要将路由表与包含目标为 0.0.0.0/0 的路由的[网关子网](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub)相关联。 这样做可能会阻止网关正常工作。 有关详细信息，请参阅 [VPN 网关常见问题解答](../vpn-gateway/vpn-gateway-vpn-faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gatewayports)中的“我的 VPN 网关上的某些端口为何处于打开状态？”问题。

请参阅 [Azure 与本地数据中心之间的外围网络](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid?toc=%2fazure%2fvirtual-network%2ftoc.json)和 [Azure 与 Internet 之间的外围网络](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz?toc=%2fazure%2fvirtual-network%2ftoc.json)，了解在 Internet 和 Azure 之间使用虚拟网关和虚拟设备时的实施详情。

## <a name="routing-example"></a>路由示例

为了说明本文中的概念，下述部分介绍了：
- 带要求的方案
- 条件所要求的自定义路由
- 为一个子网设置的路由表，其中包括条件所要求的默认路由和自定义路由

> [!NOTE]
> 不应将此示例作为建议或最佳做法来实现。 提供此示例只是为了说明本文中的概念。

### <a name="requirements"></a>要求

1. 在同一 Azure 区域实现两个虚拟网络，并允许资源在虚拟网络之间通信。
2. 允许本地网络以安全方式通过 Internet 上的 VPN 隧道与两个虚拟网络通信。 也可使用 ExpressRoute 连接，但在此示例中，使用的是 VPN 连接。
3. 对于一个虚拟网络中有一个子网的情况，请执行以下操作：
 
    - 强制来自子网的所有出站流量（到 Azure 存储和子网内的流量除外）流经网络虚拟设备进行检查和日志记录。
    - 不检查子网中专用 IP 地址之间的流量；允许流量在所有资源之间直接流动。 
    - 丢弃目标为其他虚拟网络的出站流量。
    - 允许目标为 Azure 存储的出站流量直接流向存储，不强制其流经网络虚拟设备。

4. 允许所有其他子网和虚拟网络之间的所有流量。

### <a name="implementation"></a>实现

下图显示了一个通过 Azure 资源管理器部署模型完成的符合上述要求的实现：

![网络示意图](./media/virtual-networks-udr-overview/routing-example.png)

箭头显示流量方向。 

### <a name="route-tables"></a>路由表

#### <a name="subnet1"></a>Subnet1

图中 Subnet1 的路由表包含以下路由：

|ID  |源 |状态  |地址前缀    |下一个跃点类型          |下一个跃点 IP 地址|用户定义路由的名称| 
|----|-------|-------|------              |-------                |--------           |--------      |
|第   |默认|无效|10.0.0.0/16         |虚拟网络        |                   |              |
|2   |用户   |活跃 |10.0.0.0/16         |虚拟设备      |10.0.100.4         |Within-VNet1  |
|3   |用户   |活跃 |10.0.0.0/24         |虚拟网络        |                   |Within-Subnet1|
|4   |默认|无效|10.1.0.0/16         |VNet 对等           |                   |              |
|5   |默认|无效|10.2.0.0/16         |VNet 对等           |                   |              |
|6   |用户   |活跃 |10.1.0.0/16         |无                   |                   |ToVNet2-1-Drop|
|7   |用户   |活跃 |10.2.0.0/16         |无                   |                   |ToVNet2-2-Drop|
|8   |默认|无效|10.10.0.0/16        |虚拟网络网关|[X.X.X.X]          |              |
|9   |用户   |活跃 |10.10.0.0/16        |虚拟设备      |10.0.100.4         |To-On-Prem    |
|10  |默认|活跃 |[X.X.X.X]           |虚拟网络服务终结点    |         |              |
|11  |默认|无效|0.0.0.0/0           |Internet               |                   |              |
|12  |用户   |活跃 |0.0.0.0/0           |虚拟设备      |10.0.100.4         |Default-NVA   |

每个路由 ID 的说明如下所示：

1. Azure 自动为 Virtual-network-1 中的所有子网添加了此路由，因为 10.0.0.0/16 是在虚拟网络的地址空间中定义的唯一地址范围。 如果未在路由 ID2 中创建用户定义路由，则会将发送到 10.0.0.1 和 10.0.255.254 之间地址的流量路由到虚拟网络内部，因为该前缀比 0.0.0.0/0 长，且不在任何其他路由的地址前缀中。 添加用户定义路由 ID2 时，Azure 自动将状态从“活动”更改为“无效”，因为其前缀与默认路由相同，且用户定义路由会替代默认路由。 对于 Subnet2 来说，此路由的状态仍为“活动”，因为用户定义路由 ID2 所在的路由表未关联到 Subnet2。
2. 将地址前缀为 10.0.0.0/16 的用户定义路由关联到 Virtual-network-1 虚拟网络中的 Subnet1 子网时，Azure 添加了此路由。 用户定义路由指定 10.0.100.4 作为虚拟设备的 IP 地址，因为该地址是分配到虚拟设备虚拟机的专用 IP 地址。 此路由所在的路由表未关联到 Subnet2，因此未出现在 Subnet2 的路由表中。 此路由会替代 10.0.0.0/16 前缀的默认路由 (ID1)，后者自动通过虚拟网络下一跃点类型将目标地址为 10.0.0.1 和 10.0.255.254 的流量路由到虚拟网络内部。 此路由存在的原因是为了满足[要求](#requirements) 3，强制所有出站流量流经虚拟设备。
3. 将地址前缀为 10.0.0.0/24 的用户定义路由关联到 Subnet1 子网时，Azure 添加了此路由。 目标为 10.0.0.1 和 10.0.0.254 之间地址的流量保留在子网内，而不是路由到前一规则 (ID2) 中指定的虚拟设备，因为它的前缀比 ID2 路由长。 此路由未关联到 Subnet2，因此未出现在 Subnet2 的路由表中。 对于 Subnet1 中的流量，此路由有效地替代了 ID2 路由。 此路由存在的原因是为了满足[要求](#requirements) 3。
4. 对于 Virtual-network-1 中的所有子网，Azure 在该虚拟网络与 Virtual-network-2 对等互连时自动在 ID 4 和 5 中添加了路由。 *Virtual-network-2* 在其地址空间中有两个地址范围：10.1.0.0/16/16 和 10.2.0.0/16，因此，Azure 为每个范围添加了一个路由。 如果未在路由 ID 6 和 7 中创建用户定义路由，则会将发送到 10.1.0.1-10.1.255.254 和 10.2.0.1-10.2.255.254 之间地址的流量路由到对等互连的虚拟网络，因为该前缀比 0.0.0.0/0 长，且不在任何其他路由的地址前缀中。 添加 ID 6 和 7 中的路由 时，Azure 自动将状态从“活动”更改为“无效”，因为其前缀与 ID 4 和 5 中的路由相同，且用户定义路由会替代默认路由。 在 Id 4 和 5 中路由的状态仍是*Active*有关*Subnet2*，这是因为在中，是用户定义的路由 Id 6 和 7 中的路由表未关联到*Subnet2*. 创建虚拟网络对等互连是为了满足[要求](#requirements) 1。
5. 与 ID4 的说明相同。
6. 将地址前缀为 10.1.0.0/16 和 10.2.0.0/16 的用户定义路由关联到 Subnet1 子网时，Azure 添加了此路由以及 ID7 中的路由。 Azure 放弃目标为 10.1.0.1-10.1.255.254 和 10.2.0.1-10.2.255.254 之间地址的流量，而不是将其路由到对等互连的虚拟网络，因为用户定义路由会替代默认路由。 这些路由未关联到 Subnet2，因此未出现在 Subnet2 的路由表中。 对于离开 Subnet1 的流量，这些路由替代 ID4 和 ID5 路由。 ID6 和 ID7 路由存在的目的是满足[要求](#requirements) 3，丢弃目标为其他虚拟网络的流量。
7. 与 ID6 的说明相同。
8. 在 *Virtual-network-1* 中创建 VPN 类型的虚拟网关时，Azure 自动为该虚拟网络中的所有子网添加了此路由。 Azure 向路由表添加了虚拟网关的公共 IP 地址。 发送到 10.10.0.1 和 10.10.255.254 之间地址的流量路由到虚拟网关。 此前缀比 0.0.0.0/0 长，且不在任何其他路由的地址前缀中。 创建虚拟网关是为了满足[要求](#requirements) 2。
9. 将地址前缀为 10.10.0.0/16 的用户定义路由添加到已关联到 Subnet1 的路由表时，Azure 添加了此路由。 此路由替代 ID8。 此路由将所有目标为本地网络的流量发送到 NVA 进行检查，而不是直接将流量路由到本地。 创建此路由是为了满足[要求](#requirements) 3。
10. 为子网启用 Azure 服务的服务终结点时，Azure 自动将此路由添加到了子网。 Azure 通过 Azure 基础结构网络将流量从子网路由到服务的公共 IP 地址。 此前缀比 0.0.0.0/0 长，且不在任何其他路由的地址前缀中。 创建服务终结点是为了满足[要求](#requirements) 3，允许目标为 Azure 存储的流量直接流向 Azure 存储。
11. Azure 自动向 Virtual-network-1 和 Virtual-network-2 中所有子网的路由表添加了此路由。 0.0.0.0/0 地址前缀是最短的前缀。 发送到更长地址前缀中地址的流量根据其他路由来路由。 默认情况下，如果流量的目标地址不同于在某个其他路由中指定的地址，Azure 会将所有这些流量路由到 Internet。 将地址前缀为 0.0.0.0/0 的用户定义路由 (ID12) 关联到 Subnet1 子网时，Azure 自动将该子网的状态从“活动”更改为“无效”。 对于这两个虚拟网络中的所有其他子网，此路由的状态仍为“活动”，因为此路由未关联到任何其他虚拟网络中的任何其他子网。
12. 将地址前缀为 0.0.0.0/0 的用户定义路由关联到 Subnet1 子网时，Azure 添加了此路由。 用户定义路由指定 10.0.100.4 作为虚拟设备的 IP 地址。 此路由未关联到 Subnet2，因此未出现在 Subnet2 的路由表中。 地址未包括在任何其他路由的地址前缀中的所有流量都发送到虚拟设备。 对于 Subnet1，添加此路由后，地址前缀为 0.0.0.0/0 的默认路由 (ID11) 的状态就从“活动”变成了“无效”，因为用户定义路由会替代默认路由。 此路由存在的原因是为了满足[要求](#requirements) 3。

#### <a name="subnet2"></a>Subnet2

图中 Subnet2 的路由表包含以下路由：

|源  |状态  |地址前缀    |下一个跃点类型             |下一个跃点 IP 地址|
|------- |-------|------              |-------                   |--------           
|默认 |活跃 |10.0.0.0/16         |虚拟网络           |                   |
|默认 |活跃 |10.1.0.0/16         |VNet 对等              |                   |
|默认 |活跃 |10.2.0.0/16         |VNet 对等              |                   |
|默认 |活跃 |10.10.0.0/16        |虚拟网络网关   |[X.X.X.X]          |
|默认 |活跃 |0.0.0.0/0           |Internet                  |                   |
|默认 |活跃 |10.0.0.0/8          |无                      |                   |
|默认 |活跃 |100.64.0.0/10       |无                      |                   |
|默认 |活跃 |172.16.0.0/12       |无                      |                   |
|默认 |活跃 |192.168.0.0/16      |无                      |                   |

Subnet2 的路由表包含所有 Azure 创建的默认路由，以及可选的 VNet 对等互连和虚拟网关可选路由。 向虚拟网络添加网关和对等互连时，Azure 向虚拟网络中的所有子网添加了可选路由。 向 Subnet1 添加地址前缀为 0.0.0.0/0 的用户定义路由时，Azure 从 Subnet1 路由表中删除了地址前缀为 10.0.0.0/8、172.16.0.0/12、192.168.0.0/16、100.64.0.0/10 的路由。  

## <a name="next-steps"></a>后续步骤

- [创建包含路由和网络虚拟设备的用户定义路由表](tutorial-create-route-table-portal.md)
- [为 Azure VPN 网关配置 BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [将 BGP 与 ExpressRoute 配合使用](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#route-aggregation-and-prefix-limits)
- [查看子网的所有路由](diagnose-network-routing-problem.md)。 用户定义的路由表仅显示子网的用户定义路由，不显示其默认路由和 BGP 路由。 查看所有路由时，会显示网络接口所在子网的默认路由、BGP 路由和用户定义路由。
- [确定下一跃点类型](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json)（虚拟机和目标 IP 地址之间）。 Azure 网络观察程序下一跃点功能用于确定流量是否离开子网并路由到你所期望的位置。
