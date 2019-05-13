---
title: Azure Vm 的 TCP/IP 性能优化 |Microsoft Docs
description: 了解各种常见 TCP/IP 性能优化技术，以及它们与 Azure Vm 的关系。
services: virtual-network
documentationcenter: na
author:
- rimayber
- dgoddard
- stegag
- steveesp
- minale
- btalb
- prachank
manager: paragk
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/02/2019
ms.author:
- rimayber
- dgoddard
- stegag
- steveesp
- minale
- btalb
- prachank
ms.openlocfilehash: d0124d6656167af3942e0d054b4e1fa7a2b48e8b
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410050"
---
# <a name="tcpip-performance-tuning-for-azure-vms"></a>TCP/IP 的性能优化 Azure Vm

本文介绍常见 TCP/IP 性能优化技术以及用于在 Azure 上运行的虚拟机时应注意一些事项。 它将提供的技术的基本概述并了解如何优化。

## <a name="common-tcpip-tuning-techniques"></a>常见的 TCP/IP 优化技术

### <a name="mtu-fragmentation-and-large-send-offload"></a>MTU、 碎片和大量发送卸载

#### <a name="mtu"></a>MTU

最大传输单元 (MTU) 是以字节为单位，可以通过网络接口发送指定的最大大小帧 （数据包）。 MTU 是可配置的设置。 默认值在 Azure Vm 使用 MTU，大多数网络设备上的默认设置全局范围内，为 1,500 字节。

#### <a name="fragmentation"></a>碎片

当超过网络接口的 MTU 发送一个数据包时，会出现碎片。 TCP/IP 堆栈将分解为较小的部分 （片段） 符合接口的 MTU 的数据包。 碎片发生在 IP 层，并独立于基础协议 （如 TCP)。 2,000 个字节数据包发送通过有 1,500 MTU 的网络接口后，数据包将分解为一个 1,500 字节数据包和一个 500 个字节的数据包。

源和目标之间的路径中的网络设备可以超过 MTU 或将数据包分段成较小片段或者丢弃数据包。

#### <a name="the-dont-fragment-bit-in-an-ip-packet"></a>IP 数据包中的位不分段

不要分段 (DF) 位是 IP 协议标头中的标志。 DF 位指示发送方和接收方之间的路径上的网络设备必须将数据包不分段。 原因有很多，无法设置此位。 （请参阅"路径 MTU 发现"部分中的一个示例。）如果网络设备收到的数据包不分段设置位，并且该数据包的大小超过设备的接口 MTU，标准行为是针对要丢弃该数据包的设备。 设备将 ICMP 碎片所需的消息发送回原始数据包的源。

#### <a name="performance-implications-of-fragmentation"></a>性能产生的碎片影响

碎片可能会负面的性能影响。 对性能的影响的主要原因之一是碎片的 CPU/内存影响和重组的数据包。 当网络设备需要片段数据包时，它必须分配的 CPU/内存资源来执行碎片。

数据包重新合并会发生相同的操作。 网络设备必须存储所有片段，直到在接收到以便它可以将其重新整合成原始数据包。 此过程的分段和重组也会导致延迟。

碎片的其他可能的负面性能含义是零碎的数据包可能到达顺序。 无序收到的数据包，某些类型的网络设备可以删除它们。 在这种情况下，整个数据包必须重新传输。

片段通常删除被网络防火墙等安全设备或已耗尽时的网络设备的接收缓冲区。 当网络设备的接收缓冲区已用尽，网络设备正在尝试重新组装零碎的数据包，但是没有资源来存储和 reassume 数据包。

碎片可以被视为负操作，但支持通过 internet 进行连接不同的网络时，碎片是必需的。

#### <a name="benefits-and-consequences-of-modifying-the-mtu"></a>优势和后果的修改 MTU

通常情况下，您可以通过增加 MTU 创建效率更高的网络。 传输的每个数据包已添加到原始数据包的标头信息。 当碎片创建更多数据包时，没有开销，多个标头，这使网络效率较低。

