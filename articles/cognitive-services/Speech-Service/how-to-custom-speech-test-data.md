---
title: 准备测试数据以自定义语音的语音服务
titlesuffix: Azure Cognitive Services
description: 是否要测试以查看如何准确 Microsoft 语音识别或培训自己的模型，你将需要 （以音频和/或文本的形式） 的数据。 在此页上，我们将介绍类型的数据、 如何使用它们，以及如何对其进行管理。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 1e689d7ce65fda43e5657383ed44890c90c095cd
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025879"
---
# <a name="prepare-data-for-custom-speech"></a>准备数据以自定义语音

是否要测试以查看如何准确 Microsoft 语音识别或培训自己的模型，你将需要的音频和文本形式的数据。 在此页上，我们将介绍类型的数据、 如何使用它们，以及如何对其进行管理。

## <a name="data-types"></a>数据类型

此表列出了接受的数据类型时应使用每种数据类型，以及建议的数量。 创建一个模型需要不是每个数据类型。 根据是否要创建一个测试或定型模型，数据要求会有所不同。

| 数据类型 | 使用的测试 | 数量 | 针对用于定型 | 数量 |
|-----------|-----------------|----------|-------------------|----------|
| [音频](#audio-data-for-testing) | 是<br>用于可视化检查 | 5 + 音频文件 | 否 | N/a |
| [音频 + 标记人为的学习记录](#audio--human-labeled-transcript-data-for-testingtraining) | 是<br>用于评估准确性 | 0.5-5 小时的音频 | 是 | 1-1,000 小时的音频 |
| [相关的文本](##related-text-data-for-training) | 否 | N/a | 是 | 1-200 MB 的相关的文本 |

应按类型到数据集分组并以 zip 文件上传文件。 每个数据集只能包含一种数据类型。

## <a name="upload-data"></a>上传数据

如果你已准备好将数据上传，请单击**将数据上传**以启动向导并创建第一个数据集。 你将需要选择语音数据类型的数据集，允许你将数据上传之前。

![选择音频从语音门户](./media/custom-speech/custom-speech-select-audio.png)

上传每个数据集必须满足的要求您选择的数据类型。 请务必正确地设置格式数据，然后将其上载。 这可确保由自定义语音服务准确地处理数据。 以下各节中列出了要求。

上传你的数据集后，您有几个选项：

* 您可以导航到**测试**选项卡上，并直观检查仅限音频 + 人标记为脚本数据。
* 您可以导航到**培训**选项卡和我们音频 + 人工听录数据或相关的文本数据来训练自定义模型。

## <a name="audio-data-for-testing"></a>音频数据以供测试

音频数据，最适合于测试 Microsoft 的基线语音到文本模型或自定义模型的准确性。 请记住，音频数据用于检查特定模型的性能方面的语音的准确性。 如果想要量化模型的准确性，使用[音频 + 人标记为脚本数据](#audio--human-labeled-transcript-data-for-testingtraining)。

使用此表以确保您的音频文件格式正确的与自定义语音一起使用：

| 属性 | 值 |
|----------|-------|
| 文件格式 | RIFF (WAV) |
| 采样速率 | 8,000 Hz 或 16,000 hz 为单位 |
| 声道 | 1（单音） |
| 每个音频的最大长度 | 2 小时 |
| 示例格式 | PCM，16 位 |
| 存档格式 | .zip |
| 最大存档大小 | 2 GB |

如果您的音频不满足这些属性，或者你想要检查它是否按，我们建议下载[sox](http://sox.sourceforge.net)若要检查或将音频转换。 以下是如何可以通过命令行完成每个活动的一些示例：

| 活动 | 描述 | Sox 命令 |
|----------|-------------|-------------|
| 检查音频格式 | 使用此命令检查的音频文件格式。 | `soxi <filename>.wav` |
| 将音频格式转换为 | 使用此命令将音频文件转换为单通道，16 位 48khz。 | `sox <filename>.wav -b 16 -3 signed-integer -c l -r 48k -t wav <filename>.wav` |

## <a name="audio--human-labeled-transcript-data-for-testingtraining"></a>测试/培训的音频 + 人标记为脚本数据

若要测量的 Microsoft 的语音到文本准确性准确性处理音频文件时，必须提供用于比较的用户标记为转录 （字）。 通常较长时间标记人为脚本时，有必要评估准确性并训练用例的模型。 请记住，识别中的改进才会尽量与提供的数据。 出于此原因，很重要上, 传仅高质量的学习记录。  

| 属性 | 值 |
|----------|-------|
| 文件格式 | RIFF (WAV) |
| 采样速率 | 8,000 Hz 或 16,000 hz 为单位 |
| 声道 | 1（单音） |
| 每个音频的最大长度 | 60 s |
| 示例格式 | PCM，16 位 |
| 存档格式 | .zip |
| 最大 zip 大小 | 2 GB |

如需进行改进识别 word 删除和替换、 大量数据的解决问题。 通常情况下，建议为大约 10 到 1,000 个小时的音频提供通过字转录。 应在单个纯文本文件中包含所有 WAV 文件的听录。 听录文件的每一行应包含一个音频文件的名称，后接相应的听录。 文件名和听录应以制表符 (\t) 分隔。

  例如：
```
  speech01.wav  speech recognition is awesome
  speech02.wav  the quick brown fox jumped all over the place
  speech03.wav  the lazy dog was not amused
```
> [!NOTE]
> 听录应编码为 UTF-8 字节顺序标记 (BOM)。

听录内容应经过文本规范化，以便可由系统处理。 但是，用户在将数据上传到自定义语音服务之前，必须完成一些重要的规范化操作。 准备你个转录时要使用的相应语言，请参阅[如何创建用户标记的脚本](how-to-custom-speech-human-labeled-transcriptions.md)

音频文件和相应转录已收集后，它们应打包为一个.zip 文件上传到自定义语音门户之前。 这是具有三个音频文件和一个用户标记为脚本文件的示例数据集：

![选择音频从语音门户](./media/custom-speech/custom-speech-audio-transcript-pairs.png)

## <a name="related-text-data-for-training"></a>用于训练的相关的文本数据

如果你有产品名称或功能是唯一的并且你想要确保正确识别，则必须包含用于训练的相关的文本数据。 可以提供两种类型的相关的文本数据来改进识别：

| 数据类型 | 此数据如何改进识别 |
|-----------|------------------------------------|
| 查询文本和/或句子 | 识别产品名称或一个句子的上下文中的特定于行业的词汇，这可以提高准确性。 |
| 发音 | 这些可以提高使用未定义的发音不常见的条款、 首字母缩写词或其他单词的发音。 |

语音样本可以提供为一个或多个文本文件。 越接近数据是什么将会被朗读的文本，准确性得到了改进的可能性就越大。 应为单个文本文件提供发音。 所有内容可以打包为单个 zip 文件并上传到自定义语音门户。

### <a name="guidelines-to-create-an-utterances-file"></a>创建语音样本文件的指导原则

若要创建使用相关的文本的自定义模型，您将需要提供的示例查询文本列表。 这些查询文本不需要完整的句子或通过编程方式更正，但它们必须准确地反映朗读的输入希望在生产环境中。 如果您希望进行更多的权重的某些术语，可以将若干个短句添加到相关的数据文件包含这些特定术语的。

使用此表以确保你的语音样本的相关的数据文件的格式正确：

| 属性 | 值 |
|----------|-------|
| 文本编码 | UTF-8 BOM |
| 每行的话语数 | 第 |
| 文件大小上限 | 200 MB |

此外，你将需要考虑以下限制：

* 避免重复四次以上的字符。 例如:"aaaa"或"uuuu"。
* 不要使用特殊字符或 utf-8 字符 U + 00A1 上面。
* Uri 将被拒绝。

### <a name="guidelines-to-create-a-pronunciation-file"></a>若要创建发音文件的准则

如果有不常见术语，而无需你的用户将会遇到或使用标准发音，可以提供要提高识别的自定义发音文件。

> [!IMPORTANT]
> 建议不要使用此功能，alter 常见单词的发音。

这包括用于每个语音的样本和自定义发音的示例：

| 口头形式 | 识别显示窗体 |
|--------------|--------------------------|
| 三个 c p o | 3CPO |  
| c n t k | CNTK |
| i 三重 e | IEE |

口头的形式是拼写的拼音序列。它可以包含字母、 单词、 音节或所有这三个的组合。

自定义的发音现已推出英语 (EN-US) 和德语 (DE-DE)。 下表显示受支持的字符的语言：

| 语言 | 区域设置 | 字符 |
|----------|--------|------------|
| 英语 | en-US | a、 b、 c、 d、 e、 f、 g、 h、 i，j、 k、 l、 m、 n、 o、 p、 q、 r、 s、 t、 u、 v、 w、 x、 y、 z |
| 德语 | de-DE | ä，ö，ü，a、 b、 c、 d、 e、 f、 g、 h、 i，j、 k、 l、 m、 n、 o、 p、 q、 r、 s、 t、 u、 v、 w、 x、 y、 z |

使用此表以确保你的发音的相关的数据文件的格式正确。 发音文件很小，并且不应超过几个 Kb。

| 属性 | 值 |
|----------|-------|
| 文本编码 | Utf-8 BOM （ANSI 也支持英语） |
| # of 每行的发音 | 1 |
| 文件大小上限 | 1 MB (免费版层级 1 KB) |

## <a name="next-steps"></a>后续步骤

* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)
