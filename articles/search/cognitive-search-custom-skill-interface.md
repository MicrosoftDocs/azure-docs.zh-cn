---
title: 认知搜索中自定义技能的接口定义 - Azure 搜索
description: Azure 搜索的认知搜索管道中的 web-api 自定义技能的自定义数据提取接口。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 1bf42e5f418f99f5e5327d790c1adffe2357b84e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021939"
---
# <a name="how-to-add-a-custom-skill-to-a-cognitive-search-pipeline"></a>如何将自定义技能添加到认知搜索管道

Azure 搜索中的[认知搜索索引管道](cognitive-search-concept-intro.md)可以从[预定义技能](cognitive-search-predefined-skills.md)和自己创建并添加到管道中的[自定义技能](cognitive-search-custom-skill-web-api.md)进行装配。 本文介绍如何创建一个暴露接口的自定义技能，从而允许其包含在认知搜索管道中。 

通过生成自定义技能，可插入对内容唯一的转换。 自定义技能独立执行，可应用所需的任何扩充步骤。 例如，可定义特定于域的自定义实体，生成自定义分类模型来区分商业和金融合同或文档，或者添加语音识别技能来深入了解相关内容的音频文件。 有关分步示例，请参阅[示例：创建自定义技能](cognitive-search-create-custom-skill-example.md)。

 无论需要哪种自定义功能，都有一个简单明了的接口，可将自定义技能与其余扩充管道相连接。 [技能组合](cognitive-search-defining-skillset.md)中包含的唯一需求是，能够以可在技能组合内作为整体使用的方式接受输入并发出输出。 本文的重点是扩充管道所需的输入和输出格式。

## <a name="web-api-custom-skill-interface"></a>Web API 自定义技能接口

如果未在 30 秒的期限内返回响应，自定义 WebAPI 技能终结点将默认超时。 索引管道是同步的，如果未在该期限内收到响应，索引会生成超时错误。  最多可将超时时间配置为 90 秒，方法是设置超时参数：

```json
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "This skill has a 90 second timeout",
        "uri": "https://[your custom skill uri goes here]",
        "timeout": "PT90S",
```

目前，与自定义技能交互的唯一机制是通过 Web API 接口。 Web API 需求必须满足本节中所述的要求。

### <a name="1--web-api-input-format"></a>1.Web API 输入格式

Web API 必须接受要处理的一组记录。 每条记录都必须包含一个“属性包”，该属性包是提供给 Web API 的输入。 

假设要创建一个简单的扩充器来识别合同文本中提到的第一个日期。 在此示例中，技能接受单个输入 contractText 作为合同文本。 技能也具有单个输出，即合同的日期。 若要使扩充器更有趣，请将此 contractDate 以多部分复杂类型的形式返回。

你的 Web API 应该可以接收一批输入记录。 values 数组中的每个成员都表示特定记录的输入。 每条记录都需要具有以下元素：

+ recordId 成员是特定记录的唯一标识符。 当扩充器返回结果时，它必须提供此 recordId 以便允许调用方将记录结果与其输入进行匹配。

+ data 成员，基本上是每条记录的输入域的包。

更具体地说，根据上面的示例，你的 Web API 的请求应该类似于下：

```json
{
    "values": [
      {
        "recordId": "a1",
        "data":
           {
             "contractText": 
                "This is a contract that was issues on November 3, 2017 and that involves... "
           }
      },
      {
        "recordId": "b5",
        "data":
           {
             "contractText": 
                "In the City of Seattle, WA on February 5, 2018 there was a decision made..."
           }
      },
      {
        "recordId": "c3",
        "data":
           {
             "contractText": null
           }
      }
    ]
}
```
实际上，可能会调用服务的数百或数千条记录，而不仅仅是这里显示的三条记录。

### <a name="2-web-api-output-format"></a>2.Web API 输出格式

输出的格式是一组包含 recordId 和属性包的记录 

```json
{
  "values": 
  [
      {
        "recordId": "b5",
        "data" : 
        {
            "contractDate":  { "day" : 5, "month": 2, "year" : 2018 }
        }
      },
      {
        "recordId": "a1",
        "data" : {
            "contractDate": { "day" : 3, "month": 11, "year" : 2017 }                    
        }
      },
      {
        "recordId": "c3",
        "data" : 
        {
        },
        "errors": [ { "message": "contractText field required "}   ],  
        "warnings": [ {"message": "Date not found" }  ]
      }
    ]
}
```

这个具体示例仅有一个输出，但你可以输出多个属性。 

### <a name="errors-and-warning"></a>错误和警告

如之前示例所示，可返回每条记录的错误和警告消息。

## <a name="consuming-custom-skills-from-skillset"></a>从技能组合中使用自定义技能

创建 Web API 扩充器时，可将 HTTP 标头和参数描述为请求的一部分。 下面的代码片段显示了如何将请求参数和 HTTP 标头描述为技能组合定义的一部分。

```json
{
    "skills": [
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "This skill calls an Azure function, which in turn calls TA sentiment",
        "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/DateExtractor?language=en",
        "context": "/document",
        "httpHeaders": {
            "DateExtractor-Api-Key": "foo"
        },
        "inputs": [
          {
            "name": "contractText",
            "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "contractDate",
            "targetName": "date"
          }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>后续步骤

+ [示例：为翻译文本 API 创建自定义技能](cognitive-search-create-custom-skill-example.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [创建技能组合 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何映射扩充的域](cognitive-search-output-field-mapping.md)
