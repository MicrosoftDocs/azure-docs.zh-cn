---
title: include 文件
description: include 文件
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/28/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 6cd0c72f94c020f9243a1a95faa799f1f798f36c
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198816"
---
# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-managed-and-unmanaged-premium-disks"></a>有关 Azure IaaS VM 磁盘以及托管和非托管高级磁盘的常见问题解答

本文将对有关 Azure 托管磁盘和 Azure 高级 SSD 盘的一些常见问题进行解答。

## <a name="managed-disks"></a>托管磁盘

什么是 Azure 托管磁盘？

托管磁盘是一种通过处理存储帐户管理来简化 Azure IaaS VM 的磁盘管理的功能。 有关详细信息，请参阅[托管磁盘概述](../articles/virtual-machines/windows/managed-disks-overview.md)。

如果从现有的 VHD（80 GB）创建标准托管磁盘，需要多少费用？

从 80 GB VHD 创建的标准托管磁盘被视为下一个可用的标准磁盘大小（S10 磁盘）。 我们按 S10 磁盘定价收费。 有关详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/storage)。

标准托管磁盘是否存在任何事务成本？

是的。 我们针对每个事务进行收费。 有关详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/storage)。

对于标准托管磁盘，是对磁盘上的数据实际大小收费还是对磁盘的预配容量收费？

我们根据磁盘的预配容量收费。 有关详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/storage)。

高级托管磁盘与非托管磁盘的定价有何不同？

高级托管磁盘的定价与高级非托管磁盘的定价相同。

是否可以更改托管磁盘的存储帐户类型（标准或高级）？

是的。 可以使用 Azure 门户、PowerShell 或 Azure CLI 更改托管磁盘的存储帐户类型。

是否可以使用 Azure 存储帐户中的 VHD 文件以不同的订阅创建托管磁盘？

是的。

是否可以使用 Azure 存储帐户中的 VHD 文件在不同的区域中创建托管磁盘？

不是。

客户使用托管磁盘是否存在任何规模限制？

托管磁盘取消了与存储帐户相关的限制。 但是，订阅的最大限制为每个区域、每个磁盘类型 50,000 个托管磁盘。

是否可以生成托管磁盘的增量快照？

不是。 当前的快照功能可提供托管磁盘的完整副本。

可用性集中的 VM 是否可以同时包含托管和非托管磁盘？

不是。 可用性集中的 VM 必须全部使用托管磁盘或全部使用非托管磁盘。 创建可用性集时，可以选择要使用的磁盘类型。

托管磁盘是否是 Azure 门户中的默认选项？

是的。

是否可以创建一个空托管磁盘？

是的。 可创建空磁盘。 可独立于 VM 创建托管磁盘，例如，不需要将磁盘附加到 VM。

什么是使用托管磁盘的可用性集的支持容错域计数？

使用托管磁盘的可用性集的支持容错域计数为 2 或 3，具体取决于它所在的区域。

如何设置用于诊断的标准存储帐户？

设置 VM 诊断的专用存储帐户。

托管磁盘支持哪类基于角色的访问控制？

托管磁盘支持三个密钥默认角色：

* 所有者：可管理一切内容（包括访问权限）
* 参与者：可管理除访问权限以外的一切内容
* 读者：可查看一切内容，但不可作出更改

是否可将托管磁盘复制或导出到专用存储帐户？

可以为托管磁盘生成只读共享访问签名 (SAS) URI，使用它将内容复制到专用存储帐户或本地存储。 可以通过 Azure 门户、Azure PowerShell、Azure CLI 或 [AzCopy](../articles/storage/common/storage-use-azcopy.md) 使用 SAS URI

是否可以创建托管磁盘副本？

客户可以生成托管磁盘的快照，并使用快照创建另一个托管磁盘。

是否仍支持非托管磁盘？

是的，非托管磁盘和托管磁盘均受支持。 我们建议你对新的工作负荷使用托管磁盘，并将当前的工作负荷迁移到托管磁盘。

