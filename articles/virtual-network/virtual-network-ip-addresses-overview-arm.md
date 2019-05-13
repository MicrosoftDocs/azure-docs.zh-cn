---
title: Azure 中的 IP 地址类型
titlesuffix: Azure Virtual Network
description: 了解如何在 Azure 中使用公共和专用 IP 地址
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/05/2019
ms.author: kumud
ms.openlocfilehash: 73b185eabc77d293328b1251a4af1aafffc5f319
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236356"
---
# <a name="ip-address-types-and-allocation-methods-in-azure"></a>Azure 中的 IP 地址类型和分配方法

可以将 IP 地址分配到与其他 Azure 资源通信的 Azure 资源，也可以将其分配到本地网络和 Internet。 可以在 Azure 中使用两种类型的 IP 地址：

* **公共 IP 地址**：用来与 Internet 通信，包括与面向公众的 Azure 服务通信。
* **专用 IP 地址**：使用 VPN 网关或 ExpressRoute 线路将网络扩展到 Azure 时，用于在 Azure 虚拟网络 (VNet) 和本地网络中通信。

此外，通过公共 IP 前缀创建连续的静态公共 IP 地址范围。 [了解公共 IP 前缀。](public-ip-address-prefix.md)

> [!NOTE]
> Azure 具有用于创建和处理资源的两个不同部署模型：[资源管理器部署模型和经典部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。  本文介绍如何使用 Resource Manager 部署模型。Microsoft 建议对大多数新的部署使用该模型，而不是[经典部署模型](virtual-network-ip-addresses-overview-classic.md)。
> 

如果熟悉经典部署模型，请参阅[经典部署与 Resource Manager 之间的 IP 寻址差异](virtual-network-ip-addresses-overview-classic.md#differences-between-resource-manager-and-classic-deployments)。

## <a name="public-ip-addresses"></a>公共 IP 地址

公共 IP 地址允许 Internet 资源与 Azure 资源进行入站通信。 在 IP 地址已分配给 Azure 资源的情况下，公共 IP 地址还允许这些资源与 Internet 和面向公众的 Azure 服务进行出站通信。 此地址专门用于该资源，直到你对其取消分配。 如果公共 IP 地址未分配给资源，该资源仍可与 Internet 进行出站通信，但 Azure 会动态分配不专用于该资源的可用 IP 地址。 有关 Azure 中的出站连接的详细信息，请参阅[了解出站连接](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

在 Azure 资源管理器中，[公共 IP](virtual-network-public-ip-address.md) 地址是具有其自身属性的资源。 可与公共 IP 地址资源关联的部分资源包括：

* 虚拟机网络接口
* 面向 Internet 的负载均衡器
* VPN 网关
* 应用程序网关数

### <a name="ip-address-version"></a>IP 地址版本

公共 IP 地址是使用 IPv4 或 IPv6 地址创建的。 公共 IPv6 地址只能分配到面向 Internet 的负载均衡器。

### <a name="sku"></a>SKU

使用以下 SKU 之一创建公共 IP 地址：

>[!IMPORTANT]
> 必须为负载均衡器和公用 IP 资源使用匹配的 SKU。 不能混合使用基本 SKU 资源和标准 SKU 资源。 无法将独立的虚拟机、可用性集资源中的虚拟机或虚拟机规模集资源同时附加到两个 SKU。  新的设计应当考虑使用标准 SKU 资源。  有关详细信息，请查看[标准负载均衡器](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

#### <a name="basic"></a>基本

推出 SKU 之前创建的所有公共 IP 地址为基本 SKU 公共 IP 地址。 随着 SKU 的推出，可以选择指定公共 IP 地址要采用的 SKU。 基本 SKU 地址为：

- 使用静态或动态分配方法分配。
- 具有可调整的入站发起流空闲超时，范围为 4-30 分钟，默认值为 4 分钟，出站发起流的空闲超时固定为 4 分钟。
- 默认情况下处于打开状态。  建议使用网络安全组来对入站或出站流量进行限制，但这是可选的。
- 分配到可以采用公共 IP 地址的任何 Azure 资源，例如网络接口、VPN 网关、应用程序网关和面向 Internet 的负载均衡器。
- 不支持可用性区域方案。  需要为可用性区域方案使用标准 SKU 公共 IP。 若要详细了解可用性区域，请参阅[可用性区域概述](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)与[标准负载均衡器和可用性区域](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

#### <a name="standard"></a>标准

标准 SKU 公共 IP 地址为：

- 始终使用静态分配方法。
- 具有可调整的入站发起流空闲超时，范围为 4-30 分钟，默认值为 4 分钟，出站发起流的空闲超时固定为 4 分钟。
- 默认情况下为安全的，并且对入站流量关闭。 必须使用[网络安全组](security-overview.md#network-security-groups)将允许的入站流量显式列入允许列表中。
- 分配到网络接口、标准公共负载均衡器、应用程序网关或 VPN 网关。 有关标准负载均衡器的详细信息，请参阅 [Azure 标准负载均衡器](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
- 默认情况下是区域冗余的，也可以选择为区域性的（可以创建为区域性的，并且在特定的可用性区域中保证可靠性）。 若要详细了解可用性区域，请参阅[可用性区域概述](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)与[标准负载均衡器和可用性区域](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
 
> [!NOTE]
> 在创建并关联[网络安全组](security-overview.md#network-security-groups)且显式允许所需入站流量之前，到标准 SKU 资源的入站通信将会失败。

> [!NOTE]
> 仅使用时，使用基本 SKU 公共 IP 地址都可用[实例元数据服务 IMDS](../virtual-machines/windows/instance-metadata-service.md)。 不支持标准 SKU。

### <a name="allocation-method"></a>分配方法

基本和标准 SKU 公用 IP 地址都支持“静态”分配方法。  在创建资源时会为其分配 IP 地址，在删除资源时将释放 IP 地址。

基本 SKU 公用 IP 地址还支持“动态”分配方法，这是未指定分配方法时将采用的默认方法。  为基本的公用 IP 地址资源选择“动态”分配方法意味着在创建资源时“不”分配 IP 地址。  将公用 IP 地址与虚拟机进行关联时或者将第一个虚拟机实例放置到基本负载均衡器的后端池中时，将分配公用 IP 地址。   停止（或删除）该资源时，就会释放该 IP 地址。  例如，从资源 A 中释放后，可将该 IP 地址分配到不同的资源。 如果在停止资源 A 的情况下将 IP 地址分配到不同的资源，则重启资源 A 时，会分配一个不同的 IP 地址。 如果将基本的公用 IP 地址资源的分配方法从“静态”更改为“动态”，则会释放地址。 要确保所关联资源的 IP 地址保持不变，可将分配方法显式设置为*静态*。 静态 IP 地址是立即分配的。

> [!NOTE]
> 即使将分配方法设置为“静态”，也无法通过指定方式将实际 IP 地址分配到公共 IP 地址资源。 Azure 会从创建资源时所在的 Azure 位置的可用 IP 地址池中分配 IP 地址。
>

以下情况通常使用静态公共 IP 地址：

* 必须更新防火墙规则才能与 Azure 资源通信。
* 对 DNS 名称进行解析时，如果更改了 IP 地址，则需更新 A 记录。
* Azure 资源与其他使用 IP 地址型安全模型的应用或服务通信。
* 使用链接到 IP 地址的 SSL 证书。

> [!NOTE]
> Azure 会从每个 Azure 云中每个区域的唯一地址范围中分配公共 IP 地址。 对于 Azure [公有](https://www.microsoft.com/download/details.aspx?id=56519)云、[美国政府](https://www.microsoft.com/download/details.aspx?id=57063)云、[中国](https://www.microsoft.com/download/details.aspx?id=57062)云和[德国](https://www.microsoft.com/download/details.aspx?id=57064)云，可以下载范围（前缀）的列表。
>

### <a name="dns-hostname-resolution"></a>DNS 主机名解析
可以为公共 IP 资源指定一个 DNS 域名标签，以便在 Azure 托管的 DNS 服务器中为 *domainnamelabel*.*location*.cloudapp.azure.com 创建目标为公共 IP 地址的映射。 例如，如果在创建公共 IP 资源时将 *domainnamelabel* 指定为 **contoso**，将 Azure 的“位置”指定为“美国西部”，则会将完全限定域名 (FQDN) **contoso.westus.cloudapp.azure.com** 解析成该资源的公共 IP 地址。

> [!IMPORTANT]
> 所创建的每个域名标签在其 Azure 位置中必须是唯一的。  
>

### <a name="dns-best-practices"></a>DNS 最佳实践
如果需要迁移到不同的区域，则不能迁移的公共 IP 地址的 FQDN。 最佳做法是，可以使用 FQDN 创建指向 Azure 中的公共 IP 地址的自定义域 CNAME 记录。 如果你需要将移动到不同的公共 IP，它将需要的更新而无需手动更新到新的地址的 FQDN 的 CNAME 记录。 可以使用[Azure DNS](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address)或你的 DNS 记录的外部 DNS 提供程序。 

### <a name="virtual-machines"></a>虚拟机

将公共 IP 地址分配到其**网络接口**可将其与 [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虚拟机相关联。 可以向虚拟机分配动态或静态公共 IP 地址。 详细了解如何[将 IP 地址分配到网络接口](virtual-network-network-interface-addresses.md)。

### <a name="internet-facing-load-balancers"></a>面向 Internet 的负载均衡器

可将通过任一 [SKU](#sku) 创建的公共 IP 地址与 [Azure 负载均衡器](../load-balancer/load-balancer-overview.md)相关联，只需将其分配给负载均衡器**前端**配置即可。 此公共 IP 地址充当负载均衡型虚拟 IP 地址 (VIP)。 可以向负载均衡器前端分配动态或静态公共 IP 地址。 还可以向负载均衡器前端分配多个公共 IP 地址，这会启用[多 VIP](../load-balancer/load-balancer-multivip-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 方案，如包含基于 SSL 的网站的多租户环境。 有关 Azure 负载均衡器 SKU 的详细信息，请参阅 [Azure 负载均衡器标准 SKU](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="vpn-gateways"></a>VPN 网关

[Azure VPN 网关](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json)将 Azure 虚拟网络连接到其他 Azure 虚拟网络或本地网络。 需将公共 IP 地址分配到 VPN 网关才能与远程网络通信。 只能向 VPN 网关分配”动态”基本的公用 IP 地址。

### <a name="application-gateways"></a>应用程序网关数

将公共 IP 地址分配给网关的**前端**配置可以将其与 Azure [应用程序网关](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)相关联。 此公共 IP 地址充当负载均衡型 VIP。 您只能将分配*动态*给应用程序网关 V1 前端配置时，基本公共 IP 地址和仅*静态*到 V2 前端配置的标准 SKU 地址。

### <a name="at-a-glance"></a>概览
下表显示了将公共 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |Linux |“是” |“是” |
| 面向 Internet 的负载均衡器 |前端配置 |“是” |“是” |
| VPN 网关 |网关 IP 配置 |“是” |“否” |
| 应用程序网关 |前端配置 |是（仅限 V1） |是（仅限 V2） |

## <a name="private-ip-addresses"></a>专用 IP 地址
专用 IP 地址能够让 Azure 资源在不使用可访问 Internet 的 IP 地址的情况下，与[虚拟网络](virtual-networks-overview.md)或本地网络中的其他资源（通过 VPN 网关或 ExpressRoute 线路）通信。

在 Azure 资源管理器部署模型中，可将专用 IP 地址关联到以下类型的 Azure 资源：

* 虚拟机网络接口
* 内部负载均衡器 (ILB)
* 应用程序网关数

### <a name="ip-address-version"></a>IP 地址版本

专用 IP 地址是使用 IPv4 或 IPv6 地址创建的。 只能使用动态分配方法分配专用 IPv6 地址。 无法在虚拟网络上的专用 IPv6 地址之间通信。 可以通过面向 Internet 的负载均衡器，从 Internet 与专用 IPv6 地址进行入站通信。 有关详细信息，请参阅[使用 IPv6 创建面向 Internet 的负载均衡器](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="allocation-method"></a>分配方法

可以根据资源所部署到的虚拟网络子网的地址范围来分配专用 IP 地址。 Azure 保留每个子网地址范围中的前四个地址，因此无法将这些地址分配给资源。 例如，如果子网的地址范围是 10.0.0.0/16，则地址 10.0.0.0-10.0.0.3 无法分配给资源。 子网的地址范围内的 IP 地址一次只能分配给一个资源。 

分配专用 IP 地址有两种方法：

- **动态**：Azure 会分配子网的地址范围内下一个未分配或未保留的可用 IP 地址。 例如，如果地址 10.0.0.4-10.0.0.9 已分配给其他资源，Azure 会将 10.0.0.10 分配给新资源。 动态方法是默认的分配方法。 动态 IP 地址在分配后，仅在以下情况下才会释放：网络接口已删除、已分配到同一虚拟网络中的另一子网，或者分配方法已更改为静态，这种情况下会指定另一 IP 地址。 默认情况下，当分配方法从动态更改为静态时，Azure 会将以前动态分配的地址作为静态地址分配。
- **静态**：选择并分配子网的地址范围内任何未分配或未保留的 IP 地址。 例如，如果子网的地址范围是 10.0.0.0/16，并且地址 10.0.0.4-10.0.0.9 已分配给其他资源，则可以指定 10.0.0.10 - 10.0.255.254 之间的任何地址。 只有在删除网络接口之后，静态地址才会释放。 如果将分配方法更改为动态，Azure 会动态地将以前分配的静态 IP 地址作为动态地址分配，即使该地址不是子网地址范围内的下一个可用地址。 如果将网络接口分配给同一虚拟网络中的另一子网，则该地址也会更改。但是，若要将网络接口分配给另一子网，必须先将分配方法从静态更改为动态。 将网络接口分配给另一子网以后，即可将分配方法改回为静态，并根据新子网的地址范围分配 IP 地址。

### <a name="virtual-machines"></a>虚拟机

可将一个或多个专用 IP 地址分配给 [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虚拟机的一个或多个**网络接口**。 可将每个专用 IP 地址的分配方法指定为动态或静态。

#### <a name="internal-dns-hostname-resolution-for-virtual-machines"></a>内部 DNS 主机名解析（针对虚拟机）

所有 Azure 虚拟机都默认配置了 [Azure 托管的 DNS 服务器](virtual-networks-name-resolution-for-vms-and-role-instances.md#azure-provided-name-resolution)，除非显式配置了自定义 DNS 服务器。 这些 DNS 服务器为驻留在同一个虚拟网络内的虚拟机提供内部名称解析。

创建虚拟机时，主机名到其专用 IP 地址的映射将添加到 Azure 托管的 DNS 服务器。 如果虚拟机有多个网络接口，或者一个网络接口有多个 IP 配置，主机名会映射到主要网络接口的主要 IP 配置的专用 IP 地址。

使用 Azure 托管的 DNS 服务器配置的虚拟机可将同一虚拟网络中的所有虚拟机的主机名解析为其专用 IP 地址。 若要在连接的虚拟网络中解析虚拟机的主机名，必须使用自定义 DNS 服务器。

### <a name="internal-load-balancers-ilb--application-gateways"></a>内部负载均衡器 (ILB) 和应用程序网关

可以将专用 IP 地址分配到 [Azure 内部负载均衡器](../load-balancer/load-balancer-internal-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (ILB) 或 [Azure 应用程序网关](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的**前端**配置。 此专用 IP 地址将用作内部终结点，仅供其虚拟网络和连接到该虚拟网络的远程网络中的资源访问。 可将动态或静态专用 IP 地址分配到前端配置。

### <a name="at-a-glance"></a>概览
下表显示了将专用 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |Linux |“是” |“是” |
| 负载均衡器 |前端配置 |“是” |“是” |
| 应用程序网关 |前端配置 |“是” |“是” |

## <a name="limits"></a>限制
Azure 中的[网络限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)全面阐述了对 IP 寻址施加的限制。 这些限制根据区域和订阅设置。 可以[与支持人员联系](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)，根据业务需求将默认限制提高到最大限制。

## <a name="pricing"></a>定价
公共 IP 地址可能会产生少许费用。 有关 Azure 中 IP 地址定价的详细信息，请阅读 [IP 地址定价](https://azure.microsoft.com/pricing/details/ip-addresses)页。

## <a name="next-steps"></a>后续步骤
* [使用 Azure 门户通过静态公共 IP 部署 VM](virtual-network-deploy-static-pip-arm-portal.md)
* [通过 Azure 门户使用静态专用 IP 地址部署 VM](virtual-networks-static-private-ip-arm-pportal.md)
