---
title: 分类：预测信用风险
titleSuffix: Azure Machine Learning service
description: 此可视界面示例试验演示了如何执行二元分类以预测信用风险根据信用额度应用程序中提供信息。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/02/2019
ms.openlocfilehash: 3d4ec3c71aaed6bddb012fb17ee5bb96da00cd76
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028525"
---
# <a name="sample-3---classification-predict-credit-risk"></a>示例 3-分类：预测信用风险

此可视界面示例试验演示了如何执行二元分类以预测信用风险根据信用额度应用程序中提供信息。 它显示了如何执行基本分类，包括数据处理操作控件、 设置数据集拆分为训练和测试集，为模型定型、 测试数据集，评分和评估预测。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. 选择**打开**示例 3 实验的按钮：

    ![打开试验](media/ui-sample-classification-predict-credit-risk-basic/open-sample3.png)

## <a name="related-sample"></a>相关的示例

[示例 4-分类：信用风险预测 （成本敏感）](ui-sample-classification-predict-credit-risk-cost-sensitive.md)提供了一个高级的试验，解决了此试验中相同的问题。 它演示了如何执行_成本敏感_使用的分类**执行 Python 脚本**模块，并比较两个二进制分类算法的性能。 如果你想要了解有关如何生成分类实验详细信息，请参阅它。

## <a name="data"></a>数据

我们从 UC Irvine 存储库使用德语信用卡数据集。
数据集包含 1000 示例使用 20 个特征和 1 个标签。 每个示例表示一个人。 这些功能包括数值和分类特征。 请参阅[UCI 网站](https://archive.ics.uci.edu/ml/datasets/Statlog+%28German+Credit+Data%29)分类特征的含义。 最后一列是标签，它表示信用风险，并且有只有两个可能的值： 高信用风险 = 2 和低信用风险 = 1。

## <a name="experiment-summary"></a>实验摘要


我们按照以下步骤来创建实验：

1. 将德国信用卡 UCI 数据数据集模块拖放到试验画布。
1. 添加**编辑元数据**模块，以便我们可以添加的每个列有意义的名称。
1. 添加**拆分数据**模块来创建训练和测试集。 在第一个输出数据集为 0.7 中设置的行部分。 此设置指定 70%的数据将输出到模块的左侧的端口并将剩余部分写入正确的端口。 我们使用左侧数据集进行训练和测试最适合。
1. 添加**双类提升决策树**模块初始化提升的决策树分类器。
1. 添加**训练模型**模块。 分类器从前面的步骤连接到的左侧输入端口**训练模型**。 添加训练集 (左侧输出端口**拆分数据**) 到右侧输入端口**训练模型**。 **训练模型**将训练分类器。
1. 添加**评分模型**模块并连接**训练模型**模块添加到它。 然后添加测试集 (右侧端口**拆分数据**) 到**评分模型**。 **评分模型**将进行预测。 您可以选择其输出端口，以查看预测和正类概率。
1. 添加**评估模型**模块并连接到其左侧输入端口的已评分数据集。 若要查看评估结果，请选择的输出端口**评估模型**模块，并选择**可视化**。
    
以下是完成实验关系图：

![实验的关系图](media/ui-sample-classification-predict-credit-risk-basic/overall-graph.png)


## <a name="results"></a>结果

![评估结果](media/ui-sample-classification-predict-credit-risk-basic/evaluate-result.png)

在评估结果，可以看到模型的 AUC 为 0.776。 在阈值 0.5 时，精度是 0.621，重新调用是 0.456，F1 分数是 0.526。

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>后续步骤

了解可用于可视化界面的其他示例：

- [示例 1-回归：预测汽车的价格](ui-sample-regression-predict-automobile-price-basic.md)
- [示例 2-回归：比较进行汽车价格预测算法](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [示例 4-分类：预测信用风险 （成本敏感）](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
- [示例 5-分类：预测客户流失](ui-sample-classification-predict-churn.md)