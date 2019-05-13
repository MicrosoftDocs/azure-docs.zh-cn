---
title: 导入数据：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 机器学习服务中使用导入数据模块将数据加载到机器学习实验从现有的云数据服务。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: ed51c4e7b6c7d226c7827d1ba00bc96a7be1e6b0
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028300"
---
# <a name="import-data-module"></a>导入数据模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块将数据加载到机器学习实验从现有的云数据服务。  
该模块现在功能向导来帮助你选择的存储选项，然后选择从现有的订阅和帐户中进行快速配置所有选项。 需要编辑现有的数据连接？ 没问题;该向导，以便无需从头开始重新加载所有以前的配置详细信息。 
  
定义您想和连接到源的数据后[导入数据](./import-data.md)推断每一列的值，它包含，并将数据加载到 Azure 机器学习工作区的数据类型。 输出[导入数据](./import-data.md)是可用于任何实验的数据集。

  
如果您的源数据发生更改，可以刷新数据集并添加新数据通过重新运行[导入数据](./import-data.md)。 但是，如果您不想要重新读取源每次运行试验，选择**使用缓存结果**选项为 TRUE。 选中此选项后，模块将检查是否已运行试验之前使用的相同的源和相同的输入的选项。 如果找到前一次运行，使用缓存中的数据，而不是重新加载来自源的数据。
 

## <a name="data-sources"></a>数据源

导入数据模块支持以下数据源。 单击链接以查看详细的说明和使用每个数据源的示例。 
 
如果不确定方式或位置应存储数据，请参阅此指南，数据科学过程中的常见数据方案：[在 Azure 机器学习中高级分析的方案](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-plan-sample-scenarios)。 


|数据源| 使用|
|-----------|-----------|  
|[通过 HTTP 的 web URL](./import-from-web-url-via-http.md)|获取使用 HTTP 和 CSV、 TSV、 ARFF 或 SvmLight 格式中未提供的 web URL 托管的数据|  
|[从 Azure Blob 存储导入](./import-from-azure-blob-storage.md) |获取存储在 Azure blob 服务中的数据|  

## <a name="how-to-use-import-data"></a>如何使用导入数据
 
1. 添加**导入数据**模块在试验。 您可以找到此模块中的**数据输入和输出**界面中的类别。

2. 单击**启动数据导入向导**若要使用向导将数据源配置。

    该向导获取帐户名称和凭据，并帮助您配置其他选项。 如果正在编辑现有的配置，它将首先加载的当前值。

3. 如果不想使用向导，请单击**数据源**，并选择你正在阅读从基于云的存储类型。 

    其他设置取决于您选择的存储类型和存储是否安全。 您可能需要提供帐户名、 文件类型或凭据。 某些源不需要身份验证;对于其他操作系统，可能需要知道帐户名称、 键或容器名称。

4. 选择**使用缓存结果**选项如果想要重用在后续运行将数据集缓存。

    假设不已对模块参数的任何其他更改，实验将加载数据仅在首次模块运行时，并从此使用数据集的缓存的版本。

    如果您需要重新加载数据，每次运行实验，取消选择此选项。

5. 运行试验。

    时导入数据的数据加载到该接口，它包含的值上基于每个列的数据类型推断数字或分类。

    - 如果存在一个标头，则标头用于命名输出数据集的列。

    - 如果数据中不有任何现有的列标题，使用格式 col1，col2，生成新列名称... coln *。

## <a name="results"></a>结果

导入完成后，单击输出数据集，然后选择**可视化**若要查看是否数据已成功导入。

如果你想要保存以便重复使用的数据，而不是导入新的数据集每次运行试验时，右键单击输出并选择**另存为数据集**。 选择数据集的名称。 已保存的数据集在保存的情况时保留数据并且数据不会更新时重新运行试验时，即使在实验中的数据集发生变化。 这可以非常方便地获取数据的快照。

在导入数据后, 它可能需要某些进行建模和分析的其他准备工作：


- 使用[编辑元数据](./edit-metadata.md)若要更改列名称，以处理列作为不同的数据类型，或指示某些列是标签或功能。

- 使用[选择数据集中的列](./select-columns-in-dataset.md)来选择要转换或在建模中使用的列子集。 转换后的或已删除列可以轻松地重新加入到原始数据集通过使用[中添加列](./add-columns.md)模块。  

- 使用[分区和采样](./partition-and-sample.md)分割数据集、 执行采样，或获取前 n 行。

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 