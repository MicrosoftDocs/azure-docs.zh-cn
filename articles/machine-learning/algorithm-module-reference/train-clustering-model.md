---
title: 训练聚类分析模型：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用训练聚类分析模型模块来训练聚类分析模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/06/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 41cdec1d7f1c3932b17da6f9b1de518071f3f542
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028075"
---
# <a name="train-clustering-model"></a>训练聚类分析模型

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来训练聚类分析模型。

该模块将未训练的聚类分析模型，你已配置为使用[K-means 聚类分析](k-means-clustering.md)模块，并定型模型时使用标记或未标记的数据集。 该模块创建可用于预测和一组定型数据中的每个用例的群集分配的这两个训练的模型。

> [!NOTE]
> 聚类分析模型不是使用训练[训练模型](train-model.md)模块，这是泛型的模块，用于训练机器学习模型。 这是因为[训练模型](train-model.md)仅适用于监督式学习算法。 K 平均值和其他聚类分析算法允许非监督式的学习，这意味着该算法可以了解从未标记的数据。  
  
## <a name="how-to-use-train-clustering-model"></a>如何使用训练聚类分析模型  
  
1.  添加**训练聚类分析模型**模块添加到你在 Studio 中的试验。 也可以找到该模块下的**机器学习模块**，在**训练**类别。  
  
2. 添加[K-means 聚类分析](k-means-clustering.md)模块或另一个自定义模块创建的兼容聚类分析模型，并设置聚类分析模型的参数。  
    
3.  将训练数据集附加到的右侧的输入**训练聚类分析模型**。
  
5.  在中**列设置**，从要在构建群集时使用的数据集选择的列。 请务必选择列的很好的功能： 例如，避免使用 Id 或其他具有唯一值的列或具有全部相同的值的列。

    如果标签不可用，可以使用它作为一项功能，或忽略它。  
  
6. 选择的选项**结果仅用于检查追加或取消选中**，如果你想要与新的分类标签将训练数据输出。

    如果取消选择此选项，仅群集分配会输出。 

7. 运行试验，或单击**训练聚类分析模型**模块，并选择**运行所选项**。  
  
### <a name="results"></a>结果

定型后已完成：


+  若要查看在数据集中的值，用鼠标右键单击该模块，选择**结果数据集**，然后单击**可视化**。

+ 若要保存训练的模型，以便重复使用更高版本，请右键单击模块，选择**Trained 模型**，然后单击**保存为训练的模型**。

+ 若要从模型生成分数，请使用[将数据分配到群集](assign-data-to-clusters.md)。



## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 