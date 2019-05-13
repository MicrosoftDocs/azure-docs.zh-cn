---
title: 快速入门：识别语音，C# (UWP) - 语音服务
titleSuffix: Azure Cognitive Services
description: 在本文中，请使用认知服务语音 SDK 创建 C# 通用 Windows 平台 (UWP) 应用程序。 可以通过设备的麦克风实时将语音转录为文本。 该应用程序是使用语音 SDK NuGet 包和 Microsoft Visual Studio 2017 构建的。
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 12/06/2018
ms.author: wolfma
ms.custom: seodec18
ms.openlocfilehash: 674bb3f22f7b8c7c7ea3b52c2f27d862a9961fcb
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466250"
---
# <a name="quickstart-recognize-speech-in-a-uwp-app-by-using-the-speech-sdk"></a>快速入门：使用语音 SDK 在 UWP 应用中识别语音

针对[语音翻译](quickstart-translate-speech-uwp.md)和[语音优先虚拟助手](quickstart-virtual-assistant-csharp-uwp.md)也提供了快速入门。

如果需要，请选择不同的编程语言和/或环境：<br/>
[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，请使用认知服务[语音 SDK](speech-sdk.md) 开发 C# 通用 Windows 平台（UWP；Windows 版本 1709 或更高版本）应用程序。 该程序将通过设备的麦克风实时将语音转录为文本。 该应用程序使用[语音 SDK NuGet 包](https://aka.ms/csspeech/nuget)和 Microsoft Visual Studio 2017（任何版本）生成。

> [!NOTE]
> 通用 Windows 平台允许开发在支持 Windows 10 的任何设备上运行的应用，包括电脑、Xbox、Surface Hub 和其他设备。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)
* 语音服务的 Azure 订阅密钥。 [免费获得一个](get-started.md)。

## <a name="create-a-visual-studio-project"></a>创建 Visual Studio 项目

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-uwp-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 应用程序的用户界面使用 XAML 进行定义。 在解决方案资源管理器中打开 `MainPage.xaml`。 在设计器的 XAML 视图中，将以下 XAML 代码片段插入到 Grid 标记中（位于 `<Grid>` 和 `</Grid>` 之间）。

   [!code-xml[UI elements](~/samples-cognitive-services-speech-sdk/quickstart/csharp-uwp/helloworld/MainPage.xaml#StackPanel)]

1. 打开代码隐藏源文件 `MainPage.xaml.cs`（可以发现它归在 `MainPage.xaml` 下）。 将其中的所有代码替换为以下内容。

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-uwp/helloworld/MainPage.xaml.cs#code)]

1. 在此文件的 `SpeechRecognitionFromMicrophone_ButtonClicked` 处理程序中，将字符串 `YourSubscriptionKey` 替换为订阅密钥。

1. 在 `SpeechRecognitionFromMicrophone_ButtonClicked` 处理程序中，将字符串 `YourServiceRegion` 替换为与订阅（例如，`westus` 免费试用版订阅）相关联的[区域](regions.md)。

1. 保存对项目的所有更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

1. 构建应用程序。 从菜单栏中，选择“构建” > “构建解决方案”。 现在，编译代码时应不会提示错误。

    ![Visual Studio 应用程序的屏幕截图，其中突出显示了“生成解决方案”选项](media/sdk/qs-csharp-uwp-08-build.png "成功生成")

1. 启动应用程序。 在菜单栏中，选择“调试” > “开始调试”，或按 F5。

    ![Visual Studio 应用程序的屏幕截图，其中突出显示了“启动调试”选项](media/sdk/qs-csharp-uwp-09-start-debugging.png "启动应用进入调试")

1. 弹出一个窗口。 选择“启用麦克风”，然后确认弹出的权限请求。

    ![权限请求的屏幕截图](media/sdk/qs-csharp-uwp-10-access-prompt.png "启动应用进入调试")

1. 选择“使用麦克风输入进行语音识别”，然后对着设备的麦克风说一个英文短语或句子。 你的语音将传输到语音服务并转录为文本，该文本将显示在窗口中。

    ![语音识别用户界面的屏幕截图](media/sdk/qs-csharp-uwp-11-ui-result.png)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C# 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [翻译语音](how-to-translate-speech-csharp.md)
- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
