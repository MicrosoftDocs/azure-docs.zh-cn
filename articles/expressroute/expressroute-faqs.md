---
title: 常见问题解答 - Azure ExpressRoute | Microsoft Docs
description: ExpressRoute 常见问题包含有关支持的 Azure 服务、费用、数据和连接、SLA、提供程序和位置、带宽的信息和其他技术详细信息。
services: expressroute
author: jaredr80
ms.service: expressroute
ms.topic: conceptual
ms.date: 04/16/2019
ms.author: jaredro
ms.custom: seodec18
ms.openlocfilehash: 17f596abda4febbd6e532adcb44d544cd4d104df
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64924670"
---
# <a name="expressroute-faq"></a>ExpressRoute 常见问题

## <a name="what-is-expressroute"></a>什么是 ExpressRoute？

ExpressRoute 是一项 Azure 服务，允许在 Microsoft 数据中心与本地环境或共同租用设施中的基础结构之间创建专用连接。 ExpressRoute 连接不通过公共 Internet，与通过 Internet 的典型连接相比，提供更高的安全性、可靠性、速度和更低的延迟。

### <a name="what-are-the-benefits-of-using-expressroute-and-private-network-connections"></a>使用 ExpressRoute 和专用网络连接的好处是什么？

ExpressRoute 连接不通过公共 Internet 。 与通过 Internet 的典型连接相比，ExpressRoute 连接提供更高的安全性、可靠性、速度和较低的一致延迟。 在某些情况下，使用 ExpressRoute 连接在本地设备和 Azure 之间传输数据可以产生显著的成本效益。

### <a name="where-is-the-service-available"></a>哪里提供该服务？

请参阅此页面以了解服务位置和可用性：[ExpressRoute 合作伙伴和位置](expressroute-locations.md)。

### <a name="how-can-i-use-expressroute-to-connect-to-microsoft-if-i-dont-have-partnerships-with-one-of-the-expressroute-carrier-partners"></a>我如果未与 ExpressRoute 运营商合作伙伴之一建立合作伙伴关系，则如何使用 ExpressRoute 连接到 Microsoft？

可以将区域运营商和地区以太网连接选择为支持的 Exchange 提供商位置之一。 然后，你可以在提供商位置与 Microsoft 实现对接。 查看 [ExpressRoute 合作伙伴和位置](expressroute-locations.md)的最后一部分，以确定服务提供商是否处在任何 Exchange 位置中。 然后，你可以通过服务提供商订购一条 ExpressRoute 线路以连接到 Azure。

### <a name="how-much-does-expressroute-cost"></a>ExpressRoute 的费用是多少？

