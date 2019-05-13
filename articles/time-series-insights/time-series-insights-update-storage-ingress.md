---
title: Azure 时序见解预览版中的数据存储和入口 | Microsoft Docs
description: 了解 Azure 时序见解预览版中的数据存储和入口。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: 35d9e953ade337672fd57149e325b507f6ce115f
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65405711"
---
# <a name="data-storage-and-ingress-in-azure-time-series-insights-preview"></a>Azure 时序见解预览版中的数据存储和入口

本文介绍 Azure 时序见解预览版中对数据存储和入口的更改， 其中包括底层存储结构、文件格式和时序 ID 属性。 本文还介绍了底层流入过程、吞吐量和限制。

## <a name="data-storage"></a>数据存储

创建时序见解预览版即用即付 SKU 环境时，会创建两个资源：

* 时序见解环境。
* 用于存储数据的 Azure 存储常规用途 V1 帐户。

时序见解预览版对 Parquet 文件类型使用 Azure Blob 存储。 时序见解管理所有数据操作，包括创建 Blob、编制索引以及将 Azure 存储帐户中的数据分区。 使用 Azure 存储帐户创建这些 Blob。

与其他 Azure 存储 Blob 一样，时序见解创建的 Blob 允许读取和写入，以支持各种集成方案。

> [!TIP]
> 如果过于频繁地读取或写入 Blob，时序见解的性能可能会受到负面影响。

有关 Azure Blob 存储的概述，请参阅[存储 Blob 简介](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)。

