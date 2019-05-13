---
title: 从通过 HTTP 的 Web URL 导入：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用通过 Azure 机器学习服务中的 HTTP 模块从 Web URL 导入从公共网页以便在机器学习实验中使用读取数据。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: bff913efb38c9e5589c795386dfbbc480d799a37
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411442"
---
# <a name="import-from-web-url-via-http-module"></a>通过 HTTP 模块从 Web URL 导入

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块从公共网页以便在机器学习实验中使用读取数据。

以下限制适用于在网页上发布的数据：

- 数据必须采用受支持的格式之一：CSV、 TSV、 ARFF 或 SvmLight。 其他数据会导致错误。
- 无身份验证是需要或支持。 数据必须是公开提供。 

有两种方法来获取数据： 使用向导来设置数据源，或手动配置。

## <a name="use-the-data-import-wizard"></a>使用数据导入向导

1. 添加**导入数据**模块在试验。 您可以找到该模块在界面中，在**数据输入和输出**类别。

2. 单击**启动导入数据向导**并选择通过 HTTP 的 Web URL。

3. 在 URL 中，粘贴并选择一个数据格式。

4. 当配置已完成。

若要编辑现有的数据连接，请再次启动向导。 该向导，以便无需从头开始重新加载所有以前的配置详细信息

## <a name="manually-set-properties-in-the-import-data-module"></a>在导入数据模块中手动设置属性

以下步骤介绍如何手动配置导入源。

1. 添加[导入数据](import-data.md)模块在试验。 您可以找到该模块在界面中，在**数据输入和输出**类别。

2. 有关**数据源**，选择**通过 HTTP 的 Web URL**。

3. 有关**URL**，键入或粘贴包含你想要加载的数据的页的完整 URL。

    该 URL 应包括站点 URL 和带有文件名和扩展名，到包含要加载的数据的页面的完整路径。

    例如，以下页包含的机器学习的加利福尼亚大学欧文分校的存储库中的鸢尾花数据集：

    `https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data`

4. 有关**数据格式**，选择其中一个受支持的数据格式从列表中。

    我们建议始终检查数据预先确定的格式。 UC Irvine 页使用 CSV 格式。 其他受支持的数据格式为 TSV、 ARFF 和 SvmLight。

5. 如果数据采用 CSV 或 TSV 格式，请使用**文件有标题行**选项以指示源数据是否包含标题行。 标题行用于分配列名。

6. 选择**使用缓存结果**选项如果认为不太多，更改的数据，或者如果你想要避免重新加载的数据每次运行试验。

    选中此选项后，实验将加载该模块在运行时，并从此使用数据集的缓存的版本的数据第一个时间。

    如果你想要重新加载在每次迭代实验数据集的数据集，请取消选中**使用缓存结果**选项。 如果有任何更改到的参数，结果会还会重新加载[导入数据](import-data.md)。

7. 运行试验。

## <a name="results"></a>结果

完成后，单击输出数据集，然后选择**可视化**若要查看是否数据已成功导入。


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 