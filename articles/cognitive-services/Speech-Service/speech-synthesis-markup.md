---
title: 语音合成标记语言 (SSML)-语音服务
titleSuffix: Azure Cognitive Services
description: 使用语音合成标记语言来控制文本到语音转换中的发音和韵律。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/13/2018
ms.author: erhopf
ms.custom: seodec18
ms.openlocfilehash: 9871e0106ee6caf11c5a1e24459fbd2044f5f3d7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021438"
---
# <a name="speech-synthesis-markup-language-ssml"></a>语音合成标记语言 (SSML)

语言合成标记语言 (SSML) 是一种基于 XML 的标记语言，它提供了一种方法来控制文本到语音翻译的发音和“韵律”。 韵律是指语音的节奏和音高 - 其音乐（如果你希望如此）。 你可以按照发音指定单词，提供用于解释数字的提示，插入暂停，控制音高、音量、速度和其他方面。 有关详细信息，请参阅[语音合成标记语言 (SSML) 版本 1.0](https://www.w3.org/TR/2009/REC-speech-synthesis-20090303/)。 

有关支持的语言、区域设置和语音（神经和标准）的完整列表，请参阅[语言支持](language-support.md#text-to-speech)。

以下各节提供常见语音合成任务的示例。

## <a name="adjust-speaking-style-for-neural-voices"></a>调整神经语音的讲话风格

可以使用 SSML 时使用某个神经语音调整的说话风格。

默认情况下，文本到语音转换服务将合成中非特定样式的文本。 神经语音扩展与 SSML`<mstts:express-as>`将文本转换为在不同用合成语音的元素样式。 目前，仅在使用这些语音支持样式标记：

* `en-US-JessaNeural` 
* `zh-CN-XiaoxiaoNeural`。

可以在句子级别应用谈到样式更改。 样式因语音。 如果不支持样式的类型，该服务将返回合成的语音作为默认的非特定样式。

| 语音 | Style | 描述 | 
|-----------|-----------------|----------|
| `en-US-JessaNeural` | type=`cheerful` | 表示为正数，并且高兴情感 |
| | type=`empathy` | 表示某种意义上的关注并了解 |
| `zh-CN-XiaoxiaoNeural` | type=`newscast` | 表示正式音，类似于新闻 |
| | type=`sentiment ` | 传达触摸消息或一篇文章 |

```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
<voice name='en-US-JessaNeural'>
<mstts:express-as type="cheerful"> 
    That'd be just amazing! 
</mstts:express-as></voice></speak>
```

## <a name="add-a-break"></a>添加停顿
```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Jessa24kRUS'>
    Welcome to Microsoft Cognitive Services <break time="100ms" /> Text-to-Speech API.
</voice> </speak>
```

## <a name="change-speaking-rate"></a>更改语速

说到速率可以应用于在 word 或句子级别的标准语音。 而谈到速率仅应用从句子层面神经语音。

```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Guy24kRUS'>
<prosody rate="+30.00%">
    Welcome to Microsoft Cognitive Services Text-to-Speech API.
</prosody></voice> </speak>
```

## <a name="pronunciation"></a>发音
```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Jessa24kRUS'>
    <phoneme alphabet="ipa" ph="t&#x259;mei&#x325;&#x27E;ou&#x325;"> tomato </phoneme>
</voice> </speak>
```

## <a name="change-volume"></a>更改音量

卷更改可以应用于在 word 或句子级别的标准语音。 而只能从句子层面神经语音到应用批量更改。

```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Jessa24kRUS'>
<prosody volume="+20.00%">
    Welcome to Microsoft Cognitive Services Text-to-Speech API.
</prosody></voice> </speak>
```

## <a name="change-pitch"></a>更改音高

间距更改可以应用于在 word 或句子级别的标准语音。 而只能从句子层面神经语音到应用间距更改。

```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
    <voice  name='en-US-Guy24kRUS'>
    Welcome to <prosody pitch="high">Microsoft Cognitive Services Text-to-Speech API.</prosody>
</voice> </speak>
```

## <a name="change-pitch-contour"></a>更改音高升降曲线

> [!IMPORTANT]
> 间距 contour 更改不支持与神经语音。

```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Jessa24kRUS'>
<prosody contour="(80%,+20%) (90%,+30%)" >
    Good morning.
</prosody></voice> </speak>
```

## <a name="use-multiple-voices"></a>使用多个语音
```xml
<speak version='1.0' xmlns="https://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='en-US-Jessa24kRUS'>
    Good morning!
</voice>
<voice  name='en-US-Guy24kRUS'>
    Good morning to you too Jessa!
</voice> </speak>
```

## <a name="next-steps"></a>后续步骤

* [语言支持：语音、区域设置、语言](language-support.md)
