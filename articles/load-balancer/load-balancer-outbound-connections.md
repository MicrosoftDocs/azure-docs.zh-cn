---
title: Azure 中的出站连接
titlesuffix: Azure Load Balancer
description: 本文介绍 Azure 如何使 VM 与公共 Internet 服务通信。
services: load-balancer
documentationcenter: na
author: KumudD
ms.service: load-balancer
ms.custom: seodec18
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/02/2019
ms.author: kumud
ms.openlocfilehash: d5f52829f5895b30afd160cc8ded755332aca5c5
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190174"
---
# <a name="outbound-connections-in-azure"></a>Azure 中的出站连接

Azure 通过多种不同的机制为客户部署提供出站连接。 本文介绍具体的方案、其应用方式、工作原理以及管理方式。

>[!NOTE] 
>本文仅涵盖了资源管理器部署。 有关 Azure 中的所有经典部署方案，请查看[出站连接（经典）](load-balancer-outbound-connections-classic.md)。

Azure 中的部署可与 Azure 外部的公用 IP 地址空间中的终结点进行通信。 当实例启动到公共 IP 地址空间中的目标的出站流时，Azure 会动态将专用 IP 地址映射到公共 IP 地址。 创建此映射后，此出站发起流的返回流量还可以抵达发起流的专用 IP 地址。

