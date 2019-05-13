---
title: Azure 中的负载均衡器空闲情况下的 TCP 重置
titlesuffix: Azure Load Balancer
description: 具有空闲超时情况下发送双向 TCP RST 数据包功能的负载均衡器
services: load-balancer
documentationcenter: na
author: KumudD
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/03/2019
ms.author: kumud
ms.openlocfilehash: 4a09492fcb8a7985fa27b6daae89aa5dec0fa6e0
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413862"
---
# <a name="load-balancer-with-tcp-reset-on-idle-public-preview"></a>负载均衡器空闲情况下的 TCP 重置（公共预览版）

可以使用[标准负载均衡器](load-balancer-standard-overview.md)，通过为给定规则启用“空闲时执行 TCP 重置”，为方案创建可预测度更高的应用程序行为。 负载均衡器的默认行为是当达到流的空闲超时的情况下，以静默方式删除流。  启用此功能将导致负载均衡器在空闲超时情况下发送双向 TCP 重置（TCP RST 包）。  这将通知应用程序终结点，连接已超时且不再可用。  终结点可以视需要立即建立新连接。

![负载均衡器 TCP 重置](media/load-balancer-tcp-reset/load-balancer-tcp-reset.png)

>[!NOTE] 
>负载平衡器 TCP 空闲超时功能上重置具有这一次现推出公共预览版。 此预览版在提供时没有附带服务级别协议，不建议用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
 
你可以更改此默认行为，并启用根据入站 NAT 规则、负载均衡规则和[出站规则](https://aka.ms/lboutboundrules)在空闲超时情况下发送 TCP 重置。  根据规则启用时，负载均衡器将在所有匹配流的空闲超时情况下向客户端和服务器终结点发送双向 TCP 重置（TCP RST 数据包）。

接收 TCP RST 数据包的终结点会立即关闭相应的套接字。 这会立即通知终结点该连接已释放，并且同一 TCP 连接上的任何后续通信都将失败。  应用程序可以在套接字关闭时清除连接，并根据需要重新建立连接，而无需等待 TCP 连接最终超时。

对于许多方案，这样可以减少发送 TCP（或应用层）保持连接以刷新流空闲超时的需要。 

如果空闲持续时间超过配置所允许的持续时间，或者你的应用程序显示启用了 TCP 重置的不良行为，则可能仍需要使用 TCP 保持连接（或应用层保持连接）来监视 TCP 连接的活跃性。  此外，当路径中某处的连接已经过代理时，保持连接（特别是应用层保持连接）也仍然有用。  

请仔细检查整个端到端方案，确定能否从启用 TCP 重置、调整空闲超时中受益，以及是否需要执行其他步骤来确保所需的应用程序行为。

## <a name="enabling-tcp-reset-on-idle-timeout"></a>启用空闲超时情况下的 TCP 重置

使用 API 版本 2018-07-01，可以在每个规则的基础上启用在空闲超时情况下发送双向 TCP 重置：

```json
      "loadBalancingRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

```json
      "inboundNatRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

```json
      "outboundRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

## <a name="regions"></a>区域可用性

在所有区域中可用。

## <a name="limitations"></a>限制

- 不能使用门户来配置或查看 TCP 重置。  请改为使用模板、REST API、Az CLI 2.0 或 PowerShell。
- 在已建立状态的 TCP 连接期间才发送 TCP RST。

## <a name="next-steps"></a>后续步骤

- 了解[标准负载均衡器](load-balancer-standard-overview.md)。
- 了解[出站规则](load-balancer-outbound-rules-overview.md)。
