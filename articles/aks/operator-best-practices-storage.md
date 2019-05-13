---
title: 操作员最佳做法 - Azure Kubernetes 服务 (AKS) 中的存储
description: 了解有关 Azure Kubernetes 服务 (AKS) 中的存储、数据加密和备份的群集操作员最佳做法
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 12/04/2018
ms.author: iainfou
ms.openlocfilehash: 7476747de31819907cf144e5a6b33cb29e1f866f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072653"
---
# <a name="best-practices-for-storage-and-backups-in-azure-kubernetes-service-aks"></a>有关 Azure Kubernetes 服务 (AKS) 中的存储和备份的最佳做法

在 Azure Kubernetes 服务 (AKS) 中创建和管理群集时，应用程序通常需要存储空间。 了解 pod 的性能需求和访问方法非常重要，这样才能为应用程序提供适当的存储。 AKS 节点大小可能会影响存储选择。 还应规划附加存储的还原过程的备份和测试方式。

本最佳做法文章重点介绍群集操作员应考虑的存储注意事项。 本文内容：

> [!div class="checklist"]
> * 有哪些类型的存储空间可用
> * 如何正确调整 AKS 节点的大小以获得存储性能
> * 卷的动态和静态预配之间的差异
> * 备份和保护数据卷的方法

## <a name="choose-the-appropriate-storage-type"></a>选择合适的存储类型

**最佳做法指南** - 了解应用程序的需求以选择合适的存储空间。 将高性能、SSD 支持的存储用于生产工作负载。 在需要多个并发连接时，规划基于网络的存储。

应用程序通常需要不同类型和速度的存储。 应用程序是否需要存储连接到各个 pod，或跨多个 pod 进行共享？ 存储是用于以只读方式访问数据还是写入大量结构化数据？ 这些存储需求决定了最合适的存储类型。

下表概述了可用的存储类型及其功能：

| 使用案例 | 卷插件 | 读/写一次 | 只读多次 | 读/写多次 |
|----------|---------------|-----------------|----------------|-----------------|
| 共享配置       | Azure 文件   | 是 | 是 | 是 |
| 结构化应用数据        | Azure 磁盘   | 是 | 否  | 否  |
| 应用数据，只读共享 | [Dysk（预览版）][dysk] | 是 | 是 | 否  |
| 非结构化的数据，文件系统操作 | [BlobFuse（预览版）][blobfuse] | 是 | 是 | 是 |

为 AKS 中的卷提供的两种主要存储类型由 Azure 磁盘或 Azure 文件支持。 为了提高安全性，两种类型的存储都默认使用 Azure 存储服务加密 (SSE) 来加密静态数据。 目前无法使用 AKS 节点级别的 Azure 磁盘加密对磁盘进行加密。

Azure 文件目前在标准性能层中可用。 Azure 磁盘在标准和高级性能层可用：

- *高级*磁盘由高性能固态硬盘 (SSD) 支持。 建议为所有生产工作负载使用高级磁盘。
- *标准*磁盘由常规旋转磁盘 (HDD) 支持，适用于存档或不经常访问的数据。

了解应用程序性能需求和访问模式，选择合适的存储层。 有关托管磁盘大小和性能层的详细信息，请参阅 [Azure 托管磁盘概述][managed-disks]

### <a name="create-and-use-storage-classes-to-define-application-needs"></a>创建和使用存储类来定义应用程序需求

所使用的存储类型是使用 Kubernetes 存储类定义的。 然后在 pod 或部署规范中引用存储类。 这些定义共同发生作用，创建合适的存储并将其连接到 pod。 有关详细信息，请参阅 [AKS 中的存储类][aks-concepts-storage-classes]。

## <a name="size-the-nodes-for-storage-needs"></a>根据存储需求调整节点大小

**最佳做法指南** - 每个节点大小都支持最大数量的磁盘。 不同大小的节点提供不同数量的本地存储和网络带宽。 根据应用程序需要进行规划，部署大小合适的节点。

AKS 节点作为 Azure VM 运行。 有不同类型和大小的 VM 可使用。 每种大小的 VM 提供不同数量的核心资源，例如 CPU 和内存。 对于每种不同的 VM 大小，存在可附加的最大磁盘数。 不同大小的 VM 用于实现最大本地和附加磁盘 IOPS（每秒的输入/输出操作）的存储性能也会有所不同。

若应用程序需要 Azure 磁盘作为其存储解决方案，请规划并选择合适的节点 VM 大小。 选择 VM 大小时，CPU 和内存量不是唯一的考量因素。 存储功能也很重要。 例如，Standard_B2ms 和 Standard_DS2_v2 大小的 VM 包含相似数量的 CPU 和内存资源。 但其潜在的存储性能不同，如下表所示：

