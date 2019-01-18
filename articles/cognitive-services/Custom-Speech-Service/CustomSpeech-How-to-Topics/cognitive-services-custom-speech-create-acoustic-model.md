---
title: 教程：使用自定义语音服务创建声学模型 - Microsoft 认知服务 | Microsoft Docs
description: 本教程介绍如何在 Microsoft 认证服务中使用自定义语音服务创建声学模型。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: tutorial
ms.date: 05/03/2017
ms.author: panosper
ms.openlocfilehash: 53e93a08782ba66e69b903c32c4c3c7417e5a801
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344568"
---
# <a name="tutorial-create-a-custom-acoustic-model"></a>教程：创建自定义声学模型

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

在本教程中，将为你希望应用程序能够识别的语音数据创建自定义声学模型。 如果应用程序设计用于特定的环境（例如嘈杂工厂）或特定的用户群，则创建自定义声学模型会很有帮助。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 准备数据
> * 导入声学数据集
> * 创建自定义声学模型

如果没有认知服务帐户，请在开始之前创建一个[免费帐户](https://cris.ai)。

## <a name="prerequisites"></a>先决条件

打开[认知服务订阅](https://cris.ai/Subscriptions)页，确保认知服务帐户已连接到某个订阅。

如果没有列出任何订阅，则可单击“获取免费订阅”按钮，让认知服务为你创建一个帐户。 也可单击“连接现有的订阅”按钮，连接到在 Azure 门户中创建的自定义搜索服务订阅。

若要了解如何在 Azure 门户中创建自定义搜索服务订阅，请参阅[在 Azure 门户中创建认知服务 API 帐户](../../cognitive-services-apis-create-account.md)。

## <a name="prepare-the-data"></a>准备数据

若要向特定域自定义声学模型，必须有一系列语音数据。 此集合包括一组语音数据音频文件，以及每个音频文件的听录文本文件。 音频数据应该代表要在其中使用识别器的场景。

例如：

*   若要更好地识别嘈杂工厂环境中的语音，音频文件应包括嘈杂工厂中的讲话人。
<a name="Preparing data to customize the acoustic model"></a>
*   如果想要优化单个讲话人的性能（例如，罗斯福总统的所有炉边谈话），可以听录他的所有言语，然后，音频文件应该只会包括该讲话人的多个示例。

用于自定义声学模型的声学数据集由两个部分组成：(1) 一组包含语音数据的音频文件；(2) 一个包含所有音频文件听录内容的文件。

### <a name="audio-data-recommendations"></a>音频数据建议

*   应该以 WAV (RIFF) 音频格式存储数据集中的所有音频文件。
*   音频的采样率必须是 8 kHz 或 16 kHz，样本值应存储为未压缩的 PCM 16 位带符号整数（短型）。
*   支持仅单声道（单音）音频文件。
*   音频文件的长度必须为 100 毫秒到 1 分钟。 每个音频文件的开头和末尾最好是至少有 100 毫秒的静音区（往往是 500 毫秒到 1 秒）。
*   如果数据中包含背景噪音，则我们建议在数据中讲话内容的前面和/或后面另外包含一些具有更长寂静区的示例（例如，几秒钟的寂静区）。
*   每个音频文件应包含一条陈述（例如，一条听写句子）、一个查询，或者对话系统的一个轮次。
*   数据集中的每个音频文件应有唯一的文件名和扩展名“wav”。
*   音频文件集应放置到不带子目录的单个文件夹中，整个音频文件集应打包为单个 ZIP 文件存档。

> [!NOTE]
> 通过 Web 门户导入的数据目前限制为 2 GB，因此，这是声学数据集的最大大小。 这相当于以 16 kHz 频率录制了 17 个小时的音频，或者以 8 kHz 的频率录制了 34 个小时的音频。 下表汇总了音频数据的主要要求。
>

| 属性 | 值 |
|---------- |----------|
| 文件格式 | RIFF (WAV) |
| 采样率 | 8000 Hz 或 16000 Hz |
| 声道 | 1（单音） |
| 样本格式 | PCM，16 位整数 |
| 文件持续时间 | 0.1 秒 < 持续时间 < 60 秒 |
| 无声凸环 | > 0.1 秒 |
| 存档格式 | Zip |
| 最大存档大小 | 2 GB |

### <a name="transcriptions"></a>听录

应在单个纯文本文件中包含所有 WAV 文件的听录。 听录文件的每一行应包含一个音频文件的名称，后接相应的听录。 文件名和听录应以制表符 (\t) 分隔。

  例如：
```
  speech01.wav  speech recognition is awesome

  speech02.wav  the quick brown fox jumped all over the place

  speech03.wav  the lazy dog was not amused
```

听录内容应经过文本规范化，以便可由系统处理。 但是，用户在将数据上传到自定义语音服务 _之前_ ，必须完成一些重要的规范化操作。 在准备听录内容时，请参阅相应语言的[听录准则](cognitive-services-custom-speech-transcription-guidelines.md)部分。

使用[自定义语音服务门户](https://cris.ai)完成以下步骤。 

## <a name="import-the-acoustic-data-set"></a>导入声学数据集

准备好音频文件和听录内容后，可将其导入服务 Web 门户。

为此，请先确保已登录到[自定义语音服务门户](https://cris.ai)。 然后单击顶部功能区的“自定义语音”下拉菜单，选择“适应数据”。 如果这是第一次将数据上传到自定义语音服务，则会看到名为“数据集”的空表。 

单击“声学数据集”行中的“导入”按钮，此时站点会显示一个用于上传新数据集的页面。

![尝试](../../../media/cognitive-services/custom-speech-service/custom-speech-acoustic-datasets-import.png)

在相应的文本框中输入_名称_和_说明_。 这些项有助于跟踪上传的各种数据集。 接下来，单击“听录文件”和“WAV 文件”旁边的“选择文件”，并分别选择纯文本格式的听录文件，以及 zip 存档格式的 WAV 文件。 此操作完成后，单击“导入”上传数据。 随后将会上传数据。 上传大型数据集可能需要花费几分钟时间。

上传完成后，会返回到“声学数据集”表，并会看到一个对应于声学数据集的条目。 请注意，已为其分配了一个唯一 ID (GUID)。 此数据还会有一个反映其当前状态的状态。 如果正在排队等待处理，则其状态为“正在等待”；如果正在进行验证，则其状态为“正在处理”；如果数据已做好使用准备，则其状态为“完成”。

数据验证包括针对音频文件执行一系列检查以验证文件格式、长度和采样率，并针对听录文件执行一系列检查以验证文件格式和执行一些文本规范化操作。

当状态为“完成”时，可以单击“详细信息”查看声学数据验证报告。 此时会显示通过验证和未通过验证的陈述的数目，以及未通过验证的陈述的详细信息。 在以下示例中，有两个 WAV 文件由于音频格式不正确而未通过验证（在此数据集中，一个文件的采样率不正确，另一个文件的文件格式不正确）。

![尝试](../../../media/cognitive-services/custom-speech-service/custom-speech-acoustic-datasets-report.png)

在某个时候，如果你要更改数据集的名称或说明，可以单击“编辑”链接并更改这些条目。 请注意，无法修改音频文件或听录。

## <a name="create-a-custom-acoustic-model"></a>创建自定义声学模型

当声学数据集的状态为“完成”后，可以使用它来创建自定义声学模型。 为此，请单击“自定义语音”下拉菜单中的“声学模型”。 此时会显示一个名为“你的模型”的表，其中列出了所有自定义声学模型。 首次使用时，此表为空。 当前区域设置显示在表标题中。 目前，只能为“美国英语”创建声学模型。

若要创建新模型，请单击表标题下的“新建”。 像前面一样，输入名称和说明以帮助识别此模型。 例如，可以使用“说明”字段来记录创建模型时使用了哪个起始模型和声学数据集。 接下来，在下拉菜单中选择一个“基本声学模型”。 基本模型是用作自定义操作的起点的模型。 有两个基本声学模型可供选择。 _Microsoft 搜索和听写 AM_ 适用于在应用程序中发出的语音，例如命令、搜索查询或听写。 “Microsoft 对话模型”适用于识别以对话形式说出的语音。 此类语音通常是面对另一个人在呼叫中心或会议中发出。 请注意，“对话”模型中部分结果的延迟高于“搜索”和“听写”模型中的延迟。

接下来，使用下拉菜单选择要用于执行自定义的声学数据。

![尝试](../../../media/cognitive-services/custom-speech-service/custom-speech-acoustic-models-create2.png)

处理完成后，可根据需要选择对新模型执行脱机测试。 这会使用自定义的声学模型针对指定的声学数据集运行语音到文本的评估，并报告结果。 若要执行此测试，请选中“准确度测试”复选框。 然后，从下拉菜单中选择一个语言模型。 如果尚未创建任何自定义语言模型，则下拉列表中只会显示基本语言模型。 参阅指南中基本语言模型的[说明](cognitive-services-custom-speech-create-language-model.md)，并选择最合适的模型。

最后，选择要用于评估自定义模型的声学数据集。 如果执行准确度测试，选择的声学数据必须不同于创建模型时所用的数据，这样才能获得真正有意义的模型性能。 另请注意，脱机测试限制为 1000 条话语。 如果要测试的声学数据集大于该限制，则只会评估最前面的 1000 条话语。

准备好开始运行自定义过程时，请按“创建”。

现在，声学模型表中会出现一个对应于此新模型的新条目。 过程的状态将反映在表中。 状态为“正在等待”、“正在处理”和“完成”。

![尝试](../../../media/cognitive-services/custom-speech-service/custom-speech-acoustic-models-creating.png)

## <a name="next-steps"></a>后续步骤

在本教程中，已开发了一个用于音频文件和脚本的自定义声学模型。 若要创建用于文本文件的自定义语言文件，请继续阅读有关创建自定义语言模型的教程。

> [!div class="nextstepaction"]
> [创建自定义语言模型](cognitive-services-custom-speech-create-language-model.md)
