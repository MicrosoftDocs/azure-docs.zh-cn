---
title: 整形程序认知搜索技能 - Azure 搜索
description: 从非结构化数据中提取元数据和结构化信息，并将其整形为 Azure 搜索扩充管道中的复杂类型。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 5267f81c9886e2d1d8d62c134156aedb3b2b8763
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023729"
---
#   <a name="shaper-cognitive-skill"></a>整形程序认知技能

**整形程序**技能将合并到多个输入[复杂类型](search-howto-complex-data-types.md)，可以更高版本中扩充管道引用。 借助整形程序技能，可实质上创建结构、定义该结构的成员名称，并为每个成员分配值。 合并字段搜索方案中有用的示例包括组合到单个结构、 城市和到单个结构或名称的状态和出生日期来唯一标识的单个结构的第一个和最后一个名称。

API 版本确定调整您的深度可以实现。 

| API 版本 | 调整行为 | 
|-------------|-------------------|
| 2019-05-06-preview REST API 版本 （不支持.NET SDK） | 复杂对象时，跨多个级别深，合一**整形程序**技能定义。 |
| 2019-05-06 * * （已公开发布），2017年-11-11-预览版| 复杂对象，一个层次深度。 多级别形状需要将几个整形程序步骤链接在一起。|

在预览**整形程序**中所示的技能[方案 3](#nested-complex-types)，添加一个新的可选*sourceContext*到输入的属性。 *源*并*sourceContext*属性是互相排斥。 如果输入是在该技能的上下文，只需使用*源*。 如果输入是在*不同*比技能上下文，使用上下文*sourceContext*。 *SourceContext*要求与作为源进行寻址的特定元素定义的嵌套的输入。 

在响应中，对于所有的 API 版本，输出名称是始终为"output"。 在内部，管道可以映射不同的名称，例如"analyzedText"，如下面的示例中所示，但**整形程序**技能本身在响应中返回"output"。 如果正在调试大量文档并发现存在命名差异，或者要生成自定义技能并自行构建响应，这一点非常重要。

> [!NOTE]
> **整形程序**技能未绑定到认知服务 API，不会收取使用它。 但是，你仍然应该[附加认知服务资源](cognitive-search-attach-cognitive-services.md)，以覆盖**免费**资源选项，该选项限制你每天进行少量的每日扩充。

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ShaperSkill

## <a name="scenario-1-complex-types"></a>方案 1： 复杂类型

请思考这样一种情况：想要创建名为 analyzedText 的结构，该结构具有两个成员：分别为 text 和 sentiment。 在 Azure 搜索索引，多个部分组成的可搜索字段称为*复杂类型*并通常创建时源数据具有相应的复杂结构映射到它。

但是，用于创建复杂类型的另一种方法是通过**整形程序**技能。 通过在技能组合中包含此技能，方面的技能处理过程中的内存中操作可以输出嵌套结构，然后可以映射到索引中的复杂类型的数据的形状。 

以下示例技能定义作为输入提供成员名称。 


```json
{
  "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
  "context": "/document/content/phrases/*",
  "inputs": [
    {
      "name": "text",
      "source": "/document/content/phrases/*"
    },
    {
      "name": "sentiment",
      "source": "/document/content/phrases/*/sentiment"
    }
  ],
  "outputs": [
    {
      "name": "output",
      "targetName": "analyzedText"
    }
  ]
}
```

### <a name="sample-index"></a>示例索引

技能组合调用索引器，并且索引器需要一个索引。 在索引中的复杂字段表示形式看起来像下面的示例。 

```json

    "name": "my-index",
    "fields": [
        {   "name": "myId", "type": "Edm.String", "key": true, "filterable": true   },
        {   "name": "analyzedText", "type": "Edm.ComplexType",
            "fields": [{
                    "name": "text",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true  },
          {
                    "name": "sentiment",
                    "type": "Edm.Double",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
```

### <a name="skill-input"></a>技能输入

提供可使用此输入传入 JSON 文档**整形程序**技能可能是：

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "text": "this movie is awesome",
                "sentiment": 0.9
            }
        }
    ]
}
```


### <a name="skill-output"></a>技能输出

整形程序技能使用 *text* 和 *sentiment* 组合元素生成一个名为 *analyzedText* 的新元素。 此输出符合索引架构。 它将导入并在 Azure 搜索索引中编制索引。

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
            "analyzedText": 
              {
                "text": "this movie is awesome" ,
                "sentiment": 0.9
              }
           }
      }
    ]
}
```

## <a name="scenario-2-input-consolidation"></a>方案 2： 输入的合并

在另一个示例中，假设处于管道处理的不同阶段，已提取书名以及该书不同页面上的章节标题。 现在可创建由这些不同输入组成的单个结构。

**整形程序**技能定义对于此方案看起来像下面的示例：

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "source": "/document/content/pages/*/chapterTitles/*/title"
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>技能输出
在这种情况下，**整形程序**平展所有章节标题，创建单个数组。 

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                        "Start young",
                        "Laugh often",
                        "Eat, sleep and exercise"
                    ]
                }
            }
        }
    ]
}
```

<a name="nested-complex-types"></a>

## <a name="scenario-3-input-consolidation-from-nested-contexts"></a>方案 3： 输入的合并从嵌套的上下文

> [!NOTE]
> 嵌套的 api 版本中支持的结构 = 2019年-05-06-预览可在[知识存储](knowledge-store-concept-intro.md)或在 Azure 搜索索引中。

假设您有标题、 章节和书籍的内容和内容已运行实体识别和密钥的短语，并且现在需要聚合结果从不同的技能到一个形状与章节名称、 实体和关键短语。

**整形程序**技能定义对于此方案看起来像下面的示例：

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "sourceContext": "/document/content/pages/*/chapterTitles/*",
            "inputs": [
              {
                  "name": "title",
                  "source": "/document/content/pages/*/chapterTitles/*/title"
              },
              {
                  "name": "number",
                  "source": "/document/content/pages/*/chapterTitles/*/number"
              }
            ]
        }

    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>技能输出
在这种情况下，**整形程序**创建复杂类型。 此结构存在内存中。 如果你想要将其保存到的知识存储，应在您的技能以用于定义存储特征创建投影。

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                      { "title": "Start young", "number": 1},
                      { "title": "Laugh often", "number": 2},
                      { "title": "Eat, sleep and exercise", "number: 3}
                    ]
                }
            }
        }
    ]
}
```

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [如何使用复杂类型](search-howto-complex-data-types.md)
+ [知识应用商店概述](knowledge-store-concept-intro.md)
+ [如何开始使用知识 Store](knowledge-store-howto.md)