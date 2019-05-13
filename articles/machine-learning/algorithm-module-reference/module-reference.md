---
title: 算法和模块参考
titleSuffix: Azure Machine Learning service
description: 了解有关 Azure 机器学习可视界面中可用的模块
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ms.openlocfilehash: 8825f1dc3b66a5c4981ba25a90813aec63975b1f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145139"
---
# <a name="algorithm--module-reference-overview"></a>算法和模块参考概述

本参考内容提供有关每个机器学习算法和模块可在 Azure 机器学习服务的可视界面 （预览版） 中的技术背景。

每个模块表示一组代码，可以独立运行和执行机器学习任务中，提供所需的输入。 模块可能包含特定的算法，或执行在机器学习，如缺失值替换或统计分析中非常重要的任务。

> [!TIP]
> 在可视界面中的任何实验中，可以获取有关特定模块的信息。 选择该模块，然后选择**更多帮助**中的链接**快速帮助**窗格。

## <a name="modules"></a>模块

按功能组织模块：

| 功能 | 描述 | 模块 |
| --- |--- | ---- |
| 数据格式转换 | 将数据在机器学习中使用的各种文件格式之间转换 | [转换为 CSV](convert-to-csv.md) |
| 数据输入和输出 | 将数据从云源移动到试验中。 结果或中间数据写入到 Azure 存储、 SQL 数据库或配置单元时，运行试验时，或使用云存储来试验之间交换数据。  | [导入数据](import-data.md)<br/>[导出数据](export-data.md)<br/>[手动输入数据](enter-data-manually.md) |
| 数据转换 | 对都是唯一的机器学习，例如规范化或装箱数据、 功能选择和维数约简的数据的操作。| [在数据集中选择列](select-columns-in-dataset.md) <br/> [编辑元数据](edit-metadata.md) <br/> [清理缺失数据](clean-missing-data.md) <br/> [添加列](add-columns.md) <br/> [Add Rows](add-rows.md) <br/> [删除重复行](remove-duplicate-rows.md) <br/> [拆分数据](split-data.md) <br/> [规范化数据](normalize-data.md) <br/> [分区和采样](partition-and-sample.md) |
| Python 模块 | 编写代码并将其嵌入在一个模块中将 Python 与实验进行集成。 | [执行 Python 脚本](execute-python-script.md)   <br/> [创建 Python 模型](create-python-model.md)
|  | **机器学习算法**: | |
| 分类 | 预测类。  选择从二进制文件 （两个类） 或多类算法。| [多类决策林](multiclass-decision-forest.md) <br/> [多类逻辑回归](multiclass-logistic-regression.md)  <br/> [多类神经网络](multiclass-neural-network.md)  <br/>  [双类逻辑回归](two-class-logistic-regression.md)  <br/>[双类平均感知器](two-class-averaged-perceptron.md) <br/> [双类&nbsp;提升&nbsp;决策&nbsp;树](two-class-boosted-decision-tree.md)  <br/> [双类决策林](two-class-decision-forest.md)  <br/> [双类神经网络](two-class-neural-network.md)  <br/> [两个&#8209;类&nbsp;支持&nbsp;向量&nbsp;机](two-class-support-vector-machine.md) 
| 群集功能 | 数据组合在一起。| [K 平均值聚类分析](k-means-clustering.md)
| 回归 | 预测值。 | [线性回归](linear-regression.md)  <br/> [神经网络回归](neural-network-regression.md)  <br/> [决策林回归](decision-forest-regression.md)  <br/> [提升&nbsp;决策&nbsp;树&nbsp;回归](boosted-decision-tree-regression.md)
|  | **生成和评估模型**: | |
| 定型   | 通过算法运行数据。 | [训练模型](train-model.md)  <br/> [训练聚类分析模型](train-clustering-model.md)    |
| 评估模型 | 度量已训练模型的准确性。 |  [评估模型](evaluate-model.md)
| Score | 从刚经过训练的模型获取预测。 | [应用转换](apply-transformation.md)<br/>[将分配&nbsp;数据&nbsp;到&nbsp;群集](assign-data-to-clusters.md) <br/>[评分模型](score-model.md)

## <a name="error-messages"></a>错误消息

了解如何[错误消息和异常代码](machine-learning-module-error-codes.md)可能会遇到在 Azure 机器学习服务的可视界面中使用的模块。
