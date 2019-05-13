---
title: Azure Front Door Service | Microsoft Docs
description: 本文提供 Azure Front Door 的概述。 了解它是否是针对应用程序进行用户流量负载均衡的正确选择。
services: frontdoor
documentationcenter: ''
author: sharad4u
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/29/2018
ms.author: sharadag
ms.openlocfilehash: 2d4c93b7ef707f5b09bc2f5f93d56c50c1bb3458
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60736320"
---
# <a name="what-is-azure-front-door-service"></a>什么是 Azure Front Door 服务？
在 Azure Front Door 服务中可以进行优化以实现最佳性能以及进行即时全球故障转移以实现高可用性，并以此定义、管理和监视 Web 流量的全局路由。 使用 Front Door，可将全球（多区域）消费型和企业应用程序转化成可靠、高性能、个性化的现代应用程序、API 和内容，供 Azure 全球受众访问。

Front Door 在第 7 层或 HTTP/HTTPS 层工作，使用任意广播协议和拆分 TCP 与 Microsoft 全球网络来改善全球连接。 因此，根据配置中选择的路由方法，可以确保 Front Door 将客户端请求路由到最快且可用性最高的应用程序后端。 应用程序后端是托管在 Azure 内部或外部的任何面向 Internet 的服务。 Front Door 提供多种[流量路由方法](front-door-routing-methods.md)和[后端运行状况监视选项](front-door-health-probes.md)来满足不同的应用程序需求和自动故障转移模型。 类似于[流量管理器](../traffic-manager/traffic-manager-overview.md)，Front Door 能够灵活应对故障，包括整个 Azure 区域的故障。

>[!NOTE]
> Azure 为方案提供了一套完全托管的负载均衡解决方案。 如果你在寻求基于 DNS 的全球路由，且在传输层安全性 (TLS) 协议终止（“SSL 卸载”）、HTTP/HTTPS 请求或应用程序层处理方面**没有**要求，请查看[流量管理器](../traffic-manager/traffic-manager-overview.md)。 如果你寻求在某个区域中的服务器之间实现负载均衡，在应用程序层方面，请查看[应用程序网关](../application-gateway/application-gateway-introduction.md)，在网络层负载均衡方面，请查看[负载均衡器](../load-balancer/load-balancer-overview.md)。 端到端场景可从结合所需的解决方案中受益。

Front Door 附带以下功能：

