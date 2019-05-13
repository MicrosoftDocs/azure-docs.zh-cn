---
title: Azure 搜索中的 moreLikeThis（预览版）- Azure 搜索
description: Azure 搜索 REST API 中公开了有关 moreLikeThis 功能（预览）功能的初步文档。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: d18069335bb20f78a5bcda85eb6fcb2a5abe75f7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024671"
---
# <a name="morelikethis-in-azure-search-preview"></a>Azure 搜索中的 moreLikeThis（预览版）

`moreLikeThis=[key]` 是中的查询参数[搜索文档 API](https://docs.microsoft.com/rest/api/searchservice/search-documents) ，查找文档相似的文档键所指定的文档。 当使用 `moreLikeThis` 生成搜索请求时，使用从最匹配的给定文档中提取的搜索项来生成查询。 然后使用生成的查询来生成搜索请求。 默认情况下，所有可搜索字段的内容被视为，使用指定的任何限制字段减去`searchFields`参数。 `moreLikeThis` 参数不能与搜索参数 `search=[string]` 一起使用。

默认情况下，被视为所有的顶级可搜索字段的内容。 如果你想要改为指定特定字段，则可以使用`searchFields`参数。 

> [!NOTE]
> `moreLikeThis` 在可搜索的子字段中，预览版不起作用[复杂类型](search-howto-complex-data-types.md)。

## <a name="examples"></a>示例 

下面是 moreLikeThis 查询的一个示例。 该查询会查找其描述字段与 `moreLikeThis` 参数指定的源文档字段最相似的文档。

```
Get /indexes/hotels/docs?moreLikeThis=1002&searchFields=description&api-version=2019-05-06-Preview
```

```
POST /indexes/hotels/docs/search?api-version=2019-05-06-Preview
    {
      "moreLikeThis": "1002",
      "searchFields": "description"
    }
```

## <a name="feature-availability"></a>功能可用性

`moreLikeThis`参数是目前只有预览版 REST Api (`api-version=2019-05-06-Preview`)。

## <a name="next-steps"></a>后续步骤

可以使用任何 web 测试工具来体验此功能。  我们建议对于此练习使用 Postman。

> [!div class="nextstepaction"]
> [探索 Azure 搜索 REST Api 使用 Postman](search-fiddler.md)