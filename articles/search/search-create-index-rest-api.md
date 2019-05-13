---
title: 快速入门：创建、 加载和查询使用 PowerShell 和 REST API-Azure 搜索索引
description: 创建、 加载和查询索引使用 PowerShell 的 Invoke-restmethod 和 Azure 搜索 REST API。
ms.date: 05/02/2019
author: heidisteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 9459ab44f366c87660297a8564534156a56777bd
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024143"
---
# <a name="quickstart-create-an-azure-search-index-using-powershell-and-the-rest-api"></a>快速入门：创建 Azure 搜索索引中使用 PowerShell 和 REST API
> [!div class="op_single_selector"]
> * [PowerShell (REST)](search-create-index-rest-api.md)
> * [C#](search-create-index-dotnet.md)
> * [Postman (REST)](search-fiddler.md)
> * [门户](search-create-index-portal.md)
> 

本文将指导你完成创建、 加载和查询 Azure 搜索的过程[索引](search-what-is-an-index.md)使用 PowerShell 和[Azure 搜索服务 REST API](https://docs.microsoft.com/rest/api/searchservice/)。 请求正文中的图片被提供作为格式正确的 JSON 内容的索引定义和可搜索内容。

## <a name="prerequisites"></a>必备组件

本快速入门使用以下服务和工具。 

[创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可以使用本快速入门的免费服务。 

[PowerShell 5.1 或更高版本](https://github.com/PowerShell/PowerShell)，并使用[Invoke-restmethod](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Utility/Invoke-RestMethod)的顺序和交互式步骤。

## <a name="get-a-key-and-url"></a>获取密钥和 URL

REST 调用需要在每个请求中使用服务 URL 和访问密钥。 搜索服务是使用这二者创建的，因此，如果向订阅添加了 Azure 搜索，则请按以下步骤获取必需信息：

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中获取 URL。 示例终结点可能类似于 `https://mydemo.search.windows.net`。

2. 在“设置” > “密钥”中，获取有关该服务的完全权限的管理员密钥。 有两个可交换的管理员密钥，为保证业务连续性而提供，以防需要滚动一个密钥。 可以在请求中使用主要或辅助密钥来添加、修改和删除对象。

![获取 HTTP 终结点和访问密钥](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

所有请求对发送到服务的每个请求都需要 API 密钥。 具有有效的密钥可以在发送请求的应用程序与处理请求的服务之间建立信任关系，这种信任关系以每个请求为基础。

## <a name="connect-to-azure-search"></a>连接到 Azure 搜索

在 PowerShell 中，创建 **$headers**要存储的内容类型和 API 密钥对象。 只需一次将此标头设置会话的持续时间内，但会将其添加到每个请求。 

```powershell
$headers = @{
   'api-key' = '<your-admin-api-key>'
   'Content-Type' = 'application/json' 
   'Accept' = 'application/json' }
```

创建 **$url**对象，它指定服务的索引集合。 `mydemo`服务名称用作占位符。 替换为它在此示例中的当前订阅中的有效的搜索服务。

```powershell
$url = "https://mydemo.search.windows.net/indexes?api-version=2019-05-06"
```

运行**Invoke-restmethod**将 GET 请求发送到服务并验证的连接。 添加**Convertto-json** ，以便您可以查看从服务发送回的响应。

```powershell
Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
```

如果该服务为空，并且没有索引，结果将类似于下面的示例。 否则，您将看到索引定义的 JSON 表示形式。

```
{
    "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes",
    "value":  [

              ]
}
```

## <a name="1---create-an-index"></a>1 - 创建索引

除非使用门户，可以加载数据之前，服务上必须存在索引。 此步骤中定义索引，并将其推送到服务。 [创建索引 (REST API)](https://docs.microsoft.com/rest/api/searchservice/create-index)用于此步骤。

所需的元素的索引包含一个名称和字段集合。 字段集合定义的结构*文档*。 每个字段有一个名称、 类型和特性，以确定如何使用它 (例如，是否是全文索引时才可搜索、 筛选或可在搜索结果中检索)。 一个索引，其中一个类型的字段中`Edm.String`必须指定为*密钥*文档标识。

此索引名为"hotels"，并且请参阅下面的字段定义。 索引定义指定[语言分析器](index-add-language-analyzers.md)为`description_fr`字段，因为它用于存储法语文本，我们将在后面的示例中添加。

将此示例粘贴到 PowerShell 来创建 **$body**对象，包含索引架构。

```powershell
$body = @"
{
    "name": "hotels",  
    "fields": [
        {"name": "hotelId", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
        {"name": "baseRate", "type": "Edm.Double"},
        {"name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false},
        {"name": "description_fr", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
        {"name": "hotelName", "type": "Edm.String", "facetable": false},
        {"name": "category", "type": "Edm.String"},
        {"name": "tags", "type": "Collection(Edm.String)"},
        {"name": "parkingIncluded", "type": "Edm.Boolean", "sortable": false},
        {"name": "smokingAllowed", "type": "Edm.Boolean", "sortable": false},
        {"name": "lastRenovationDate", "type": "Edm.DateTimeOffset"},
        {"name": "rating", "type": "Edm.Int32"},
        {"name": "location", "type": "Edm.GeographyPoint"}
    ]
}
"@
```

将 URI 设置为你的服务上的索引集合和*hotels*索引。

```powershell
$url = "https://mydemo.search.windows.net/indexes/hotels?api-version=2019-05-06"
```

运行该命令与 **$url**， **$headers**，并 **$body**在服务上创建索引。 

```powershell
Invoke-RestMethod -Uri $url -Headers $headers -Method Put -Body $body | ConvertTo-Json
```
结果应类似于此 （到前两个字段为简洁起见已截断）：

```
{
    "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes/$entity",
    "@odata.etag":  "\"0x8D6A99E2DED96B0\"",
    "name":  "hotels",
    "defaultScoringProfile":  null,
    "fields":  [
                   {
                       "name":  "hotelId",
                       "type":  "Edm.String",
                       "searchable":  false,
                       "filterable":  true,
                       "retrievable":  true,
                       "sortable":  false,
                       "facetable":  false,
                       "key":  true,
                       "indexAnalyzer":  null,
                       "searchAnalyzer":  null,
                       "analyzer":  null,
                       "synonymMaps":  ""
                   },
                   {
                       "name":  "baseRate",
                       "type":  "Edm.Double",
                       "searchable":  false,
                       "filterable":  true,
                       "retrievable":  true,
                       "sortable":  true,
                       "facetable":  true,
                       "key":  false,
                       "indexAnalyzer":  null,
                       "searchAnalyzer":  null,
                       "analyzer":  null,
                       "synonymMaps":  ""
                   },
. . .
```

> [!Tip]
> 可能还检查索引列表在门户中，或重新运行用于验证服务连接的命令，以查看进行验证， *hotels*索引集合中列出的索引。

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - 加载文档

若要将推送文档，请使用对索引的 URL 终结点的 HTTP POST 请求。 此任务的 REST api[添加、 更新或删除文档](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)。

将此示例粘贴到 PowerShell 来创建 **$body**对象，其中包含你想要上载的文档。 

此请求包括两个完整备份和一个部分的记录。 部分记录演示你可以上传不完整的文档。 `@search.action`参数指定索引如何实现。 有效值包括上传、 合并、 mergeOrUpload 和删除。 MergeOrUpload 行为会创建新文档 hotelId = 3，或更新内容，如果它已存在。

```powershell
$body = @"
{
    "value": [
        {
            "@search.action": "upload",
            "hotelId": "1",
            "baseRate": 199.0,
            "description": "Best hotel in town",
            "hotelName": "Fancy Stay",
            "category": "Luxury",
            "tags": ["pool", "view", "wifi", "concierge"],
            "parkingIncluded": false,
            "smokingAllowed": false,
            "lastRenovationDate": "2010-06-27T00:00:00Z",
            "rating": 5,
            "location": { "type": "Point", "coordinates": [-122.131577, 47.678581] }
        },
        {
            "@search.action": "upload",
            "hotelId": "2",
            "baseRate": 79.99,
            "description": "Cheapest hotel in town",
            "hotelName": "Roach Motel",
            "category": "Budget",
            "tags": ["motel", "budget"],
            "parkingIncluded": true,
            "smokingAllowed": true,
            "lastRenovationDate": "1982-04-28T00:00:00Z",
            "rating": 1,
            "location": { "type": "Point", "coordinates": [-122.131577, 49.678581] }
        },
        {
            "@search.action": "mergeOrUpload",
            "hotelId": "3",
            "baseRate": 129.99,
            "description": "Close to town hall and the river"
        }
    ]
}
"@
```

将终结点设置为*hotels*文档集合，包括在索引操作 (索引/酒店/docs/index)。

```powershell
$url = "https://mydemo.search.windows.net/indexes/hotels/docs/index?api-version=2019-05-06"
```

运行该命令与 **$url**， **$headers**，并 **$body**若要将文档加载到 hotels 索引。

```powershell
Invoke-RestMethod -Uri $url -Headers $headers -Method Post -Body $body | ConvertTo-Json
```
结果应类似于下面的示例。 您应看到状态代码 201。 所有状态代码的说明，请参阅[HTTP 状态代码 （Azure 搜索）](https://docs.microsoft.com/rest/api/searchservice/HTTP-status-codes)。

```
{
    "@odata.context":  "https://mydemo.search.windows.net/indexes(\u0027hotels\u0027)/$metadata#Collection(Microsoft.Azure.Search.V2017_11_11.IndexResult)",
    "value":  [
                  {
                      "key":  "1",
                      "status":  true,
                      "errorMessage":  null,
                      "statusCode":  201
                  },
                  {
                      "key":  "2",
                      "status":  true,
                      "errorMessage":  null,
                      "statusCode":  201
                  },
                  {
                      "key":  "3",
                      "status":  true,
                      "errorMessage":  null,
                      "statusCode":  201
                  }
              ]
}
```

## <a name="3---search-an-index"></a>3 - 搜索索引

此步骤说明如何使用索引进行查询[搜索文档 API](https://docs.microsoft.com/rest/api/searchservice/search-documents)。

将终结点设置为*hotels*文档集合，并添加**搜索**要包括查询字符串参数。 此字符串是空搜索并返回所有文档的 unranked 的列表。

```powershell
$url = 'https://mydemo.search.windows.net/indexes/hotels/docs?api-version=2019-05-06&search=*'
```

运行命令以发送 **$url**到服务。

```powershell
Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
```

结果应类似于以下输出。

```
{
    "@odata.context":  "https://mydemo.search.windows.net/indexes(\u0027hotels\u0027)/$metadata#docs(*)",
    "value":  [
                  {
                      "@search.score":  1.0,
                      "hotelId":  "1",
                      "baseRate":  199.0,
                      "description":  "Best hotel in town",
                      "description_fr":  null,
                      "hotelName":  "Fancy Stay",
                      "category":  "Luxury",
                      "tags":  "pool view wifi concierge",
                      "parkingIncluded":  false,
                      "smokingAllowed":  false,
                      "lastRenovationDate":  "2010-06-27T00:00:00Z",
                      "rating":  5,
                      "location":  "@{type=Point; coordinates=System.Object[]; crs=}"
                  },
                  {
                      "@search.score":  1.0,
                      "hotelId":  "2",
                      "baseRate":  79.99,
                      "description":  "Cheapest hotel in town",
                      "description_fr":  null,
                      "hotelName":  "Roach Motel",
                      "category":  "Budget",
                      "tags":  "motel budget",
                      "parkingIncluded":  true,
                      "smokingAllowed":  true,
                      "lastRenovationDate":  "1982-04-28T00:00:00Z",
                      "rating":  1,
                      "location":  "@{type=Point; coordinates=System.Object[]; crs=}"
                  },
                  {
                      "@search.score":  1.0,
                      "hotelId":  "3",
                      "baseRate":  129.99,
                      "description":  "Close to town hall and the river",
                      "description_fr":  null,
                      "hotelName":  null,
                      "category":  null,
                      "tags":  "",
                      "parkingIncluded":  null,
                      "smokingAllowed":  null,
                      "lastRenovationDate":  null,
                      "rating":  null,
                      "location":  null
                  }
              ]
}
```

请尝试几个其他查询示例，若要了解的语法。 可以执行字符串搜索，原义 $filter 查询，结果集，作用域搜索到特定字段，以及更多限制。

```powershell
# Query example 1
# Search the entire index for the term 'budget'
# Return only the `hotelName` field, "Roach hotel"
$url = 'https://mydemo.search.windows.net/indexes/hotels/docs?api-version=2019-05-06&search=budget&$select=hotelName'

# Query example 2 
# Apply a filter to the index to find hotels cheaper than $150 per night
# Returns the `hotelId` and `description`. Two documents match.
$url = 'https://mydemo.search.windows.net/indexes/hotels/docs?api-version=2019-05-06&search=*&$filter=baseRate lt 150&$select=hotelId,description'

# Query example 3
# Search the entire index, order by a specific field (`lastRenovationDate`) in descending order
# Take the top two results, and show only `hotelName` and `lastRenovationDate`
$url = 'https://mydemo.search.windows.net/indexes/hotels/docs?api-version=2019-05-06&search=*&$top=2&$orderby=lastRenovationDate desc&$select=hotelName,lastRenovationDate'
```
## <a name="clean-up"></a>清理 

如果不再需要应删除该索引。 一项免费服务被限制为三个索引。 您可能想要删除任何未主动使用，以便可以单步执行其他教程的索引。

```powershell
# Set the URI to the hotel index
$url = 'https://mydemo.search.windows.net/indexes/hotels?api-version=2019-05-06'

# Delete the index
Invoke-RestMethod -Uri $url -Headers $headers -Method Delete
```

## <a name="next-steps"></a>后续步骤

尝试将法语说明添加到索引。 下面的示例包括法语字符串，并演示其他搜索操作。 使用 mergeOrUpload 创建或添加到现有字段。 以下字符串需要是 utf-8 编码。

```json
{
    "value": [
        {
            "@search.action": "mergeOrUpload",
            "hotelId": "1",
            "description_fr": "Meilleur hôtel en ville"
        },
        {
            "@search.action": "merge",
            "hotelId": "2",
            "description_fr": "Hôtel le moins cher en ville"
        }
    ]
}
```
