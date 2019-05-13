---
title: 规划 Azure 文件部署 | Microsoft Docs
description: 了解规划 Azure 文件部署时应考虑的问题。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 04/25/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 82c0aa53fa8905e0e58cb784a478ade474ec5601
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232762"
---
# <a name="planning-for-an-azure-files-deployment"></a>规划 Azure 文件部署

[Azure 文件](storage-files-introduction.md)在云中提供完全托管的文件共享，这些共享项可通过行业标准 SMB 协议进行访问。 由于 Azure 文件是完全托管的，因此在生产方案中对其进行部署比部署和管理文件服务器或 NAS 设备简单得多。 本文介绍在组织内部署 Azure 文件共享以供生产使用时应考虑的主题。

## <a name="management-concepts"></a>管理概念

 下图说明了 Azure 文件管理构造：

![文件结构](./media/storage-files-introduction/files-concepts.png)

* **存储帐户**：对 Azure 存储进行的所有访问都要通过存储帐户完成。 有关存储帐户容量的详细信息，请参阅[可伸缩性和性能目标](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

* **共享**：文件存储共享是 Azure 中的 SMB 文件共享。 所有目录和文件都必须在父共享中创建。 一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到文件共享的 5TiB 总容量为止。

* **目录**：可选的目录层次结构。

* **文件**：共享中的文件。 文件大小最大可以为 1 TiB。

* **URL 格式**：对于使用文件 REST 协议向 Azure 文件共享提出的请求，可采用以下 URL 格式对文件进行寻址：

    ```
    https://<storage account>.file.core.windows.net/<share>/<directory>/<file>
    ```

## <a name="data-access-method"></a>数据访问方法

Azure 文件提供两个内置的简便数据访问方法，用户可单独使用或结合使用这些方法来访问数据：

1. **直接云访问**：可使用行业标准服务器消息块 (SMB) 协议或通过文件 REST API，由 [Windows](storage-how-to-use-files-windows.md)、[macOS](storage-how-to-use-files-mac.md) 和/或 [Linux](storage-how-to-use-files-linux.md) 装载任意 Azure 文件共享。 使用 SMB，可直接在 Azure 中的文件共享上读取和写入共享文件。 若要装载在 Azure VM 上，操作系统中的 SMB 客户端必须至少支持 SMB 2.1。 若要装载在本地（例如用户工作站），工作站支持的 SMB 客户端必须至少支持 SMB 3.0（已加密）。 除 SMB 以外，新应用程序或服务可通过文件 REST 直接访问文件共享，该文件 REST 为软件开发提供简单可缩放的应用程序编程接口。
2. **Azure 文件同步**：可使用 Azure 文件同步将共享复制到本地或 Azure 中的 Windows Server。 用户可通过 SMB 或 NFS 共享等 Windows Server 访问文件共享。 这适用于要在远离 Azure 数据中心的位置访问和修改数据的方案，例如分支机构方案。 可在多个 Windows Server 终结点（例如多个分支机构）之间复制数据。 最后，可将数据分层到 Azure 文件，以便所有数据仍可通过 Server 进行访问，但 Server 没有完整的数据副本。 相反，数据由用户打开时会被无缝召回。

下表说明了用户和应用程序如何访问 Azure 文件共享：

| | 直接云访问 | Azure 文件同步 |
|------------------------|------------|-----------------|
| 需使用哪些协议？ | Azure 文件支持 SMB 2.1、SMB 3.0 和文件 REST API。 | 通过 Windows Server 上支持的任意协议（SMB、NFS、FTPS 等）访问 Azure 文件共享 |  
| 在何处运行工作负荷？ | **在 Azure 中**：Azure 文件支持直接访问数据。 | **网络速度慢的本地文件共享**：Windows、Linux 和 macOS 客户端可以将本地 Windows 文件共享装载为 Azure 文件共享的快速缓存。 |
| 需要何种级别的 ACL？ | 共享和文件级别。 | 共享、文件和用户级别。 |

## <a name="data-security"></a>数据安全

Azure 文件提供可确保数据安全的几个内置选项：

* 支持以下两种在线协议加密：SMB 3.0 加密和通过 HTTPS 的文件 REST。 默认情况下： 
    * 支持 SMB 3.0 加密的客户端发送和接收数据通过加密通道。
    * 不支持 SMB 3.0 加密的客户端可以通过 SMB 2.1 或 SMB 3.0 通信数据中心内部的但未加密。 不允许 SMB 客户端通过无加密功能的 SMB 2.1 或 SMB 3.0 进行数据中心内通信。
    * 客户端可以通过 HTTP 或 HTTPS 与文件 REST 通信。
* 静态加密（[Azure 存储服务加密](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)）：存储服务加密 (SSE) 对所有存储帐户启用。 静态数据使用完全托管的密钥进行加密。 静态加密不会增加存储成本，也不会降低性能。 
* 加密数据在传输中的可选要求：选定后，Azure 文件拒绝通过未加密通道访问数据。 具体而言，仅允许具有加密连接的 HTTPS 和 SMB 3.0。

    > [!Important]  
    > 要求安全传输数据将导致较早的 SMB 客户端无法与 SMB 3.0 通信，进而造成加密失败。 有关详细信息，请参阅[在 Windows 上装载](storage-how-to-use-files-windows.md)、[在 Linux 上装载](storage-how-to-use-files-linux.md)和[在 macOS 上装载](storage-how-to-use-files-mac.md)。

为了实现最大安全性，强烈建议始终启用这两个静态加密功能，并在使用新式客户端访问数据时启用数据传输加密。 例如，如需在仅支持 SMB 2.1 的 Windows Server 2008 R2 VM 上装载共享，则需要允许存储帐户接受未加密的流量，因为 SMB 2.1 不支持加密。

如果使用 Azure 文件同步访问 Azure 文件共享，我们将始终使用 HTTPS 和加密的 SMB 3.0 将数据同步到 Windows Server，而不考虑是否需要对静态数据加密。

## <a name="file-share-performance-tiers"></a>文件共享性能层

Azure 文件提供两个性能层： 标准和高级。

* **标准文件共享**由可轮转的硬盘驱动器 (HDD) 提供支持，这些驱动器为对性能波动不太敏感的 IO 工作负荷（例如，常规用途文件共享和开发/测试环境）提供可靠的性能。 标准文件共享只能在即用即付计费模型下使用。
* **高级文件共享（预览版）** 由固态磁盘 (SSD) 提供支持，对于大多数 IO 操作，对于大多数 IO 密集型工作负荷，这些磁盘可以提供稳定的高性能和低延迟，延迟为以个位数计的毫秒数。 这使得它们适合于各种各样的工作负荷，例如数据库、网站托管、开发环境，等等。高级文件共享只能在预配的计费模型下使用。 高级文件共享使用的一种部署模式不同于标准文件共享。

Azure 备份是可用于高级文件共享和 Azure Kubernetes 服务中 1.13 版及更高版本支持高级文件共享。

如果你想要了解如何创建高级文件共享，请参阅有关该主题的我们的文章:[如何创建 Azure 高级文件存储帐户](storage-how-to-create-premium-fileshare.md)。

目前，你不能直接转换之间的标准文件共享和高级文件共享。 如果你想要切换到任一层，必须在该层中创建新的文件共享并手动将数据从原始共享复制到你创建的新共享。 您可以执行此操作使用任何支持的 Azure 文件复制工具，例如 AzCopy。

> [!IMPORTANT]
> 高级文件共享仍处于预览状态，才可启用 LRS，并可提供存储帐户的大多数区域中。 若要查找在您所在地区是否当前可用高级文件共享，请参阅[区域的可用产品](https://azure.microsoft.com/global-infrastructure/services/?products=storage)适用于 Azure 的页。

### <a name="provisioned-shares"></a>预配的共享

高级文件共享 （预览版） 预配基于固定的 GiB/IOPS/吞吐量比率。 对于预配的每个 GiB，将向该共享分配 1 IOPS 和 0.1 MiB/秒的吞吐量，最多可达每个共享的最大限制。 允许的最小预配为 100 GiB 以及最小的 IOPS/吞吐量。

最大限度地提供服务时，对于预配的存储，所有共享都可以突增到每 GiB 3 IOPS，持续 60 分钟或更长时间，具体取决于共享大小。 新共享将根据预配的容量以完全突增额度开始。

以 1 GiB 为增量，必须预配共享。 最小大小为 100 GiB 下, 一步大小为 101 GiB，依此类推。

> [!TIP]
> 基线 IOPS = 1 * 预配 GiB。 （最多 max 100,000 IOPS)。
>
> 限制 = 3 * 基线 IOPS。 （最多 max 100,000 IOPS)。
>
> 出口率 = 60 MiB/秒 + 0.06 * 预配 GiB
>
> 入口速率 = 40 MiB/秒 + 0.04 * 预配 GiB

