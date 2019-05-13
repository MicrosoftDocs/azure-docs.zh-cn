---
title: 有关直接行语音常见问题
titleSuffix: Azure Cognitive Services
description: 获取有关语音的第一个虚拟助手使用直接行语音通道的最常见问题的解答。
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.openlocfilehash: 8427417c9b579c7dfa21f834ce1ca77099159eb2
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072606"
---
# <a name="voice-first-virtual-assistants-preview-frequently-asked-questions"></a>语音第一个虚拟助手预览：常见问题

如果本文档中找不到您的问题的答案，请查看[其他支持选项](support.md)。

## <a name="general"></a>常规

**问：什么是直接行语音？**

**答:**`SpeechBotConnector`语音 SDK 中提供了双向的与连接到 Bot Framework 上的直接行语音通道的智能机器人的异步通信。 此通道提供协调的访问语音转文本和文本到语音转换，从 Azure 语音服务，使机器人能够以变得完全中的语音，语音出对话式体验。 与唤醒字词和唤醒 Word 验证的支持，此解决方案使可能生成高度可自定义语音第一个虚拟助理为你的品牌或产品。

**问：如何开始使用？**

**答:** 最好的方式开始创建语音第一个虚拟助手是首先[创建 Bot Framework 智能机器人基本](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-basic-deploy?view=azure-bot-service-4.0)。 接下来，连接到智能机器人[直接行语音通道](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech)。

## <a name="debugging"></a>调试

**问：在连接时，我收到 401 错误和无法正常工作。我知道我语音的订阅密钥有效。这是怎么回事？**

**答:** 在预览版中，直接行语音上使用的订阅具有非常特定的限制。 请确保你正在使用**语音**资源 （Microsoft.CognitiveServicesSpeechServices，"Speech"） 和*不***认知服务**资源 (Microsoft.CognitiveServicesAllInOne，"所有认知服务")。 此外，请注意，只有**westus2**当前支持的区域。

![更正订阅的直拨电话语音](media/voice-first-virtual-assistants/faq-supported-subscription.png "兼容的语音订阅的示例")

**问：我收到识别文本直接行语音，但我从我的智能机器人中看到"1011"错误和执行任何操作。为什么？**

**答:** 此错误指示的智能机器人应用程序和直接行语音之间的通信问题。 确保你已经[连接直接行语音通道](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-directlinespeech)，[添加了流式处理协议支持](https://aka.ms/botframework/addstreamingprotocolsupport)到你的智能机器人应用程序 （具有相关 Web 套接字支持），并检查是否在智能机器人应用程序响应传入从通道的请求。

**问：这仍然不起作用和/或我收到其他错误使用 SpeechBotConnector 时并不清楚我该怎么办。什么*应*我？**

**答:** 基于文件的日志记录提供了更多详细信息，可帮助加速支持请求。 若要启用此功能，请参阅[如何使用文件日志记录](how-to-use-logging.md)。

## <a name="next-steps"></a>后续步骤

* [故障排除](troubleshooting.md)
* [发行说明](releasenotes.md)
