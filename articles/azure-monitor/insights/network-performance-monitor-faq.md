---
title: 常见问题解答 - Azure 中的网络性能监视器解决方案 | Microsoft Docs
description: 这篇文章捕获有关在 Azure 中的网络性能监视器的常见问题。 网络性能监视器 (NPM) 可帮助你监视网络以近乎实时的性能和检测并找到网络性能瓶颈。
services: log-analytics
documentationcenter: ''
author: vinynigam
manager: agummadi
editor: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/12/2018
ms.author: vinynigam
ms.openlocfilehash: d573b7ad9edac6b1502744b61e85cba3402a6f68
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232670"
---
# <a name="network-performance-monitor-solution-faq"></a>网络性能监视器解决方案常见问题解答

![网络性能监视器符号](media/network-performance-monitor-faq/npm-symbol.png)

本文收集了有关 Azure 中网络性能监视器 (NPM) 的常见问题 (FAQ)

[网络性能监视器](/azure/networking/network-monitoring-overview)是一项基于云的[混合网络监视](../../azure-monitor/insights/network-performance-monitor-performance-monitor.md)解决方案，可帮助你监视网络基础结构中不同点之间的网络性能。 它还可以监视到[服务和应用程序终结点](../../azure-monitor/insights/network-performance-monitor-service-connectivity.md)的网络连接，以及[监视 Azure ExpressRoute 的性能](../../azure-monitor/insights/network-performance-monitor-expressroute.md)。 

网络性能监视器可检测诸如流量黑洞、路由错误之类的网络问题，以及传统网络监视方法无法检测到的问题。 只要突破网络链接的阈值，解决方案就会生成警报并进行通知。 它还可以确保及时检测到网络性能问题，然后确定问题根源所在的特定网络段或设备。 

