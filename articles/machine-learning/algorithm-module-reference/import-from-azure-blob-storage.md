---
title: 从 Azure Blob 存储导入：模块参考
titleSuffix: Azure Machine Learning service
description: 了解本主题介绍如何使用 Azure Blob 存储模块中的导入 Azure 机器学习服务中从 Azure blob 存储读取数据，以便可以在机器学习实验中使用数据。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 4ac98516c1a326e1ede09bbb9660113ffd0642a0
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029680"
---
# <a name="import-from-azure-blob-storage-module"></a>从 Azure Blob 存储模块导入

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块从 Azure blob 存储读取数据，以便可以在机器学习实验中使用数据。  

Azure Blob 服务是用于存储大量数据，包括二进制数据。 通过使用 HTTP 或 HTTPS，可以从任何位置访问 azure blob。 具体取决于 blob 存储的类型可能需要使用身份验证。 

- 任何人，或具有 SAS URL 的用户可以访问公共 blob。
- 专用 blob 需要一个登录名和凭据。

从 blob 存储导入需要的数据存储在使用的 blob**块 blob**格式。 存储 blob 中的文件必须使用逗号分隔 (CSV) 或制表符分隔 (TSV) 格式。 当您读取该文件时，记录和任何适用的属性标题将作为行加载到内存中作为数据集。


我们强烈建议在分析分析你的数据导入前，请确保该架构为预期结果。 导入进程将扫描一定数量的头行，以确定架构，但更高版本的行可能包含额外的列或导致错误的数据。



## <a name="manually-set-properties-in-the-import-data-module"></a>在导入数据模块中手动设置属性

以下步骤介绍如何手动配置导入源。

1. 添加**导入数据**模块在试验。 您可以找到此模块在界面中，在**数据输入和输出**

2. 有关**数据源**，选择**Azure Blob 存储**。

3. 有关**身份验证类型**，选择**公共 (SAS URL)** 如果知道已作为公共数据源中提供的信息。 SAS URL 是你可以通过使用 Azure 存储实用程序生成的公共访问权限的时间绑定 URL。

    否则，请选择**帐户**。

4. 如果你的数据在**公共**blob 可通过使用 SAS URL，则不需其他凭据，因为 URL 字符串包含下载和身份验证所需的所有信息。

    在中**URI**字段中，键入或粘贴用于定义的帐户和公共 blob 的完整 URI。



5. 如果你的数据处于**专用**帐户，必须提供凭据包括帐户名称和密钥。

    - 有关**帐户名**，键入或粘贴包含你想要访问的 blob 的帐户的名称。

        例如，如果存储帐户的完整 URL 为`http://myshared.blob.core.windows.net`，你需要键入`myshared`。

    - 有关**帐户密钥**，粘贴与帐户相关联的存储访问密钥。

        如果不知道访问密钥，请参阅的部分，这篇文章中的"管理你的 Azure 存储帐户":[关于 Azure 存储帐户](https://docs.microsoft.com/azure/storage/storage-create-storage-account)。

6. 有关**容器、 目录或 blob 路径**，键入你想要检索的特定 blob 的名称。

    例如，如果名为的文件上传**data01.csv**到容器**trainingdata**帐户中名为**mymldata**，该文件的完整 URL 将是： `http://mymldata.blob.core.windows.net/trainingdata/data01.txt`.

    因此，在该字段**容器、 目录或 blob 路径**，你需要键入： `trainingdata/data01.csv`

    若要导入多个文件，可以使用通配符字符`*`（星号） 或`?`（问号）。

    例如，假设容器`trainingdata`包含多个文件的兼容的格式，您可以使用以下规范读取开头的所有文件`data`，然后将它们连接成单个数据集：

    `trainingdata/data*.csv`

    容器名称中，不能使用通配符。 如果你需要从多个容器导入文件，使用一个单独的实例**导入数据**用于每个容器，然后合并数据集使用的模块[添加行](./add-rows.md)模块。

    > [!NOTE]
    > 如果选择了选项，**使用缓存结果**，对容器中的文件进行任何更改不会触发在实验中的数据刷新。

7. 有关**Blob 文件格式**，选择一个选项，以便 Azure 机器学习可以适当地处理数据指示 blob 中存储的数据的格式。 支持以下格式：

    - **CSV**:以逗号分隔值 (CSV) 是用于导出和导入 Azure 机器学习中的文件的默认存储格式。 如果数据已经包含一个标题行，请务必选择选项，**文件有标题行**，或该标头将被视为数据行。

       

    - **TSV**:制表符分隔值 (TSV) 是由许多机器学习工具所使用的格式。 如果数据已经包含一个标题行，请务必选择选项，**文件有标题行**，或该标头将被视为数据行。

       

    - **ARFF**:此格式支持导入 Weka 工具集使用的格式中的文件。 

   

8. 运行试验。


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 