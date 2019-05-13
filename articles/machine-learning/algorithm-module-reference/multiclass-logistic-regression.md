---
title: 多类逻辑回归：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中的多类逻辑回归模块来创建可用于预测多个值的逻辑回归模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: ac4310e851808d6e6d89d1a2b506975eea3b1d69
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029320"
---
# <a name="multiclass-logistic-regression-module"></a>多类逻辑回归模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来创建可用于预测多个值的逻辑回归模型。

使用逻辑回归分类是一种监督式的学习方法，并因此需要标记的数据集。 通过为模块的输入中提供的模型和标记的数据集将如训练模型[训练模型](./train-model.md)。 然后可以使用训练的模型来预测新输入示例的值。

Azure 机器学习还提供[双类逻辑回归](./two-class-logistic-regression.md)模块，适合用于分类的二进制或二叉分变量。

## <a name="about-multiclass-logistic-regression"></a>有关多类逻辑回归

逻辑回归是众所周知的方法中用于预测结果概率的并且经常用于分类任务的统计信息。 该算法预测的数据拟合到逻辑函数的事件的出现概率。 

在多类逻辑回归分类器可用于预测多个结果。

## <a name="configure-a-multiclass-logistic-regression"></a>配置多类逻辑回归

1. 添加**多类逻辑回归**到实验的模块。

2. 指定要通过设置训练的模型的方式**创建训练器模式**选项。

    + **单一参数**:如果你知道你想要配置模型，并提供一组特定的值作为参数，请使用此选项。

    + **参数范围**:如果不确定最佳的参数，并且想要使用参数扫描，请使用此选项。

3. **优化容差**，指定优化器收敛的阈值。 如果迭代间的改进低于阈值，则算法将停止并返回当前模型。

4. **L1 正则化权重**， **L2 正则化权重**:键入要用于 L1 和 L2 正则化参数的值。 同时建议一个非零值。

    正则化是一种方法来处罚模型进而具有极端系数值防止过度拟合。 正则化的工作原理是所关联到假设的错误的系数值有关的处罚添加。 具有极端系数值的准确模型将受到处罚，但具有更加保守的值的不太准确模型将不容易受到处罚。

     L1 和 L2 正则化具有不同的作用，并使用。 L1 可以应用于稀疏模型处理高维数据时，这很有用。 与此相反，L2 正则化更不是稀疏的数据。  此算法支持 L1 和 L2 正则化值的线性组合： 即，如果`x = L1`并`y = L2`，`ax + by = c`定义正则化条款的线性跨度。

     不同的线性组合 L1 和 L2 条款的制定对于逻辑回归模型，如[弹性网络的正则化](https://wikipedia.org/wiki/Elastic_net_regularization)。

6. **随机数种子**:键入要用于该算法使用作为种子，如果你想要通过运行可重复的结果的整数值。 否则，系统时钟值用作种子，这可能会产生相同的试验运行中的略有不同结果。

8. 连接标记的数据集和定型模块之一：

    + 如果您设置**创建训练器模式**到**单个参数**，使用[训练模型](./train-model.md)模块。

9. 运行试验。

## <a name="results"></a>结果

完成培训后，您可以看到该模型的参数，以及从培训中了解到的功能权重的摘要，右键单击的输出[训练模型](./train-model.md)模块，然后选择**可视化**.


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 