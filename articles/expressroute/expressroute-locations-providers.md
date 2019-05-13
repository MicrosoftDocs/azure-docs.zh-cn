---
title: 位置和连接提供商：Azure ExpressRoute | Microsoft Docs
description: 本文详细说明了服务的上市区域，以及如何连接到 Azure 区域。 按位置排序。
services: expressroute
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
ms.assetid: feb67da3-5abc-4acb-bad4-f78e3c541ded
ms.service: expressroute
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/08/2019
ms.author: pareshmu
ms.openlocfilehash: ac1d5f43d5711e6a688608e4cfc78e39221b8fac
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467780"
---
# <a name="expressroute-partners-and-peering-locations"></a>ExpressRoute 合作伙伴和对等位置

> [!div class="op_single_selector"]
> * [按提供商列出的位置](expressroute-locations.md)
> * [按位置列出的提供商](expressroute-locations-providers.md)


本文中的表格提供有关 ExpressRoute 连接提供商、ExpressRoute 地理覆盖范围、通过 ExpressRoute 支持的 Microsoft 云服务以及 ExpressRoute 系统集成商 (SI) 的信息。

## <a name="partners"></a>ExpressRoute 连接服务提供商
所有的 Azure 区域和位置都支持 ExpressRoute。 以下地图提供了 Azure 区域和 ExpressRoute 位置的列表。 ExpressRoute 位置是指 Microsoft 与多个服务提供商对等互连的位置。

![位置地图][0]

如果至少与地缘政治区域内的一个 ExpressRoute 位置连接，将有权访问地缘政治区域内所有区域中的 Azure 服务。 

### <a name="azure-regions-to-expressroute-locations-within-a-geopolitical-region"></a>地缘政治区域中 ExpressRoute 位置与 Azure 区域的映射
下表提供了地缘政治区域内 ExpressRoute 位置与 Azure 区域的映射。

| **地缘政治区域** | **区域** | **Azure 区域** | **ExpressRoute 位置** |
| --- | --- | --- | --- |
| **澳大利亚政府** | 第 | 澳大利亚中部、澳大利亚中部 2 |堪培拉、堪培拉 2 |
| **欧洲** | 第 |法国中部、法国南部、北欧、西欧、英国西部、英国南部 |阿姆斯特丹、 Amsterdam2、 都柏林、 法兰克福、 伦敦、 London2、 马赛、 纽波特 （威尔士）、 巴黎、 苏黎世 |
| **北美** | 第 |美国东部、美国西部、美国东部 2 区、美国西部 2 区、美国中部、美国中南部、美国中北部、美国中西部、加拿大中部、加拿大东部 |亚特兰大、 芝加哥、 达拉斯、 丹佛、 拉斯维加斯、 洛杉矶、 迈阿密、 纽约、 圣安东尼奥、 西雅图、 硅谷、 硅 Valley2、 华盛顿 DC，华盛顿 DC2、 蒙特利尔、 魁北克市、 多伦多 |
| **亚洲** | 2 |东亚、东南亚 |中国香港特别行政区，吉隆坡、 新加坡、 新加坡 2 台北 |
| **澳大利亚** | 2 |澳大利亚东南部、澳大利亚东部 |墨尔本、珀斯、悉尼 | 
| **印度** | 2 |印度西部、印度中部、印度南部 |金奈、金奈 2、孟买、孟买 2 |
| **日本** | 2 |日本西部、日本东部 |大坂、东京 |
| **韩国** | 2 |韩国中部、韩国南部 |釜山、首尔|
| **UAE** | 3 | 阿拉伯联合酋长国中部、 阿拉伯联合酋长国北部 | Dubai |
| **南非** | 3 |南非西部、 南非北部 |开普敦、约翰内斯堡 |
| **南美洲** | 3 |巴西南部 |Sao Paulo |

### <a name="regions-and-geopolitical-boundaries-for-national-clouds"></a>国家/地区云的区域和地缘政治边界
下表提供了有关国家/地区云的区域和地缘政治边界的信息。

| **地缘政治区域** | **Azure 区域** | **ExpressRoute 位置** |
| --- | --- | --- |
| **美国政府云** |US Gov 亚利桑那州、US Gov 爱荷华州、US Gov 德克萨斯州、US Gov 弗吉尼亚州、US DoD 中部、US DoD 东部  |芝加哥、达拉斯、纽约、凤凰城、圣安东尼奥、西雅图、硅谷、华盛顿特区 |
| **中国东部** |中国东部、中国东部 2 |上海、上海 2 |
| **中国北部** |中国北部、中国北部 2 |北京、北京 2 |
| **德国** |德国中部、德国东部 |柏林、法兰克福 |