下面是一个示例。 以太网标头大小为 14 个字节加上用于确保帧一致性的 4 字节帧检查序列。 如果发送一个 2,000 个字节数据包，是在网络上添加 18 个字节的以太网开销。 如果该数据包分为碎片到 1,500 字节数据包和 500 字节数据包，每个数据包将具有 18 个字节的以太网标头，共 36 个字节。

请记住，增加 MTU 一定不会创建更高效的网络。 如果应用程序发送仅 500 字节数据包，是否 MTU 为 1500 字节或 9000 字节将存在同一标题开销。 网络将变得更有效率，仅当它使用更大 MTU 受影响的数据包大小。

#### <a name="azure-and-vm-mtu"></a>Azure 和 VM MTU

默认值适用于 Azure Vm 的 MTU 为 1500 字节。 Azure 虚拟网络堆栈将尝试片段为 1400 个字节的数据包。 但虚拟网络堆栈将允许数据包达 2,006 个字节时的不分段位设置 IP 标头中。

请注意，虚拟网络堆栈不本质上是效率低下，因为它片段为 1400 个字节的数据包，即使 Vm 具有 MTU 为 1500。 大量的网络数据包都比 1400 或 1,500 字节数小得多。

#### <a name="azure-and-fragmentation"></a>Azure 和碎片

虚拟网络堆栈设置以删除"无序碎片，"不以其原始的碎片化顺序到达，它是零碎的数据包。 主要是因为它在名为 FragmentSmack 2018 年 11 月宣布的网络安全漏洞会丢弃这些数据包。

FragmentSmack 是 Linux 内核处理重组零碎的 IPv4 和 IPv6 数据包的方式中的一个缺陷。 远程攻击者可以使用此触发器昂贵片段重组操作，这将导致增加的 CPU 和拒绝服务在目标系统上的缺陷。

#### <a name="tune-the-mtu"></a>优化 MTU

你可以在任何其他操作系统中可以配置 Azure VM MTU。 应考虑出现在 Azure 中，上文所述的碎片，但当你配置 MTU。

我们不鼓励客户增加 VM Mtu。 本文旨在介绍 Azure 如何实现 MTU 和执行碎片的详细信息。

> [!IMPORTANT]
>增加 MTU 不认为可提高性能，并且可能具有对应用程序性能的负面影响。
>
>

#### <a name="large-send-offload"></a>大量发送卸载

