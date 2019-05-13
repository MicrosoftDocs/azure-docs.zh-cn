---
title: 排查常见的搜索索引器问题 - Azure 搜索
description: 修复 Azure 搜索中索引器的错误和常见问题，包括数据源连接、防火墙和丢失的文档。
author: mgottein
manager: cgronlun
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: magottei
ms.custom: seodec2018
ms.openlocfilehash: 81987115ba0406e704f2198061c6ee3cae8a72df
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024695"
---
# <a name="troubleshooting-common-indexer-issues-in-azure-search"></a>排查 Azure 搜索中的常见索引器问题

在将数据索引到 Azure 搜索中时，索引器可能会遇到许多问题。 故障的主要类别包括：

* [连接到数据源](#data-source-connection-errors)
* [文档处理](#document-processing-errors)
* [目标为索引的文档引入](#index-errors)

## <a name="data-source-connection-errors"></a>数据源连接错误

### <a name="blob-storage"></a>Blob 存储

#### <a name="storage-account-firewall"></a>存储帐户防火墙

Azure 存储提供可配置的防火墙。 默认情况下，防火墙处于禁用状态，因此 Azure 搜索可以连接到存储帐户。

防火墙启用后，没有具体的错误消息。 通常情况下，防火墙错误类似于：`The remote server returned an error: (403) Forbidden`。

可以在[门户](https://docs.microsoft.com/azure/storage/common/storage-network-security#azure-portal)中验证防火墙是否已启用。 如果防火墙已启用，则可以通过两个选项来解决此问题：

1. 禁用防火墙，方法是：选择允许从[“所有网络”](https://docs.microsoft.com/azure/storage/common/storage-network-security#azure-portal)进行访问
1. 针对搜索服务的 IP 地址[添加一个例外](https://docs.microsoft.com/azure/storage/common/storage-network-security#managing-ip-network-rules)。 若要查找该 IP 地址，请使用以下命令：

`nslookup <service name>.search.windows.net`

例外不适用于[认知搜索](cognitive-search-concept-intro.md)。 唯一解决方法是禁用防火墙。

### <a name="cosmos-db"></a>Cosmos DB

#### <a name="indexing-isnt-enabled"></a>索引未启用

Azure 搜索对 Cosmos DB 索引存在隐式依赖。 如果在 Cosmos DB 中关闭自动索引，Azure 搜索会返回成功状态，但无法索引容器内容。 有关如何查看设置和启用索引功能的说明，请参阅[管理 Azure Cosmos DB 中的索引编制](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-indexing-policy#manage-indexing-using-azure-portal)。

## <a name="document-processing-errors"></a>文档处理错误

### <a name="unprocessable-or-unsupported-documents"></a>文档无法处理或不受支持

[显式支持可记录格式的 Blob 索引器文档](search-howto-indexing-azure-blob-storage.md#supported-document-formats)。 有时候，Blob 存储容器包含不受支持的文档。 而另一些时候，可能存在有问题的文档。 可以通过[更改配置选项](search-howto-indexing-azure-blob-storage.md#dealing-with-errors)来避免停止这些文档上的索引器：

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false, "failOnUnprocessableDocument" : false } }
}
```

### <a name="missing-document-content"></a>缺少文档内容

Blob 索引器可[查找并提取容器中 Blob 的文本](search-howto-indexing-azure-blob-storage.md#how-azure-search-indexes-blobs)。 提取文本时出现的一些问题包括：

* 文档仅包含扫描的图像。 包含扫描图像 (JPG) 之类的非文本内容的 PDF Blob 不会在标准 Blob 索引管道中生成结果。 如果图像内容包含文本元素，则可通过[认知搜索](cognitive-search-concept-image-scenarios.md)来查找并提取文本。
* Blob 索引器配置为仅索引元数据。 若要提取内容，必须将 Blob 索引器配置为[提取内容和元数据](search-howto-indexing-azure-blob-storage.md#controlling-which-parts-of-the-blob-are-indexed)：

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "dataToExtract" : "contentAndMetadata" } }
}
```

## <a name="index-errors"></a>索引错误

### <a name="missing-documents"></a>缺少文档

索引器从[数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)查找文档。 有时候，索引中似乎会缺失数据源中本应进行索引的文档。 之所以发生这些错误，有多种常见原因：

* 文档尚未进行索引。 查看门户中是否有成功的索引器运行。
* 文档在索引器运行之后已更新。 如果索引器已在[计划](https://docs.microsoft.com/rest/api/searchservice/create-indexer#indexer-schedule)之中，它最终会重新运行并选取该文档。
* 在数据源中指定的 [query](https://docs.microsoft.com/rest/api/searchservice/create-data-source#request-body-syntax) 排除了该文档。 索引器不能索引不属于数据源的文档。
* [字段映射](https://docs.microsoft.com/rest/api/searchservice/create-indexer#fieldmappings)或[认知搜索](https://docs.microsoft.com/azure/search/cognitive-search-concept-intro)已更改此文档，因此它看起来不同于预期。
* 使用[查找文档 API](https://docs.microsoft.com/rest/api/searchservice/lookup-document) 来查找文档。
