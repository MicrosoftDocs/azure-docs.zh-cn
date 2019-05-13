---
title: 获取语音设备 SDK
titleSuffix: Azure Cognitive Services
description: 语音服务适用于多种设备和音频源。 现在，可以通过匹配的硬件和软件进一步利用语音应用程序。 在本文中，将了解如何访问语音设备 SDK 并开始开发。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.custom: seodec18
ms.openlocfilehash: b9a0890000cda0b3663ac29bee61fc1c702f6254
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410693"
---
# <a name="get-the-cognitive-services-speech-devices-sdk"></a>获取认知服务语音设备 SDK

语音设备 SDK 是用于处理目的而专门开发工具包和不同的麦克风阵列配置一个 pretuned 的库。

## <a name="choose-a-development-kit"></a>选择开发工具包

|设备|规格|描述|方案|
|--|--|--|--|
|[Roobo 智能音频的开发人员工具包](https://ddk.roobo.com)</br>[安装程序](speech-devices-sdk-roobo-v1.md) / [快速入门](speech-devices-sdk-android-quickstart.md)![Roobo 智能音频的开发人员工具包](media/speech-devices-sdk/device-roobo-v1.jpg)|7 mic 数组，ARM SOC WIFI、 音频输出、 IO。 </br>Android|第一个语音设备 SDK，以适应 Microsoft Mic 数组和前处理 SDK，用于开发高质量录音和语音方案|会话的脚本、 智能中发表演讲，语音代理，可穿戴设备|
|[Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/)![Azure Kinect DK](media/speech-devices-sdk/device-azure-kinect-dk.jpg)|7 的 mic 数组 RGB 和深度照相机。 </br>Windows/Linux|使用用于构建复杂的计算机视觉和语音模型的高级的人工智能 (AI) 传感器开发工具包。 它将与视频摄像机和方向传感器结合中最佳空间麦克风数组和深度照相机 — 一个小型设备具有多个模式、 选项和 Sdk 以容纳的范围中的所有计算类型。|会话的脚本，机器人，智能建筑|
|Roobo 智能音频的开发人员工具包 2![Roobo 智能音频的开发人员工具包 2](media/speech-devices-sdk/device-roobo-v2.jpg)|7 mic 数组，ARM SOC WIFI、 蓝牙、 IO。 </br>Linux|第 2 代语音设备 SDK 提供备用的 OS 和经济高效的参考设计中的更多功能。|会话的脚本、 智能中发表演讲，语音代理，可穿戴设备|
|URbetter T11 开发板![URbetter DDK](media/speech-devices-sdk/device-urbetter.jpg)|7 mic 数组，ARM SOC WIFI、 以太网、 HDMI、 USB 摄像头。 </br>Linux|语音设备 SDK 采用 Microsoft Mic 数组并支持一个行业级别扩展 HDMI/以太网和多个 USB 外围设备等 I/O|会话的脚本、 教育、 医院、 机器人、 OTT 框中，语音代理，驱动器控件|

## <a name="download-the-speech-devices-sdk"></a>下载语音设备 SDK

下载[语音设备 SDK](https://aka.ms/sdsdk-download)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [语音设备 SDK 入门](https://aka.ms/sdsdk-quickstart)
