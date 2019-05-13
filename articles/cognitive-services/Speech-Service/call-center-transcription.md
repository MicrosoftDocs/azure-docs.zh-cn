---
title: 呼叫中心听录 - 语音服务
titleSuffix: Azure Cognitive Services
description: 语音转文本的一个常见应用场景是听录来自各种系统（例如交互式语音应答 (IVR)）的大量电话数据。 收到的可能是立体声或单音的原始音频，音频信号只经过极少量的后期处理，甚至未经任何后期处理。 企业可以在多种音频捕获系统中使用语音服务和统一语音模型获得高质量的听录。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: overview
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 7d844f4d2ad77f5b7cc53275a24167e5f2e71b78
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025379"
---
# <a name="speech-services-for-telephony-data"></a>用于处理电话数据的语音服务

通过座机、手机和无线电生成的电话数据通常质量很低，处于 8 KHz 的窄频范围内，在执行语音转文本过程中会带来挑战。 Azure 语音服务的最新语音识别模型非常擅长听录这些电话数据，即使是人类也难以识别的数据。 这些模型已使用大量的电话数据进行训练，即使在嘈杂的环境中，也能保持行业领先的准确度。

语音转文本的一个常见应用场景是听录来自各种系统（例如交互式语音应答 (IVR)）的大量电话数据。 这些系统提供的音频可能是立体声或单音的，音频信号只经过极少量的后期处理，甚至未经任何后期处理。 无论使用哪种系统来捕获音频，企业都可以使用语音服务和统一语音模型获得高质量的听录。

电话数据可用于更好地了解客户的需求，识别新的营销机会，或者评估呼叫中心座席的工作绩效。 听录数据后，企业可以使用输出内容来改进遥测功能，识别关键短语或分析客户的情绪。

本页中所述的技术是 Microsoft 在内部针对各种处于实时和批处理模式的电话支持处理服务开发的。

让我们回顾一下 Azure 语音服务提供的技术和相关功能。

> [!IMPORTANT]
> 语音服务统一模型已使用多样化的数据进行训练，可针对从听写到电话分析在内的多种方案提供单一模型解决方案。

## <a name="azure-technology-for-call-centers"></a>适用于呼叫中心的 Azure 技术

此外，在功能方面，应用于呼叫中心的语音服务的主要用途是改善客户体验。 在此方面存在三个明确的应用领域 

* 呼叫后分析，即，对通话录音进行批处理 
* 在通话过程中，对音频信号进行实时的分析处理，以提取各种见解（最常见的用例是情绪分析）
* 使用虚拟助手（机器人）推动客户与机器人之间的对话，以尝试在无需座席干预的情况下解决客户的问题，或者应用 AI 协议来为座席提供辅助。

下面的![呼叫中心听录体系结构](media/scenarios/call-center-transcription-architecture.png)插图描绘了批处理方案实现的典型体系结构示意图

## <a name="speech-analytics-technology-components"></a>语音分析技术组件

无论应用领域是呼叫后处理还是实时处理，Azure 都能提供一组成熟的新潮技术来改善客户体验。 

### <a name="speech-to-text-stt"></a>语音转文本 (STT) 

在任何呼叫中心解决方案中，[语音转文本](speech-to-text.md)都是最受欢迎的功能。 由于许多下游分析流程都依赖于听录的文本，因此误字率 (WER) 至关重要。 呼叫中心听录的主要难题之一是呼叫中心普遍存在的噪音（例如，其他座席在幕后讲话）、各种不同的语言区域性和方言，以及低质量的实际电话信号。 WER 与针对给定区域性训练声音和语言模型的效果密切相关，因此，能够根据区域性自定义模型非常重要。 我们的最新 4.x 版统一模型是可以改善听录准确度和延迟的解决方案。 统一模型已使用数万小时的声音数据和几十亿段词法信息进行训练，是市场中可听录呼叫中心数据的最准确模型之一。

### <a name="sentiment"></a>情绪
衡量客户是否获得了良好的体验是在呼叫中心领域应用的语音分析服务的最重要功能之一。 我们的[批量听录 API](batch-transcription.md) 基于每段言语提供情绪分析。 可以聚合在听录通话过程中获取的值集，以确定座席和客户在通话时的情绪。

### <a name="silence-non-talk"></a>静默（无对话）
在呼叫支持的过程中，有 35% 的时间双方无对话也是常用的事。 出现无对话的情况包括：座席正在查阅以往与客户之间展开的案例历史记录、座席正在使用工具访问客户的桌面和执行功能、客户正在等待转接电话，等等。 衡量通话中出现静默的时间极其重要，因为在此类场景中，及时对重要客户灵敏做出回复以及何时可以出现静默都是有绩效分数的。

