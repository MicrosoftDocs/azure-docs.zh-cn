---
title: 快速入门：合成语音，C++ (Windows) - 语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何在 Windows 桌面上使用语音 SDK 通过 C++ 合成语音
services: cognitive-services
author: yinhew
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 4/03/2019
ms.author: yinhew
ms.openlocfilehash: 7e9aebd3a49182f84e05473da9ed166499f34a28
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465708"
---
# <a name="quickstart-synthesize-speech-in-c-on-windows-by-using-the-speech-sdk"></a>快速入门：使用语音 SDK 在 Windows 上的 C++ 中合成语音

针对[语音识别](quickstart-cpp-windows.md)和[语音翻译](quickstart-translate-speech-cpp-windows.md)也提供了快速入门。

在本文中，请创建适用于 Windows 的 C++ 控制台应用程序。 使用认知服务[语音 SDK](speech-sdk.md) 实时从文本合成语音，并在电脑的扬声器上播放语音。 该应用程序是使用[语音 SDK NuGet 包](https://aka.ms/csspeech/nuget)和 Microsoft Visual Studio 2017（任何版本）构建的。

可以使用[语音 SDK 1.5.0](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech/1.5.0) 实现本文介绍的功能。

有关可用于语音合成的语言/语音的完整列表，请参阅[语言支持](language-support.md#text-to-speech)。

## <a name="prerequisites"></a>先决条件

需要有语音服务订阅密钥才能完成此快速入门。 你可以免费获得一个。 有关详细信息，请参阅[免费试用语音服务](get-started.md)。

## <a name="create-a-visual-studio-project"></a>创建 Visual Studio 项目

[!INCLUDE [Quickstart C++ project](../../../includes/cognitive-services-speech-service-quickstart-cpp-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 打开源文件 *helloworld.cpp*。 将初始 include 语句（`#include "stdafx.h"` 或 `#include "pch.h"`）下面的所有代码替换为以下内容：

   [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/text-to-speech/cpp-windows/helloworld/helloworld.cpp#code)]

1. 在同一文件中，将字符串 `YourSubscriptionKey` 替换为你的订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

1. 保存对项目的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

1. 构建应用程序。 从菜单栏中，选择“构建” > “构建解决方案”。 编译代码时应不会出错。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“生成解决方案”选项](media/sdk/qs-cpp-windows-06-build.png)

1. 启动应用程序。 在菜单栏中，选择“调试” > “开始调试”，或按 F5。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“启动调试”选项](media/sdk/qs-cpp-windows-07-start-debugging.png)

1. 此时会显示控制台窗口，提示你键入一些文本。 键入几个单词或一个句子。 键入的文本将传输到语音服务，并合成为语音，在扬声器上播放。

   ![成功合成后的控制台输出的屏幕截图](media/sdk/qs-tts-cpp-windows-console-output.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何将语音保存到音频文件。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C++ 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [自定义语音字体](how-to-customize-voice-font.md)
- [录制语音示例](record-custom-voice-samples.md)
