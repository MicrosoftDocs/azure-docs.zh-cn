---
title: 应用转换：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中的应用转换模块来修改基于以前计算转换输入数据集。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 810f375642af49814049589cb83ad17fea578b13
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028705"
---
# <a name="apply-transformation-module"></a>“应用转换”模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来修改输入数据集基于以前计算的转换。  
  
例如，如果使用 z 分数规范化训练数据通过使用**规范化数据**模块，想要使用在评分阶段的培训计算 z 评分值。 在 Azure 机器学习中，你可以保存规范化方法为转换，然后使用**应用转换**评分之前应用于输入数据的 z-分数。
  
Azure 机器学习提供支持的创建，然后将应用许多不同类型的自定义转换。 例如，你可能想要保存，然后在重用转换到：  
  
- 删除或替换缺失值，使用**清理缺失数据**
- 规范化数据，使用**规范化数据**
  

## <a name="how-to-use-apply-transformation"></a>如何使用应用转换  
  
1. 添加**应用转换**模块在试验。 您可以找到此模块下的**机器学习**，在**分数**类别。 
  
2. 找到的现有转换，使用作为输入。  以前保存的转换可在**转换**组在左侧的导航窗格中。  
  
   
  
3. 连接你想要转换的数据集。 数据集应具有完全相同的架构 （列数、 列名、 数据类型） 首次为其设计转换的数据集。  
  
4. 由于所有自定义都是定义该转换时设置所不需要的任何其他参数。  
  
5. 若要将转换应用到新的数据集，运行试验。  

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 