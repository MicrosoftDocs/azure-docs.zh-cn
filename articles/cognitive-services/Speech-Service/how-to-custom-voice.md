---
title: 什么是自定义语音？ -语音服务
titlesuffix: Azure Cognitive Services
description: 自定义语音是一组的联机工具，您可以创建自己的品牌的可识别的、 独一无二的声音。 若要开始所需的全部是数个音频文件和关联的转录。 请按照以下链接，开始创建自定义语音转文本体验。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: erhopf
ms.openlocfilehash: 8cf9dc6cbfc96448462aac3a64807f8beb6036ad
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65156907"
---
# <a name="get-started-with-custom-voice"></a>开始使用自定义语音

自定义语音是一组的联机工具，您可以创建自己的品牌的可识别的、 独一无二的声音。 若要开始所需的全部是数个音频文件和关联的转录。 请按照以下链接，开始创建自定义文本到语音转换体验。

## <a name="whats-in-custom-voice"></a>什么是自定义语音？

在开始之前使用自定义语音，将需要一个 Azure 帐户和语音服务订阅。 一旦您创建一个帐户，可以准备数据、 训练和测试您的模型、 评估语音质量和最终部署自定义语音模型。

下图突出显示了创建自定义语音模型中使用自定义语音门户的步骤。 使用链接以了解详细信息。

![自定义语音体系结构关系图](media/custom-voice/custom-voice-diagram.png)

1.  [订阅并创建一个项目](https://review.docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice?branch=release-build-cogserv-speech-services#set-up-your-azure-account)-创建 Azure 帐户并创建语音服务订阅。 此统一的订阅，您可以访问语音转文本、 文本到语音，语音翻译和自定义语音门户。 然后，使用的语音服务订阅，创建第一个自定义语音项目。

2.  [将数据上传](https://review.docs.microsoft.com/azure/cognitive-services/speech-service/how-to-customize-voice-font?branch=release-build-cogserv-speech-services#upload-your-datasets)-上传数据 （音频和文本） 使用自定义语音门户或自定义语音 API。 在门户中，你可以调查并评估发音分数和干扰信号比率。 有关详细信息，请参阅[如何准备数据以自定义语音](how-to-custom-voice-prepare-data.md)。

3.  [训练模型](https://review.docs.microsoft.com/azure/cognitive-services/speech-service/how-to-customize-voice-font?branch=release-build-cogserv-speech-services#build-your-voice-font)– 使用你的数据来创建自定义文本到语音转换语音模型。 可以在不同的语言中对模型进行定型。 完成定型后，测试您的模型，并且如果您对结果满意，可以将模型部署。

4.  [将模型部署](https://review.docs.microsoft.com/azure/cognitive-services/speech-service/how-to-customize-voice-font?branch=release-build-cogserv-speech-services#create-and-use-a-custom-endpoint)-创建自定义终结点为您的文本到语音转换语音模型，并将其用于产品、 工具和应用程序中的语音合成。

## <a name="set-up-your-azure-account"></a>设置 Azure 帐户

语音服务订阅是必需的然后才能使用自定义语音门户来创建自定义模型。 按照以下说明在 Azure 中创建的语音服务订阅。 如果还没有 Azure 帐户，你可以注册一个新。  

一旦您创建一个 Azure 帐户和语音服务订阅，你将需要登录到自定义语音门户并连接你的订阅。

1. 从 Azure 门户获取的语音服务订阅密钥。
2. 登录到[自定义语音门户](https://aka.ms/custom-voice)。
3. 选择你的订阅并创建语音项目。
4. 如果你想要切换到另一个语音订阅，使用位于顶部导航栏中的齿轮图标。

> [!NOTE]
> 自定义语音服务不支持 30 天免费试用版密钥。 您必须具有 F0 或之前您可以使用该服务在 Azure 中创建 S0 密钥。

## <a name="how-to-create-a-project"></a>如何创建项目

正如组织数据、 模型、 测试和终结点的内容**项目**自定义语音门户中。 每个项目是语音的特定于国家/地区/语言和想要创建的性别。 例如，可能会创建用于使用英文版在美国 (EN-US) 的呼叫中心的聊天机器人的女性声音的项目。

若要创建第一个项目，选择**Text-到-Speech/自定义语音**选项卡，然后单击**新项目**。 按照向导提供的说明，创建你的项目。 已创建了项目后，您将看到四个选项卡：**数据**，**培训**，**测试**，并且**部署**。 使用中提供的链接[后续步骤](https://review.docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice?branch=release-build-cogserv-speech-services#next-steps)若要了解如何使用每个选项卡。

## <a name="next-steps"></a>后续步骤

- [准备自定义语音数据](how-to-custom-voice.md)
- [创建自定义语音](how-to-custom-voice-create-voice.md)
- [指南：记录您的语音示例](record-custom-voice-samples.md)