可在线查看相关信息，深入了解[网络性能监视器](https://docs.microsoft.com/azure/networking/network-monitoring-overview)支持的各种功能。

## <a name="set-up-and-configure-agents"></a>安装和配置代理

### <a name="what-are-the-platform-requirements-for-the-nodes-to-be-used-for-monitoring-by-npm"></a>NPM 进行监视所用的节点要满足哪些平台要求？
下面列出了 NPM 各项功能所要满足的平台要求：

- NPM 的性能监视器和服务连接性监视器功能支持 Windows server 和 Windows 桌面/客户端操作系统。 支持的 Windows 服务器操作系统版本是 2008 SP1 或更高版本。 支持的 Windows 桌面/客户端版本是 Windows 10、 Windows 8.1、 Windows 8 和 Windows 7。 
- NPM 的 ExpressRoute 监视器功能仅支持 Windows Server（2008 SP1 或更高版本）操作系统。

### <a name="can-i-use-linux-machines-as-monitoring-nodes-in-npm"></a>是否可以使用 Linux 计算机作为 NPM 中的监视节点？
若要监视网络使用基于 Linux 的节点的功能目前处于预览状态。 请联系客户经理了解详细信息。 Linux 代理仅为 NPM 的性能监视器功能提供监视功能，不适用于服务连接监视器和 ExpressRoute 监视器功能

### <a name="what-are-the-size-requirements-of-the-nodes-to-be-used-for-monitoring-by-npm"></a>NPM 进行监视所用的节点要满足哪些大小要求？
要在节点 VM 上运行 NPM 解决方案以监视网络，节点应至少有 500 MB 内存和 1 个核心。 您不必使用单独的节点，用于运行 NPM。 该解决方案可以在运行了其他工作负荷的节点上运行。 该解决方案包含超过 5%的 CPU 功能，若要停止监视进程，如果它使用。

### <a name="to-use-npm-should-i-connect-my-nodes-as-direct-agent-or-through-system-center-operations-manager"></a>若要使用 NPM，是要以直接代理的形式还是通过 System Center Operations Manager 连接节点？
性能监视器和服务连接性监视器功能支持节点[作为直接代理连接](../../azure-monitor/platform/agent-windows.md)并[通过 Operations Manager 连接](../../azure-monitor/platform/om-agents.md)。

对于 ExpressRoute 监视器功能，Azure 节点只能以直接代理的形式连接。 不支持通过 Operations Manager 连接的 Azure 节点。 对于在本地节点，用于监视 ExpressRoute 线路支持作为直接代理和 Operations Manager 通过连接的节点。

### <a name="which-protocol-among-tcp-and-icmp-should-be-chosen-for-monitoring"></a>应选择 TCP 还是 ICMP 协议进行监视？
如果您在监视你使用 Windows server 基于节点的网络，我们建议使用 TCP 作为监视协议因为它提供更好的准确性。 

建议将 ICMP 用于基于 Windows 桌面/客户端操作系统的节点。 此平台 does'nt 允许 TCP 原始套接字，NPM 使用来发现网络拓扑通过发送的数据。

可在[此处](../../azure-monitor/insights/network-performance-monitor-performance-monitor.md#choose-the-protocol)详细了解每个协议的相对优势。

### <a name="how-can-i-configure-a-node-to-support-monitoring-using-tcp-protocol"></a>如何将节点配置为支持使用 TCP 协议进行监视？
要使节点支持使用 TCP 协议进行监视： 
* 请确保节点平台是 Windows Server（2008 SP1 或更高版本）。
* 在该节点上运行 [EnableRules.ps1](https://aka.ms/npmpowershellscript) Powershell 脚本。 参阅[说明](../../azure-monitor/insights/network-performance-monitor.md#configure-log-analytics-agents-for-monitoring)了解更多详细信息。


### <a name="how-can-i-change-the-tcp-port-being-used-by-npm-for-monitoring"></a>如何更改 NPM 用来监视的 TCP 端口？
可以运行 [EnableRules.ps1](https://aka.ms/npmpowershellscript) 脚本来更改 NPM 用来监视的 TCP 端口。 需要输入用作参数的端口号。 例如，若要在端口 8060 上启用 TCP，请运行 `EnableRules.ps1 8060`。 确保在用于监视的所有节点上使用相同的 TCP 端口。

此脚本仅在本地配置 Windows 防火墙。 如果有网络防火墙或网络安全组 (NSG) 规则，请确保它们允许将流量发往 NPM 所用 TCP 端口。

### <a name="how-many-agents-should-i-use"></a>应使用多少个代理？
对于要监视的每个子网，至少要使用一个代理。

### <a name="what-is-the-maximum-number-of-agents-i-can-use-or-i-see-error--youve-reached-your-configuration-limit"></a>什么是我可以使用的代理的最大数目或看到错误"...你已达到配置限制"？
NPM 将 IP 数限制为每个工作区 5000 个 IP。 如果某个节点同时拥有 IPv4 和 IPv6 地址，则计为该节点的 2 个 IP。 因此，此 5000 个 IP 的限制会决定代理数的上限。 可从“NPM”>>“配置”中的“节点”选项卡删除非活动代理。 NPM 还维护所有曾经指派给虚拟机主持代理和每个 Ip 被计为单独的 IP 分配给该上限 5000 Ip 的历史记录。 到释放一些 Ip 的工作区中，您可以使用节点页上删除未使用的 Ip。

## <a name="monitoring"></a>监视

### <a name="how-are-loss-and-latency-calculated"></a>如何计算丢包率和延迟
源代理按固定间隔将 TCP SYN 请求（如果选择 TCP 作为用于监视的协议）或 ICMP ECHO 请求（如果选择 ICMP 作为用于监视的协议）发送到目标 IP，以确保涵盖源-目标 IP 组合之间的所有路径。 将测量收到的数据包百分比和数据包往返时间，以计算每个路径的丢包率和延迟。 将会基于轮询间隔和所有路径聚合此数据，以获取特定轮询间隔内 IP 组合的失包率和延迟的聚合值。

### <a name="with-what-frequency-does-the-source-agent-send-packets-to-the-destination-for-monitoring"></a>源代理按哪种频率将数据包发送到目标进行监视？
对于性能监视器和 ExpressRoute 监视器功能，源每隔 5 秒发送数据包，并记录网络测量值。 将会基于 3 分钟轮询间隔聚合此数据，以计算丢失率和延迟的平均值与峰值。 对于服务连接监视器功能，发送数据包进行网络测量的频率由用户在配置测试时为特定测试输入的频率确定。

### <a name="how-many-packets-are-sent-for-monitoring"></a>要发送多少个数据包进行监视？
源代理在轮询中发送到目标的数据包数目是自适应性的，由我们的专属算法确定，该算法可能因网络拓扑的不同而异。 源-目标 IP 组合之间的网络路径越多，则发送的数据包数目也就越多。 系统会确保涵盖源-目标 IP 组合之间的所有路径。

### <a name="how-does-npm-discover-network-topology-between-source-and-destination"></a>NPM 如何发现源与目标之间的网络拓扑？
NPM 使用基于跟踪路由的专属算法来发现源与目标之间的所有路径和跃点。

### <a name="does-npm-provide-routing-and-switching-level-info"></a>NPM 提供路由和交换级别信息 
尽管 NPM 可以检测源代理与目标之间所有可能的路由，但它不提供特定工作负荷发送的数据包所采用的路由的深入信息。 该解决方案可帮助识别那些超预期增大延迟的路径和底层网络跃点。

### <a name="why-are-some-of-the-paths-unhealthy"></a>为何某些路径不正常？
源与目标 IP 之间可能存在不同的网络路径，而每个路径可能存在不同的丢包率和延迟值。 NPM 将这些路径标记为不正常（以红色表示），它们的失包率和/或延迟值大于监视配置中设置的相关阈值。

### <a name="what-does-a-hop-in-red-color-signify-in-the-network-topology-map"></a>网络拓扑图中红色的跃点表示什么？
如果某个跃点为红色，表示它属于至少一个不正常的路径。 NPM 只会将路径标记为不正常，而不会隔离每个路径的运行状况。 若要识别有问题的跃点，可以查看每个跃点的延迟，并隔离那些超预期增大延迟的跃点。

### <a name="how-does-fault-localization-in-performance-monitor-work"></a>性能监视器中的故障定位的工作原理是什么？
NPM 根据每个网络路径、网段和构成网络跃点所属的不正常路径数，使用概率机制向它们分配故障概率。 随着网段和跃点属于越来越多的不正常路径，与之关联的故障概率将会增大。 如果有许多包含 NPM 代理的节点相互连接，因此增加了用于计算故障概率的数据点，则此算法的效果最佳。

### <a name="how-can-i-create-alerts-in-npm"></a>如何在 NPM 中创建警报？
请参阅[文档中的警报部分](https://docs.microsoft.com/azure/log-analytics/log-analytics-network-performance-monitor#alerts)获取分步说明。

### <a name="can-npm-monitor-routers-and-servers-as-individual-devices"></a>NPM 是否可以将路由器和服务器作为单个设备进行监视？
NPM 只能识别源与目标 IP 之间的底层网络跃点（交换机、路由器、服务器等）的 IP 和主机名。 此外，它还能识别这些已识别的跃点之间的延迟。 它不会单独监视这些底层跃点。

### <a name="can-npm-be-used-to-monitor-network-connectivity-between-azure-and-aws"></a>是否可以使用 NPM 来监视 Azure 与 AWS 之间的网络连接？
可以。 有关详细信息，请参阅[使用 NPM 监视 Azure、AWS 和本地网络](https://blogs.technet.microsoft.com/msoms/2016/08/30/monitor-on-premises-cloud-iaas-and-hybrid-networks-using-oms-network-performance-monitor/)一文。

### <a name="is-the-expressroute-bandwidth-usage-incoming-or-outgoing"></a>ExpressRoute 带宽用量是指传入还是传出带宽？
带宽用量是传入和传出带宽的总计。 它以“位/秒”为单位表示。

### <a name="can-we-get-incoming-and-outgoing-bandwidth-information-for-the-expressroute"></a>是否可以获取 ExpressRoute 的传入和传出带宽信息？
可以捕获主要和辅助带宽的传入和传出值。

如需对等互连级信息，请在日志搜索中使用下面所述的查询

    NetworkMonitoring 
    | where SubType == "ExpressRoutePeeringUtilization"
    | project CircuitName,PeeringName,PrimaryBytesInPerSecond,PrimaryBytesOutPerSecond,SecondaryBytesInPerSecond,SecondaryBytesOutPerSecond
  
如需线路级信息，请使用下面所述的查询 

    NetworkMonitoring 
    | where SubType == "ExpressRouteCircuitUtilization"
    | project CircuitName,PrimaryBytesInPerSecond, PrimaryBytesOutPerSecond,SecondaryBytesInPerSecond,SecondaryBytesOutPerSecond

### <a name="which-regions-are-supported-for-npms-performance-monitor"></a>NPM 的性能监视器支持哪些区域？
NPM 可以通过某个[受支持区域](../../azure-monitor/insights/network-performance-monitor.md#supported-regions)中托管的工作区，监视全球任意位置的网络之间的连接

### <a name="which-regions-are-supported-for-npms-service-connectivity-monitor"></a>NPM 的服务连接监视器支持哪些区域？
NPM 可以通过某个[受支持区域](../../azure-monitor/insights/network-performance-monitor.md#supported-regions)中托管的工作区，监视全球任意位置的服务的连接

### <a name="which-regions-are-supported-for-npms-expressroute-monitor"></a>NPM 的 ExpressRoute 监视器支持哪些区域？
NPM 可以监视任何 Azure 区域中的 ExpressRoute 线路。 若要载入到 NPM，必须需要某个[受支持区域](/azure/expressroute/how-to-npm)中托管的 Log Analytics 工作区

## <a name="troubleshoot"></a>排除故障

### <a name="why-are-some-of-the-hops-marked-as-unidentified-in-the-network-topology-view"></a>网络拓扑视图中为何有些跃点标记为不可识别？
NPM 使用跟踪路由的修改版来发现从源代理到目标的拓扑。 不可识别的跃点表示该网络跃点未响应源代理的跟踪路由请求。 如果三个连续的网络跃点不响应代理的跟踪路由，该解决方案将标记为无法识别的响应跃点，并不会尝试发现更多跃点。

如果存在以下一种或多种情况，跃点可能不会响应跟踪路由：

* 路由器已配置为隐藏其标识。
* 网络设备不允许 ICMP_TTL_EXCEEDED 流量。
* 防火墙阻止了来自网络设备的 ICMP_TTL_EXCEEDED 响应。

### <a name="why-does-my-link-show-unhealthy-but-the-topology-does-not"></a>为什么不我链接显示不正常，但拓扑却不 
NPM 监视端到端丢失、 延迟和在不同的时间间隔的拓扑。 丢失和延迟进行测量一次每隔 5 秒和聚合每隔 3 分钟 （适用于性能监视器和 Express 路由监视器） 时使用跟踪路由每 10 分钟一次计算拓扑。 例如，3:44，4:04，拓扑可能会更新三次 (3:44，3 分 54 秒，4:04)，但会丢失和延迟更新大约 7 次 (3:44，3:47，3:50，3:53，3:56，3:59，4:02)。 将丢失和延迟获取计算的 3:56 个呈现在 3 分 54 秒生成的拓扑 3:59 和 4:02。 假设收到警报 ER 线路处于不正常状态 3:59。 登录到 NPM 和尝试将拓扑时间设置为 3:59。 NPM 将呈现在 3 分 54 秒生成的拓扑。 若要了解你的网络的最后一个已知的拓扑，比较字段 TimeProcessed （在计算哪些数据丢失和延迟的时间） 和 TracerouteCompletedTime （在哪种拓扑计算时间）。 

### <a name="what-is-the-difference-between-the-fields-e2emedianlatency-and-avghoplatencylist-in-the-networkmonitoring-table"></a>中的字段 E2EMedianLatency 和 AvgHopLatencyList NetworkMonitoring 表之间的区别是什么
E2EMedianLatency 是更新聚合 tcp ping 测试的结果，而 AvgHopLatencyList 是更新基于 traceroute 每隔 10 分钟之后每隔 3 分钟的延迟。 若要了解的确切时间，在其中计算 E2EMedianLatency，使用字段 TimeProcessed。 若要了解哪些 traceroute 完成并更新的 AvgHopLatencyList 的确切时间，使用字段 TracerouteCompletedTime

### <a name="why-does-hop-by-hop-latency-numbers-differ-from-hoplatencyvalues"></a>逐跳延迟数从 HopLatencyValues 为何不同 
HopLatencyValues 是到终结点的源。
例如：跃点-A，B，c。 AvgHopLatency-10,15,20。 这意味着对延迟源 = 10，到 B 的延迟源 = 15，对 C 延迟的源为 20。 UI 将计算为拓扑中的 5 A-B 跃点延迟

### <a name="the-solution-shows-100-loss-but-there-is-connectivity-between-the-source-and-destination"></a>解决方案会显示 100% 的丢包率，但源与目标之间已建立连接
如果主机防火墙或中间防火墙（网络防火墙或 Azure NSG）阻止了源代理与目标之间通过 NPM 用于监视的端口（端口默认为 8084，除非客户更改了端口）进行的通信，则可能会发生这种情况。

* 若要确认主机防火墙是否未阻止所需端口上的通信，请通过以下视图查看源和目标节点的运行状况：“网络性能监视器”->“配置”->“节点”。 
  如果这些节点不正常，请查看说明并采取纠正措施。 如果节点正常，请转到下面的 使用。
* 若要确认中间网络防火墙或 Azure NSG 是否未阻止所需端口上的通信，请遵照下面的说明使用第三方 PsPing 实用工具：
  * 可从[此处](https://technet.microsoft.com/sysinternals/psping.aspx)获取 psping 实用工具。 
  * 在源节点中运行以下命令。
    * psping -n 15 \<目标节点 IP 地址\>:<端口号>。NPM 默认使用端口 8084。 如果使用 EnableRules.ps1 脚本显式更改了此端口，请输入所用的自定义端口号。 这是从 Azure 机器向本地执行的 ping
* 检查 ping 是否成功。 如果未成功，则表示中间网络防火墙或 Azure NSG 阻止了此端口上的流量。
* 现在，从目标节点向源节点 IP 运行该命令。


### <a name="there-is-loss-from-node-a-to-b-but-not-from-node-b-to-a-why"></a>从节点 A 到 B 的通信发生了丢包，但从节点 B 到 A 的通信未发生丢包，这是为什么？
由于从 A 到 B 之间的网络路径可能不同于从 B 到 A 之间的网络路径，因此，可能会观察到不同的丢包率和延迟值。

### <a name="why-are-all-my-expressroute-circuits-and-peering-connections-not-being-discovered"></a>为何发现不了我的所有 ExpressRoute 线路和对等互连？
NPM 现在可以在用户有权访问的所有订阅中发现 ExpressRoute 线路和对等连接。 选择链接 Express Route 资源的所有订阅，并为发现的每个资源启用监视。 NPM 在发现专用对等互连时查找连接对象，因此请检查 VNET 是否与对等互连关联。

### <a name="the-er-monitor-capability-has-a-diagnostic-message-traffic-is-not-passing-through-any-circuit-what-does-that-mean"></a>ER 监视器功能发出了诊断消息“流量无法通过任何线路”。 这是什么意思？

可能存在这种情况：本地与 Azure 节点之间建立了正常的连接，但流量不能通过配置为由 NPM 监视的 ExpressRoute 线路。 

在以下情况下可能发生这种问题：

* ER 线路已关闭。
* 路由筛选器的配置使得其他路由（例如 VPN 连接或其他 ExpressRoute 线路）的优先级高于所需的 ExpressRoute 线路。 
* 监视配置中选择用来监视 ExpressRoute 线路的本地和 Azure 节点未通过所需的 ExpressRoute 线路相互建立连接。 确保选择正确的节点，并通过所要监视的 ExpressRoute 线路让它们相互建立连接。

### <a name="while-configuring-monitoring-of-my-expressroute-circuit-the-azure-nodes-are-not-being-detected"></a>为 ExpressRoute 线路配置监视时，并未检测 Azure 节点。
如果 Azure 节点是通过 Operations Manager 连接的，则可能发生这种情况。 ExpressRoute 监视器功能仅支持以直接代理形式连接的 Azure 节点。

### <a name="i-cannot-discover-by-expressroute-circuits-in-the-oms-portal"></a>无法在 OMS 门户中发现 ExpressRoute 线路
尽管在 Azure 门户和 OMS 门户中都可以使用 NPM，但 ExpressRoute 监视器中的线路发现功能只能通过 Azure 门户执行。 通过 Azure 门户发现线路后，可在上述任一门户中使用该功能。 

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-network-loss-as-well-as-latency-are-shown-as-na"></a>在服务连接监视器功能中，服务响应时间、网络丢包和延迟显示为“不适用”
如果存在以下一种或多种情况，则可能会发生此问题：

* 服务已关闭。
* 用来检查服务的网络连接的节点已关闭。
* 在测试配置中输入的目标不正确。
* 节点未建立任何网络连接。

### <a name="in-the-service-connectivity-monitor-capability-a-valid-service-response-time-is-shown-but-network-loss-as-well-as-latency-are-shown-as-na"></a>在服务连接监视器功能中，有效服务响应时间已显示，但网络丢包和延迟显示为“不适用”
 如果存在以下一种或多种情况，则可能会发生此问题：

* 如果用来检查服务的网络连接的节点是 Windows 客户端计算机，则原因是目标服务正在阻止 ICMP 请求，或者网络防火墙正在阻止 ICMP 来自该节点的请求。
* 在测试配置中，“执行网络测量”复选框为空。

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-is-na-but-network-loss-as-well-as-latency-are-valid"></a>在服务连接监视器功能中，服务响应时间为“不适用”，但网络丢包和延迟有效
如果目标服务不是 Web 应用程序，但测试配置为 Web 测试，则可能会发生这种情况。 编辑测试配置，选择“网络”而不是“Web”作为测试类型。

## <a name="miscellaneous"></a>杂项

### <a name="is-there-a-performance-impact-on-the-node-being-used-for-monitoring"></a>用于监视的节点的性能是否受影响？
NPM 进程配置为当它的主机 CPU 资源利用率超过 5% 时停止。 这是为了确保可以持续使用这些节点来处理其常规工作负荷，而不会影响性能。

### <a name="does-npm-edit-firewall-rules-for-monitoring"></a>NPM 是否会编辑用于监视的防火墙规则？
NPM 只会在运行 EnableRules.ps1 Powershell 脚本的节点上创建本地 Windows 防火墙规则，来允许代理在指定的端口上相互建立 TCP 连接。 该解决方案不会修改任何网络防火墙或网络安全组 (NSG) 规则。

### <a name="how-can-i-check-the-health-of-the-nodes-being-used-for-monitoring"></a>如何检查用于监视的节点的运行状况？
可通过以下视图查看用于监视的节点的运行状况：“网络性能监视器”->“配置”->“节点”。 如果某个节点不正常，可以查看错误详细信息并采取建议的措施。

### <a name="can-npm-report-latency-numbers-in-microseconds"></a>NPM 是否可以微秒为单位报告延迟数字？
NPM 在 UI 中以毫秒为单位将延迟数字四舍五入。 相同的数据将以更高的粒度存储（精确度有时可高达四位小数）。

## <a name="next-steps"></a>后续步骤

- 参阅 [Azure 中的网络性能监视器解决方案](../../azure-monitor/insights/network-performance-monitor.md)来详细了解网络性能监视器。
