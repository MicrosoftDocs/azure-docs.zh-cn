---
title: 使用 Azure Active Directory 应用程序代理时的网络拓扑注意事项 | Microsoft 文档
description: 介绍使用 Azure AD 应用程序代理时的网络拓扑注意事项。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/28/2017
ms.author: celested
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f5b87e452b0c79ae9cdc1d7f9f391a611dceda2f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231518"
---
# <a name="network-topology-considerations-when-using-azure-active-directory-application-proxy"></a>使用 Azure Active Directory 应用程序代理时的网络拓扑注意事项

本文介绍使用 Azure Active Directory (Azure AD) 应用程序代理远程发布和访问应用程序时的网络拓扑注意事项。

## <a name="traffic-flow"></a>流量流

通过 Azure AD 应用程序代理发布应用程序时，从用户发往应用程序的流量将通过三个连接流动：

1. 用户连接到 Azure 上的 Azure AD 应用程序代理服务公共终结点
2. 应用程序代理服务连接到应用程序代理连接器
3. 应用程序代理连接器连接到目标应用程序

![显示从用户发往目标应用程序的流量流的示意图](./media/application-proxy-network-topology/application-proxy-three-hops.png)

## <a name="tenant-location-and-application-proxy-service"></a>租户位置和应用程序代理服务

当你注册 Azure AD 租户时，租户的区域由你指定的国家/地区/区域决定。 启用应用程序代理时，将在 Azure AD 租户所在的同一区域或者与它最靠近的区域中选择或创建租户的应用程序代理服务实例。

例如，如果你的 Azure AD 租户所在的区域为英国，则所有 Azure AD 应用程序代理连接器都使用位于 EU 数据中心内的服务实例。 当用户访问发布的应用程序时，其流量会流经此位置中的应用程序代理服务实例。

## <a name="considerations-for-reducing-latency"></a>有关降低延迟的注意事项

所有代理解决方案都会给网络连接造成延迟。 无论选择哪种代理或 VPN 解决方案作为远程访问解决方案，它始终会包含一组服务器来实现与企业网络内部的连接。

组织通常已在其外围网络中添置了服务器终结点。 但是，使用 Azure AD 应用程序代理时，流量将流经云中的代理服务，而连接器驻留在企业网络中。 不需要外围网络。

以下各部分包含可帮助进一步降低延迟的其他建议。 

### <a name="connector-placement"></a>连接器的位置

应用程序代理根据租户的位置选择实例的位置。 但是，需确定要将连接器安装在哪个位置，以便能够定义网络流量的延迟特征。

设置应用程序代理服务时，请提出以下问题：

* 应用位于何处？
* 访问应用的大部分用户位于何处？
* 应用程序代理实例位于何处？
* 是否已经与 Azure 数据中心建立专用网络连接，例如，使用 Azure ExpressRoute 或类似的 VPN？

连接器必须与 Azure 和应用程序通信（流量流程图中的步骤 2 和步骤 3），因此，连接器的位置会影响这两个连接的延迟时间。 评估连接器的位置时，请考虑以下几点：

* 如果要使用 Kerberos 约束委派 (KCD) 进行单一登录，连接器需要与数据中心保持直通。 此外，连接器服务器需加入域。  
* 如果不很确定，请将连接器安装在比较靠近应用程序的位置。

### <a name="general-approach-to-minimize-latency"></a>最小化延迟的常规方法

可通过优化每个网络连接来尽量降低端到端流量的延迟。 可通过以下方式优化每个连接：

* 减少跃点两个端点之间的距离。
* 选择要遍历的适当网络。 例如，遍历专用网络而不遍历公共 Internet 可能速度更快，因为前者是专用链路。

如果在 Azure 与企业网络之间建立了专用 VPN 或 ExpressRoute 链路，可以利用它。

## <a name="focus-your-optimization-strategy"></a>注重优化策略

很难控制用户与应用程序代理之间的连接。 用户可以从家庭网络、 咖啡店或其他国家/地区访问你的应用。 不过，可以优化从应用程序代理服务到应用程序代理连接器再到应用的连接。 考虑在环境中整合以下模式。

### <a name="pattern-1-put-the-connector-close-to-the-application"></a>模式 1：将连接器放在靠近应用程序的位置

将连接器放在客户网络中靠近目标应用程序的位置。 由于连接器靠近应用程序，所以此配置可最小化拓扑图中的步骤 3。 

如果连接器需连接到域控制器，此模式十分有利。 这种方法适用于大多数方案，因此大多数客户都使用此模式。 也可以将此模式与模式 2 结合使用，优化服务与连接器之间的流量。

### <a name="pattern-2-take-advantage-of-expressroute-with-microsoft-peering"></a>模式 2：将 ExpressRoute 与 Microsoft 对等互连配合使用

如果你为 ExpressRoute 设置了 Microsoft 对等互连，则可为应用程序代理与连接器之间的流量使用更快的 ExpressRoute 连接。 连接器仍在网络中靠近应用的位置。

### <a name="pattern-3-take-advantage-of-expressroute-with-private-peering"></a>模式 3：将 ExpressRoute 与专用对等互连配合使用

如果已在 Azure 与企业网络之间结合专用对等互连设置了专用 VPN 或 ExpressRoute，则还可以使用另一个选项。 在此配置中，Azure 中的虚拟网络通常被视为企业网络的扩展。 因此，可将连接器安装在 Azure 数据中心，同时仍可满足连接器到应用的连接的低延迟要求。

因为流量通过专用连接流动，因此延迟不会加重。 由于连接器安装在 Azure 数据中心内靠近 Azure AD 租户的位置，因此可以改善应用程序代理服务与连接器之间的网络延迟。

