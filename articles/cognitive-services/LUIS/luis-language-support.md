---
title: 语言支持
titleSuffix: Azure Cognitive Services
description: LUIS 在服务中具有多种功能。 并非所有功能都会同等地以各种语言提供。 请确保你所定位的语言文化支持你感兴趣的功能。 LUIS 应用特定于区域性，一旦设置即无法更改。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 03/19/2019
ms.author: diberry
ms.openlocfilehash: 8f067bc005c4de9ddc87ed598b1717f8fbb29a6a
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072378"
---
# <a name="language-and-region-support-for-luis"></a>LUIS 的语言和区域支持

LUIS 在服务中具有多种功能。 并非所有功能都会同等地以各种语言提供。 请确保你所定位的语言文化支持你感兴趣的功能。 LUIS 应用特定于区域性，一旦设置即无法更改。

## <a name="multi-language-luis-apps"></a>多语言 LUIS 应用

如果需要多语言 LUIS 客户端应用程序（例如聊天机器人），可通过几种方法实现。 如果 LUIS 支持所有语言，则需面向每种语言开发一个 LUIS 应用。 每个 LUIS 应用都具有唯一的应用 ID 和终结点日志。 如果需要为 LUIS 不支持的语言提供语言理解，可使用 [Microsoft Translator API](../Translator/translator-info-overview.md) 将表述翻译成受支持的语言，将表述提交到 LUIS 终结点，然后接收生成的分数。

## <a name="languages-supported"></a>支持的语言

LUIS 理解以下语言：

