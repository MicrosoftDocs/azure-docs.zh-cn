---
title: Azure 数据工厂中的带分隔符的文本格式 |Microsoft Docs
description: 本主题介绍如何处理 Azure 数据工厂中的带分隔符的文本格式。
author: linda33wj
manager: craigg
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/29/2019
ms.author: jingwang
ms.openlocfilehash: 407b8ba2fda35d3acbf1b425bb15fe20778613d7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146007"
---
# <a name="delimited-text-format-in-azure-data-factory"></a>Azure 数据工厂中的带分隔符的文本格式

要遵循此篇文章**分析带分隔符的文本文件或将数据写入带分隔符的文本格式**。 

带分隔符的文本格式支持以下连接器：[Amazon S3](connector-amazon-simple-storage-service.md)， [Azure Blob](connector-azure-blob-storage.md)， [Azure 数据湖存储 Gen1](connector-azure-data-lake-store.md)， [Azure 数据湖存储第 2 代](connector-azure-data-lake-storage.md)， [Azure 文件存储](connector-azure-file-storage.md)，[文件系统](connector-file-system.md)， [FTP](connector-ftp.md)， [Google 云存储](connector-google-cloud-storage.md)， [HDFS](connector-hdfs.md)， [HTTP](connector-http.md)，和[SFTP](connector-sftp.md)。

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供带分隔符的文本数据集支持的属性的列表。