是否可以在同一 VM 上归置非托管和托管磁盘？

不。

**如果我创建一个 128 GB 的磁盘，然后将大小增加到 130 gibibyte 计算的 (GiB) 收费的下一步的磁盘大小 (256 GiB)？**

可以。

是否可以创建本地冗余存储、异地冗余存储和区域冗余存储托管磁盘？

Azure 托管磁盘当前仅支持本地冗余存储托管磁盘。

是否可以收缩或缩小托管磁盘？

不是。 目前，不支持此功能。

是否可以在磁盘上中断租用？

不可以。 目前不支持此功能，因为租用的作用是防止磁盘在使用时被意外删除。

当使用专用（未使用系统准备工具创建或未通用化）操作系统磁盘预配 VM 时，是否可以更改计算机名称属性？

不可以。 无法更新计算机名称属性。 新 VM 从创建操作系统磁盘时所用的父 VM 继承该属性。 

在哪里可找到用于使用托管磁盘创建 VM 的示例 Azure 资源管理器模板？
* [使用托管磁盘的模板列表](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md)
* https://github.com/chagarw/MDPP

**基于 blob 创建磁盘时，与该源 blob 之间是否会一直保持任何现有关系？**

否，创建新磁盘时，它是该 blob 在该时间的完整独立副本，两者之间没有联系。 如果你愿意，可以在创建磁盘后删除源 blob，这对新创建的磁盘没有任何影响。

**创建托管或非托管磁盘后，是否可以对其进行重命名？**

对于托管磁盘，无法对其进行重命名。 但是，可以对非托管磁盘进行重命名，只要它当前未附加到 VHD 或 VM。

**可否在 Azure 磁盘上使用 GPT 分区？**

GPT 分区仅可在数据磁盘上使用，而不可在操作系统磁盘上使用。 操作系统磁盘必须使用 MBR 分区样式。

## <a name="uploading-to-a-managed-disk"></a>上传到托管磁盘

**可以将数据上载到现有托管磁盘？**

否上, 传只能在创建与新的空磁盘过程**ReadyToUpload**状态。

**可以将附加磁盘到 VM 时处于上载状态？**

不。

**能否将上传状态中的托管磁盘的快照？**

不。

## <a name="standard-ssd-disks"></a>标准 SSD 盘

Azure 标准 SSD 盘是什么？
标准 SSD 盘是受固态介质支持的标准磁盘，经过优化而作为在较低 IOPS 级别需要一致性能的工作负载的高性价比存储。

<a id="standard-ssds-azure-regions"></a>**当前支持标准 SSD 盘的区域有哪些？**
所有 Azure 区域现在都支持标准 SSD 盘。

**使用标准 SSD 时是否可以使用 Azure 备份？**
是的，Azure 备份现已可用。

如何创建标准 SSD 盘？
可以使用 Azure 资源管理器模板、SDK、PowerShell 或 CLI 创建标准 SSD 盘。 以下为创建标准 SSD 盘时资源管理器模板中所需的参数：

* Microsoft.Compute 的 apiVersion 必须设置为 `2018-04-01`（或更高）
* 将 managedDisk.storageAccountType 指定为 `StandardSSD_LRS`

以下示例显示了使用标准 SSD 盘的 VM 的 properties.storageProfile.osDisk 部分：

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

有关如何使用模板创建标准 SSD 盘的完整模板示例，请参阅[使用标准 SSD 数据磁盘从 Windows 映像创建 VM](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/)。