| 语言 |区域设置  |  预生成域 | 预生成实体 | 短语列表建议 | \**[文本分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/text-analytics-supported-languages)<br>（情绪和<br>关键字）|
|--|--|:--:|:--:|:--:|:--:|
| 美国英语 |`en-US` | ✔ | ✔  |✔|✔|
| *[中文](#chinese-support-notes) |`zh-CN` | ✔ | ✔ |✔|-|
| 荷兰语 |`nl-NL` |-|  -   |-|✔|
| 法语(法国) |`fr-FR` |-| ✔ |✔ |✔|
| 法语(加拿大) |`fr-CA` |-|   -   |-|✔|
| 德语 |`de-DE` |-| ✔ |✔ |✔|
| 意大利语 |`it-IT` |-| ✔ |✔|✔|
| *[日语](#japanese-support-notes) |`ja-JP` |-| ✔ |✔|仅关键短语|
| 韩语 |`ko-KR` |-|   -   |-|仅关键短语|
| 葡萄牙语(巴西) |`pt-BR` |-| ✔ |✔ |并非所有亚区域性|
| 西班牙语(西班牙) |`es-ES` |-| ✔ |✔|✔|
| 西班牙语(墨西哥)|`es-MX` |-|  -   |✔|✔|
| 土耳其语 | `tr-TR` |-|-|-|仅情绪|


[预生成实体](luis-reference-prebuilt-entities.md)和[预生成域](luis-reference-prebuilt-domains.md)具有不同的语言支持。

### <a name="chinese-support-notes"></a>*中文支持说明

 - 在 `zh-cn` 区域性中，LUIS 要求简体中文字符集，而不是繁体字符集。
 - 意向、实体、功能和正则表达式的名称可采用中文或罗马字符。
 - 请参阅[预生成的域引用](luis-reference-prebuilt-domains.md)有关中支持预生成的域的信息`zh-cn`区域性。
<!--- When writing regular expressions in Chinese, do not insert whitespace between Chinese characters.-->

### <a name="japanese-support-notes"></a>*日语支持说明

 - 由于 LUIS 不提供句法分析，并且不能理解敬语和非正式日语之间的差异，因此需要将不同的正式程度作为培训示例整合到应用程序中。
     - でございます 与 です 不同。
     - です 与 だ 不同。

### <a name="text-analytics-support-notes"></a>\*\*文本分析支持说明
文本分析包含 keyPhrase 预生成的实体和情绪分析。 仅葡萄牙语支持亚区域性：`pt-PT` 和 `pt-BR`。 主区域性级别支持所有其他区域性。 详细了解文本分析[支持的语言](https://docs.microsoft.com/azure/cognitive-services/text-analytics/text-analytics-supported-languages)。

### <a name="speech-api-supported-languages"></a>语音 API 支持的语言
请参阅语音[支持的语言](https://docs.microsoft.com/azure/cognitive-services/Speech/api-reference-rest/supportedlanguages##interactive-and-dictation-mode)，了解语音听写模式语言。

### <a name="bing-spell-check-supported-languages"></a>必应拼写检查支持的语言
请参阅必应拼写检查[支持的语言](https://docs.microsoft.com/azure/cognitive-services/bing-spell-check/bing-spell-check-supported-languages)，了解支持的语言和状态。

## <a name="rare-or-foreign-words-in-an-application"></a>应用程序中的罕见字词或外来字词
在 `en-us` 区域性中，LUIS 可学习区分大多数英文字词，包括俚语。 在 `zh-cn` 区域性中，LUIS 可学习区分大多数中文字符。 如果在 `en-us` 或 `zh-cn` 中使用一个罕见字词或字符，并且 LUIS 似乎无法识别该字词或字符，则可将该字词或字符添加到[短语列表功能](luis-how-to-add-features.md)。 例如，应将超出应用程序区域性的字词（即外来字词）添加到短语列表功能。 应将此短语列表标记为不可互换，以指示罕见字词集组成 LUIS 应学会识别的类，但它们不是同义词，也不能彼此互换。

### <a name="hybrid-languages"></a>混合语言
混合语言混含两个区域性的字词，如英语和中文。 由于单个应用仅基于单个区域性，因此 LUIS 不支持此类语言。

## <a name="tokenization"></a>词汇切分
为了执行机器学习，LUIS 基于区域性将表述拆分成[词法单元](luis-glossary.md#token)。

|语言|  每个空格或特殊字符 | 字符级|复合词|[返回的切分后的实体](luis-concept-data-extraction.md#tokenized-entity-returned)
|--|:--:|:--:|:--:|:--:|
|中文||✔||✔|
|荷兰语|||✔|✔|
|美国英语|✔ ||||
|法语 (fr-FR)|✔||||
|法语 (fr-CA)|✔||||
|德语|||✔|✔|
|意大利语|✔||||
|日语||||✔|
|韩语||✔||✔|
|葡萄牙语(巴西)|✔||||
|西班牙语 (es-ES)|✔||||
|西班牙语 (es-MX)|✔||||

### <a name="custom-tokenizer-versions"></a>自定义标记器版本

以下区域性具有自定义标记器版本：

|环境|Version|目的|
|--|--|--|
|德语<br>`de-de`|1.0.0|基于单词拆分使用机器学习基于标记化器尝试分解复合单词按其单个组件对其进行标记。<br>如果用户输入`Ich fahre einen krankenwagen`作为查询文本，它诉诸于`Ich fahre einen kranken wagen`。 允许的标记`kranken`和`wagen`独立地为不同的实体。|
|德语<br>`de-de`|1.0.2|基于字词的拆分在空间上进行标记。<br> 如果用户输入`Ich fahre einen krankenwagen`作为查询文本，它将保持的单个标记。 因此`krankenwagen`标记作为单个实体。 |

### <a name="migrating-between-tokenizer-versions"></a>标记器版本之间进行迁移
<!--
Your first choice is to change the tokenizer version in the app file, then import the version. This action changes how the utterances are tokenized but allows you to keep the same app ID. 

Tokenizer JSON for 1.0.0. Notice the property value for  `tokenizerVersion`. 

```JSON
{
    "luis_schema_version": "3.2.0",
    "versionId": "0.1",
    "name": "german_app_1.0.0",
    "desc": "",
    "culture": "de-de",
    "tokenizerVersion": "1.0.0",
    "intents": [
        {
            "name": "i1"
        },
        {
            "name": "None"
        }
    ],
    "entities": [
        {
            "name": "Fahrzeug",
            "roles": []
        }
    ],
    "composites": [],
    "closedLists": [],
    "patternAnyEntities": [],
    "regex_entities": [],
    "prebuiltEntities": [],
    "model_features": [],
    "regex_features": [],
    "patterns": [],
    "utterances": [
        {
            "text": "ich fahre einen krankenwagen",
            "intent": "i1",
            "entities": [
                {
                    "entity": "Fahrzeug",
                    "startPos": 23,
                    "endPos": 27
                }
            ]
        }
    ],
    "settings": []
}
```

Tokenizer JSON for version 1.0.1. Notice the property value for  `tokenizerVersion`. 

```JSON
{
    "luis_schema_version": "3.2.0",
    "versionId": "0.1",
    "name": "german_app_1.0.1",
    "desc": "",
    "culture": "de-de",
    "tokenizerVersion": "1.0.1",
    "intents": [
        {
            "name": "i1"
        },
        {
            "name": "None"
        }
    ],
    "entities": [
        {
            "name": "Fahrzeug",
            "roles": []
        }
    ],
    "composites": [],
    "closedLists": [],
    "patternAnyEntities": [],
    "regex_entities": [],
    "prebuiltEntities": [],
    "model_features": [],
    "regex_features": [],
    "patterns": [],
    "utterances": [
        {
            "text": "ich fahre einen krankenwagen",
            "intent": "i1",
            "entities": [
                {
                    "entity": "Fahrzeug",
                    "startPos": 16,
                    "endPos": 27
                }
            ]
        }
    ],
    "settings": []
}
```
-->

在应用级别发生词汇切分。 没有支持的版本级别词汇切分。 

[导入该文件作为新的应用程序](luis-how-to-start-new-app.md#import-an-app-from-file)，而不是版本。 此操作的含义，新应用程序具有不同的应用 ID，但使用的文件中指定的标记器版本。 