---
title: 文本拆分认知搜索技能 - Azure 搜索
description: 基于 Azure 搜索扩充管道中的长度，将文本分解为区块或文本页。
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
ms.openlocfilehash: 78ed2fab81bfb1562125135c5901a2de395c3843
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023937"
---
#   <a name="text-split-cognitive-skill"></a>文本拆分认知技能

文本拆分技能将文本分解为文本区块。 你可以指定是要将文件分解为句子还是特定长度的页面。 当其他技能下游有最大文本长度要求时，此技能尤其有用。 

> [!NOTE]
> 此技能未绑定到认知服务 API，你使用它无需付费。 但是，你仍然应该[附加认知服务资源](cognitive-search-attach-cognitive-services.md)，以覆盖**免费**资源选项，该选项限制你每天进行少量的每日扩充。

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.SplitSkill 

## <a name="skill-parameters"></a>技能参数

参数区分大小写。

| 参数名称     | 描述 |
|--------------------|-------------|
| textSplitMode      | “pages”或“sentences” | 
| maximumPageLength | 如果将 textSplitMode 设置为“pages”，它指的是由 `String.Length` 测量的最大页面长度。 最小值是 100。  如果 textSplitMode 设置为“pages”，则该算法将尝试将文本拆分为大小最多为“maximumPageLength”的区块。 在这种情况下，该算法将尽力断开句子边界上的句子，因此区块的大小可能略小于“maximumPageLength”。 | 
| defaultLanguageCode   | （可选）以下语言代码之一：`da, de, en, es, fi, fr, it, ko, pt` 默认为英语 (en)。 注意事项：<ul><li>如果你传递的是 languagecode-countrycode 格式，只会使用格式的 languagecode 部分。</li><li>如果语言不在上述列表中，拆分技能会在字符边界分解文本。</li><li>提供语言代码有助于避免将非空格的语言（例如，中文、日语和韩语）的单词一分为二。</li></ul>  |


## <a name="skill-inputs"></a>技能输入

| 参数名称       | 描述      |
|----------------------|------------------|
| Text  | 要拆分为子字符串的文本。 |
| languageCode  | （可选）文档的语言代码。  |

## <a name="skill-outputs"></a>技能输出 

| 参数名称     | 描述 |
|--------------------|-------------|
| textItems | 提取的子字符串数组。 |


##  <a name="sample-definition"></a>示例定义

```json
{
    "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
    "textSplitMode" : "pages", 
    "maximumPageLength": 1000,
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "textItems",
            "targetName": "mypages"
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
                "text": "This is a the loan application for Joe Romero, he is a Microsoft employee who was born in Chile and then moved to Australia…",
                "languageCode": "en"
            }
        },
        {
            "recordId": "2",
            "data": {
                "text": "This is the second document, which will be broken into several pages...",
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
                "textItems": [
                    "This is the loan…",
                    "On the second page we…"
                ]
            }
        },
        {
            "recordId": "2",
            "data": {
                "textItems": [
                    "This is the second document...",
                    "On the second page of the second doc…"
                ]
            }
        }
    ]
}
```

## <a name="error-cases"></a>错误案例
如果不支持某种语言，会生成一个警告，并在字符边界拆分文本。

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
