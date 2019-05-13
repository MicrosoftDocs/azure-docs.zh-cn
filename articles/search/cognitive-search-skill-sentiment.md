---
title: 情绪认知搜索技能 - Azure 搜索
description: 从 Azure 搜索扩充管道中的文本提取积极-消极情绪分数。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: f52f5200f33d11db44d94b5a5f26d246f711e224
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023807"
---
#   <a name="sentiment-cognitive-skill"></a>情绪认知技能

情绪技能可沿着一个积极-消极连续统评估非结构化文本，对于每个记录，会返回一个介于 0 和 1 之间的数字评分。 评分接近 1 代表积极的情绪，评分接近 0 代表消极的情绪。 此技能使用认知服务中的[文本分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview)提供的机器学习模型。

> [!NOTE]
> 当您展开作用域提高频率的处理，添加更多文档，或添加更多的 AI 算法，你将需要[附加可计费的认知服务资源](cognitive-search-attach-cognitive-services.md)。 调用认知服务中的 API，以及在 Azure 搜索中的文档破解阶段提取图像时，会产生费用。 提取文档中的文本不会产生费用。
>
> 执行的内置技能收费的现有[认知服务付款现转价格](https://azure.microsoft.com/pricing/details/cognitive-services/)。 介绍了图像提取定价[Azure 搜索定价页](https://go.microsoft.com/fwlink/?linkid=2042400)。


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.SentimentSkill

## <a name="data-limits"></a>数据限制
记录的最大大小应为 5000 个字符，通过 `String.Length` 进行测量。 如果需要在将数据发送到情绪分析器之前拆分数据，请使用[文本拆分技能](cognitive-search-skill-textsplit.md)。


## <a name="skill-parameters"></a>技能参数

参数区分大小写。

| 参数名称 |                      |
|----------------|----------------------|
| defaultLanguageCode | （可选）要应用到未显式指定语言的文档的语言代码。 <br/> 请参阅[支持的语言的完整列表](../cognitive-services/text-analytics/text-analytics-supported-languages.md) |

## <a name="skill-inputs"></a>技能输入 

| 输入名称 | 说明 |
|--------------------|-------------|
| text | 要分析的文本。|
| languageCode  |  （可选）表示记录的语言的字符串。 如果未指定此属性，则默认值为“en”。 <br/>请参阅[支持的语言的完整列表](../cognitive-services/text-analytics/text-analytics-supported-languages.md)。|

## <a name="skill-outputs"></a>技能输出

| 输出名称 | 说明 |
|--------------------|-------------|
| score | 介于 0 和 1 之间的值，表示所分析的文本的情绪。 值接近 0 代表消极情绪，接近 0.5 代表中性情绪，接近 1 代表积极的情绪。|


##  <a name="sample-definition"></a>示例定义

```json
{
    "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
    "inputs": [
        {
            "name": "text",
            "source": "/document/content"
        },
        {
            "name": "languageCode",
            "source": "/document/languagecode"
        }
    ],
    "outputs": [
        {
            "name": "score",
            "targetName": "mySentiment"
        }
    ]
}
```

##  <a name="sample-input"></a>示例输入

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "text": "I had a terrible time at the hotel. The staff was rude and the food was awful.",
                "languageCode": "en"
            }
        }
    ]
}
```


##  <a name="sample-output"></a>示例输出

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "score": 0.01
            }
        }
    ]
}
```

## <a name="notes"></a>说明
如果为空，则不为这些记录返回情绪分数。

## <a name="error-cases"></a>错误案例
如果某种语言不受支持，则生成错误，并且不会返回情绪分数。

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
