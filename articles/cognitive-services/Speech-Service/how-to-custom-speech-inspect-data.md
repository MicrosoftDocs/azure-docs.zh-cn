---
title: 对自定义语音的语音服务检查数据质量
titlesuffix: Azure Cognitive Services
description: 自定义语音提供了工具，可用于以可视方式检查，方法是比较与相应的识别结果的音频数据识别模型的质量。 在自定义语音门户中，可以播放已上传的音频和确定提供的标识结果是否正确。  此工具，可快速检查而无需进行转录任何音频数据的 Microsoft 的基线语音到文本模式或受过培训的自定义模型的质量。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 682f3df362a7fbb0e95a07aa8a8f3a068367eef2
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025814"
---
# <a name="inspect-custom-speech-data"></a>检查自定义语音数据

> [!NOTE]
> 此页面假定您已阅读[准备测试数据以进行自定义语音](how-to-custom-speech-test-data.md)和已上传数据集进行检查。

自定义语音提供了工具，可用于以可视方式检查，方法是比较与相应的识别结果的音频数据识别模型的质量。 在自定义语音门户中，可以播放已上传的音频和确定提供的标识结果是否正确。 此工具，可快速检查而无需进行转录任何音频数据的 Microsoft 的基线语音到文本模式或受过培训的自定义模型的质量。

在本文档中，将学习如何以可视方式检查使用之前上传的定型数据的模型的质量。

在此页上，您将学习如何直观地检查 Microsoft 的基线语音到文本模型和/或已训练的自定义模型的质量。 您将使用上传到的数据**数据**用于测试的选项卡。

## <a name="create-a-test"></a>创建测试

按照这些说明创建一个测试：

1. 导航到**语音到文本 > 自定义语音 > 测试**。
2. 单击**添加测试**。
3. 选择**检查质量 （仅限音频的数据）**。 为测试提供名称、 说明，然后选择音频数据集。
4. 选择你想要测试的最多两个模型。
5. 单击**创建**。

已成功创建的测试后，您可以比较模型并排显示。

## <a name="side-by-side-model-comparisons"></a>通过并行模型比较

测试状态何时*Succeeded*，测试项名称以查看测试的详细信息中单击。 此详细信息页列出了在你的数据集，指示识别结果的两个模型以及从提交的数据集脚本中的所有语音样本。

若要帮助检查-并排比较，可以切换包括插入、 删除和替换各种错误类型。 收听音频，并将每个列 （显示用户标记为脚本以及两个语音到文本模型的结果） 中的识别结果进行比较，您可以决定哪种模型满足你的需求和需要改进的地方。

检查质量测试可用于验证的语音识别终结点的质量是否满足应用程序。  进行客观度量准确性，需要改写成文字的音频，请执行正在测试的说明执行操作：评估准确性。

## <a name="next-steps"></a>后续步骤

* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)

## <a name="additional-resources"></a>其他资源

* [准备测试数据以自定义语音](how-to-custom-speech-test-data.md)