| 属性         | 说明                                                  | 必选 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 数据集的 type 属性必须设置为**DelimitedText**。 | 是      |
| 位置         | 文件的位置设置。 每个基于文件的连接器具有其自己的位置类型和支持的属性下的`location`。 **请参阅连接器文章中的详细信息-> 数据集属性部分**。 | 是      |
| columnDelimiter  | 用于分隔文件中的列的字符。 目前，仅映射数据流而不复制活动支持多字符分隔符。 <br>默认值是**逗号`,`** ，当列分隔符定义为空字符串，这意味着没有分隔符，将整行被视为单个列。 | 否       |
| rowDelimiter     | 单个字符或者"\r\n"用于分隔文件中的行。<br>默认值是以下值之一**在读取: ["\r\n"、"\r"、"\n"]**，并 **"\n"或"\r\n"上写入**映射数据的流，并分别复制活动。 <br>当`rowDelimiter`设置为无分隔符 （空字符串），`columnDelimiter`必须设置为，这意味着将整个内容视为单个值作为无分隔符 （空字符串）。 | 否       |
| quoteChar        | 要引用列的值，如果它包含列分隔符的单个字符。 <br>默认值是**双引号** `"`。 <br>有关映射数据流，`quoteChar`不能为空字符串。 <br>对于复制活动，当`quoteChar`定义为空字符串，这意味着没有任何引号字符，并且不引用列的值，和`escapeChar`用于转义的列分隔符和本身。 | 否       |
| escapeChar       | 要转义的带引号的值的单个字符。<br>默认值是**反斜杠`\`** 。 <br>有关映射数据流，`escapeChar`不能为空字符串。 <br/>对于复制活动，当`escapeChar`定义为空字符串，`quoteChar`必须设置为空字符串的形式，在这种情况下确保所有列的值不都包含分隔符。 | 否       |
| firstRowAsHeader | 指定是否将/品牌的第一行作为标头行与列的名称。<br>允许的值为 **，则返回 true**并**false** （默认值）。 | 否       |
| nullValue        | 指定 null 值的字符串表示形式。 <br>默认值是**空字符串**。 | 否       |
| encodingName     | 用于读/写测试文件的编码类型。 <br>允许的值如下所示："UTF-8"、"UTF-16"、"-16BE"、"UTF-32"、"UTF-32BE"，"US ASCII"、"UTF-7"、"BIG5"、"EUC-JP"、"EUC-KR"、"GB2312"、"GB18030"、"JOHAB"、"SHIFT JIS"、"CP875"、"CP866"、"IBM00858"、"IBM037"、"IBM273"、"IBM437"、"IBM500"、"IBM737"、"IBM775"、"IBM850""IBM852"，"IBM855"、"IBM857"、"IBM860"、"IBM861"、"IBM863"、"IBM864"、"IBM865"、"IBM869"、"IBM870"、"IBM01140"、"IBM01141"、"IBM01142"、"IBM01143"、"IBM01144"、"IBM01145"、"IBM01146"、"IBM01147"、"IBM01148"、"IBM01149"、"ISO-2022年-JP"、"ISO-2022年-KR 在"，"ISO-8859-1"、"ISO-8859-2"、"ISO-8859-3"、"ISO-8859-4"、"ISO-8859-5"、"ISO-8859-6"、"ISO-8859-7"、"ISO-8859-8"、"ISO-8859-9"、"ISO-8859-13"、"ISO-8859-15"、"WINDOWS-874"、"WINDOWS-1250"、"WINDOWS-1251"、"WINDOWS-1252"、"WINDOWS-1253"，"WINDOWS-1254年"、"WINDOWS-1255"、"WINDOWS-1256"、"WINDOWS-1257"、"WINDOWS-1258"。<br>请注意映射数据流不支持 utf-7 编码。 | 否       |
| compressionCodec | 压缩编解码器用于读/写文本文件。 <br>允许的值为**bzip2**， **gzip**， **deflate**， **ZipDeflate**， **snappy**，或**lz4**。 若要保存该文件时使用。 <br>请注意当前复制活动不支持"snappy"和"lz4"，并映射数据流不支持"ZipDeflate"。 | 否       |
| compressionLevel | 压缩率。 <br>允许的值为**Optimal**或**Fastest**。<br>- **最快：** 尽快完成压缩操作，不过，无法以最佳方式压缩生成的文件。<br>- **最佳**:以最佳方式完成压缩操作，不过，需要耗费更长的时间。 有关详细信息，请参阅 [Compression Level](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx)（压缩级别）主题。 | 否       |

下面是 Azure Blob 存储上的带分隔符的文本数据集的示例：

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
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
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供带分隔符的文本源和接收器支持的属性的列表。

### <a name="delimited-text-as-source"></a>带分隔符的文本作为源 

将复制活动中支持以下属性***\*源\**** 部分。

| 属性       | 说明                                                  | 必选 |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | 复制活动源的 type 属性必须设置为**DelimitedTextSource**。 | 是      |
| formatSettings | 一组的属性。 请参阅**带分隔符的文本读取设置**下表。 | 否       |
| storeSettings  | 有关如何从数据存储读取数据的属性的组。 每个基于文件的连接器具有其自己支持的读取的设置下`storeSettings`。 **请参阅连接器文章中的详细信息-> 复制活动属性部分**。 | 否       |

支持**分隔的文本读取设置**下`formatSettings`:

| 属性      | 说明                                                  | 必选 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | FormatSettings 的类型必须设置为**DelimitedTextReadSetting**。 | 是      |
| skipLineCount | 指示从输入文件读取数据时要跳过的非空行数。 <br>如果同时指定了 skipLineCount 和 firstRowAsHeader，则先跳过代码行，然后从输入文件读取标头信息。 | 否       |

### <a name="delimited-text-as-sink"></a>带分隔符的文本作为接收器

将复制活动中支持以下属性***\*接收器\**** 部分。

| 属性       | 说明                                                  | 必选 |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | 复制活动源的 type 属性必须设置为**DelimitedTextSink**。 | 是      |
| formatSettings | 一组的属性。 请参阅**带分隔符的文本写入设置**下表。 |          |
| storeSettings  | 如何将数据写入到数据存储区上的属性组。 每个基于文件的连接器具有其自己的受支持的写入设置下`storeSettings`。 **请参阅连接器文章中的详细信息-> 复制活动属性部分**。 | 否       |

支持**分隔的文本写入设置**下`formatSettings`:

| 属性      | 说明                                                  | 必选                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | FormatSettings 的类型必须设置为**DelimitedTextWriteSetting**。 | 是                                                   |
| fileExtension | 用来命名输出文件，例如文件扩展名`.csv`， `.txt`。 它必须是指定何时`fileName`输出中未指定 DelimitedText 数据集。 | 当在未指定文件名称是输出数据集 |

## <a name="mapping-data-flow-properties"></a>数据流属性映射

详细信息请参阅[源转换](data-flow-source.md)并[接收器转换](data-flow-sink.md)中映射数据流动。

## <a name="next-steps"></a>后续步骤

- [复制活动概述](copy-activity-overview.md)
- [映射数据流](concepts-data-flow-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)
- [GetMetadata 活动](control-flow-get-metadata-activity.md)