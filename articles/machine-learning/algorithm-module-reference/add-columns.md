---
title: 添加列：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用添加列模块连接两个数据集。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: f1e087e97007c6ba271651a9791c7c3b38a9b9b7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029350"
---
# <a name="add-columns-module"></a>添加列模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块连接两个数据集。 组合中指定作为输入用于创建单个数据集的两个数据集的所有列。 如果你需要连接两个以上的数据集，使用的多个实例**中添加列**。



## <a name="how-to-configure-add-columns"></a>如何配置中添加列
1. 添加**中添加列**模块在试验。

2. 连接你想要连接两个数据集。 如果你想要合并两个以上的数据集，您可以链接在一起的多个组合**中添加列**。

    - 就可以具有不同数量的行的两个列组合在一起。 输出数据集则填充缺少值的较小的源列中的每一行。

    - 不能选择要添加的单个列。 你使用时，将连接每个数据集中的所有列**中添加列**。 因此，如果你想要添加的列的子集，使用选择的列中数据集来创建具有所需的列的数据集。

3. 运行试验。

### <a name="results"></a>结果
运行实验： 之后

- 若要查看新的数据集的第一个行，右键单击的输出**中添加列**，然后选择可视化效果。

新数据集中的列数等于两个输入数据集的列的总和。

如果有两个列具有相同名称的输入数据集中，数字后缀添加到列的名称。 例如，如果有一个名为 TargetOutcome 列的两个实例，左侧的列将被重命名 TargetOutcome_1 和右侧的列将会重命名 TargetOutcome_2。

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 