大量发送卸载 (LSO) 可以通过卸载到以太网适配器的数据包分段提高网络性能。 启用 LSO 后，TCP/IP 堆栈将创建一个大型的 TCP 数据包，并将其发送到转发它之前的分段的以太网适配器。 LSO 的好处是它可以免费从细分到符合 MTU 和该将处理卸载到硬件中执行其中的以太网接口的大小的数据包的 CPU。 若要了解有关 LSO 优点的详细信息，请参阅[支持大量发送卸载](https://docs.microsoft.com/windows-hardware/drivers/network/performance-in-network-adapters#supporting-large-send-offload-lso)。

启用 LSO 后，Azure 客户可能会在执行数据包捕获时看到大帧大小。 某些客户认为出现碎片或大 MTU，正在使用不时，可能会导致这些大帧大小。 此外，使用以太网适配器可以播发更大的最大段大小 (MSS) 到 TCP/IP 堆栈，以便创建更大的 TCP 数据包。 此非分段的整个帧然后转发到的以太网适配器，并且会在 VM 上执行数据包捕获中可见。 但是，数据包将分解为多个较小的帧通过以太网适配器，根据以太网适配器的 MTU。

### <a name="tcp-mss-window-scaling-and-pmtud"></a>TCP MSS 窗口缩放和 PMTUD

#### <a name="tcp-maximum-segment-size"></a>TCP 最大段大小

TCP 最大段大小 (MSS) 是一个设置，这样可以避免碎片的 TCP 数据包的 TCP 段的大小限制。 通常，操作系统将使用此公式来设置 MSS:

`MSS = MTU - (IP header size + TCP header size)`

IP 标头和 TCP 标头是每个，即 20 个字节或 40 个字节总数。 因此 1,500 的 mtu 接口将具有 1,460 MSS。 但 MSS 是可配置。

在源和目标之间建立 TCP 会话时，此设置 TCP 三次握手中同意到中。 双方都发送 MSS 值和 TCP 连接所使用的两个较小。

请记住 Mtu 的源和目标不是唯一因素确定 MSS 值。 中间网络设备，如 VPN 网关，包括 Azure VPN 网关，可以调整独立于源和目标 MTU，以确保最佳的网络性能。

#### <a name="path-mtu-discovery"></a>路径 MTU 发现

MSS 进行协商，但它不一定表明可以使用实际 MSS。 这是因为源和目标之间的路径中的其他网络设备可能具有低于 MTU 值的源和目标。 在这种情况下，其 MTU 小于数据包的设备将丢弃该数据包。 设备将发送回包含其 MTU 的 ICMP 碎片需要 （类型 3，代码 4） 的消息。 此 ICMP 消息允许适当地降低其路径 MTU 源主机。 过程被称为路径 MTU 发现 (PMTUD)。

PMTUD 过程会导致效率低下，并且会影响网络性能。 当超过网络路径的 MTU 发送数据包时，则需要与较低 MSS 重新传输数据包。 如果将也许发件人不会由于网络防火墙的路径中收到 ICMP 碎片需要消息，(通常称为*PMTUD 黑洞*)，发送方不知道它需要降低 mss 交互，并将持续重新传输该数据包。 这就是原因，我们不建议增加 Azure VM MTU。

#### <a name="vpn-and-mtu"></a>VPN 和 MTU

如果使用执行 （如 IPsec Vpn) 封装的虚拟机，有一些有关数据包大小和 MTU 的其他注意事项。 Vpn 将多个标头添加到数据包，这样会增加数据包大小，并且需要较小 MSS。

Azure，我们建议您设置 TCP MSS 钳位为 1,350 字节和隧道接口 MTU 1400。 有关详细信息，请参阅[VPN 设备和 IPSec/IKE 参数页](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices)。

### <a name="latency-round-trip-time-and-tcp-window-scaling"></a>延迟、 往返时间和 TCP 窗口缩放

#### <a name="latency-and-round-trip-time"></a>延迟和往返时间

网络延迟受光速纤程光纤网络上。 由两个网络设备之间的往返时间 (RTT) 也会有效地控制 TCP 的网络吞吐量。

| | | | |
|-|-|-|-|
|Route|**距离**|**单向时间**|**RTT**|
|纽约飞往旧金山|4,148 公里|21 ms|42 ms|
|伦敦到纽约|5,585 公里|28 ms|56 ms|
|纽约到悉尼|15,993 公里|80 毫秒|160 ms|

此表显示了两个位置之间的直线距离。 在网络中，距离长于通常的直线距离。 下面是一个简单的公式来计算最小的 RTT 为受光速：

`minimum RTT = 2 * (Distance in kilometers / Speed of propagation)`

有关传播的速度，可以使用 200。 这是距离，以米为单位，光传输 1 毫秒。

让我们纽约到旧金山看作为示例。 直线距离是 4,148 km。 将该值插入到公式，我们得到以下结果：

`Minimum RTT = 2 * (4,148 / 200)`

公式的输出是以毫秒为单位。

如果你想要获得最佳的网络性能，逻辑的方法是选择与它们之间最短的距离的目标。 您还应设计虚拟网络以优化流量的路径并减少延迟。 有关详细信息，请参阅这篇文章的"网络设计注意事项"部分。

#### <a name="latency-and-round-trip-time-effects-on-tcp"></a>对 TCP 的延迟和往返时间影响

