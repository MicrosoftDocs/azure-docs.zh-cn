---
title: 添加行：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用添加行模块连接两个数据集。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: ffd693ea3452ef48dc3e05e7bc4a6d3988a487b0
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028750"
---
# <a name="add-rows-module"></a>添加行模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块连接两个数据集。 在串联中的第二个数据集的行添加到第一个数据集的末尾。  
  
串联的行是以下应用场景中很有用：  
  
+ 已生成一系列评估统计数据，并想要将它们组合成一个表中以方便报告。  
  
+ 你一直使用不同的数据集，并且想要合并数据集以创建最终的数据集。  

## <a name="how-to-use-add-rows"></a>如何使用添加行  

若要连接两个数据集中的行，行必须具有相同的架构。 这意味着，相同的列，数和相同类型的列中的数据。

1.  拖动**添加行**入试验的模块，你可以找到它下**Data Transformation**，在**操作**类别。

2. 将数据集连接到两个输入端口。 要追加的数据集应连接到第二个 （右） 端口。 
  
3.  运行试验。 在输出数据集中的行数应等于这两个输入数据集的行的总和。

    如果将同一数据集添加到的两个输入**添加行**重复模块，该数据集。 

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 