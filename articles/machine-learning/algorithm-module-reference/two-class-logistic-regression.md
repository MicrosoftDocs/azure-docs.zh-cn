---
title: 双类逻辑回归：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用双类逻辑回归模块来创建逻辑回归模型，可用于预测两个 （而且仅两个） 的结果。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: aacaf6c64ef77d0e694f97e3675060eca33794ed
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029245"
---
# <a name="two-class-logistic-regression-module"></a>双类逻辑回归模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来创建逻辑回归模型，可用于预测两个 （而且仅两个） 的结果。 

逻辑回归是众所周知的统计方法，用于对建模许多类型的问题。 这种算法*监督式学习*方法; 因此，必须提供已包含的结果来训练模型的数据集。  

### <a name="about-logistic-regression"></a>有关逻辑回归  

逻辑回归是众所周知的方法中用于预测的结果的概率，尤其是经常用于分类任务的统计信息。 该算法预测的数据拟合到逻辑函数的事件的出现概率。
  
在此模块中，已针对二叉分或二进制变量优化分类算法。 如果你需要对多个结果进行分类，使用[多类逻辑回归](./multiclass-logistic-regression.md)模块。

##  <a name="how-to-configure"></a>如何配置  

若要训练此模型，必须提供包含标签或类列的数据集。 因为此模块适合于双类问题，标签或类列必须包含正好两个值。 

例如，标签列可能 [投票] 具有可能值的"Yes"或"否"。 或者，可能会 [信用风险]，使用"高"或"低"的可能的值。 
  
1.  添加**双类逻辑回归**模块在试验。  
  
2.  指定要通过设置训练的模型的方式**创建训练器模式**选项。  
  
    -   **单一参数**:如果你知道你想要配置该模型，您可以提供一组特定的值作为参数。  
  
3.  有关**优化容差**，指定要优化模型时使用的阈值。 如果迭代间的改进低于指定阈值，该算法被视为一种解决方案，集中讨论和培训停止。  
  
4.  有关**L1 正则化权重**并**L2 正则化权重**，键入要用于 L1 和 L2 正则化参数的值。 同时建议一个非零值。  
  
     *正则化*是一种方法通过牺牲具有极端系数值的模型防止过度拟合。 正则化的工作原理是所关联到假设的错误的系数值有关的处罚添加。 因此，具有极端系数值的准确模型将受到处罚，但是具有更加保守的值的不太准确模型将不容易受到处罚。  
  
     L1 和 L2 正则化具有不同的作用，并使用。  
  
    -   L1 可以应用于稀疏模型处理高维数据时，这很有用。  
  
    -   与此相反，L2 正则化更不是稀疏的数据。  
  
     此算法支持 L1 和 L2 正则化值的线性组合： 即，如果<code>x = L1</code>并<code>y = L2</code>，然后<code>ax + by = c</code>定义正则化条款的线性跨度。  
  
    > [!NOTE]
    >  想要了解有关 L1 和 L2 正则化的详细信息？ 以下文章提供了 L1 和 L2 正则化有何不同，以及它们如何影响模型拟合，与逻辑回归和神经网络模型的代码示例的讨论：[L1 和 L2 正则化的机器学习](https://msdn.microsoft.com/magazine/dn904675.aspx)  
    >
    > 逻辑回归模型中设计了不同的 L1 和 L2 条款的线性组合： 例如，[弹性网络的正则化](https://wikipedia.org/wiki/Elastic_net_regularization)。 我们建议引用这些组合来定义在您的模型中有效的线性组合。
      
5.  有关**L-BFGS 的内存大小**，指定要使用的内存量*L-BFGS*优化。  
  
     L-BFGS 代表"受限内存 Broyden Fletcher Goldfarb Shanno"。 它是一种优化算法，常用于参数估计。 此参数指示过去的位置和渐变来为下一个步骤的计算存储的数字。  
  
     此优化参数限制用于计算下一个步骤和方向的内存量。 指定内存更少，训练速度就越快，但不太准确。  
  
6.  有关**随机种子**，键入一个整数值。 定义一个种子值是重要信息： 如果你想要对多个运行相同的试验可重现的结果。  
  
  
8. 将标记数据集添加到试验，并连接之一[培训模块](module-reference.md)。  
  
    -   如果您设置**创建训练器模式**到**单个参数**，使用[训练模型](./train-model.md)模块。  
  
9. 运行试验。  
  
## <a name="results"></a>结果

培训完成后：

+ 若要查看的模型的参数，以及从培训，学习到的功能权重摘要右键单击的输出[训练模型](./train-model.md)，然后选择**可视化**。   
  
+ 若要对新数据进行预测，使用训练的模型和新数据作为输入[评分模型](./score-model.md)模块。 


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 