---
title: 短语列表-语音服务
titlesuffix: Azure Cognitive Services
description: 了解如何提供与短语列表使用语音服务`PhraseListGrammar`对象以改进语音到文本识别结果。
services: cognitive-services
author: rhurey
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 5/02/2019
ms.author: rhurey
ms.openlocfilehash: 576d3c4a70c8870a31bc352b9f7723d2c2e69854
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025801"
---
# <a name="phrase-lists-for-speech-to-text"></a>语音到文本的短语列表

通过提供与短语列表语音服务，可以提高语音识别的准确性。 短语列表用于确定音频数据，例如一个人的名称或某个特定位置中的已知的短语。

例如，如果您有一个命令"移动到"和"Ward"，可能就读的可能目标可以添加的"移动到 Ward"条目。 添加一个短语将增加的可能性，当"移动到 Ward"将会被识别而不是"推动"识别音频。

将单个单词或完整的短语可以添加到短语列表。 识别过程如果音频中包含的完全匹配项，则使用短语列表中的条目。 如果短语列表中包含"到 Ward 移动"，并捕获该短语生成上一示例中，为"移动到缓慢"，则识别结果将为"缓慢移动到 Ward"。

## <a name="how-to-use-phrase-lists"></a>如何使用短语列表

下面的示例演示了如何以生成短语列表使用`PhraseListGrammar`对象。

```C++
auto phraselist = PhraseListGrammar::FromRecognizer(recognizer);
phraselist->AddPhrase("Move to Ward");
phraselist->AddPhrase("Move to Bill");
phraselist->AddPhrase("Move to Ted");
```

```cs
PhraseListGrammar phraseList = PhraseListGrammar.FromRecognizer(recognizer);
phraseList.AddPhrase("Move to Ward");
phraseList.AddPhrase("Move to Bill");
phraseList.AddPhrase("Move to Ted");
```

```Python
phrase_list_grammar = speechsdk.PhraseListGrammar.from_recognizer(reco)
phrase_list_grammar.addPhrase("Move to Ward")
phrase_list_grammar.addPhrase("Move to Bill")
phrase_list_grammar.addPhrase("Move to Ted")
```

```JavaScript
var phraseListGrammar = SpeechSDK.PhraseListGrammar.fromRecognizer(reco);
phraseListGrammar.addPhrase("Move to Ward");
phraseListGrammar.addPhrase("Move to Bill");
phraseListGrammar.addPhrase("Move to Ted");
```

```Java
PhraseListGrammar phraseListGrammar = PhraseListGrammar.fromRecognizer(recognizer);
phraseListGrammar.addPhrase("Move to Ward");
phraseListGrammar.addPhrase("Move to Bill");
phraseListGrammar.addPhrase("Move to Ted");
```

>[!Note]
> 语音服务将用于匹配语音短语列表的最大数目为 1024年。

此外可清除与相关联的短语`PhraseListGrammar`通过调用 clear （）。

```C++
phraselist->Clear();
```

```cs
phraseList.Clear();
```

```Python
phrase_list_grammar.clear()
```

```JavaScript
phraseListGrammar.clear();
```

```Java
phraseListGrammar.clear();
```

> [!NOTE]
> 将更改为`PhraseListGrammar`对象或重新连接到语音服务后面的下一步识别上生效。

## <a name="next-steps"></a>后续步骤

* [语音 SDK 参考文档](speech-sdk.md)
