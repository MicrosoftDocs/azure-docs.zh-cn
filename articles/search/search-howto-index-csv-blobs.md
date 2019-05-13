---
title: 使用 Azure 搜索 Blob 索引器为 CSV Blob 编制索引 - Azure 搜索
description: 使用 Azure 搜索索引在 Azure Blob 存储中抓取 CSV Blob 以进行全文搜索。 索引器可自动为所选数据源（如 Azure Blob 存储）引入数据。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 193ed7099293fb1ee4c056abcc5c2f34d78627b7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024721"
---
# <a name="indexing-csv-blobs-with-azure-search-blob-indexer"></a>使用 Azure 搜索 Blob 索引器对 CSV blob 编制索引
默认情况下，[Azure 搜索 Blob 索引器](search-howto-indexing-azure-blob-storage.md)会将分隔的文本 blob 分析为单个文本块。 但在 blob 含有 CSV 数据的情况下，通常希望将 blob 中的每一行视为一个单独文档。 例如，给定以下带分隔符的文本，可能要将其分析为两个文档，每个文档包含“id”、“datePublished”和“tags”字段： 

    id, datePublished, tags
    1, 2016-01-12, "azure-search,azure,cloud" 
    2, 2016-07-07, "cloud,mobile" 

在本文中，您将了解如何使用 Azure 搜索 blob indexerby 设置分析 CSV blob`delimitedText`分析模式。 

`delimitedText`分析模式当前处于公共预览状态，不建议用于生产工作负荷。

> [!NOTE]
> 请按照中的索引器配置建议[到多索引](search-howto-index-one-to-many-blobs.md)输出从一个 Azure blob 的多个搜索文档。

## <a name="setting-up-csv-indexing"></a>设置 CSV 索引
若要对 CSV blob 编制索引，请使用 `delimitedText` 分析模式创建或更新索引器定义：  

    {
      "name" : "my-csv-indexer",
      ... other indexer properties
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "firstLineContainsHeaders" : true } }
    }

有关创建索引器 API 的更多详细信息，请参阅[创建索引器](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。

`firstLineContainsHeaders` 指示每个 blob 的第一行（非空）包含标头。
如果 blob 未包含初始标头行，则应在索引器配置中指定标头： 

    "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } } 

可以使用 `delimitedTextDelimiter` 配置设置来自定义分隔符字符。 例如：

    "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextDelimiter" : "|" } }

> [!NOTE]
> 目前，仅支持 UTF-8 编码。 如果需要支持其他编码，请在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 上为其投票。

> [!IMPORTANT]
> 当使用分隔文本分析模式时，Azure 搜索假定数据源中的所有 blob 都将是 CSV。 如果需要在同一数据源中支持混用 CSV 和非 CSV blob，请在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 上为其投票。
> 
> 

## <a name="request-examples"></a>请求示例
汇总后，以下是完整的有效负载示例。 

数据源： 

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06-Preview
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "<optional, my-folder>" }
    }   

索引器：

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06-Preview
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-csv-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } }
    }

## <a name="help-us-make-azure-search-better"></a>帮助我们改进 Azure 搜索
如果有功能请求或改进建议，请在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search/) 上提供相关意见。