有关定价信息，请查看[定价详细信息](https://azure.microsoft.com/pricing/details/expressroute/)。

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-does-the-vpn-connection-i-purchase-from-my-network-service-provider-have-to-be-the-same-speed"></a>如果我购买了具有给定带宽的 ExpressRoute 线路，我必须从网络服务提供商购买具有相同速度的 VPN 连接吗？

不。 可以从服务提供商购买任何速度的 VPN 连接。 但是，与 Azure 的连接速度限制为购买的 ExpressRoute 线路带宽。

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-do-i-have-the-ability-to-burst-up-to-higher-speeds-if-necessary"></a>如果我购买了具有给定带宽的 ExpressRoute 线路，是否可以根据需要提升到更高的速度？

 是的。 ExpressRoute 线路的配置允许免费将速度提升到所购带宽限制的两倍。 请咨询你的服务提供商，确定他们是否支持此功能。 这不是时间很长一段并不保证。 

### <a name="can-i-use-the-same-private-network-connection-with-virtual-network-and-other-azure-services-simultaneously"></a>能否同时与虚拟网络和其他 Azure 服务使用同一专用网络连接？

可以。 设置 ExpressRoute 线路后，可以同时访问虚拟网络中的服务和其他 Azure 服务。 通过专用对等路径可连接到虚拟网络，通过 Microsoft 对等路径可连接到其他服务。

### <a name="does-expressroute-offer-a-service-level-agreement-sla"></a>ExpressRoute 是否提供服务级别协议 (SLA)？

有关信息，请参阅 [ExpressRoute SLA](https://azure.microsoft.com/support/legal/sla/) 页。

## <a name="supported-services"></a>支持的服务

ExpressRoute 对各种服务类型支持[三个路由域](expressroute-circuit-peerings.md)。

### <a name="private-peering"></a>专用对等互连

* 虚拟网络，包括所有虚拟机和云服务

### <a name="public-peering"></a>公共对等互连

>[!NOTE]
>在新的 ExpressRoute 线路上已禁用公共对等互连。 Azure 服务在 Microsoft 对等互连上可用。
>

* Power BI
* Dynamics 365 for Finance and Operations（以前称为 Dynamics AX Online）
* 支持大多数 Azure 服务。 请直接对要使用的服务进行确认来验证是否支持。<br><br>
  **不支持以下服务**：
    * CDN
    * 多重身份验证
    * 流量管理器

### <a name="microsoft-peering"></a>Microsoft 对等互连

* [Office 365](https://aka.ms/ExpressRouteOffice365)
* Dynamics 365 
* Power BI
* Azure Active Directory
* [Azure DevOps](https://blogs.msdn.microsoft.com/devops/2018/10/23/expressroute-for-azure-devops/)（Azure 全球服务社区）
* 支持大多数 Azure 服务。 请直接对要使用的服务进行确认来验证是否支持。<br><br>**不支持以下服务**：
    * CDN
    * 多重身份验证
    * 流量管理器

## <a name="data-and-connections"></a>数据和连接

### <a name="are-there-limits-on-the-amount-of-data-that-i-can-transfer-using-expressroute"></a>对于使用 ExpressRoute 可以传输的数据量是否有限制？

我们不对数据传输量设置限制。 有关带宽价格的信息，请参阅[定价详细信息](https://azure.microsoft.com/pricing/details/expressroute/)。

### <a name="what-connection-speeds-are-supported-by-expressroute"></a>ExpressRoute 支持的连接速度是多少？

支持的带宽如下：

50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、2 Gbps、5 Gbps、10 Gbps

### <a name="which-service-providers-are-available"></a>可以选择哪些服务提供商？

有关服务提供商和位置的列表，请参阅 [ExpressRoute 合作伙伴和位置](expressroute-locations.md)。

## <a name="technical-details"></a>技术详细信息

### <a name="what-are-the-technical-requirements-for-connecting-my-on-premises-location-to-azure"></a>将本地位置连接到 Azure 有哪些技术要求？

有关要求，请参阅 [ExpressRoute 先决条件页](expressroute-prerequisites.md)。

### <a name="are-connections-to-expressroute-redundant"></a>与 ExpressRoute 的连接是冗余的吗？

可以。 每条 ExpressRoute 线路都配置了一对冗余的交叉连接，以便提供高可用性。

### <a name="will-i-lose-connectivity-if-one-of-my-expressroute-links-fail"></a>如果我的某个 ExpressRoute 链路出现故障，我会失去连接吗？

如果其中一个交叉连接出现故障，不会失去连接。 可使用冗余连接，以支持网络负载和提供 ExpressRoute 线路的高可用性。 另外，还可以在不同对等位置创建一条线路以获得线路级恢复能力。

### <a name="how-do-i-implement-redundancy-on-private-peering"></a>如何在专用对等互连上实现冗余？

从不同的对等互连位置的多条 ExpressRoute 线路可以连接到同一虚拟网络提供高可用性在这种情况单个线路变得不可用。 然后，可以[较高权重分配](https://docs.microsoft.com/azure/expressroute/expressroute-optimize-routing#solution-assign-a-high-weight-to-local-connection)倾向于选择的本地连接到首选特定的线路。 强烈建议客户安装至少两个 ExpressRoute 线路，以避免单点故障。 

### <a name="how-i-do-implement-redundancy-on-microsoft-peering"></a>如何冗余实现 Microsoft 对等互连？

强烈建议客户使用 Microsoft 对等互连才能访问 Azure 公共服务，如 Azure 存储或 Azure SQL，以及客户使用 Microsoft Office 365 它们不同对等互连中实现多条线路的对等互连时若要避免单点 faiure 的位置。 客户可以将这两个线路上相同的前缀播发并使用[AS PATH 追加](https://docs.microsoft.com/azure/expressroute/expressroute-optimize-routing#solution-use-as-path-prepending)或播发前缀不同，若要确定从本地路径。

### <a name="how-do-i-ensure-high-availability-on-a-virtual-network-connected-to-expressroute"></a>如何确保连接到 ExpressRoute 的虚拟网络上的高可用性？

通过将不同的对等互连位置（例如，新加坡、新加坡 2）中的 ExpressRoute 线路连接到虚拟网络可以实现高可用性。 如果一条 ExpressRoute 线路出现故障，连接会故障转移到另一条 ExpressRoute 线路。 默认情况下，将基于等成本多路径路由 (ECMP) 对离开虚拟网络的流量进行路由。 可以使用连接权重来使一条线路优先于另一条线路。 有关详细信息，请参阅[优化 ExpressRoute 路由](expressroute-optimize-routing.md)。

### <a name="onep2plink"></a>如果我不在云交换中共置，而我的服务提供商提供点到点连接，我需要在本地网络与 Microsoft 之间订购两个物理连接吗？

如果服务提供商可以通过物理连接建立两条以太网虚拟线路，则只需要一个物理连接。 物理连接（例如光纤）的终点在第 1 层 (L1) 设备上（请参阅下图）。 两条以太网虚拟线路使用不同的 VLAN ID 进行标记，一个供主要线路使用，一个供辅助线路使用。 这些 VLAN ID 位于外部 802.1Q 以太网标头中。 内部 802.1Q 以太网标头（不显示）会映射到特定的 [ExpressRoute 路由域](expressroute-circuit-peerings.md)。

![](./media/expressroute-faqs/expressroute-p2p-ref-arch.png)

### <a name="can-i-extend-one-of-my-vlans-to-azure-using-expressroute"></a>能否使用 ExpressRoute 将我的一个 VLAN 扩展到 Azure？

不。 不支持将第 2 层连接扩展到 Azure。

### <a name="can-i-have-more-than-one-expressroute-circuit-in-my-subscription"></a>能否在我的订阅中有多条 ExpressRoute 线路？

可以。 可以在订阅中有多条 ExpressRoute 线路。 默认限制设置为 10。 如果需要增大限制，请联系 Microsoft 支持部门。

### <a name="can-i-have-expressroute-circuits-from-different-service-providers"></a>能否使用不同服务提供商的 ExpressRoute 线路？

可以。 可以使用许多服务提供商的 ExpressRoute 线路。 每条 ExpressRoute 线路只与一个服务提供商相关联。 

### <a name="i-see-two-expressroute-peering-locations-in-the-same-metro-for-example-singapore-and-singapore2-which-peering-location-should-i-choose-to-create-my-expressroute-circuit"></a>我在同一城市内看到两个 ExpressRoute 对等互连位置（例如，新加坡和新加坡 2）。 我应选择哪个对等位置来创建我的 ExpressRoute 线路？
如果服务提供商在两个站点均提供 ExpressRoute，则可以与提供商协作，选择任意一个站点来设置 ExpressRoute。 

### <a name="can-i-have-multiple-expressroute-circuits-in-the-same-metro-can-i-link-them-to-the-same-virtual-network"></a>在相同市内是否可以有多条 ExpressRoute 线路？ 是否可以将这些线路链接到同一虚拟网络？

可以。 可以具有多条包含相同或不同服务提供商的 ExpressRoute 线路。 如果城区内有多个 ExpressRoute 对等位置，并且线路创建在了不同的对等位置，则可以将这些线路链接到同一虚拟网络。 如果在相同对等位置创建线路，则仍可以到同一虚拟网络，但仅在每个对等互连位置的最多 4 ExpressRoute 线路链接它们。 例如，你可以选择对等互连位置“Singapore”和“Singapore2”，并将来自每个位置的线路连接到同一虚拟网络。 

### <a name="how-do-i-connect-my-virtual-networks-to-an-expressroute-circuit"></a>如何将我的虚拟网络连接到 ExpressRoute 线路

基本步骤如下：

* 建立一条 ExpressRoute 线路并让服务提供商启用它。
* 你或者提供商必须配置 BGP 对等互连。
* 将虚拟网络连接到 ExpressRoute 线路。

有关详细信息，请参阅 [ExpressRoute 线路预配工作流和线路状态](expressroute-workflows.md)。

### <a name="are-there-connectivity-boundaries-for-my-expressroute-circuit"></a>我的 ExpressRoute 线路是否存在连接界限？

可以。 [ExpressRoute 合作伙伴和位置](expressroute-locations.md)一文概述了 ExpressRoute 线路的连接界限。 一条 ExpressRoute 线路的连接范围限制为单个地缘政治区域。 可以通过启用 ExpressRoute 高级功能，将连接扩展为跨地缘政治区域。

### <a name="can-i-link-to-more-than-one-virtual-network-to-an-expressroute-circuit"></a>能否将多个虚拟网络链接到一条 ExpressRoute 线路？

是的。 在标准 ExpressRoute 线路上最多可以有 10 个虚拟网络连接，在[高级 ExpressRoute 线路](#expressroute-premium)上最多可以有 100 个。 

### <a name="i-have-multiple-azure-subscriptions-that-contain-virtual-networks-can-i-connect-virtual-networks-that-are-in-separate-subscriptions-to-a-single-expressroute-circuit"></a>我有多个包含虚拟网络的 Azure 订阅。 能否将不同订阅中的虚拟网络连接到单个 ExpressRoute 线路？

可以。 可以在同一订阅中最多链接 10 个虚拟网络作为线路，或在不同的订阅中使用单一 ExpressRoute 线路。 可以通过启用 ExpressRoute 高级功能来提高此限制。

有关详细信息，请参阅[在多个订阅之间共享 ExpressRoute 线路](expressroute-howto-linkvnet-arm.md)。

### <a name="i-have-multiple-azure-subscriptions-associated-to-different-azure-active-directory-tenants-or-enterprise-agreement-enrollments-can-i-connect-virtual-networks-that-are-in-separate-tenants-and-enrollments-to-a-single-expressroute-circuit-not-in-the-same-tenant-or-enrollment"></a>我有多个关联到不同 Azure Active Directory 租户或企业协议合约的 Azure 订阅。 是否可以将位于单独租户和合约中的虚拟网络连接到不在同一租户或合约中的单个 ExpressRoute 线路？

是的。 ExpressRoute 授权可以跨订阅、租户和合约边界，不需要进行额外配置。 

有关详细信息，请参阅[在多个订阅之间共享 ExpressRoute 线路](expressroute-howto-linkvnet-arm.md)。

### <a name="are-virtual-networks-connected-to-the-same-circuit-isolated-from-each-other"></a>连接到同一线路的虚拟网络相互隔离吗？

不。 从路由角度看，连接到同一 ExpressRoute 线路的所有虚拟网络都属于同一路由域，不是相互隔离的。 如果需要路由隔离，则需要创建单独的 ExpressRoute 线路。

### <a name="can-i-have-one-virtual-network-connected-to-more-than-one-expressroute-circuit"></a>能否将一个虚拟网络连接到多条 ExpressRoute 线路？

可以。 可将一个虚拟网络最多链接到四条 ExpressRoute 线路。 必须通过四个不同的 [ExpressRoute 位置](expressroute-locations.md)订购这些线路。

### <a name="can-i-access-the-internet-from-my-virtual-networks-connected-to-expressroute-circuits"></a>能否从连接到 ExpressRoute 线路的虚拟网络访问 Internet？

可以。 如果尚未通过 BGP 会话公布默认路由 (0.0.0.0/0) 或 Internet 路由前缀，可以从连接到 ExpressRoute 线路的虚拟网络连接到 Internet。

### <a name="can-i-block-internet-connectivity-to-virtual-networks-connected-to-expressroute-circuits"></a>能否阻止与连接到 ExpressRoute 线路的虚拟网络建立 Internet 连接？

可以。 可以播发默认路由 (0.0.0.0/0) 以阻止与虚拟网络内部署的虚拟机建立所有 Internet 连接，并通过 ExpressRoute 线路路由出所有流量。

请注意，如果播发默认路由，我们会强制将传送到通过 Microsoft 对等互连提供的服务（如 Azure 存储和 SQL DB）的流量传回本地。 必须将路由器配置为通过 Microsoft 对等互连路径或通过 Internet 将流量传回 Azure。 如果已启用了该服务的一个服务终结点，则不会强制将发送到该服务的流量传回本地。 流量将保持在 Azure 主干网络中。 若要详细了解服务终结点，请参阅[虚拟网络服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md?toc=%2fazure%2fexpressroute%2ftoc.json)

### <a name="can-virtual-networks-linked-to-the-same-expressroute-circuit-talk-to-each-other"></a>连接到同一 ExpressRoute 线路的虚拟网络能否互相对话？

是的。 连接到同一 ExpressRoute 线路的虚拟网络中部署的虚拟机可以彼此通信。

### <a name="can-i-use-site-to-site-connectivity-for-virtual-networks-in-conjunction-with-expressroute"></a>能否将站点到站点连接与 ExpressRoute 一起用于虚拟网络？

是的。 ExpressRoute 可与站点到站点 VPN 共存。 请参阅[配置 ExpressRoute 和站点到站点并存连接](expressroute-howto-coexist-resource-manager.md)。

### <a name="why-is-there-a-public-ip-address-associated-with-the-expressroute-gateway-on-a-virtual-network"></a>为什么有一个公共 IP 地址与虚拟网络上的 ExpressRoute 网关相关联？

此公共 IP 地址仅用于内部管理，不会给虚拟网络带来安全隐患。

### <a name="are-there-limits-on-the-number-of-routes-i-can-advertise"></a>对于我可以公布的路由数有限制吗？

可以。 对于专用对等互连，我们最多接受 4000 个路由前缀；对于 Microsoft 对等互连，接受 200 个。 如果启用 ExpressRoute 高级功能，可以将专用对等互连的此限制提高为 10,000 个路由。

### <a name="are-there-restrictions-on-ip-ranges-i-can-advertise-over-the-bgp-session"></a>对于我可以通过 BGP 会话公布的 IP 范围有限制吗？

对于 Microsoft 对等 BGP 会话中，不接受私有前缀 (RFC1918)。

### <a name="what-happens-if-i-exceed-the-bgp-limits"></a>如果超过 BGP 限制，会发生什么情况？

BGP 会话会被删除。 当前缀计数低于限制后，将重置这些会话。

### <a name="what-is-the-expressroute-bgp-hold-time-can-it-be-adjusted"></a>ExpressRoute BGP 保持时间是多少？ 是否可以调整它？

保持时间为 180。 Keep-Alive（保持活动）消息每隔 60 秒发送一次。 这些是 Microsoft 端的固定设置，不能更改。 可以配置不同的计时器，BGP 会话参数会相应地协商。

### <a name="can-i-change-the-bandwidth-of-an-expressroute-circuit"></a>是否可以更改 ExpressRoute 线路的带宽？

是的，可以尝试在 Azure 门户中或者使用 PowerShell.mpt 来增加 ExpressRoute 线路的带宽。 如果在创建线路的物理端口上有容量可用，则更改会成功。 

如果更改失败，这意味着当前端口上没有剩余足够的容量，需要创建具有更高带宽的新 ExpressRoute 线路；或者意味着在该位置没有额外的容量，在这种情况下将无法增加带宽。 

还必须跟进连接服务提供商，确保他们更新其网络中的限制以支持带宽增加。 不过，无法减小 ExpressRoute 线路的带宽。 必须创建具有更低带宽的新 ExpressRoute 线路并删除旧线路。

### <a name="how-do-i-change-the-bandwidth-of-an-expressroute-circuit"></a>如何更改 ExpressRoute 线路的带宽？

可以使用 REST API 或 PowerShell cmdlet 更新 ExpressRoute 线路的带宽。

## <a name="expressroute-premium"></a>ExpressRoute 高级版

### <a name="what-is-expressroute-premium"></a>什么是 ExpressRoute 高级版？

ExpressRoute 高级版是以下功能的集合：

* 对于专用对等互连，将路由表限制从 4000 个路由提升为 10,000 个路由。
* 增加了可在 ExpressRoute 线路上启用的 VNet 和 ExpressRoute Global Reach 连接的数量（默认为 10）。 有关详细信息，请参阅 [ExpressRoute 限制](#limits)表。
* 连接到 Office 365 和 Dynamics 365。
* 通过 Microsoft 核心网络建立全局连接。 现在，可将一个地缘政治区域中 VNet 链接到另一个区域中的 ExpressRoute 线路。<br>
    **示例：**

    *  可将欧洲西部创建的 VNet 链接到硅谷创建的 ExpressRoute 线路。 
    *  在 Microsoft 对等互连中，会播发来自其他地缘政治区域的前缀，以便可以进行连接，例如，通过硅谷的线路连接到欧洲西部地区的 SQL Azure。


### <a name="limits"></a>如果我启用了 ExpressRoute 高级版，可以在 ExpressRoute 线路上启用多少 Vnet 和 ExpressRoute Global Reach 连接？

下表显示了 ExpressRoute 限制和每条 ExpressRoute 线路的 VNet 和 ExpressRoute Global Reach 连接数：

[!INCLUDE [ExpressRoute limits](../../includes/expressroute-limits.md)]

### <a name="how-do-i-enable-expressroute-premium"></a>如何启用 ExpressRoute 高级版？

在启用相应的功能后，可启用 ExpressRoute 高级功能，并可通过更新线路状态关闭高级功能。 可以在创建线路时启用 ExpressRoute 高级版，或者调用 REST API/PowerShell cmdlet。

### <a name="how-do-i-disable-expressroute-premium"></a>如何禁用 ExpressRoute 高级版？

可以通过调用 REST API 或 PowerShell cmdlet 来禁用 ExpressRoute 高级版。 必须确保已调整连接需求以满足默认限制，再禁用 ExpressRoute 高级版。 如果使用规模超出了默认限制，禁用 ExpressRoute 高级版的请求会失败。

### <a name="can-i-pick-and-choose-the-features-i-want-from-the-premium-feature-set"></a>我是否可以从高级功能集选择所需的功能？

不。 无法选择功能。 如果启用 ExpressRoute 高级版，我们会启用所有功能。

### <a name="how-much-does-expressroute-premium-cost"></a>ExpressRoute 高级版的费用是多少？

有关费用，请参阅[定价详细信息](https://azure.microsoft.com/pricing/details/expressroute/)。

### <a name="do-i-pay-for-expressroute-premium-in-addition-to-standard-expressroute-charges"></a>除了支付 ExpressRoute 高级版费用以外，是否还要支付标准版 ExpressRoute 的费用？

是的。 ExpressRoute 高级版的费用是在 ExpressRoute 线路费用以及连接提供商所收费用的基础之上收取的。

## <a name="expressroute-for-office-365"></a>适用于 Office 365 的 ExpressRoute

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

### <a name="how-do-i-create-an-expressroute-circuit-to-connect-to-office-365-services"></a>如何创建 ExpressRoute 线路，以便连接到 Office 365 服务？

1. 请查看 [ExpressRoute 先决条件页](expressroute-prerequisites.md)，以确保满足要求。
2. 为确保满足连接需求，请查看 [ExpressRoute 合作伙伴和位置](expressroute-locations.md)一文中的服务提供商和位置列表。
3. 请查看[针对 Office 365 的网络规划和性能优化](https://aka.ms/tune/)，规划容量要求。
4. 遵循工作流中列出的步骤来设置连接：[ExpressRoute 线路预配工作流和线路状态](expressroute-workflows.md)。

> [!IMPORTANT]
> 确保在配置与 Office 365 服务的连接时已启用 ExpressRoute 高级版外接程序。
> 
> 

### <a name="can-my-existing-expressroute-circuits-support-connectivity-to-office-365-services-and-dynamics-365"></a>现有 ExpressRoute 线路是否支持连接到 Office 365 服务和 Dynamics 365？

可以。 可以将现有 ExpressRoute 线路配置为支持连接到 Office 365 服务。 确保容量足以连接到 Office 365 服务，并确保已启用高级版外接程序。 [针对 Office 365 的网络规划和性能优化](https://aka.ms/tune/)可帮助规划连接需求。 另外，请参阅[创建和修改 ExpressRoute 线路](expressroute-howto-circuit-classic.md)。

### <a name="what-office-365-services-can-be-accessed-over-an-expressroute-connection"></a>通过 ExpressRoute 连接可以访问哪些 Office 365 服务？

如需可以通过 ExpressRoute 使用的服务的最新列表，请参阅 [Office 365 URL 和 IP 地址范围](https://aka.ms/o365endpoints)页。

### <a name="how-much-does-expressroute-for-office-365-services-cost"></a>用于 Office 365 服务的 ExpressRoute 的费用是多少？

Office 365 服务要求启用高级版外接程序。 有关费用，请参阅[定价详细信息页](https://azure.microsoft.com/pricing/details/expressroute/)。

### <a name="what-regions-is-expressroute-for-office-365-supported-in"></a>哪些区域支持适用于 Office 365 的 ExpressRoute？

有关信息，请参阅 [ExpressRoute 合作伙伴和位置](expressroute-locations.md)。

### <a name="can-i-access-office-365-over-the-internet-even-if-expressroute-was-configured-for-my-organization"></a>是否即使为组织配置了 ExpressRoute，也可以通过 Internet 访问 Office 365？

可以。 即使为网络配置了 ExpressRoute，也可以通过 Internet 访问 Office 365 服务终结点。 如果你所在位置的网络配置为通过 ExpressRoute 连接到 Office 365 服务，请咨询组织的网络团队。

### <a name="how-can-i-plan-for-high-availability-for-office-365-network-traffic-on-azure-expressroute"></a>如何在 Azure ExpressRoute 上规划 Office 365 网络流量的高可用性？
请参阅有关[使用 Azure ExpressRoute 实现高可用性和故障转移](https://aka.ms/erhighavailability)的建议

### <a name="can-i-access-office-365-us-government-community-gcc-services-over-an-azure-us-government-expressroute-circuit"></a>是否可以通过 Azure 美国政府版 ExpressRoute 线路访问 Office 365 美国政府社区 (GCC) 服务？

可以。 可以通过 Azure 美国政府版 ExpressRoute 访问 Office 365 GCC 服务终结点。 但是，首先需要在 Azure 门户上开具支持票证，向 Microsoft 提供要播发的前缀。 解决支持票证后，将建立到 Office 365 GCC 服务的连接。 

## <a name="route-filters-for-microsoft-peering"></a>用于 Microsoft 对等互连的路由筛选器

### <a name="i-am-turning-on-microsoft-peering-for-the-first-time-what-routes-will-i-see"></a>首次启用 Microsoft 对等互连时，会看到哪些路由？

不会看到任何路由。 必须将路由筛选器附加到线路才能启动前缀播发。 有关说明，请参阅[配置用于 Microsoft 对等互连的路由筛选器](how-to-routefilter-powershell.md)。

### <a name="i-turned-on-microsoft-peering-and-now-i-am-trying-to-select-exchange-online-but-it-is-giving-me-an-error-that-i-am-not-authorized-to-do-it"></a>我已启用 Microsoft 对等互连，目前正在尝试选择 Exchange Online，但有错误指出我无权执行此操作。

使用路由筛选器时，任何客户都可以启用 Microsoft 对等互连。 但是，若要使用 Office 365 服务，仍需获得 Office 365 的授权。

### <a name="do-i-need-to-get-authorization-for-turning-on-dynamics-365-over-microsoft-peering"></a>是否需要获得授权才能通过 Microsoft 对等互连启用 Dynamics 365？

否，不需要获得 Dynamics 365 的授权。 可以创建一个规则并选择 Dynamics 365 社区版，而无需获得授权。

### <a name="i-enabled-microsoft-peering-prior-to-august-1-2017-how-can-i-take-advantage-of-route-filters"></a>我在 2017 年 8 月 1 日前已启用了 Microsoft 对等互连，如何利用路由筛选器？

现有线路将继续为 Office 365 和 Dynamics 365 播发前缀。 如果希望添加基于同一 Microsoft 对等互连的 Azure 公共前缀播发，可以创建一个路由筛选器，选择需要播发的服务（包括你需要的 Office 365 服务和 Dynamics 365），并将筛选器附加到你的 Microsoft 对等互连。 有关说明，请参阅[配置用于 Microsoft 对等互连的路由筛选器](how-to-routefilter-powershell.md)。

### <a name="i-have-microsoft-peering-at-one-location-now-i-am-trying-to-enable-it-at-another-location-and-i-am-not-seeing-any-prefixes"></a>我在一个位置建立了 Microsoft 对等互连，目前我正尝试在另一个位置启用它，但未看到任何前缀。

* 在 2017 年 8 月 1 日之前配置的 ExpressRoute 线路的 Microsoft 对等互连会通过 Microsoft 对等互连播发所有服务前缀，即使未定义路由筛选器。

* 在 2017 年 8 月 1 日或之后配置的 ExpressRoute 线路的 Microsoft 对等互连的任何前缀只在将路由筛选器附加到线路之后才会播发。 默认情况下，不会显示任何前缀。

## <a name="expressRouteDirect"></a>ExpressRoute Direct

[!INCLUDE [ExpressRoute Direct](../../includes/expressroute-direct-faq-include.md)]

## <a name="globalreach"></a>全球覆盖

[!INCLUDE [Global Reach](../../includes/expressroute-global-reach-faq-include.md)]
