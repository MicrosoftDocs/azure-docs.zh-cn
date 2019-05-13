---
title: 语音设备 SDK 文档
titleSuffix: Azure Cognitive Services
description: 发行说明 - 最新版本中的内容更改
services: cognitive-services
author: wsturman
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/07/2018
ms.author: gracez
ms.openlocfilehash: 9147f02f6ef2e2f3f776e67aa1f103e95ef132aa
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153038"
---
# <a name="release-notes-of-cognitive-services-speech-devices-sdk"></a>认知服务语音设备 SDK 发行说明

以下部分列出了最新版本中的更改。

## <a name="cognitive-services-speech-devices-sdk-150-2019-may-release"></a>认知服务语音设备 SDK 1.5.0:2019-5 月版本

*   语音设备 SDK 现在是 GA 和不再封闭的预览。
*   更新[Speech SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference)组件到版本 1.5.0。 有关详细信息，请参阅其[发行说明](https://aka.ms/csspeech/whatsnew)。
*   新的唤醒 word 技术方面做出了重大的质量改进，请参阅的重大更改。
*   改进了远端域识别的新音频处理管道。

**重大更改**

*   由于新的唤醒 word 技术必须重新创建在我们改进了的唤醒 word 门户唤醒的所有单词。 若要完全删除旧关键字从设备卸载旧应用。
    - adb uninstall com.microsoft.coginitiveservices.speech.samples.sdsdkstarterapp

## <a name="cognitive-services-speech-devices-sdk-140-2019-apr-release"></a>认知服务语音设备 SDK 1.4.0:2019 年 4 月版本 

* 更新[Speech SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference) 1.4.0 版组件。 有关详细信息，请参阅其[发行说明](https://aka.ms/csspeech/whatsnew)。 

## <a name="cognitive-services-speech-devices-sdk-131-2019-mar-release"></a>认知服务语音设备 SDK 1.3.1:2019 年 3 月版本 

* 更新[Speech SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference) 1.3.1 版的组件。 有关详细信息，请参阅其[发行说明](https://aka.ms/csspeech/whatsnew)。 
*   更新的唤醒文字处理，请参阅的重大更改。
*   示例应用程序添加语音识别和翻译的语言的选择。

**重大更改** 

*   [安装唤醒文字](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-devices-sdk-qsg#run-a-sample-application)已简化，现在是应用程序的一部分并且不需要单独安装在设备上的。
*   唤醒单词标识已更改，并支持两个事件。
    - RecognizingKeyword，指示语音结果包含 （未验证） 关键字的文本。
    - RecognizedKeyword，指示完成该关键字识别识别给定的关键字。


## <a name="cognitive-services-speech-devices-sdk-110-2018-nov-release"></a>认知服务语音设备 SDK 1.1.0：2018 年 11 月版本 

* 已将[语音 SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference) 组件更新到 1.1.0 版。 有关详细信息，请参阅其[发行说明](https://aka.ms/csspeech/whatsnew)。 
* 远场语音识别准确性已通过我们增强的音频处理算法得到提高。
* 示例应用程序添加了中文语音识别支持。

## <a name="cognitive-services-speech-devices-sdk-101-2018-oct-release"></a>认知服务语音设备 SDK 1.0.1：2018 年 10 月版本 

* 将[语音 SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference) 组件更新到了 1.0.1 版。 有关详细信息，请参阅其[发行说明](https://aka.ms/csspeech/whatsnew)。 
* 我们改进的音频处理算法将提高语音识别的准确性  
* 修复了一个连续识别音频会话 bug。

**重大更改** 

* 该版本中推出了大量重大更改。 有关 API 的详细信息，请查看[此页](https://aka.ms/csspeech/breakingchanges_1_0_0)。 
* KWS 模型文件与语音设备 SDK 1.0.1 不兼容。 将新的唤醒词文件写入设备后，将删除现有的唤醒词文件。 

## <a name="cognitive-services-speech-devices-sdk-050-2018-aug-release"></a>认知服务语音设备 SDK 0.5.0：2018 年 8 月版本

* 通过修复音频处理代码中的 Bug，改进了语音识别的准确性。
* 将[语音 SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk-reference) 组件更新到了 0.5.0 版。 有关详细信息，请参阅其[发行说明](releasenotes.md#cognitive-services-speech-sdk-050-2018-july-release)。

## <a name="cognitive-services-speech-devices-sdk-0212733-2018-may-release"></a>认知服务语音设备 SDK 0.2.12733：2018 年 5 月版本

认知服务语音设备 SDK 的第一个公共预览版本。
