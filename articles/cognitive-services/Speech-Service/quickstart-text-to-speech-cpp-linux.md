---
title: 快速入门：合成语音，C++ (Linux) - 语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何在 Linux 上使用语音 SDK 通过 C++ 合成语音
services: cognitive-services
author: yinhew
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 05/02/2019
ms.author: yinhew
ms.openlocfilehash: b5db21d40f1e270fb6771ba47af28a9e90228ea4
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465588"
---
# <a name="quickstart-synthesize-speech-in-c-on-linux-by-using-the-speech-sdk"></a>快速入门：在 Linux 上使用语音 SDK 通过 C++ 合成语音

针对[语音识别](quickstart-cpp-linux.md)也提供了快速入门。

在本文中，你将创建一个适用于 Linux（Ubuntu 16.04、Ubuntu 18.04、Debian 9）的 C++ 控制台应用程序。 使用认知服务[语音 SDK](speech-sdk.md) 实时从文本合成语音，并在电脑的扬声器上播放语音。 该应用程序是通过[适用于 Linux 的语音 SDK](https://aka.ms/csspeech/linuxbinary) 和你的 Linux 发行版的 C++ 编译器（例如 `g++`）构建的。

## <a name="prerequisites"></a>先决条件

需要有语音服务订阅密钥才能完成此快速入门。 你可以免费获得一个。 有关详细信息，请参阅[免费试用语音服务](get-started.md)。

## <a name="install-speech-sdk"></a>安装语音 SDK

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

认知服务语音 SDK 的当前版本是 `1.5.0`。

适用于 Linux 的语音 SDK 可用于构建 64 位和 32 位应用程序。 可以从 https://aka.ms/csspeech/linuxbinary 以 tar 文件格式下载必需的库和头文件。

下载并安装 SDK，如下所示：

1. 确保安装了 SDK 的依赖项。

   * 在 Ubuntu 上：

     ```sh
     sudo apt-get update
     sudo apt-get install build-essential libssl1.0.0 libasound2 wget
     ```

   * 在 Debian 9 上：

     ```sh
     sudo apt-get update
     sudo apt-get install build-essential libssl1.0.2 libasound2 wget
     ```

1. 选择应将语音 SDK 文件提取到的目录，然后将 `SPEECHSDK_ROOT` 环境变量设置为指向该目录。 使用此变量，在将来的命令中可以轻松引用目录。 例如，如果要使用主目录中的 `speechsdk` 目录，请使用如下所示的命令：

   ```sh
   export SPEECHSDK_ROOT="$HOME/speechsdk"
   ```

1. 如果该目录尚不存在，请创建该目录。

   ```sh
   mkdir -p "$SPEECHSDK_ROOT"
   ```

1. 下载并提取包含语音 SDK 二进制文件的 `.tar.gz` 存档：

   ```sh
   wget -O SpeechSDK-Linux.tar.gz https://aka.ms/csspeech/linuxbinary
   tar --strip 1 -xzf SpeechSDK-Linux.tar.gz -C "$SPEECHSDK_ROOT"
   ```

1. 验证所提取的程序包的顶级目录的内容：

   ```sh
   ls -l "$SPEECHSDK_ROOT"
   ```

   目录列表应当包含第三方通告和许可证文件，以及一个包含头文件 (`.h`) 的 `include` 目录和一个包含库的 `lib` 目录。

   [!INCLUDE [Linux Binary Archive Content](../../../includes/cognitive-services-speech-service-linuxbinary-content.md)]

## <a name="add-sample-code"></a>添加示例代码

1. 创建一个名为 `helloworld.cpp` 的 C++ 源文件，并将以下代码粘贴到其中。

   [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/text-to-speech/cpp-linux/helloworld.cpp#code)]

1. 在此新文件中，将字符串 `YourSubscriptionKey` 替换为你的语音服务订阅密钥。

1. 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)（例如，免费试用版订阅的 `westus`）。

## <a name="build-the-app"></a>生成应用

> [!NOTE]
> 请确保将以下命令输入在单个命令行上。 执行该操作的最简单方法是使用每个命令旁边的“复制按钮”来复制命令，然后将其粘贴到 shell 提示符下。

* 在 **x64**（64 位）系统上，运行以下命令来生成应用程序。

  ```sh
  g++ helloworld.cpp -o helloworld -I "$SPEECHSDK_ROOT/include/cxx_api" -I "$SPEECHSDK_ROOT/include/c_api" --std=c++14 -lpthread -lMicrosoft.CognitiveServices.Speech.core -L "$SPEECHSDK_ROOT/lib/x64" -l:libasound.so.2
  ```

* 在 **x86**（32 位）系统上，运行以下命令来生成应用程序。

  ```sh
  g++ helloworld.cpp -o helloworld -I "$SPEECHSDK_ROOT/include/cxx_api" -I "$SPEECHSDK_ROOT/include/c_api" --std=c++14 -lpthread -lMicrosoft.CognitiveServices.Speech.core -L "$SPEECHSDK_ROOT/lib/x86" -l:libasound.so.2
  ```

## <a name="run-the-app"></a>运行应用

1. 将加载程序的库路径配置为指向语音 SDK 库。

   * 在 **x64**（64 位）系统上，输入以下命令。

     ```sh
     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$SPEECHSDK_ROOT/lib/x64"
     ```

   * 在 **x86**（32 位）系统上，输入以下命令。

     ```sh
     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$SPEECHSDK_ROOT/lib/x86"
     ```

1. 运行应用程序。

   ```sh
   ./helloworld
   ```

1. 在控制台窗口中，会出现一个提示，提示你键入一些文本。 键入几个单词或一个句子。 你键入的文本将传输到语音服务，并合成为语音，在你的扬声器上播放。

   ```text
   Type some text that you want to speak...
   > hello
   Speech synthesized to speaker for text [hello]
   Press enter to exit...
   ```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 C++ 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [自定义语音字体](how-to-customize-voice-font.md)
- [录制语音示例](record-custom-voice-samples.md)