### <a name="translation"></a>翻译
某些公司正在试验提供从外语支持呼叫翻译的听录内容，使交付经理能够了解其客户的全球体验。 我们的[翻译](translation.md)功能非常优秀。 我们可以将大量区域中的音频翻译成音频，或者将音频翻译成文本。

### <a name="text-to-speech"></a>文本到语音转换
在实现可与客户交互的机器人时，[文本转语音](text-to-speech.md)是另一个重要的方面。 典型的流程是客户讲话、将客户的语音转录为文本、分析文本中的意向、基于识别的意向合成响应，然后向客户呈现一个资产，或者生成合成的语音响应。 当然，所有这些步骤必须快速完成 – 因此，这些系统的成功与否与延迟密切相关。 

考虑到涉及的[语音转文本](speech-to-text.md)、[Luis](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/)、[Bot Framework](https://dev.botframework.com/)、[文本转语音](text-to-speech.md)等各种技术，我们的端到端延迟非常低。 

我们的新语音与人类语音没有区别。 我们的语音可让机器人获得独特的个性。

### <a name="search"></a>搜索
分析的另一个重要方面是在发生特定的事件或体验时识别交互。 这通常是使用两种方法实现的：即席搜索 - 用户只需键入短语，系统即可做出响应；结构化程度更高的查询 - 分析人员可以创建一组逻辑语句用于识别通话中的场景，然后可以针对这些查询集为每次通话编制索引。 一个典型的搜索示例是我们经常听到的合规性语句“为保证服务质量，此次通话将被录音...” – 许多公司希望确保其座席在真正为通话录音之前，向客户提供这段免责声明。 大多数分析系统都可以跟踪查询/搜索算法找到的行为 – 这种趋势报告最终是分析系统的最重要功能之一。 通过[认知服务目录](https://azure.microsoft.com/services/cognitive-services/directory/search/)，可以使用索引和搜索功能显著增强端到端解决方案。

### <a name="key-phrase-extraction"></a>关键短语提取
此领域是更具挑战性的分析应用之一，它受益于人工智能和机器学习的应用。 此处的主要场景是推断客户的意向。 客户为何来电？ 客户遇到了哪种问题？ 客户为何会受到负面的体验？ 我们的[文本分析服务](https://azure.microsoft.com/services/cognitive-services/text-analytics/)提供一组现成的分析功能，可以快速升级端到端解决方案以提取这些重要的关键字或短语。

现在，让我们更详细地了解一下语音识别的批处理和实时管道。

## <a name="batch-transcription-of-call-center-data"></a>批量听录呼叫中心数据

为了批量听录音频，我们开发了[批量听录 API](batch-transcription.md)。 批量听录 API 旨在以异步方式听录大量的音频数据。 在听录呼叫中心数据方面，我们的解决方案基于以下支柱：

* **准确度**：我们通过第四代统一模型提供无可比拟的听录质量。
* **延迟**：我们知道，在执行批量听录时，需要保证听录的速度。 通过[批量听录 API](batch-transcription.md) 启动的听录作业将立即排入队列，一旦执行该作业，其执行速度比实时听录更快。
* **安全性**：我们知道，通话中可能包含敏感数据。 请放心，保证安全是我们的最高优先事务之一。 我们的服务已通过 ISO、SOC、HIPAA 和 PCI 认证。

呼叫中心每天生成大量的音频数据。 如果你的企业将电话数据存储在某个中心位置（例如 Azure 存储），则你可以使用 [批量听录 API]((batch-transcription.md) 以异步方式请求和接收听录内容。

典型的解决方案使用以下服务：

* 使用 Azure 语音服务将语音转录为文本。 若要使用批量听录 API，需要具备语音服务的标准订阅 (SO)。 免费订阅 (F0) 不可用。
* 使用 [Azure 存储](https://azure.microsoft.com/services/storage/)来存储电话数据以及批量听录 API 返回的听录内容。 此存储帐户应使用通知，特别是添加新文件时。 这些通知用于触发听录过程。
* 使用 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/) 来为每个录制内容创建共享访问签名 (SAS) URI，以及触发启动听录的 HTTP POST 请求。 此外，Azure Functions 用于创建通过批量听录 API 检索和删除听录内容的请求。
* 听录完成时，使用 [WebHook](webhooks.md) 来接收通知。

在内部，我们使用上述技术来支持批量模式的 Microsoft 客户呼叫。
![Batch 体系结构](media/scenarios/call-center-batch-pipeline.png)

## <a name="real-time-transcription-for-call-center-data"></a>呼叫中心数据的实时听录

某些企业需要实时听录对话。 实时听录可用于识别关键字、针对对话相关的内容和资源触发搜索、监视情绪、改善可访问性，或者为听不懂本地语言的客户和座席提供翻译。

对于需要实时听录的场景，我们建议使用[语音 SDK](speech-sdk.md)。 目前，语音转文本支持 [20 多种语言](language-support.md)，该 SDK 适用于 C++、C#、Java、Python、Node.js 和 Javascript。 [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk) 上提供了适用于每种语言的示例。 有关最新消息和更新，请参阅[发行说明](releasenotes.md)。

在内部，我们使用上述技术来分析实时发生的 Microsoft 客户呼叫。

![Batch 体系结构](media/scenarios/call-center-reatime-pipeline.png)

## <a name="a-word-on-ivrs"></a>有关 IVR 的说明

使用[语音 SDK](speech-sdk.md) 或 [REST API](rest-apis.md) 可以轻松将语音服务集成到任何解决方案中。 但是，呼叫中心听录可能需要额外的技术。 通常，需要在 IVR 系统与 Azure 之间建立连接。 尽管我们不提供此类组件，但会描述需要与 IVR 建立哪种连接。

有多种 IVR 或电话服务产品（例如 Genesys 或 AudioCodes）可以提供集成功能，利用这些功能可与 Azure 服务建立入站和出站音频直通连接。 简单而言，自定义的 Azure 服务可以提供一个特定的接口用于定义电话呼叫会话（例如“呼叫开始”或“呼叫结束”），并公开一个 WebSocket API 用于接收与语音服务一起使用的入站流音频。 出站响应（例如对话听录或者与 Bot Framework 的连接）可与 Microsoft 文本转语音服务合成，并返回到 IVR 进行播放。

另一种方案是直接 SIP 集成。 Azure 服务连接到 SIP 服务器，因此可以获取要在语音转文本和文本转语音阶段使用的入站流和出站流。 若要连接到 SIP 服务器，可以使用市售的软件产品/服务，例如 Ozieki SDK，或[团队呼叫和会议 API](https://docs.microsoft.com/graph/api/resources/calls-api-overview?view=graph-rest-beta)（目前为 Beta 版），它们都旨在支持此类音频呼叫方案。

## <a name="customize-existing-experiences"></a>自定义现有体验

Azure 语音服务能够很好地与内置模型配合工作，但是，你可能想要根据自己的产品或环境，进一步自定义和优化体验。 自定义选项的范围从声学模型优化，到专属于自有品牌的语音字体。 生成自定义模型后，可将其与实时或批量模式下的任何 Azure 语音服务配合使用。

| 语音服务 | 模型 | 说明 |
|----------------|-------|-------------|
| 语音转文本 | [声学模型](how-to-customize-acoustic-models.md) | 为特定环境（例如汽车或工厂车间）中使用的应用程序、工具或设备创建自定义声学模型，每个模型具有特定的录制条件。 示例包括带有口音的讲话、特定的背景噪音，或使用特定的麦克风录制音频。 |
| | [语言模型](how-to-customize-language-model.md) | 创建自定义语言模型来改善行业特定的词汇和语法的听录，例如医疗术语中或 IT 行话。 |
| | [发音模型](how-to-customize-pronunciation.md) | 借助自定义发音模型，可以定义语音形式以及字词或术语的显示。 它适用于处理自定义术语，如产品名称或首字母缩略词。 只需使用发音文件（简单的 .txt 文件）即可。 |
| 文本转语音 | [语音字体](how-to-customize-voice-font.md) | 使用自定义语音字体可为自有品牌创建可识别的独一无二的声音。 只需使用少量的数据即可开始创建。 提供的数据越多，语音字体就越自然，且越接近人类语音。 |

## <a name="sample-code"></a>代码示例

GitHub 中提供了每个 Azure 语音服务的示例代码。 这些示例涵盖了常见方案，例如，从文件或流中读取音频、连续和单次识别，以及使用自定义模型。 使用以下链接查看 SDK 和 REST 示例：

* [语音转文本和语音翻译示例 (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
* [批量听录示例 (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
* [文本转语音示例 (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="reference-docs"></a>参考文档

* [语音 SDK](speech-sdk-reference.md)
* [语音设备 SDK](speech-devices-sdk.md)
* [REST API：语音转文本](rest-speech-to-text.md)
* [REST API：文本转语音](rest-text-to-speech.md)
* [REST API：批量听录和自定义](https://westus.cris.ai/swagger/ui/index)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [免费获取语音服务订阅密钥](get-started.md)
