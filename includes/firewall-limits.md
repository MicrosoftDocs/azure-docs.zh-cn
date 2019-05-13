---
title: include 文件
description: include 文件
services: firewall
author: vhorne
ms.service: firewall
ms.topic: include
ms.date: 5/3/2019
ms.author: victorh
ms.custom: include file
ms.openlocfilehash: 8709d4d903bd31ff94d04ec61e226857e4190407
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150305"
---
| 资源 | 默认限制 |
| --- | --- |
| 数据吞吐量 |30 Gbps<sup>1</sup> |
|规则|10000，所有规则类型组合。|
|最小 AzureFirewallSubnet 大小 |/26|
|网络和应用程序规则的端口范围|0-64,000。 目前正在努力放宽此限制。|
|路由表|默认情况下，AzureFirewallSubnet 0.0.0.0/0 路由的 NextHopType 值设置为**Internet**。<br><br>Azure 防火墙必须具有直接 Internet 连接。 如果你 AzureFirewallSubnet 了解到通过 BGP 将本地网络的默认路由，则必须使用 0.0.0.0/0 UDR 用于覆盖这**NextHopType**值设置为**Internet**维护直接Internet 连接。 默认情况下，Azure 防火墙不支持强制隧道连接到本地网络。<br><br>但是，如果你的配置要求强制隧道连接到本地网络，Microsoft 将支持其上具体情况。 联系支持人员，以便我们可以查看您的案例。 如果接受，我们将你的订阅的白名单，并确保维持所需的防火墙的 Internet 连接。|

<sup>1</sup>如果需要增加这些限制，请联系 Azure 支持部门。