Azure 使用源网络地址转换 (SNAT) 来执行此功能。 当多个专用 IP 地址伪装成单个公共 IP 地址时，Azure 将使用[端口地址转换 (PAT)](#pat) 来伪装专用 IP 地址。 临时端口用于 PAT，是基于池大小[预先分配](#preallocatedports)的。

有多种[出站方案](#scenarios)。 可根据需要结合这些方案。 请认真分析这些方案，以了解在部署模型和应用方案中应用这些方案时的功能、约束和模式。 查看有关[管理这些方案](#snatexhaust)的指导。

>[!IMPORTANT] 
>标准负载均衡器和标准公共 IP 为出站连接引入了新功能和不同的行为。  它们不同于基本 SKU。  如果在使用标准 SKU 时需要出站连接，则必须使用标准公共 IP 地址或标准公共负载均衡器显式定义它。  这包括在使用时创建出站连接和内部标准负载均衡器。  建议始终使用标准公共负载均衡器上的出站规则。  [方案 3](#defaultsnat)不适用于标准 SKU。  这意味着使用内部标准负载均衡器时，如果需要出站连接，则需要采取步骤为后端池中的 VM 创建出站连接。  在出站连接的上下文中，单独的 VM、可用性集中的所有 VM、VMSS 中的所有实例都是一个组。 这意味着，如果可用性集中的单个 VM 与标准 SKU 关联，则该可用性集中的所有 VM 实例现在都遵循相同的规则，就好像这些 VM 实例与标准 SKU 相关联一样，即使单个实例与标准 SKU 没有直接关联。  请仔细查看整个文档以了解整体概念，查看[标准负载均衡器](load-balancer-standard-overview.md)了解 SKU 之间的差异，并查看[出站规则](load-balancer-outbound-rules-overview.md)。  使用出站规则可以对出站连接的所有方面进行细化管理控制。

## <a name="scenarios"></a>方案概述

Azure 负载均衡器和相关资源是使用 [Azure 资源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)时显式定义的。  Azure 目前提供三种不同的方法实现 Azure 资源管理器资源的出站连接。 

| SKU | 场景 | 方法 | IP 协议 | 描述 |
| --- | --- | --- | --- | --- |
| 标准、基本 | [1.具有实例级公共 IP 地址的 VM（有或没有负载均衡器）](#ilpip) | SNAT，不使用端口伪装 | TCP、UDP、ICMP、ESP | Azure 使用分配实例 NIC 的 IP 配置的公共 IP。 此实例具有所有可用的临时端口。 在使用标准负载均衡器时，应使用[出站规则](load-balancer-outbound-rules-overview.md)显式定义出站连接 |
| 标准、基本 | [2.与 VM 关联的公共负载均衡器（实例上没有实例级公共 IP 地址）](#lb) | 使用负载均衡器前端进行端口伪装 (PAT) 的 SNAT | TCP、UDP |Azure 与多个专用 IP 地址共享公共负载均衡器前端的公共 IP 地址。 Azure 使用前端的临时端口进行 PAT。 |
| 无或基本 | [3.独立 VM（无负载均衡器，无实例级公共 IP 地址）](#defaultsnat) | 使用端口伪装 (PAT) 的 SNAT | TCP、UDP | Azure 自动指定用于 SNAT 的公共 IP 地址，与可用性集的多个专用 IP 地址共享此公共 IP 地址，并使用此公共 IP 地址的临时端口。 此方案是前述方案的回退方案。 如果需要可见性和控制，则我们不建议采用。 |

如果不希望 VM 与 Azure 外部的公共 IP 地址空间中的终结点通信，则可以根据需要使用网络安全组 (NSG) 来阻止访问。 [阻止出站连接](#preventoutbound)部分详细介绍了 NSG。 本文不会介绍有关在无任何出站访问权限的情况下，如何设计和管理虚拟网络的设计和实施指导。

### <a name="ilpip"></a>场景 1：具有实例级公共 IP 地址的 VM

在此场景中，向 VM 分配了实例级公共 IP (ILPIP)。 就出站连接而言，VM 是否经过负载均衡并不重要。 此方案优先于其他方案。 使用 ILPIP 时，VM 将 ILPIP 用于所有出站流。  

分配到 VM 的公共 IP 属于 1 对 1 关系（而不是 1 对多关系），并实现为无状态的 1 对 1 NAT。  不使用端口伪装 (PAT)，并且 VM 具有所有可供使用的临时端口。

如果应用程序启动很多出站流，并且遇到 SNAT 端口耗尽的情况，可以考虑分配 [ILPIP 以缓解 SNAT 约束](#assignilpip)。 请查看[管理 SNAT 耗尽](#snatexhaust)。

### <a name="lb"></a>场景 2：无实例级公共 IP 地址的负载均衡 VM

在此方案中，VM 是公共负载均衡器池的一部分。 没有分配给 VM 的公共 IP 地址。 必须为负载均衡器资源配置一个负载均衡器规则，以在公共 IP 前端与后端池之间创建链接。

如果没有完成此规则配置，则行为将如适用于[没有实例级公共 IP 的独立 VM](#defaultsnat) 方案中所述。 不需要在规则中添加后端池的正常运行的侦听器或者运行状况探测就能成功实现。

当负载均衡的 VM 创建出站流时，Azure 将此出站流的专用源 IP 地址转换为公共负载均衡器前端的公共 IP 地址。 Azure 使用 (SNAT) 来执行此功能。 Azure 还使用 [PAT](#pat) 来伪装公共 IP 地址后面的多个专用 IP 地址。 

使用负载均衡器的公共 IP 地址前端的临时端口区分由 VM 产生的各个流。 创建出站流后，SNAT 动态使用[预先分配的临时端口](#preallocatedports)。 在此情况下，用于 SNAT 的临时端口被称为 SNAT 端口。

SNAT 端口是按照[了解 SNAT 和 PAT](#snat) 部分中所述预先分配的。 它们是可能会耗尽的有限资源。 因此了解它们的[使用](#pat)方式很重要。 请查看[管理 SNAT 耗尽](#snatexhaust)，了解如何根据需要进行设计和缓解。

如果[多个公共 IP 地址与负载均衡器基本版相关联](load-balancer-multivip-overview.md)，则所有这些公共 IP 地址都是出站流的候选项，并且会随机选择其中一个。  

若要监视的出站连接负载均衡器基本运行状况，可以使用[的负载均衡器日志的 Azure Monitor](load-balancer-monitor-log.md)并[警报事件日志](load-balancer-monitor-log.md#alert-event-log)要监视 SNAT 端口耗尽消息。

### <a name="defaultsnat"></a>场景 3：无实例级公共 IP 地址的独立 VM

在此场景中，VM 不是公共负载均衡器池的一部分（也不是内部标准负载均衡器池的一部分），并且没有分配给它的 ILPIP 地址。 当 VM 创建出站流时，Azure 将此出站流的专用源 IP 地址转换为公共源 IP 地址。 用于此出站流的公共 IP 地址是不可配置的，并且不会影响订阅的公共 IP 资源限制。 此公共 IP 地址不属于你，不能保留。 如果重新部署 VM、可用性集或虚拟机规模集，则将释放此公共 IP 地址并请求新的公共 IP 地址。 请不要使用此方案将 IP 地址加入允许列表。 而是使用其他两个方案之一，其中你显式声明出站方案和要用于出站连接的公共 IP 地址。

>[!IMPORTANT] 
>仅当附加了内部基本负载均衡器时，此场景才适用。 如果已将内部标准负载均衡器附加到 VM，则场景 3 不适用。  除了使用内部标准负载均衡器以外，还必须显式创建[场景 1](#ilpip) 或[场景 2](#lb)。

Azure 结合端口伪装 ([PAT](#pat)) 使用 SNAT 来执行此功能。 此方案类似于[方案 2](#lb)，但无法控制使用的 IP 地址。 这是方案 1 和方案 2 不存在时的回退方案。 如果需要控制出站地址，则我们不建议使用此方案。 如果出站连接是应用程序的关键部分，应该选择另一种方案。

SNAT 端口是根据[了解 SNAT 和 PAT](#snat) 部分中所述预先分配的。  共享可用性集的 VM 数目决定了适用的预分配层。  没有可用性集的独立 VM 实际上是用于确定预分配（1024 SNAT 端口）的、包含 1 个 VM 的池。 SNAT 端口是可能会被耗尽的有限资源。 因此了解它们的[使用](#pat)方式很重要。 请查看[管理 SNAT 耗尽](#snatexhaust)，了解如何根据需要进行设计和缓解。

### <a name="combinations"></a>多个组合方案

可以结合前面部分中所述的方案来实现特定的效果。 存在多个方案时，优先顺序如下：[方案 1](#ilpip) 优先于[方案 2](#lb)和 [3](#defaultsnat)。 [方案 2](#lb) 优先于[方案 3](#defaultsnat)。

例如，在 Azure 资源管理器部署中，应用程序严重依赖于与有限数量的目标建立出站连接，但也通过负载均衡器前端接收入站流。 在此情况下，可以结合方案 1 和 2 来缓解问题。 有关其他模式，请查看[管理 SNAT 耗尽](#snatexhaust)。

### <a name="multife"></a>对出站流使用多个前端

#### <a name="standard-load-balancer"></a>标准负载均衡器

存在[多个（公共）IP 前端](load-balancer-multivip-overview.md)时，标准负载均衡器同时使用出站流的所有候选项。 如果对出站连接启用了负载均衡规则，则每个前端的可用预分配 SNAT 端口数将会倍增。

可以使用一个新的负载均衡规则选项，来禁止对出站连接使用某个前端 IP 地址：

```json    
      "loadBalancingRules": [
        {
          "disableOutboundSnat": false
        }
      ]
```

通常，`disableOutboundSnat` 选项默认为 _false_，表示此规则将为负载均衡规则的后端池中关联的 VM 编制出站 SNAT。 可将 `disableOutboundSnat` 更改为 _true_，防止负载均衡器对此负载均衡规则的后端池中 VM 的出站连接使用关联的前端 IP 地址。  此外，仍可以根据[多个组合方案](#combinations)中所述，为出站流指定特定的 IP 地址。

#### <a name="load-balancer-basic"></a>负载均衡器基本版

当[多个（公共）IP 前端](load-balancer-multivip-overview.md)适用于出站流时，负载均衡器基本版将选择单个前端用于出站流。 此项选择不可配置，应将选择算法视为随机。 可以根据[多个组合方案](#combinations)中所述，为出站流指定特定的 IP 地址。

### <a name="az"></a>可用性区域

结合使用[标准负载均衡器和可用性区域](load-balancer-standard-availability-zones.md)时，区域冗余的前端可以提供区域冗余的出站 SNAT 连接，发生区域故障时，SNAT 编程可以幸存。  使用区域性前端时，出站 SNAT 连接的生存状况与它们所属的区域相同。

## <a name="snat"></a>了解 SNAT 和 PAT

### <a name="pat"></a>端口伪装 SNAT (PAT)

公共负载均衡器资源与 VM 实例相关联时，将重写每个出站连接源。 出站连接源从虚拟网络专用 IP 地址空间重新写入负载均衡器的前端公共 IP 地址。 在公共 IP 地址空间中，流的 5 元组（源 IP 地址、源端口、IP 转换协议、目标 IP 地址、目标端口）必须唯一。  端口伪装 SNAT 可与 TCP 或 UDP IP 协议一起使用。

重写专用源 IP 地址后，临时端口（SNAT 端口）用于实现此目的，因为多个流源自单个公共 IP 地址。 伪装 SNAT 算法的端口为 UDP 与 TCP 分配不同的 SNAT 端口。

#### <a name="tcp"></a>TCP SNAT 端口

每个到单个目标 IP 地址、端口的流使用一个 SNAT 端口。 对于到相同的目标 IP 地址、端口和协议的多个 TCP 流，每个 TCP 流使用一个 SNAT 端口。 这可以确保源自相同的公共 IP 地址，并到相同的目标 IP 地址、端口和协议的流的唯一性。 

每个流均流到不同目标 IP 地址、端口和协议的多个流共用一个 SNAT 端口。 目标 IP 地址、端口和协议使流保持唯一，无需使用其他源端口来区分公共 IP 地址空间中的流。

#### <a name="udp"></a> UDP SNAT 端口

UDP SNAT 端口由与 TCP SNAT 端口不同的算法管理。  负载均衡器对 UDP 使用称为“端口受限锥形 NAT”的算法。  无论目标 IP 地址、端口如何，每个流都会使用一个 SNAT 端口。

#### <a name="exhaustion"></a>耗尽

如果 SNAT 端口资源已经耗尽，那么在现有流释放 SNAT 端口之前出站流会失败。 当流关闭时，负载均衡器将回收 SNAT 端口，并使用 [4 分钟空闲超时](#idletimeout)回收空闲流中的 SNAT 端口。

由于使用的算法不同，UDP SNAT端口的耗尽速度通常比 TCP SNAT 端口快得多。 在进行设计和规模测试时必须考虑到这种差异。

有关可以使用哪些模式来缓解通常导致 SNAT 端口耗尽的状态，请查看[管理 SNAT](#snatexhaust) 部分。

### <a name="preallocatedports"></a>端口伪装 SNAT (PAT) 的临时端口预先分配

使用端口伪装 SNAT ([PAT](#pat)) 时，Azure 使用某种算法根据后端池的大小来确定可用的预先分配 SNAT 端口数目。 SNAT 端口是可用于特定公共 IP 源地址的临时端口。

将分别为 UDP 和 TCP 预分配相同数量的 SNAT 端口，并根据 IP 传输协议独立地使用这些端口。  但是，SNAT 端口使用情况会因流是 UDP 还是 TCP 而有所不同。

>[!IMPORTANT]
>标准 SKU SNAT 编程依据 IP 传输协议并且派生自负载均衡规则。  如果只存在一个 TCP 负载均衡规则，则 SNAT 仅可用于 TCP。 如果只有一个 TCP 负载均衡规则并且 UDP 需要出站 SNAT，请创建从同一个前端到同一个后端池的 UDP 负载均衡规则。  这将触发针对 UDP 的 SNAT 编程。  不需要采用工作规则或运行状况探测。  无论在负载均衡规则中指定了什么传输协议，基本 SKU SNAT 都始终针对 IP 传输协议编写 SNAT 程序。

Azure 向每个 VM 的 NIC IP 配置预先分配 SNAT 端口。 将 IP 配置添加到池后，将会根据后端池的大小预先分配此 IP 配置的 SNAT 端口。 创建出站流后，当流关闭或[空闲超时](#idletimeout)时，[PAT](#pat) 动态使用（不超过预先分配的限制）和释放这些端口。

下表显示了针对后端池大小层的 SNAT 端口预分配：

| 池大小（VM 实例） | 每个 IP 配置的预先分配 SNAT 端口|
| --- | --- |
| 1-50 | 1,024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1,000 | 32 |

>[!NOTE]
> 结合[多个前端](load-balancer-multivip-overview.md)使用标准负载均衡器时，上表中每个前端 IP 地址的可用 SNAT 端口数目将会倍增。 例如，如果某个后端池包含 50 个 VM 和 2 个负载均衡规则，并且每个 VM 使用独立的前端 IP 地址，则该后端池将为每个 IP 配置使用 2048 个 (2 x 1024) SNAT 端口。 参阅有关[多个前端](#multife)的详细信息。

请记住，可用的 SNAT 端口数不会直接转换为流数。 可以针对多个唯一目标重用单个 SNAT 端口。 仅当需要使流保持唯一时，才使用端口。 有关设计和缓解指导，请参阅[如何管理这项可耗尽的资源](#snatexhaust)；另请参阅介绍 [PAT](#pat) 的部分。

更改后端池大小可能会影响建立的某些流。 如果后端池大小递增并转换为下一层，则在转换为下一个更大的后端池层期间，一半的预先分配 SNAT 端口将被回收。 与回收的 SNAT 端口关联的流会超时，必须重新建立连接。 如果尝试新流，则只要预先分配的端口可用，则该流就能立即成功。

如果后端池减小并转换到更低层级，可用的 SNAT 端口数会增多。 在这种情况下，现有的分配 SNAT 端口及其相应的流不会受到影响。

SNAT 端口分配特定于 IP 传输协议（TCP 和 UDP 是分别维护的），并在以下条件下释放：

### <a name="tcp-snat-port-release"></a>TCP SNAT 端口释放

- 如果任一服务器/客户端发送 FINACK，SNAT 端口将 240 秒后释放。
- 如果出现 RST，则 SNAT 端口在 15 秒后释放。
- 如果已达到空闲超时，则释放端口。

### <a name="udp-snat-port-release"></a>UDP SNAT 端口释放

- 如果已达到空闲超时，则释放端口。

## <a name="problemsolving"></a>解决问题 

本部分旨在帮助解决 SNAT 耗尽的问题，以及 Azure 中的出站连接可能出现的其他情况。

### <a name="snatexhaust"></a>应对 SNAT (PAT) 端口耗尽问题
用于 [PAT](#pat) 的[临时端口](#preallocatedports)是可用尽的资源，如[无实例级公共 IP 地址的独立 VM](#defaultsnat) 和[无实例级公共 IP 地址的负载均衡 VM](#lb) 中所述。

如果知道正在启动与同一目标 IP 地址和端口的多个出站 TCP 或 UDP 连接，观察失败的出站连接，或者支持人员通知已耗尽 SNAT 端口（[PAT](#pat) 使用的预先分配[临时端口](#preallocatedports)），则有几个常见缓解选项可供选择。 查看这些选项，确定可用且最适合自己的方案的选项。 一个或多个选项可能有助于管理此方案。

如果难以理解出站连接行为，可以使用 IP 堆栈统计 (netstat)。 或者使用数据包捕获来观察连接行为。 可以在实例的来宾 OS 中执行这些数据包捕获，或使用[网络观察程序来捕获数据包](../network-watcher/network-watcher-packet-capture-manage-portal.md)。

#### <a name="connectionreuse"></a>修改应用程序以重复使用连接 
在应用程序中重复使用连接，可以降低对用于 SNAT 的临时端口的需求。 这尤其适用于 HTTP/1.1 这样的协议，在这些协议中，默认重复使用连接。 其他使用 HTTP 作为其传输（如 REST）的协议也可以因此受益。 

对每个请求来说，重复使用总是优于单独的原子 TCP 连接。 重复使用可以带来更高的性能和非常高效的 TCP 事务。

#### <a name="connection pooling"></a>修改应用程序以使用连接池
可以在应用程序中使用连接池方案，其中在一组固定的连接（在可能的情况下，重复使用每一个）上，内部分布请求。 此方案会限制正在使用的临时端口的数量，并创建更加可预测的环境。 此方案还可以通过在操作答复上单个连接阻塞时允许多个同时操作，增加请求的吞吐量。  

连接池可能已经存在于正在使用的框架中，以开发应用程序或应用程序的配置设置。 可将连接池与连接重复使用相结合。 多个请求使用指向相同目标 IP 地址和端口的可预测固定数量的端口。 请求还可以从 TCP 事务的高效使用中受益，从而减少延迟和资源利用。 UDP 事务也能从中受益，因为管理 UDP 流数又能避免耗尽情况和管理 SNAT 端口利用率。

#### <a name="retry logic"></a>修改应用程序以使用主动性较低的重试逻辑
当用于 [PAT](#pat) 的[预先分配临时端口](#preallocatedports)耗尽或应用程序故障发生时，无衰减或回退逻辑的积极重试或暴力重试会使耗尽状况再次发生或一直持续。 使用主动性较低的重试逻辑，可以降低对临时端口的需求。 

临时端口有 4 分钟的空闲超时（不可调整）。 如果重试太过积极，则消耗没有机会进行自行清除。 因此，应用程序停用事务的方式和频率对于设计至关重要。

#### <a name="assignilpip"></a>将实例级公共 IP 分配给每个 VM
分配 ILPIP 会将方案更改为 [VM 的实例级公共 IP](#ilpip)。 用于各 VM 的公共 IP 的所有临时端口都可供 VM 使用。 （与以下方案相反：公共 IP 的临时端口与同相应后端池关联的 VM 的所有临时端口共享）。需要作出一些权衡，比如公共 IP 地址的额外成本和将大量个人 IP 地址列入白名单所产生的潜在影响。

>[!NOTE] 
>此选项不适用于 Web 辅助角色。

#### <a name="multifesnat"></a>使用多个前端

使用公共标准负载均衡器时，可[为出站连接分配多个前端 IP 地址](#multife)，并[将可用的 SNAT 端口数倍增](#preallocatedports)。  创建前端 IP 配置、规则和后端池，以触发将 SNAT 编程到前端公共 IP。  该规则不需要运行，并且运行状况探测不需要成功。  如果对入站连接（而不仅仅是出站连接）也使用多个前端，应使用自定义的运行状况探测来确保可靠性。

>[!NOTE]
>在大多数情况下，SNAT 端口耗尽是设计不当的征兆。  在使用更多前端来添加 SNAT 端口之前，请确保已了解端口耗尽的原因。  否则可能会忽视问题，导致以后出现故障。

#### <a name="scaleout"></a>横向扩展

[预分配端口](#preallocatedports)是根据后端池大小分配的并且分组到各个层中，这样，当某些端口必须重新分配以适应后端池更大的下一个层时，可以最大限度地减少中断。  有可能可以通过将后端池扩展到给定层的最大大小来提高给定前端的 SNAT 端口利用率。  这需要应用程序高效地进行横向扩展。

例如，后端池中的两个虚拟机对于每个 IP 配置将有 1024 个 SNAT 端口可用，整个部署总共有 2048 个 SNAT 端口。  如果部署已增加到 50 台虚拟机，虽然每台虚拟机的预分配端口数保持不变，但整个部署可以使用 51,200 (50 x 1024) 个 SNAT 端口。  如果希望横向扩展部署，请检查每层的[预分配端口](#preallocatedports)的数量以确保将横向扩展规划为各自层的最大容量。  在上述示例中，如果选择了横向扩展到 51 个而非 50 个实例，则你将提升到下一个层，最终，每台 VM 的 SNAT 端口数以及端口总数会更少。

如果横向扩展到后端池较大的下一层，并且需要重新分配已分配端口，则部分出站连接可能会超时。  如果仅使用部分 SNAT 端口，则在后端池较大的下一层中横向扩展无意义。  每次移动到后端池的下一层时，半数现有端口将重新分配。  如果不希望发生此行为，则需要将部署规划为层大小。  或者确保应用程序可以根据需要进行检测和重试。  TCP keepalive 可以帮助检测 SNAT 端口何时由于被重新分配而不再工作。

### <a name="idletimeout"></a>保持 keepalive 重置出站空闲超时

出站连接有 4 分钟的空闲超时。 此超时不可调整。 但是，可以根据需要使用传输（例如 TCP keepalive）或应用层 keepalive 来刷新空闲流和重置此空闲超时。  

使用 TCP keepalive 时，在连接的一端启用它们就足够了。 例如，若要重置流的空闲计时器，在服务器端启用它们就足够了，没有必要在两端都启动 TCP keepalive。  应用程序层（包括数据库客户端-服务器配置）也存在类似的概念。  检查服务器端对于特定于应用程序的 keepalive 存在哪些选项。

## <a name="discoveroutbound"></a>发现 VM 使用的公共 IP
有多种方法来确定出站连接的公共源 IP 地址。 OpenDNS 提供了一种服务可以向你显示 VM 的公共 IP 地址。 

使用 nslookup 命令，可以将名称 myip.opendns.com 的 DNS 查询发送到 OpenDNS 解析程序。 该服务返回用于发送此查询的源 IP 地址。 在 VM 中运行以下查询时，返回的是用于该 VM 的公共 IP。

    nslookup myip.opendns.com resolver1.opendns.com

## <a name="preventoutbound"></a>阻止出站连接
有时允许 VM 创建出站流是不可取的。 或者，可能需要管理哪些目标可以通过出站流访问或哪些目标可以启动入站流。 在此情况下，可以使用[网络安全组](../virtual-network/security-overview.md)管理 VM 可访问的目标。 还可以使用 NSG 来管理可启动入站流的公共目标。

将 NSG 应用于负载均衡的 VM 时，需要注意[服务标记](../virtual-network/security-overview.md#service-tags)和[默认安全规则](../virtual-network/security-overview.md#default-security-rules)。 必须确保 VM 可以接收来自 Azure 负载均衡器的运行状况探测请求。 

如果 NSG 阻止来自 AZURE_LOADBALANCER 默认标记的运行状况探测请求，那么 VM 的运行状况探测程序会失败，并且 VM 被标记为停机。 负载均衡器停止向此 VM 发送新流。

## <a name="limitations"></a>限制
- 在门户中配置负载均衡规则时，不能将 DisableOutboundSnat 用作选项。  请改用 REST、模板或客户端工具。
- 由于 VNet 之前的服务和其他平台服务功能的副作用，如果仅使用内部标准负载均衡器，则可以访问没有 VNet 和其他 Microsoft 平台服务的辅助角色。 请勿依赖此副作用，因为相应的服务本身或底层平台可能会在不通知的情况下进行更改。 在仅使用内部标准负载均衡器时，必须始终假定需要明确创建出站连接。 本文中所述的[默认 SNAT](#defaultsnat) 方案 3 不可用。

## <a name="next-steps"></a>后续步骤

- 详细了解[标准负载均衡器](load-balancer-standard-overview.md)。
- 详细了解标准公共负载均衡器的[出站规则](load-balancer-outbound-rules-overview.md)。
- 详细了解[负载均衡器](load-balancer-overview.md)。
- 详细了解[网络安全组](../virtual-network/security-overview.md)。
- 了解 Azure 的部分其他关键[网络功能](../networking/networking-overview.md)。