是否可以将现有磁盘转换为标准 SSD？
是的，你可以。 请参阅[将 Azure 托管磁盘存储从标准转换为高级，反之亦然](https://docs.microsoft.com/azure/virtual-machines/windows/convert-disk-storage)，以了解有关转换托管磁盘的常规指南。 此外，使用以下值将磁盘类型更新为标准 SSD。
-AccountType StandardSSD_LRS

**使用标准 SSD 盘而不使用 HDD 的好处是什么？**
与 HDD 磁盘相比，标准 SSD 盘可以提供更好的延迟、一致性、可用性和可靠性。 因此，应用程序工作负荷可以更平稳地在标准 SSD 上运行。 注意，高级 SSD 盘是适用于大多数 IO 密集型生产工作负荷的建议解决方案。

是否可将标准 SSD 用作非托管磁盘？
不可以，标准 SSD 盘仅可用作托管磁盘。

标准 SSD 磁盘是否支持“单实例 VM SLA”？
不是，标准 SSD 没有单实例 VM SLA。 将高级 SSD 磁盘用于单实例 VM SLA。

## <a name="migrate-to-managed-disks"></a>迁移到托管磁盘

**迁移对托管磁盘性能是否有影响？**

迁移涉及将磁盘从一个存储位置移动到另一个存储位置。 这是通过在后台复制数据来安排的，可能需要花费数小时才能完成，通常少于 24 小时，具体取决于磁盘中的数据量。 在此期间，由于一些读取可能被重定向到原始位置，所以应用程序可能会经历比平常更高的读取延迟，并且可能需要花费更长时间才能完成。 在此期间，对写入延迟没有影响。  

迁移到托管磁盘之前/之后，需要在现有的 Azure 备份服务配置中进行哪些更改？

不需要进行任何更改。

在迁移之前通过 Azure 备份服务创建的 VM 备份是否可继续工作？

是的，备份可以顺利工作。

迁移到托管磁盘之前/之后，需要在现有的 Azure 磁盘加密配置中进行哪些更改？

不需要进行任何更改。

**自动的迁移现有的虚拟机规模集的设置从非托管磁盘为支持托管磁盘？**

不。 可以使用包含非托管磁盘的旧规模集中的映像创建包含托管磁盘的新规模集。

是否可以通过迁移到托管磁盘之前创建的页 Blob 快照创建托管磁盘？

不可以。 可将页 Blob 快照导出为页 Blob，然后从导出的页 Blob 创建托管磁盘。

是否可将 Azure Site Recovery 保护的本地计算机故障转移到包含托管磁盘的 VM？

是的，可以选择故障转移到包含托管磁盘的 VM。

迁移是否影响 Azure Site Recovery 通过 Azure 到 Azure 复制保护的 Azure VM？

可以。 目前，Azure 站点恢复 Azure 到 Azure 保护的 Vm 使用托管磁盘现推出 GA 服务。

是否可以迁移位于存储帐户中现在或以前已加密的 VM 的非托管磁盘迁移到托管磁盘？

是

## <a name="managed-disks-and-storage-service-encryption"></a>托管磁盘和存储服务加密

创建托管磁盘时，是否会默认启用 Azure 存储服务加密？

是的。

加密密钥由谁管理？

Microsoft 管理加密密钥。

是否可以为托管磁盘禁用存储服务加密？

不可以。

存储服务加密是否仅适用于特定区域？

不可以。 它适用于托管磁盘可用的所有区域。 托管磁盘适用于所有公共区域和德国。 这也适用于中国，但是，仅适用于 Microsoft 托管密钥，不适用于客户托管密钥。

如何确定我的托管磁盘是否已加密？

你可以从 Azure 门户、Azure CLI 和 PowerShell 确定托管磁盘的创建时间。 如果时间是在 2017 年 6 月 9 日之后，则磁盘已加密。

如何对 2017 年 6 月 10 日之前创建的现有磁盘加密？

自 2017 年 6 月 10 日起，写入到现有托管磁盘的新数据会自动加密。 我们还打算对现有数据进行加密，且在后台以异步方式加密。 如果必须立即对现有数据进行加密，请创建磁盘的副本。 将对新磁盘进行加密。

* [使用 Azure CLI 复制托管磁盘](../articles/virtual-machines/scripts/virtual-machines-linux-cli-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)
* [使用 PowerShell 复制托管磁盘](../articles/virtual-machines/scripts/virtual-machines-windows-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)

是否已加密托管快照和映像？

是的。 2017 年 6 月 9 日之后创建的所有托管快照和映像均会自动加密。 

是否可以将 VM 的位于存储帐户且现在或以前已加密的非托管磁盘转换为托管磁盘？

是

是否同时会加密从托管磁盘或快照导出的 VHD？

不可以。 但如果将 VHD 从加密托管磁盘或快照导出到加密存储帐户，则会对其进行加密。 

## <a name="premium-disks-managed-and-unmanaged"></a>高级磁盘：托管和非托管

如果 VM 使用支持高级 SSD 盘的大小系列（比如 DSv2），是否可以同时附加高级和标准数据磁盘？ 

是的。

是否可以同时将高级和标准数据磁盘附加到不支持高级 SSD 盘的大小系列，例如 D、Dv2、G 或 F 系列？

不可以。 只可以将标准数据磁盘附加到不使用支持高级 SSD 盘的大小系列的 VM。

如果从现有的 VHD (80 GB) 创建高级数据磁盘，需要多少费用？

从 80 GB VHD 创建的高级数据磁盘被视为下一个可用的高级磁盘大小（P10 磁盘）。 我们按 P10 磁盘定价收费。

使用高级 SSD 盘时是否存在事务成本？

每个磁盘大小都有固定成本，其根据 IOPS 和吞吐量的特定限制进行预配。 其他成本包括出站带宽和快照容量（如果适用）。 有关详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/storage)。

