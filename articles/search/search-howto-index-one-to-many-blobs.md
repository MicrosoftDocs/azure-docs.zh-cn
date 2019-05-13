---
title: 从 Azure Blob 索引器进行全文搜索-Azure 搜索索引文档中包含多个 blob 编制索引搜索
description: 爬网，对于使用 Azure 搜索 Blob 索引器的文本内容的 Azure blob。 每个 blob 可能包含一个或多个 Azure 搜索索引文档。
ms.date: 05/02/2019
author: arv100kri
manager: briansmi
ms.author: arjagann
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seofeb2018
ms.openlocfilehash: 628ced069c9d32c6e874c2e36a1e3b752c476003
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024650"
---
# <a name="indexing-blobs-producing-multiple-search-documents"></a>为 blob 生成多个搜索文档编制索引
默认情况下，blob 索引器会将 blob 的内容视为单个搜索文档。 某些**parsingMode**值支持为单个 blob 可能导致多个搜索文档的方案。 不同类型的**parsingMode**允许超过一个搜索文档从 blob 索引器提取是：
+ `delimitedText`
+ `jsonArray`
+ `jsonLines`

## <a name="one-to-many-document-key"></a>一个多文档键
由文档键唯一标识每个文档都显示在 Azure 搜索索引。 

如果指定了不分析模式，并且没有显式的映射，Azure 搜索索引中的键字段自动[映射](search-indexer-field-mappings.md)`metadata_storage_path`属性作为键。 此映射可确保每个 blob 显示为 distinct 搜索文档。

在使用任何上面列出的分析模式时，一个 blob 将映射到"多"搜索文档，使文档键仅基于 blob 元数据不合适。 若要解决此约束，Azure 搜索是能够生成从 blob 中提取每个单个实体的"多"文档键。 此属性名为`AzureSearch_DocumentKey`并添加到从 blob 中提取每个单个实体。 此属性的值保证是唯一的每个单个实体_跨 blob_ ，实体将显示为单独的搜索文档。

默认情况下，当指定的键索引字段没有显式字段映射时，`AzureSearch_DocumentKey`映射到它，则使用`base64Encode`字段映射函数。

## <a name="example"></a>示例
假设你已定义具有以下字段的索引定义：
+ `id`
+ `temperature`
+ `pressure`
+ `timestamp`

和 blob 容器包含 blob 具有以下结构：

_Blob1.json_

    { "temperature": 100, "pressure": 100, "timestamp": "2019-02-13T00:00:00Z" }
    { "temperature" : 33, "pressure" : 30, "timestamp": "2019-02-14T00:00:00Z" }

_Blob2.json_

    { "temperature": 1, "pressure": 1, "timestamp": "2018-01-12T00:00:00Z" }
    { "temperature" : 120, "pressure" : 3, "timestamp": "2013-05-11T00:00:00Z" }

当创建索引器和设置**parsingMode**到`jsonLines`-而无需指定任何显式字段映射的键字段，下面的映射将隐式应用
    
    {
        "sourceFieldName" : "AzureSearch_DocumentKey",
        "targetFieldName": "id",
        "mappingFunction": { "name" : "base64Encode" }
    }

此安装程序会导致包含以下信息 (为简洁起见缩短 base64 编码 id) 的 Azure 搜索索引

| id | 温度 | 压力 | timestamp |
|----|-------------|----------|-----------|
| aHR0...YjEuanNvbjsx | 100 | 100 | 2019-02-13T00:00:00Z |
| aHR0...YjEuanNvbjsy | 33 | 30 | 2019-02-14T00:00:00Z |
| aHR0...YjIuanNvbjsx | 第 | 第 | 2018-01-12T00:00:00Z |
| aHR0...YjIuanNvbjsy | 120 | 3 | 2013-05-11T00:00:00Z |

## <a name="custom-field-mapping-for-index-key-field"></a>索引键字段的自定义字段映射

假设相同的索引定义与前面的示例，假设你的 blob 容器包含 blob 具有以下结构：

_Blob1.json_

    recordid, temperature, pressure, timestamp
    1, 100, 100,"2019-02-13T00:00:00Z" 
    2, 33, 30,"2019-02-14T00:00:00Z" 

_Blob2.json_

    recordid, temperature, pressure, timestamp
    1, 1, 1,"2018-01-12T00:00:00Z" 
    2, 120, 3,"2013-05-11T00:00:00Z" 

当你创建使用索引器`delimitedText` **parsingMode**，它可能会感觉自然设置到的键字段的字段映射函数，如下所示：

    {
        "sourceFieldName" : "recordid",
        "targetFieldName": "id"
    }

但是，此映射将_不_导致在索引中，显示 4 个文档，因为`recordid`字段不是唯一_跨 blob_。 因此，我们建议你要使用的应用中的隐式字段映射`AzureSearch_DocumentKey`于"多"分析模式的键索引字段的属性。

如果你确实想要设置显式字段映射，请确保_sourceField_是为每个单个实体不同**跨所有 blob**。

> [!NOTE]
> 使用的方法`AzureSearch_DocumentKey`确保每个提取实体的唯一性是可能发生变更，因此您不应依赖于它的值为应用程序的需求。

## <a name="see-also"></a>另请参阅

+ [Azure 搜索中的索引器](search-indexer-overview.md)
+ [使用 Azure 搜索为 Azure Blob 存储编制索引](search-howto-index-json-blobs.md)
+ [使用 Azure 搜索 Blob 索引器为 CSV Blob 编制索引](search-howto-index-csv-blobs.md)
+ [使用 Azure 搜索 blob 索引器的 JSON blob 编制索引](search-howto-index-json-blobs.md)

## <a name="NextSteps"></a>后续步骤
* 若要详细了解 Azure 搜索，请参阅[搜索服务页](https://azure.microsoft.com/services/search/)。