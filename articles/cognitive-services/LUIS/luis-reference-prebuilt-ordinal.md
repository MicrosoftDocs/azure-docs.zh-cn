---
title: 序号预生成实体
titleSuffix: Azure
description: 本文包含了语言理解 (LUIS) 中的序号预构建实体的信息。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 752a26b21854ec9030fc1945024ae461445815a9
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146718"
---
# <a name="ordinal-prebuilt-entity-for-a-luis-app"></a>LUIS 应用的序号预生成实体
序号是一个集合内的对象的数字表示形式：`first`、`second`、`third`。 此实体已定型，因此不需要将包含序号的陈述示例添加到应用程序意向中。 [许多语言区域](luis-reference-prebuilt-entities.md)都支持序号实体。 

## <a name="types-of-ordinal"></a>序号类型
序号托管在 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-Numbers.yaml#L45) GitHub 存储库中

## <a name="resolution-for-prebuilt-ordinal-entity"></a>预构建序号实体的解析

### <a name="api-version-2x"></a>API 版本 2.x

以下示例展示了 **builtin.ordinal** 实体的解析。

```json
{
  "query": "Order the second option",
  "topScoringIntent": {
    "intent": "OrderFood",
    "score": 0.9993253
  },
  "intents": [
    {
      "intent": "OrderFood",
      "score": 0.9993253
    },
    {
      "intent": "None",
      "score": 0.05046708
    }
  ],
  "entities": [
    {
      "entity": "second",
      "type": "builtin.ordinal",
      "startIndex": 10,
      "endIndex": 15,
      "resolution": {
        "value": "2"
      }
    }
  ]
}
```

### <a name="preview-api-version-3x"></a>预览 API 版本 3.x

以下 JSON 是与`verbose`参数设置为`false`:

```json
{
    "query": "Order the second option",
    "prediction": {
        "normalizedQuery": "order the second option",
        "topIntent": "None",
        "intents": {
            "None": {
                "score": 0.7124502
            }
        },
        "entities": {
            "ordinal": [
                {
                    "offset": 2,
                    "relativeTo": "start"
                }
            ]
        }
    }
}
```

以下 JSON 是与`verbose`参数设置为`true`:

```json
{
    "query": "Order the second option",
    "prediction": {
        "normalizedQuery": "order the second option",
        "topIntent": "None",
        "intents": {
            "None": {
                "score": 0.7124502
            }
        },
        "entities": {
            "ordinal": [
                {
                    "offset": 2,
                    "relativeTo": "start"
                }
            ],
            "$instance": {
                "ordinal": [
                  {
                    "type": "builtin.ordinal",
                    "text": "second",
                    "startIndex": 10,
                    "length": 6,
                    "modelTypeId": 2,
                    "modelType": "Prebuilt Entity Extractor",
                    "recognitionSources": [
                        "model"
                    ]
                  }
                ]
            }
        }
    }
}
```

## <a name="next-steps"></a>后续步骤

了解如何[百分比](luis-reference-prebuilt-percentage.md)，[电话号码](luis-reference-prebuilt-phonenumber.md)，并[温度](luis-reference-prebuilt-temperature.md)实体。 
