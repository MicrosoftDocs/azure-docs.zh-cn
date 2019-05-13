---
title: 仪表板-语言理解
titleSuffix: Azure Cognitive Services
description: 使用分析摘要仪表板，可视化的报告工具修复意向。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: a518a697369ff74689a0c4ac05af96453b6a5ca4
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072490"
---
# <a name="how-to-use-the-dashboard-to-improve-your-app"></a>如何使用仪表板来提高您的应用程序

查找并修复问题的已训练的应用程序的意向时使用的示例查询文本。 摘要仪表板显示整体应用程序信息，与应修复的意向的要点。 

查看仪表板分析是一个迭代过程，重复如更改和改进您的模型。

此页将不具有相关的应用程序意向，称为中没有任何示例语音样本分析_模式仅_应用。 

## <a name="what-issues-can-be-fixed-from-dashboard"></a>可以从仪表板固定什么问题？

在仪表板中解决的三个问题是：

|问题|图表颜色|说明|
|--|--|--|
|数据不平衡|-|示例语音样本的数量变化较大时，将发生这种情况。 需要具有所有意图_大致_相同数量的示例查询文本的 None 意向除外。 它只应在应用中具有 10%-15%的语音样本的总数量。<br><br> 如果数据不均衡，但意向的准确性高于特定阈值，这一失衡不会报告为一个问题。<br><br>**启动与此问题： 它可能是其他问题的根本原因。**|
|不清楚的预测|橙色|发生这种情况时关闭它们可能会由于翻转下一步的培训，顶部意向和下一步意向分数[负采样](luis-how-to-train.md#train-with-all-data)或更多示例语音样本添加到意向。 |
|不正确预测|红色|示例查询文本不带标签的意向 （它处于意向） 的预测时，将发生这种情况。|

正确预测用蓝色表示。

摘要仪表板显示了这些问题并告诉你的意向会影响并建议您应该做什么来改进应用。 

## <a name="before-app-is-trained"></a>训练应用之前 

训练应用之前，摘要仪表板不包含任何建议的修补程序。 训练应用以查看这些建议。  

## <a name="check-your-publishing-status"></a>检查发布状态

**发布状态**卡包含有关活动的信息的最后一次发布版本。 

检查活动的版本是你想要修复的版本。 

![摘要仪表板显示了应用的外部服务，发布区域，并聚合终结点命中数。](./media/luis-how-to-use-dashboard/analytics-card-1-shows-app-summary-and-endpoint-hits.png)

这还显示了任何外部服务、 已发布的区域，并聚合终结点命中数。 

## <a name="review-training-evaluation"></a>查看培训评估

**培训评估**卡包含的应用程序的总体准确性按区域聚合的摘要。 分数指示意向的质量。 

![培训评估卡包含有关应用程序的总体准确性的信息的第一个区域。](./media/luis-how-to-use-dashboard/analytics-card-2-shows-app-overall-accuracy.png)

图表指示正确预测的意向，使用不同颜色的问题区域。 改进的建议，此分数增加应用程序。 

建议的修复分隔问题类型和最重要的应用程序。 如果您想审查和修复问题的意图，使用每**[但出现错误的意向](#intents-with-errors)** 卡页的底部。 

每个问题区域具有需要修复的意图。 当您选择意向的名称，**意向**页随即打开到语音样本应用的筛选器。 此筛选器，可专注于导致了该问题的语音样本。

### <a name="compare-changes-across-versions"></a>跨版本比较更改

对应用进行更改之前创建的新版本。 在新的版本中，建议的更改对意向的示例查询文本，然后重新训练。 仪表板页上**培训评估**卡上，使用**从训练版本显示更改**进行比较所做的更改。 

![跨版本比较更改](./media/luis-how-to-use-dashboard/compare-improvement-across-versions.png)

### <a name="fix-version-by-adding-or-editing-example-utterances-and-retraining"></a>通过添加或编辑示例语音样本和重新训练修复版本

修复您的应用程序的主要方法将添加或编辑的示例查询文本并重新训练。 新的或更改查询文本需要遵循的准则[不同的语音样本](luis-concept-utterance.md)。

添加示例查询文本应完成的某个人员：

* 具有很高的语音样本中相应的意图各有哪些了解
* 了解如何在一个意向的语音样本可能会干扰另一个意向
* 能够确定是否两个意向，经常与相互混淆，应折叠为单个的目的，以及不同的数据提取实体

### <a name="patterns-and-phrase-lists"></a>模式和短语列表

在分析页并不指示何时使用[模式](luis-concept-patterns.md)或[短语列表](luis-concept-feature.md)。 如果您添加它们，它可以帮助不正确或不清楚的预测，但不会帮助进行数据不平衡。 

### <a name="review-data-imbalance"></a>查看数据不平衡

启动与此问题： 它可能是其他问题的根本原因。

**数据不平衡**意向列表显示了需要更多语音样本才能更正数据不平衡的意图。 

**若要解决此问题**:

* 到意向添加更多语音样本，然后重新训练。 

不要为 None 意向添加语音样本，除非的建议摘要仪表板上。

> [!Tip]
> 在页上，使用第三个部分**每个意向的语音样本**与**语音样本 （数字）** 设置，来快速 visual 意向需要更多语音样本。  
    ![使用查询文本 （数字） 来查找与数据不平衡的意图。](./media/luis-how-to-use-dashboard/predictions-per-intent-number-of-utterances.png)

### <a name="review-incorrect-predictions"></a>查看不正确的预测

**错误的预测**意向列表显示已作为示例用于以特定的意向，但预测不同意向的语音样本添加的意图。 

**若要解决此问题**:

* 编辑查询文本要再次更具体的意图和训练。
* 如果语音样本太密切和重新训练，组合意向。

### <a name="review-unclear-predictions"></a>查看不清楚的预测

**不清楚预测**意向列表显示了与具有并不是从其最接近的对手，将语音样本顶部意向可能会由于更改下一步的培训，到足够程度方法的预测评分的语音样本的意向[负采样](luis-how-to-train.md#train-with-all-data)。

**若要解决此问题**;

* 编辑查询文本要再次更具体的意图和训练。
* 如果语音样本太密切和重新训练，组合意向。

## <a name="utterances-per-intent"></a>每个意向的语音样本

此卡显示意向的总体应用运行状况。 在修复意向和重新导流，继续粗略地看一下此卡的问题。

下图显示了包含几乎没有要解决的问题的较为平衡应用程序。

![下图显示了包含几乎没有要解决的问题的较为平衡应用程序。](./media/luis-how-to-use-dashboard/utterance-per-intent-shows-data-balance.png)

下图显示了包含很多要解决的问题的效果不佳平衡应用程序。

![下图显示了包含几乎没有要解决的问题的较为平衡应用程序。](./media/luis-how-to-use-dashboard/utterance-per-intent-shows-data-imbalance.png)

将鼠标悬停在每个意向条，以获取有关意向的信息上。 

![下图显示了包含几乎没有要解决的问题的较为平衡应用程序。](./media/luis-how-to-use-dashboard/utterances-per-intent-with-details-of-errors.png)

使用**作为排序依据**功能来排列按问题类型意向，以便你可以专注于与该问题最值得怀疑的意向。 

## <a name="intents-with-errors"></a>出现错误的意向

此卡可以查看以特定的意向的问题。 此卡的默认视图是最值得怀疑的意图，因此你知道将精力集中的位置。

![使用错误卡意向，可查看以特定的意向的问题。 卡片将筛选为最值得怀疑的意向，默认情况下，您告诉您应重点关注的位置。](./media/luis-how-to-use-dashboard/most-problematic-intents-with-errors.png)

顶部的圆环图图表显示三个问题类型意向问题。 如果三个问题类型中的问题，每个类型都有自己下面，以及任何对手意向的图表。 

### <a name="filter-intents-by-issue-and-percentage"></a>筛选器的问题和百分比的意向

在卡的此部分可以找到示例语音样本，正落在超出错误阈值。 理想情况下你想为重要的正确预测。 该百分比是业务和客户驱动的。 

确定您已经习惯了针对您企业的阈值百分比。 

可以使用筛选器以查找与特定问题的意图：

|筛选器|建议的百分比|目的|
|--|--|--|
|最值得怀疑的意向|-|**从这里开始**-修复中此意向的语音样本将提高应用程序，多个其他修补程序。|
|下面的正确预测|60%|这是在所选的意向语音样本，是正确的但置信度得分低于阈值的百分比。 |
|不清楚预测更高版本|15%|这是与最接近的对手意向混淆中所选的意向的语音样本的百分比。|
|上述不正确预测|15%|这是未正确预测所选的意向的语音样本的百分比。 |

### <a name="correct-prediction-threshold"></a>正确的预测阈值

什么是对您确信预测置信度分数？ 在应用程序开发的开头，60%可能是你的目标。 使用**更正预测下面**60%，以便在选中需要修复的方法中查找任何语音样本的百分比。

### <a name="unclear-or-incorrect-prediction-threshold"></a>不明白或不正确预测阈值

这些两个筛选器允许您查找查询文本中超出阈值的所选目的。 您可以将这些两个百分比为错误百分比。 如果您熟悉预测 10-15%错误率，请将筛选器阈值设置为 15%，以便查找大于此值的所有语音样本。 

## <a name="next-steps"></a>后续步骤

* [管理 Azure 资源](luis-how-to-azure-subscription.md)
