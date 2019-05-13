---
title: 快速入门：翻译语音，Java (Windows, Linux) - 语音服务
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，你将创建一个简单的 Java 应用程序来捕获用户语音，将其翻译为另一种语言，并将文本输出到命令行。 本指南适用于 Windows 和 Linux 用户。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 69cb48df4e765250247605fb5c4e1f421554fd20
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467021"
---
# <a name="quickstart-translate-speech-with-the-speech-sdk-for-java"></a>快速入门：使用适用于 Java 的语音 SDK 转换语音

针对[语音转文本](quickstart-java-jre.md)和[语音优先虚拟助手](quickstart-virtual-assistant-java-jre.md)也提供了快速入门。

在本快速入门中，你将创建一个简单的 Java 应用程序，该应用程序从计算机的麦克风中捕获用户语音，翻译语音，并将翻译后的文本实时转录到命令行。 此应用程序需在 64 位 Windows 或 64 位 Linux（Ubuntu 16.04、Ubuntu 18.04、Debian 9）或者 macOS 10.13 或更高版本上运行。 它是使用语音 SDK Maven 包和 Eclipse Java IDE 生成的。

有关可用于语音翻译的语言的完整列表，请参阅[语言支持](language-support.md)。

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

> [!NOTE]
> 对于语音设备 SDK 和 Roobo 设备，请参阅[语音设备 SDK](speech-devices-sdk.md)。

## <a name="create-and-configure-project"></a>创建并配置项目

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-java-create-proj.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 若要向 Java 项目添加新的空类，请选择“文件” > “新建” > “类”。

1. 在“新建 Java 类”窗口中，在“包”字段内输入 **speechsdk.quickstart**，在“名称”字段内输入 **Main**。

   ![“新建 Java 类”窗口的屏幕截图](media/sdk/qs-java-jre-06-create-main-java.png)

1. 将 `Main.java` 中的所有代码替换为以下代码片段：

   [!code-java[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/speech-translation/java-jre/src/speechsdk/quickstart/Main.java#code)]

1. 将字符串 `YourSubscriptionKey` 替换为你的订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

1. 保存对项目的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

按 F11，或选择“运行” > “调试”。

通过麦克风提供的语音输入将转译为德语，并记录到控制台窗口中。 按“Enter”停止捕获语音。

![成功识别后的控制台输出的屏幕截图](media/sdk/qs-translate-java-jre-output.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何从音频文件中读取语音，以及将翻译后的文本作为合成语音输出。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Java 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [快速入门：识别语音，Java（Windows、Linux）](quickstart-java-jre.md)
- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