可从磁盘缓存获取的 IOPS 和吞吐量限制是多少？

DS 系列的缓存和本地 SSD 合并限制是每个核心 4,000 IOPS，以及每个核心每秒 33 MiB。 GS 系列提供每个核心 5,000 IOPS，每个核心每秒 50 MiB。

托管磁盘 VM 是否支持本地 SSD？

本地 SSD 是托管磁盘 VM 随附的临时存储。 临时存储不需要额外的成本。 建议不要使用此本地 SSD 来存储应用程序数据，因为这些数据不会永久保存在 Azure Blob 存储中。

在高级磁盘上使用 TRIM 是否有任何影响？

在高级或标准磁盘的 Azure 磁盘上使用 TRIM 没有负面影响。

## <a name="new-disk-sizes-managed-and-unmanaged"></a>新磁盘大小：托管和非托管

**操作系统和数据磁盘支持的最大托管磁盘大小是多少？**

Azure 支持的操作系统磁盘的分区类型是主启动记录 (MBR)。 MBR 格式支持的磁盘最大大小为 2 TiB。 Azure 支持的操作系统磁盘的最大大小为 2 TiB。 Azure 支持最多 32 TiB 的全局 Azure，Azure 主权云 4 TiB 中的托管的数据磁盘。

**操作系统和数据磁盘支持的最大非托管磁盘大小是多少？**

Azure 支持的操作系统磁盘的分区类型是主启动记录 (MBR)。 MBR 格式支持的磁盘最大大小为 2 TiB。 Azure 支持的操作系统非托管磁盘的最大大小为 2 TiB。 Azure 支持的非托管数据磁盘最大大小为 4 TiB。

支持的最大页 blob 大小是多少？

Azure 支持的最大页 blob 大小是 8 TiB (8,191 GiB)。 附加到 VM 作为数据或操作系统磁盘时，最大页 blob 大小为 4 TiB (4,095 GiB)。

**是否需要使用新版本的 Azure 工具来创建、附加、上传大于 1 TiB 的磁盘并重设其大小？**

无需升级现有 Azure 工具即可创建、附加大于 1 TiB 的磁盘或重设其大小。 若要直接从本地将 VHD 文件作为页 blob 或非托管磁盘上传到到 Azure，需要使用下方列出的最新工具集。 我们仅支持最大大小为 8 TiB 的 VHD 上传。

|Azure 工具      | 支持的版本                                |
|-----------------|---------------------------------------------------|
|Azure PowerShell | 版本号 4.1.0：2017 年 6 月版本或更高版本|
|Azure CLI v1     | 版本号 0.10.13：2017 年 5 月版本或更高版本|
|Azure CLI v2     | 版本号 2.0.12：2017 年 7 月版本或更高版本|
|AzCopy           | 版本号 6.1.0：2017 年 6 月版本或更高版本|

