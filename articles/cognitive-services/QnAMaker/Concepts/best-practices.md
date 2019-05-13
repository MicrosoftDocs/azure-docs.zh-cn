---
title: 最佳做法 - QnA Maker
titlesuffix: Azure Cognitive Services
description: 使用这些最佳做法来改进知识库，并向应用程序/聊天机器人的最终用户提供更好的结果。
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 05/10/2019
ms.author: tulasim
ms.custom: seodec18
ms.openlocfilehash: 78c445e70fcf972be81ef6dfd5d675f8e4843621
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541509"
---
# <a name="best-practices-of-a-qna-maker-knowledge-base"></a>QnA Maker 知识库的最佳做法
[知识库开发生命周期](../Concepts/development-lifecycle-knowledge-base.md)介绍如何从头至尾地管理 KB。 使用这些最佳做法来改进知识库，并向应用程序/聊天机器人的最终用户提供更好的结果。

## <a name="extraction"></a>提取
QnA Maker 服务持续改进着从内容提取 QnA 的算法，并扩展支持的文件和 HTML 格式的列表。 按[指南](../Concepts/data-sources-supported.md)操作，根据文档类型进行数据提取。 

一般情况下，常见问题解答页面应单独存在，且不会与其他信息合并。 产品手册应该具备明确的标题，并且最好有一个索引页。 

## <a name="creating-good-questions-and-answers"></a>创建有价值的问题和解答

### <a name="good-questions"></a>有价值的问题

最佳的问题都很简单。 考虑每个问题的关键词或词组，然后为关键词或词组创建简单的问题。 

根据需要添加尽可能多的备用问题，但变更要简单。 添加更多并非问题主要目标的单词或词组无助于 QnA Maker 查找匹配。 

### <a name="good-answers"></a>有价值的答案

最佳答案都是简单的答案，但不要太简单，比如“是”和“否”。 如果答案可能会链接到其他来源，或者提供丰富的媒体和链接体验，请使用[标记](../how-to/metadata-generateanswer-usage.md)来区分所期望的答案类型，然后将该标记与查询一起提交，以便获得正确答案版本。

## <a name="chit-chat"></a>聊天内容
向机器人中添加聊天内容可以轻松地使其更健谈而有趣。 可以轻松地从预定义的个性添加 chit 聊天数据集时创建知识库，并在任何时候更改其。 了解如何[向知识库添加聊天内容](../How-To/chit-chat-knowledge-base.md)。 

### <a name="choosing-a-personality"></a>选择一种个性
支持多个预定义的个性 chit 聊天： 

|个性 |QnA Maker 数据集文件 |
|---------|-----|
|专业的 |[qna_chitchat_professional.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_professional.tsv) |
|友好 |[qna_chitchat_friendly.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_friendly.tsv) |
|与红 |[qna_chitchat_witty.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_witty.tsv) |
|关注 |[qna_chitchat_caring.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_caring.tsv) |
|热情 |[qna_chitchat_enthusiastic.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_enthusiastic.tsv) |

响应既有正式的，也有非正式的和粗鲁的。 应该选择与所需机器人语气最接近的个性。 您可以查看[数据集](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets)，并选择一个可作为类的基础，智能机器人，然后自定义响应。 

### <a name="edit-bot-specific-questions"></a>编辑特定于机器人的问题
有一些特定于机器人的问题，这些问题是聊天数据集的一部分，并且已经使用泛型答案进行填充。 更改这些答案，使之能够最好地反映机器人的细节。 

建议使下述聊天 QnA 更具体：

* 您是谁?
* 该怎么办？
* 几岁了?
* 谁创建了你？
* 你好
   

## <a name="rankingscoring"></a>排名/评分
请确保自己充分利用了 QnA Maker 支持的排名功能。 这样可提高给定的用户查询获得适当响应的可能性。

### <a name="choosing-a-threshold"></a>选择一个阈值
用作阈值的默认置信度分数为 50，但是，可以根据需要更改知识库的该分数。 由于知识库各不相同，因此应该进行测试，选择最适合知识库的阈值。 详细了解[置信度分数](../Concepts/confidence-score.md)。 

### <a name="add-alternate-questions"></a>添加替代问题
[替代问题](../How-To/edit-knowledge-base.md)可提高用户查询匹配到结果的可能性。 如果同样的问题能以多种方式提出，则替代问题就很有用。 这可以包含句子结构和措辞方式的更改。

|原始查询|备用查询|更改| 
|--|--|--|
|是否有停车的地方？|你有停车的地方吗？|句子结构|
 |嗨|呦<br>你好！|措辞方式或俚语|

<a name="use-metadata-filters"></a>

### <a name="use-metadata-tags-to-filter-questions-and-answers"></a>使用元数据标记筛选问题和解答

[元数据](../How-To/edit-knowledge-base.md)可添加根据元数据标记缩小用户查询结果的范围的能力。 即使查询相同，知识库的答案也可能因元数据标记而有所不同。 例如，如果餐馆分店的位置不同（即，元数据是“位置：西雅图”和“位置：雷德蒙德”），则“停车位在哪里？”就会有不同的答案。

### <a name="use-synonyms"></a>使用同义词
英语中对同义词提供一定程度的支持，可使用不区分大小写的 [word alterations](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fd) 将同义词添加到具有不同形式的关键字中。 同义词应添加到 QnA Maker 服务级别并由服务中的所有知识库共享。

|原始字|同义词|
|--|--|
|buy|purchase<br>netbanking<br>net banking|

### <a name="use-distinct-words-to-differentiate-questions"></a>使用不同的词来区分问题
如果每个问题都有不同需求，那么最好是使用 QnA Maker 的匹配排名算法，这些算法将用户查询与知识库中的问题相匹配。 如果问题间存在重复的相同词组，则会降低为含有这些词的给定用户查询选择正确答案的概率。 

例如，对于以下问题，你可能有两个不同的 QnA：

|QnA|
|--|
|停车位置在哪里|
|ATM 位置在哪里|

由于这两个 QnA 使用很类似的单词来措辞，因此这种相似性可能导致许多措辞类似于“`<x>` 位置在哪里”的用户查询获得很类似的分数。 可以改为使用“停车点在哪里”和“ATM 在哪里”之类的查询进行清晰的区分，避免使用“位置”这样的词，此类词存在于知识库的许多问题中。 

## <a name="collaborate"></a>协作
QnA Maker 让用户可以在知识库上进行[协作](../How-to/collaborate-knowledge-base.md)。 用户需要具备对 Azure QnA Maker 资源组的访问权限，以便访问知识库。 某些组织可能想外包知识库的编辑工作和维护工作，但仍要能保护 Azure 资源的访问权限。 在不同订阅中设置两个完全相同的 [QnA maker 服务](../How-to/set-up-qnamaker-service-azure.md)并选择一个用于编辑测试循环，即可完成编辑者-审批者模型。 完成测试后，请使用[导入-导出](../Tutorials/migrate-knowledge-base.md)进程将知识库内容转移到审批者 QnA Maker 服务，由审批者进行最终的知识库发布和终结点更新。

## <a name="active-learning"></a>主动学习

[主动学习](../How-to/improve-knowledge-base.md)在提供大量基于用户的高质量和高数量的查询时，可以最好地建议备选问题。 重要的是允许客户端应用程序的用户查询在没有审查的情况下参与主动学习反馈循环。 后问题建议 QnA Maker 门户中，你可以**[筛选器由建议](../How-To/improve-knowledge-base.md#add-active-learning-suggestion-to-knowledge-base)** 然后审阅和接受或拒绝这些建议。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [编辑知识库](../How-to/edit-knowledge-base.md)
