---
title: 微调文本到语音转换输出的语音服务
titleSuffix: Azure Cognitive Services
description: 启用语音 SDK 中的日志记录。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 2352594b168f5d37ddb6a46064f9ac5801fb035f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508401"
---
# <a name="fine-tune-text-to-speech-output"></a>微调文本转语音输出

Azure 的语音服务，可以调整速度、 发音、 卷、 音调、 和的文本到语音转换输出使用轮廓[语音合成标记语言 (SSML)](speech-synthesis-markup.md)。 使用标记来告知服务有关的哪项功能需要调整基于 XML 的标记语言为 SSML。 SSML 消息然后文本到语音转换服务的每个请求的正文中发送。 若要简化自定义过程，语音服务现在提供[语音优化](https://aka.ms/voicetuning)真实时间中的工具，它允许你以可视化方式检查并微调文本到语音转换输出。

优化语音工具支持 Microsoft 的[标准](language-support.md#standard-voices)，[神经](language-support.md#text-to-speech)，并[自定义语音](how-to-customize-voice-font.md)。

## <a name="get-started-with-the-voice-tuning-tool"></a>开始使用语音优化工具

在可以开始微调文本到语音转换输出使用语音优化工具之前，您将需要完成以下步骤：

1. 创建[免费的 Microsoft 帐户](https://account.microsoft.com/account)如果还没有一个。
2. 创建[Azure 免费帐户](https://azure.microsoft.com/free/)如果还没有一个。 单击**免费开始**，并创建新的 Azure 帐户使用 Microsoft 帐户。

3. 在 Azure 门户中创建的语音服务订阅。 有关分步说明[如何创建语音资源](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started#create-a-speech-resource-in-azure)可用。
   >[!NOTE]
   >当在 Azure 门户中创建语音资源时，需要匹配与 TTS 语音区域的 Azure 位置信息。 神经 TTS 语音支持的 Azure 位置的子组。 有关支持的完整列表，请参阅[区域](regions.md#text-to-speech)。

   >[!NOTE]
   >需要 F0 或之前您可以使用该服务在 Azure 门户中创建 S0 密钥。 语音 Tuning**不会**支持[30 天免费试用版密钥](https://review.docs.microsoft.com/en-us/azure/cognitive-services/speech-service/get-started?branch=release-build-cogserv-speech-services#free-trial)。

4. 登录到[语音优化](https://aka.ms/voicetuning)门户中，并连接你的语音服务的订阅。 选择单个的语音服务订阅，然后创建一个项目。
5. 选择**新的优化**。 然后执行以下步骤：

   * 找到并选择**的所有订阅**。  
   * 选择“连接现有订阅”。  
     ![连接现有的订阅](./media/custom-voice/custom-voice-connect-subscription.png)。
   * 输入你 Azure 语音服务的订阅密钥，然后选择**添加**。 订阅密钥是从语音自定义门户中提供[订阅页](https://go.microsoft.com/fwlink/?linkid=2090458)。 此外可以在的资源管理窗格中获取的密钥[Azure 门户](https://portal.azure.com/)。 
   * 如果你有多个你打算使用的语音服务订阅，请为每个订阅重复这些步骤。

## <a name="customize-the-text-to-speech-output"></a>自定义文本到语音转换输出

现在，创建了帐户，并链接订阅后，可以开始优化文本到语音转换输出。

1. 选择一种声音。
2. 输入你想要编辑的文本。
3. 在开始进行编辑之前，播放音频，若要了解的输出。
4. 选择你想要进一步调整，word/句子并开始试验不同的基于 SSML 的函数。

>[!TIP]
> 有关调整 SSML 和优化的语音输出的详细信息，请参阅[语音合成标记语言 (SSML)](speech-synthesis-markup.md)。

## <a name="limitations"></a>限制

神经语音优化是比优化为标准和自定义语音略有不同。

* 不支持声调神经语音。
* 俯仰和卷的功能仅适用于完整的句子。 这些功能不可用在 word 级别。
* 率，一些神经语音可以根据其进行调整的单词，而其他人需要您选择整个句子。

> [!TIP]
> 优化语音工具提供了有关功能和优化的上下文信息。

## <a name="next-steps"></a>后续步骤
* [在 Azure 中创建语音资源](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started#create-a-speech-resource-in-azure)
* [启动语音优化](https://speech.microsoft.com/app.html#/VoiceTuning)
* [语音合成标记语言 (SSML)](speech-synthesis-markup.md)
