---
title: Azure 数据工厂中的复制活动 | Microsoft Docs
description: 了解 Azure 数据工厂中可用于将数据从支持的源数据存储复制到支持的接收器数据存储的复制活动。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 04/29/2019
ms.author: jingwang
ms.openlocfilehash: 8f5a7d3f6300be100feffd23b98bd7dcd8f48148
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150877"
---
# <a name="copy-activity-in-azure-data-factory"></a>Azure 数据工厂中的复制活动

## <a name="overview"></a>概述

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1](v1/data-factory-data-movement-activities.md)
> * [当前版本](copy-activity-overview.md)

在 Azure 数据工厂中，可使用复制活动在本地和云数据存储之间复制数据。 复制数据后，可对其进一步执行转换和分析操作。 还可使用复制活动发布有关商业智能 (BI) 和应用程序消耗的转换和分析结果。

![复制活动的角色](media/copy-activity-overview/copy-activity.png)

复制活动在[集成运行时](concepts-integration-runtime.md)上执行。 对于不同的数据复制方案，可以利用不同风格的集成运行时：

* 在两个可公开访问的数据存储间复制数据时，复制活动可由 **Azure 集成运行时**授权，该活动安全可靠、可缩放并可[全局访问](concepts-integration-runtime.md#integration-runtime-location)。
* 当从本地数据存储或位于具有访问控制的网络（如 Azure 虚拟网络）中的数据存储中复制数据，或将数据复制到此处时，需要安装**自承载集成运行时**来授权数据复制。

集成运行时需要与每个源数据存储和接收器数据存储相关联。 了解复制活动如何[决定使用哪个 IR](concepts-integration-runtime.md#determining-which-ir-to-use) 的详细信息。

将数据从源复制到接收器，复制活动要经过以下步骤。 为复制活动提供支持的服务可：

1. 读取源数据存储中的数据。
2. 执行序列化/反序列化、压缩/解压缩、列映射等。此服务基于输入数据集、输出数据集和复制活动的配置执行这些操作。
3. 将数据写入接收器/目标数据存储。

![复制活动概述](media/copy-activity-overview/copy-activity-overview.png)

## <a name="supported-data-stores-and-formats"></a>支持的数据存储和格式

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores.md)]

### <a name="supported-file-formats"></a>支持的文件格式

可以使用复制活动**复制**基于文件的两个数据存储间的文件，这种情况下，数据可以高效复制而无需任何序列化/反序列化。

复制活动还支持以指定的格式读取和写入文件：**文本、 JSON、 Avro、 ORC 和 Parquet**，并进行压缩和解压缩文件使用以下编解码器：**GZip、 Deflate，BZip2 和 ZipDeflate**。 有关详细信息，请参阅[支持的文件和压缩格式](supported-file-formats-and-compression-codecs.md)。

例如，可执行以下复制活动：

* 在本地 SQL Server 中复制数据，以 Parquet 格式写入到 Azure 数据湖存储第 2 代。
* 从本地文件系统中复制文本 (CSV) 格式文件，并将其以 Avro 格式写入 Azure Blob。
* 从本地文件系统复制压缩的文件并解压缩然后传到 Azure 数据湖存储第 2 代。
* 从 Azure Blob 复制 GZip 压缩文本 (CSV) 格式的数据，并将其写入 Azure SQL 数据库。
* 和使用序列化/反序列化或压缩/解压缩的很多更多情况下需要。

## <a name="supported-regions"></a>支持的区域