## <a name="accelerate-application-performance"></a>加速应用程序性能
Front Door 使用基于拆分 TCP 的任意广播协议确保最终用户能够立即连接到最靠近的 Front Door POP（接入点）。 使用 Microsoft 全球网络从 Front Door 连接到应用程序后端可在保持性能的同时，确保更高的可用性和可靠性。 这种后端连接方式也立足于最低的网络延迟。 详细了解[拆分 TCP](front-door-routing-architecture.md#splittcp) 和[任意广播协议](front-door-routing-architecture.md#anycast)等 Front Door 路由方法。

## <a name="increase-application-availability-with-smart-health-probes"></a>使用智能运行状况探测提高应用程序可用性

Front Door 使用其智能运行状况探测为关键应用程序提供高可用性，监视后端的延迟和可用性，并在后端出现故障时提供即时自动故障转移。 因此，无需停机即可针对应用程序运行计划内的维护操作。 在维护过程中，Front Door 会将流量定向到备用的后端。

## <a name="url-based-routing"></a>基于 URL 的路由
基于 URL 路径的路由用于根据请求的 URL 路径，将流量路由到后端池。 方案之一是将针对不同内容类型的请求路由到不同的后端池。

例如，`http://www.contoso.com/users/*` 的请求路由到 UserProfilePool，`http://www.contoso.com/products/*` 路由到 ProductInventoryPool。  Front Door 使用最佳匹配算法，允许更复杂的路由匹配方案，因此，如果没有任何路径模式匹配，则会选择 `http://www.contoso.com/*` 的默认路由规则，并将流量定向到默认全方位路由规则。 在[路由匹配](front-door-route-matching.md)中了解详细信息。

## <a name="multiple-site-hosting"></a>多站点托管
使用多站点托管可以在同一 Front Door 配置中配置多个网站。 使用此功能可将不同的网站添加到单个 Front Door 配置，以便为部署配置更有效的拓扑。 根据应用程序的体系结构，可将 Azure Front Door 服务配置为向其自身的后端池定向每个网站，或者将不同的网站定向到同一个后端池。 例如，Front Door 可为名为 ImagePool 和 VideoPool 的两个后端池中 `images.contoso.com` 和 `videos.contoso.com` 的流量提供服务。 或者，可将两个前端主机配置为向名为 MediaPool 单个后端池定向流量。

同样，可以在同一个 Front Door 上配置两个不同的域 `www.contoso.com` 和 `www.fabrikam.com`。

## <a name="session-affinity"></a>会话相关性
需要在同一应用程序后端上保留用户会话时，基于 Cookie 的会话关联功能非常有用。 使用 Front Door 托管 Cookie 时，来自用户会话的后续流量将定向到同一应用程序后端进行处理。 在会话状态是为用户会话而本地保存在后端的情况下，此功能十分重要。

## <a name="secure-sockets-layer-ssl-termination"></a>安全套接字层 (SSL) 终止
Front Door 支持边缘上的 SSL 终端，即，各个用户可以直接与 Front Door 环境建立 SSL 连接，而无需与应用程序后端建立远距离的连接。 此外，Front Door 支持在 Front Door 环境与后端之间建立 HTTP 和 HTTPS 连接。 因此，还可以设置端到端的 SSL 加密。 例如，如果应用程序工作负荷的 Front Door 在一分钟内收到 5000 多个请求，则由于热连接的重复使用，对于活动的服务，Front Door 只会与应用程序后端建立大约 500 个连接，因此能够大大减少来自后端的负载。

## <a name="custom-domains-and-certificate-management"></a>自定义域和证书管理
使用 Front Door 交付内容时，如果希望自己的域名在 Front Door URL 中可见，则必须使用自定义域。 使用可见的域名可以方便客户，适用于推广品牌。
Front Door 支持对自定义域名使用 HTTPS。 可以通过为流量选择 Front Door 托管证书或者上传自己的自定义 SSL 证书来使用此功能。

## <a name="application-layer-security"></a>应用程序层安全性
Azure Front Door 允许编写自定义 Web 应用程序防火墙 (WAF) 规则进行访问控制，以基于客户端 IP 地址、国家/地区代码和 http 参数来防范 HTTP/HTTPS 工作负荷遭到恶意利用。 此外，Front Door 还允许创建速率限制规则来阻止恶意机器人流量。 

Front Door 平台本身受 [Azure DDoS 防护](../virtual-network/ddos-protection-overview.md)基本版的保护。 若要进一步提供保护，可在 VNET 中启用 Azure DDoS 防护标准版，并通过自动优化和缓解措施来防范资源遭到网络层 (TCP/UDP) 攻击。 Front Door 是第 7 层反向代理，它仅允许 Web 流量通过后端，默认会阻止其他类型的流量。

## <a name="url-rewrite"></a>URL 重写
Front Door 支持 [URL 重写](front-door-url-rewrite.md)，允许配置可选的自定义转发路径，以便在构建要转发到后端的请求时使用。 此外，Front Door 还允许配置在将请求转发到后端时要发送的主机标头。

## <a name="protocol-support---ipv6-and-http2-traffic"></a>协议支持 - IPv6 和 HTTP/2 流量
Azure Front Door 原生支持端到端 IPv6 连接以及 HTTP/2 协议。 

HTTP/2 协议通过长时间运行的 TCP 连接，在应用程序后端与客户端之间实现全双工通信。 HTTP/2 允许在后端与客户端之间进行交互性更强的通信。这种通信可以是双向的，而且不像基于 HTTP 的实现那样需要轮询。 不同于 HTTP，HTTP/2 协议的开销很低，并且可以对多个请求或响应重复使用同一 TCP 连接，进而提高资源利用率。 详细了解 [Azure Front Door 服务中的 HTTP/2 支持](front-door-http2.md)。

## <a name="pricing"></a>定价

有关定价信息，请参阅 [Front Door 定价](https://azure.microsoft.com/pricing/details/frontdoor/)。

## <a name="next-steps"></a>后续步骤

- 了解如何[创建 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的工作原理](front-door-routing-architecture.md)。
