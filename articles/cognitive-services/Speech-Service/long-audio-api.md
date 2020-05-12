---
title: 长音频 API （预览）-语音服务
titleSuffix: Azure Cognitive Services
description: 了解长音频 API 的设计方式是将长格式文本的异步合成转换为语音。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/30/2020
ms.author: trbye
ms.openlocfilehash: b7cca314ec59e46cf17751b1aec28b5c3ea029ed
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2020
ms.locfileid: "81401062"
---
# <a name="long-audio-api-preview"></a>长音频 API （预览）

长音频 API 设计用于将长格式文本的异步合成到语音（例如：音频书籍）。 此 API 不会实时返回合成音频，而是期望你会轮询响应并使用这些输出，因为服务中提供了这些响应。 与语音 SDK 使用的文本到语音 API 不同，长音频 API 可以创建超过10分钟的合成音频，使其成为发布者和音频内容平台的理想选择。

长音频 API 的其他好处：

* 服务返回的合成语音使用神经声音，这可确保高保真音频输出。
* 由于实时响应不受支持，因此无需部署语音终结点。

> [!NOTE]
> 长音频 API 现在仅支持[自定义的神经声音](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice#custom-neural-voices)。

## <a name="workflow"></a>工作流

通常，使用长音频 API 时，你将提交一个文本文件或文件以进行合成，轮询状态，如果状态为 "成功"，则可以下载音频输出。

此图提供工作流的高级概述。

![长音频 API 工作流关系图](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>为合成准备内容

准备文本文件时，请确保：

* 为纯文本（.txt）或 SSML 文本（.txt）
* 编码为[utf-8，具有字节顺序标记（BOM）](https://www.w3.org/International/questions/qa-utf8-bom.en#bom)
* 是单个文件，而不是 zip
* 对于 SSML 文本，对于 SSML 文本包含400个以上的字符[，对于 SSML 文本包含400个](https://docs.microsoft.com/azure/cognitive-services/speech-service/text-to-speech#pricing-note)，小于10000个段落
  * 对于纯文本，每个段落都按 "**输入/返回**-查看[纯文本输入" 示例](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt)分隔
  * 对于 SSML 文本，每个 SSML 部分都被视为一个段落。 SSML 部分应由不同的段落分隔-查看[ssml 文本输入示例](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt)
> [!NOTE]
> 对于中文（大陆）、中文（香港特别行政区）、中文（台湾）、日语和韩语，一个字将被计为两个字符。 

## <a name="submit-synthesis-requests"></a>提交合成请求

准备好输入内容后，请遵循[长格式的音频合成快速](https://aka.ms/long-audio-python)启动来提交请求。 如果有多个输入文件，则需要提交多个请求。 存在一些需要注意的限制： 
* 每个 Azure 订阅帐户都允许客户端每秒向服务器提交多达5次请求。 如果超出限制，客户端将收到429错误代码（请求过多）。 请减少每秒请求数
* 允许服务器运行每个 Azure 订阅帐户，并为其列队多达120请求。 如果超过此限制，则服务器将返回429错误代码（请求过多）。 请等待，并避免提交新请求，直到完成一些请求
* 服务器将为每个 Azure 订阅帐户最多保留20000请求。 如果超出限制，请在提交新请求之前删除一些请求

## <a name="audio-output-formats"></a>音频输出格式

支持灵活的音频输出格式。 可以通过设置 "concatenateResult" 参数，按段落生成音频输出或将音频连接到一个输出。 长音频 API 支持以下音频输出格式：

> [!NOTE]
> 默认音频格式为 riff-16khz-16bit。

* riff-8khz-16bit
* riff-16khz-16bit
* riff-24khz-16bit
* riff-48khz-16bit
* 16khz-32kbitrate-mp3
* 16khz-64kbitrate-mp3
* 16khz-128kbitrate-mp3
* 24khz-48kbitrate-mp3
* 24khz-96kbitrate-mp3
* 24khz-160kbitrate-mp3

## <a name="quickstarts"></a>快速入门

我们提供旨在帮助你成功运行长音频 API 的快速入门。 此表包含按语言组织的长音频 API 快速入门列表。

* [快速入门： Python](https://aka.ms/long-audio-python)

## <a name="sample-code"></a>示例代码
GitHub 上提供了长音频 API 的示例代码。

* [示例代码： Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [示例代码： C#](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [示例代码： Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)
