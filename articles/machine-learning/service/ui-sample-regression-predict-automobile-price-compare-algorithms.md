---
title: 回归：预测价格和比较算法
titleSuffix: Azure Machine Learning service
description: 此可视界面示例试验演示了如何比较两个预测汽车的价格的回归模型的性能。 此过程包括训练、 测试和评估汽车价格数据 （原始） 数据集上的模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/02/2019
ms.openlocfilehash: 2a4a9e74fa7f56b67f0f4a64f6619db1c5c69a2c
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442125"
---
# <a name="sample-2---regression-predict-price-and-compare-algorithms"></a>示例 2-回归：预测价格和比较算法

此可视界面示例试验演示了如何比较两个预测汽车的价格的回归模型的性能。 过程包括训练、 测试和使用评估模型**汽车价格数据 （原始）** 数据集。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. 选择**打开**示例 2 实验的按钮：

    ![打开试验](media/ui-sample-regression-predict-automobile-price-compare-algorithms/open-sample2.png)

## <a name="related-sample"></a>相关的示例

[示例 1-回归：汽车价格预测 （基本）](ui-sample-regression-predict-automobile-price-basic.md)提供更简单的试验，解决了此试验中相同的问题，但需要使用只有一个回归模型。 如果要查找有关回归的基本示例，请参阅它。

## <a name="experiment-summary"></a>实验摘要

我们使用以下步骤来生成试验：

1. 获取数据。
1. 预先处理数据。
1. 训练该模型。
1. 测试、 评估和比较模型。

下面是实验的完整图形：

[![实验的关系图](media/ui-sample-regression-predict-automobile-price-compare-algorithms/graph.png)](media/ui-sample-regression-predict-automobile-price-compare-algorithms/graph.png#lightbox)


## <a name="get-the-data"></a>获取数据

在此试验中，我们将使用**汽车价格数据 （原始）** 数据集，这是从 UCI 机器学习存储库。 此数据集包含 26 个列包含有关汽车，包括制造商、 型号、 价格、 车辆功能 （如气缸数）、 MPG、 和保险风险评分的信息。 本实验的目标是预测汽车的价格。

## <a name="pre-process-the-data"></a>预处理数据

主数据准备任务包括执行数据清理、 集成、 转换、 缩短，原本和离散化或量化。 在可视化界面，您可以找到模块来执行这些操作和其他数据预先处理中的任务**数据转换**组在左侧面板中。

在此试验中，我们将使用**选择数据集中的列**模块，以排除规范化损失具有多个缺失值。 然后，我们使用**清理缺失数据**删除具有缺失值的行。 这有助于创建干净的定型数据集。

![数据预处理](media/ui-sample-regression-predict-automobile-price-compare-algorithms/data-processing.png)

## <a name="train-the-model"></a>训练模型

机器学习问题而有所不同。 常见的机器学习任务包括分类，聚类分析，回归和推荐器系统，其中每个可能需要不同的算法。 所选的算法通常取决于用例的要求。 在选择一种算法后，您需要调整其参数来定型更准确的模型。 然后需要评估准确性、 清晰度和效率等度量值所基于的所有模型。

由于本实验的目标是预测汽车价格，并且因为标签列 （价格） 包含实数，回归模型是一个不错的选择。 考虑到，许多功能是相对较小 （小于 100），这些功能不是稀疏，则很可能是非线性的决策边界。

若要比较的不同算法的性能，我们使用两个非线性算法**提升决策树回归**并**决策林回归**、 生成模型。 这两种算法具有参数，您可以更改，但我们对于此试验中使用的默认值。

我们使用**拆分数据**模块来将输入的数据随机划分，以便训练数据集包含 70%的原始数据和测试数据集包含 30%的原始数据。

## <a name="test-evaluate-and-compare-the-models"></a>测试、 评估和比较模型

我们使用两个不同的随机选择的数据集来训练和测试模型，在上一部分中所述。 我们将数据集拆分，并使用不同的数据集来训练和测试模型，以便执行更客观的模型的评估。

训练模型后，我们使用**评分模型**并**评估模型**模块来生成预测的结果和评估模型。 **评分模型**使用经过训练的模型生成测试数据集的预测。 我们然后传递到分数**评估模型**生成评估指标。

在此试验中，我们使用的两个实例**评估模型**来比较两个对的模型。

首先，我们将训练数据集上的两种算法进行比较。
其次，我们将进行比较测试数据集上的两种算法。
下面是结果：

![比较的结果](media/ui-sample-regression-predict-automobile-price-compare-algorithms/result.png)

这些结果显示，与生成的模型**提升决策树回归**具有较低的根生成的模型比均方根误差**决策林回归**。

这两种算法比在定型数据集上不可见的测试数据集上具有较低的错误。

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>后续步骤

了解可用于可视化界面的其他示例：

- [示例 1-回归：预测汽车的价格](ui-sample-regression-predict-automobile-price-basic.md)
- [示例 3-分类：预测信用风险](ui-sample-classification-predict-credit-risk-basic.md)
- [示例 4-分类：预测信用风险 （成本敏感）](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
- [示例 5-分类：预测客户流失](ui-sample-classification-predict-churn.md)
