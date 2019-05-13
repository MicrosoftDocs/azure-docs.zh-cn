---
title: 自定义语音第一个虚拟助手 （预览版）-语音服务
titleSuffix: Azure Cognitive Services
description: 功能、 功能和限制使用直接行语音通道上 Bot Framework 和认知服务语音软件开发工具包 (SDK) 的自定义语音第一个虚拟助手概述。
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: 1344de526564ab623a51eb903951b6a2e866048d
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523479"
---
# <a name="about-custom-voice-first-virtual-assistants-preview"></a>有关自定义语音第一个虚拟助手预览

自定义虚拟助手使用 Azure 语音服务为开发人员助力，使开发人员可以为其应用程序和体验创建自然的、类似于人类的对话接口。 Bot Framework 的 Direct Line Speech 通道通过为兼容机器人提供协调的、安排好的入口点来实现延迟时间短、可靠性高的双向语音交互，从而增强了这些功能。 这些智能机器人可以使用 Microsoft 的语言理解 (LUIS) 进行自然语言交互。 通过使用语音软件开发工具包 (SDK) 的设备访问直接行语音。

   ![直拨电话语音业务流程服务流的概念图](media/voice-first-virtual-assistants/overview.png "语音通道流")

直接行语音和它的相关的功能的自定义语音第一个虚拟助手是理想之选补充[虚拟助手解决方案](https://docs.microsoft.com/azure/bot-service/bot-builder-virtual-assistant-introduction)并[企业级模板](https://docs.microsoft.com/azure/bot-service/bot-builder-virtual-assistant-introduction)。 尽管直接行语音可以使用任何兼容的智能机器人应用程序，这些资源的高质量的对话式体验，以及常见支持技能和模型的快速入门提供的可重用的基线。

## <a name="core-features"></a>核心功能

| Category | 功能 |
|----------|----------|
|[自定义唤醒文字](speech-devices-sdk-create-kws.md) | 可以让用户使用智能机器人使用自定义的关键字，如"您好 Contoso"。 开始会话 使用语音 SDK，可以使用自定义唤醒文字配置中的自定义唤醒 word 引擎完成此任务[可以在此处生成](speech-devices-sdk-create-kws.md)。 直接行语音通道包括提高与单独的设备的唤醒 word 激活准确性的服务端唤醒 word 验证。
|[语音转文本](speech-to-text.md) | 直接行语音通道包括为识别的文本，并使用实时听录音频[语音到文本](speech-to-text.md)从 Azure 语音服务。 此文本是可用于智能机器人和客户端应用程序，如转录。
|[文本到语音转换](text-to-speech.md) | 将使用合成文本响应从智能机器人[文本到语音转换](text-to-speech.md)从 Azure 语音服务。 此合成然后可向客户端应用程序为音频流。 Microsoft 提供的功能构建到自己的品牌，若要了解详细信息中提供语音自己自定义、 高质量神经 TTS 语音[联系我们](mailto:mstts@microsoft.com)。
|[直拨电话语音](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech) | Bot Framework 内的信道，作为直接行语音允许客户端应用程序、 兼容智能机器人应用程序，以及 Azure 语音服务的功能之间的平稳、 无缝连接。 配置智能机器人使用直接行语音通道的详细信息，请参阅[Bot Framework 文档中的其页面](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech)。

## <a name="sample-code"></a>示例代码

GitHub 上提供了用于创建语音第一个虚拟助手的示例代码。 这些示例涉及到几种常用编程语言中智能机器人连接的客户端应用程序。

* [虚拟助手语音第一个示例 (SDK)](https://aka.ms/csspeech/samples)
* [快速入门： 语音的第一个虚拟助手 (C#)](quickstart-virtual-assistant-csharp-uwp.md)
* [快速入门： 语音第一个虚拟助手 (Java)](quickstart-virtual-assistant-java-jre.md)

## <a name="customization"></a>自定义

使用 Azure 的语音服务生成的语音第一个虚拟助手可以使用自定义选项可用于各种[语音到文本](speech-to-text.md)，[文本到语音转换](text-to-speech.md)，和[自定义关键字选择](speech-devices-sdk-create-kws.md)。

> [!NOTE]
> 自定义选项因语言/区域设置而异 (请参阅[支持的语言](supported-languages.md))。

## <a name="reference-docs"></a>参考文档

* [语音 SDK](speech-sdk-reference.md)
* [Azure 机器人服务](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0)

## <a name="next-steps"></a>后续步骤

* [免费获取语音服务订阅密钥](get-started.md)
* [获取语音 SDK](speech-sdk.md)
* [创建和部署基本机器人](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-basic-deploy?view=azure-bot-service-4.0)
* [获取虚拟助手解决方案和企业级模板](https://github.com/Microsoft/AI)
