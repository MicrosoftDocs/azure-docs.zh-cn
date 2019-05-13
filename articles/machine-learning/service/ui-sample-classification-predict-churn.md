---
title: '分类：预测流失情况、 亲和力，和向上销售 '
titleSuffix: Azure Machine Learning service
description: 此可视界面示例试验演示了二元分类器预测的改动，客户关系管理 (CRM) 的常见任务。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: article
author: xiaoharper
ms.author: zhanxia
ms.reviewer: sgilley
ms.date: 05/02/2019
ms.openlocfilehash: 1cb533348236905b7c4e9b58968041745af0e71b
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028435"
---
# <a name="sample-5---classification-predict-churn-appetency-and-up-selling"></a>示例 5-分类：预测流失情况、 亲和力，和向上销售 

此可视界面示例试验演示了二元分类器预测流失情况、 亲和力和向上销售，客户关系管理 (CRM) 的常见任务。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [aml-ui-prereq](../../../includes/aml-ui-prereq.md)]

4. 选择**打开**示例 5 实验的按钮。

    ![打开试验](media/ui-sample-classification-predict-churn/open-sample5.png)

## <a name="data"></a>数据

对于此试验，我们使用的数据是来自 KDD Cup 2009。 数据集具有 50000 行和 230 特征列。 该任务是预测流失情况、 亲和力，和向上销售的客户使用这些功能。 请参阅[KDD 网站](https://www.kdd.org/kdd-cup/view/kdd-cup-2009)有关详细信息数据和任务。

## <a name="experiment-summary"></a>实验摘要

以下是完成实验关系图：

![试验图](./media/ui-sample-classification-predict-churn/experiment-graph.png)

首先，我们进行一些简单的数据处理。

- 原始数据集包含大量的缺失值。 我们使用**清理缺失数据**模块来替换缺失值 0。

    ![清除数据集](./media/ui-sample-classification-predict-churn/cleaned-dataset.png)

- 功能和相应的流失，亲和力，并追加销售标签以不同的数据集。 我们使用**中添加列**模块以将标签列追加到特征列。 第一列**Col1**，标签列。 列，其余**Var1**， **Var2**，依此类推，是特征列。
 
    ![添加列的数据集](./media/ui-sample-classification-predict-churn/added-column1.png)

- 我们使用**拆分数据**模块将数据集拆分为定型和测试集。


    我们然后使用提升决策树的二元分类器具有默认参数来生成预测模型。 我们构建一个模型，每个任务，即，一个模型每个预测向上销售、 亲和力和改动。

## <a name="results"></a>结果

可视化的输出**评估模型**模块，我们看到模型性能根据测试集。 向上销售任务中，对于 ROC 曲线显示则模型会比随机模型更好地。 曲线 (AUC) 下的区域是 0.857。 在阈值 0.5 时，精度为 0.7，重新调用是 0.463，F1 分数是 0.545。

![评估结果](./media/ui-sample-classification-predict-churn/evaluate-result.png)

 可以移动**阈值**滑块和度量值更改为二进制分类任务，请参阅。

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>后续步骤

了解可用于可视化界面的其他示例：

- [示例 1-回归：预测汽车的价格](ui-sample-regression-predict-automobile-price-basic.md)
- [示例 2-回归：比较进行汽车价格预测算法](ui-sample-regression-predict-automobile-price-compare-algorithms.md)
- [示例 3-分类：预测信用风险](ui-sample-classification-predict-credit-risk-basic.md)
- [示例 4-分类：预测信用风险 （成本敏感）](ui-sample-classification-predict-credit-risk-cost-sensitive.md)
