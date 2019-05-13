---
title: include 文件
description: include 文件
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 12/21/2018
ms.author: cynthn;davberg
ms.custom: include file
ms.openlocfilehash: ca0c45b5d74140e0398595070df71da669caab53
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65468333"
---
Azure 计算单位 (ACU) 这一概念提供一种比较 Azure SKU 的计算 (CPU) 性能的方法。 这有助于轻松确定最有可能满足性能需求的 SKU。  ACU 目前在小型 (Standard_A1) VM 上标准为 100，而所有其他 SKU 表示 SKU 在运行标准基准测试时大约可以有多快。 

> [!IMPORTANT]
> ACU 只是一种规则。  工作负荷的结果可能会有所不同。 
> 
> 

<br>

| SKU 系列 | ACU \ vCPU | vCPU：内核 |
| --- | --- |---|
| [A0](../articles/virtual-machines/windows/sizes-general.md) |50 | 1:1 |
| [A1 - A4](../articles/virtual-machines/windows/sizes-general.md) |100 | 1:1 |
| [A5 - A7](../articles/virtual-machines/windows/sizes-general.md) |100 | 1:1 |
| [A1_v2 - A8_v2](../articles/virtual-machines/windows/sizes-general.md) |100 | 1:1 |
| [A2m_v2 - A8m_v2](../articles/virtual-machines/windows/sizes-general.md) |100 | 1:1 |
| [A8 - A11](../articles/virtual-machines/windows/sizes-hpc.md) |225* | 1:1 |
| [D1 - D14](../articles/virtual-machines/windows/sizes-general.md) |160 - 250 | 1:1 |
| [D1_v2 - D15_v2](../articles/virtual-machines/windows/sizes-general.md) |210 - 250* | 1:1 |
| [DS1 - DS14](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |160 - 250 | 1:1 |
| [DS1_v2 - DS15_v2](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |210 - 250* | 1:1 |
| [D_v3](../articles/virtual-machines/virtual-machines-windows-sizes-general.md) |160 - 190* | 2:1\*\*\* |
| [Ds_v3](../articles/virtual-machines/virtual-machines-windows-sizes-general.md) |160 - 190* | 2:1\*\*\* |
| [E_v3](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |160 - 190* | 2:1\*\*\*|
| [Es_v3](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |160 - 190* | 2:1\*\*\* |
| [F2s_v2 - F72s_v2](../articles/virtual-machines/windows/sizes-compute.md) |195 - 210* | 2:1\*\*\* |
| [F1 - F16](../articles/virtual-machines/windows/sizes-compute.md) |210 - 250* | 1:1 |
| [F1s - F16s](../articles/virtual-machines/windows/sizes-compute.md) |210 - 250* | 1:1 |
| [G1 - G5](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |180 - 240* | 1:1 |
| [GS1 - GS5](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) |180 - 240* | 1:1 |
| [H](../articles/virtual-machines/windows/sizes-hpc.md) |290 - 300* | 1:1 |
| [HB](../articles/virtual-machines/windows/sizes-hpc.md) |199 - 216** | 1:1 |
| [HC](../articles/virtual-machines/windows/sizes-hpc.md) |297 - 315* | 1:1 |
| [L4s - L32s](../articles/virtual-machines/windows/sizes-storage.md) |180 - 240* | 1:1 |
| [L8s_v2 - L80s_v2](../articles/virtual-machines/windows/sizes-storage.md) |150 - 175** | 2:1 |
| [M](../articles/virtual-machines/virtual-machines-windows-sizes-memory.md) | 160 - 180 | 2:1\*\*\* |

*ACU 使用 Intel® Turbo 技术来增加 CPU 频率和提升性能。  性能提升程度可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。

**ACU 使用 AMD® Boost 技术来增加 CPU 频率和提升性能。  性能提升程度可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。

***超线程，能够运行嵌套虚拟化
