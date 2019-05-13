---
title: Azure 虚拟机网络吞吐量 | Microsoft Docs
description: 了解 Azure 虚拟机网络吞吐量。
services: virtual-network
documentationcenter: na
author: steveesp
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 4/26/2019
ms.author: kumud,steveesp, mareat
ms.openlocfilehash: 9d74e53c754367ecfa63642514db93354fcadf25
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153728"
---
# <a name="virtual-machine-network-bandwidth"></a>虚拟机网络带宽

Azure 提供各种 VM 大小和类型，每一种包含的性能各不相同。 其中一种是网络吞吐量（也称带宽），以兆位/秒 (Mbps) 表示。 由于虚拟机托管在共享硬件上，因此网络容量必须在共享同一硬件的虚拟机中公平地共享。 在分配时，较大的虚拟机相对于较小的虚拟机会获得相对较多的带宽。
 
分配给每个虚拟机的网络带宽按虚拟机的传出（出站）流量计算。 从虚拟机流出的所有网络流量均计入分配限制，不管流向哪个目标。 例如，如果虚拟机的限制为 1,000 Mbps，则不管出站流量的目标是同一虚拟网络中的另一虚拟机，还是 Azure 外部，均适用该限制。
 
传入流量不直接计算，或者说不直接受到限制。 但是，其他因素（例如 CPU 和存储限制）可能会影响虚拟机处理传入数据的能力。

加速网络是一项旨在改进网络性能（包括延迟、吞吐量和 CPU 使用率）的功能。 虽然加速网络可以改进虚拟机的吞吐量，但仍受分配给该虚拟机的带宽的限制。 若要详细了解如何使用加速网络，请查看适用于 [Windows](create-vm-accelerated-networking-powershell.md) 或 [Linux](create-vm-accelerated-networking-cli.md) 虚拟机的加速网络。
 
Azure 虚拟机必须有一个（但也可能有多个）连接的网络接口。 分配给某个虚拟机的带宽是流经所有网络接口（已连接到该虚拟机）的所有出站流量的总和。 换言之，分配的带宽是针对每个虚拟机的，不管为该虚拟机连接了多少网络接口。 若要了解不同的 Azure VM 大小支持的网络接口数，请查看 Azure [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 和 [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) VM 大小。 

## <a name="expected-network-throughput"></a>预期的网络吞吐量

若要详细了解每种 VM 大小支持的预期出站吞吐量和网络接口数，请查看 Azure [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 和 [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) VM 大小。 选择一个类型（例如“通用”），然后在生成的页面上选择一个大小系列（例如“Dv2 系列”）。 每个系统都有一个表，在最后一列（名为“最大 NIC 数/预期网络性能(Mbps)”）中包含网络规格。 

吞吐量限制适用于虚拟机。 吞吐量不受以下因素影响：
- **网络接口数**:带宽限制都是累积的所有出站流量从虚拟机。
- **加速网络**:尽管该功能可以有助于流量达到已发布的限制，它不会更改此限制。
- **流量目标**:所有目标都计入出站限制。
- **协议**：所有协议的所有出站流量将计入此限制。

## <a name="network-flow-limits"></a>网络流限制

除了带宽，在任何给定时间在 VM 上存在网络连接数可能会影响其网络性能。 Azure 网络堆栈维护每个方向的名为流的数据结构中的 TCP/UDP 连接的状态。 典型的 TCP/UDP 连接将有 2 个流创建，另一个用于入站和出站方向的另一个。 

终结点之间的数据传输，需要创建多个流，除了执行数据传输。 一些示例为创建的 DNS 解析的流和创建负载均衡器运行状况探测的流。 此外请注意，网络如网关、 代理、 防火墙和虚拟设备 (Nva) 将看到正在终止在设备和由设备发起的连接创建的流。 

![通过转发设备的 TCP 会话的流数](media/virtual-machine-network-throughput/flow-count-through-network-virtual-appliance.png)

## <a name="flow-limits-and-recommendations"></a>流限制和建议

目前，Azure 网络堆栈支持具有良好的性能 250k 个总网络流的 Vm 使用大于 8 个 CPU 内核和良好性能的 Vm 使用少于 8 个 CPU 内核的 100 k 总流。 超过此限制网络性能下降正常的最大为 1 M 硬限制的其他流的总流，50 万个入站和 500 K 出站、 后的其他流将被删除。

||使用 Vm < 8 个 CPU 内核|具有 8 个 CPU 内核的 Vm|
|---|---|---|
|<b>良好的性能</b>|10 万个流 |250k 个流|
|<b>性能下降</b>|上面 100k 为单位的流|上面 250k 流|
|<b>流限制</b>|1 百万的流|1 百万的流|

度量值位于[Azure Monitor](../azure-monitor/platform/metrics-supported.md#microsoftcomputevirtualmachines)用于跟踪 VM 或 VMSS 实例上的网络流和流创建速率。

![azure-monitor-flow-metrics.png](media/virtual-machine-network-throughput/azure-monitor-flow-metrics.png)

连接建立和终止速率也会影响网络性能作为连接建立和终止共享 CPU 与数据包处理例程。 我们建议，都设为基准根据预期的流量模式和横向扩展工作负荷的工作负荷适当地以满足性能需求。 

## <a name="next-steps"></a>后续步骤

- [优化虚拟机操作系统的网络吞吐量](virtual-network-optimize-network-bandwidth.md)
- 针对虚拟机[测试网络吞吐量](virtual-network-bandwidth-testing.md)。