共享的大小可以在任何时间增加，但仅在 24 小时，自上次增加后，可以减小。 等待 24 小时内，而无需将大小增加之后，可以减少无数次，随时进行，直至再次增加的共享大小。 大小更改后的几分钟内，IOPS/吞吐量规模更改将生效。

就可以减少使用 GiB 下面你预配共享的大小。 如果这样做，将不会丢失数据，但是，您仍将支付所用的大小，并获得预配共享，不使用的大小的性能 （基线 IOPS、 吞吐量和 IOPS 迸发）。

下表说明了这些公式的预配的共享大小的几个示例：

|容量 (GiB) | 基线 IOPS | 迸发 IOPS | 出口 （MiB/秒） | 入口 （MiB/秒） |
|---------|---------|---------|---------|---------|
|100         | 100     | 最多 300     | 66   | 44   |
|500         | 500     | 最多包含 1,500 位   | 90   | 60   |
|1,024       | 1,024   | 最多 3,072   | 122   | 81   |
|5,120       | 5,120   | 最多采用长度为 15,360  | 368   | 245   |
|10,240      | 10,240  | 最多 30,720  | 675 | 450   |
|33,792      | 33,792  | 最多 100,000 个 | 2,088 | 1,392   |
|51,200      | 51,200  | 最多 100,000 个 | 3,132 | 2,088   |
|102,400     | 100,000 | 最多 100,000 个 | 6,204 | 4,136   |

