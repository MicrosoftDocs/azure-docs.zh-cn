---
title: 决策林回归：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用决策林回归模块来创建回归模型基于决策树的系综构成。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: d372adf75d46fdedb7a6f2b17e47822475d1f155
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442362"
---
# <a name="decision-forest-regression-module"></a>决策林回归模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来创建回归模型基于决策树的系综构成。

配置完该模型后，你必须训练模型时使用标记的数据集和[训练模型](./train-model.md)模块。  然后可以使用训练的模型进行预测。 

## <a name="how-it-works"></a>工作原理

决策树是为每个实例，遍历二元树数据结构，直到达到叶节点 （决策） 执行一系列简单测试的非参数模型。

决策树具有以下优势：

- 它们是训练和预测期间在计算资源和内存使用效率高。

- 它们可以表示非线性决策边界。

- 它们执行集成的特征选择和分类，并可在出现干扰特征时恢复。

此回归模型由决策树的系综构成。 在回归决策林的每个树输出为预测高斯分布。 若要在模型中找到高斯分布所有树的组合分布到最接近的树的系综通过执行聚合。

有关此算法，其实现的理论框架的详细信息，请参阅这篇文章：[决策林：用于分类、 回归、 密度估计的统一的框架集管学习和半监督式的学习](https://www.microsoft.com/en-us/research/publication/decision-forests-a-unified-framework-for-classification-regression-density-estimation-manifold-learning-and-semi-supervised-learning/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D158806#)

## <a name="how-to-configure-decision-forest-regression-model"></a>如何配置决策林回归模型

1. 添加**决策林回归**到实验的模块。 也可以在接口中找到该模块**机器学习**，**初始化模型**，并**回归**。

2. 打开模块属性，以及**来重新采样方法**，选择用来创建单个树的方法。  您可以从中**装袋**或**复制**。

    - **装袋**:也称为装袋*bootstrap 聚合*。 在回归决策林的每个树输出预测通过高斯分布。 聚合是若要查找其前两个时间匹配的 Gaussians 通过组合所有 Gaussians 返回单个树的给定组合在分钟高斯。

         有关详细信息，请参阅的 Wikipedia 条目[Bootstrap 聚合](https://wikipedia.org/wiki/Bootstrap_aggregating)。

    - **复制**:在复制中，每个树训练，则完全相同的输入数据。 为每个树节点的拆分的使用谓词决定保持随机，树将不同。

         有关定型过程的详细信息**复制**选项，请参阅[决策林以进行计算机视觉和医疗图像分析。Criminisi 和 J.Shotton。Springer 2013。](https://research.microsoft.com/projects/decisionforests/).

3. 指定要通过设置训练的模型的方式**创建训练器模式**选项。

    - **单个参数**

      如果你知道你想要配置该模型，您可以提供一组特定的值作为参数。 您可能已通过试验了解这些值，或接收到作为指导。



4. 有关**决策树数**，指示决策树系综中创建的总数。 通过创建多个决策树，你可能获得更好的覆盖范围，但训练时间会增加。

    > [!TIP]
    > 此值还可以控制当可视化训练的模型时，显示的树数。 如果你想要查看或打印的单个树，可以将值设置为 1;但是，这意味着，只有一个树会生成 （与初始参数集的树），并且将执行任何进一步迭代。

5. 有关**决策树的最大深度**，键入一个数字，以限制任何决策树的最大深度。 增加树的深度可以提高精确性，但一些过度拟合和训练时间增加。

6. 有关**的每个节点的随机拆评分**，键入生成树的每个节点时要使用的拆分数。 一个*拆分*随机划分功能树 （节点） 的每个级别的方式。

7. 有关**最小每个叶节点的样本数**，指示的情况下在树中创建任何终端节点 （叶） 所需的最小数量。

     通过增加此值后，可以增加创建新规则的阈值。 例如，默认值为 1，甚至单个案例可以导致要创建新规则。 如果您增加的值为 5，训练数据将必须包含至少五个满足相同条件的情况。


9. 连接标记的数据集，选择包含不超过两个结果的单标签列，连接到[训练模型](./train-model.md)。

    - 如果您设置**创建训练器模式**选项设为**单个参数**，来使用训练该模型[训练模型](./train-model.md)模块。

   

10. 运行试验。

### <a name="results"></a>结果

培训完成后：

+ 若要查看在每次迭代创建的树，右键单击定型模块的输出，并选择**可视化**。

+ 若要查看每个节点的规则，请单击每个树，并向下钻取到拆分。

+ 若要保存的训练的模型快照，请右键单击定型模块的输出，并选择**保存为训练的模型**。 连续运行试验时不更新模型的此副本。 

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 