非托管磁盘或页 blob 是否支持 P4 和 P6 磁盘大小？

非托管磁盘和页 blob 不支持 P4 (32 GiB) 和 P6 (64 GiB) 磁盘大小作为默认磁盘层。 需要显式[设置 Blob 层](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier)，将其设为 P4 和 P6，以便存储映射到这些层的磁盘。 如果使用少于 32 GiB 或介于 32 GiB 到 64 GiB 之间的磁盘大小或内容长度部署非托管磁盘或页 blob 而不设置 Blob 层，则将继续停留在 P10（500 IOPS 和 100 MiB/秒）和映射的定价层。

**如果在支持较小磁盘（约 2017 年 6 月 15 日）之前创建了小于 64 GiB 的高级托管磁盘，将如何计费？**

根据 P10 定价层继续对小于 64 GiB 的现有高级小磁盘计费。

**如何将小于 64 GiB 的高级小磁盘的磁盘层级从 P10 切换到 P4 或 P6？**

你可以拍摄小磁盘的快照，然后创建磁盘以自动根据预配大小将定价层切换到 P4 或 P6。

**可以您重设大小现有托管磁盘大小从少于 4 tib 的) 为最多 32 TiB 的新新引入的磁盘大小？**

可以。

**什么是 Azure 备份和 Azure Site Recovery 服务支持的最大磁盘大小？**

Azure 备份和 Azure Site Recovery 服务支持的最大磁盘大小为 4 TiB。 将很快增加对多达 32 TiB 更大的磁盘支持。

**什么是建议的 VM 大小更大的磁盘大小 (> 4 TiB) 标准 SSD 和 HDD 标准磁盘，以实现优化磁盘 IOPS 和带宽？**

若要实现标准的 SSD 和 HDD 标准大型磁盘大小的磁盘吞吐量 (> 4 TiB) 超出 500 IOPS 和 60 MiB/秒，我们建议部署新的 VM 从一个以下 VM 大小以优化性能：B 系列、 DSv2 系列、 Dsv3 系列，ESv3 系列、 Fs 系列 Fsv2 系列 M 系列、 GS 系列，NCv2 系列，NCv3 系列或 Ls 系列 Vm。 将较大的磁盘附加到现有 Vm 或 Vm 未使用上述建议的大小可能会遇到较低的性能。

**如何升级我的磁盘 (> 4 TiB) 的已部署在更大的磁盘大小预览期间为了在正式版推出后获得更高的 IOPS 和带宽？**

你可以停止和启动磁盘附加到 VM 或，分离并重新附加磁盘。 为高级 Ssd 和标准 Ssd 发行时增加了更大的磁盘大小的性能目标

**有哪些区域？ 8 TiB、 16 TiB 和 32 TiB 中支持的托管的磁盘大小**

在全局 Azure 下的所有区域中支持 8 TiB、 16 TiB 和 32 TiB 磁盘 Sku。 对 Microsoft Azure 政府和 Azure 中国的 21Vianet 的支持尚不可用。

**我们是否支持启用缓存主机上所有磁盘大小？**

我们支持在小于 4 TiB 的磁盘大小上启用只读和读/写主机缓存。 对于超过 4 TiB 的磁盘大小，除提供“无”选项外，我们并不支持设置缓存选项。 建议利用较小磁盘大小的缓存，以便通过缓存到 VM 的数据观察到明显的性能提升。

## <a name="what-if-my-question-isnt-answered-here"></a>如果未在此处找到相关问题怎么办？

如果未在此处找到相关问题，请联系我们获取帮助。 你可以在本文末尾的评论中发布问题。 若要与 Azure 存储团队和其他社区成员就本文进行沟通，请使用 MSDN [Azure 存储论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)。

若要提出功能请求，请将请求和想法提交到 [Azure 存储反馈论坛](https://feedback.azure.com/forums/217298-storage)。