### <a name="bursting"></a>爆发

高级文件共享可以突增系数为 3 到其 IOPS。 爆发自动的而且运行根据信用额度系统。 爆发处理的最大努力和迸发限制并不保证数据、 文件共享可以突增*达*限制。

每当为文件共享流量低于基线 IOPS，信用额度在迸发存储桶中累积。 例如，100 GiB 共享具有 100 基线 IOPS。 如果共享上的实际流量 40 IOPS 为特定时间间隔 1 秒，60 未使用的 IOPS 将贷到迸发存储桶。 这些信用额度然后将使用更高版本时操作将超出基线 IOPs。

> [!TIP]
> 突发事件存储桶的大小 = 基线 IOPS * 2 * 3600。

共享超出基线 IOPS，并且具有迸发的存储桶中的信用额度，只要它将突发。 共享可以继续迸发只要剩余的信用额度，但共享小于 50 TiB 仅停留在最多一小时的迸发限制。 共享大于 50 TiB 从技术上讲可以超过这一个小时的限制，注册到两个小时，但这取决于所产生的迸发可修读人数。 每个 IO 超过基线 IOPS 使用一个信用额度并共享所有信用额度都用完后将返回到基线 IOPS。

共享的信用额度有三种状态：

- 文件共享使用小于基线 IOPS 时产生。
- 不断下降，当爆发文件共享。
- 当没有任何信用额度或基线时保持不变，IOPS 正在使用中。

新文件共享，请先在其迸发存储桶中的信用额度的全部数量。 如果共享 IOPS 低于基线 IOPS 因限制由服务器而不将应计迸发信用额度。

## <a name="file-share-redundancy"></a>文件共享冗余

标准的 azure 文件共享支持三个数据冗余选项： 本地冗余存储 (LRS)、 区域冗余存储 (ZRS) 和异地冗余存储 (GRS)。

Azure 文件高级共享仅支持本地冗余存储 (LRS)。

以下部分介绍了不同的冗余选项之间的差异：

### <a name="locally-redundant-storage"></a>本地冗余存储

[!INCLUDE [storage-common-redundancy-LRS](../../../includes/storage-common-redundancy-LRS.md)]

### <a name="zone-redundant-storage"></a>区域冗余存储

[!INCLUDE [storage-common-redundancy-ZRS](../../../includes/storage-common-redundancy-ZRS.md)]

### <a name="geo-redundant-storage"></a>异地冗余存储

> [!Warning]  
> 如果在 GRS 存储帐户中使用 Azure 文件共享作为云终结点，则不应启动存储帐户故障转移。 执行此操作将导致同步停止工作，并且还可能导致新分层的文件出现意外数据丢失。 对于 Azure 区域丢失，Microsoft 会以与 Azure 文件同步兼容的方式触发存储帐户故障转移。