| 节点类型和大小 | vCPU | 内存 (GiB) | 最大数据磁盘数 | 最大未缓存磁盘 IOPS | 最大未缓存吞吐量 (MBps) |
|--------------------|------|--------------|----------------|------------------------|--------------------------------|
| Standard_B2ms      | 2    | 8            | 4              | 1,920                  | 22.5                           |
| Standard_DS2_v2    | 2    | 7            | 8              | 6,400                  | 96                             |

此处，Standard_DS2_v2 可将附加磁盘数量增加一倍，并提供三到四倍的 IOPS 和磁盘吞吐量。 若只关注核心计算资源和成本，则可以选择 Standard_B2ms 大小的 VM，但存储性能较差且存在较多限制。 与应用程序开发团队沟通，了解他们的存储容量和性能需求。 为 AKS 节点选择合适的 VM 大小以满足或超出其性能需求。 定期调整应用程序的基线，以根据需要调整 VM 大小。

有关可用 VM 大小的详细信息，请参阅 [Azure 中的 Linux 虚拟机大小][vm-sizes]。

## <a name="dynamically-provision-volumes"></a>动态预配卷

**最佳做法指南** - 为减少管理开销和实现缩放，请不要以静态方式创建和分配永久性卷。 使用动态预配。 在存储类中，定义合适的回收策略，以便在删除 pod 后最大限度地减少不必要的存储成本。

当需要将存储附加到 pod 时，可以使用永久性卷。 可手动或动态创建这些永久性卷。 手动创建永久性卷会增加管理开销，并限制缩放能力。 使用动态永久性卷预配来简化存储管理，让应用程序能够根据需要缩放和扩大整体规模。

![Azure Kubernetes 服务 (AKS) 群集中的永久性卷声明](media/concepts-storage/persistent-volume-claims.png)

通过永久性卷声明 (PVC)，可根据需要动态创建存储。 基础 Azure 磁盘是根据 pod 的请求创建的。 在 pod 定义中，请求创建卷并将其附加到设计的装载路径

有关如何动态创建和使用卷，请参阅[永久性卷声明][aks-concepts-storage-pvcs]。

要了解这些卷的运行原理，请参阅如何使用 [Azure 磁盘][dynamic-disks]或 [Azure 文件][dynamic-files]动态创建和使用永久性卷。

作为存储类定义的一部分，请设置相应的 reclaimPolicy。 删除 Pod 后且可能不再需要永久性卷时，此 reclaimPolicy 可控制基础 Azure 存储资源在此情况下的行为。 可删除基础存储资源，也可保留基础存储资源以便与未来的 Pod 配合使用。 可将 reclaimPolicy 设置为“保留”或“删除”。 了解应用程序需求，并定期检查存储，以最大限度地减少未利用的存储量和费用。

有关存储类选项的详细信息，请参阅[存储回收策略][reclaim-policy]。

## <a name="secure-and-back-up-your-data"></a>保护和备份数据

**最佳实践指南**-备份你的数据为您的存储类型，如 Velero 或 Azure Site Recovery 使用相应的工具。 验证这些备份的完整性和安全性。

当应用程序存储和使用永久存储在磁盘或文件中的数据时，需要定期备份或创建数据的快照。 Azure 磁盘可以使用内置快照技术。 在执行快照操作之前，可能需要一个挂钩，让应用程序能够刷新磁盘的写入操作。 [Velero] [ velero]可以备份以及其他群集资源和配置的永久性卷。 如果无法[从应用程序中删除状态][remove-state]，请从永久性卷备份数据并定期测试还原操作以验证数据完整性和所需的过程。

了解不同数据备份方法的局限性，以及是否需要在快照之前使数据处于静默状态。 数据备份不一定能恢复应用程序的群集部署环境。 有关这些方案的详细信息，请参阅 [AKS 中的业务连续性和灾难恢复的最佳做法][best-practices-multi-region]。

## <a name="next-steps"></a>后续步骤

本文重点介绍了 AKS 中的最佳存储做法。 有关 Kubernetes 中存储基础知识的详细信息，请参阅 [AKS 中应用程序的存储概念][aks-concepts-storage]。

<!-- LINKS - External -->
[velero]: https://github.com/heptio/velero
[dysk]: https://github.com/Azure/kubernetes-volume-drivers/tree/master/flexvolume/dysk
[blobfuse]: https://github.com/Azure/azure-storage-fuse

<!-- LINKS - Internal -->
[aks-concepts-storage]: concepts-storage.md
[vm-sizes]: ../virtual-machines/linux/sizes.md
[dynamic-disks]: azure-disks-dynamic-pv.md
[dynamic-files]: azure-files-dynamic-pv.md
[reclaim-policy]: concepts-storage.md#storage-classes
[aks-concepts-storage-pvcs]: concepts-storage.md#persistent-volume-claims
[aks-concepts-storage-classes]: concepts-storage.md#storage-classes
[managed-disks]: ../virtual-machines/linux/managed-disks-overview.md
[best-practices-multi-region]: operator-best-practices-multi-region.md
[remove-state]: operator-best-practices-multi-region.md#remove-service-state-from-inside-containers
