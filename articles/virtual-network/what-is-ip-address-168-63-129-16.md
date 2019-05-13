---
title: IP 地址 168.63.129.16 是什么？ | Microsoft Docs
description: 了解 IP 地址 168.63.129.16 以及它如何与资源一起工作。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: v-jesits
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2019
ms.author: genli
ms.openlocfilehash: 78d2392e32465b3091c49032dc5df5f3a5b6061a
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416036"
---
# <a name="what-is-ip-address-1686312916"></a>IP 地址 168.63.129.16 是什么？

IP 地址 168.63.129.16 是虚拟公共 IP 地址，用于简化 Azure 平台资源的通信通道。 客户可以在 Azure 中为其专有虚拟网络定义任何地址空间。 因此，Azure 平台资源必须显示为一个唯一的公共 IP 地址。 此虚拟公共 IP 地址有助于实现以下几个方面：

- 使 VM 代理能够与 Azure 平台通信，以表明它处于“就绪”状态。
- 启用与 DNS 虚拟服务器的通信，以便为没有自定义 DNS 服务器的资源（如 VM）提供筛选的名称解析。 此筛选确保客户只能解析其自己资源的主机名。
- 使[从 Azure 负载均衡器运行状况探测](../load-balancer/load-balancer-custom-probe-overview.md)来确定 Vm 的运行状况状态。
- 使 VM 能够从 Azure 中的 DHCP 服务获取动态 IP 地址。
- 为 PaaS 角色启用来宾代理检测信号消息。

## <a name="scope-of-ip-address-1686312916"></a>IP 地址 168.63.129.16 的作用域

公共 IP 地址 168.63.129.16 用于所有区域和国家/地区的所有云。 此特殊的公共 IP 地址由 Microsoft 拥有，并且将不会更改。 默认网络安全组规则允许此 IP 地址。 建议在所有本地防火墙策略中允许此 IP 地址。 此特殊 IP 地址和资源之间的通信是安全的，因为只有内部 Azure 平台才能从此 IP 地址获得消息。 如果阻止此地址，可能会在各种场景中出现意外行为。

[Azure 负载均衡器运行状况探测](../load-balancer/load-balancer-custom-probe-overview.md)源自此 IP 地址。 如果阻止此 IP 地址，则将探测会失败。

在非虚拟网络方案中，运行状况探测源自某个专用 IP 和未使用 168.63.129.16。

## <a name="next-steps"></a>后续步骤

- [安全组](security-overview.md)
- [创建、更改或删除网络安全组](manage-network-security-group.md)
