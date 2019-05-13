---
title: 在区域冗余存储 (ZRS) 上构建具有高可用性的 Azure 存储应用程序 | Microsoft Docs
description: 区域冗余存储 (ZRS) 提供了一种简单方法来构建具有高可用性的应用程序。 ZRS 针对数据中心内的硬件故障提供保护，还针对某些区域性灾难提供保护。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 10/24/2018
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.openlocfilehash: 9fd594fb264ff0467cc315f59927cd157068d522
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146173"
---
# <a name="zone-redundant-storage-zrs-highly-available-azure-storage-applications"></a>区域冗余存储 (ZRS)：具有高可用性的 Azure 存储应用程序
[!INCLUDE [storage-common-redundancy-ZRS](../../../includes/storage-common-redundancy-zrs.md)]

## <a name="support-coverage-and-regional-availability"></a>支持覆盖范围和区域可用性
ZRS 目前支持标准的常规用途 v2 帐户类型。 有关存储帐户类型的详细信息，请参阅 [Azure 存储帐户概述](storage-account-overview.md)。

ZRS 适用于块 Blob、非磁盘页 Blob、文件、表和队列。

ZRS 在以下区域推出了正式版：

- 亚洲东南部
- 西欧
- 北欧
- 法国中部
- 日本东部
- 英国南部
- 美国东部
- 美国东部 2
- 美国西部 2
- 美国中部

