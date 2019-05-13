---
title: 快速入门：识别语音，C++ (Windows) - 语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何在 Windows 桌面上使用语音 SDK 通过 C++ 识别语音
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 12/13/2018
ms.author: wolfma
ms.openlocfilehash: 672a691cdad5c83583ea840907fa7dd193c7ae3a
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466967"
---
# <a name="quickstart-recognize-speech-in-c-on-windows-by-using-the-speech-sdk"></a>快速入门：使用语音 SDK 在 Windows 上的 C++ 中识别语音

针对[文本转语音](quickstart-text-to-speech-cpp-windows.md)和[语音翻译](quickstart-translate-speech-cpp-windows.md)也提供了快速入门。

如果需要，请选择不同的编程语言和/或环境：<br/>
[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，请创建适用于 Windows 的 C++ 控制台应用程序。 请使用认知服务[语音 SDK](speech-sdk.md)，通过电脑的麦克风实时将语音转录为文本。 该应用程序是使用[语音 SDK NuGet 包](https://aka.ms/csspeech/nuget)和 Microsoft Visual Studio 2017（任何版本）构建的。

## <a name="prerequisites"></a>先决条件

需要有语音服务订阅密钥才能完成此快速入门。 你可以免费获得一个。 有关详细信息，请参阅[免费试用语音服务](get-started.md)。

## <a name="create-a-visual-studio-project"></a>创建 Visual Studio 项目

[!INCLUDE [Quickstart C++ project](../../../includes/cognitive-services-speech-service-quickstart-cpp-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 打开源文件 *helloworld.cpp*。 将初始 include 语句（`#include "stdafx.h"` 或 `#include "pch.h"`）下面的所有代码替换为以下内容：

   [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/cpp-windows/helloworld/helloworld.cpp#code)]

1. 在同一文件中，将字符串 `YourSubscriptionKey` 替换为你的订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

1. 保存对项目的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

1. 构建应用程序。 从菜单栏中，选择“构建” > “构建解决方案”。 编译代码时应不会出错。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“生成解决方案”选项](media/sdk/qs-cpp-windows-06-build.png)

1. 启动应用程序。 在菜单栏中，选择“调试” > “开始调试”，或按 F5。

   ![Visual Studio 应用程序的屏幕截图，其中突出显示了“启动调试”选项](media/sdk/qs-cpp-windows-07-start-debugging.png)

1. 此时将显示控制台窗口，提示你说出任意内容。 说一个英语短语或句子。 你的语音将传输到语音服务并转录为文本，该文本将显示在同一窗口中。

   ![成功识别后的控制台输出的屏幕截图](media/sdk/qs-cpp-windows-08-console-output-release.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何从音频文件中读取语音。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C++ 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
