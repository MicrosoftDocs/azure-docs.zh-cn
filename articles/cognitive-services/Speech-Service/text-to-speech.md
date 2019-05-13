---
title: 文本到语音转换与 Azure 的语音服务
titleSuffix: Azure Cognitive Services
description: Azure 语音服务从文本到语音转换为一个服务，使你的应用程序、 工具或设备，将文本转换为自然人合成语音。 从标准和神经声音，选择或创建你自己的自定义语音唯一的产品或品牌。 75 标准语音可在多个 45 语言和区域设置，以及 5 神经语音有 4 个语言和区域设置。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: erhopf
ms.custom: seodec18
ms.openlocfilehash: cbabdde2dd5df2ca40ee45fe0f7b0437f095673b
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65020383"
---
# <a name="what-is-text-to-speech"></a>文本到语音转换是什么？

Azure 语音服务从文本到语音转换为一个服务，使你的应用程序、 工具或设备，将文本转换为自然人合成语音。 从标准和神经声音，选择或创建你自己的自定义语音唯一的产品或品牌。 75 标准语音可在多个 45 语言和区域设置，以及 5 神经语音有 4 个语言和区域设置。 有关完整列表，请参阅[支持的语言](language-support.md#text-to-speech)。

文本到语音转换技术，使内容创建者可以以不同方式与他们的用户交互。 通过为用户提供用于呼叫与内容进行交互的选项，文本到语音转换可以提高可访问性。 无论用户有视觉障碍者，学习为残疾人士，还是需要导航信息，同时还能降低，文本到语音转换可以提高现有体验。 文本到语音转换也是有价值的语音机器人和虚拟助手外接程序。

### <a name="standard-voices"></a>标准语音

使用统计参数合成和/或串联合成技术创建标准的语音。 这些语音是高度可识别，听起来很自然。 您可以轻松实现在多个 45 语言中，与范围广泛的语音选项说出您的应用程序。 这些语音提供高发音的准确性，包括对缩写、 首字母缩略词扩展、 日期/时间的解释、 polyphones，和的详细信息的支持。 使用标准的语音用户与你的内容时传递语音交互，从而提高应用程序和服务可访问性。

### <a name="neural-voices"></a>神经语音

神经语音使用深层神经网络来克服中匹配模式的压力和声调和到计算机使用的语音合成语音的单位中口述语言的传统文本到语音转换系统的限制。 标准文本到语音转换将分解为单独的语言分析和由独立的模型管理的声学预测步骤诗体论。 这可以导致 muffled、 buzzy 语音合成。 我们神经功能执行诗体论预测和语音合成，这会导致更流畅和自然发音的语音。

可以使用神经语音使与聊天机器人和虚拟助手的交流更加自然和专注、可将数字文本（如电子书）转换为有声读物以及可增强车载导航系统。 使用类似于人类的自然诗体论和明确的单词，神经语音显著减少侦听疲劳与 AI 系统进行交互时。 

神经语音支持不同的样式，如非特定和令人高兴。 例如，Jessa (EN-US) 语音可以说出 cheerfully，热、 高兴会话进行了优化。 可以调整语音输出，如音，间距，并使用[语音合成标记语言](speech-synthesis-markup.md)。 有关可用的声音的完整列表，请参阅[支持的语言](language-support.md#text-to-speech)。

若要了解有关神经语音的优势的详细信息，请参阅[Microsoft 的新神经文本到语音转换服务可帮助与人一样说出计算机](https://azure.microsoft.com/blog/microsoft-s-new-neural-text-to-speech-service-helps-machines-speak-like-people/)。

### <a name="custom-voices"></a>自定义语音

语音自定义项，可以创建自己的品牌的可识别的、 独一无二的声音。 若要创建自定义语音字体，则请 studio 录制并上传与训练数据关联的脚本。 然后，此服务会根据你的录音创建唯一的语音模型。 可以使用此自定义语音字体合成语音。 有关详细信息，请参阅[自定义语音](how-to-customize-voice-font.md)。

## <a name="core-features"></a>核心功能

此表列出了文本到语音转换的核心功能：

| 使用案例 | SDK 中 IsInRole 中的声明 | REST |
|----------|-----|------|
| 将文本到语音转换。 | 是 | 是 |
| 上传为语音适配的数据集。 | 否 | 是\* |
| 创建和管理语音字体模型。 | 否 | 是\* |
| 创建和管理语音字体部署。 | 否 | 是\* |
| 创建和管理语音字体测试。 | 否 | 是\* |
| 管理订阅。 | 否 | 是\* |

\* *这些服务可使用 cris.ai 终结点。请参阅[Swagger 引用](https://westus.cris.ai/swagger/ui/index)。这些自定义语音培训和管理 Api 实施限制，限制为每 5 秒的 25 个请求而语音合成 API 本身实现限制，允许每秒为最高 200 个请求。当出现阻止情况，则会通知您通过消息标头。*

## <a name="get-started-with-text-to-speech"></a>开始使用文本到语音转换

我们提供了快速入门旨在让您在 10 分钟内运行代码。 此表包含一系列按语言的文本到语音转换快速入门。

### <a name="sdk-quickstarts"></a>SDK 快速入门

| 快速入门 (SDK) | 平台 | API 参考 |
|------------|----------|---------------|
| [C#.NET framework](quickstart-text-to-speech-dotnet-windows.md) | Windows | [Browse](https://aka.ms/csspeech/csharpref) |
| [C++](quickstart-text-to-speech-cpp-windows.md) | Windows | [Browse](https://aka.ms/csspeech/cppref) |
| [C++](quickstart-text-to-speech-cpp-linux.md) | Linux | [Browse](https://aka.ms/csspeech/cppref) |

### <a name="rest-quickstarts"></a>REST 快速入门

| 快速入门 (REST) | 平台 | API 参考 |
|------------|----------|---------------|
| [C#, .NET Core](quickstart-dotnet-text-to-speech.md) | Windows、 macOS、 Linux | [Browse](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis) |
| [Node.js](quickstart-nodejs-text-to-speech.md) | 窗口中，macOS、 Linux | [Browse](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis) |
| [Python](quickstart-python-text-to-speech.md) | 窗口中，macOS、 Linux | [Browse](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis) |

## <a name="sample-code"></a>代码示例

文本到语音转换的示例代码位于 GitHub 上提供。 这些示例涉及最常用编程语言中的文本到语音转换。

* [文本到语音转换示例 (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
* [文本转语音示例 (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="reference-docs"></a>参考文档

* [语音 SDK](speech-sdk-reference.md)
* [语音设备 SDK](speech-devices-sdk.md)
* [REST API：语音转文本](rest-speech-to-text.md)
* [REST API：文本转语音](rest-text-to-speech.md)
* [REST API：批量听录和自定义](https://westus.cris.ai/swagger/ui/index)

## <a name="next-steps"></a>后续步骤

* [获取免费语音服务订阅](get-started.md)
* [创建自定义语音字体](how-to-customize-voice-font.md)
