---
title: 双类支持向量机：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用**双类支持向量机**Azure 机器学习服务创建基于支持向量机算法的模型中的模块。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 2f076dd3a5b1ceb9e24548652a71fda5b9aa48b7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027925"
---
# <a name="two-class-support-vector-machine-module"></a>双类支持向量机模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块创建基于支持向量机算法的模型。 

支持向量机 (Svm) 是一类精心研究的监督式的学习方法。 此特定的实现都适合于连续或分类变量的两个可能的结果的预测。

定义模型参数之后, 使用的培训模块，并提供来训练该模型*标记数据集*包含标签或结果列。

## <a name="about-support-vector-machines"></a>有关支持向量机

支持向量机是在最早的机器学习算法，并已从信息检索到的文本和图像分类的许多应用程序中使用 SVM 模型。 Svm 可用于分类和回归任务。

此 SVM 模型是需要标记的数据的监督式的学习模型。 在定型过程中，算法会分析输入的数据，并识别调用多维度特征空间中的模式*超平面*。  输入的所有示例都映射到输出中类别宽度除以和清除尽可能间隙的方式的类别和此空间中点形式表示。

用于预测，SVM 算法将新示例分配到一个类别或其他，将其映射到该相同的空间。 

## <a name="how-to-configure"></a>如何配置 

对于这种模型类型，建议使用它来训练分类器之前规范化该数据集。
  
1.  添加**双类支持向量机**模块在试验。  
  
2.  指定要通过设置训练的模型的方式**创建训练器模式**选项。  
  
    -   **单一参数**:如果你知道你想要配置该模型，您可以提供一组特定的值作为参数。  

3.  有关**的迭代次数**，键入一个数字来指示生成模型时使用的迭代数量。  
  
     此参数可用于控制训练速度和准确性之间进行权衡。  
  
4.  有关**Lambda**，键入要用作 L1 正则化权重的值。  
  
     此正则化系数可用于优化模型。 较大的值妨碍更复杂的模型。  
  
5.  选择的选项**规范化特征**，如果希望在定型之前的特征进行规范化。
  
     如果应用规范化，在定型之前, 的数据点是在平均值处居中，并且扩展一个单位的标准偏差。
  
6.  选择的选项**投影到单位球**、 规范化系数。
  
     投影到单元空间值意味着训练之前，数据点将以 0 为中心和缩放，以一个单位的标准偏差。
  
7.  在中**随机种子**，键入要用作种子，如果你想要确保跨运行重现能力的整数值。  否则，系统时钟值用作种子，这可能导致在运行的略有不同的结果。
  
9. 连接标记的数据集，以及之一[培训模块](module-reference.md):
  
    -   如果您设置**创建训练器模式**到**单个参数**，使用[训练模型](train-model.md)模块。
  

10. 运行试验。

## <a name="results"></a>结果

培训完成后：

+ 若要查看的模型的参数，以及从培训，学习到的功能权重摘要右键单击的输出[训练模型](./train-model.md)，然后选择**可视化**。

+ 若要使用已定型的模型进行预测，连接到的已训练的模型[评分模型](score-model.md)模块。


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 