为复制活动提供支持的服务面向 [Azure Integration Runtime 位置](concepts-integration-runtime.md#integration-runtime-location)中列出的区域和地理位置全球发布。 全局可用拓扑可确保高效的数据移动，此类移动通常避免跨区域跃点。 有关某区域内数据工厂和数据移动的可用性，请参阅[服务（按区域）](https://azure.microsoft.com/regions/#services)。

## <a name="configuration"></a>配置

要使用 Azure 数据工厂中的复制活动，需要执行以下操作：

1. **创建用于源数据存储和接收器数据存储的链接服务。** 有关如何配置支持的属性，请参阅连接器文章的“链接服务属性”部分。 受支持的连接器列表可以在[受支持的数据存储和格式](#supported-data-stores-and-formats)部分找到。
2. **为源和接收器创建数据集。** 有关如何配置支持的属性，请参阅源和接收器连接器文章的“数据集属性”部分。
3. **借助复制活动创建管道。** 接下来的部分将提供示例。

### <a name="syntax"></a>语法

以下复制活动模板包含受支持属性的详尽列表。 指定适合你的方案的属性。

```json
"activities":[
    {
        "name": "CopyActivityTemplate",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<source dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<sink dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>",
                <properties>
            },
            "sink": {
                "type": "<sink type>"
                <properties>
            },
            "translator":
            {
                "type": "TabularTranslator",
                "columnMappings": "<column mapping>"
            },
            "dataIntegrationUnits": <number>,
            "parallelCopies": <number>,
            "enableStaging": true/false,
            "stagingSettings": {
                <properties>
            },
            "enableSkipIncompatibleRow": true/false,
            "redirectIncompatibleRowSettings": {
                <properties>
            }
        }
    }
]
```

### <a name="syntax-details"></a>语法详细信息

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动的 type 属性必须设置为：**Copy** | 是 |
| inputs | 指定创建的指向源数据的数据集。 复制活动仅支持单个输入。 | 是 |
| outputs | 指定创建的指向接收器数据的数据集。 复制活动仅支持单个输出。 | 是 |
| typeProperties | 一组用来配置复制活动的属性。 | 是 |
| source | 指定有关如何检索数据的复制源类型和相应的属性。<br/><br/>有关详细信息，请参阅[受支持的数据存储和格式](#supported-data-stores-and-formats)中所列的连接器文章中的“复制活动属性”部分。 | 是 |
| 接收器 | 指定有关如何写入数据的复制接收器类型和相应的属性。<br/><br/>有关详细信息，请参阅[受支持的数据存储和格式](#supported-data-stores-and-formats)中所列的连接器文章中的“复制活动属性”部分。 | 是 |
| 转换器 | 指定从源到接收器的显式列映射。 当默认复制行为无法满足需求时适用。<br/><br/>有关详细信息，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。 | 否 |
| dataIntegrationUnits | 指定 [Azure 集成运行时](concepts-integration-runtime.md)的强度来授权数据复制。 以前称为云数据移动单位 (DMU)。 <br/><br/>从[数据集成单位](copy-activity-performance.md#data-integration-units)了解详细信息。 | 否 |
| parallelCopies | 指定从源读取数据和向接收器写入数据时想要复制活动使用的并行度。<br/><br/>详细信息请参阅[并行复制](copy-activity-performance.md#parallel-copy)。 | 否 |
| enableStaging<br/>stagingSettings | 选择暂存中而不是直接从源到接收器的复制数据的 blob 存储的临时数据。<br/><br/>请参阅[暂存复制](copy-activity-performance.md#staged-copy)了解有用的方案和配置详细信息。 | 否 |
| enableSkipIncompatibleRow<br/>redirectIncompatibleRowSettings| 选择将数据从源复制到接收器时如何处理不兼容的行。<br/><br/>有关详细信息，请参阅[容错](copy-activity-fault-tolerance.md)。 | 否 |

## <a name="monitoring"></a>监视

可通过编程方式或在 Azure 数据工厂的“创作和监视”UI 上监视复制活动运行。 然后，可将方案的性能和配置与内部测试中复制活动的[性能参考](copy-activity-performance.md#performance-reference)进行比较。

### <a name="monitor-visually"></a>直观地监视

若要直观地监视复制活动运行，请转到数据工厂 ->“创作和监视” -> “监视”选项卡，可看到一个管道运行列表，其“操作”列中提供了“查看活动运行”链接。

![监视管道运行](./media/load-data-into-azure-data-lake-store/monitor-pipeline-runs.png)

单击以查看此管道运行中的活动列表。 在“操作”列中，具有指向复制活动输入、输出、错误（如果复制活动运行失败）和详细信息的链接。

![监视活动运行](./media/load-data-into-azure-data-lake-store/monitor-activity-runs.png)

单击“操作”下的“详细信息”链接，查看复制活动的执行详细信息和性能特征。 它显示复制方案的以下信息：从源复制到接收器的数据量/行/文件、吞吐量、所执行的步骤和相应的持续时间，以及使用的配置。

>[!TIP]
>对于某些方案，你还会在复制监视页面的顶部看到“**性能优化提示**”，这告诉你所识别出的瓶颈并指导你进行一些更改来提升复制吞吐量，请参阅[此处](#performance-and-tuning)包含详细信息的示例。

**示例：从 Amazon S3 复制到 Azure Data Lake Store**
![监视活动运行详细信息](./media/copy-activity-overview/monitor-activity-run-details-adls.png)

**示例：使用暂存复制从 Azure SQL 数据库复制到 Azure SQL 数据仓库**
![监视活动运行详细信息](./media/copy-activity-overview/monitor-activity-run-details-sql-dw.png)

### <a name="monitor-programmatically"></a>以编程方式监视

“复制活动运行结果”->“输出”部分中也会返回复制活动执行详细信息和性能特征。 下面是一个详细清单；将只显示适用于用户复制方案的内容。 了解如何从[快速入门监视部分](quickstart-create-data-factory-dot-net.md#monitor-a-pipeline-run)监视活动运行。

| 属性名称  | 描述 | 单位 |
|:--- |:--- |:--- |
| dataRead | 从源中读取的数据大小 | Int64 值（**字节**） |
| DataWritten | 写入接收器的数据大小 | Int64 值（**字节**） |
| filesRead | 从文件存储复制数据时要复制的文件数。 | Int64 值（未指定单位） |
| filesWritten | 将数据复制到文件存储时要复制的文件数。 | Int64 值（未指定单位） |
| rowsRead | 正在从 （不适用于二进制副本） 的源中读取的行数。 | Int64 值（未指定单位） |
| rowsCopied | 正在复制到接收器 （不适用于二进制副本） 的行数。 | Int64 值（未指定单位） |
| rowsSkipped | 跳过的不兼容行数。 可以通过将“enableSkipIncompatibleRow”设置为 true 来启用该功能。 | Int64 值（未指定单位） |
| throughput | 上传输数据的比率。 | 浮点数 (**KB/s**) |
| copyDuration | 复制持续时间。 | Int32 值（秒） |
| sourcePeakConnections | 峰值在复制期间建立到源数据存储的并发连接数。 | Int32 值 |
| sinkPeakConnections| 峰值到接收器数据存储在复制期间建立的并发连接数。| Int32 值 |
| sqlDwPolyBase | 如果将数据复制到 SQL 数据仓库时使用了 PolyBase。 | Boolean |
| redshiftUnload | 如果从 Redshift 复制数据时使用了 UNLOAD。 | Boolean |
| hdfsDistcp | 如果从 HDFS 复制数据时使用了 DistCp。 | Boolean |
| effectiveIntegrationRuntime | 以 `<IR name> (<region if it's Azure IR>)` 格式显示运行活动时使用的集成运行时。 | 文本（字符串） |
| usedDataIntegrationUnits | 复制期间的有效数据集成单位。 | Int32 值 |
| usedParallelCopies | 复制期间的有效 parallelCopies。 | Int32 值|
| redirectRowPath | 在“redirectIncompatibleRowSettings”下配置的 blob 存储中跳过的不兼容行的日志路径。 请参阅下面的示例。 | 文本（字符串） |
| executionDetails | 有关复制活动经历的各个阶段、相应步骤、持续时间、使用的配置等内容的更多详细信息。不建议分析此节，因为它有可能发生更改。 | Array |

```json
"output": {
    "dataRead": 107280845500,
    "dataWritten": 107280845500,
    "filesRead": 10,
    "filesWritten": 10,
    "copyDuration": 224,
    "throughput": 467707.344,
    "errors": [],
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US 2)",
    "usedDataIntegrationUnits": 32,
    "usedParallelCopies": 8,
    "executionDetails": [
        {
            "source": {
                "type": "AmazonS3"
            },
            "sink": {
                "type": "AzureDataLakeStore"
            },
            "status": "Succeeded",
            "start": "2018-01-17T15:13:00.3515165Z",
            "duration": 221,
            "usedDataIntegrationUnits": 32,
            "usedParallelCopies": 8,
            "detailedDurations": {
                "queuingDuration": 2,
                "transferDuration": 219
            }
        }
    ]
}
```

## <a name="schema-and-data-type-mapping"></a>架构和数据类型映射

请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)，其中说明了复制活动如何将源数据映射到接收器。

## <a name="fault-tolerance"></a>容错

默认情况下，如果遇到源和接收器之间数据不兼容，复制活动将停止复制数据，并返回故障。 您可以显式配置以跳过并记录不兼容的行，仅复制那些兼容的数据，以便成功复制。 有关详细信息，请参阅[复制活动容错](copy-activity-fault-tolerance.md)。

## <a name="performance-and-tuning"></a>性能和优化

请参阅[复制活动性能和优化指南](copy-activity-performance.md)，其中介绍了影响 Azure 数据工厂中数据移动（复制活动）性能的关键因素。 还列出了在内部测试期间观察到的性能，并讨论了优化复制活动性能的多种方法。

在某些情况下，当你在 ADF 中执行复制活动时，会直接在[复制活动监视页面](#monitor-visually)上看到“**性能优化提示**”，如以下示例所示。 它不仅告诉你针对给定复制运行所识别出的瓶颈，而且还指导你进行一些更改来提升复制吞吐量。 目前的性能优化提示提供如下建议：在将数据复制到 Azure SQL 数据仓库时使用 PolyBase；在数据存储端资源出现瓶颈时增加 Azure Cosmos DB RU 或 Azure SQL DB DTU；删除不必要的暂存副本等等。性能优化规则也将逐渐丰富。

**示例：复制到 Azure SQL DB 时的性能优化提示**

在此示例中，在复制运行期间，ADF 注意到接收器 Azure SQL DB 达到了很高的 DTU 利用率，这会减慢写入操作，因此，建议使用更多的 DTU 来增加 Azure SQL DB 层。

![包含性能优化提示的复制监视](./media/copy-activity-overview/copy-monitoring-with-performance-tuning-tips.png)

## <a name="incremental-copy"></a>增量复制
数据工厂支持以递增方式将增量数据从源数据存储复制到目标数据存储的方案。 请参阅[教程：以递增方式复制数据](tutorial-incremental-copy-overview.md)。

## <a name="read-and-write-partitioned-data"></a>读取和写入分区数据
在版本 1 中，Azure 数据工厂支持使用 SliceStart/SliceEnd/WindowStart/WindowEnd 系统变量读取或写入分区的数据。 在当前版本中，可使用管道参数和触发器的开始时间/计划时间作为参数值实现此行为。 有关详细信息，请参阅[如何读取或写入分区的数据](how-to-read-write-partitioned-data.md)。

## <a name="next-steps"></a>后续步骤
请参阅以下快速入门、教程和示例：

- [在同一 Azure Blob 存储中将数据从一个位置复制到另一个位置](quickstart-create-data-factory-dot-net.md)
- [将数据从 Azure Blob 存储复制到 Azure SQL 数据库](tutorial-copy-data-dot-net.md)
- [将数据从本地 SQL Server 复制到 Azure](tutorial-hybrid-copy-powershell.md)
