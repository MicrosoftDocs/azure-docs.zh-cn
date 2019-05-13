---
title: 连接提供商和位置：Azure ExpressRoute | Microsoft Docs
description: 本文详细说明了服务的上市区域，以及如何连接到 Azure 区域。 按连接服务提供商排序。
services: expressroute
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
ms.assetid: c878513a-d594-42ad-8b0e-403efd0c4b25
ms.service: expressroute
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/08/2019
ms.author: pareshmu
ms.openlocfilehash: 63d6d1419b0a70a0bcf3532a13d6f7491b792593
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467710"
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

### <a name="azure-regions-to-expressroute-locations-within-a-geopolitical-region"></a>地缘政治区域中 ExpressRoute 位置与 Azure 区域的映射。
下表提供了地缘政治区域内 ExpressRoute 位置与 Azure 区域的映射。

| **地缘政治区域** | **区域** | **Azure 区域** | **ExpressRoute 位置** |
| --- | --- | --- | --- |
| **澳大利亚政府** | 第 | 澳大利亚中部、澳大利亚中部 2 |堪培拉、堪培拉 2 |
| **欧洲** | 第 |法国中部、法国南部、北欧、西欧、英国西部、英国南部 |阿姆斯特丹、 Amsterdam2、 都柏林、 法兰克福、 伦敦、 London2、 马赛、 纽波特 （威尔士）、 巴黎、 苏黎世 |
| **北美** | 1 |美国东部、美国西部、美国东部 2 区、美国西部 2 区、美国中部、美国中南部、美国中北部、美国中西部、加拿大中部、加拿大东部 |亚特兰大、 芝加哥、 达拉斯、 丹佛、 拉斯维加斯、 洛杉矶、 迈阿密、 纽约、 圣安东尼奥、 西雅图、 硅谷、 硅 Valley2、 华盛顿 DC，华盛顿 DC2、 蒙特利尔、 魁北克市、 多伦多 |
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