Microsoft 将继续在其他 Azure 区域推出 ZRS。 请不时地查看 [Azure 服务更新](https://azure.microsoft.com/updates/)来了解有关新区域的信息。

## <a name="what-happens-when-a-zone-becomes-unavailable"></a>当一个区域不可用时，会发生什么情况？
即使某个区域不可用，也仍可访问你的数据进行读写操作。 Microsoft 建议持续遵循暂时性故障处理的做法。 这些做法包括结合指数退让实施重试策略。

当某个区域不可用时，Azure 会执行网络更新，例如 DNS 重新指向。 如果完成更新之前访问数据，这些更新可能会影响应用程序。

如果多个区域永久受到影响，在发生区域性的灾难时，ZRS 可能无法保护数据。 如果区域只是暂时不可用，则 ZRS 可以提供数据复原能力。 为了在发生区域性灾难时提供保护，Microsoft 建议使用异地冗余存储 (GRS)。 有关 GRS 的详细信息，请参阅[异地冗余存储 (GRS)：Azure 存储的跨区域复制](storage-redundancy-grs.md)。

## <a name="converting-to-zrs-replication"></a>转换为 ZRS 复制
在 LRS、GRS 和 RA-GRS 之间来回迁移的过程非常直接。 使用 Azure 门户或存储资源提供程序 API 可以更改帐户的冗余类型。 然后，Azure 会相应地复制数据。 

与 ZRS 之间来回迁移数据需要不同的策略。 ZRS 迁移涉及到将数据从单个存储阵列物理迁移到某个区域中的多个阵列。

有用于迁移到 ZRS 的两个主要选项： 

- 手动将现有帐户中的数据复制或移动到新的 ZRS 帐户。
- 请求实时迁移。

Microsoft 强烈建议执行手动迁移。 手动迁移比实时迁移更灵活。 使用手动迁移可以控制时间。

若要执行手动迁移，可使用以下选项：
- 使用现有的工具（例如 AzCopy）、某个 Azure 存储客户端库或可靠的第三方工具。
- 如果你熟悉 Hadoop 或 HDInsight，可将源和目标 (ZRS) 帐户附加到群集。 然后使用 DistCp 等工具来并行化数据复制过程。
- 使用某个 Azure 存储客户端库生成自己的工具。

手动迁移可能导致应用程序关闭。 如果应用程序需要高可用性，则还可以使用 Microsoft 提供的实时迁移选项。 实时迁移属于就地迁移。 

在实时迁移过程中，在源与目标存储阵列之间迁移数据时，可以使用自己的存储帐户。 在迁移过程中，你将具有相同级别的持久性和可用性 SLA，因为您通常情况下进行。

请注意实时迁移的以下限制：

- 尽管 Microsoft 会尽快处理实时迁移的请求，但无法保证实时迁移何时完成。 如果需要在特定的日期之前将数据迁移到 ZRS，则 Microsoft 建议执行手动迁移。 通常，帐户中的数据越多，迁移这些数据所需的时间就越长。 
- 只有使用 LRS 或 GRS 复制的存储帐户才支持实时迁移。 如果你的帐户使用 RA-GRS，则需要先将帐户的复制类型更改为 LRS 或 GRS，然后继续。 此中间步骤可确保在迁移之前，删除 RA-GRS 提供的辅助只读终结点。
- 帐户必须包含数据。
- 只能在同一区域中迁移数据。 若要将数据迁移到不包含源帐户的区域中的 ZRS 帐户，则必须执行手动迁移。
- 只有标准存储帐户类型才支持实时迁移。 高级存储帐户必须手动迁移。
- 不支持从 ZRS 到 LRS、 GRS 或 RA-GRS 的实时迁移。 你将需要手动将数据移到新的或现有的存储帐户。

可以通过 [Azure 支持门户](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)请求实时迁移。 在门户中，选择要转换为 ZRS 的存储帐户。
1. 选择“新建支持请求”
2. 根据帐户信息填写“基本信息”。 在“服务”部分，选择“存储帐户管理”以及要转换为 ZRS 的资源。 
3. 选择“**下一步**”。 
4. 在“问题”部分指定以下值： 
    - **严重性**：保留默认值。
    - **问题类型**：选择“数据迁移”。
    - **类别**：选择“迁移到区域中的 ZRS”。
    - **标题**：键入描述性的标题，例如“ZRS 帐户迁移”。
    - **详细信息**：在“详细信息”框中键入更多详细信息，例如，“我想要从 \_\_ 区域中的 [LRS, GRS] 迁移到 ZRS”。 
5. 选择“**下一步**”。
6. 检查“联系信息”边栏选项卡中的联系信息是否正确。
7. 选择“创建”。

支持人员将与你取得联系，并提供所需的任何帮助。

## <a name="live-migration-to-zrs-faq"></a>实时迁移到 ZRS 常见问题

**我应该针对任何停机时间在迁移过程？**

不会引起迁移造成停机。 在实时迁移期间可以继续您的存储帐户，而你的数据源和目标存储戳间迁移。 在迁移过程中，你将具有相同级别的持久性和可用性 SLA，因为您通常情况下进行。

**是否有任何与迁移相关的数据丢失？**

没有与迁移相关联无数据丢失。 在迁移过程中，你将具有相同级别的持久性和可用性 SLA，因为您通常情况下进行。

**迁移完成后，请将所需的应用程序的任何更新？**

迁移完成后的帐户的复制类型将更改为"区域冗余存储 (ZRS)"。 服务终结点访问密钥，SAS 和任何其他帐户配置选项保持不变且不变。

**可以请求的实时迁移到 ZRS 我常规用途 v1 帐户吗？**

ZRS 仅支持常规用途 v2 帐户，因此在提交之前为实时迁移到 ZRS 请求请确保将你的帐户升级到常规用途 v2。 请参阅[Azure 存储帐户概述](https://docs.microsoft.com/azure/storage/common/storage-account-overview)并[升级到常规用途 v2 存储帐户](https://docs.microsoft.com/azure/storage/common/storage-account-upgrade)的更多详细信息。

**可以请求的实时迁移到 ZRS 我读取访问异地冗余存储 (RA-GRS) 帐户吗？**

在提交之前请确保你的应用程序或工作负载不再需要辅助只读的终结点访问和更改你的存储帐户的复制类型为异地冗余存储 (GRS) 为实时迁移到 ZRS 的请求。 请参阅[更改复制策略](https://docs.microsoft.com/azure/storage/common/storage-redundancy#changing-replication-strategy)的更多详细信息。

**可以请求的实时迁移到另一个区域 ZRS 到我的存储帐户吗？**

如果你想要将数据迁移到 ZRS 帐户位于不同区域中的源帐户的区域，则必须执行手动迁移。

## <a name="zrs-classic-a-legacy-option-for-block-blobs-redundancy"></a>ZRS 经典版：用于块 blob 冗余的传统选项
> [!NOTE]
> Microsoft 将于 2021 年 3 月 31 日弃用并迁移 ZRS 经典版帐户。 在弃用之前会向 ZRS 经典版客户提供更多详细信息。 
>
> 当 ZRS 变得[正式](#support-coverage-and-regional-availability)在区域中，客户无法再从该区域中的门户创建 ZRS 经典版帐户。 在弃用 ZRS 经典版之前，可以使用 Microsoft PowerShell 和 Azure CLI 创建 ZRS 经典版帐户。

ZRS 经典版以异步方式在一到两个区域中的数据中心之间复制数据。 除非 Microsoft 发起了到次要区域的故障转移，否则复制的数据可能不可用。 ZRS 经典版帐户无法与 LRS、GRS 或 RA-GRS 相互转换。 ZRS 经典版帐户也不支持指标或日志记录。

ZRS 经典版仅适用于常规用途 V1 (GPv1) 存储帐户中的块 Blob。 有关存储帐户的详细信息，请参阅 [Azure 存储帐户概述](storage-account-overview.md)。

若要向/从 LRS、ZRS 经典版、GRS 或 RA-GRS 帐户手动迁移 ZRS 帐户数据，请使用以下工具之一：AzCopy、Azure 存储资源管理器、Azure PowerShell 或 Azure CLI。 此外，可以使用某个 Azure 存储客户端库生成自己的迁移解决方案。

此外可以升级到 ZRS 在门户或使用 Azure PowerShell 或 Azure CLI 在 ZRS 可用的区域中的 ZRS 经典版帐户。

若要升级到在门户中的 ZRS 转到该帐户的配置节，并选择升级：![升级到 ZRS 在门户中的 ZRS 经典](media/storage-redundancy-zrs/portal-zrs-classic-upgrade.jpg)

若要升级到 ZRS 使用 PowerShell，请调用以下命令：
```powershell
Set-AzStorageAccount -ResourceGroupName <resource_group> -AccountName <storage_account> -UpgradeToStorageV2
```

若要升级到 ZRS 使用 CLI，请调用以下命令：
```cli
az storage account update -g <resource_group> -n <storage_account> --set kind=StorageV2
```

## <a name="see-also"></a>另请参阅
- [Azure 存储复制](storage-redundancy.md)
- [本地冗余存储 (LRS)：Azure 存储的低成本数据冗余](storage-redundancy-lrs.md)
- [异地冗余存储 (GRS)：Azure 存储的跨区域复制](storage-redundancy-grs.md)
