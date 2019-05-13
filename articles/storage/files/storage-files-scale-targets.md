---
title: Azure 文件可伸缩性和性能目标 | Microsoft Docs
description: 了解 Azure 文件的可伸缩性和性能目标信息，包括容量、请求速率以及入站和出站带宽限制。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 5/5/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: c4928050f945ac88dd1f86e2a13b5d26d385e55a
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190014"
---
# <a name="azure-files-scalability-and-performance-targets"></a>Azure 文件可伸缩性和性能目标

[Azure 文件](storage-files-introduction.md)在云中提供完全托管的文件共享，这些共享项可通过行业标准 SMB 协议进行访问。 本文讨论了 Azure 文件和 Azure 文件同步的可伸缩性和性能目标。

此处列出的可伸缩性和性能目标是高端目标，但可能会受部署中的其他变量影响。 例如，除了受限于托管着 Azure 文件服务的服务器之外，针对文件的吞吐量还可能会受限于可变的网络带宽。 强烈建议你对使用模式进行测试，以确定 Azure 文件的可伸缩性和性能是否满足你的要求。 随着时间的推移，我们也一直在努力提高这些限制。 如果希望我们提高某些限制，请尽管通过下面的评论或者通过 [Azure 文件 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) 向我们提供反馈。

## <a name="azure-storage-account-scale-targets"></a>Azure 存储帐户规模目标

Azure 文件共享的父资源是 Azure 存储帐户。 存储帐户表示 Azure 中的一个存储池，该存储池可供包括 Azure 文件在内多个存储服务用来存储数据。 在存储帐户中存储数据的其他服务有 Azure Blob 存储、Azure 队列存储和 Azure 表存储。 以下目标适用于在存储帐户中存储数据的所有存储服务：

[!INCLUDE [azure-storage-limits](../../../includes/azure-storage-limits.md)]

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

> [!Important]  
> 常规用途存储帐户从其他存储服务的利用率会影响 Azure 文件共享存储帐户中。 例如，如果由于 Azure Blob 存储而达到了最大存储帐户容量，则将无法在 Azure 文件共享上创建新文件，即使 Azure 文件共享低于最大共享大小。

## <a name="azure-files-scale-targets"></a>Azure 文件规模目标

### <a name="premium-scale-targets"></a>高级缩放目标

有三个类别的限制要考虑的高级文件共享： 存储帐户、 共享和文件。

例如：单个共享可以实现 100,000 IOPS 和单个文件可以扩展最多 5,000 个 IOPS。 因此，例如，如果您有一个共享中的三个文件，就可以从该共享的最大 IOPS 是 15000。

### <a name="premium-filestorage-account-limits"></a>高级文件存储帐户限制

高级文件共享中一个名为特殊的存储帐户预配**文件存储 （预览版）**。 此帐户具有比使用标准文件共享的存储帐户的略有不同的缩放目标。 有关存储帐户规模目标，请参阅中的表[Azure 存储帐户规模目标](#azure-storage-account-scale-targets)部分。

> [!IMPORTANT]
> 存储帐户限制适用于所有共享。 最多缩放的存储帐户的最大值才可实现，如果只有一个共享每个存储帐户。

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

## <a name="azure-file-sync-scale-targets"></a>Azure 文件同步规模目标

对于 Azure 文件同步，在设计时我们已尽最大努力来实现不受限的使用，但这并非始终可能。 下表指明了我们的测试边界以及哪些目标实际上是硬性限制：

[!INCLUDE [storage-sync-files-scale-targets](../../../includes/storage-sync-files-scale-targets.md)]

### <a name="azure-file-sync-performance-metrics"></a>Azure 文件同步性能指标

由于 Azure 文件同步代理在连接到 Azure 文件共享的 Windows Server 计算机上运行，因此有效的同步性能取决于基础结构中的多个因素：Windows Server 和基础磁盘配置、服务器与 Azure 存储之间的网络带宽、文件大小、总数据集大小以及数据集上的活动。 由于 Azure 文件同步在文件级别工作，因此可以更好地通过每秒处理的对象数（文件和目录）这一指标来度量基于 Azure 文件同步的解决方案的性能特征。

对于 Azure 文件同步，性能在两个阶段至关重要：

1. **初始的一次性预配**：若要优化初始预配的性能，请参阅[使用 Azure 文件同步进行载入](storage-sync-files-deployment-guide.md#onboarding-with-azure-file-sync)来了解最佳部署详细信息。
2. **持续同步**：在 Azure 文件同步中初次植入数据后，Azure 文件同步会使多个终结点保持同步。

为帮助你计划每个阶段的部署，下面提供了在采用某个配置的系统上进行内部测试期间观察到的结果

| 系统配置 |  |
|-|-|
| CPU | 64 个带 64 MiB L3 高速缓存的虚拟内核 |
| 内存 | 128 GiB |
| 磁盘 | 采用 RAID 10 且带有以电池供电的高速缓存的 SAS 磁盘 |
| 网络 | 1 Gbps 网络 |
| 工作负荷 | 常规用途文件服务器|

| 初始的一次性预配  |  |
|-|-|
| 对象数 | 2500 万个对象 |
| 数据集大小| ~4.7 TiB |
| 平均文件大小 | ~ 200 KiB (最大的文件：100 GiB） |
| 上传吞吐量 | 每秒 20 个对象 |
| 命名空间下载吞吐量* | 每秒 400 个对象 |

*当创建新的服务器终结点时，Azure 文件同步代理不下载任何文件内容。 它将首先同步整个命名空间，然后触发后台调用来将文件整体下载，或者根据服务器终结点上设置的云分层策略进行下载（如果启用了云分层）。

| 持续同步  |   |
|-|--|
| 同步的对象数| 125,000 个对象（~1% 的改动） |
| 数据集大小| 50 GiB |
| 平均文件大小 | ~500 KiB |
| 上传吞吐量 | 每秒 30 个对象 |
| 完整下载吞吐量* | 每秒 60 个对象 |

*如果启用了云分层，则性能可能更好，因为只会下载一部分文件数据。 只有当已缓存文件的数据在任何终结点上发生更改时，Azure 文件同步才会下载这些数据。 对于任何已分层的或新创建的文件，代理不会下载文件数据，而仅会将命名空间同步到所有服务器终结点。 代理还支持在用户访问已分层文件时下载这些文件的一部分。 

> [!Note]  
> 上述数字不代表你会实现的性能。 实际性能将取决于本部分开头列出的多个因素。

作为常规部署指南，应当牢记以下几件事情：

- 对象吞吐量大约按服务器上的同步组数量成比例缩放。 在服务器上将数据拆分到多个同步组会实现更好的吞吐量，吞吐量还受限于服务器和网络。
- 对象吞吐量与每秒 MiB 数这一吞吐量成反比。 对于较小的文件，每秒处理的对象数这一吞吐量较高，但每秒 MiB 数这一吞吐量较低。 相反，对于较大的文件，每秒处理的对象数较少，但每秒 MiB 数这一吞吐量较高。 每秒 MiB 数这一吞吐量受限于 Azure 文件规模目标。

## <a name="see-also"></a>另请参阅

- [规划 Azure 文件部署](storage-files-planning.md)
- [规划 Azure 文件同步部署](storage-sync-files-planning.md)
- [其他存储服务的可伸缩性和性能目标](../common/storage-scalability-targets.md)
