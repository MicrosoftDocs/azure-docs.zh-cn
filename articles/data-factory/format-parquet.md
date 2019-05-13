---
title: Azure 数据工厂中的 parquet 格式 |Microsoft Docs
description: 本主题介绍如何使用 Azure 数据工厂中的 Parquet 格式处理。
author: linda33wj
manager: craigg
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/29/2019
ms.author: jingwang
ms.openlocfilehash: 360b794f0d8ba9c145a92f015f264eb624fbb0f1
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65144883"
---
# <a name="parquet-format-in-azure-data-factory"></a>Azure 数据工厂中的 parquet 格式

要遵循此篇文章**分析 Parquet 文件或将数据写入 Parquet 格式**。 

Parquet 格式支持以下连接器：[Amazon S3](connector-amazon-simple-storage-service.md)， [Azure Blob](connector-azure-blob-storage.md)， [Azure 数据湖存储 Gen1](connector-azure-data-lake-store.md)， [Azure 数据湖存储第 2 代](connector-azure-data-lake-storage.md)， [Azure 文件存储](connector-azure-file-storage.md)，[文件系统](connector-file-system.md)， [FTP](connector-ftp.md)， [Google 云存储](connector-google-cloud-storage.md)， [HDFS](connector-hdfs.md)， [HTTP](connector-http.md)，和[SFTP](connector-sftp.md)。

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供了支持的 Parquet 数据集属性的列表。

| 属性         | 说明                                                  | 必选 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 数据集的 type 属性必须设置为**Parquet**。 | 是      |
| 位置         | 文件的位置设置。 每个基于文件的连接器具有其自己的位置类型和支持的属性下的`location`。 **请参阅连接器文章中的详细信息-> 数据集属性部分**。 | 是      |
| compressionCodec | 要写入到 Parquet 文件时使用的压缩编解码器。 当读取 Parquet 文件，数据工厂会自动确定基于文件的元数据的压缩编解码器。<br>支持的类型为"**无**"，"**gzip**"、"**snappy**"（默认值） 和"**lzo**"。 请注意当前复制活动不支持 LZO。 | 否       |

> [!NOTE]
> Parquet 文件不支持列名称中的空白区域。

下面是在 Azure Blob 存储的 Parquet 数据集的示例：

```json
{
    "name": "ParquetDataset",
    "properties": {
        "type": "Parquet",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compressionCodec": "snappy"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Parquet 源和接收器支持的属性的列表。

### <a name="parquet-as-source"></a>Parquet 作为源

将复制活动中支持以下属性***\*源\**** 部分。

| 属性      | 说明                                                  | 必选 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 复制活动源的 type 属性必须设置为**ParquetSource**。 | 是      |
| storeSettings | 有关如何从数据存储读取数据的属性的组。 每个基于文件的连接器具有其自己支持的读取的设置下`storeSettings`。 **请参阅连接器文章中的详细信息-> 复制活动属性部分**。 | 否       |

### <a name="parquet-as-sink"></a>Parquet 如接收器

将复制活动中支持以下属性***\*接收器\**** 部分。

| 属性      | 说明                                                  | 必选 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 复制活动源的 type 属性必须设置为**ParquetSink**。 | 是      |
| storeSettings | 如何将数据写入到数据存储区上的属性组。 每个基于文件的连接器具有其自己的受支持的写入设置下`storeSettings`。 **请参阅连接器文章中的详细信息-> 复制活动属性部分**。 | 否       |

## <a name="mapping-data-flow-properties"></a>数据流属性映射

详细信息请参阅[源转换](data-flow-source.md)并[接收器转换](data-flow-sink.md)中映射数据流动。

## <a name="data-type-support"></a>数据类型支持

Parquet 复杂数据类型目前不支持 （例如地图、 列出、 结构）。

## <a name="using-self-hosted-integration-runtime"></a>使用自承载的集成运行时

> [!IMPORTANT]
> 对于由自承载集成运行时（例如，在本地与云数据存储之间）支持的复制，如果不是**按原样**复制 Parquet 文件，则需要在 IR 计算机上安装 **64 位 JRE 8（Java 运行时环境）或 OpenJDK**。 请参阅下面段落中的更多详细信息。

对于使用 Parquet 文件序列化/反序列化在自承载集成运行时上运行的复制，ADF 将通过首先检查 JRE 的注册表项 *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* 来查找 Java 运行时，如果未找到，则会检查系统变量 *`JAVA_HOME`* 来查找 OpenJDK。

- **若要使用 JRE**：64 位 IR 需要 64 位 JRE。 可在[此处](https://go.microsoft.com/fwlink/?LinkId=808605)找到它。
- **若要使用 OpenJDK**：从 IR 版本 3.13 开始受支持。 将 jvm.dll 以及所有其他必需的 OpenJDK 程序集打包到自承载 IR 计算机中，并相应地设置系统环境变量 JAVA_HOME。

> [!TIP]
> 如果使用自承载集成运行时将数据复制为 Parquet 格式或从 Parquet 格式复制数据，并遇到“调用 java 时发生错误，消息: java.lang.OutOfMemoryError:Java 堆空间”的错误，则可以在托管自承载 IR 的计算机上添加环境变量 `_JAVA_OPTIONS`，以便调整 JVM 的最小/最大堆大小，以支持此类复制，然后重新运行管道。

![在自承载 IR 上设置 JVM 堆大小](C:/AzureContent/azure-docs-pr/articles/data-factory/media/supported-file-formats-and-compression-codecs/set-jvm-heap-size-on-selfhosted-ir.png)

示例：将变量 `_JAVA_OPTIONS` 的值设置为 `-Xms256m -Xmx16g`。 标志 `Xms` 指定 Java 虚拟机 (JVM) 的初始内存分配池，而 `Xmx` 指定最大内存分配池。 这意味着 JVM 初始内存为 `Xms`，并且能够使用的最多内存为 `Xmx`。 默认情况下，ADF 最少使用 64MB 且最多使用 1G。

## <a name="next-steps"></a>后续步骤

- [复制活动概述](copy-activity-overview.md)
- [映射数据流](concepts-data-flow-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)
- [GetMetadata 活动](control-flow-get-metadata-activity.md)