![显示 Azure 数据中心内安装的连接器的示意图](./media/application-proxy-network-topology/application-proxy-expressroute-private.png)

### <a name="other-approaches"></a>其他方法

尽管本文的重点是连接器的位置，但是，也可以通过改变应用程序的位置来获得更好的延迟特征。

越来越多的组织正在将其网络移入托管环境。 这样，便可以将应用放在同样属于企业网络一部分的托管环境中，而这些应用仍会保留在域中。 在这种情况下，可向新的应用程序位置应用前面部分中所述的模式。 考虑此选项时，请参阅 [Azure AD 域服务](../../active-directory-domain-services/active-directory-ds-overview.md)。

此外，考虑使用[连接器组](application-proxy-connector-groups.md)针对不同位置和网络中的应用进行连接器排列组织。 

## <a name="common-use-cases"></a>常见用例

本部分将演练几个常见方案。 假设 Azure AD 租户（因此也包括代理服务终结点）位于美国 (US)。 在全球的其他区域，这些用例中所述的注意事项也适用。

对于这些方案，为了方便讨论，我们将每个连接称为“跃点”并为其编号：

- **跃点 1**：从用户到应用程序代理服务
- **跃点 2**：从应用程序代理服务到应用程序代理连接器
- **跃点 3**：从应用程序代理连接器到目标应用程序 

### <a name="use-case-1"></a>用例 1

**场景：** 该应用位于组织在美国部署的网络中，其用户位于同一区域。 Azure 数据中心与企业网络之间不存在 ExpressRoute 或 VPN。

**建议：** 遵循上一部分中所述的模式 1。 为了降低延迟，可以根据需要考虑使用 ExpressRoute。

这是一个简单的模式。 可以通过将连接器放置在应用附近来优化跃点 3。 这也是一种自然而然的选择，因为安装的连接器通常与应用和数据中心建立直通连接，以便执行 KCD 操作。

![显示用户、代理、连接器和应用都位于美国的示意图](./media/application-proxy-network-topology/application-proxy-pattern1.png)

### <a name="use-case-2"></a>用例 2

**场景：** 该应用位于组织在美国部署的网络中，其用户遍布全球各地。 Azure 数据中心与企业网络之间不存在 ExpressRoute 或 VPN。

**建议：** 遵循上一部分中所述的模式 1。 

同样，常见的模式是优化跃点 3，其中的连接器放置在应用附近。 如果将整个跃点 3 放在同一区域，则它的系统开销通常不大。 但是，根据用户所在的位置，跃点 1 的系统开销可能更大，因为世界各地的用户必须访问位于美国的应用程序代理实例。 值得注意的是，由于用户遍布全球各地，任何代理解决方案将具有类似的特征。

![显示用户遍布全球，但代理、连接器和应用位于美国的示意图](./media/application-proxy-network-topology/application-proxy-pattern2.png)

### <a name="use-case-3"></a>用例 3

**场景：** 该应用位于组织在美国部署的网络中。 Azure 与企业网络之间存在采用 Microsoft 对等互连的 ExpressRoute。

**建议：** 遵循上一部分中所述的模式 1 和模式 2。

首先，使连接器尽量靠近应用。 然后，系统自动为跃点 2 使用 ExpressRoute。 

如果 ExpressRoute 链路使用 Microsoft 对等互连，则代理与连接器之间的流量将通过该链路流动。 跃点 2 的延迟将得到优化。

![显示代理与连接器之间的 ExpressRoute 连接的示意图](./media/application-proxy-network-topology/application-proxy-pattern3.png)

### <a name="use-case-4"></a>用例 4

**场景：** 该应用位于组织在美国部署的网络中。 Azure 与企业网络之间存在结合专用对等互连的 ExpressRoute。

**建议：** 遵循上一部分中所述的模式 3。

将连接器放置在通过 ExpressRoute 专用对等互连连接到企业网络的 Azure 数据中心内。 

可将连接器放置在 Azure 数据中心内。 由于连接器仍旧通过专用网络与应用程序和数据中心建立直通连接，跃点 3 将保持优化。 此外，跃点 2 会进一步优化。

![显示 Azure 数据中心内的连接器以及连接器与应用之间的  ExpressRoute 连接的示意图](./media/application-proxy-network-topology/application-proxy-pattern4.png)

### <a name="use-case-5"></a>用例 5

**场景：** 应用位于组织在欧盟部署的网络中，应用程序代理实例和大多数用户位于美国。

**建议：** 将连接器放置在应用的附近。 由于美国用户访问的应用程序代理实例正好处于同一区域，因此跃点 1 的系统开销也不会太高。 跃点 3 经过优化。 考虑使用 ExpressRoute 来优化跃点 2。 

![显示用户和代理位于美国，连接器和应用位于欧盟的示意图](./media/application-proxy-network-topology/application-proxy-pattern5b.png)

还可以考虑对此用例使用另一种变化形式。 如果组织中的大多数用户位于美国，则组织有可能也会将网络扩展到美国。 将连接器放置在美国，使用专用的内部企业网络线路访问位于欧盟的应用程序。 因此，跃点 2 和跃点 3 将得到优化。

![显示用户、代理和连接器位于美国，应用位于欧盟的示意图](./media/application-proxy-network-topology/application-proxy-pattern5c.png)

## <a name="next-steps"></a>后续步骤

- [启用应用程序代理](application-proxy-add-on-premises-application.md)
- [启用单一登录](application-proxy-configure-single-sign-on-with-kcd.md)
- [启用条件性访问](application-proxy-integrate-with-sharepoint-server.md)
- [解决使用应用程序代理时遇到的问题](application-proxy-troubleshoot.md)
