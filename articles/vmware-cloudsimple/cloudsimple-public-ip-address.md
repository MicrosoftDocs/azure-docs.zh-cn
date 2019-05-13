---
title: VMware 解决方案 CloudSimple-Azure 公共 IP 地址
description: 了解公共 IP 地址上通过 CloudSimple VMware 解决方案及其优势
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: f57b7397f4a2d288cd2b8b55cf23b2d635aa5f8c
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209565"
---
# <a name="cloudsimple-public-ip-address-overview"></a>CloudSimple 公共 IP 地址概述

公共 IP 地址允许 internet 资源的通信入站，私有云资源的专用 IP 地址。 专用 IP 地址是虚拟机或软件负载均衡器。 在私有云 vCenter 的专用 IP 地址。 公共 IP 地址，可公开到 internet 在私有云上运行的服务。

公共 IP 地址专门用于专用 IP 地址，直到取消该分配。 公共 IP 地址只能分配到一个专用 IP 地址。

始终与公共 IP 地址相关联的资源使用的公共 IP 地址进行 internet 访问。 默认情况下，仅使用出站 internet 访问允许公共 IP 地址。  上的公共 IP 地址的传入流量将被拒绝。  若要允许入站的流量，请创建的特定端口的公共 IP 地址的防火墙规则。

## <a name="benefits"></a>优点

使用公共 IP 地址进行入站通信提供了：

* 分布式的拒绝服务 (DDoS) 攻击防护。 这种保护会自动启用的公共 IP 地址。
* 始终可用的流量监控和实时风险缓解常见网络级别攻击。 这些防御措施都由 Microsoft 在线服务使用的相同防御。
* 整个 Azure 全球网络的规模。 在网络可用来分散和缓解跨区域攻击流量。  

## <a name="next-steps"></a>后续步骤

* 了解如何[分配公共 IP 地址](https://docs.azure.cloudsimple.com/public-ips/)
