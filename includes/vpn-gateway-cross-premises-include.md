---
title: include 文件
description: include 文件
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/16/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 14793d7f787780bfc8604e4af11eb05f1ff0d937
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60859724"
---
|  | **点到站点** | **站点到站点** | **ExpressRoute** |
| --- | --- | --- | --- |
| **Azure 支持的服务** |云服务和虚拟机 |云服务和虚拟机 |[服务列表](../articles/expressroute/expressroute-faqs.md#supported-services) |
| **典型带宽** |基于网关 SKU |通常 < 1 Gbps（总计） |50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、2 Gbps、5 Gbps、10 Gbps |
| **支持的协议** |安全套接字隧道协议 (SSTP) 和 IPsec |IPsec |通过 VLAN、NSP 的 VPN 技术（MPLS、VPLS...）直接连接 |
| **路由** |基于路由（动态） |支持基于策略（静态路由）和基于路由（动态路由 VPN） |BGP |
| **连接复原能力** |主动-被动 |主动-被动或主动-主动 |主动-主动 |
| **典型用例** |云服务和虚拟机的原型设计、开发/测试/实验方案 |云服务和虚拟机的开发/测试/实验方案和小规模生产工作负荷 |访问所有 Azure 服务（已验证列表）、企业级和任务关键型工作负荷、备份、大数据、Azure 即 DR 站点 |
| **SLA** |[SLA](https://azure.microsoft.com/support/legal/sla/) |[SLA](https://azure.microsoft.com/support/legal/sla/) |[SLA](https://azure.microsoft.com/support/legal/sla/) |
| **定价** |[定价](https://azure.microsoft.com/pricing/details/vpn-gateway/) |[定价](https://azure.microsoft.com/pricing/details/vpn-gateway/) |[定价](https://azure.microsoft.com/pricing/details/expressroute/) |
| **技术文档** |[VPN 网关文档](https://azure.microsoft.com/documentation/services/vpn-gateway/) |[VPN 网关文档](https://azure.microsoft.com/documentation/services/vpn-gateway/) |[ExpressRoute 文档](https://azure.microsoft.com/documentation/services/expressroute/) |
| **常见问题** |[VPN 网关常见问题](../articles/vpn-gateway/vpn-gateway-vpn-faq.md) |[VPN 网关常见问题](../articles/vpn-gateway/vpn-gateway-vpn-faq.md) |[ExpressRoute 常见问题](../articles/expressroute/expressroute-faqs.md) |
