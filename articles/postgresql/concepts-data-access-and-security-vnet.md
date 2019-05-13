---
title: 虚拟网络 (VNet) 服务终结点概述在 Azure Database for PostgreSQL-单个服务器
description: 了解虚拟网络 (VNet) 服务终结点如何支持 Azure Database for PostgreSQL-单个服务器。
author: bolzmj
ms.author: mbolz
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: c873abcdf3c64f8357c584c8e24809270946bf74
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073476"
---
# <a name="use-virtual-network-service-endpoints-and-rules-for-azure-database-for-postgresql---single-server"></a>用于对 Azure Database for PostgreSQL-单个服务器使用虚拟网络服务终结点和规则

虚拟网络规则是一种防火墙安全功能，用于控制是否允许 Azure Database for PostgreSQL 服务器接受从虚拟网络中的特定子网发送的通信。 本文说明了为何有时候最好选择虚拟网络规则功能来安全地允许与 Azure Database for PostgreSQL 服务器的通信。

若要创建虚拟网络规则，首先必须存在可供规则引用的[虚拟网络][vm-virtual-network-overview] (VNet) 和[虚拟网络服务终结点][vm-virtual-network-service-endpoints-overview-649d]。 下图说明了虚拟网络服务终结点如何与 Azure Database for PostgreSQL 配合使用：

![VNet 服务终结点的工作原理示例](media/concepts-data-access-and-security-vnet/vnet-concept.png)

> [!NOTE]
> Azure 公有云中的所有区域均提供此功能，其中 Azure Database for PostgreSQL 部署用于常规用途和内存优化服务器。
> 在 VNet 对等互连的情况下，如果流量通过具有服务终结点的公共 VNet 网关流动，并且应该流向对等机，请创建 ACL/VNet 规则，以便网关 VNet 中的 Azure 虚拟机能够访问 Azure Database for PostgreSQL 服务器。

<a name="anch-terminology-and-description-82f" />

## <a name="terminology-and-description"></a>术语和说明

**虚拟网络：** 可以让虚拟网络与 Azure 订阅相关联。

**子网：** 虚拟网络包含**子网**。 你所拥有的任何 Azure 虚拟机 (VM) 都会分配到子网。 一个子网可能包含多个 VM 或其他计算节点。 虚拟网络之外的计算节点不能访问虚拟网络，除非已将安全性配置为允许这样的访问。

**虚拟网络服务终结点：**[虚拟网络服务终结点][vm-virtual-network-service-endpoints-overview-649d]是一个子网，其属性值包括一个或多个正式的 Azure 服务类型名称。 本文介绍 **Microsoft.Sql** 的类型名称，即名为“SQL 数据库”的 Azure 服务。 此服务标记也适用于 Azure Database for PostgreSQL 和 MySQL 服务。 务必要注意的一点是，将 Microsoft.Sql 服务标记应用到 VNet 服务终结点时，它将为子网上的所有 Azure SQL 数据库、Azure Database for PostgreSQL 和 Azure Database for MySQL 服务器配置服务终结点流量。 

**虚拟网络规则：** 适用于 Azure Database for PostgreSQL 服务器的虚拟网络规则是一个子网，该子网列在 Azure Database for PostgreSQL 服务器的访问控制列表 (ACL) 中。 该子网必须包含“Microsoft.Sql”类型名称才会列在 Azure Database for PostgreSQL 服务器的 ACL 中。

虚拟网络规则要求 Azure Database for PostgreSQL 服务器接受来自该子网上所有节点的通信。







<a name="anch-benefits-of-a-vnet-rule-68b" />

## <a name="benefits-of-a-virtual-network-rule"></a>虚拟网络规则的优势

在你执行相应操作之前，子网上的 VM 不能与 Azure Database for PostgreSQL 服务器通信。 建立通信的一项操作是创建虚拟网络规则。 若要弄清楚为何选择 VNet 规则方法，必须进行一个比较和对比式的讨论，其中涉及到防火墙提供的竞争性安全选项。

### <a name="a-allow-access-to-azure-services"></a>A. 允许访问 Azure 服务

