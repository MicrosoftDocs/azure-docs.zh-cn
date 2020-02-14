---
title: 主权云-语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何使用主权云
services: cognitive-services
author: cbasoglu
manager: xdh
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 1/14/2020
ms.author: cbasoglu
ms.openlocfilehash: 3b96ded52aa2148df9e96d6f1e878d50b821abab
ms.sourcegitcommit: d29e7d0235dc9650ac2b6f2ff78a3625c491bbbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2020
ms.locfileid: "76170464"
---
# <a name="speech-services-with-sovereign-clouds"></a>带有主权云的语音服务

## <a name="azure-government-united-states"></a>Azure 政府版（美国）

仅美国联邦、州、本地和部落政府机构及其合作伙伴可访问此专用实例，该实例的运营由甄选出的美国公民控制。
- 区域： US Gov 弗吉尼亚州
- SR in SpeechSDK：*config。FromHost （"wss：//virginia.stt.speech.azure.us"，"\<你的键\>"）;*
- SpeechSDK 中的 TTS： *config。FromHost （"https[]()：//virginia.tts.speech.azure.us"，"\<-key\>"）;*
- 身份验证令牌：[]()https：//virginia.api.cognitive.microsoft.us/sts/v1.0/issueToken
- Azure 门户： https://portal.azure.us  
- 自定义语音门户： https://virginia.cris.azure.us/Home/CustomSpeech
- 可用的 Sku： S0
- 支持的功能：
  - 语音转文本
  - 自定义语音（声音/语言调整）
  - 文本转语音
  - 语音翻译
- 不支持的功能
  - 自定义语音
  - 文本到语音转换的神经语音
- 支持的区域设置：支持以下语言的区域设置。
  - 阿拉伯语（ar-*）
  - 中文（zh-chs）
  - 英语（zh-cn）
  - 法语（fr）
  - 德语（de *）
  - Hindi
  - 韩语
  - 俄语
  - 西班牙语（es）

## <a name="microsoft-azure-china"></a>中国区 Microsoft Azure

位于中国，一种 Azure 数据中心，可直接访问中国 Mobile、中国电信、中国 Unicom 和其他主要承运商主干网络，为中文用户提供高速且稳定的本地网络访问体验。
- 区域：中国东部2（上海）
- SR in SpeechSDK： *config。FromHost （"wss：//chinaeast2.stt.speech.azure.cn"，"\<你的键\>"）;*
- SpeechSDK 中的 TTS： *config。FromHost （"https[]()：//chinaeast2.tts.speech.azure.cn"，"\<-key\>"）;*
- 身份验证令牌：[]()https：//chinaeast2.api.cognitive.azure.cn/sts/v1.0/issueToken
- Azure 门户： https://portal.azure.cn
- 自定义语音门户： https://chinaeast2.cris.azure.cn/Home/CustomSpeech
- 可用的 Sku： S0
- 支持的功能：
  - 语音转文本
  - 自定义语音（声音/语言调整）
  - 文本转语音
  - 语音翻译
- 不支持的功能
  - 自定义语音
  - 文本到语音转换的神经语音
- 支持的区域设置：支持以下语言的区域设置。
  - 阿拉伯语（ar-*）
  - 中文（zh-chs）
  - 英语（zh-cn）
  - 法语（fr）
  - 德语（de *）
  - Hindi
  - 韩语
  - 俄语
  - 西班牙语（es）

