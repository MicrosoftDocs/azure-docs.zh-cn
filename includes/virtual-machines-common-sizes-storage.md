---
title: include 文件
description: include 文件
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 04/17/2019
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: fb829cc5f1eef9c151a70a6479e419076e1c3b24
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509842"
---
存储优化 VM 大小可提供较高的磁盘吞吐量和 IO，非常适合用于大数据、SQL 和 NoSQL 数据库、数据仓库和大型事务数据库。  示例包括 Cassandra、MongoDB、Cloudera 和 Redis。 本文介绍关于 vCPU、数据磁盘和 NIC 的数量以及每个优化大小的本地存储吞吐量和网络带宽的信息。

Lsv2 系列具有高吞吐量、低延迟、在 [AMD EPYC &trade; 7551 处理器](https://www.amd.com/en/products/epyc-7000-series)上运行的直接映射的本地 NVMe 存储，且所有核心提升 2.55GHz，最大提升 3.0GHz。 Lsv2 系列 VM 提供同时进行的多线程配置中的 8 到 80 个 vCPU。  每个 vCPU 有 8 GiB 内存，每 8 个 vCPU 有一个 1.92TB 的 NVMe SSD M.2 设备，并且在 L80s v2 上可以使用最多 19.2TB (10x1.92TB)。

> [!NOTE]
> Lsv2 系列 Vm 进行了优化以使用直接附加到 VM，而不是使用持久性数据磁盘的节点上的本地磁盘。 这可实现工作负载的更大的 IOPS/吞吐量。 Lsv2 和 Ls 系列不支持以增加持久性数据磁盘的 IOPs 可实现本地缓存的创建。
>
> 高吞吐量和 IOPs 的本地磁盘使得 Lsv2 和 Ls 系列 Vm 非常适合用于 NoSQL 存储 Apache Cassandra 和 MongoDB 等跨多个 Vm 以实现持久性发生的单一 vm 故障时复制数据。
>
> 若要了解详细信息，请参阅[优化 Lsv2 系列虚拟机上的性能](../articles/virtual-machines/linux/storage-performance.md)。  


## <a name="lsv2-series"></a>Lsv2 系列

ACU：150-175

高级存储：支持

高级存储缓存：不受支持

| 大小          | vCPU | 内存 (GiB) | 临时磁盘<sup>1</sup> (GiB) | NVMe 磁盘<sup>2</sup> | NVMe 磁盘吞吐量<sup>3</sup>（读取 IOPS/MBps） | 非缓存的最大数据磁盘吞吐量 (IOPs/MBps)<sup>4</sup> | 最大数据磁盘 | 最大 NIC 数/预期网络带宽 (MBps) |
|---------------|-----------|-------------|--------------------------|----------------|---------------------------------------------------|-------------------------------------------|------------------------------|------------------------------| 
| Standard_L8s_v2   |  8 |  64 |  80 |  1x1.92 TB  | 400,000 / 2,000 | 8,000/160 | 16 | 2 / 3,200  | 
| Standard_L16s_v2  | 16 | 128 | 160 |  2x1.92 TB  | 800,000 / 4,000 | 16,000/320 | 32 | 4 / 6,400  | 
| Standard_L32s_v2  | 32 | 256 | 320 |  4x1.92 TB  | 1.5M / 8,000    | 32,000/640 | 32 | 8 / 12,800 | 
| Standard_L64s_v2  | 64 | 512 | 640 |  8x1.92 TB  | 2.9M / 16,000   | 64,000/1,280 | 32 | 8 / 16,600+ |
| Standard_L80s_v2  | 80 | 640 | 800 | 10x1.92TB   | 3.8M / 20,000   | 80,000/1,400 | 32 | 8 / 16,000+ |

<sup>1</sup> Lsv2 系列 VM 具有标准的基于 SCSI 的临时资源磁盘，用于 OS 分页/交换文件的使用（对于 Windows，为 D:；对于 Linux，为 /dev/sdb）。 此磁盘为每 8 个 vCPU 提供 80 GiB 的存储空间、4000 IOPS 和 80 MBps 的传输速率（例如，Standard_L80s_v2 在 40000 IOPS 和 800 MBps 下提供 800 GiB）。 这确保 NVMe 驱动器可以完全专用于应用程序的使用。 此磁盘是临时的，停止/解除分配时将丢失所有数据。

<sup>2</sup> 本地 NVMe 磁盘是临时的，如果停止/解除分配 VM，这些磁盘上的数据都将丢失。

<sup>3</sup> Hyper-V NVMe Direct 技术提供对安全映射到来宾 VM 空间的本地 NVMe 驱动器的无限制访问。  实现最大性能需要使用最新的 WS2019 版本或者 Azure 市场上的 Ubuntu 18.04 或 16.04。  写入性能因 IO 大小、驱动器负载和容量利用率而异。

<sup>4</sup> Lsv2 系列 VM 不为数据磁盘提供主机缓存，因为这不会让 Lsv2 工作负荷受益。  然而，Lsv2 VM 可以调整 Azure 的临时 VM OS 磁盘选项（最多 30 GiB）。

## <a name="size-table-definitions"></a>大小表定义

- 存储容量的单位为 GiB 或 1024^3 字节。 比较以 GB（1000^3 字节）为单位的磁盘和以 GiB（1024^3 字节）为单位的磁盘时，请记住以 GiB 为单位的容量数显得更小。 例如，1023 GiB = 1098.4 GB
- 磁盘吞吐量的单位为每秒输入/输出操作数 (IOPS) 和 MBps，其中 MBps = 10^6 字节/秒。
- 若要获得 VM 的最佳性能，应将数据磁盘数限制为每 vCPU 2 个磁盘。
- **预期的网络带宽**是指跨所有 NIC [为每个 VM 类型分配的最大聚合带宽](../articles/virtual-network/virtual-machine-network-throughput.md)，适用于所有目标。 不保证能够达到上述预期上限，但目的是提供相关指导，以便为目标应用程序选择适当的 VM 类型。 实际的网络性能将取决于许多因素，包括网络拥塞、应用程序负载和网络设置。 有关如何优化网络吞吐量的信息，请参阅[为 Windows 和 Linux 优化网络吞吐量](../articles/virtual-network/virtual-network-optimize-network-bandwidth.md)。 若要在 Linux 或 Windows 中达到预期的网络性能，可能需要选择特定的版本，或者需要优化 VM。 有关详细信息，请参阅[如何针对虚拟机吞吐量进行可靠的测试](../articles/virtual-network/virtual-network-bandwidth-testing.md)。