“连接安全性”窗格有一个标签为“允许访问 Azure 服务”的“启用/禁用”按钮。 “启用”设置允许来自所有 Azure IP 地址和所有 Azure 子网的通信。 这些 Azure IP 或子网可能不是你所拥有的。 此“启用”设置可能超出你在开放方面对 Azure Database for PostgreSQL 数据库的需要。 虚拟网络规则功能提供精细得多的控制。

### <a name="b-ip-rules"></a>B. IP 规则

可以通过 Azure Database for PostgreSQL 防火墙指定 IP 地址范围，处于该范围内的通信允许进入 Azure Database for PostgreSQL 数据库。 此方法适用于 Azure 专用网络外部的稳定 IP 地址。 但是，Azure 专用网络内部的许多节点都配置了动态 IP 地址。 某些情况下（例如重启 VM 时），动态 IP 地址可能会变化。 处于生产环境中时，在防火墙规则中指定一个动态 IP 地址并不明智。

可以通过获取 VM 的静态 IP 地址，对 IP 选项进行“补救”。 有关详细信息，请参阅[使用 Azure 门户为虚拟机配置专用 IP 地址][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]。

但是，静态 IP 方法可能会变得难以管理，在规模大时操作成本高。 虚拟网络规则更易于制定和管理。

### <a name="c-cannot-yet-have-azure-database-for-postgresql-on-a-subnet-without-defining-a-service-endpoint"></a>C. 在没有定义服务终结点的情况下，子网上还不能有 Azure Database for PostgreSQL

如果 **Microsoft.Sql** 服务器是虚拟网络子网上的一个节点，则该虚拟网络中的所有节点都可以与 Azure Database for PostgreSQL 服务器通信。 在这种情况下，VM 可以与 Azure Database for PostgreSQL 通信，而不需要任何虚拟网络规则或 IP 规则。

但截至 2018 年 8 月，Azure Database for PostgreSQL 服务仍然无法直接分配给子网。

<a name="anch-details-about-vnet-rules-38q" />

## <a name="details-about-virtual-network-rules"></a>虚拟网络规则详细信息

此部分介绍虚拟网络规则的多项详细信息。

### <a name="only-one-geographic-region"></a>只有一个地理区域

一个虚拟网络服务终结点只能应用于一个 Azure 区域。 终结点不允许其他区域接受来自该子网的通信。

任何虚拟网络规则都只能应用于基础终结点应用到的区域。

### <a name="server-level-not-database-level"></a>服务器级而非数据库级

每个虚拟网络规则都适用于整个 Azure Database for PostgreSQL 服务器，而不仅仅是该服务器上某个特定数据库。 换句话说，虚拟网络规则适用于服务器级而非数据库级。

#### <a name="security-administration-roles"></a>安全管理角色

在管理虚拟网络服务终结点时，安全角色是分开的。 下述每个角色都需要进行操作：

- **网络管理员：**&nbsp; 启用终结点。
- **数据库管理员：**&nbsp; 更新访问控制列表 (ACL)，将给定的子网添加到 Azure Database for PostgreSQL 服务器。

RBAC 备用：

网络管理员和数据库管理员角色的权限超出虚拟网络规则的管理需要， 只有部分权限是必需的。

可以选择在 Azure 中使用[基于角色的访问控制 (RBAC)][rbac-what-is-813s]，创建一个只有部分必需权限的自定义角色。 在涉及到网络管理员或数据库管理员时，可以使用自定义角色来代替。与向两个主要的管理员角色添加用户相比，向自定义角色添加用户的安全风险较低。

> [!NOTE]
> 在某些情况下，Azure Database for PostgreSQL 和 VNet 子网位于不同的订阅中。 在这些情况下，必须确保以下配置：
> - 两个订阅都必须属于同一 Azure Active Directory 租户。
> - 用户具有启动操作所需的权限，例如启用服务终结点，以及向给定服务器添加 VNet-子网。

## <a name="limitations"></a>限制

对于 Azure Database for PostgreSQL，虚拟网络规则功能具有以下限制：

