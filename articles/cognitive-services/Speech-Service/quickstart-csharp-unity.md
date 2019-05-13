---
title: 快速入门：识别语音，Unity - 语音服务
titleSuffix: Azure Cognitive Services
description: 参考本指南使用 Unity 和适用于 Unity 的语音 SDK (Beta) 创建语音转文本应用程序。 完成后，可以使用计算机的麦克风实时将语音转录为文本。
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 2/20/2019
ms.author: wolfma
ms.openlocfilehash: 3cedfaf1ae16c17026314fc24dbdc7bb11494caf
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65020957"
---
# <a name="quickstart-recognize-speech-with-the-speech-sdk-for-unity-beta"></a>快速入门：使用适用于 Unity 的语音 SDK (Beta) 识别语音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

参考本指南使用 [Unity](https://unity3d.com/) 和适用于 Unity 的语音 SDK (Beta) 创建语音转文本应用程序。
完成后，可以使用计算机的麦克风实时将语音转录为文本。
如果你不熟悉 Unity，我们建议在开始开发应用程序之前，先学习 [Unity 用户手册](https://docs.unity3d.com/Manual/UnityManual.html)。

> [!NOTE]
> 适用于 Unity 的语音 SDK 目前为 Beta 版。
> 它支持 Windows 桌面版（x86 和 x64）或通用 Windows 平台（x86、x64、ARM/ARM64）以及 Android（x86、ARM32/64）。

## <a name="prerequisites"></a>先决条件

若要完成本项目，需要：

* [Unity 2018.3 或更高版本](https://store.unity.com/)，以及[支持 UWP ARM64 的 Unity 2019.1](https://blogs.unity3d.com/2019/04/16/introducing-unity-2019-1/#universal)
* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)
     * 为了支持 ARM64，请安装[适用于 ARM64 的可选版本，以及适用于 ARM64 的 Windows 10 SDK](https://blogs.windows.com/buildingapps/2018/11/15/official-support-for-windows-10-on-arm-development/) 
* 语音服务的订阅密钥。 [免费获得一个](get-started.md)。
* 能够访问计算机的麦克风。

## <a name="create-a-unity-project"></a>创建 Unity 项目

* 启动 Unity，并在“项目”选项卡下选择“新建”。
* 指定 **csharp-unity** 作为“项目名称”，指定“3D”作为“模板”，然后选择一个位置。
  选择“创建项目”。
* 片刻之后，Unity 编辑器窗口应会弹出。

## <a name="install-the-speech-sdk"></a>安装语音 SDK

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

* 适用于 Unity 的语音 SDK (Beta) 打包为 Unity 资产包 (.unitypackage)。
  从[此处](https://aka.ms/csspeech/unitypackage)下载它。
* 选择“资产” > “导入包” > “自定义包”导入 Speech SDK。
  有关详细信息，请查看 [Unity 文档](https://docs.unity3d.com/Manual/AssetPackages.html)。
* 在文件选取器中，选择前面下载的语音 SDK .unitypackage 文件。
* 确保选择所有文件，然后单击“导入”：

  ![导入语音 SDK Unity 资产包时的 Unity 编辑器屏幕截图](media/sdk/qs-csharp-unity-01-import.png)

## <a name="add-ui"></a>添加 UI

我们将在场景中添加少量的 UI，其中包括一个用于触发语音识别的按钮，以及一个用于显示结果的文本字段。

* 在[层次结构窗口](https://docs.unity3d.com/Manual/Hierarchy.html)（默认位于左侧）中，显示了 Unity 创建的、包含新项目的示例场景。
* 单击“层次结构窗口”顶部的“创建”按钮，然后选择“UI” > “按钮”。
* 这会创建三个显示在“层次结构窗口”中的游戏对象：一个嵌套在“画布”对象中的“按钮”对象，以及一个“事件系统”对象。
* [浏览场景视图](https://docs.unity3d.com/Manual/SceneViewNavigation.html)，以全面查看[场景视图](https://docs.unity3d.com/Manual/UsingTheSceneView.html)中的画布和按钮。
* 在“层次结构窗口”中单击“按钮”对象，以便在[检查器窗口](https://docs.unity3d.com/Manual/UsingTheInspector.html)（默认位于右侧）中显示其设置。
* 将“位置 X”和“位置 Y”属性设置为 **0**，使按钮在画布上居中。
* 再次单击“层次结构窗口”顶部的“创建”按钮，然后选择“UI” > “文本”以创建文本字段。
* 在“层次结构窗口”中单击“文本”对象，以便在[检查器窗口](https://docs.unity3d.com/Manual/UsingTheInspector.html)（默认位于右侧）中显示其设置。
* 将“位置 X”和“位置 Y”属性分别设置为 **0** 和 **120**，将“宽度”和“高度”属性分别设置为 **240** 和 **120**，以确保文本字段和按钮不会重叠。

完成后，UI 应如以下屏幕截图所示：

[![Unity 编辑器中快速入门用户界面的屏幕截图](media/sdk/qs-csharp-unity-02-ui-inline.png)](media/sdk/qs-csharp-unity-02-ui-expanded.png#lightbox)

## <a name="add-the-sample-code"></a>添加示例代码

1. 在[项目窗口](https://docs.unity3d.com/Manual/ProjectView.html)（默认位于左下角）中，单击“创建”按钮并选择“C# 脚本”。 将脚本命名为 `HelloWorld`。

1. 双击脚本对其进行编辑。

   > [!NOTE]
   > 可以在“编辑” > “首选项”下配置要启动的代码编辑器，详情请参阅 [Unity 用户手册](https://docs.unity3d.com/Manual/Preferences.html)。

1. 将所有代码替换为以下内容：

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-unity/Assets/Scripts/HelloWorld.cs#code)]

1. 找到字符串 `YourSubscriptionKey` 并将其替换为你的语音服务订阅密钥。

1. 找到字符串 `YourServiceRegion` 并将其替换为与订阅关联的[区域](regions.md)。 例如，如果使用的是免费试用版，区域是 `westus`。

1. 保存对脚本所做的更改。

1. 返回到 Unity 编辑器。需要将脚本作为组件添加到游戏对象之一。

   * 单击“层次结构窗口”中的“画布”对象。 这会在[检查器窗口](https://docs.unity3d.com/Manual/UsingTheInspector.html)（默认位于右侧）中打开设置。
   * 单击“检查器窗口”中的“添加组件”按钮，然后搜索并添加前面创建的 HelloWorld 脚本。
   * 请注意，Hello World 组件包含两个未初始化的属性：“输出文本”和“开始识别按钮”，它们与 `HelloWorld` 类的公共属性相匹配。
     若要连接它们，请单击对象选取器（属性右侧的小圆圈图标），并选择前面创建的文本和按钮对象。

     > [!NOTE]
     > 按钮中还包括嵌套的文本对象。 请确保不要意外选择该对象来提供文本输出（或者，可以使用“检查器窗口”中的“名称”字段来重命名某个文本对象，以避免混淆）。

## <a name="run-the-application-in-the-unity-editor"></a>在 Unity 编辑器中运行应用程序

* 在 Unity 编辑器工具栏（菜单栏下方）中按下“播放”按钮。

* 启动应用后，请单击创建的按钮，对着计算机的麦克风讲出英语短语或句子。 你的语音将传输到语音服务并转录为文本，该文本将显示在窗口中。

  [![Unity 游戏窗口中运行的快速入门应用程序的屏幕截图](media/sdk/qs-csharp-unity-03-output-inline.png)](media/sdk/qs-csharp-unity-03-output-expanded.png#lightbox)

* 在[控制台窗口](https://docs.unity3d.com/Manual/Console.html)中检查调试消息。

* 完成语音识别后，在 Unity 编辑器工具栏中单击“播放”按钮以停止应用。

## <a name="additional-options-to-run-this-application"></a>用于运行此应用程序的其他选项

还可将此应用程序作为 Windows 单机应用或 UWP 应用程序部署到 Android。
请参阅 quickstart/csharp-unity 文件夹中的[示例存储库](https://aka.ms/csspeech/samples)，其中描述了这些附加目标的配置。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C# 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
