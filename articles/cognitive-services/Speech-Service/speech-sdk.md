---
title: 关于语音 SDK - 语音服务
titleSuffix: Azure Cognitive Services
description: 使用语音软件开发工具包 (SDK)，应用程序可以本机访问语音服务的功能，这使得软件开发工作更为容易。 本文提供了有关适用于 Windows、Linux 和 Android 的 SDK 的其他详细信息。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: wolfma
ms.openlocfilehash: a16874dc9536518874fb6a9d1f219b3b31e16364
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65020401"
---
# <a name="about-the-speech-sdk"></a>关于语音 SDK

语音软件开发工具包 (SDK) 为您的应用程序的访问权限提供功能的语音服务，使其更轻松地开发支持语音的软件。 目前，Sdk 提供访问权限**语音到文本**，**文本到语音转换**，**语音翻译**，以及**意图识别**。 可以在文档上找到有关功能和支持的平台的常规概述[入口页](https://aka.ms/csspeech)。

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

## <a name="get-the-sdk"></a>获取 SDK

### <a name="windows"></a>Windows

对于 Windows，我们支持以下语言：

* C#（UWP 和 .NET）、C++：可以引用和使用语音 SDK NuGet 包的最新版本。 此包包括 32 位和 64 位客户端库，以及托管 (.NET) 库。 可以使用 NuGet 在 Visual Studio 中安装该 SDK。 搜索 Microsoft.CognitiveServices.Speech NuGet。

* Java:可以引用和使用语音 SDK Maven 包的最新版本，该包仅支持 Windows x64。 在 Maven 项目中，将 `https://csspeechstorage.blob.core.windows.net/maven/` 添加为附加存储库，并将 `com.microsoft.cognitiveservices.speech:client-sdk:1.5.0` 引用为一个依赖项。

### <a name="linux"></a>Linux

> [!NOTE]
> 目前，在 PC 上支持 Ubuntu 16.04、 Ubuntu 18.04 和 Debian 9 (x86 或 x64 的C++开发和 x64 的.NET Core、 Java 和 Python)。

请确保已通过运行以下 shell 命令来安装所需的库：

在 Ubuntu 上：

```sh
sudo apt-get update
sudo apt-get install libssl1.0.0 libasound2
```

在 Debian 9:

```sh
sudo apt-get update
sudo apt-get install libssl1.0.2 libasound2
```

* C#：可以引用和使用语音 SDK NuGet 包的最新版本。 若要引用该 SDK，请向你的项目中添加以下包引用：

  ```xml
  <PackageReference Include="Microsoft.CognitiveServices.Speech" Version="1.5.0" />
  ```

* Java:可以引用和使用语音 SDK Maven 包的最新版本。 在 Maven 项目中，将 `https://csspeechstorage.blob.core.windows.net/maven/` 添加为附加存储库，并将 `com.microsoft.cognitiveservices.speech:client-sdk:1.5.0` 引用为一个依赖项。

* C++：将 SDK 下载为 [.tar 包](https://aka.ms/csspeech/linuxbinary)，并将文件解压缩到所选的一个目录中。 下表显示了 SDK 文件夹结构：

  |路径|描述|
  |-|-|
  |`license.md`|许可|
  |`ThirdPartyNotices.md`|第三方声明|
  |`include`|用于 C 和 C++ 的头文件|
  |`lib/x64`|用于与应用程序链接的本机 x64 库|
  |`lib/x86`|用于与应用程序链接的本机 x86 库|

  要创建应用程序，请将必需的二进制文件（以及库）复制到开发环境中。 在生成过程中根据需要添加它们。

### <a name="android"></a>Android

将适用于 Android 的 Java SDK 打包为 [AAR（Android 库）](https://developer.android.com/studio/projects/android-library)，其内附必要的库以及所需的 Android 权限。 它作为包 `com.microsoft.cognitiveservices.speech:client-sdk:1.5.0` 托管在 `https://csspeechstorage.blob.core.windows.net/maven/` 的 Maven 存储库中。

若要从你的 Android Studio 项目中使用该包，请进行以下更改：

* 在项目级 build.gradle 文件中，向 `repository` 部分添加以下内容：

  ```gradle
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

* 在模块级 build.gradle 文件中，向 `dependencies` 部分添加以下内容：

  ```gradle
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.5.0'
  ```

Java SDK 也是[语音设备 SDK](speech-devices-sdk.md) 的一部分。

[!INCLUDE [Get the samples](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>后续步骤

* [获取语音试用订阅](https://azure.microsoft.com/try/cognitive-services/)
* [了解如何在 C# 中识别语音](quickstart-csharp-dotnet-windows.md)
