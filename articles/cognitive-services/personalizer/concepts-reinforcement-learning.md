---
title: 强化学习 - 个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: 个性化体验创建服务使用有关操作和上下文的信息来做出更好的排名建议。 有关这些操作和上下文的信息是称作“特征”的特性或属性。
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: b65073c0646db0cd0c27a71005bb4f74b091ae09
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506882"
---
# <a name="what-is-reinforcement-learning"></a>什么是强化学习？

强化学习是一种机器学习方法，它可以在使用机器学习的过程中通过获取反馈来学习行为。
 
强化学习的工作原理：

* 提供某种机会或自由度来启用行为 - 例如做出决策或选择。
* 提供有关环境和选项的上下文信息。
* 提供有关该行为对实现特定目标的贡献程度的反馈。

强化学习存在许多的子类型和样式，个性化体验创建服务中的相关概念如下：

* 应用程序提供从备选项列表中显示一段内容的机会。
* 应用程序提供有关每个备选项和用户上下文的信息。
* 应用程序计算奖励评分。

与某些强化学习方法不同，个性化体验创建服务不需要模拟即可正常运行。 其学习算法旨对外界做出反应（而不是控制它），并使用理解功能从每个数据点学习信息，这是一种权衡创建时间与成本的独特机会，即使性能欠佳，也不会带来遗憾（可能奖励的丢失）。

## <a name="what-type-of-reinforcement-learning-algorithms-does-personalizer-use"></a>个性化体验创建服务使用哪种类型的强化学习算法？

最新版本的个性化体验创建服务使用**上下文赌博机**，该强化学习方法限定为在给定上下文中的离散操作之间做出决策或选择。

决策内存（在给定上下文后，训练的模型将捕获可能最佳的决策）使用一组线性模型。 这些方法反复展示了业务成果并且是经过证实的方法，一部分原因是它们可以极快地从真实世界中学习信息，而无需多次通过训练，另一部分原因是它们可为监督式学习模型和深度神经网络模型提供补充。

探索/利用流量分配是遵循针对探索设置的百分比随机发生的，用于探索的默认算法是 Epsilon-Greedy。

### <a name="history-of-contextual-bandits"></a>上下文赌博机的历史

John Langford 杜撰了“上下文赌博机”一词（Langford 和 Zhang [2007]），用于描述强化学习的易驾驭子集，他编写了五六篇论文来提高我们对范式的理解：

* Beygelzimer et al. [2011]
* Dudík et al. [2011a,b]
* Agarwal et al. [2014, 2012]
* Beygelzimer and Langford [2009]
* Li et al. [2010]

John 还曾经编写了几篇教程来阐述联合预测 (ICML 2015)、上下文赌博机理论 (NIPS 2013)、主动学习 (ICML 2009) 和示例复杂性边界 (ICML 2003) 等主题

## <a name="what-machine-learning-frameworks-does-personalizer-use"></a>个性化体验创建服务使用哪种机器学习框架？

个性化体验创建服务目前使用 [Vowpal Wabbit](https://github.com/VowpalWabbit/vowpal_wabbit/wiki) 作为机器学习的基础。 使用所有事件做出个性化排名和训练模型时，此框架可以实现最大的吞吐量和最低的延迟。

## <a name="references"></a>参考

* [Making Contextual Decisions with Low Technical Debt](https://arxiv.org/abs/1606.03966)（做出上下文决策并降低技术债务）
* [A Reductions Approach to Fair Classification](https://arxiv.org/abs/1803.02453)（公平分类的化简方法）
* [Efficient Contextual Bandits in Non-stationary Worlds](https://arxiv.org/abs/1708.01799)（非固定世界中的有效上下文赌博机）
* [Residual Loss Prediction:Reinforcement: learning With No Incremental Feedback](https://openreview.net/pdf?id=HJNMYceCW)（残余损失预测：使用非增量反馈进行强化学习）
* [Mapping Instructions and Visual Observations to Actions with Reinforcement Learning](https://arxiv.org/abs/1704.08795)（使用强化学习将指令和可视观测结果映射到操作）
* [Learning to Search Better Than Your Teacher](https://arxiv.org/abs/1502.02206)（学会如何比老师更好地进行搜索）

## <a name="next-steps"></a>后续步骤

[脱机评估](concepts-offline-evaluation.md) 