异地冗余存储 (GRS) 通过将数据复制到距主要区域数百英里以外的次要区域，用于在给定的一年内至少为对象提供 99.99999999999999%（16 个 9）的持久性。 如果存储帐户启用了 GRS，则即使遇到区域完全停电或导致主区域不可恢复的灾难，数据也能持久保存。

如果选择使用读取访问异地冗余存储 (RA-GRS)，您应该知道，Azure 文件不支持读取访问异地冗余存储 (RA-GRS) 在任何区域中这一次。 RA-GRS 存储帐户中的文件共享的工作方式与它们在 GRS 帐户中一样，GRS 价格收费。

GRS 将数据复制到次要区域中的另一个数据中心，但仅当 Microsoft 发起了从主要区域到次要区域的故障转移时，才可读取这些数据。

对于启用了 GRS 的存储帐户，使用本地冗余存储 (LRS) 首先复制所有数据。 首先将更新提交到主要位置，并使用 LRS 复制更新。 然后，使用 GRS 以异步方式将更新复制到次要区域。 将数据写入次要位置后，还会使用 LRS 在该位置复制数据。

主要和次要区域在一个存储缩放单元内管理跨单独的容错域和升级域管理副本。 存储缩放单元是数据中心内的基本复制单元。 此级别的复制由 LRS 提供；有关详细信息，请参阅[本地冗余存储 (LRS)：Azure 存储的低成本数据冗余](../common/storage-redundancy-lrs.md)。

确定要使用哪个复制选项时，请记住以下几点：

* 区域冗余存储 (ZRS) 提供了高可用性与同步复制，可能会在某些情况下比 GRS 相比更好的选择。 有关 ZRS 的详细信息，请参阅 [ZRS](../common/storage-redundancy-zrs.md)。
* 对于异步复制，从数据写入到主要区域到数据复制到次要区域，这之间存在延迟。 发生区域性灾难时，如果无法从主要区域中恢复数据，则尚未复制到次要区域的更改可能会丢失。
* 使用 GRS 时，副本不可用于读取或写入访问，除非 Microsoft 启动到次要区域的故障转移。 如果发生故障转移，则在故障转移完成后，你将具有对该数据的读取和写入访问权限。 有关详细信息，请参阅[灾难恢复指南](../common/storage-disaster-recovery-guidance.md)。

## <a name="data-growth-pattern"></a>数据增长模式

目前，Azure 文件共享的最大大小是 5 TiB (100 TiB 高级文件共享，处于公共预览状态)。 鉴于此当前限制，必须考虑部署 Azure 文件共享时的预期数据增长。

可使用 Azure 文件同步将多个 Azure 文件共享同步到单个 Windows 文件服务器。这可确保本地的较旧、大型文件共享能够导入到 Azure 文件同步。有关详细信息，请参阅[规划 Azure 文件同步部署](storage-files-planning.md)。

## <a name="data-transfer-method"></a>数据传输方法

可通过多种简单的选项将数据从现有文件共享（例如本地文件共享）批量传输到 Azure 文件。 几种常用选项包括（非详尽列表）：

* **Azure 文件同步**：在 Azure 文件共享（“云终结点”）和 Windows 目录命名空间（“服务器终结点”）之间首次同步期间，Azure 文件同步将把现有文件共享中的所有数据复制到 Azure 文件。
* **[Azure 导入/导出](../common/storage-import-export-service.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)**：使用 Azure 导入/导出服务，可将硬盘驱动器寄送到 Azure 数据中心，从而安全地将大量数据传输到 Azure 文件共享。 
* **[Robocopy](https://technet.microsoft.com/library/cc733145.aspx)**：Robocopy 是 Windows 和 Windows Server 自带的一款知名复制工具。 Robocopy 可用于将数据传输到 Azure 文件，方法是在本地装载文件共享，然后使用装载位置作为 Robocopy 命令的目标位置。
* **[AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#upload-files-to-an-azure-file-share)**：AzCopy 是一个命令行实用程序，专用于使用具有优化性能的简单命令在 Azure 文件和 Azure Blob 存储中复制/粘贴数据。 AzCopy 适用于 Windows 和 Linux。

## <a name="next-steps"></a>后续步骤
* [规划 Azure 文件同步部署](storage-sync-files-planning.md)
* [部署 Azure 文件](storage-files-deployment-guide.md)
* [部署 Azure 文件同步](storage-sync-files-deployment-guide.md)