- Web 应用可以映射到 VNet/子网中的专用 IP。 即使已从给定 VNet/子网启用服务终结点，从 Web 应用到服务器的连接也将具有 Azure 公共 IP 源，而不是 VNet/子网源。 若要启用从 Web 应用到具有 VNet 防火墙规则的服务器的连接，必须在该服务器上允许 Azure 服务访问服务器。

- 在 Azure Database for PostgreSQL 的防火墙中，每个虚拟网络规则都引用一个子网。 引用的所有这些子网都必须托管在同一个托管 Azure Database for PostgreSQL 的地理区域内。

- 对于任何给定的虚拟网络，每个 Azure Database for PostgreSQL 服务器最多可以有 128 个 ACL 条目。

- 虚拟网络规则仅适用于 Azure 资源管理器虚拟网络，不适用于[经典部署模型][arm-deployment-model-568f]网络。

- 使用 **Microsoft.Sql** 服务标记为 Azure Database for PostgreSQL 启用虚拟网络服务终结点也会为以下所有 Azure 数据库服务启用终结点：Azure Database for MySQL、Azure Database for PostgreSQL、Azure SQL 数据库和 Azure SQL 数据仓库。

- 只有常规用途和内存优化服务器才支持 VNet 服务终结点。

- 在防火墙上，IP 地址范围适用于以下网络项，但虚拟网络规则并不适用：
    - [站点到站点 (S2S) 虚拟专用网络 (VPN)][vpn-gateway-indexmd-608y]
    - 通过 [Expressroute][expressroute-indexmd-744v] 进行的本地连接

## <a name="expressroute"></a>ExpressRoute

如果网络通过使用 [ ExpressRoute ][expressroute-indexmd-744v] 连接到 Azure 网络，则每个线路在 Microsoft Edge 配置有两个公共 IP 地址。 这两个 IP 地址用于通过使用 Azure 公共对等互连连接到 Azure 存储等 Microsoft 服务。

若要允许从线路到 Azure Database for PostgreSQL 的通信，则必须为线路的公共 IP 地址创建 IP 网络规则。 为查找 ExpressRoute 线路的公共 IP 地址，请使用 Azure 门户开具 ExpressRoute 支持票证。

## <a name="adding-a-vnet-firewall-rule-to-your-server-without-turning-on-vnet-service-endpoints"></a>在未打开 VNET 服务终结点的情况下，将 VNET 防火墙规则添加到服务器

仅设置防火墙规则无助于保护连接到 VNet 的服务器。 还必须打开 VNet 服务终结点才能确保安全。 打开服务终结点后，VNet 子网会出现停机，直到它从“关”的状态变成了“开”。 这在大型 VNet 的上下文中尤其如此。 可以使用 **IgnoreMissingServiceEndpoint** 标志，减少或消除转换期间的停机时间。

可使用 Azure CLI 或门户设置 IgnoreMissingServiceEndpoint 标志。

## <a name="related-articles"></a>相关文章
- [Azure 虚拟网络][vm-virtual-network-overview]
- [Azure 虚拟网络服务终结点][vm-virtual-network-service-endpoints-overview-649d]

## <a name="next-steps"></a>后续步骤
有关创建 VNet 规则的文章，请参阅：
- [使用 Azure 门户创建和管理 Azure Database for PostgreSQL VNet 规则](howto-manage-vnet-using-portal.md)
- [使用 Azure CLI 创建和管理 Azure Database for PostgreSQL VNet 规则](howto-manage-vnet-using-cli.md)


<!-- Link references, to text, Within this same GitHub repo. -->
[arm-deployment-model-568f]: ../azure-resource-manager/resource-manager-deployment-model.md

[vm-virtual-network-overview]: ../virtual-network/virtual-networks-overview.md

[vm-virtual-network-service-endpoints-overview-649d]: ../virtual-network/virtual-network-service-endpoints-overview.md

[vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]: ../virtual-network/virtual-networks-static-private-ip-arm-pportal.md

[rbac-what-is-813s]: ../active-directory/role-based-access-control-what-is.md

[vpn-gateway-indexmd-608y]: ../vpn-gateway/index.yml

[expressroute-indexmd-744v]: ../expressroute/index.yml
