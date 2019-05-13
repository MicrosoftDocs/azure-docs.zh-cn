---
title: 导出数据：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中导出数据模块将结果、 中间数据和处理的数据将试验保存到外部 Azure 机器学习的云存储目标。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: c3744803f172edf9fbf2556a12677e8faef370c2
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028315"
---
# <a name="export-data-module"></a>导出数据模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块可将结果、 中间数据和处理的数据将试验保存到外部 Azure 机器学习的云存储目标。

此模块支持导出或将数据保存到以下云数据服务：


- **将导出到 Azure Blob 存储**:将数据保存到 Azure 中的 Blob 服务。 可以公开共享 Blob 服务中的数据，或将其保存在受保护的应用程序数据存储中。

  
## <a name="how-to-configure-export-data"></a>如何配置导出数据

1. 添加**导出数据**界面中进行实验的模块。 您可以找到此模块中的**输入和输出**类别。

2. 连接**导出数据**你想要导出的模块所包含的数据。

3. 双击**导出数据**以打开**属性**窗格。

4. 有关**数据目标**，选择将保存你的数据的位置的云存储的类型。 如果对此选项进行任何更改，会重置所有其他属性。 因此请确保首先选择此选项 ！

5. 提供访问指定的存储帐户所需的帐户名称和身份验证方法。

    **将导出到 Azure Blob 存储**个人预览版中是唯一的选项。 下面显示了如何设置该模块。
    1. Azure blob 服务是用于存储大量数据，包括二进制数据。 有两种类型的 blob 存储： 公共 blob 和 blob，需要登录凭据。

    2. 有关**身份验证类型**，选择**公共 (SAS)** 如果知道存储是否支持通过 SAS URL 的访问权限。

          SAS URL 是一种特殊的 URL，可以生成使用 Azure 存储的实用程序，并可仅在有限的时间。  它包含用于身份验证和下载所需的所有信息。

        有关**URI**，键入或粘贴用于定义的帐户和公共 blob 的完整 URI。

        有关文件格式，支持 CSV 和 TSV。

    3. 对于专用帐户，请选择**帐户**，并提供帐户名和帐户密钥，以便可以将实验写入到存储帐户。

         - **帐户名**：键入或粘贴你想要将数据保存的帐户的名称。 例如，如果存储帐户的完整 URL 为`http://myshared.blob.core.windows.net`，你需要键入`myshared`。

        - **帐户密钥**：粘贴与帐户相关联的存储访问密钥。

        -  **容器、 目录或 blob 路径**:键入将存储导出的数据的 blob 的名称。 例如，若要将实验的结果保存到新 blob 名为**results01.csv**容器中**预测**帐户中名为**mymldata**的完整 URLblob 是`http://mymldata.blob.core.windows.net/predictions/results01.csv`。

            因此，在该字段**容器、 目录或 blob 路径**、 将指定的容器和 blob 名称后面： `predictions/results01.csv`

        - 如果指定的名称的 blob，尚不存在，Azure 将为您创建 blob。

       -  写入到现有 blob 时，可以指定将属性设置为覆盖 blob 的当前内容**Azure blob 存储写入模式**。 默认情况下，此属性设置为**错误**，这意味着只要找到具有相同名称的现有 blob 文件，将引发错误。


    4. 有关**blob 文件的文件格式**，选择应在其中存储数据的格式。

        - **CSV**:以逗号分隔值 (CSV) 是默认存储格式。 若要导出的数据的列标题，请选择选项，**写入 blob 标题行**。  以逗号分隔格式在 Azure 机器学习中使用的详细信息，请参阅[转换为 CSV](./convert-to-csv.md)。

        - **TSV**:制表符分隔值 (TSV) 格式是与许多机器学习工具兼容。 若要导出的数据的列标题，请选择选项，**写入 blob 标题行**。  

 
    5. **使用缓存的结果**:如果你想要避免重写结果保存到 blob 文件每次运行实验，请选择此选项。 如果不有任何其他更改到模块参数，实验将结果写入只在首次运行模块时，或者在存在对数据的更改。

    6. 运行试验。

## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 