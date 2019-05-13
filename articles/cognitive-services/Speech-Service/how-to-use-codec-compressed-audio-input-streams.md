---
title: Stream 编解码器压缩的音频和语音 SDK-语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何将流式传输到 Azure 使用语音 SDK 的语音服务的压缩的音频。 适用于C++， C#，并适用于 Linux 的 Java。
services: cognitive-services
author: amitkumarshukla
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: amishu
ms.openlocfilehash: 41a55eca321cbe1bfa23a889b8e3ce7c701ce769
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65468045"
---
# <a name="using-codec-compressed-audio-input-with-the-speech-sdk"></a>使用编解码器压缩音频输入使用语音 SDK

Speech SDK**压缩音频输入 Stream** API 提供了一种方法来流式传输到语音服务的压缩的音频使用 PullStream 或 PushStream。

> [!IMPORTANT]
> 仅支持流式处理压缩的音频C++， C#，并在 Linux （Ubuntu 16.04、 Ubuntu 18.04、 Debian 9） 上的 Java。

Wav/PCM 请参阅主线语音文档。  外部 wav/PCM，支持以下编解码器压缩输入的格式：

- MP3
- 大作/OGG

## <a name="prerequisites-to-using-codec-compressed-audio-input"></a>必备组件以使用编解码器压缩音频输入

安装适用于 Linux 的语音 SDK 使用压缩的音频输入这些其他依赖项：

```sh
sudo apt install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly
```

## <a name="example-code-using-codec-compressed-audio-input"></a>示例代码使用编解码器压缩音频输入

若要流式传输到语音服务以压缩音频格式，创建`PullAudioInputStream`或`PushAudioInputStream`。 然后，创建`AudioConfig`流类的实例，从指定流的压缩格式。

我们假设您有一个名为的输入的流类`myPushStream`并且正在使用大作/OGG。 你的代码可能如下所示：

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var speechConfig = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");

// Create an audio config specifying the compressed audio format and the instance of your input stream class.
var audioFormat = AudioStreamFormat.GetCompressedFormat(AudioStreamContainerFormat.OGG_OPUS);
var audioConfig = AudioConfig.FromStreamInput(myPushStream, audioFormat);

var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

var result = await recognizer.RecognizeOnceAsync();

var text = result.GetText();
```

## <a name="next-steps"></a>后续步骤

- [获取语音试用订阅](https://azure.microsoft.com/try/cognitive-services/)
- [了解如何在 C# 中识别语音](quickstart-csharp-dotnet-windows.md)
