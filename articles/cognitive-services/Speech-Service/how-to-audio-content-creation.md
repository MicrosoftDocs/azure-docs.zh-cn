---
title: 音频内容创建-语音服务
titleSuffix: Azure Cognitive Services
description: 音频内容创建是一个在线工具，可用于为应用和产品自定义和微调 Microsoft 的文本到语音输出。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: trbye
ms.openlocfilehash: a263e7e17cda64a8519bab215f97fdf26e88d9d2
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2020
ms.locfileid: "81402242"
---
# <a name="improve-synthesis-with-audio-content-creation"></a>通过音频内容创建改善合成

[音频内容创建](https://aka.ms/audiocontentcreation)是一个在线工具，可用于为应用和产品自定义和微调 Microsoft 的文本到语音输出。 您可以使用此工具来微调公共和自定义语音以获取更准确的自然表达式，并在云中管理输出。

音频内容创建工具基于[语音合成标记语言（SSML）](speech-synthesis-markup.md)。 为了简化自定义和优化，音频内容创建使你能够以可视方式实时检查文本到语音输出。

## <a name="how-does-it-work"></a>工作原理

此图显示了优化和导出自定义的语音到文本输出所需要执行的步骤。 使用以下链接详细了解每个步骤。

![](media/audio-content-creation/audio-content-creation-diagram.jpg)

1. 第一步是[创建 Azure 帐户、注册语音资源并获取订阅密钥](#create-a-speech-resource)。 拥有订阅密钥后，可以使用它来调用语音服务，并访问[音频内容创建](https://aka.ms/audiocontentcreation)。
2. 使用纯文本或 SSML[创建音频优化文件](#create-an-audio-tuning-file)。
3. 选择要优化的语音和语言。 音频内容创建包括[Microsoft 的文本到语音转换](language-support.md#text-to-speech)的所有声音。 您可以使用标准、神经或您自己的自定义语音。
   >[!NOTE]
   > "封闭访问" 适用于自定义的神经声音，这允许您创建类似于自然语音的高清晰声音。 有关更多详细信息，请参阅控制[过程](https://aka.ms/ignite2019/speech/ethics)。

4. 查看默认结果。 然后使用优化工具调整发音、音调、速率、intonation、语音样式等。 有关选项的完整列表，请参阅[语音合成标记语言](speech-synthesis-markup.md)。
5. 保存并[导出优化的音频](#export-tuned-audio)。 在系统中保存优化轨后，可以继续工作，并在输出上循环访问。 如果对输出满意，可以使用导出功能创建音频创建任务。 你可以查看导出任务的状态，并下载用于应用和产品的输出。
6. 最后一步是在应用和产品中使用自定义的优化声音。

## <a name="create-a-speech-resource"></a>创建语音资源

按照以下步骤创建语音资源，并将其连接到 Speech Studio。

1. 按照以下说明[注册 Azure 帐户](get-started.md#new-resource)并[创建语音资源](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started#create-the-resource)。 请确保定价层设置为**S0**。 如果正在使用其中一个神经声音，请确保在[受支持的区域](regions.md#standard-and-neural-voices)中创建资源。
2. 登录到[音频内容创建](https://aka.ms/audiocontentcreation)。
3. 选择一个现有项目，或单击 "**新建**"。
4. 你随时都可以通过 "**设置**" 选项修改订阅，该选项位于顶部导航栏中。

## <a name="create-an-audio-tuning-file"></a>创建音频优化文件

可以通过两种方式将内容引入音频内容创建工具。

**选项 1：**

1. 登录到[音频内容创建](https://aka.ms/audiocontentcreation)后，请单击 "**音频调谐**" 以创建新的音频优化文件。
2. 在编辑窗口出现时，最多可以输入10000个字符。
3. 别忘了保存。

**选项 2：**

1. 登录到[音频内容创建](https://aka.ms/audiocontentcreation)后，单击 "**上载**" 以导入一个或多个文本文件。 支持纯文本和 SSML。
2. 上传文本文件时，请确保内容满足这些要求。

   | properties | 值/注释 |
   |----------|---------------|
   | 文件格式 | 纯文本 (.txt)<br/> SSML 文本（.txt）<br/> 不支持 Zip 文件 |
   | 编码格式 | UTF-8 |
   | 文件名 | 每个文件必须具有唯一的名称。 不支持重复项。 |
   | 文本长度 | 文本文件的长度不能超过10000个字符。 |
   | SSML 限制 | 每个 SSML 文件只能包含一条 SSML。 |

### <a name="plain-text-example"></a>纯文本示例

```txt
Welcome to use Audio Content Creation to customize audio output for your products.
```

### <a name="ssml-text-example"></a>SSML 文本示例

```xml
<speak xmlns="http://www.w3.org/2001/10/synthesis" xmlns:mstts="http://www.w3.org/2001/mstts" version="1.0" xml:lang="en-US">
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)">
    Welcome to use Audio Content Creation <break time="10ms" />to customize audio output for your products.
    </voice>
</speak>
```

## <a name="export-tuned-audio"></a>导出优化音频

查看音频输出并且对调整和调整满意后，可以导出音频。

1. 从[音频内容创建](https://aka.ms/audiocontentcreation)工具中，单击 "**导出**" 以创建音频创建任务。
2. 选择优化音频的输出格式。 下面提供了支持的格式和采样速率的列表。
3. 可以在 "**导出任务**" 选项卡上查看任务的状态。如果任务失败，请参阅详细信息页获取完整的报表。
4. 完成该任务后，可以在 "**音频库**" 选项卡上下载音频。
5. 单击 **“下载”** 。 现在，你已准备好在你的应用或产品中使用自定义的优化音频。

### <a name="supported-audio-formats"></a>支持的音频格式

| 格式 | 16 kHz 采样率 | 24 kHz 采样率 |
|--------|--------------------|--------------------|
| wav | riff-16khz-16bit | riff-24khz-16bit |
| mp3 | 16khz-128kbitrate-mp3 | 24khz-160kbitrate-mp3 |

## <a name="see-also"></a>请参阅

* [长音频 API](https://aka.ms/long-audio-api)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Speech Studio](https://speech.microsoft.com)
