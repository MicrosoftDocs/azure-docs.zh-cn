---
title: 用户标记为转录准则-语音服务
titlesuffix: Azure Cognitive Services
description: 如果想要提高识别精确度，尤其是单词中删除或替换不正确，导致的问题将想要使用你的音频数据以及用户标记为转录。 用户标记为转录有哪些？ 这很简单，它们通过字、 原义转录的音频文件。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 7f0b467284872f3d936984741c6d092705008a5a
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025918"
---
# <a name="how-to-create-human-labeled-transcriptions"></a>如何创建用户标记为转录

如果想要提高识别精确度，尤其是单词中删除或替换不正确，导致的问题将想要使用你的音频数据以及用户标记为转录。 用户标记为转录有哪些？ 这很简单，它们通过字、 原义转录的音频文件。

需要大型脚本数据示例来改进识别，建议提供的脚本数据的 10 到 1,000 小时之间。 在此页上，我们将回顾，以帮助您创建高质量转录的指导原则。 本指南分为各个区域设置，使用美国英语、 普通话中文和德语的部分。

## <a name="us-english-en-us"></a>美国英语 (en-US)

为纯文本，只使用 ASCII 字符，必须提供标记人为转录为英语的音频。 避免使用 Latin 1 或 Unicode 标点字符。 从文字处理应用程序或抓取数据从 web pages 中复制文本时，经常会无意中会添加这些字符。 如果这些字符都存在，请确保使用适当的 ASCII 替换更新这些值。

以下是一些示例：

| 要避免的字符 | 替换字符 | 说明 |
|---------------------|--------------|-------|
| "Hello world" | “Hello world” | 开始和结束引号都有相应的 ASCII 字符已替换。 |
| John 的一天 | John 的一天 | 撇号已替换的相应 ASCII 字符。 |
| it was good—no, it was great! | it was good--no, it was great! | 长破折号已替换为两个连字符。 |

### <a name="text-normalization-for-us-english"></a>对于美国英语文本规范化

文本规范化是单词转换为模型定型时使用一致的格式。 某些规范化规则自动应用于文本，但是，我们建议使用这些准则在你准备你的用户标记的脚本数据：

* 写出单词中的缩写。
* 写出单词 （如记帐条款） 中的非标准数字字符串。
* 应转录非字母字符或混合的字母数字字符，明显。
* 缩写为单词变得明显不应编辑 （例如"雷达图"、"激光打印机"、"内存"或"北约"）。
* 将写出变得明显作为与由空格分隔每个字母的不同字母的缩写。

下面是规范化的几个示例应该对脚本执行：

| 原始文本 | 经过规范化之后的文本 |
|---------------|--------------------------|
| Dr.Bruce Banner | Doctor Bruce Banner |
| James Bond, 007 | James Bond, double oh seven |
| Ke$ha | Kesha |
| How long is the 2x4 | How long is the two by four |
| The meeting goes from 1-3pm | The meeting goes from one to three pm |
| 我的血液类型是 O + | My blood type is O positive |
| water 是 H20 | 使用复杂度为 O H 2 |
| 播放通过 Van Halen OU812 | 播放 O U 通过 Van Halen 8 1 2 |
| UTF-8 with BOM | U T F 8 with BOM |

以下的规范化规则自动应用到转录：

* 使用小写字母。
* 删除所有标点符号字以内的撇号除外。
* 扩展到朗读单词/窗体，如美元金额，数字。

下面是规范化的几个示例录音上自动执行：

| 原始文本 | 经过规范化之后的文本 |
|---------------|--------------------------|
| "Holy cow!" said Batman. | holy cow said batman |
| "What?" said Batman's sidekick, Robin. | what said batman's sidekick robin |
| Go get -em! | go get em |
| I'm double-jointed | I'm double jointed |
| 104 Elm Street | one oh four Elm street |
| Tune to 102.7 | tune to one oh two seven |
| Pi is about 3.14 | pi is about three point one four |
It costs $3.14| it costs three fourteen |

## <a name="mandarin-chinese-zh-cn"></a>普通话中文 (zh cn)

用户标记为转录普通话中文音频必须是 utf-8 编码的字节顺序标记。 避免使用半角标点字符。 这些字符可以包含无意中准备字处理程序中的数据或擦除网页中的数据时。 如果这些字符都存在，请确保使用适当的全角替换更新这些值。

