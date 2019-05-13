---
title: 对自定义语音的语音服务评估准确性
titlesuffix: Azure Cognitive Services
description: 在本文档将学习如何衡量标准测量 Microsoft 的语音到文本模型或自定义模型的质量。 需要音频 + 人标记为脚本数据以测试准确性，并应提供 5 个小时的具有代表性的音频的 30 分钟。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 7c1b3602b09c1a129923180c4b1d4a5f8293de2c
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025905"
---
# <a name="evaluate-custom-speech-accuracy"></a>评估自定义语音准确性

在本文档中，将学习如何衡量标准测量 Microsoft 的语音到文本模型或自定义模型的质量。 需要音频 + 人标记为脚本数据以测试准确性，并应提供 5 个小时的具有代表性的音频的 30 分钟。

## <a name="what-is-word-error-rate-wer"></a>什么是 Word 错误速率 (WER)？

是行业标准来度量模型准确性*Word 错误率*(WER)。 WER 的不正确识别过程标识的单词进行计数，然后除以总人标记为脚本中提供的单词数。 最后，该数字乘以 100%来计算 WER。

![WER 公式](./media/custom-speech/custom-speech-wer-formula.png)

错误地标识单词归为三个类别：

* 插入 (I):错误地添加假设脚本中的单词
* 删除 (D):假设脚本中未检测到的单词
* 替换 (S):引用和假设之间会被替换的单词

下面是一个示例：

![未正确识别单词的示例](./media/custom-speech/custom-speech-dis-words.png)

## <a name="resolve-errors-and-improve-wer"></a>解决错误并提高 WER

从计算机识别结果 WER 可用于评估的模型将用于应用程序、 工具或产品的质量。 WER 的 5%-10%被认为是高质量，并可供使用。 20%的 WER 为可接受的但可能需要额外的培训。 WER 30%或更多的信号质量不佳，并且需要自定义和培训。

这些错误如何分布很重要。 当遇到许多删除错误时，通常是由于弱音频信号强度。 若要解决此问题，您需要收集到源的音频数据更接近。 插入错误意味着在嘈杂的环境中记录音频和串音可能存在，导致识别问题。 已提供为人标记为转录或相关的特定于域的术语的足够示例时经常遇到替换错误文本。

通过分析单独的文件，可以确定哪种类型的错误存在，以及哪些错误是唯一的特定文件。 在文件级别的了解问题将帮助你为目标的改进。

## <a name="create-a-test"></a>创建测试

如果你想要测试 Microsoft 的语音到文本基线模型或已训练的自定义模型的质量，可以比较两个模型并排比较以评估准确性。 比较内容包括 WER 和识别结果。 通常情况下，自定义模型与 Microsoft 的基准模型进行比较。

若要评估模型并排显示：

1. 导航到**语音到文本 > 自定义语音 > 测试**。
2. 单击**添加测试**。
3. 选择**评估准确性**。 为测试提供名称、 说明，然后选择音频 + 人标记为脚本数据集。
4. 选择你想要测试的最多两个模型。
5. 单击**创建**。

已成功创建你的测试后，您可以比较结果并排显示。

## <a name="side-by-side-comparison"></a>通过并行比较

测试完成后，由状态更改为*Succeeded*，您可以在测试中包含两种模型找到 WER 编号。 单击测试名称以查看测试详细信息页。 此详细信息页列出了在你的数据集，指示识别结果的两个模型以及从提交的数据集脚本中的所有语音样本。 若要帮助检查-并排比较，可以切换包括插入、 删除和替换各种错误类型。 通过收听音频和比较每个列，显示用户标记为脚本中的识别结果和两个语音到文本模型的结果，您可以决定哪种模型满足你的需求和额外的培训和改进的必填。

## <a name="next-steps"></a>后续步骤

* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)

## <a name="additional-resources"></a>其他资源

* [准备和测试你的数据](how-to-custom-speech-test-data.md)
* [检查你的数据](how-to-custom-speech-inspect-data.md)
