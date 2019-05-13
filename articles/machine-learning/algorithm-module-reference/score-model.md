---
title: 评分模型：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用评分模型模块来生成使用经过训练的分类或回归模型的预测。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: f8f7bfcbbf013f2cf32957772086d7e44d31e310
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029260"
---
# <a name="score-model-module"></a>“评分模型”模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来生成使用经过训练的分类或回归模型的预测。

## <a name="how-to-use"></a>如何使用

1. 添加**评分模型**模块在试验。

2. 附加已训练的模型和包含新的输入的数据的数据集。 

    数据应为与所使用的已训练模型的类型兼容的格式。 输入数据集的架构通常也应匹配用于为模型定型的数据的架构。

3. 运行试验。

## <a name="results"></a>结果

生成一组使用的评分之后[评分模型](./score-model.md):

+ 若要生成的一组用于评估模型的准确性 （性能） 的指标。  你可以连接到的已评分数据集[评估模型](./evaluate-model.md)， 
+ 右键单击模块并选择**可视化**可查看结果的示例。
+ 将结果保存到数据集。

分数或预测的值，可以采用许多不同的格式，具体取决于模型和输入的数据：

- 对于分类模型，[评分模型](./score-model.md)输出类，以及预测值的概率的预测的值。
- 对于回归模型，[评分模型](./score-model.md)生成只是预测的数字值。
- 对于图像分类模型，评分可能是对象的映像或一个布尔值，该值指示是否找到了特定功能中的类。

## <a name="publish-scores-as-a-web-service"></a>将评分发布为 web 服务

评分的一个常见用途是返回输出作为预测 web 服务的一部分。 有关详细信息，请参阅如何创建基于 Azure 机器学习中的实验的 web 服务的此教程：


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 