---
title: 开始使用自定义语音的语音服务
titlesuffix: Azure Cognitive Services
description: 自定义语音是一组可用于评估和改进 Microsoft 的语音到文本准确性的应用程序、 工具和产品的联机工具。 若要开始所需的全部是几个测试音频文件。 请按照以下链接，开始创建自定义语音转文本体验。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: ab33feb1ffdbced193afaba8f52719b3c215652f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65511119"
---
# <a name="what-is-custom-speech"></a>什么是自定义语音？

[自定义语音](https://aka.ms/custom-speech)是一组可用于评估和改进 Microsoft 的语音到文本准确性的应用程序、 工具和产品的联机工具。 若要开始所需的全部是几个测试音频文件。 请按照以下链接，开始创建自定义语音转文本体验。

## <a name="whats-in-custom-speech"></a>什么是自定义语音？

您可以执行自定义语音的任何操作之前，将需要一个 Azure 帐户和语音服务订阅。 一旦您有一个帐户，你可以准备你的数据、 训练和测试您的模型、 检查识别质量、 评估准确性，并最终部署和使用自定义语音转文本模型。

此关系图突出显示的部分组成的自定义语音门户。 使用以下链接了解有关每个步骤的详细信息。

![突出显示了将自定义语音门户的不同组件。](./media/custom-speech/custom-speech-overview.png)

1. [订阅并创建一个项目](#set-up-your-azure-account)-创建 Azure 帐户和订阅语音服务。 此统一的订阅，您可以访问语音转文本、 文本到语音，语音翻译和自定义语音门户。 然后，使用的语音服务订阅，创建第一个自定义语音项目。

2. [将测试数据上传](how-to-custom-speech-test-data.md)-上传测试数据 （音频文件） 以评估 Microsoft 的文字语音应用程序、 工具和产品的产品/服务。

3. [检查识别质量](how-to-custom-speech-inspect-data.md)-使用自定义语音门户回放已上传的音频，以查看您的测试数据的语音识别质量。 定量度量，请参阅[检查数据](how-to-custom-speech-inspect-data.md)。

4. [评估准确性](how-to-custom-speech-evaluate-data.md)-评估语音到文本模型的准确性。 自定义语音门户将提供*Word 错误率*，这可用于确定是否需要额外的培训。 如果您满意准确性，可以直接使用语音服务 Api。 如果你想要提高准确性相对平均值的 5%-20%，通过使用**培训**门户上传额外的培训数据，例如用户标记的学习记录和相关的文本中的选项卡。

5. [为模型定型](how-to-custom-speech-train-model.md)-通过提供编写的脚本 （10-1000 台小时数） 来增强语音到文本模型的准确性和相关文本 (< 200 MB) 以及您的音频测试数据。 此数据帮助语音到文本模型进行定型。 完成定型后，重新测试，并且如果您对结果满意，则可以部署您的模型。

6. [将模型部署](how-to-custom-speech-deploy-model.md)-创建自定义终结点为您的语音到文本模型并在应用程序、 工具或产品中使用它。

## <a name="set-up-your-azure-account"></a>设置 Azure 帐户

语音服务订阅是必需的然后才能使用自定义语音门户来创建自定义模型。 请按照以下说明创建标准的语音服务订阅操作：[创建语音订阅](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started#new-azure-account)。

> [!NOTE]
> 请务必创建标准 (S0) 订阅，不支持免费试用版 (F0) 订阅。

一旦您创建一个 Azure 帐户和语音服务订阅，你将需要登录到自定义语音门户并连接你的订阅。

1. 从 Azure 门户获取的语音服务订阅密钥。
2. 登录到[自定义语音门户](https://aka.ms/custom-speech)。
3. 选择的订阅需要处理和创建语音项目。
4. 如果你想要修改你的订阅，使用**嵌齿**图标位于顶部导航栏中。

## <a name="how-to-create-a-project"></a>如何创建项目

正如组织数据、 模型、 测试和终结点的内容**项目**自定义语音门户中。 每个项目是特定于域和国家/地区/语言。 例如，可能会创建用于使用美国英语的呼叫中心的项目。

若要创建第一个项目，选择**语音-到-文本/自定义语音**，然后单击**新项目**。 按照向导提供的说明，创建你的项目。 已创建了项目后，应看到四个选项卡：**数据**，**测试**，**培训**，并且**部署**。 使用中提供的链接[后续步骤](#next-steps)若要了解如何使用每个选项卡。

## <a name="next-steps"></a>后续步骤

* [准备和测试你的数据](how-to-custom-speech-test-data.md)
* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)
