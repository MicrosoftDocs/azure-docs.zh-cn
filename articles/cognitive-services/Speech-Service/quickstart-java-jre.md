---
title: 快速入门：识别语音，Java (Windows, Linux) - 语音服务
titleSuffix: Azure Cognitive Services
description: 本快速入门介绍如何创建一个简单的 Java 应用程序，用于从计算机的麦克风中捕获和转录用户语音。
services: cognitive-services
author: fmegen
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 05/02/2019
ms.author: fmegen
ms.openlocfilehash: 05475dbfb7dc1ee3f5de1a9b858eaf212b674eac
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466391"
---
# <a name="quickstart-recognize-speech-with-the-speech-sdk-for-java"></a>快速入门：使用适用于 Java 的语音 SDK 识别语音

针对[语音转语音翻译](quickstart-translate-speech-java-jre.md)和[语音优先虚拟助手](quickstart-virtual-assistant-java-jre.md)也提供了快速入门。

如果需要，请选择不同的编程语言和/或环境：<br/>
[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，你将使用[语音 SDK](speech-sdk.md) 创建一个 Java 控制台应用程序。 可以通过电脑的麦克风实时将语音转录为文本。 此应用程序是使用语音 SDK Maven 程序包和 Eclipse Java IDE (v4.8) 在 64 位 Windows、64 位 Linux（Ubuntu 16.04、Ubuntu 18.04、Debian 9）或 macOS 10.13 或更高版本上生成的。 它在 64 位 Java 8 运行时环境 (JRE) 中运行。

> [!NOTE]
> 对于语音设备 SDK 和 Roobo 设备，请参阅[语音设备 SDK](speech-devices-sdk.md)。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* 操作系统：64 位 Windows、64 位 Linux（Ubuntu 16.04、Ubuntu 18.04、Debian 9）或 macOS 10.13 或更高版本
* [Eclipse Java IDE](https://www.eclipse.org/downloads/)
* [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) 或 [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* 语音服务的 Azure 订阅密钥。 [免费获得一个](get-started.md)。

如果运行 Linux，请确保在启动 Eclipse 之前安装这些依赖项。

* 在 Ubuntu 上：

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.0 libasound2
  ```

* 在 Debian 9 上：

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.2 libasound2
  ```

如果运行 Windows（64 位），请确保已安装适用于平台的 Microsoft Visual C++ Redistributable。
* [下载 Microsoft Visual C++ Redistributable for Visual Studio 2017](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)

## <a name="create-and-configure-project"></a>创建并配置项目

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-java-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 若要向 Java 项目添加新的空类，请选择“文件” > “新建” > “类”。

1. 在“新建 Java 类”窗口中，在“包”字段内输入 **speechsdk.quickstart**，在“名称”字段内输入 **Main**。

   ![“新建 Java 类”窗口的屏幕截图](media/sdk/qs-java-jre-06-create-main-java.png)

1. 将 `Main.java` 中的所有代码替换为以下代码片段：

   [!code-java[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/src/speechsdk/quickstart/Main.java#code)]

1. 将字符串 `YourSubscriptionKey` 替换为你的订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

1. 保存对项目的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

按 F11，或选择“运行” > “调试”。
接下来的 15 秒，通过麦克风提供的语音输入将被识别并记录到控制台窗口中。

![成功识别后的控制台输出的屏幕截图](media/sdk/qs-java-jre-07-console-output.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何从音频文件中读取语音。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Java 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [快速入门：翻译语音，Java（Windows、Linux）](quickstart-translate-speech-java-jre.md)
- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
