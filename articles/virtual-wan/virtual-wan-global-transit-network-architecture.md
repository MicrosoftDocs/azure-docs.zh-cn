---
title: Azure 虚拟 WAN 的全局传输网络体系结构 |Microsoft Docs
description: 了解全局传输网络体系结构适用于虚拟 WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: article
ms.date: 05/06/2019
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to understand global transit network architecture as it relates to Virtual WAN.
ms.openlocfilehash: 8cda617ca60a17fceaaa818480ff9bbaef46c3fd
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65414053"
---
# <a name="global-transit-network-architecture-and-virtual-wan"></a>全局传输网络体系结构和虚拟 WAN

全局传输网络体系结构是被采用的企业能够合并、 连接和控制以云为中心的现代企业 IT 足迹。 在现代以云为中心 enterprise 中，网络流量不需要为返回到总部。 全局传输网络体系结构基于熟悉的网络概念，以及是唯一的云和基于云的体系结构的新概念。

![体系结构](./media/virtual-wan-global-transit-network-architecture/architecture2.png)

图 1：**全局传输网络与虚拟 WAN**

现代企业中要求跨云和本地之间超分布式应用程序、 数据和用户无所不在的连接。 Azure 虚拟 WAN 的 Vnet、 站点、 应用程序和用户的全球分布式集之间的通用、 任意到任意连接，从而允许全局传输网络体系结构。 Azure 虚拟 WAN 是 Microsoft 托管的服务。 所有网络组件，其中包含此服务托管和由 Microsoft 管理。 有关虚拟 WAN 的详细信息，请参阅[虚拟 WAN 概述](virtual-wan-about.md)一文。

在 Azure 虚拟 WAN 体系结构，Azure 区域作为可连接你的分支的中心。 分支连接后，你可以利用 Azure 的主干，若要建立分支 VNet 和 （可选） 分支到分支的连接。

您可以通过在具有最大数目的辐射 （分支，Vnet，用户），在区域中创建单个虚拟 WAN 中心，然后将连接到中心的其他区域中的辐射建立虚拟 WAN。 或者，如果地理上分散辐射，您可以实例化区域中心而互连的中心。 中心都属于同一个虚拟 WAN，但它们可以与不同区域的策略相关联。

## <a name="hub"></a>中心辐射型传输

全局传输网络体系结构基于其中托管的云网络中心可实现可以分布在不同类型的辐射的终结点之间的可传递的连接的经典的中心辐射连接模型。
  
在此模型中，辐射可以是：

* 虚拟网络 (Vnet)
* 物理 branch 站点
* 远程用户
* Internet

![中心和辐射全局传输关系图](./media/virtual-wan-global-transit-network-architecture/architecture.png)

图 2：**中心辐射型**

图 2 显示地理位置分散的用户、 物理站点和 Vnet 互连通过在云中托管的网络中心的全球网络的逻辑视图。 此体系结构可实现逻辑的一个跃点传输网络终结点之间的连接。 辐射的各种 Azure 网络服务等 ExpressRoute 或站点到站点 VPN 的物理分支，VNet 对等互连的 Vnet 以及为远程用户的点到站点 VPN 连接到中心。

## <a name="crossregion"></a>跨区域连接

对于企业来说，云足迹通常遵循的物理覆盖。 大多数企业从靠近其物理站点和用户的区域访问云。 密钥的主体的全球网络体系结构之一是启用网络实体和终结点之间的跨区域连接。 云足迹可以跨多个区域。 这意味着从连接到一个区域中云的分支的流量可以到达另一个分支或不同区域中的 VNet。

## <a name="any"></a>任意到任意连接

全局传输网络体系结构使*任意到任意连接*通过中心网络中心。 此体系结构可以消除或减少需要完整网格或构建和维护更复杂的部分的网格连接模型。 此外，在中心辐射与网格网络路由控件是更轻松地配置和维护。

任意到任意连接的全局体系结构上下文中允许使用全球分布的用户、 分支、 数据中心、 Vnet 和应用程序通过传输中心连接到每个其他企业。 传输中心充当全局传输系统。