下表显示按服务提供商列出的位置。 若要按位置查看可用的提供商，请参阅[按位置列出的服务提供商](expressroute-locations-providers.md#locations)。


### <a name="production-azure"></a>生产 Azure

| **服务提供商** | **Microsoft Azure** | **Office 365 和 Dynamics 365** | **位置** |
| --- | --- | --- | --- |
| **[AARNet](https://www.aarnet.edu.au/network-and-services/cloud-services-applications/azure-expressroute/)** |支持 |支持 |墨尔本、悉尼 |
| **[Airtel](https://www.airtel.in/creatingsmiles/)** | 支持 | 支持 | 金奈 2、孟买 2 |
| **[Aryaka Networks](https://www.aryaka.com/)** |支持 |支持 |阿姆斯特丹、 芝加哥、 达拉斯、 中国香港特别行政区、 圣保罗、 西雅图、 硅谷、 新加坡、 东京、 华盛顿特区 |
| **[Ascenty Data Centers](https://ascenty.com/servicos/cloud-connect/microsoft-expressroute/)** |支持 |支持 |Sao Paulo |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |支持 |支持 |阿姆斯特丹、芝加哥、达拉斯、伦敦、硅谷、新加坡、悉尼、东京、多伦多、华盛顿特区 |
| **[Bell Canada](https://business.bell.ca/shop/enterprise/cloud-connect-access-to-cloud-partner-services)** |支持 |支持 |蒙特利尔、多伦多、魁北克市 |
| **[British Telecom](https://www.globalservices.bt.com/en/solutions/products/bt-compute-for-microsoft-azure)** |支持 |支持 |阿姆斯特丹、 中国香港特别行政区、 伦敦、 纽波特 （威尔士）、 圣保罗、 硅谷、 新加坡、 悉尼、 东京、 华盛顿特区 |
| **C3ntro** |即将推出 |即将推出 |迈阿密 |
| **CDC** | 支持 | 支持 | 堪培拉、堪培拉 2 |
| **[CenturyLink Cloud Connect](https://www.centurylink.com/cloudconnect)** |支持 |支持 |拉斯维加斯、纽约、圣安东尼奥、硅谷、东京、多伦多 |
| **Chief Telecom** |支持 |支持 |台北 |
| **China Telecom Global** |支持 |不受支持 |香港特别行政区 |
| **[Cologix](https://www.cologix.com/solutions/cloud-connect/public-clouds/microsoft-cloud/)** |支持 |支持 |芝加哥、达拉斯、蒙特利尔、多伦多、华盛顿特区 |
| **[Colt](https://www.colt.net/uk/en/news/colt-announces-dedicated-cloud-access-for-microsoft-azure-services-en.htm)** |支持 |支持 |阿姆斯特丹、都柏林、伦敦、巴黎、新加坡 2、东京 |
| **[Comcast](https://business.comcast.com/landingpage/microsoft-azure)** |支持 |支持 |芝加哥、硅谷、华盛顿特区 |
| **[CoreSite](https://www.coresite.com/solutions/cloud-services/public-cloud-providers/microsoft-azure-expressroute)** |支持 |支持 |芝加哥、 丹佛、 洛杉矶、 纽约、 硅谷、 硅 Valley2 华盛顿 DC，华盛顿 DC2 |
| **[DE-CIX](https://www.de-cix.net/en/de-cix-service-world/directcloud/find-a-cloud-service/detail/microsoft-azure)** | 支持 |支持 |Amsterdam2、 法兰克福|
| **eir** |支持 |支持 |Dublin|
| **Epsilon Global Communications** |支持 |支持 |新加坡、新加坡 2 |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |支持 |支持 |阿姆斯特丹、 亚特兰大、 芝加哥、 达拉斯、 都柏林、 中国香港特别行政区、 伦敦、 London2、 洛杉矶、 墨尔本、 迈阿密、 纽约、 大阪、 巴黎、 圣保罗、 西雅图、 硅谷、 新加坡、 悉尼、 东京、 多伦多、 华盛顿特区 |
| **Etisalat 阿拉伯联合酋长国** |支持 |支持 |Dubai|
| **[euNetworks](https://eunetworks.com/services/solutions/cloud-connect/microsoft-azure-expressroute/)** |支持 |支持 |阿姆斯特丹、都柏林、伦敦 |
| **FarEasTone** |支持 |支持 |台北|
| **GÉANT** |支持 |支持 |阿姆斯特丹 |
| **[Global Cloud Xchange (GCX)](https://globalcloudxchange.com/cloud-platform/cloud-x-fusion/)** | 支持| 支持 | 金奈、孟买 |
| **[InterCloud](https://www.intercloud.com/)** |支持 |支持 |阿姆斯特丹、 芝加哥、 伦敦、 纽约、 巴黎、 硅谷、 新加坡、 华盛顿特区 |
| **Internet2** |支持 |支持 |芝加哥、达拉斯、华盛顿特区 |
| **[Internet Initiative Japan Inc. - IIJ](https://www.iij.ad.jp/en/news/pressrelease/2015/1216-2.html)** |支持 |支持 |大坂、东京 |
| **[Internet Solutions - Cloud Connect](https://www.is.co.za/solution/cloud-connect/)** |支持 |支持 |开普敦、约翰内斯堡、伦敦 |
| **[Interxion](https://www.interxion.com/why-interxion/colocate-with-the-clouds/Microsoft-Azure/)** |支持 |支持 |阿姆斯特丹、 Amsterdam2、 都柏林、 法兰克福、 伦敦、 马赛、 巴黎、 苏黎世 |
| **[IX Reach](https://www.ixreach.com/partners/cloud-partners/microsoft-azure/)**|支持 |支持 | 阿姆斯特丹、 London2、 硅谷、 多伦多 |
| **[Jisc](https://www.jisc.ac.uk/microsoft-azure-expressroute)** |支持 |支持 |伦敦 |
| **[KINX](https://www.kinx.net/service/network/cloudhub/ms-expressroute/?lang=en)** |支持 |支持 |Seoul |
| **[Kordia](https://www.kordia.co.nz/cloudconnect)** | 支持 |支持 |悉尼 |
| **[KPN](https://www.kpn.com/zakelijk/cloud/connect.htm)** | 支持 | 支持 | 阿姆斯特丹 | 
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |支持 |支持 |阿姆斯特丹、芝加哥、达拉斯、伦敦、新港（威尔士）、圣保罗、西雅图、硅谷、新加坡、华盛顿特区 |
| **LG CNS** |支持 |支持 |釜山、首尔 |
| **[Liquid Telecom](https://www.liquidtelecom.com/products-and-services/cloud.html)** |支持 |支持 |开普敦、约翰内斯堡 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支持 |支持 |阿姆斯特丹、 亚特兰大、 芝加哥、 达拉斯、 丹佛、 都柏林、 中国香港特别行政区、 拉斯维加斯、 伦敦、 洛杉矶、 墨尔本、 迈阿密、 纽约、 珀斯、 魁北克市、 圣安东尼奥、 西雅图、 硅谷、 新加坡、 新加坡 2、 悉尼、 多伦多、 华盛顿特区 |
| **[MTN](https://www.mtnbusiness.com/en/enterprise/Pages/microsoft-express-route.aspx)** |支持 |支持 |伦敦 |
| **[Neutrona Networks](http://www.neutrona.com/index.php/azure-expressroute/)** |支持 |支持 |达拉斯、洛杉矶、迈阿密、圣保罗 |
| **[Next Generation Data](https://www.nextgenerationdata.co.uk/ngd-cloud-gateway/)** |支持 |支持 |Newport(Wales) |
| **[NEXTDC](https://www.nextdc.com/services/axon-ethernet/microsoft-expressroute)** |支持 |支持 |墨尔本、珀斯、悉尼 |
| **[NTT Communications](https://www.ntt.com/en/services/network/virtual-private-network.html)** |支持 |支持 |阿姆斯特丹、 中国香港特别行政区、 伦敦、 洛杉矶、 大阪、 新加坡、 悉尼、 东京、 华盛顿特区 |
| **[NTT EAST](https://flets.com/cloudgateway/crossconnect/)** |支持 |支持 |东京 |
| **[NTT SmartConnect](https://cloud.nttsmc.com/cxc/azure.html)** |支持 |支持 |大阪 |
| **[Optus](https://www.optus.com.au/enterprise/networking/network-connectivity/express-link)** |支持 |支持 |墨尔本、悉尼 |
| **[Orange](https://www.orange-business.com/en/products/business-vpn-galerie)** |支持 |支持 |阿姆斯特丹、 中国香港特别行政区、 伦敦、 巴黎、 圣保罗、 硅谷、 新加坡、 悉尼、 东京、 华盛顿特区 |
| **[PacketFabric](https://www.packetfabric.com/packetcor/microsoft-azure/)** |支持 |支持 |芝加哥、硅谷、华盛顿特区 |
| **[PCCW Global Limited](https://consoleconnect.com/clouds/#azureRegions)** |支持 |支持 |芝加哥、 香港特别行政区、 伦敦 |
| **[Sejong Telecom](https://www.sejongtelecom.net/en/pages/service/cloud_ms)** |支持 |支持 |首尔 |
| **[SIFY](http://telecom.sify.com/azure-expressroute.html)** |支持 |支持 |金奈、孟买 2 |
| **[SingTel](http://info.singtel.com/about-us/news-releases/singtel-provide-secure-private-access-microsoft-azure-public-cloud)** |支持 |支持 |新加坡、新加坡 2 |
| **[Softbank](https://www.softbank.jp/biz/cloud/cloud_access/direct_access_for_az/)** |支持 |支持 |大坂、东京 |
| **[Sprint](https://business.sprint.com/solutions/cloud-networking/)** |支持 |支持 |芝加哥、硅谷、华盛顿特区 |
| **[Tata Communications](https://www.tatacommunications.com/lp/izo/azure/azure_index.html)** |支持 |支持 |阿姆斯特丹、 金奈、 中国香港特别行政区、 伦敦、 孟买、 圣保罗、 硅谷、 新加坡、 华盛顿特区 |
| **Telecity Group** |支持 |支持 |阿姆斯特丹 |
| **[Telefonica](https://www.business-solutions.telefonica.com/es/enterprise/solutions/efficient-infrastructure/managed-voice-data-connectivity/)** |支持 |支持 |阿姆斯特丹、圣保罗 |
| **[Telehouse - KDDI](https://www.telehouse.net/solutions/cloud-services/cloud-link)** |支持 |支持 |伦敦 |
| **Telenor** |支持 |支持 |阿姆斯特丹、伦敦 |
| **[Telia Carrier](https://teliacarrier.com/our-services/connectivity/cloud-connect.html?title=Cloud%20Connect)** | 支持 | 支持 |阿姆斯特丹、 芝加哥、 达拉斯、 伦敦、 巴黎、 华盛顿特区 |
| **Telmex Uninet**| 支持 | 支持 | 达拉斯 |
| **[Telstra Corporation](https://www.telstra.com.au/business-enterprise/network-services/networks/cloud-direct-connect/)** |支持 |支持 |墨尔本、新加坡、悉尼 |
| **[Telus](https://www.telus.com)** |支持 |支持 |蒙特利尔、多伦多 |
| **[Teraco](https://www.teraco.co.za/services/africa-cloud-exchange/)** |支持 |支持 |开普敦、约翰内斯堡 |
| **[TIME dotCom](https://www.time.com.my/enterprise/connectivity/cloud-interconnect)** | 支持 | 支持 | Kuala Lumpur |
| **[UOLDIVEO](https://www.uoldiveo.com.br/)** |支持 |支持 |Sao Paulo |
| **[Verizon](https://enterprise.verizon.com/products/network/application-enablement/secure-cloud-interconnect/)** |支持 |支持 |阿姆斯特丹、 芝加哥、 达拉斯、 中国香港特别行政区、 伦敦、 硅谷、 新加坡、 悉尼、 东京、 华盛顿特区 |
| **[Vodafone](https://www.vodafone.com/business/global-enterprise/global-connectivity/vodafone-ip-vpn-cloud-connect)** |支持 |支持 |伦敦、新加坡 |
| **Vodafone Idea** | 支持 | 支持 | 孟买、孟买 2 |
| **[Zayo](https://www.zayo.com/solutions/industries/cloud-connectivity/microsoft-expressroute)** |支持 |支持 |阿姆斯特丹、 芝加哥、 达拉斯、 丹佛、 伦敦、 洛杉矶、 蒙特利尔、 纽约、 巴黎、 西雅图、 硅谷、 多伦多、 华盛顿特区 |

 **+** 表示即将推出

### <a name="national-cloud-environment"></a>国家/地区云环境

### <a name="us-government-cloud"></a>美国政府云

| **服务提供商** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |支持 |支持 |芝加哥、 Phoenix、 华盛顿特区 |
| **[CenturyLink Cloud Connect](https://www.centurylink.com/cloudconnect)** |支持 |支持 |纽约，Phoenix，圣安东尼奥 |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |支持 |支持 |芝加哥、达拉斯、纽约、西雅图、硅谷、华盛顿特区 |
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |支持 |支持 |芝加哥、硅谷、华盛顿特区 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支持 | 支持 | 芝加哥、 达拉斯、 圣安东尼奥、 西雅图、 华盛顿特区 |
| **[Verizon](http://news.verizonenterprise.com/2014/04/secure-cloud-interconnect-solutions-enterprise/)** |支持 |支持 |芝加哥、达拉斯、纽约、硅谷、华盛顿特区 |

### <a name="china"></a>中国

| **服务提供商** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **中国电信** |支持 |不受支持 |北京、上海 |
| **[GDS](http://en.gds-services.com/news_detail/newsId=21.html)** |支持 |不受支持 |北京 2、上海 2 |

若要了解详细信息，请参阅 [位于中国的 ExpressRoute](http://www.windowsazure.cn/home/features/expressroute/)。

### <a name="germany"></a>德国

| **服务提供商** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **[Colt](https://www.colt.net/uk/en/news/colt-announces-dedicated-cloud-access-for-microsoft-azure-services-en.htm)** |支持 |不受支持 |法兰克福 |
| **[Equinix](https://www.equinix.com/partners/microsoft-azure/)** |支持 |不受支持 |法兰克福 |
| **[e-shelter](https://www.e-shelter.de/en/microsoft-expressroutetm)** |支持 |不受支持 |柏林 |
| **Interxion** |支持 |不受支持 |法兰克福 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支持  | 不受支持 | 柏林 |
| **T-Systems** |支持 |不受支持 |柏林 |

## <a name="connectivity-through-exchange-providers"></a>通过 Exchange 提供商建立的连接

如果前面部分中未列出连接服务提供商，仍可以建立连接。

* 请咨询连接服务提供商，以确定他们是否已连接到上表中列出的任何 Exchange 提供商。 可以访问以下链接，以收集 Exchange 提供商所提供的服务的详细信息。 已有多个连接提供商连接到以太网 Exchange。
  * [Cologix](https://www.cologix.com/)
  * [CoreSite](https://www.coresite.com/)
  * [Equinix Cloud Exchange](https://www.equinix.com/services/interconnection-connectivity/cloud-exchange/)
  * [Interxion](https://www.interxion.com/products/interconnection/cloud-connect/)
  * [IX Reach](https://www.ixreach.com/partners/cloud-partners/microsoft-azure/)
  * [Megaport](https://www.megaport.com/services/microsoft-expressroute/)
  * [NextDC](https://www.nextdc.com/)
  * [PacketFabric](https://www.packetfabric.com/packetcor/microsoft-azure/) 
* 让连接提供商将网络扩展到选择的对等互连位置。
  * 确保连接服务提供商以高可用性方式扩展连接，以防出现单点故障。
* 从 Exchange 连接服务提供商处订购一条 ExpressRoute 线路以连接到 Microsoft。
  * 根据 [创建 ExpressRoute 线路](expressroute-howto-circuit-classic.md) 中的步骤来设置连接。

## <a name="connectivity-through-additional-service-providers"></a>通过其他服务提供商建立的连接

| **连接服务提供商** | **Exchange** | **位置** |
| --- | --- | --- |
| **[1CLOUDSTAR](https://www.1cloudstar.com/service/cloudconnect-azure-expressroute/)** |Equinix |新加坡 |
| **[Airgate Technologies, Inc.](https://www.airgate.ca/expressroute)** | Equinix Cologix | 多伦多、蒙特利尔 |
| **[Alaska Communications](https://www.alaskacommunications.com/For-Your-Business/Direct-Cloud-Service)** |Equinix |西雅图 |
| **[Altice Business](https://golightpath.com/transport)** |Equinix |纽约、华盛顿特区 |
| **[Arteria Networks Corporation](https://www.arteria-net.com/business/service/cloud/sca/)** |Equinix |东京 |
| **[Axtel](https://alestra.mx/landing/expressrouteazure/)** |Equinix |达拉斯|
| **[Bezeq International Ltd.](https://www.bezeqint.net/english)** | euNetworks | 伦敦 |
| **[BICS](https://bics.com/bics-solutions-suite/cloud-connect/)** | Equinix | 阿姆斯特丹、法兰克福、伦敦、新加坡、华盛顿特区 |
| **[BroadBand Tower, Inc.](https://www.bbtower.co.jp/product-service/data-center/network/dcconnect-for-azure/)** | Equinix | 东京 |
| **[C3ntro Telecom](https://www.c3ntro.com/data/express-route)** | Equinix、Megaport | 达拉斯 |
| **[Chief](https://www.chief.com.tw/dispPageBox/ct.aspx?ddsPageID=ENCLOUDSERVICE&dbid=4852937342)** | Equinix | 香港特别行政区 |
| **[Cinia](https://www.cinia.fi/en/services/connectivity-services/direct-public-cloud-connection.html)** | Equinix、Megaport | 法兰克福、汉堡 |
| **[CloudXpress](https://www2.telenet.be/fr/business/produits-services/internet/cloudxpress/)** | Equinix | 阿姆斯特丹 | 
| **[Cogeco Peer 1](https://www.cogecopeer1.com/en/)**| Equinix | 蒙特利尔、多伦多 |
| **[CoreAzure](http://coreazure.com/expressroute)**| Equinix | 伦敦 |
| **[Cox Business](https://www.cox.com/business/networking/cloud-connectivity.html)** | Equinix | 达拉斯、硅谷、华盛顿特区 | 
| **[Data Foundry](https://www.datafoundry.com/services/cloud-connect)** | Megaport | 达拉斯 |
| **[Epsilon Telecommunications Limited](https://www.epsilontel.com/data-connectivity/cloud-access/)** | Equinix | 伦敦、新加坡、华盛顿特区 |
| **[Eurofiber](https://eurofiber.nl/microsoft-azure/)** | Equinix | 阿姆斯特丹 |
| **[Exponential E](https://www.exponential-e.com/services/connectivity-services/cloud-connect-exchange)** | Equinix | 伦敦 |
| **[Fastweb S.p.A](https://www.fastweb.it/grandi-aziende/connessione-voce-e-wifi/scheda-prodotto/rete-privata-virtuale/)** | Equinix | 阿姆斯特丹 |
| **[Fibrenoire](https://www.fibrenoire.ca/en/cloudextn)** | Megaport | 魁北克市 |
| **[Gtt Communications Inc](https://www.gtt.net)** |Equinix | 华盛顿特区 |
| **[Gulf Bridge International](https://www.gbiinc.com/microsoft-azure-expressroute/)** | Equinix | 阿姆斯特丹 |
| **[HSO](https://www.hso.co.uk/products/cloud-direct)** |Equinix | 伦敦、斯劳 |
| **[IVedha Inc](http://www.ivedha.com/cloud/manage-azure-cloud/express-route-4/)**| Equinix | 多伦多 |
| **LGA Telecom** |Equinix |新加坡|
| **[Lightower](http://www.lightower.com/network-solutions/cloud-connect/#microsoft-azure)** |Equinix | 芝加哥、纽约、华盛顿特区 |
| **[Macroview Telecom](http://www.macroview.com/en/scripts/catitem.php?catid=solution&sectionid=expressroute)** |Equinix |香港特别行政区 |
| **[Macquarie Telecom Group](https://macquariegovernment.com/secure-cloud/secure-cloud-exchange/)** | Megaport | 悉尼 |
| **[MainOne](https://www.mainone.net/services/connectivity/cloud-connect/)** |Equinix | 阿姆斯特丹 |
| **[Masergy](https://www.masergy.com/solutions/hybrid-networking/cloud-marketplace/microsoft-azure)** | Equinix | 华盛顿特区 |
| **[MTN](https://www.mtnbusiness.com/en/enterprise/Pages/microsoft-express-route.aspx)** | Teraco | 开普敦、约翰内斯堡 |
| **[NexGen Networks](https://www.nexgen-net.com/nexgen-networks-direct-connect-microsoft-azure-expressroute.html)** | Interxion | 伦敦 |
| **[Nianet](https://nianet.dk/produkter/internet/microsoft-expressroute)** |Telecity | 阿姆斯特丹、法兰克福 |
| **[Post](https://www.teralinksolutions.com/cloud-connectivity/cloudbridge-to-azure-expressroute/)**|Equinix | 阿姆斯特丹 |
| **[Proximus](https://www.proximus.be/en/id_b_cl_proximus_external_cloud_connect/companies-and-public-sector/discover/magazines/expert-blog/proximus-external-cloud-connect.html)**|Equinix | 阿姆斯特丹、都柏林、伦敦、巴黎 |
| **[QSC AG](https://www.qsc.de/de/produkte-loesungen/cloud-services-und-it-outsourcing/pure-enterprise-cloud/multi-cloud-management/azure-expressroute/)** |Interxion | 法兰克福 |  
| **Rogers** | Cologix、Equinix | 蒙特利尔、多伦多 |
| **[Tamares Telecom](http://www.tamarestelecom.com/our-services/#Connectivity)** | Telecity | 伦敦 | 
| **[TDC Erhverv](https://tdc.dk/Produkter/cloudaccessplus)** | Equinix | 阿姆斯特丹 | 
| **[Telecom Italia Sparkle](https://www.tisparkle.com/our-platform/corporate-platform/sparkle-cloud-connect#catalogue)**| Equinix | 阿姆斯特丹 |
| **[Telia](https://www.telia.se/foretag/losningar/produkter-tjanster/datanet)** | Equinix | 阿姆斯特丹 |
| **[ThinkTel](https://www.thinktel.ca/services/agile-ix-data/expressroute/)** | Equinix | 多伦多 | 
| **[Transtelco](https://www.transtelco.net/tcloud/microsoft)** |Equinix | 达拉斯、洛杉矶 |
| **[United Information Highway (UIH)](https://www.uih.co.th/en/internet-solution/cloud-direct/uih-cloud-direct-for-microsoft-azure-expressroute)**| Equinix | 新加坡 |
| **[Venha Pra Nuvem](https://venhapranuvem.com.br/)** | Equinix | Sao Paulo |
| **[Webair](https://www.webair.com/microsoft-express-route-partnership/)**| Megaport | 纽约 |
| **[Windstream](https://www.windstreambusiness.com/solutions/cloud-services/cloud-and-managed-hosting-services)**| Equinix | 芝加哥、硅谷、华盛顿特区 |
| **Zain** |Equinix |伦敦|
| **[Zertia](https://www.zertia.es)**| Level 3 | Madrid |
| **[Zirro](https://zirro.com/services/)**| Equinix | 蒙特利尔、多伦多 |

## <a name="connectivity-through-datacenter-providers"></a>通过数据中心提供商建立的连接

| **提供程序** | **Exchange** |
| --- | --- |
| **[Cyrus One](https://cyrusone.com/enterprise-data-center-services/connectivity-and-interconnection/cloud-connectivity-reaching-amazon-microsoft-google-and-more/microsoft-azure-expressroute/?doing_wp_cron=1498512235.6733090877532958984375)** | Megaport |
| **[Cyxtera](https://www.cyxtera.com/data-center-services/interconnection)** | Megaport |
| **[Digital Realty](https://www.digitalrealty.com/services/interconnection/service-exchange/)** | Megaport |
| **[EdgeConnex](https://www.edgeconnex.com/services/edge-data-centers-proximity-matters/)** | Megaport |
| **[RagingWire Data Centers](https://www.ragingwire.com/wholesale/wholesale-data-centers-worldwide-nexcenters)** | IX Reach |
| **[T5 Datacenters](https://t5datacenters.com/network-cloud-connect/)** | IX Reach |

## <a name="connectivity-through-national-research-and-education-networks-nren"></a>通过国家科研与教育网络 (NREN) 建立的连接

| **提供程序**|
| --- |
| **AARNET**| 
| **DeIC，通过 GÉANT**|
| **GARR，通过 GÉANT**|
| **GÉANT**|
| **HEAnet，通过 GÉANT**|
| **Internet2**|
| **JISC**|
| **RedIRIS，通过 GÉANT**|
| **SINET**|
| **Surfnet，通过 GÉANT**|

* 如果连接提供商未在此处列出，请查看其是否连接到上面列出的任何 ExpressRoute Exchange 合作伙伴。

## <a name="expressroute-system-integrators"></a>ExpressRoute 系统集成商
根据网络的规模，有时，很难启用专用连接来满足需要。 可以与下表中列出的任一系统集成商合作，以帮助你将加入 ExpressRoute。

| **系统集成商** | **所在洲** |
| --- | --- |
| **[Altogee](https://altogee.be/diensten/express-route/)** | 欧洲 |
| **[Avanade Inc.](https://www.avanade.com/)** | 亚洲、欧洲、北美、南美 |
| **[Bright Skies GmbH](https://bskies.io/expressroute)** | 欧洲
| **[Ensyst](https://www.ensyst.com.au)** | 亚洲
| **[Equinix Professional Services](https://www.equinix.com/services/consulting/)** | 北美 |
| **[FlexManage](https://www.flexmanage.com/cloud)** | 北美 |
| **[Lightstream](https://www.lightstream.tech/partners/microsoft-azure/)** | 北美 |
| **[The IT Consultancy Group](https://itconsult.com.au/)** | 澳大利亚 |
| **[MOQdigital](https://www.moqdigital.com.au/insights/technical/network-connectivity-options-for-azure)** | 澳大利亚 |
| **[MSG Services](https://www.msg-services.de/it-services/managed-services/cloud-outsourcing/)** | 欧洲（德国） |
| **[Nelite](https://www.nelite.com/offres-services/)** | 欧洲 |
| **[New Signature](https://newsignature.com/technologies/express-route/)** | 欧洲 |
| **[OneAs1a](https://www.oneas1a.com/connectivity.html)** | 亚洲 |
| **[Orange Networks](https://orange-networks.com/blog/88-azureexpressroute)** | 欧洲 |
| **[Perficient](https://www.perficient.com/Partners/Microsoft/Cloud/Azure-ExpressRoute)** | 北美 |
| **[Presidio](https://info.presidio.com/microsoft-azure-expressroute)** | 北美 |
| **[sol-tec](https://www.sol-tec.com/services)** | 欧洲 |
| **[Venha Pra Nuvem](https://venhapranuvem.com.br/)** | 南美 |
| **[Vigilant.IT](https://vigilant.it/expressroute)** | 澳大利亚 |

## <a name="next-steps"></a>后续步骤
* 有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](expressroute-faqs.md)。
* 确保符合所有先决条件。 请参阅 [ExpressRoute 先决条件](expressroute-prerequisites.md)。

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "位置地图"
