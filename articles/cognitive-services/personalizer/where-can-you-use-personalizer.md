---
title: 方案评估 - 个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: 在应用程序能够选择要显示的正确项、操作或产品的任何场合下，都可以应用个性化体验创建服务，以改善体验，实现更好的业务成果，或提高工作效率。
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: e51ef9afd0e49b690a4f9cab09fdfbd3e86eee66
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441036"
---
# <a name="where-can-you-use-personalizer"></a>可在哪种场合下使用个性化体验创建服务？

在应用程序需要选择要显示的正确项、操作或产品的任何场合下，都可以应用个性化体验创建服务，以改善体验，实现更好的业务成果，或提高工作效率。 

个性化体验创建服务使用机器学习来选择要向用户显示的操作。 所做的选择根据发送到服务的数据数量、质量和分布而有很大的不同。

### <a name="checklist-for-applying-personalizer"></a>应用个性化体验创建服务的核对清单


可以应用个性化体验创建服务的场合包括：

* 为应用程序制定了业务目标或可用性目标。
* 在应用程序中做出有关要向用户显示哪些数据的上下文决策可以改善该目标。
* 可以并且应该从集体用户行为和总体奖励评分中学习最佳选择。
* 个性化机器学习的使用遵循[负责使用准则](ethics-responsible-use.md)和选择的选项。
* 上下文决策可以表达为基于一组有限的选项对最佳选择（操作）进行排名。
* 排名选项对应用程序的适用程度的确定方式包括衡量用户行为的某些方面，并以_奖励评分_的形式来表达此衡量结果。 此评分是 -1 到 1 的数字。
* 奖励评分不会引入过多的混淆或外部因素。 试验持续时间足够短，以便在奖励评分仍然相关的情况下可对其进行计算。
* 可将排名上下文表达为至少包含 5 个[特征](concepts-features.md)的列表，你认为这些特征有助于做出正确的选择，并且它们不包括个人身份信息 (PII)。
* 你已获得有关每个内容选项的信息，即_操作_ - 至少包含 5 个[功能](concepts-features.md)的列表，你认为这些特征有助于个性化体验创建服务做出正确的选择。
* 应用程序可将数据保留足够长的时间，以累积至少 100,000 次交互的历史记录。

## <a name="machine-learning-considerations-for-applying-personalizer"></a>应用个性化体验创建服务时的机器学习注意事项

个性化体验创建服务基于强化学习 - 由提供的反馈传授的机器学习方法。 

在以下情况下，个性化体验创建服务的学习效果最佳：

* 如果问题随时变化（例如新闻或时装的偏向），有足够的事件建立在最佳个性化的基础之上。 个性化体验创建服务可适应现实世界的不断变化，但如果未提供足够的事件和数据用于发现和解决新的模式，则结果不会是最佳的。 应选择发生频率够高的用例。 考虑查找每日至少发生 500 次的用例。
* 上下文和操作提供足够多的[特征](concepts-features.md)用于促进学习。
* 每次调用要排名的操作数目少于 50 个。
* 数据保留设置允许个性化体验创建服务收集足够的数据来执行脱机评估和策略优化。 这通常是至少 50,000 个数据点。

## <a name="monitor-effectiveness-of-personalizer"></a>监视个性化体验创建服务的有效性

可以定期执行[脱机评估](concepts-offline-evaluation.md)来监视个性化体验创建服务的有效性。

## <a name="use-personalizer-with-recommendation-engines"></a>结合推荐引擎使用个性化体验创建服务

许多公司使用推荐引擎、市场营销活动工具、受众细分和聚类、协作筛选和其他方法来向客户推荐大型目录中的产品。

[Microsoft 推荐器 GitHub 存储库](https://github.com/Microsoft/Recommenders)提供了有关生成 Jupyter Notebook 形式的推荐系统的示例和最佳做法。 其中提供了有关为许多常见方案（包括 xDeepFM、SAR、ALS、RBM、DKN）准备数据、生成模型、评估、优化和操作化推荐引擎的可行示例。

个性化体验创建服务可以使用推荐引擎（如果存在）。

* 推荐引擎采用大量的项（例如 500,000 个），并从数百或数千个选项中推荐某个子集（例如前 20 个）。
* 个性化体验创建服务采用少量的操作（但这些操作包含有关自身的大量信息），并根据给定的丰富上下文为操作排名；而大多数推荐引擎只使用有关用户、产品及其交互的少量属性。
* 个性化体验创建服务旨在自主探索用户在所有时间的偏向，这可以产生更好的结果，其中的内容会快速变化，例如新闻、实时事件、实时社区内容、每日更新的内容或季节性内容。

一种常见用途是采用推荐引擎的输出（例如，特定客户的前 20 个产品）并将其用作个性化体验创建服务的输入操作。

## <a name="next-steps"></a>后续步骤

[道德和负责使用](ethics-responsible-use.md)。