![通信路径](./media/virtual-wan-global-transit-network-architecture/trafficpath.png)

图 3：**虚拟 WAN 的通信路径**

Azure 虚拟 WAN 支持以下全局传输连接路径。 在括号中的字母将映射到图 3。

* 分支到 VNet (a)  
* 分支到分支 (b)
* 远程用户到 VNet (c)
* 远程用户到分支 (d)
* VNet 到 VNet 使用 VNet 对等互连 (e)
* ExpressRoute 的全球性覆盖 

### <a name="branchvnet"></a>分支到 VNet

分支 VNet 是支持的 Azure 虚拟 WAN 的主要路径。 此路径可以连接到 Azure IAAS 企业工作负荷的 Azure Vnet 中部署的分支。 分支可以连接到虚拟 WAN 通过 ExpressRoute 或站点到站点 VPN。 流量传输到 Vnet 连接到 VNet 连接通过虚拟 WAN 中心。

### <a name="branchbranch"></a>分支到分支

分支可以连接到 Azure 虚拟 WAN 中心使用 ExpressRoute 线路和/或站点到站点 VPN 连接。 您可以连接到分支到最靠近的区域中的虚拟 WAN 中心的分支。

此选项允许企业利用 Azure 的主干，若要连接的分支。 但是，即使有此功能，您应权衡通过 Azure 虚拟 WAN 连接的分支，还是要使用专用 WAN 的优势。

### <a name="usertovnet"></a>远程用户到 VNet

可以启用对 Azure 中使用从远程用户客户端与虚拟 WAN 的点到站点连接的直接、 安全远程访问。 企业远程用户不必再参加 hairpin 到云使用公司的 VPN。

### <a name="usertobranch"></a>远程用户到分支

远程用户到分支路径使得远程用户可以通过传输在云中使用点到站点连接到 Azure 的访问的本地工作负荷和应用程序。 此路径，远程用户访问工作负载部署在 Azure 和本地灵活。 企业可以在 Azure 虚拟 WAN 的中心的基于云的安全远程访问服务。

### <a name="vnetvnet"></a>使用 VNet 对等互连的 VNet 到 VNet 传输

若要将 Vnet 连接到彼此，为了支持在多个 Vnet 之间实现的多层应用程序，请使用 VNet 对等互连。 通过 Azure 虚拟 WAN 的 VNet 到 VNet 传输方案目前不支持，但在 Azure 的计划之中。 通过 VNet 对等互连连接 Vnet 是建议的解决办法的需要相互连接的 Vnet。 VNet 对等互连的详细信息，请参阅[VNet 对等互连概述](../virtual-network/virtual-network-peering-overview.md)。

### <a name="globalreach"></a>ExpressRoute 的全球性覆盖

ExpressRoute 是私有的、 有弹性的方式来将你的本地网络连接到 Microsoft 云。 ExpressRoute 全球覆盖是 ExpressRoute 一项附加功能。 与全局访问，可以链接在一起，做成的本地网络之间的专用网络的 ExpressRoute 线路。 连接到 Azure 虚拟 WAN 使用 ExpressRoute 的分支需要 ExpressRoute 全球覆盖与彼此进行通信。

在此模型中，连接到使用 ExpressRoute 的虚拟 WAN 中心每个分支可以连接到 Vnet 使用的分支 VNet 路径。 分支到分支流量不会传输到中心，因为 ExpressRoute 全球覆盖支持通过 Azure WAN 更优的路径。

## <a name="security"></a>安全和策略控制

虚拟网络中心的互连，并可能会看到传输的所有流量。 它可以是主机中心的网络功能和服务，例如此类云路由、 网络策略和安全性和 Internet 访问控制的位置。

## <a name="next-steps"></a>后续步骤

创建使用虚拟 WAN 连接。

* [使用虚拟 WAN 的站点到站点连接](virtual-wan-site-to-site-portal.md)
* [点到站点连接使用虚拟 WAN](virtual-wan-point-to-site-portal.md)
* [使用虚拟 WAN 的 ExpressRoute 连接](virtual-wan-expressroute-portal.md)
