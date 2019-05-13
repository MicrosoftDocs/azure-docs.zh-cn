---
title: 转换为 CSV:模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中转换为 CSV 模块转换为 CSV 格式，可以下载、 导出，或与 R 或 Python 脚本模块共享数据集。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 8b8b6758cc2df7a092ce36e9507f84ac534d0e3d
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028720"
---
# <a name="convert-to-csv-module"></a>将转换为 CSV 模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块将转换为 CSV 格式，可以下载、 导出，或与 R 或 Python 脚本模块共享数据集。

### <a name="more-about-the-csv-format"></a>有关 CSV 格式的详细信息 

CSV 格式，它代表"逗号分隔值"，是由许多外部机器学习工具所使用的文件格式。 CSV 是一种通用的交换格式时使用的开源语言 R 或 Python 等。

即使您完成大部分工作在 Azure 机器学习中，有些的时候时您可能会发现它便于将你的数据集转换为 CSV 外部工具中使用。 例如：

+ 下载 CSV 文件以使用 Excel 时，将其打开，或将其导入关系数据库。  
+ 保存 CSV 文件以云存储和从创建可视化效果的 Power BI 连接到它。  
+ 使用 CSV 格式来准备用于 R 和 Python 中使用的数据。 只需右键单击要生成的代码直接从 Python 或 Jupyter notebook 中访问数据所需的模块的输出。 

当将数据集转换为 CSV 时，文件保存在 Azure 机器学习工作区中。 一个 Azure 存储实用程序，可用于打开和文件直接使用，或您可以右键单击模块输出和 CSV 文件下载到您的计算机，或 R 或 Python 代码中使用它。  

## <a name="how-to-configure-convert-to-csv"></a>如何配置转换为 CSV

1.  添加[转换为 CSV](./convert-to-csv.md)模块在试验。 您可以找到此模块中的**数据格式转换**组中的接口。 

2. 将其连接到输出数据集的任何模块。   
  
3.  运行试验。

### <a name="results"></a>结果
  

双击的输出[转换为 CSV](./convert-to-csv.md)，然后选择以下选项之一。  

 + **结果数据集-> 下载**:立即在 CSV 格式，您可以将其保存到本地文件夹中打开数据的副本。 如果未指定文件夹，将应用默认文件名称和 CSV 文件保存在本地**下载**库。


 + **结果数据集-> 另存为数据集**:作为单独的数据集 CSV 文件将保存到 Azure 机器学习工作区。

 + **生成数据访问代码**:Azure 机器学习生成两的组代码，以便访问数据，使用 Python 或。若要访问的数据，将复制到你的应用程序的代码片段。 (*即将提供生成数据访问代码。*)

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 