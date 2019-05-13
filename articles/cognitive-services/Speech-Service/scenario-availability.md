---
title: 方案可用性-语音服务
titlesuffix: Azure Cognitive Services
description: 语音服务区域的参考。
services: cognitive-services
author: chrisbasoglu
manager: xdh
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: cbasoglu
ms.openlocfilehash: d844b171ff99dc97e5d1107bcb745f9e8d5b3e9d
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65519830"
---
# <a name="scenario-availability"></a>方案可用性

语音服务 SDK 功能很多情况下跨各种编程语言和环境。  并非所有方案都尚在所有的编程语言或所有环境中当前可用。  下面列出的是每个方案的可用性。

- **语音识别 (SR)、 短语列表、 意图、 转换和在本地容器**
  - 所有编程语言/环境没有箭头链接 <img src="media/index/link.jpg" height="15" width="15"></img> 快速入门表中[此处](https://aka.ms/csspeech)。
- **Text-to-Speech (TTS)**
  - C++/ Windows 和 Linux
  - C#/Windows
  - 可以在每个其他情况下使用 TTS REST API。
- **唤醒文字 (关键字 Spotter/KWS)**
  - C++/ Windows 和 Linux
  - C#/Windows & Linux
  - Python/Windows & Linux
  - Java/Windows 和 Linux 和 Android （语音设备 SDK）
  - 唤醒字 (关键字 Spotter/KWS) 功能可能会使用任何麦克风类型，官方 KWS 支持，但是，当前限制为麦克风阵列中找到 Azure Kinect DK 硬件或语音设备 SDK
- **语音第一个虚拟助手**
  - C++/ Windows 和 Linux 和 macOS
  - C#/Windows
  - Java/Windows 和 Linux 和 macOS 和 Android (语音设备 SDK)
- **对话听录**
  - C++/ Windows 和 Linux
  - C#（framework 和.NET Core） / Windows UWP & Linux
  - Java/Windows 和 Linux 和 Android （语音设备 SDK）
- **呼叫中心听录**
  - REST API，可以在任何情况下使用
- **编解码器压缩音频输入**
  - C++/Linux
  - C#/Linux
  - Java/Linux 和 Android
