---
title: 数据提取
titleSuffix: Language Understanding - Azure Cognitive Services
description: 从与意图和实体的查询文本文本中提取数据。 了解可以提取的数据类型从语言理解 (LUIS)。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: diberry
ms.openlocfilehash: 15d6b0d28f926bdb39b35b763b89422cddcccc84
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150682"
---
# <a name="extract-data-from-utterance-text-with-intents-and-entities"></a>从与意图和实体的查询文本文本中提取数据
使用 LUIS 可以从用户的自然语言陈述中获取信息。 信息以一种程序、应用程序或聊天机器人能够使用其来采取操作的方式进行提取。 在以下部分中，通过 JSON 示例了解从意向和实体返回了什么数据。

最难提取的数据是机器学习的数据，因为它不是确切的文本匹配。 机器学习[实体](luis-concept-entity-types.md)的数据提取需要作为[创作周期](luis-concept-app-iteration.md)的一部分，直到你确信已接收到所需的数据。

## <a name="data-location-and-key-usage"></a>数据位置和密钥用法
LUIS 从已发布的[终结点](luis-glossary.md#endpoint)提供数据。 HTTPS 请求（POST 或 GET）包含陈述以及一些可选配置，例如暂存或生产环境。

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

在编辑 LUIS 应用时，`appID` 可在该 LUIS 应用的“设置”页上找到，也可在 URL 中找到（在 `/apps/` 之后）。 `subscription-key` 是用于查询应用的终结点密钥。 虽然可以在学习 LUIS 时使用免费的创作/初学者密钥，但是将终结点密钥更改为支持[所需 LUIS 用法](luis-boundaries.md#key-limits)的密钥非常重要。 `timezoneOffset` 的单位是分钟。

HTTPS 响应包含 LUIS 可基于当前发布的暂存或生产终结点的模型确定的所有意向和实体信息。 终结点 URL 位于 [LUIS](luis-reference-regions.md) 网站的“管理”部分的“密钥和终结点”页上。

## <a name="data-from-intents"></a>意向中的数据
主数据是评分最高的意向名称。 使用 `MyStore` [快速入门](luis-quickstart-intents-only.md)，则终结点响应为：

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

|数据对象|数据类型|数据位置|值|
|--|--|--|--|
|意向|String|topScoringIntent.intent|"GetStoreInfo"|

如果机器人或 LUIS 调用应用基于不止一个意向评分来进行决策，则通过设置查询字符串参数 `verbose=true` 来返回所有意向的评分。 终结点响应为：

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

意向按评分从高到低排序。

|数据对象|数据类型|数据位置|值|Score|
|--|--|--|--|:--|
|意向|String|intents[0].intent|"GetStoreInfo"|0.984749258|
|意向|String|intents[1].intent|"None"|0.0168218873|

如果添加预构建的域，则意向名称指示该域，例如 `Utilties` 或 `Communication` 以及意向：

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

|域|数据对象|数据类型|数据位置|值|
|--|--|--|--|--|
|实用程序|意向|String|intents[0].intent|"<b>Utilities</b>.ShowNext"|
|通信|意向|String|intents[1].intent|<b>Communication</b>.StartOver"|
||意向|String|intents[2].intent|"None"|


## <a name="data-from-entities"></a>实体中的数据
大多数机器人和应用程序需要的都不止是意向名称。 此额外的可选数据来源于在陈述中发现的实体。 每种类型的实体返回有关匹配项的不同信息。

陈述中的单个单词或短语可以匹配多个实体。 在这种情况下，会返回每个匹配实体及其评分。

所有实体都返回在终结点响应中的“实体”数组中：

```JSON
"entities": [
  {
    "entity": "bob jones",
    "type": "Name",
    "startIndex": 0,
    "endIndex": 8,
    "score": 0.473899543
  },
  {
    "entity": "3",
    "type": "builtin.number",
    "startIndex": 16,
    "endIndex": 16,
    "resolution": {
      "value": "3"
    }
  }
]
```

## <a name="tokenized-entity-returned"></a>返回的切分后的实体
几个[区域性](luis-language-support.md#tokenization)会返回 `entity` 值[已切分](luis-glossary.md#token)的实体对象。 在实体对象中由 LUIS 返回的 startIndex 和 endIndex 不会映射到已切分的新值，而是映射到原始查询，从而以编程方式提取原始实体。 

例如，在德语中，单词 `das Bauernbrot` 会切分为 `das bauern brot`。 返回已切分的值 `das bauern brot`，并且能以编程方式从原始查询的 startIndex 和 endIndex 确定原始值，并提供 `das Bauernbrot`。

## <a name="simple-entity-data"></a>简单实体数据

[简单实体](luis-concept-entity-types.md)是一种机器学习值。 它可以是一个单词或短语。

`Bob Jones wants 3 meatball pho`

在之前的陈述中，`Bob Jones` 被标记为一个简单的 `Customer` 实体。

从终结点返回的数据包括实体名称、从陈述中发现的文本、所发现文本的位置，以及评分：

```JSON
"entities": [
  {
  "entity": "bob jones",
  "type": "Customer",
  "startIndex": 0,
  "endIndex": 8,
  "score": 0.473899543
  }
]
```

|数据对象|实体名称|值|
|--|--|--|
|简单实体|`Customer`|`bob jones`|

## <a name="composite-entity-data"></a>复合实体数据
[复合](luis-concept-entity-types.md)实体是机器学习的，并且可包括单词或短语。 例如，考虑一个预构建的 `number` 和 `Location::ToLocation` 的复合实体，其具有以下陈述：

`book 2 tickets to paris`

注意数字 `2` 和 ToLocation `paris` 之间有单词，这些单词不属于任何实体。 [LUIS](luis-reference-regions.md) 网站中的已标记话语中使用的绿色下划线指示复合实体。

![复合实体](./media/luis-concept-data-extraction/composite-entity.png)

复合实体返回在 `compositeEntities` 数组中，且该复合中的所有实体也都返回在 `entities` 数组中：

```JSON

"entities": [
    {
    "entity": "2 tickets to cairo",
    "type": "ticketInfo",
    "startIndex": 0,
    "endIndex": 17,
    "score": 0.67200166
    },
    {
    "entity": "2",
    "type": "builtin.number",
    "startIndex": 0,
    "endIndex": 0,
    "resolution": {
        "subtype": "integer",
        "value": "2"
    }
    },
    {
    "entity": "cairo",
    "type": "builtin.geographyV2",
    "startIndex": 13,
    "endIndex": 17
    }
],
"compositeEntities": [
    {
    "parentType": "ticketInfo",
    "value": "2 tickets to cairo",
    "children": [
        {
        "type": "builtin.geographyV2",
        "value": "cairo"
        },
        {
        "type": "builtin.number",
        "value": "2"
        }
    ]
    }
]
```    

|数据对象|实体名称|值|
|--|--|--|
|预构建实体 - 数量|"builtin.number"|"2"|
|Prebuilt Entity - GeographyV2|"Location::ToLocation"|"paris"|

## <a name="list-entity-data"></a>列表实体数据

[列表](luis-concept-entity-types.md)实体不进行机器学习。 它是确切的文本匹配。 列表代表列表中的项以及这些项的同义词。 LUIS 将任何列表中某个项的任何匹配项标记为响应中的实体。 同义词可位于多个列表中。

假设应用有一个名为 `Cities` 的列表，允许城市名称的变体，包括机场城市 (Sea-tac)、机场代码 (SEA)、邮政编码 (98101) 和电话区号 (206)。

|列表项|项同义词|
|---|---|
|`Seattle`|`sea-tac`、`sea`、`98101`、`206`、`+1` |
|`Paris`|`cdg`、`roissy`、`ory`、`75001`、`1`、`+33`|

`book 2 tickets to paris`

在之前的陈述中，单词 `paris` 映射至属于 `Cities` 列表实体一部分的“巴黎”项。 列表实体同时匹配项的规范化名称及其同义词。

```JSON
"entities": [
  {
    "entity": "paris",
    "type": "Cities",
    "startIndex": 18,
    "endIndex": 22,
    "resolution": {
      "values": [
        "Paris"
      ]
    }
  }
]
```

另一个使用巴黎的同义词的示例陈述：

`book 2 tickets to roissy`

```JSON
"entities": [
  {
    "entity": "roissy",
    "type": "Cities",
    "startIndex": 18,
    "endIndex": 23,
    "resolution": {
      "values": [
        "Paris"
      ]
    }
  }
]
```

## <a name="prebuilt-entity-data"></a>预构建实体数据
[预构建](luis-concept-entity-types.md)实体是基于正则表达式匹配项、使用开源 [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 项目发现的。 预构建实体返回在实体数组中，并使用前缀为 `builtin::` 的类型名称。 以下文本是一个示例陈述，其中包含返回的预构建实体：

`Dec 5th send to +1 360-555-1212`

```JSON
"entities": [
    {
      "entity": "dec 5th",
      "type": "builtin.datetimeV2.date",
      "startIndex": 0,
      "endIndex": 6,
      "resolution": {
        "values": [
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2017-12-05"
          },
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2018-12-05"
          }
        ]
      }
    },
    {
      "entity": "1",
      "type": "builtin.number",
      "startIndex": 18,
      "endIndex": 18,
      "resolution": {
        "value": "1"
      }
    },
    {
      "entity": "360",
      "type": "builtin.number",
      "startIndex": 20,
      "endIndex": 22,
      "resolution": {
        "value": "360"
      }
    },
    {
      "entity": "555",
      "type": "builtin.number",
      "startIndex": 26,
      "endIndex": 28,
      "resolution": {
        "value": "555"
      }
    },
    {
      "entity": "1212",
      "type": "builtin.number",
      "startIndex": 32,
      "endIndex": 35,
      "resolution": {
        "value": "1212"
      }
    },
    {
      "entity": "5th",
      "type": "builtin.ordinal",
      "startIndex": 4,
      "endIndex": 6,
      "resolution": {
        "value": "5"
      }
    },
    {
      "entity": "1 360 - 555 - 1212",
      "type": "builtin.phonenumber",
      "startIndex": 18,
      "endIndex": 35,
      "resolution": {
        "value": "1 360 - 555 - 1212"
      }
    }
  ]
```

## <a name="regular-expression-entity-data"></a>正则表达式实体数据
[正则表达式](luis-concept-entity-types.md)实体是基于正则表达式匹配项、使用创建实体时提供的表达式发现的。 如果将 `kb[0-9]{6}` 用作正则表达式实体定义，则下面的 JSON 响应就是一个示例陈述，其包含为查询 `When was kb123456 published?` 返回的正则表达式实体：

```JSON
{
  "query": "when was kb123456 published?",
  "topScoringIntent": {
    "intent": "FindKBArticle",
    "score": 0.933641255
  },
  "intents": [
    {
      "intent": "FindKBArticle",
      "score": 0.933641255
    },
    {
      "intent": "None",
      "score": 0.04397359
    }
  ],
  "entities": [
    {
      "entity": "kb123456",
      "type": "KB number",
      "startIndex": 9,
      "endIndex": 16
    }
  ]
}
```

## <a name="extracting-names"></a>提取名称
从陈述提取名称非常困难，因为名称几乎可以是字母和单词的任何组合。 根据要提取的名称类型，有若干选项。 以下建议不是规则，而是更多准则。

### <a name="add-prebuilt-personname-and-geographyv2-entities"></a>添加预构建的 PersonName 和 GeographyV2 实体

[PersonName](luis-reference-prebuilt-person.md) 和 [GeographyV2](luis-reference-prebuilt-geographyV2.md) 实体在某些[语言区域性](luis-reference-prebuilt-entities.md)中可用。 

### <a name="names-of-people"></a>人的姓名

人的姓名可能会带有些许格式，具体取决于语言和区域性。 使用任一预构建**[personName](luis-reference-prebuilt-person.md)** 实体或**[简单实体](luis-concept-entity-types.md#simple-entity)** 与[角色](luis-concept-roles.md)的第一个和最后一个名称。 

如果您使用简单的实体，请确保提供跨所有意图包括无使用语音样本，在不同长度的查询文本和语音样本中的不同部分中的第一个和最后一个名称的示例意向。 定期[查看](luis-how-to-review-endoint-utt.md)终结点陈述以标记未能正确预测的任何名称。

### <a name="names-of-places"></a>地名

设置和城市、 县、 状态、 省和国家/地区等的已知位置名称。 使用预生成的实体**[geographyV2](luis-reference-prebuilt-geographyv2.md)** 提取位置信息。

### <a name="new-and-emerging-names"></a>新出现的名称

一些应用需要能够找到新出现的名称，例如产品或公司。 这些类型的名称是难度最大数据提取的类型。 以开头**[简单实体](luis-concept-entity-types.md#simple-entity)** ，并添加[短语列表](luis-concept-feature.md)。 定期[查看](luis-how-to-review-endoint-utt.md)终结点陈述以标记未能正确预测的任何名称。

## <a name="pattern-roles-data"></a>模式角色数据
角色是实体间的上下文差别。

```JSON
{
  "query": "move bob jones from seattle to redmond",
  "topScoringIntent": {
    "intent": "MoveAssetsOrPeople",
    "score": 0.9999998
  },
  "intents": [
    {
      "intent": "MoveAssetsOrPeople",
      "score": 0.9999998
    },
    {
      "intent": "None",
      "score": 1.02040713E-06
    },
    {
      "intent": "GetEmployeeBenefits",
      "score": 6.12244548E-07
    },
    {
      "intent": "GetEmployeeOrgChart",
      "score": 6.12244548E-07
    },
    {
      "intent": "FindForm",
      "score": 1.1E-09
    }
  ],
  "entities": [
    {
      "entity": "bob jones",
      "type": "Employee",
      "startIndex": 5,
      "endIndex": 13,
      "score": 0.922820568,
      "role": ""
    },
    {
      "entity": "seattle",
      "type": "Location",
      "startIndex": 20,
      "endIndex": 26,
      "score": 0.948008537,
      "role": "Origin"
    },
    {
      "entity": "redmond",
      "type": "Location",
      "startIndex": 31,
      "endIndex": 37,
      "score": 0.7047979,
      "role": "Destination"
    }
  ]
}
```

## <a name="patternany-entity-data"></a>Pattern.any 实体数据
Pattern.any 实体是长度可变的实体，用于某个[模式](luis-concept-patterns.md)的模板陈述中。

```JSON
{
  "query": "where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?",
  "topScoringIntent": {
    "intent": "FindForm",
    "score": 0.999999464
  },
  "intents": [
    {
      "intent": "FindForm",
      "score": 0.999999464
    },
    {
      "intent": "GetEmployeeBenefits",
      "score": 4.883697E-06
    },
    {
      "intent": "None",
      "score": 1.02040713E-06
    },
    {
      "intent": "GetEmployeeOrgChart",
      "score": 9.278342E-07
    },
    {
      "intent": "MoveAssetsOrPeople",
      "score": 9.278342E-07
    }
  ],
  "entities": [
    {
      "entity": "understand your responsibilities as a member of the community",
      "type": "FormName",
      "startIndex": 18,
      "endIndex": 78,
      "role": ""
    }
  ]
}
```


## <a name="sentiment-analysis"></a>情绪分析
如果配置了情绪分析，LUIS json 响应会包含情绪分析内容。 请在[文本分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/)文档中详细了解情绪分析。

### <a name="sentiment-data"></a>情绪数据
情绪数据是一个介于 0 到 1 之间的分数，指示数据的正面情绪（分数接近 1）或负面情绪（分数接近 0）。

当区域性为 `en-us` 时，响应为：

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

对于所有其他区域性，响应为：

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```


### <a name="key-phrase-extraction-entity-data"></a>关键短语提取实体数据
关键短语提取实体返回陈述中的关键短语，由[文本分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/)提供。

```JSON
{
  "query": "Is there a map of places with beautiful views on a favorite trail?",
  "topScoringIntent": {
    "intent": "GetJobInformation",
    "score": 0.764368951
  },
  "intents": [
    ...
  ],
  "entities": [
    {
      "entity": "beautiful views",
      "type": "builtin.keyPhrase",
      "startIndex": 30,
      "endIndex": 44
    },
    {
      "entity": "map of places",
      "type": "builtin.keyPhrase",
      "startIndex": 11,
      "endIndex": 23
    },
    {
      "entity": "favorite trail",
      "type": "builtin.keyPhrase",
      "startIndex": 51,
      "endIndex": 64
    }
  ]
}
```

## <a name="data-matching-multiple-entities"></a>匹配多个实体的数据

LUIS 返回在陈述中发现的所有实体。 因此，机器人可能需要基于这些结果进行决策。 一个陈述中可能包含很多实体：

`book me 2 adult business tickets to paris tomorrow on air france`

LUIS 终结点可以发现不同实体中的相同数据：

```JSON
{
  "query": "book me 2 adult business tickets to paris tomorrow on air france",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 1.0
  },
  "intents": [
    {
      "intent": "BookFlight",
      "score": 1.0
    },
    {
      "intent": "Concierge",
      "score": 0.04216196
    },
    {
      "intent": "None",
      "score": 0.03610297
    }
  ],
  "entities": [
    {
      "entity": "air france",
      "type": "Airline",
      "startIndex": 54,
      "endIndex": 63,
      "score": 0.8291798
    },
    {
      "entity": "adult",
      "type": "Category",
      "startIndex": 10,
      "endIndex": 14,
      "resolution": {
        "values": [
          "adult"
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Cities",
      "startIndex": 36,
      "endIndex": 40,
      "resolution": {
        "values": [
          "Paris"
        ]
      }
    },
    {
      "entity": "tomorrow",
      "type": "builtin.datetimeV2.date",
      "startIndex": 42,
      "endIndex": 49,
      "resolution": {
        "values": [
          {
            "timex": "2018-02-21",
            "type": "date",
            "value": "2018-02-21"
          }
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Location::ToLocation",
      "startIndex": 36,
      "endIndex": 40,
      "score": 0.9730773
    },
    {
      "entity": "2",
      "type": "builtin.number",
      "startIndex": 8,
      "endIndex": 8,
      "resolution": {
        "value": "2"
      }
    },
    {
      "entity": "business",
      "type": "Seat",
      "startIndex": 16,
      "endIndex": 23,
      "resolution": {
        "values": [
          "business"
        ]
      }
    },
    {
      "entity": "2 adult business",
      "type": "TicketSeatOrder",
      "startIndex": 8,
      "endIndex": 23,
      "score": 0.8784727
    }
  ],
  "compositeEntities": [
    {
      "parentType": "TicketSeatOrder",
      "value": "2 adult business",
      "children": [
        {
          "type": "Category",
          "value": "adult"
        },
        {
          "type": "builtin.number",
          "value": "2"
        },
        {
          "type": "Seat",
          "value": "business"
        }
      ]
    }
  ]
}
```

## <a name="data-matching-multiple-list-entities"></a>匹配多个列表实体的数据

如果一个单词或短语与多个列表实体匹配，则终结点查询会返回每个列表实体。

对于查询 `when is the best time to go to red rock?`，且应用中的单词 `red` 出现在多个列表中，LUIS 会识别所有实体，并返回一组实体作为 JSON 终结点响应的一部分： 

```JSON
{
  "query": "when is the best time to go to red rock?",
  "topScoringIntent": {
    "intent": "Calendar.Find",
    "score": 0.06701678
  },
  "entities": [
    {
      "entity": "red",
      "type": "Colors",
      "startIndex": 31,
      "endIndex": 33,
      "resolution": {
        "values": [
          "Red"
        ]
      }
    },
    {
      "entity": "red rock",
      "type": "Cities",
      "startIndex": 31,
      "endIndex": 38,
      "resolution": {
        "values": [
          "Destinations"
        ]
      }
    }
  ]
}
```

## <a name="next-steps"></a>后续步骤

请参阅[添加实体](luis-how-to-add-entities.md)，详细了解如何将实体添加到 LUIS 应用。