标准 ExpressRoute SKU 不支持跨地缘政治区域的连接。 需要启用 ExpressRoute 高级版附加组件才能支持全球连接。 不支持连接到国家/地区云环境。 如果需要，请与连接服务提供商合作。

## <a name="locations"></a>连接服务提供商位置

下表显示连接位置和每个位置的服务提供商。 若要查看服务提供商及其提供服务的位置，请参阅[按服务提供商列出的位置](expressroute-locations.md#locations)。 


### <a name="production-azure"></a>生产 Azure
| **位置** | **对等位置所有者** | **服务提供商** |
| --- | --- | --- |
| **阿姆斯特丹** | Equinix | Aryaka Networks、AT&T NetBond、British Telecom、Colt、Equinix、euNetworks、GÉANT、InterCloud、Interxion、KPN、IX Reach、Level 3 Communications、Megaport、NTT Communications、Orange、Tata Communications、TeleCity Group、Telefonica、Telenor、Telia Carrier、Verizon、Zayo |
| **阿姆斯特丹 2** | Interxion | DE CIX、 Interxion |
| **亚特兰大** | Equinix | Equinix、Megaport |
| **釜山** |LG CNS | LG CNS |
| **堪培拉** | CDC | CDC |
| **堪培拉 2** | CDC | CDC |
| **开普敦** | Teraco | Internet Solutions - Cloud Connect、Liquid Telecom、Teraco |
| **金奈** | Tata Communications | Global CloudXchange (GCX)、SIFY、Tata Communications |
| **金奈 2** | Airtel | Airtel |
| **芝加哥** | Equinix | Aryaka Networks、 AT & T NetBond、 Cologix、 Comcast、 Coresite、 Equinix、 InterCloud、 Internet2，Level 3 Communications、 Megaport、 PacketFabric、 PCCW Global Limited、 冲刺 （sprint)、 Telia 运营商、 Verizon、 Zayo |
| **达拉斯** | Equinix | Aryaka Networks、 AT & T NetBond、 Cologix、 Equinix、 Internet2，Level 3 Communications、 Megaport、 Neutrona Networks、 Telmex Uninet Telia 运营商、 Verizon、 Zayo|
| **丹佛** | CoreSite | CoreSite、 Megaport、 Zayo |
| **Dubai** | Etisalat 阿拉伯联合酋长国 | Etisalat 阿拉伯联合酋长国 |
| **都柏林** | Equinix | Colt、eir、Equinix、Interxion、Megaport |
| **法兰克福** | Interxion | DE CIX、 Interxion |
| **香港特別行政区** | Equinix | Aryaka Networks、British Telecom、China Telecom Global、Equinix、Megaport、NTT Communications、Orange、PCCW Global Limited、Tata Communications、Verizon |
| **约翰内斯堡** | Teraco | Internet Solutions - Cloud Connect、Liquid Telecom、Teraco |
| **吉隆坡** | TIME dotCom | TIME dotCom |
| **拉斯维加斯** | 切换 | CenturyLink Cloud Connect、Megaport |
| **伦敦** | Equinix | AT&T NetBond、British Telecom、Colt、Equinix、InterCloud、Internet Solutions - Cloud Connect、Interxion、Jisc、Level 3 Communications、Megaport、MTN、NTT Communications、Orange、PCCW Global Limited、Tata Communications、Telehouse - KDDI、Telenor、Telia Carrier、Verizon、Vodafone、Zayo |
| **London2** | Telehouse | IX 市场宣传 Equinix |
| **洛杉矶** | CoreSite | CoreSite、Equinix、Megaport、Neutrona Networks、NTT、Zayo |
| **马赛** |Interxion | Interxion |
| **墨尔本** | NextDC | AARNet、 Equinix、 Megaport、 NEXTDC、 Optus、 Telstra Corporation，TPG 电信 |
| **迈阿密** | Equinix | C3ntro+、Equinix、Megaport、Neutrona Networks |
| **Montreal** | Cologix | Bell Canada、Cologix、Telus、Zayo |
| **Mumbai** | Tata Communications | Global CloudXchange (GCX)、 依赖性 Jio、 Sify、 Tata Communications、 |
| **孟买 2** | Airtel | Airtel、Sify、Vodafone Idea |
| **纽约** | Equinix | CenturyLink Cloud Connect、 Coresite、 Equinix、 InterCloud、 Megaport、 数据包、 Zayo |
| **Newport(Wales)** | Next Generation Data | British Telecom、Level 3 Communications、Next Generation Data |
| **大阪** | Equinix | Equinix、Internet Initiative Japan Inc. - IIJ、NTT Communications、NTT SmartConnect、Softbank |
| **巴黎** | Interxion | Colt、 Equinix、 Intercloud、 Interxion、 橙色、 Telia 运营商、 Zayo |
| **珀斯** | NextDC | Megaport、 NextDC |
| **魁北克市** | 4Degrees | Bell Canada、Megaport |
| **圣安东尼奥** | CyrusOne | CenturyLink Cloud Connect、Megaport |
| **圣保罗** | Equinix | Aryaka Networks、Ascenty Data Centers、British Telecom、Equinix、Level 3 Communications、Neutrona Networks、Orange、Tata Communications、Telefonica、UOLDIVEO |
| **西雅图** | Equinix | Aryaka Networks、Equinix、Level 3 Communications、Megaport、Zayo |
| **首尔** | KINX | KINX、LG CNS、Sejong Telecom |
| **硅谷** | Equinix | Aryaka Networks，AT 和 T NetBond、 British Telecom、 CenturyLink Cloud Connect、 Comcast、 Coresite、 Equinix、 InterCloud、 IX 达到，数据包、 PacketFabric、 Level 3 Communications、 Megaport、 Orange、 冲刺 （sprint）、 Tata Communications、 Verizon、 Zayo |
| **硅谷 Valley2** | Coresite | Coresite | 
| **新加坡** | Equinix | Aryaka Networks、AT&T NetBond、British Telecom、Epsilon Global Communications、Equinix、InterCloud、Level 3 Communications、Megaport、NTT Communications、Orange、SingTel、Tata Communications、Telstra Corporation、Verizon、Vodafone |
| **新加坡 2** | Global Switch | Colt、 Epsilon 全局 Communications、 Megaport、 SingTel |
| **悉尼** | Equinix | AARNet、 AT & T NetBond，British Telecom，Devoli，Equinix，Kordia，Megaport、 NEXTDC、 NTT 通信，Optus，橙色，Telstra Corporation，TPG Telecom、 Verizon |
| **台北** | Chief Telecom | 首席 Telecom、 FarEasTone |
| **东京** | Equinix | Aryaka Networks、AT&T NetBond、British Telecom、CenturyLink Cloud Connect、Colt、Equinix、Internet Initiative Japan Inc. - IIJ、NTT Communications、NTT EAST、Orange、Softbank、Verizon |
| **多伦多** | Cologix | At&t netbond、 Bell Canada、 CenturyLink Cloud Connect、 Cologix、 Equinix、 IX 达到 Megaport、 Telus、 Zayo |
| **华盛顿特区** | Equinix | Aryaka Networks、 AT & T NetBond、 British Telecom、 Cologix、 Comcast、 Coresite、 Equinix、 Internet2、 InterCloud、 Level 3 Communications、 Megaport、 Neutrona Networks、 NTT Communications、 Orange、 PacketFabric、 冲刺 （sprint）、 Tata Communications、 Telia 承运人Verizon、 Zayo |
| **华盛顿特区 2** | Coresite | Coresite | 
| **Zurich** | Interxion | Interxion |

 **+** 表示即将推出

