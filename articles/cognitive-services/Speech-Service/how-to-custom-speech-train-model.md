---
title: 为自定义语音的语音服务训练模型
titlesuffix: Azure Cognitive Services
description: 定型语音到文本是需要为提高识别为 Microsoft 的基准模型或您正在想创建一个自定义模型的准确性。 对模型进行训练使用用户标记为转录和相关的文本。 以前上传的音频数据，以及这些数据集用于优化和训练语音到文本模型来识别单词、 短语、 首字母缩略词、 名称和其他特定于产品的术语。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 194ae477bb3cba4ac7e3350da6b793c6fea6ecdb
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025853"
---
# <a name="train-a-model-for-custom-speech"></a>为自定义语音训练模型

定型语音到文本是需要为提高识别为 Microsoft 的基准模型或您正在想创建一个自定义模型的准确性。 对模型进行训练使用用户标记为转录和相关的文本。 以前上传的音频数据，以及这些数据集用于优化和训练语音到文本模型来识别单词、 短语、 首字母缩略词、 名称和其他特定于产品的术语。 多域的数据集提供 （与用户会说和你想要识别内容相关的数据），更准确地将您的模型，这会导致改进识别。 请注意，通过将不相关的数据馈送到您的培训，你可以减少，或者会损害您的模型的准确性。

## <a name="use-training-to-resolve-accuracy-issues"></a>使用定型解决准确性问题

如果遇到识别问题与您的模型，使用用户标记的学习记录和相关的数据的额外的培训有助于提高准确性。 使用此表来确定要使用来解决您问题的数据集：

| 使用案例 | 数据类型 | 数据数量 |
|----------|-----------|---------------|
| 专有名词是错误识别 | 相关文本 （句子语音样本） | 10 MB 到 500 MB |
| 单词将由于强调文字颜色 | 相关的文本 （发音） | 提供 misrecognized 的单词 |
| 常见的单词被删除或错误识别 | 音频 + 标记人为的学习记录 | 10 到 1,000 个转录小时 |

> [!IMPORTANT]
> 如果你尚未上传数据集，请参阅[准备和测试数据](how-to-custom-speech-test-data.md)。 本文档提供有关上传数据，以及用于创建高质量数据集的指导原则的说明。

## <a name="train-and-evaluate-a-model"></a>训练和评估模型

训练模型的第一步是将定型数据上传。 使用[准备和测试数据](how-to-custom-speech-test-data.md)有关准备人标记为转录和相关的文本 （语音样本和发音） 的分步说明。 上传培训数据后，按照以下说明开始训练模型：

1. 导航到**语音到文本 > 自定义语音 > 培训**。
2. 单击**训练模型**。
3. 接下来，为您的培训**名称**并**说明**。
4. 从**方案和基线模型**下拉列表菜单中，选择最适合你的域的方案。 如果您不确定选择哪种方案，请选择**常规**。 基线模型是培训的起始点。 如果没有首选项，可以使用最新版本。
5. 从**选择训练数据**页上，选择一个或多个音频 + 人标记为脚本的数据集，你想要使用进行培训。
6. 培训完成后，你可以选择执行测试的最新训练模型的准确性。 此步骤是可选的。
7. 选择**创建**生成自定义模型。

培训表显示到这个新创建的模型相对应的新项。 该表还显示的状态：处理、 成功、 失败。

## <a name="evaluate-the-accuracy-of-a-trained-model"></a>评估已训练模型的准确性

您可以检查数据和评估模型准确性使用这些文档：

* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)


如果你选择测试准确性，务必选择与你用于与您的模型获取模型的性能的实际意义上的一个不同的声学数据集。

## <a name="next-steps"></a>后续步骤

* [将模型部署](how-to-custom-speech-deploy-model.md)

## <a name="additional-resources"></a>其他资源

* [准备和测试你的数据](how-to-custom-speech-test-data.md)
* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
