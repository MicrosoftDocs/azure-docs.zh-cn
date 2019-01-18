---
title: Graph 搜索方法 - 学术知识 API
titlesuffix: Azure Cognitive Services
description: 学术知识 API 中的 Graph 搜索方法可用于，根据特定 Graph 模式返回一组学术实体。
services: cognitive-services
author: alch-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: conceptual
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: 83f29106d72f564f894c968102b703ab6bb5d8c2
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902363"
---
# <a name="graph-search-method"></a>Graph 搜索方法

Graph 搜索 REST API 用于根据给定 Graph 模式返回一组学术实体。  响应是一组符合用户指定约束的 Graph 路径。 Graph 路径是 Graph 节点和边缘的交错序列，格式为“v0, e0, v1, e1, ..., vn”，其中 v0 是路径的起始节点。
<br>

**REST 终结点：**  
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?
```   
<br>

## <a name="request-parameters"></a>请求参数  
名称     | 值 | 必需？  | Description
-----------|-----------|---------|--------
**mode**       | 文本字符串 | 是 | 要使用的模式的名称。 值为 json 或 lambda。

必须通过 HTTP POST 请求调用 Graph 搜索方法。 post 请求应包括内容类型请求头：application/json。

##### <a name="json-search"></a>JSON 搜索 

对于 JSON 搜索，POST 正文是 JSON 对象。 JSON 对象描述了含用户指定约束的路径模式（对于 JSON 搜索，请参阅 [JSON 对象规范](JSONSearchSyntax.md)）。


##### <a name="lambda-search"></a>lambda 搜索

对于 lambda 搜索，POST 正文是纯文本字符串。 POST 正文是 LIKQ lambda 查询字符串，即为一个 C# 语句（对于 lambda 搜索，请参阅[查询字符串规范](LambdaSearchSyntax.md)）。 

<br>
## <a name="response-json"></a>响应 (JSON)
名称 | Description
-------|-----   
**results** | 一组与查询表达式匹配的 0 个或多个实体。 每个实体都包含请求获取的属性值。 如果已成功处理请求，此字段就会显示。
**error** | HTTP 状态代码。 如果请求失败，此字段就会显示。
**message** | 错误消息。 如果请求失败，此字段就会显示。

如果无法在 _800 毫秒_ 内处理查询，就会返回 _超时_ 错误。 

<br>
#### <a name="example"></a>示例：

##### <a name="json-search"></a>JSON 搜索
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?mode=json
```
<br>
对于 JSON 搜索，若要获得标题包含“Graph 引擎”且由“bin shao”撰写的论文，可以指定如下查询。

```JSON
{
  "path": "/paper/AuthorIDs/author",
  "paper": {
    "type": "Paper",
    "NormalizedTitle": "graph engine",
    "select": [
      "OriginalTitle"
    ]
  },
  "author": {
    "return": {
      "type": "Author",
      "Name": "bin shao"
    }
  }
}
```

查询输出的是一组 Graph 路径。 Graph 路径是一组与查询路径中指定的节点对应的节点对象。 这些节点对象至少有一个表示实体 ID 的属性 CellID。 其他属性的检索方法为，通过[遍历操作对象](JSONSearchSyntax.md)的 select 运算符来指定属性名。

```JSON
{
  "Results": [
    [
      {
        "CellID": 2160459668,
        "OriginalTitle": "Trinity: a distributed graph engine on a memory cloud"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 2171539317,
        "OriginalTitle": "A distributed graph engine for web scale RDF data"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 2411554868,
        "OriginalTitle": "A distributed graph engine for web scale RDF data"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 73304046,
        "OriginalTitle": "The Trinity graph engine"
      },
      {
        "CellID": 2093502026
      }
    ]
  ]
}
 ```

##### <a name="lambda-search"></a>lambda 搜索 

```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?mode=lambda
```
<br>
对于 lambda 搜索，若要获取给定论文的作者 ID，可以编写如下查询。

```
MAG.StartFrom(@"{
    type  : ""Paper"",
    match : {
        NormalizedTitle : ""trinity: a distributed graph engine on a memory cloud""
    }
}").FollowEdge("AuthorIDs").VisitNode(Action.Return)
```

lambda 搜索查询输出的也是一组 Graph 路径：

```JSON
{
  "Results": [
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2142490828
      }
    ],
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2116756368
      }
    ],
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2093502026
      }
    ]
  ]
}
```
 
## <a name="graph-schema"></a>Graph 架构

Graph 架构对于编写 Graph 搜索查询非常有用。 如下图所示。

![Microsoft Academic Graph 架构](./Images/AcademicGraphSchema.png)