### <a name="national-cloud-environments"></a>国家/地区云环境

### <a name="us-government-cloud"></a>美国政府云
| **位置** | **服务提供商** |
| --- | --- |
| **芝加哥** |AT&T NetBond、Equinix、Level 3 Communications、Verizon |
| **达拉斯** |Equinix、Megaport、Verizon |
| **纽约** |Equinix、CenturyLink Cloud Connect、Verizon |
| **Phoenix** | 在 & T NetBond、 CenturyLink Cloud Connect |
| **圣安东尼奥** | CenturyLink Cloud Connect、Megaport |
| **硅谷** | Equinix、Level 3 Communications、Verizon |
| **西雅图** | Equinix、Megaport |
| **华盛顿特区** |At&t netbond、 Equinix、 Level 3 Communications、 Megaport、 Verizon |

### <a name="china"></a>中国
| **位置** | **服务提供商** |
| --- | --- |
| **北京** |中国电信 |
| **北京 2** | GDS |
| **上海** |中国电信 |
| **上海 2** | GDS |

若要了解详细信息，请参阅 [位于中国的 ExpressRoute](http://www.windowsazure.cn/home/features/expressroute/)

### <a name="germany"></a>德国
| **位置** | **服务提供商** |
| --- | --- |
| **柏林** |e-shelter、Megaport+、T-Systems |
| **法兰克福** |Colt、Equinix、Interxion |

## <a name="c1partners"></a>通过 Exchange 提供商建立的连接
如果前面部分中未列出连接服务提供商，仍可以建立连接。

* 请咨询连接服务提供商，以确定他们是否已连接到上表中列出的任何 Exchange 提供商。 可以访问以下链接，以收集 Exchange 提供商所提供的服务的详细信息。 已有多个连接提供商连接到以太网 Exchange。
  * [Cologix](https://www.cologix.com/)
  * [CoreSite](https://www.coresite.com/)
  * [Equinix Cloud Exchange](https://www.equinix.com/services/interconnection-connectivity/cloud-exchange/)
  * [InterXion](https://www.interxion.com/)
  * [NextDC](https://www.nextdc.com/)
  * [Megaport](https://www.megaport.com/services/microsoft-expressroute/)
  * [PacketFabric](https://www.packetfabric.com/packetcor/microsoft-azure/)
  
* 让连接提供商将网络扩展到选择的对等互连位置。
  * 确保连接服务提供商以高可用性方式扩展连接，以防出现单点故障。
* 从 Exchange 连接服务提供商处订购一条 ExpressRoute 线路以连接到 Microsoft。
  * 根据 [创建 ExpressRoute 线路](expressroute-howto-circuit-classic.md) 中的步骤来设置连接。

## <a name="c1partners"></a>通过其他服务提供商建立的连接
| **位置** | **Exchange** | **连接提供商** |
| --- | --- | --- |
| **阿姆斯特丹** | Equinix、Telecity | BICS、CloudXpress、Eurofiber、Fastweb S.p.A、Gulf Bridge International、MainOne、Nianet、Post、Proximus、TDC Erhverv、Telecom Italia Sparkle、Telia |
| **开普敦** | Teraco | MTN |
| **芝加哥** | Equinix | Lightower、Windstream |
| **达拉斯** | Equinix、Megaport | Axtel、C3ntro Telecom、Cox Business、Data Foundry、Transtelco |
| **法兰克福** | Telecity | BICS、Cinia、Nianet、QSC AG |
| **汉堡** | Equinix | Cinia |
| **香港特別行政区** | Equinix | Chief、Macroview Telecom |
| **约翰内斯堡** | Teraco | MTN |
| **伦敦** | BICS、Equinix、euNetworks、Telecity | Bezeq International Ltd.、 CoreAzure，Telecommunications Limited、 Exponential E、 HSO、 NexGen Networks、 Proximus、 Tamares Telecom、 Zain |
| **洛杉矶** | Equinix |Transtelco |
| **马德里** | Level3 | Zertia |
| **Montreal** | Cologix、Equinix | Airgate Technologies, Inc.Cogeco Peer 1、Rogers、Zirro |
| **纽约** |Equinix、Megaport | Altice Business、Lightower、Webair |
| **巴黎** | Equinix | Proximus |
| **魁北克市** | Megaport | Fibrenoire |
| **圣保罗** | Equinix | Venha Pra Nuvem |
| **西雅图** |Equinix | Alaska Communications |
| **硅谷** |Equinix | Cox Business、Windstream |
| **新加坡** |Equinix |1CLOUDSTAR、BICS、Epsilon Telecommunications Limited、LGA Telecom、United Information Highway (UIH) |
| **Slough** | Equinix | HSO|
| **悉尼** | Megaport | Macquarie Telecom Group|
| **东京** | Equinix | ARTERIA Networks Corporation、BroadBand Tower, Inc. |
| **多伦多** | Equinix | Airgate Technologies, Inc.Cogeco Peer 1、IVedha Inc、Rogers、Thinktel、Zirro|
| **华盛顿特区** |Equinix | Altice Business、BICS、Cox Business、Gtt Communications Inc、Epsilon、Telecommunications Limited、Lightower、Masergy、Windstream |

## <a name="expressroute-system-integrators"></a>ExpressRoute 系统集成商
根据网络的规模，有时，很难启用专用连接来满足需要。 可以与下表中列出的任一系统集成商合作，以帮助将你加入 ExpressRoute。

| **所在洲** | **系统集成商** |
| --- | --- |
| **亚洲** |Avanade Inc.、OneAs1a |
| **澳大利亚** | Ensyst、IT Consultancy、MOQdigital、Vigilant.IT |
| **欧洲** |Avanade Inc.、Altogee、Bright Skies GmbH、Inframon、MSG Services、New Signature、Nelite、Orange Networks、sol-tec |
| **北美** |Avanade Inc.、Equinix Professional Services、FlexManage、Lightstream、Perficient、Presidio |
| **南美洲** |Avanade Inc.、Venha Pra Nuvem |
## <a name="next-steps"></a>后续步骤
* 有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](expressroute-faqs.md)。
* 确保符合所有先决条件。 请参阅 [ExpressRoute 先决条件](expressroute-prerequisites.md)。

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "位置地图"
