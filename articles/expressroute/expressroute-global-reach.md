---
title: 使用 Global Reach 将本地网络连接到 Microsoft 云 - Azure ExpressRoute |Microsoft Docs
description: 本文介绍 ExpressRoute Global Reach。
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: cherylmc
ms.custom: seodec18
ms.openlocfilehash: 28cfbc7183628b00c463f1c3a8a954ba4074409b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415962"
---
# <a name="expressroute-global-reach"></a>ExpressRoute 的全球性覆盖
ExpressRoute 是用来将本地网络连接到 Microsoft 云的一种专用且可复原的方式。 可以从专用数据中心或公司网络访问许多 Microsoft 云服务，例如 Azure、Office 365 和 Dynamics 365。 例如，你可能在旧金山有一家分公司，其 ExpressRoute 线路位于硅谷，在伦敦有另一家分公司，其 ExpressRoute 线路位于同一城市。 这两个分公司都可以与美国西部和英国南部的 Azure 资源高速连接。 但是，分公司不能直接相互交换数据。 换句话说，10.0.1.0/24 可以将数据发送到 10.0.3.0/24 和 10.0.4.0/24，但不能发送到 10.0.2.0/24。

![不带时][1]

使用 **ExpressRoute Global Reach**，可以将 ExpressRoute 线路链接到一起，以在本地网络之间建立专用网络。 在上面的示例中，通过添加 ExpressRoute Global Reach，你的旧金山分公司 (10.0.1.0/24) 便可以通过现有的 ExpressRoute 线路和 Microsoft 的全球网络直接与你的伦敦分公司 (10.0.2.0/24) 交换数据。 

![带有时][2]

## <a name="use-case"></a>使用案例
ExpressRoute Global Reach 旨在补充服务提供商的 WAN 实施，并连接你在世界各地的分公司。 例如，如果你的服务提供商主要在美国运营并且已连接你在美国的所有分公司，但服务提供商不在日本和香港运营，则可以使用 ExpressRoute Global Reach 与本地服务提供商合作，Microsoft 将使用 ExpressRoute 和我们的全球网络将你在那里的分公司连接到美国的分公司。

![用例][3]

## <a name="availability"></a>可用性 
目前，以下地方支持 ExpressRoute Global Reach。

* 澳大利亚
* 加拿大
* 法国
* 德国
* 香港特别行政区
* 爱尔兰
* 日本
* 韩国
* 荷兰
* 新加坡
* 瑞士
* 英国
* 美国

在以上国家/地区中，必须在 [ExpressRoute 对等互连位置](expressroute-locations.md)创建 ExpressRoute 线路。 若要在[不同的地缘政治区域](expressroute-locations.md)之间启用 ExpressRoute Global Reach，你的线路必须是高级 SKU。

## <a name="next-steps"></a>后续步骤
1. [详细了解 ExpressRoute Global Reach](expressroute-faqs.md)
2. [如何启用 ExpressRoute Global Reach](expressroute-howto-set-global-reach.md)
3. [将 ExpressRoute 线路链接到 Azure 虚拟网络](expressroute-howto-linkvnet-arm.md)


<!--Image References-->
[1]: ./media/expressroute-global-reach/1.png "不带 Global Reach 时的示意图"
[2]: ./media/expressroute-global-reach/2.png "带有 Global Reach 时的示意图"
[3]: ./media/expressroute-global-reach/3.png "Global Reach 的用例"
