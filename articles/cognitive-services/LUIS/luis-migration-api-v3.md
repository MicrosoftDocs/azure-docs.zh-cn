---
title: V2 到 V3 API 迁移
titleSuffix: Azure Cognitive Services
description: 第 3 版终结点 Api 已发生变化。 使用本指南来了解如何将迁移到版本 3 终结点 Api。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 6412f0a2e295a19f741c70e7870a4d198ee03b71
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233554"
---
# <a name="preview-migrate-to-api-version-3x--for-luis-apps"></a>预览版：迁移到的 API 版本 3.x LUIS 应用

查询预测终结点 Api 已发生变化。 使用本指南来了解如何将迁移到版本 3 终结点 Api。 

此 V3 API 提供了以下新功能，包括重大 JSON 请求和/或响应更改： 

* [外部实体](#external-entities-passed-in-at-prediction-time)
* [动态列表](#dynamic-lists-passed-in-at-prediction-time)
* [预生成的实体的 JSON 更改](#prebuilt-entities-with-new-json)

<!--
* [Multi-intent detection of utterance](#detect-multiple-intents-within-single-utterance)
-->

查询预测终结点[请求](#request-changes)并[响应](#response-changes)有重大更改，以支持更高版本，列出包括以下新功能：

* [响应对象更改](#top-level-json-changes)
* [实体而不是实体名称的角色名称引用](#entity-role-name-instead-of-entity-name)
* [若要将标记语音样本中的实体的属性](#marking-placement-of-entities-in-utterances)

以下 LUIS 功能**不支持**V3 API 中：

* 必应拼写检查 V7

[参考文档](https://aka.ms/luis-api-v3)适用于 V3。

## <a name="prebuilt-entities-with-new-json"></a>使用新 JSON 预生成的实体

V3 响应对象更改包括[预生成的实体](luis-reference-prebuilt-entities.md)。 

## <a name="request-changes"></a>请求更改 

### <a name="query-string-parameters"></a>查询字符串参数

V3 API 具有不同的查询字符串参数。

|参数名称|Type|Version|目的|
|--|--|--|--|
|`query`|string|仅 V3|**在 V2**，将语音样本来进行预测处于`q`参数。 <br><br>**在 V3**，在传递功能`query`参数。|
|`show-all-intents`|boolean|仅 V3|返回所有与中的相应分数意向**prediction.intents**对象。 作为对象的父文件夹中返回意向`intents`对象。 这允许以编程方式访问而无需查找数组中的意图： `prediction.intents.give`。 在 V2 中，这些已返回数组中。 |
|`verbose`|boolean|V2 & V3|**V2 中**，当返回设置为 true，所有预测意向。 如果你需要所有预测的意向，使用 V3 param 的`show-all-intents`。<br><br>**V3 中**，此参数仅提供实体的实体预测的元数据详细信息。  |

<!--
|`multiple-segments`|boolean|V3 only|Break utterance into segments and predict each segment for intents and entities.|
-->


### <a name="the-query-prediction-json-body-for-the-post-request"></a>查询预测 JSON 正文`POST`请求

```JSON
{
    "query":"your utterance here",
    "options":{
        "timezoneOffset": "-8:00"
    },
    "externalEntities":[],
    "dynamicLists":[]
}
```

## <a name="response-changes"></a>响应更改

在查询响应对 JSON 进行更改，以允许更高版本以编程方式访问最常用的数据。 

### <a name="top-level-json-changes"></a>顶部级别的 JSON 更改

V2 的顶级 JSON 属性，当`verbose`设置为 true，返回所有意图和其评分`intents`属性：

```JSON
{
    "query":"this is your utterance you want predicted",
    "topScoringIntent":{},
    "intents":[],
    "entities":[],
    "compositeEntities":[]
}
```

V3 的顶级 JSON 属性包括：

```JSON
{
    "query": "this is your utterance you want predicted",
    "prediction":{
        "normalizedQuery": "this is your utterance you want predicted - after normalization",
        "topIntent": "intent-name-1",
        "intents": {}, 
        "entities":{}
    }
}
```

`intents`对象是未排序的列表。 不要假定中的第一个子`intents`对应于`topIntent`。 请改用`topIntent`值以查找分数：

```nodejs
const topIntentName = response.prediction.topIntent;
const score = intents[topIntentName];
```

响应 JSON 架构更改允许的：

* 清除原始查询文本之间的区别`query`，并返回预测， `prediction`。
* 预测数据更轻松地以编程方式访问。 而不是枚举访问一系列 V2 中，您可以访问的值**名为**意向和实体。 对于预测的实体角色，因为它是唯一的整个应用程序，则返回角色名称。
* 数据类型，如果确定，会遵循。 不能再作为字符串返回的数字。
* 在返回第一个优先级预测信息和其他元数据之间的差异`$instance`对象。 

### <a name="access-instance-for-entity-metadata"></a>访问`$instance`实体元数据

如果您需要实体元数据，需要使用查询字符串`verbose=true`标志和响应包含中的元数据`$instance`对象。 以下部分中的 JSON 响应中显示了示例。

### <a name="each-predicted-entity-is-represented-as-an-array"></a>数组形式表示预测的每个实体

`prediction.entities.<entity-name>`对象包含一个数组，因为可以将语音样本中超过一次预测每个实体。 

### <a name="list-entity-prediction-changes"></a>列出实体的预测更改

已更改为数组的数组列表实体预测的 JSON:

```JSON
"entities":{
    "my_list_entity":[
        ["canonical-form-1","canonical-form-2"],
        ["canonical-form-2"]
    ]
}
```
每个内部数组对应于将语音样本中的文本。 内部对象是一个数组，因为相同的文本可以出现在列表中实体的多个子列表。 

时之间的映射`entities`对象传递给`$instance`对象，该列表实体预测的保留对象的顺序。

```nodejs
const item = 0; // order preserved, use same enumeration for both
const predictedCanonicalForm = entities.my_list_entity[item];
const associatedMetadata = entities.$instance.my_list_entity[item];
```

### <a name="entity-role-name-instead-of-entity-name"></a>实体而不是实体名称的角色名称 

在 V2 中，`entities`正在的唯一标识符的实体名称返回预测的所有实体的数组。 在 V3，如果该实体使用角色，并且该预测是针对实体的角色中，主标识符是角色名称。 这是可能的因为在整个应用程序包括其他模型 （目的，实体） 的名称必须是实体的角色名称唯一。

在下面的示例： 考虑包含文本、 查询文本`Yellow Bird Lane`。 此文本预测为自定义`Location`实体的角色的`Destination`。

|查询文本的文本|实体名称|角色名称|
|--|--|--|
|`Yellow Bird Lane`|`Location`|`Destination`|

在 V2 中，实体由标识_实体名称_与作为对象的属性的角色：

```JSON
"entities":[
    {
        "entity": "Yellow Bird Lane",
        "type": "Location",
        "startIndex": 13,
        "endIndex": 20,
        "score": 0.786378264,
        "role": "Destination"
    }
]
```

V3，在引用实体_实体角色_，如果该预测是针对角色：

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ]
}
```

V3 中, 相同的结果，与`verbose`标志，用于返回实体的元数据：

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ],
    "$instance":{
        "Destination": [
            {
                "role": "Destination",
                "type": "Location",
                "text": "Yellow Bird Lane",
                "startIndex": 25,
                "length":16,
                "score": 0.9837309,
                "modelTypeId": 1,
                "modelType": "Entity Extractor"
            }
        ]
    }
}
```

## <a name="external-entities-passed-in-at-prediction-time"></a>在预测时传递的外部实体

外部实体使 LUIS 应用程序能够识别并在运行时，可用作针对现有实体的功能标记实体。 这允许您将查询发送到你的预测终结点之前，使用你自己的单独和自定义实体提取器。 这是在查询预测终结点，因为不需要重新训练并发布您的模型。

客户端应用程序通过管理实体匹配并确定该匹配实体的查询文本中的位置，然后将发送该信息与请求一起提供其自己的实体提取器。 

外部实体是用于仍在使用作为与其他模型，如角色、 复合和其他信号的同时扩展任何实体类型的机制。

这非常有用的实体，只能在查询预测运行时具有可用数据。 此类型的数据的示例数据或特定于每个用户一直在变化。 您可以扩展 LUIS 联系人实体使用的外部用户的联系人列表信息。 

### <a name="entity-already-exists-in-app"></a>实体已存在于应用程序中

值`entityName`的终结点请求 POST 正文中传递的外部实体必须已存在于受过培训且已发布应用程序在发出请求的时间。 支持所有类型、 实体类型并不重要。

### <a name="first-turn-in-conversation"></a>第一个会话中打开

用户在其中输入以下信息不完整的聊天机器人对话中第一个查询文本，请考虑：

`Send Hazem a new message`

LUIS 从聊天机器人的请求可以传入 POST 正文中的信息有关`Hazem`以便直接与匹配作为一个用户的联系人信息。

```json
    "externalEntities": [
        {
            "entityName":"contacts",
            "startIndex": 5,
            "entityLength": 5,
            "resolution": {
                "employeeID": "05013",
                "preferredContactType": "TeamsChat"
            }
        }
    ]
```

预测响应包括该外部实体，与所有其他预测实体，因为它定义在请求中。  

### <a name="second-turn-in-conversation"></a>第二个会话中打开

聊天机器人的下一个用户语句使用更模糊的术语：

`Send him a calendar reminder for the party.`

在上一语音样本，使用查询文本`him`对的引用作为`Hazem`。 对话式聊天机器人，在 POST 正文中，可以将映射`him`从第一个查询文本，提取的实体值`Hazem`。

```json
    "externalEntities": [
        {
            "entityName":"contacts",
            "startIndex": 5,
            "entityLength": 3,
            "resolution": {
                "employeeID": "05013",
                "preferredContactType": "TeamsChat"
            }
        }
    ]
```

预测响应包括该外部实体，与所有其他预测实体，因为它定义在请求中。  

#### <a name="resolution"></a>解决方法

_可选_`resolution`属性将返回在预测响应中，您可以在与外部实体，相关联的元数据中传递，然后接收其响应中返回。 

主要用途是将扩展预生成的实体，但并不局限于该实体类型。 

`resolution`属性可以是一个数字、 字符串、 对象或数组：

* "Dallas"
* {"text": "value"}
* 12345 
* ["a", "b", "c"]


## <a name="dynamic-lists-passed-in-at-prediction-time"></a>在预测时传递中的动态列表

动态列表使你可以扩展现有的训练和已发布列表实体，已在 LUIS 应用。 

当你列表的实体值需要定期更改时，请使用此功能。 此功能可以进行扩展的列表中已受过培训且已发布实体：

* 在查询预测终结点请求的时间。
* 针对单个请求。

列表实体可以是空 LUIS 应用程序中，但它必须存在。 LUIS 应用中的列表实体不更改，但在终结点的预测功能经过扩展，包含最多 2 个具有 1,000 个项的列表。

### <a name="dynamic-list-json-request-body"></a>动态列表 JSON 请求正文

以下 JSON 正文中发送，以将新的子列表与同义词添加到列表中，并预测文本，该列表实体`LUIS`，使用`POST`查询预测请求：

```JSON
{
    "query": "Send Hazem a message to add an item to the meeting agenda about LUIS.",
    "options":{
        "timezoneOffset": "-8:00"
    },
    "dynamicLists": [
        {
            "listEntityName":"ProductList",
            "requestLists":[
                {
                    "name": "Azure Cognitive Services",
                    "canonicalForm": "Azure-Cognitive-Services",
                    "synonyms":[
                        "language understanding",
                        "luis",
                        "qna maker"
                    ]
                }
            ]
        }
    ]
}
```

预测响应包含该列表实体，与所有其他预测实体，因为它定义在请求中。 

## <a name="timezoneoffset-renamed-to-datetimereference"></a>重命名为 datetimeReference TimezoneOffset

**在 V2**，则`timezoneOffset`[参数](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity)如果作为 GET 或 POST 请求发送该请求而不考虑为查询字符串参数，在预测请求中发送。 

**在 V3**，其中 POST 正文参数提供的相同功能`datetimeReference`。 

## <a name="marking-placement-of-entities-in-utterances"></a>将标记放置在查询文本中的实体

**在 V2**，一个实体标记中使用查询文本`startIndex`和`endIndex`。 

**在 V3**，该实体将标有`startIndex`和`entityLength`。


## <a name="next-steps"></a>后续步骤

使用 V3 API 文档，以更新现有的 REST 调用到 LUIS[终结点](https://aka.ms/luis-api-v3)Api。 