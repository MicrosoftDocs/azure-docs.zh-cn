---
title: 手动输入数据：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中的手动输入数据模块中键入值来创建一个小型数据集。 数据集可以有多个列。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: ee15b6fb7160ece907d55e790b0ae38ee458ab96
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028600"
---
# <a name="enter-data-manually-module"></a>输入手动数据模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块通过键入值创建一个小型数据集。 数据集可以有多个列。
  
此模块可以是以下应用场景中很有帮助：  
  
- 生成少量用于测试的值  
  
- 创建一个标签的简短列表
  
- 键入要在数据集中插入的列名称的列表

## <a name="enter-data-manually"></a>手动输入数据 
  
1.  添加[手动输入数据](./enter-data-manually.md)模块在试验。 您可以找到此模块中的**数据输入和输出**Azure 机器学习中的类别。 
  
2.  有关**DataFormat**，选择以下选项之一。 这些选项用于确定应如何分析您提供的数据。 为每个格式要求差异很大，因此请务必阅读相关的主题。  
  
    -   **ARFF**。 属性关系文件格式，使用的 Weka。   
  
    -   **CSV**。 以逗号分隔值格式。 有关详细信息，请参阅[转换为 CSV](./convert-to-csv.md)。  
  
    -   **SVMLight**。 使用 Vowpal Wabbit 和其他机器学习框架的格式。  
  
    -   **TSV**。 制表符分隔值格式。

     如果您选择一种格式，并且不提供符合格式规范的数据，会发生运行时错误。
  
3.  在单击**数据**文本框中输入数据。 以下格式需要特别注意：  
  
    - **CSV**:若要创建多个列，在以逗号分隔的文本将粘贴或键入字段之间用逗号分隔的多个列。
  
        如果选择**HasHeader**选项，您可以使用值的第一行作为列标题。  
  
        如果您取消选中此选项、 列名称、 Col1、 Col2 等，则使用。 可以添加或更改列名称使用以后[编辑元数据](./edit-metadata.md)。  
  
    - **TSV**:若要创建多个列，以制表符分隔文本将粘贴或键入多个列的字段之间使用选项卡。  
  
        如果选择**HasHeader**选项，您可以使用值的第一行作为列标题。  
  
        如果您取消选中此选项、 列名称、 Col1、 Col2 等，则使用。 可以添加或更改列名称使用以后[编辑元数据](./edit-metadata.md)。  
  
    -   **ARFF**:将粘贴的现有 ARFF 格式文件中。 如果要直接键入值，请务必在数据的开头添加的可选标头和所需的属性字段。 
    
        例如，以下标头和属性行无法添加到一个简单的列表。 列标题会`SampleText`。
    
        ```text
        % Title: SampleText.ARFF  
        % Source: Enter Data module  
        @ATTRIBUTE SampleText STRING  
        @DATA  
        \<type first data row here>  
        ```

    -   **SVMLight**:键入或粘贴使用 SVMLight 格式的值。  
  
        例如，下面的示例表示 SVMight 格式中的献血数据集的前几行：  
  
        ```text  
        # features are [Recency], [Frequency], [Monetary], [Time]  
        1 1:2 2:50 3:12500 4:98   
        1 1:0 2:13 3:3250 4:28   
        ```  
  
        在运行时[手动输入数据](./enter-data-manually.md)模块，这些行转换为列的数据集和索引值，如下所示：  
  
        |Col1|第 2 列|Col3|第 4 列|标签|  
        |-|-|-|-|-|  
        |0.00016|0.004|0.999961|0.00784|第|  
        |0|0.004|0.999955|0.008615|第|  
  
4.  在每个行，以开始新行后按 ENTER。  
  
     **请确保在最后一行后按 ENTER。** 
     
     如果按 ENTER 多次添加多个空尾随行、 最后一个空的行中删除已剪裁，但其他为空的行被视为缺失值。  
  
     如果您创建具有缺失值的行，可以始终其进行筛选更高版本。  
  
5.  右键单击模块并选择**运行所选**来分析数据，并将其加载到作为数据集的工作区。  
  
     若要查看数据集，请单击输出端口，然后选择**可视化**。  
## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 