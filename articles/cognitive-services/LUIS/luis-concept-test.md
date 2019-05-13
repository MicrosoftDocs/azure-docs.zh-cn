---
title: 测试 LUIS 应用
titleSuffix: Language Understanding - Azure Cognitive Services
description: 进行测试过程中，会向 LUIS 提供示例话语并获取 LUIS 识别出的意向和实体响应。
author: diberry
manager: nitinme
services: cognitive-services
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 01/09/2019
ms.author: diberry
ms.openlocfilehash: 02c079e9cda88c969e5382b2c70c551cf9a88e14
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522483"
---
# <a name="testing-example-utterances-in-luis"></a>在 LUIS 中测试示例话语

进行测试过程中，会向 LUIS 提供示例话语并获取 LUIS 识别出的意向和实体响应。 

可以交互方式[测试](luis-interactive-test.md) LUIS，一次提供一条话语，或[一批](luis-concept-batch-test.md)话语。 通过测试，可比较当前[活动](luis-concept-version.md#active-version)模型与已发布的模型。 

<a name="A-test-score"></a>
<a name="Score-all-intents"></a>
<a name="E-(exponent)-notation"></a>

## <a name="what-is-a-score-in-testing"></a>测试中的分数是什么？
请参阅[预测分数](luis-concept-prediction-score.md)概念，详细了解预测分数。

## <a name="interactive-testing"></a>交互式测试
交互式测试在网站的“测试”面板上进行。 可输入话语，了解意向和实体的识别和打分方式。 在测试窗格中，如果 LUIS 根据话语预测的意向和实体不符合预期，则将其作为新话语复制到“意向”页。 然后标记该话语部分并训练 LUIS。 

## <a name="batch-testing"></a>批处理测试
若要同时测试多条话语，请参阅[批处理测试](luis-concept-batch-test.md)。

## <a name="endpoint-testing"></a>终结点测试
可使用[终结点](luis-glossary.md#endpoint)进行测试，最多可使用两个版本的应用。 将主要或实时版本的应用设置为“生产”终结点，将另一版本添加到“暂存”终结点。 通过此方法可获得三个版本的话语：[LUIS](luis-reference-regions.md) 网站“测试”窗格中的当前模型，以及两个不同终结点上的两个版本。 

所有终结点测试均计入使用配额。 

## <a name="do-not-log-tests"></a>不记录测试
如果对终结点进行测试，并且不希望记录话语，请记得使用 `logging=false` 查询字符串配置。

## <a name="where-to-find-utterances"></a>在哪里可以找到话语
LUIS 将记录的所有话语存储在查询日志中，可在 [LUIS](luis-reference-regions.md) 网站的“应用”列表页，以及 LUIS [创作 API](https://go.microsoft.com/fwlink/?linkid=2092087) 上下载。 

[LUIS](luis-reference-regions.md) 网站的[“查看终结点话语”](luis-how-to-review-endpoint-utterances.md)页列出了 LUIS 不确定的所有话语。 

![查看终结点话语](./media/luis-concept-test/review-endpoint-utterances.png)
 
## <a name="remember-to-train"></a>请记住进行训练
请记住在更改模型后[训练](luis-how-to-train.md) LUIS。 在训练应用前在测试中看不到对 LUIS 应用的更改。 

## <a name="best-practices"></a>最佳做法
了解[最佳实践](luis-concept-best-practices.md)。

## <a name="next-steps"></a>后续步骤

* 详细了解[测试](luis-interactive-test.md)话语。
