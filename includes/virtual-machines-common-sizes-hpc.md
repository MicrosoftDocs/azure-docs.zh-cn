---
title: include 文件
description: include 文件
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 04/26/2019
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: f3c1a946afb7f51a1c7543dabae7f2f42ff466be
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198846"
---
Azure H 系列虚拟机 (Vm) 旨在提供领导类性能，MPI 可伸缩性和成本效率的真实 HPC 工作负荷有多种。

HB 系列 VM 针对内存带宽驱动的应用进行了优化，例如流体动力学、显式有限元素分析和天气建模等应用。 HB VM 具有 60 个 AMD EPYC 7551 处理器核心、每 CPU 核心 4 GB RAM，无超线程。 AMD EPYC 平台提供超过 260 GB/秒的内存带宽。

针对由密集计算功能，例如隐式有限元分析、 分子动力学和计算化学驱动应用程序进行了优化 HC 系列 Vm。 HB VM 具有 44 个 Intel Xeon Platinum 8168 处理器核心、每 CPU 核心 8 GB RAM，无超线程。 Intel Xeon 白金平台支持 Intel 的丰富生态系统的软件工具，例如 Intel 数学内核库。

HB 和混合连接的 Vm 功能是 100 Gb/秒中非阻塞 fat Mellanox EDR InfiniBand 树 RDMA 性能相一致的配置。 HB 和混合连接 Vm 支持标准 Mellanox/OFED 驱动程序，以便也支持所有 MPI 类型和版本，以及 RDMA 谓词。

H 系列 Vm 的高 CPU 频率或每个核心要求较大内存由驱动应用程序进行了优化。 H 系列 Vm 的功能 8 或 16 Intel Xeon E5 2667 v3 处理器个核心，7 或 14 GB 的每个 CPU 内核、 RAM 和任何超线程。 H 系列提供功能，56 Gb/秒中非阻塞 fat Mellanox FDR InfiniBand 树 RDMA 性能相一致的配置。 H 系列 Vm 支持 Intel MPI 5.x 和 MS MPI。

## <a name="hb-series"></a>HB 系列

高级存储：支持高级存储缓存：支持

| 大小 | vCPU | 处理器 | 内存 (GB) | 内存带宽 GB/秒 | 基本 CPU 频率 (GHz) | 所有核心频率 （g h z，峰值） | 单核频率 （g h z，峰值） | RDMA 性能 (GB/s) | MPI 支持 | 临时存储 (GB) | 最大数据磁盘数 | 最大以太网 Nic |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB60rs | 60 | AMD EPYC 7551 | 240 | 263 | 2.0 | 2.55 | 2.55 | 100 | 全部 | 700 | 4 | 第 |

<br>

## <a name="hc-series"></a>HC 系列

高级存储：支持高级存储缓存：支持


| 大小 | vCPU | 处理器 | 内存 (GB) | 内存带宽 GB/秒 | 基本 CPU 频率 (GHz) | 所有核心频率 （g h z，峰值） | 单核频率 （g h z，峰值） | RDMA 性能 (GB/s) | MPI 支持 | 临时存储 (GB) | 最大数据磁盘数 | 最大以太网 Nic |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HC44rs | 44 | Intel Xeon Platinum 8168 | 352 | 191 | 2.7 | 3.4 | 3.7 | 100 | 全部 | 700 | 4 | 第 |


<br>

## <a name="h-series"></a>H 系列

ACU：290-300

高级存储：不支持

高级存储缓存：不支持

| 大小 | vCPU | 处理器 | 内存 (GB) | 内存带宽 GB/秒 | 基本 CPU 频率 (GHz) | 所有核心频率 （g h z，峰值） | 单核频率 （g h z，峰值） | RDMA 性能 (GB/s) | MPI 支持 | 临时存储 (GB) | 最大数据磁盘数 | 最大以太网 Nic |
| --- | --- |--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_H8 | 8 | Intel Xeon E5 2667 v3 | 56 | 40 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x，MS MPI | 1000 | 32 | 2 |
| Standard_H16 | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 |  56 | Intel 5.x，MS MPI | 2000 | 64 | 4 |
| Standard_H8m | 8 | Intel Xeon E5 2667 v3 | 112 | 40 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x，MS MPI | 1000 | 32 | 2 |
| Standard_H16m | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x，MS MPI | 2000 | 64 | 4 |
| Standard_H16r <sup>1</sup> | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x，MS MPI | 2000 | 64 | 4 |
| Standard_H16mr <sup>1</sup> | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x，MS MPI | 2000 | 64 | 4 |

<sup>1</sup> 对于 MPI 应用程序来说，专用 RDMA 后端网络是通过 FDR InfiniBand 网络启用的，后者可以提供相当低的延迟和高带宽。

<br>
