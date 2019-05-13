---
title: 快速入门：翻译语音，C++ (Windows) - 语音服务
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，你将创建一个简单的 C++ 应用程序来捕获用户语音，将其翻译为另一种语言，并将文本输出到命令行。 本指南适用于 Windows 用户。
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 03/13/2019
ms.author: erhopf
ms.openlocfilehash: bf07e6a2daac7889e11df2cb5d689daef44266f9
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465657"
---
# <a name="quickstart-translate-speech-with-the-speech-sdk-for-c"></a>快速入门：使用适用于 C++ 的语音 SDK 翻译语音

针对[语音识别](quickstart-cpp-windows.md)和[文本转语音](quickstart-text-to-speech-cpp-windows.md)也提供了快速入门。

在本快速入门中，你将创建一个简单的 C++ 应用程序，该应用程序从计算机的麦克风中捕获用户语音，翻译语音，并将翻译后的文本实时转录到命令行。 此应用程序设计为在 64 位 Windows 上运行，并使用[语音 SDK NuGet 包](https://aka.ms/csspeech/nuget)和 Microsoft Visual Studio 2017 构建。

有关可用于语音翻译的语言的完整列表，请参阅[语言支持](language-support.md)。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)
* 语音服务的 Azure 订阅密钥。 [免费获得一个](get-started.md)。

## <a name="create-a-visual-studio-project"></a>创建 Visual Studio 项目

[!INCLUDE [Quickstart C++ project](../../../includes/cognitive-services-speech-service-quickstart-cpp-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 打开源文件 *helloworld.cpp*。 将初始 include 语句（`#include "stdafx.h"` 或 `#include "pch.h"`）下面的所有代码替换为以下内容：

    [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/speech-translation/cpp-windows/helloworld/helloworld.cpp#code)]

1. 在同一文件中，将字符串 `YourSubscriptionKey` 替换为你的订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

1. 保存对项目的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

1. 构建应用程序。 从菜单栏中，选择“构建” > “构建解决方案”。 编译代码时应不会出错。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“生成解决方案”选项](media/sdk/qs-cpp-windows-06-build.png)

1. 启动应用程序。 在菜单栏中，选择“调试” > “开始调试”，或按 F5。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“启动调试”选项](media/sdk/qs-cpp-windows-07-start-debugging.png)

1. 此时将显示控制台窗口，提示你说出任意内容。 说一个英语短语或句子。 你的语音将传输到语音服务、进行翻译并转录为文本，该文本将显示在同一窗口中。

   ![成功翻译后的控制台输出的屏幕截图](media/sdk/qs-translate-cpp-windows-output.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何从音频文件中读取语音，以及将翻译后的文本作为合成语音输出。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C++ 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