以下是一些示例：

| 要避免的字符 | 替换字符 | 说明 |
|---------------------|--------------|-------|
| "你好" | "你好" | 开始和结束引号都与相应的字符已替换。 |
| 需要什么帮助? | 需要什么帮助？ | 问号已替换合适的字符。 |

### <a name="text-normalization-for-mandarin-chinese"></a>对于中文普通话文本规范化

文本规范化是单词转换为模型定型时使用一致的格式。 某些规范化规则自动应用于文本，但是，我们建议使用这些准则在你准备你的用户标记的脚本数据：

* 写出单词中的缩写。
* 用口语形式写数字字符串。

下面是规范化的几个示例应该对脚本执行：

| 原始文本 | 经过规范化之后的文本 |
|---------------|--------------------------|
| 我今年21 | 我今年二十一 |
| 3号楼504 | 三号 楼 五 零 四 |

以下的规范化规则自动应用到转录：

* 删除所有标点
* 扩展到口头形式的数字
* 将全角字符字母都转换为半宽字母
* 对所有英语单词使用大写字母

下面是规范化的几个示例录音上自动执行：

| 原始文本 | 经过规范化之后的文本 |
|---------------|--------------------------|
| 3.1415 | 三 点 一 四 一 五 |
| ￥3.5 | 三 元 五 角 |
| w f y z |W F Y Z |
| 1992年8月8日 | 一 九 九 二 年 八 月 八 日 |
| 你吃饭了吗? | 你 吃饭 了 吗 |
| 下午5:00的航班 | 下午 五点 的 航班 |
| 我今年21岁 | 我 今年 二十 一 岁 |

## <a name="german-de-de-and-other-languages"></a>德语 (DE-DE) 和其他语言

德语 （和其他非英语或音频普通话中文语言） 标记人为转录必须是 utf-8 编码的字节顺序标记。 应为每个音频文件提供一个用户标记的脚本。

### <a name="text-normalization-for-german"></a>德语文本规范化

文本规范化是单词转换为模型定型时使用一致的格式。 某些规范化规则自动应用于文本，但是，我们建议使用这些准则在你准备你的用户标记的脚本数据：

*   编写为小数点"，"和不"。"。
*   编写作为时间分隔符":"和不"。"(例如：12:00 Uhr)。
*   不替换“ca.” 等缩写。 我们建议使用完整的口头的形式。
*   删除四个主要的数学运算符（+、-、\* 和 /）。 我们建议将它们替换书面形式为:"加号，""减，""错误，"和"geteilt。"
*   删除比较运算符 (=、 <、 和 >)。 我们建议其替换为“gleich”、“kleiner als”和“grösser als”。
*   编写包含小数，如 3/4 书面形式 (例如:"drei viertel"而不是 3/4)。
*   "-"符号替换为其编写窗体"欧元。"

下面是规范化的几个示例应该对脚本执行：

| 原始文本 | 用户规范化后的文本 | 系统规范化后的文本 |
|---------------|-------------------------------|---------------------------------|
| Es ist 12.23 Uhr | Es ist 12:23 Uhr | es ist zwölf uhr drei und zwanzig uhr |
| {12.45} | {12,45} | zwölf komma vier fünf |
| 2 + 3 - 4 | 2 plus 3 minus 4 | zwei plus drei minus vier |

以下的规范化规则自动应用到转录：

* 使用小写字母的所有文本。
* 删除所有标点，包括各种类型的引号 （"test"、 测试、"测试"和 «测试» 是确定）。
* 放弃此组中的任何特殊字符的行:，¤ リ...第节 © ª ¬® ° ² 为 μ × ÿ Ø¬¬。
* 展开到口头形式，包括美元或欧元的金额的数字。
* 接受变元音仅适用于 a、 o 和您。 其他人"th"将替换为或被丢弃。

下面是规范化的几个示例录音上自动执行：

| 原始文本 | 经过规范化之后的文本 |
|---------------|--------------------------|
| Frankfurter Ring | frankfurter ring |
| ¡Eine Frage! | eine frage |
| wir, haben | wir haben |

## <a name="next-steps"></a>后续步骤

* [准备和测试你的数据](how-to-custom-speech-test-data.md)
* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)
