---
title: 在工作区中创建和管理文件
titleSuffix: Azure Machine Learning
description: 了解如何在 Azure 机器学习 studio 中的工作区中创建和管理文件。
services: machine-learning
author: abeomor
ms.author: osomorog
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 02/05/2021
ms.openlocfilehash: 474b3123513e4b8acf19ba9cdb42c3384ea3ced2
ms.sourcegitcommit: 24f30b1e8bb797e1609b1c8300871d2391a59ac2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/10/2021
ms.locfileid: "100101220"
---
# <a name="how-to-create-and-manage-files-in-your-workspace"></a>如何在工作区中创建和管理文件

了解如何创建和管理 Azure 机器学习工作区中的文件。  这些文件存储在默认的工作区存储区中。 文件和文件夹可与具有工作区读取访问权限的任何其他人共享，并可用于工作区中的任何计算实例。

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://aka.ms/AMLFree)。
* 机器学习工作区。 请参阅[创建 Azure 机器学习工作区](how-to-manage-workspace.md)。

## <a name="create-files"></a><a name="create"></a> 创建文件

若要在默认文件夹中创建新文件 (`Users > yourname`) ：

1. 在 [Azure 机器学习工作室](https://ml.azure.com)中打开工作区。
1. 在左侧选择“笔记本”。
1. 选择 **+** 映像。
1. 选择 "  **创建新文件** 映像"。

    :::image type="content" source="media/how-to-run-jupyter-notebooks/create-new-file.png" alt-text="新建文件":::

1. 为文件命名。
1. 选择文件类型。
1. 选择“创建”  。

笔记本和大多数文本文件类型可在“预览”部分显示。  大多数其他文件类型没有预览。

在其他文件夹中创建新文件：
1. 选择 "..."位于文件夹结尾
1. 选择“创建新文件”。

> [!IMPORTANT]
> 笔记本和脚本中的内容可能会从会话中读取数据，并在组织不在 Azure 中的情况下访问数据。  仅从受信任的源加载文件。 有关详细信息，请参阅[安全代码最佳做法](concept-secure-code-best-practice.md#azure-ml-studio-notebooks)。

## <a name="manage-files-with-git"></a>用 Git 管理文件

[使用计算实例终端](how-to-access-terminal.md#git) 克隆和管理 Git 存储库。

## <a name="clone-samples"></a>克隆示例

你的工作区包含一个 **示例笔记本** 文件夹，其中包含的笔记本旨在帮助你探索 SDK，并作为你自己的机器学习项目的示例。   将这些笔记本独立到你自己的文件夹，以运行和编辑它们。  

有关示例，请参阅[教程：创建第一个 ML 试验](tutorial-1st-experiment-sdk-setup.md#azure)。

## <a name="share-files"></a>共享文件

复制并粘贴用于共享文件的 URL。  只有工作区的其他用户才能访问此 URL。  详细了解如何[授权访问工作区](how-to-assign-roles.md)。

## <a name="delete-a-file"></a>删除文件

*不能* 删除 **示例笔记本** 文件。  这些文件是工作室的一部分，每次发布新的 SDK 时都会更新这些文件。  

*可以* 通过以下任一方式删除在 **文件** 中找到的文件：

* 在工作室中，选择文件夹或文件末尾的“...”。  请确保使用支持的浏览器（Microsoft Edge、Chrome 或 Firefox）。
* [使用](how-to-access-terminal.md) 工作区中任何计算实例的终端。 文件夹 **~/cloudfiles** 映射到你的工作区存储帐户上的存储。
* 在 Jupyter 或 JupyterLab 中使用自带工具删除。
