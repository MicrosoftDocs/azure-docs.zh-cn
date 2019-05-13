---
title: 将数据分配到群集：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用群集模块将分配的数据进行评分聚类分析模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/06/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 1c2d2a02ecfb617551dd9174b87f363d57b151a8
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467199"
---
# <a name="module-assign-data-to-clusters"></a>模块：将数据分配到群集

本文介绍如何使用*将数据分配到群集*模块在 Azure 机器学习可视界面。 该模块将生成通过使用已训练的聚类分析模型的预测*k-means 聚类分析*算法。

将数据分配到群集模块返回包含每个新的数据点的可能分配的数据集。 


## <a name="how-to-use-assign-data-to-clusters"></a>如何使用将分配到群集的数据
  
1. 在 Azure 机器学习 visual 界面中，找到之前已训练的聚类分析模型。 可以创建和使用以下方法之一来训练聚类分析模型：  
  
    - 使用配置 K 平均值聚类分析算法[K-means 聚类分析](k-means-clustering.md)模块，并通过使用数据集和定型聚类分析模型模块 （详见本文） 模型定型。  
  
    - 您还可以添加从现有的训练聚类分析模型**保存模型**组工作区中。

2. 将训练的模型附加到的左侧输入端口**将数据分配到群集**。  

3. 附加新的数据集作为输入。 

   在此数据集，标签是可选的。 通常，聚类分析是一种非监督式的学习方法。 您不需要事先知道类别。 但是，输入的列必须是与用于的训练聚类分析模型，或出现错误的列相同。

    > [!TIP]
    > 若要减少写入到该接口从群集预测的列数，请使用[在数据集中选择列](select-columns-in-dataset.md)，并选择列的子集。 
    
4. 将保留**结果仅用于检查追加或取消选中**复选框的选中状态，如果你想要包含完整输入数据集，其中包括显示结果 （群集分配） 的列的结果。
  
    如果清除此复选框，则返回的结果。 作为 web 服务的一部分创建预测时，此选项可能会很有用。
  
5.  运行试验。  
  
### <a name="results"></a>结果

+  若要查看在数据集中的值，用鼠标右键单击该模块，选择**结果数据集**，然后选择**可视化**。