有关 Parquet 文件类型的详细信息，请参阅 [Azure 存储中支持的文件类型](https://docs.microsoft.com/azure/data-factory/supported-file-formats-and-compression-codecs#Parquet-format)。

## <a name="parquet-file-format"></a>Parquet 文件格式

Parquet 是面向列的数据文件格式，旨在实现：

* 互操作性
* 空间效率
* 查询效率

时序见解之所以选择 Parquet，是因为它提供高效的数据压缩和编码方案，并增强了性能，可以批量处理复杂的数据。

若要更好地了解 Parquet 文件格式，请参阅 [Parquet 文档](https://parquet.apache.org/documentation/latest/)。

### <a name="event-structure-in-parquet"></a>Parquet 中的事件结构

时序见解使用以下两种格式创建和存储 Blob 的副本：

1. 首先，按抵达时间将初始副本分区：

    * `V=1/PT=Time/Y=<YYYY>/M=<MM>/<YYYYMMDDHHMMSSfff>_<TSI_INTERNAL_SUFFIX>.parquet`
    * 按抵达时间将 Blob 的 Blob 创建时间分区。

1. 其次，按时序 ID 的动态分组将重新分区的副本分区：

    * `V=1/PT=TsId/Y=<YYYY>/M=<MM>/<YYYYMMDDHHMMSSfff>_<TSI_INTERNAL_SUFFIX>.parquet`
    * 按时序 ID 将 Blob 中 Blob 的最小事件时间戳分区。

> [!NOTE]
> * `<YYYY>` 映射到 4 位数的年份表示形式。
> * `<MM>` 映射到 2 位数的月份表示形式。
> * `<YYYYMMDDHHMMSSfff>` 映射到包含 4 位数年份 (`YYYY`)、2 位数月份 (`MM`)、2 位数日期 (`DD`)、2 位数小时 (`HH`)、2 位数分钟 (`MM`)、2 位数字秒 (`SS`) 和 3 位数毫秒 (`fff`) 的的时间戳表示形式。

时序见解事件按如下所示映射到 Parquet 文件内容：

* 每个事件映射到单个行。
* 包含事件时间戳的内置“时间戳”列。 “时间戳”属性永远不会为 null。 如果未在事件源中指定“时间戳”属性，该属性默认为“事件源排队时间”。 时间戳采用 UTC 时间。 
* 映射到列的其他所有属性以 `_string`（字符串）、`_bool`（布尔值）、`_datetime`（日期时间）或 `_double`（双精度值）结尾，具体取决于属性类型。
* 这是文件格式的第一个版本（称为 **V=1**）的映射方案。 随着此功能的不断演进，名称将递增为 **V=2**、**V=3**，依次类推。

## <a name="partitions"></a>分区

每个时间系列 Insights 预览版环境必须有**时间系列 ID**属性和一个**时间戳**唯一识别它的属性。 时序 ID 充当数据的逻辑分区，为时序见解预览版环境提供自然边界，以便跨物理分区分布数据。 时序见解预览版在 Azure 存储帐户中管理物理分区。

时序见解通过删除和重新创建分区，使用动态分区来优化存储和查询性能。 时序见解预览版动态分区算法会尽量防止在单个物理分区中包含多个不同逻辑分区的数据。 换言之，分区算法专门在 Parquet 文件中保留单个时序 ID 特定的所有数据，使其不与其他时序 ID 重叠。 动态分区算法还会尽量保留单个时序 ID 中事件的原始顺序。

最初在流入时，数据将按时间戳分区，因此，给定时间范围内的单个逻辑分区可以分散在多个物理分区之间。 单个物理分区中还可以包含多个或所有逻辑分区。 由于 Blob 大小有限制，即使使用最佳分区方案，单个逻辑分区也仍可能占用多个物理分区。

> [!NOTE]
> 默认情况下，“时间戳”值是配置的事件源中消息的*排队时间*。

若要上传历史数据或批消息，请将要连同数据一起存储的值分配到映射至相应时间戳的“时间戳”属性。 “时间戳”属性区分大小写。 有关详细信息，请参阅[时序模型](./time-series-insights-update-tsm.md)。

### <a name="physical-partitions"></a>物理分区

物理分区是存储在存储帐户中的块 Blob。 Blob 的实际大小可能不同，因为大小取决于推送速率。 但是，我们预期 Blob 大小大约为 20 MB 到 50 MB。 时序见解团队可以参考这种预期，选择 20 MB 作为大小来优化查询性能。 此大小根据文件大小和数据流入速度不断变化。

> [!NOTE]
> * 将 Blob 大小调整为 20 MB。
> * 为了提高性能，偶尔会通过删除和重新创建将 Azure Blob 重新分区。
> * 此外，相同的时序见解数据可以保存在两个或更多个 Blob 中。

### <a name="logical-partitions"></a>逻辑分区

逻辑分区指物理分区内用于存储与单个分区键值关联的所有数据的分区。 时序见解预览版基于两个属性对每个 Blob 进行逻辑分区：

* **时序 ID**：事件流和模型中所有时序见解数据的分区键。
* **时间戳**：基于初始流入的时间。

时序见解预览版提供基于这两个属性的性能查询。 这两个属性还提供用于快速传送时序见解数据的最有效方法。

必须选择适当的时序 ID，因为它是不可变的属性。 有关详细信息，请参阅[选择时序 ID](./time-series-insights-update-how-to-id.md)。

## <a name="azure-storage"></a>Azure 存储

### <a name="your-storage-account"></a>你的存储帐户

创建时序见解即用即付环境时，会创建两个资源：时序见解环境，以及用于存储数据的 Azure 存储常规用途 V1 帐户。 我们已选择将 Azure 存储常规用途 V1 用作默认资源，由于它在互操作性、价格和性能方面具有优势。 

时序见解在 Azure 存储帐户中发布每个事件的最多两个副本。 初始副本始终会保留，使你能够使用其他服务对它进行快速查询。 可以基于原始 Parquet 文件对时序 ID 轻松使用 Spark、Hadoop 和其他熟悉的工具，因为这些引擎支持基本的文件名筛选。 按年份和月份将 Blob 分组，是列出自定义作业特定时间范围内的 Blob 的有效方式。 

此外，时序见解可将 Parquet 文件重新分区，以优化时序见解 API。 还会保存最近重新分区的文件。

在公共预览版中，数据无限期存储在 Azure 存储帐户中。

### <a name="writing-and-editing-time-series-insights-blobs"></a>编写和编辑时序见解 Blob

为了确保查询性能和数据可用性，请不要编辑或删除时序见解创建的任何 Blob。

### <a name="accessing-and-exporting-data-from-time-series-insights-preview"></a>从时序见解预览版访问和导出数据

你可能想要访问时序见解预览版资源管理器中存储的数据，以便与其他服务结合使用这些数据。 例如，你可能想要使用数据在 Power BI 中生成报表、通过 Azure 机器学习工作室执行机器学习，或者结合 Jupyter Notebook 在 Notebook 应用程序中使用数据。

可通过三种常规方式访问数据：

* 从时间系列 Insights 预览版资源管理器： 可以将数据导出为 CSV 文件中的时间系列 Insights 预览版资源管理器。 有关详细信息，请参阅[时序见解预览版资源管理器](./time-series-insights-update-explorer.md)。
* 从时序见解预览 Api: API 终结点可以到达`/getRecorded`。 有关此 API 的详细信息，请参阅[时序查询](./time-series-insights-update-tsq.md)。
* 直接从 Azure 存储帐户 （下面）。

#### <a name="from-an-azure-storage-account"></a>通过 Azure 存储帐户

* 需要对用于访问时序见解数据的任何帐户拥有读取访问权限。 有关详细信息，请参阅[管理对存储帐户资源的访问权限](https://docs.microsoft.com/azure/storage/blobs/storage-manage-access-to-resources)。
* 有关直接从 Azure Blob 存储读取数据的详细信息，请参阅[将数据移入和移出存储帐户](https://docs.microsoft.com/azure/storage/common/storage-moving-data?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
* 从 Azure 存储帐户导出数据：
    * 首先确保帐户符合导出数据的必要要求。 有关详细信息，请参阅[存储导入和导出要求](https://docs.microsoft.com/azure/storage/common/storage-import-export-requirements)。
    * 若要了解从 Azure 存储帐户导出数据的其他方法，请参阅[从 Blob 导入和导出数据](https://docs.microsoft.com/azure/storage/common/storage-import-export-data-from-blobs)。

### <a name="data-deletion"></a>数据删除

不要删除 Blob，因为时序见解预览版在其中保留有关 Blob 的元数据。

## <a name="time-series-insights-data-ingress"></a>时间时序见解数据入口

### <a name="ingress-policies"></a>入口策略

时序见解预览版支持时序见解目前所支持的相同事件源和文件类型。

支持的事件源包括：

- Azure IoT 中心
- Azure 事件中心
  
  > [!NOTE]
  > Azure 事件中心实例支持 Kafka。

支持的文件类型包括：

* JSON：有关我们可以处理的受支持 JSON 形状的详细信息，请参阅[如何为 JSON 造型](./time-series-insights-send-events.md#json)。

### <a name="data-availability"></a>数据可用性

时序见解预览版使用 Blob 大小优化策略为数据编制索引。 根据数据传入量以及传入速度为数据编制索引后，这些数据可用于查询。

> [!IMPORTANT]
> * 在接通事件源的 60 秒内，时序见解正式版 (GA) 即可提供数据。 
> * 在预览版中，预期需要在更长的时间后才会提供数据。
> * 如果遇到很长的延迟，请务必联系我们。

### <a name="scale"></a>缩放

时序见解预览版最高支持每个环境每秒 6 Mbps 的初始流入规模。 我们正在增强缩放支持， 到时会更新我们的文档以反映这些改进

## <a name="next-steps"></a>后续步骤

- 阅读 [Azure 时序见解预览版存储和入口](./time-series-insights-update-storage-ingress.md)。

- 了解新型[数据建模](./time-series-insights-update-tsm.md)。

<!-- Images -->
[1]: media/v2-update-storage-ingress/storage-architecture.png
[2]: media/v2-update-storage-ingress/parquet-files.png
[3]: media/v2-update-storage-ingress/blob-storage.png