往返时间有直接影响最大 TCP 吞吐量。 在 TCP 协议中，*窗口大小*是发件人需要从接收方收到确认之前，可以通过 TCP 连接发送的流量的最长。 如果 TCP MSS 设置为 1,460 和 TCP 窗口大小设置为 65,535，发件人可以发送 45 数据包之前必须从接收方接收确认。 如果发件人不会收到确认，则会重新传输数据。 下面是公式：

`TCP window size / TCP MSS = packets sent`

在此示例中，65,535 / 舍入 1,460 达 45。

此"等待确认"状态下，一种机制来确保可靠传递数据，是哪些因素会导致影响 TCP 吞吐量的 RTT。 发件人确认将等待的时间越长，它需要发送更多数据之前等待的时间就越长。

下面是用于计算单个 TCP 连接的最大吞吐量的公式：

`Window size / (RTT latency in milliseconds / 1,000) = maximum bytes/second`

下表显示了最大兆字节 / 每秒吞吐量的单个 TCP 连接。 （出于可读性考虑，兆字节为单位用于度量单位。）

| | | | |
|-|-|-|-|
|**TCP 窗口大小 （字节）**|**RTT 延迟 (ms)**|**最大兆字节/秒吞吐量**|**最大兆比特/秒吞吐量**|
|65,535|1|65.54|524.29|
|65,535|30|2.18|17.48|
|65,535|60|1.09|8.74|
|65,535|90|.73|5.83|
|65,535|120|.55|4.37|

如果数据包都将丢失，而发件人一段重新传输已发送的数据将减少 TCP 连接的最大吞吐量。

#### <a name="tcp-window-scaling"></a>TCP 窗口缩放

TCP 窗口缩放是一种方法，动态地增加 TCP 窗口大小，以允许更多数据需要提供确认之前发送。 在上一示例中之前确认所需,，则会发送 45 数据包。 如果您增加需要确认前可以发送的数据包数，会缩减的发件人正在等待确认时，这会增加 TCP 最大吞吐量的次数。

此表说明了这些关系：

| | | | |
|-|-|-|-|
|**TCP 窗口大小 （字节）**|**RTT 延迟 (ms)**|**最大兆字节/秒吞吐量**|**最大兆比特/秒吞吐量**|
|65,535|30|2.18|17.48|
|131,070|30|4.37|34.95|
|262,140|30|8.74|69.91|
|524,280|30|17.48|139.81|

但 TCP 窗口大小的 TCP 标头值是仅 2 个字节长，这意味着接收窗口的最大值为 65535。 若要增加最大窗口大小，引入了 TCP 窗口缩放比例。

缩放比例也是可以在操作系统中配置的设置。 下面是用于通过使用比例系数计算 TCP 窗口大小的公式：

`TCP window size = TCP window size in bytes \* (2^scale factor)`

以下是 3 和 65,535 的窗口大小的窗口缩放比例的计算：

`65,535 \* (2^3) = 262,140 bytes`

14 的比例因子会导致 TCP 窗口大小为 14 （允许的最大偏移）。 TCP 窗口大小将为 1,073,725,440 字节 （8.5 千兆字节）。

#### <a name="support-for-tcp-window-scaling"></a>TCP 窗口缩放的支持

Windows 可以为不同的连接类型设置不同的缩放比例。 （连接类包括数据中心，internet 上，依次类推。）您使用`Get-NetTCPConnection`PowerShell 命令查看窗口缩放连接类型：

```powershell
Get-NetTCPConnection
```

可以使用`Get-NetTCPSetting`PowerShell 命令来查看每个类的值：

```powershell
Get-NetTCPSetting
```

