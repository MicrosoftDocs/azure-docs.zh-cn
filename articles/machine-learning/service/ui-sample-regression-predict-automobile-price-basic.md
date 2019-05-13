---
title: 回归：预测价格
titleSuffix: Azure Machine Learning service
description: 此可视界面示例试验演示了如何生成一个回归模型来预测汽车的价格。 此过程包括训练、 测试和评估汽车价格数据 （原始） 数据集上的模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/02/2019
ms.openlocfilehash: fa9b9179cda767d69d08dcd357a03123bde901cb
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028885"
---
# <a name="sample-1---regression-predict-price"></a>示例 1-回归：预测价格

此可视界面示例实验演示如何生成一个回归模型来预测汽车的价格。 过程包括训练、 测试和使用评估模型**汽车价格数据 （原始）** 数据集。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. 选择**打开**示例 1 实验的按钮：

    ![打开试验](media/ui-sample-regression-predict-automobile-price-basic/open-sample1.png)

## <a name="related-sample"></a>相关的示例

[示例 2-回归：汽车价格预测 （比较算法）](ui-sample-regression-predict-automobile-price-compare-algorithms.md)提供通过使用两个不同的回归模型来解决此试验中相同的问题的更复杂的示例实验。 它演示了如何快速比较不同的算法。 如果将它签正在寻找更高级示例。

## <a name="experiment-summary"></a>实验摘要

我们使用以下步骤来生成试验：

1. 获取数据。
1. 预先处理数据。
1. 训练该模型。
1. 测试、 评估和比较模型。

下面是实验的完整图形：

![实验的关系图](media/ui-sample-regression-predict-automobile-price-basic/overall-graph.png)

## <a name="get-the-data"></a>获取数据

在此试验中，我们将使用**汽车价格数据 （原始）** 数据集，这是从 UCI 机器学习存储库。 数据集包含 26 个列包含有关汽车，包括制造商、 型号、 价格、 车辆功能 （如气缸数）、 MPG、 和保险风险评分的信息。 本实验的目标是预测汽车的价格。

## <a name="pre-process-the-data"></a>预处理数据

主数据准备任务包括执行数据清理、 集成、 转换、 缩短，原本和离散化或量化。 在可视化界面，您可以找到模块来执行这些操作和其他数据预先处理中的任务**数据转换**组在左侧面板中。

我们使用**选择数据集中的列**模块，以排除规范化损失具有多个缺失值。 然后，我们使用**清理缺失数据**删除具有缺失值的行。 这有助于创建干净的定型数据集。

![数据预处理](./media/ui-sample-regression-predict-automobile-price-basic/data-processing.png)

## <a name="train-the-model"></a>训练模型
机器学习问题而有所不同。 常见的机器学习任务包括分类，聚类分析，回归和推荐器系统，其中每个可能需要不同的算法。 所选的算法通常取决于用例的要求。 在选择一种算法后，您需要调整其参数来定型更准确的模型。 然后需要评估准确性、 清晰度和效率等度量值所基于的所有模型。

由于本实验的目标是预测汽车价格，并且因为标签列 （价格） 包含实数，回归模型是一个不错的选择。 考虑到，许多功能是相对较小 （小于 100），这些功能不是稀疏，则很可能是非线性的决策边界。 因此，我们使用**决策林回归**对于此试验。

我们使用**拆分数据**模块来将输入的数据随机划分，以便训练数据集包含 70%的原始数据和测试数据集包含 30%的原始数据。

## <a name="test-evaluate-and-compare"></a>测试、 评估和比较

 我们将数据集拆分，并使用不同的数据集来训练和测试模型，以便执行更客观的模型的评估。

训练模型后，我们使用**评分模型**并**评估模型**模块来生成预测的结果和评估模型。

**评分模型**使用经过训练的模型生成测试数据集的预测。 若要检查的结果，请选择的输出端口**评分模型**，然后选择**可视化**。

![评分结果](./media/ui-sample-regression-predict-automobile-price-basic/score-result.png)

我们然后传递到分数**评估模型**模块生成的评估指标。 若要检查的结果，请选择的输出端口**评估模型**，然后选择**可视化**。

![评估结果](./media/ui-sample-regression-predict-automobile-price-basic/evaluate-result.png)

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>后续步骤

了解可用于可视化界面的其他示例：

- [示例 2-回归：比较进行汽车价格预测算法](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [示例 3-分类：预测信用风险](ui-sample-classification-predict-credit-risk-basic.md)
- [示例 4-分类：预测信用风险 （成本敏感）](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
- [示例 5-分类：预测客户流失](ui-sample-classification-predict-churn.md)
