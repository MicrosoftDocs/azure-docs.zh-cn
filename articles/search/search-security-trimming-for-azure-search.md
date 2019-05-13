---
title: 用于修整结果的安全筛选器 - Azure 搜索
description: 使用安全筛选器和用户标识对 Azure 搜索内容进行访问控制。
ms.service: search
ms.topic: conceptual
services: search
ms.date: 05/02/2019
author: brjohnstmsft
ms.author: brjohnst
manager: jlembicz
ms.custom: seodec2018
ms.openlocfilehash: a222b9e506988929c25a560361611b8f78142053
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024370"
---
# <a name="security-filters-for-trimming-results-in-azure-search"></a>用于在 Azure 搜索中修整结果的安全筛选器

可以应用安全筛选器，以根据用户标识对 Azure 搜索中的搜索结果进行修整。 此搜索体验通常需要将请求搜索的任何人的标识，与包含拥有文档权限的主体的字段进行比较。 如果找到匹配项，则该用户或主体（例如组或角色）有权访问该文档。

实现安全筛选的方法之一是对相等表达式进行复杂析取：例如 `Id eq 'id1' or Id eq 'id2'`，等等。 此方法容易出错且难以维护，如果列表包含数百甚至数千个值，会将查询响应时间减慢许多秒。 

更简单快捷的方法是使用 `search.in` 函数。 如果使用 `search.in(Id, 'id1, id2, ...')` 而不是相等表达式，有望获得亚秒级响应时间。

本文介绍如何使用以下步骤实现安全筛选：
> [!div class="checklist"]
> * 创建包含主体标识符的字段 
> * 推送或更新包含相关主体标识符的现有文档
> * 发出包含 `search.in` `filter` 的搜索请求

>[!NOTE]
> 本文档未介绍检索主体标识符的过程。 应该从标识服务提供程序获取该标识符。

## <a name="prerequisites"></a>必备组件

本文假设读者拥有 [Azure 订阅](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)、[Azure 搜索服务](https://docs.microsoft.com/azure/search/search-create-service-portal)和 [Azure 搜索索引](https://docs.microsoft.com/azure/search/search-create-index-portal)。  

## <a name="create-security-field"></a>创建安全字段

文档必须包含一个指定哪些组拥有访问权限的字段。 此信息将成为筛选条件，在返回给请求发出者的结果集中选择或拒绝文档时，将以此条件为依据。
我们假设为受保护的文件创建了一个索引，每个文件可由一组不同的用户访问。
1. 将字段 `group_ids`（此处可选择任意名称）为 `Collection(Edm.String)`。 确保该字段的 `filterable` 属性设置为 `true`，以便根据用户拥有的访问权限筛选搜索结果。 例如，如果针对 `file_name` 为“secured_file_b”的文档将 `group_ids` 字段设置为 `["group_id1, group_id2"]`，则只有属于组 ID“group_id1”或“group_id2”的用户才对该文件拥有读访问权限。
   确保字段的 `retrievable` 属性设置为 `false`，以便不会将其返回为搜索请求的一部分。
2. 此外，针对此示例添加 `file_id` 和 `file_name` 字段。  

```JSON
{
    "name": "securedfiles",  
    "fields": [
        {"name": "file_id", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
        {"name": "file_name", "type": "Edm.String"},
        {"name": "group_ids", "type": "Collection(Edm.String)", "filterable": true, "retrievable": false}
    ]
}
```

## <a name="pushing-data-into-your-index-using-the-rest-api"></a>使用 REST API 将数据推送到索引中
  
向索引的 URL 终结点发出 HTTP POST 请求。 HTTP 请求的正文是一个 JSON 对象，包含要添加的文档：

```
POST https://[search service].search.windows.net/indexes/securedfiles/docs/index?api-version=2019-05-06  
Content-Type: application/json
api-key: [admin key]
```

在请求正文中，指定文档的内容：

```JSON
{
    "value": [
        {
            "@search.action": "upload",
            "file_id": "1",
            "file_name": "secured_file_a",
            "group_ids": ["group_id1"]
        },
        {
            "@search.action": "upload",
            "file_id": "2",
            "file_name": "secured_file_b",
            "group_ids": ["group_id1", "group_id2"]
        },
        {
            "@search.action": "upload",
            "file_id": "3",
            "file_name": "secured_file_c",
            "group_ids": ["group_id5", "group_id6"]
        }
    ]
}
```

如需使用组列表更新现有文档，可以使用 `merge` 或 `mergeOrUpload` 操作：

```JSON
{
    "value": [
        {
            "@search.action": "mergeOrUpload",
            "file_id": "3",
            "group_ids": ["group_id7", "group_id8", "group_id9"]
        }
    ]
}
```

有关添加或更新文档的完整详细信息，可以阅读[编辑文档](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)。
   
## <a name="apply-the-security-filter"></a>应用安全筛选器

若要基于 `group_ids` 访问权限修整文档，应发出包含 `group_ids/any(g:search.in(g, 'group_id1, group_id2,...'))` 筛选器的搜索查询，其中，'group_id1, group_id2,...' 是搜索请求发出者所属的组。
此筛选器匹配其 `group_ids` 字段包含某个给定标识符的所有文档。
有关使用 Azure 搜索搜索文档的完整详细信息，可以阅读[搜索文档](https://docs.microsoft.com/rest/api/searchservice/search-documents)。
请注意，此示例演示如何使用 POST 请求搜索文档。

发出 HTTP POST 请求：

```
POST https://[service name].search.windows.net/indexes/securedfiles/docs/search?api-version=2019-05-06
Content-Type: application/json  
api-key: [admin or query key]
```

在请求正文中指定筛选器：

```JSON
{
   "filter":"group_ids/any(g:search.in(g, 'group_id1, group_id2'))"  
}
```

应该获取 `group_ids` 包含“group_id1”或“group_id2”的文档。 换而言之，应获取请求发出者对其拥有读访问权限的文档。

```JSON
{
 [
   {
    "@search.score":1.0,
     "file_id":"1",
     "file_name":"secured_file_a",
   },
   {
     "@search.score":1.0,
     "file_id":"2",
     "file_name":"secured_file_b"
   }
 ]
}
```
## <a name="conclusion"></a>结束语

本文介绍了如何基于用户标识和 Azure 搜索 `search.in()` 函数筛选结果。 此函数可用于传入请求的用户可以与每个目标文档关联的主体标识符匹配的主体标识符。 处理搜索请求时，`search.in` 函数会筛选出任何用户主体都对其没有读访问权限的搜索结果。 主体标识符可以表示安全组、角色甚至用户自己的标识等信息。
 
## <a name="see-also"></a>另请参阅

+ [使用 Azure 搜索筛选器进行 Active Directory 基于标识的访问控制](search-security-trimming-for-azure-search-with-aad.md)
+ [Azure 搜索中的筛选器](search-filters.md)
+ [Azure 搜索操作中的数据安全性和访问控制](search-security-overview.md)