您可以设置初始 TCP 窗口大小和比例因子 TCP 在 Windows 中使用`Set-NetTCPSetting`PowerShell 命令。 有关详细信息，请参阅[集 NetTCPSetting](https://docs.microsoft.com/powershell/module/nettcpip/set-nettcpsetting?view=win10-ps)。

```powershell
Set-NetTCPSetting
```

这些是有效的 TCP 设置`AutoTuningLevel`:

| | | | |
|-|-|-|-|
|**AutoTuningLevel**|**比例因子**|**缩放乘数**|**公式<br/>计算最大窗口大小**|
|已禁用|无|无|窗口大小|
|受限|4|2^4|窗口大小 * (2 ^4)|
|严格限制|2|2^2|窗口大小 * (2 ^2)|
|普通|8|2^8|窗口大小 * (2 ^8)|
|实验性|14|2^14|窗口大小 * (2 ^14)|

这些设置是最有可能会影响 TCP 性能，但请记住通过 internet、 Azure 中，控件范围之外的许多其他因素也会影响 TCP 性能。

#### <a name="increase-mtu-size"></a>增加 MTU 大小

因为较大 MTU 意味着更大 MSS，您可能想知道是否增加 MTU 可以提高 TCP 性能。 很有可能并非如此。 有优点也有缺点超出只需 TCP 通信的数据包大小。 如前所述，TCP 吞吐量性能影响的最重要因素是 TCP 窗口大小、 数据包丢失和 RTT。

> [!IMPORTANT]
> 我们不建议 Azure 客户更改虚拟机上的默认 MTU 值。
>
>

### <a name="accelerated-networking-and-receive-side-scaling"></a>加速网络和接收方缩放

#### <a name="accelerated-networking"></a>更快的网络连接

虚拟机网络功能就是长久以来 CPU 资源在来宾 VM 和虚拟机监控程序/主机上。 通过在主机上传输的每个数据包在软件中处理的主机 CPU，包括所有虚拟网络封装和解封。 通过主机出现更多流量，因此更高的 CPU 负载。 并且，如果主机 CPU 正忙于处理其他操作，将还影响网络吞吐量和延迟。 Azure，解决此问题，具有加速网络。

加速的网络提供了通过 Azure 与 SR-IOV 之类的技术的内部可编程硬件一致 ultralow 网络延迟。 加速的网络移动 Azure 软件定义网络堆栈在 Cpu 和到基于 FPGA 的 SmartNICs 大部分。 此变化可让最终用户应用程序以回收计算周期，这是将较低负载放在 VM 中，延迟减少抖动和不一致。 换而言之，性能可能要更具确定性。

加速的网络可以在来宾 VM 以绕过主机并建立直接与主机的 SmartNIC 数据路径，从而提高性能。 下面是加速网络的一些好处：

- **降低延迟较高每秒数据包数 (pps) /**:从数据路径中删除虚拟交换机可以消除数据包在主机中进行策略处理所花费的时间并提高可在 VM 中处理的数据包数。

- **减少抖动**:虚拟交换机处理取决于需要应用的策略数量和正在执行处理的 CPU 的工作负荷。 通过将数据包传递到该 VM，不再主机 VM 通信和所有软件中断和上下文切换直接卸载到硬件强制执行的策略删除该可变性。

- **减少 CPU 使用率**:绕过主机中的虚拟交换机可以减少用于处理流量的 CPU 资源。

若要使用加速的网络，需要在每个适用的 VM 上显式启用它。 请参阅[创建具有加速网络的 Linux 虚拟机](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli)有关的说明。

#### <a name="receive-side-scaling"></a>接收方缩放

接收的方缩放 (RSS) 是一种网络驱动程序技术的更有效地将接收网络流量分发的分发在多处理器系统中接收处理跨多个 Cpu。 简单来说，RSS 允许系统处理更接收的流量，因为它而不是只是一个使用所有可用的 Cpu。 有关 RSS 的更多技术讨论，请参阅[简介接收方缩放](https://docs.microsoft.com/windows-hardware/drivers/network/introduction-to-receive-side-scaling)。

若要获得最佳性能的 VM 上启用加速的网络时，需要启用 RSS。 RSS 还可以提供不使用的 Vm 上的优势加速网络。 有关如何确定是否启用了 RSS 和如何启用它的概述，请参阅[的 Azure 虚拟机优化网络吞吐量](https://aka.ms/FastVM)。

### <a name="tcp-timewait-and-timewait-assassination"></a>TCP TIME_WAIT 和 TIME_WAIT assassination

TCP TIME_WAIT 是会影响网络和应用程序性能的另一个常见设置。 在忙碌的 Vm，会打开和关闭许多套接字，作为客户端或服务器 （源 IP:Source 端口 + 目标 IP:Destination 端口） 上，TCP 在正常操作期间，给定的套接字可能出现的处于 TIME_WAIT 状态很长时间。 TIME_WAIT 状态是指允许在套接字上提供才会关闭任何其他数据。 因此 TCP/IP 堆栈通常以无提示方式删除客户端的 TCP SYN 数据包通过防止重复使用的套接字。

一个套接字处于 TIME_WAIT 的时间量进行配置。 它可以为 240 秒范围从 30 秒。 套接字是有限的资源，并可以在任何给定时间使用的套接字的数目是可配置。 （可用的插槽数是通常约 30000。）如果可用的套接字的消耗，或者如果客户端和服务器具有不匹配的 TIME_WAIT 设置，并且 VM 尝试重复使用套接字处于 TIME_WAIT 状态，新连接将失败为以无提示方式丢弃数据包的 TCP SYN。

出站套接字的端口范围中的值为操作系统的 TCP/IP 堆栈中通常可配置。 这同样适用于 TCP TIME_WAIT 设置和套接字重复使用。 更改这些数字可能提高可伸缩性。 但是，根据具体情况，这些更改可能会导致互操作性问题。 你应小心，如果更改这些值。

可以使用 TIME_WAIT assassination 为了解决此缩放限制。 TIME_WAIT assassination 允许套接字后，可以重用在某些情况下，如新的连接的 IP 数据包中的序列号超出了与以前的连接的最后一个数据包的序列号。 在这种情况下，操作系统将允许建立新连接 （它会接受新 SYN ACK） 和强制关闭上一个连接处于 TIME_WAIT 状态。 在 Azure 中 Windows Vm 上支持此功能。 若要了解有关中的其他 Vm 的支持，请向 OS 供应商。

若要了解有关配置 TCP TIME_WAIT 设置和源端口范围，请参阅[设置可修改以提高网络性能](https://docs.microsoft.com/biztalk/technical-guides/settings-that-can-be-modified-to-improve-network-performance)。

## <a name="virtual-network-factors-that-can-affect-performance"></a>虚拟网络可能会影响性能的因素

### <a name="vm-maximum-outbound-throughput"></a>VM 的最大出站吞吐量

Azure 提供了各种 VM 大小和类型，每个都有各种不同的性能。 这些功能之一是网络吞吐量 （或带宽），这以兆位 / 秒 (Mbps)。 因为虚拟机托管共享硬件上，需使用相同的硬件的虚拟机之间合理共享网络容量。 更大的虚拟机分配比较小的虚拟机的更多带宽。

分配给每个虚拟机的网络带宽按虚拟机的传出（出站）流量计算。 从虚拟机流出的所有网络流量均计入分配限制，不管流向哪个目标。 例如，如果虚拟机有 1,000 Mbps 限制，该限制将应用是否将出站流量发送到在同一虚拟网络或 Azure 外部的另一个虚拟机。

传入流量不直接计算，或者说不直接受到限制。 但还有其他一些因素，如 CPU 和存储限制，可能会影响虚拟机的能力来处理传入的数据。

加速的网络旨在改进网络性能，包括延迟、 吞吐量和 CPU 使用率。 加速的网络可以提高虚拟机的吞吐量，但它可以执行该操作只能由虚拟机的已分配的带宽。

Azure 虚拟机都至少一个网络接口附加到它们。 它们可能具有多个。 分配给虚拟机的带宽是流经所有网络接口附加到计算机的所有出站流量的总和。 换而言之，在每个虚拟机基础，无论多少个网络接口附加到计算机上分配带宽。

中详细描述了预期的出站吞吐量和每个 VM 大小支持的网络接口数[大小的 Windows Azure 中的虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json)。 若要查看的最大吞吐量，请选择一种类型，如**常规用途**，并找到有关上生成的页面 （例如，"Dv2-系列"） 的大小系列的部分。 对于每个序列没有表提供了网络中的标题为的最后一列规范"最大 nic 数 / 预期网络带宽 (Mbps)。"

吞吐量限制适用于虚拟机。 吞吐量不受这些因素：

- **网络接口数**:带宽限制应用到的所有出站流量和虚拟机。

- **加速网络**:尽管此功能可以有助于流量达到已发布的限制，它不会更改此限制。

- **流量目标**:所有目标都计入出站限制。

- **协议**：所有协议的所有出站流量将计入此限制。

有关详细信息，请参阅[虚拟机的网络带宽](https://aka.ms/AzureBandwidth)。

### <a name="internet-performance-considerations"></a>Internet 性能注意事项

如本文所述，在 internet 上和 Azure 的控制范围之外的因素会影响网络性能。 下面是一些因素：

- **延迟**：在中间网络上的问题、 流量并不需要的"最短"的距离路径，以及非最优的对等互连路径，则可能会影响两个目标之间的往返时间。

- **数据包丢失**:数据包丢失可能引起网络拥塞、 物理路径问题和性能不佳的网络设备。

- **MTU 大小/碎片**:在路径上的碎片可能会导致延迟在数据到达或数据包到达顺序，这可能会影响数据包的传递。

跟踪路由是一个不错的工具，用于测量每个源设备和目标设备之间的网络路径上的网络性能特征 （如数据包丢失和延迟）。

### <a name="network-design-considerations"></a>网络设计注意事项

在本文前面部分中所述的注意事项，以及虚拟网络的拓扑可能会影响网络的性能。 例如，backhauls 全局流量到单个中心虚拟网络的中心辐射型设计将引入网络延迟，这将影响网络的总体性能。

网络设备的网络流量将通过传递数还会影响总体延迟。 例如，在中心辐射型设计中，如果流量传输到 internet 之前, 通过辐射网络虚拟设备和中心虚拟设备通过网络虚拟设备可以引入延迟。

### <a name="azure-regions-virtual-networks-and-latency"></a>Azure 区域、 虚拟网络和延迟

Azure 区域由一个常用地理区域中存在的多个数据中心组成。 这些数据中心可能无法以物理方式彼此。 在某些情况下它们被分隔多达 10 千米以内。 虚拟网络是基于 Azure 的物理数据中心网络逻辑覆盖。 虚拟网络并不意味着在数据中心内的任何特定的网络拓扑。

例如，两个 Vm 位于同一个虚拟网络和子网中可能在不同机架、 行或甚至数据中心。 它们无法分隔通过光纤电缆英尺或公里光纤电缆。 此变体可能会引入变量不同 Vm 之间的延迟 （几毫秒为单位差异）。

配置可用性集和可用性区域可以影响 Vm 的地理位置和之间的两个 Vm，则可能导致的延迟。 但在区域中的数据中心之间的距离按区域中的数据中心拓扑是特定于区域和主要影响。

### <a name="source-nat-port-exhaustion"></a>源 NAT 端口耗尽的情况

在 Azure 中的部署可以在公共 internet 上和/或公共 IP 空间中的 Azure 外部的终结点进行通信。 当实例启动时的出站连接时，Azure 动态将专用 IP 地址映射到公共 IP 地址。 Azure 将创建此映射后，出站发起流的返回流量可以访问的专用 IP 地址发起流。

对于每个出站连接，Azure 负载均衡器需要维护的某个时间段内此映射。 Azure 的多租户特性，维护此映射的每个 VM 每个出站流可以是需要消耗大量资源。 因此有限制的设置和基于 Azure 虚拟网络的配置。 或者，若要更精确地说，Azure VM 可以仅请一定数量的出站连接在给定的时间。 当达到这些限制时，VM 将无法进行更多出站连接。

但此行为是可配置。 详细了解 SNAT 和 SNAT 端口耗尽的情况，请参阅[这篇文章](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections)。

## <a name="measure-network-performance-on-azure"></a>度量值在 Azure 上的网络性能

与网络延迟相关的这篇文章中的性能最大配置数量 / 往返时间 (RTT) 之间的两个 Vm。 本部分提供有关如何测试延迟/RTT 以及如何测试 TCP 性能和 VM 的网络性能的一些建议。 您可以调整和性能测试使用在本部分中所述的方法前面讨论的 TCP/IP 和网络值。 您可以插入先前提供的计算延迟、 MTU、 MSS 和窗口大小值，并比较理论最大值为您在测试期间观察到的实际值。

### <a name="measure-round-trip-time-and-packet-loss"></a>度量值往返时间和数据包丢失

TCP 性能严重依赖 RTT 和数据包丢失。 PING 实用程序，可在 Windows 和 Linux 提供测量 RTT 和数据包丢失的最简单方法。 PING 的输出将显示源和目标之间的最小值/最大/平均延迟。 它还会显示数据包丢失。 默认情况下，PING 使用 ICMP 协议。 可以使用 PsPing 来测试 TCP RTT。 有关详细信息，请参阅[PsPing](https://docs.microsoft.com/sysinternals/downloads/psping)。

### <a name="measure-actual-throughput-of-a-tcp-connection"></a>度量值的 TCP 连接的实际吞吐量

NTttcp 是用于测试的 Linux 或 Windows VM 的 TCP 性能的工具。 可以更改各种 TCP 设置，然后使用 NTttcp 测试优势。 有关详细信息，请参阅以下资源：

- [带宽/吞吐量测试 (NTttcp)](https://aka.ms/TestNetworkThroughput)

- [NTttcp 实用程序](https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769)

### <a name="measure-actual-bandwidth-of-a-virtual-machine"></a>度量值实际带宽的虚拟机

您可以测试的不同 VM 类型，加速网络、 性能等，通过使用一个名为 iPerf 工具。 iPerf 也是在 Linux 和 Windows 上可用。 iPerf 可用于 TCP 或 UDP 测试总体网络吞吐量。 iPerf TCP 吞吐量测试受影响的因素 （如延迟和 RTT） 这篇文章中所述。 因此 UDP 可能会产生更好的结果，如果只是想要测试的最大吞吐量。

有关详细信息，请参阅以下文章：

- [Expressroute 网络性能故障排除](https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-network-performance)

- [如何验证虚拟网络的 VPN 吞吐量](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-validate-throughput-to-vnet)

### <a name="detect-inefficient-tcp-behaviors"></a>检测低效 TCP 行为

在数据包捕获，Azure 客户可能会看到可能表示网络性能问题的 TCP 标志 （SACK、 重复确认，重新传输和快速重新传输） 的 TCP 数据包。 这些数据包明确指出网络效率低下而导致的数据包丢失。 但数据包丢失并不一定是由 Azure 性能问题导致。 性能问题可能是应用程序的问题、 操作系统问题或其他可能不直接与 Azure 平台的问题的结果。

此外，请注意，某些重新传输和重复确认是在网络上正常。 TCP 协议生成可靠。 证据的数据包捕获这些 TCP 数据包并不一定表明系统网络问题，除非它们是过多。

尽管如此，这些数据包类型是 TCP 吞吐量未达到其最大性能，这篇文章的其他各节所述的原因的指示。

## <a name="next-steps"></a>后续步骤

现在，你已了解有关 Azure Vm 的 TCP/IP 性能优化，你可能想要阅读有关的其他注意事项[规划虚拟网络](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)或[了解有关连接和配置虚拟网络的详细信息](https://docs.microsoft.com/azure/virtual-network/).
