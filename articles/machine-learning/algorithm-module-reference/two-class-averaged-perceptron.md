---
title: 决策林回归：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用双类平均感知器模块以创建机器学习基于平均感知器算法的模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: f0fec525ed87f91cf102053383b2934aac4b71c0
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029230"
---
# <a name="two-class-averaged-perceptron-module"></a>双类平均感知器模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来创建机器学习基于平均感知器算法的模型。  
  
这种分类算法是一种监督式的学习方法，并且需要*标记数据集*，其中包括标签列。 你可以通过作为输入提供的模型和标记数据集训练该模型[训练模型](./train-model.md)。 然后可以使用训练的模型来预测新的输入示例的值。  

### <a name="about-averaged-perceptron-models"></a>有关平均感知器模型

*平均感知器方法*是简单的早期版本的神经网络。 在此方法中，输入分为多个可能的输出基于线性函数，然后将与一组派生自特征向量的权重组合 — 因此称之为"感知器。"

更简单的感知器模型适合于学习线性可分模式，而神经网络 (尤其是深层神经网络) 可以进行更复杂的类边界建模。 但是，感知器的速度要快，并且因为它们连续处理用例，感知器可用于连续训练。

## <a name="how-to-configure-two-class-averaged-perceptron"></a>如何配置双类平均感知器

1.  添加**双类平均感知器**模块在试验。  

2.  指定要通过设置训练的模型的方式**创建训练器模式**选项。  
  
    -   **单一参数**:如果你知道你想要配置该模型，提供一组特定的值作为参数。
  
3.  有关**学习速率**，为指定值*学习速率*。 学习速率值控制的每当测试和更正模型时使用随机梯度下降中的步的大小。
  
     通过使速率较小，您测试模型更多时候，风险，您可能会陷入局部停滞。 通过使更大的步骤，您可以聚合速度较快，而且可能会步长。
  
4.  有关**最大迭代数**，键入的号码的次数你想要检查的训练数据的算法。  
  
     早期停止通常提供更好的泛化。 增加迭代次数可改善拟合，但会导致过度拟合风险。
  
5.  有关**随机种子**，可以选择键入要用作种子的整数值。 如果你想要确保跨实验的可再现性运行，建议使用种子。  
  
1.  连接定型数据集和定型模块之一：
  
    -   如果您设置**创建训练器模式**到**单个参数**，使用[训练模型](train-model.md)模块。

## <a name="results"></a>结果

培训完成后：

+ 若要查看的模型的参数，以及从培训，学习到的功能权重摘要右键单击的输出[训练模型](./train-model.md)。


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 