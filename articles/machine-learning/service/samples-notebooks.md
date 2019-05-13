---
title: 示例 Jupyter Notebook
titleSuffix: Azure Machine Learning service
description: 找到并使用示例 Jupyter 笔记本，以便在 Python 中探索 Azure 机器学习服务。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: sample
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.date: 12/04/2018
ms.custom: seodec18
ms.openlocfilehash: cd88fd85ce6d18287c700a54e42b6237a42ea5c9
ms.sourcegitcommit: eea74d11a6d6ea6d187e90e368e70e46b76cd2aa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035381"
---
# <a name="use-jupyter-notebooks-to-explore-azure-machine-learning-service"></a>使用 Jupyter 笔记本探索 Azure 机器学习服务

为了方便用户，我们开发了一系列 Jupyter Python 笔记本，用于探索 Azure 机器学习服务。 

了解如何将服务与有关本站点的文档配合使用，并使用这些笔记本，根据情况来自定义它们。 

使用下面的路径之一，以便运行笔记本服务器和这些示例笔记本。  待服务器运行以后，在 **tutorials** 文件夹中找到教程笔记本，或者在 **how-to-use-azureml** 文件夹中浏览不同的功能。

## <a name="a-managed-cloud-notebook-server"></a>托管云笔记本服务器

可以从你自己的基于云的笔记本服务器着手，这很容易。 当你创建此云资源时，示例笔记本和[用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 就已为你安装和配置了。  

[!INCLUDE [aml-azure-notebooks](../../../includes/aml-azure-notebooks.md)]

* 笔记本网页上提供这些示例。

## <a name="a-data-science-virtual-machine-dsvm"></a>Data Science Virtual Machine (DSVM)

已在 DSVM 上安装和配置[用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 和笔记本服务器。 

[创建 DSVM](how-to-configure-environment.md#dsvm) 以后，在 DSVM 上根据这些步骤运行笔记本。

[!INCLUDE [aml-dsvm-server](../../../includes/aml-dsvm-server.md)]

## <a name="your-own-jupyter-notebook-server"></a>你自己的 Jupyter 笔记本服务器

执行这些步骤，在计算机上创建本地 Jupyter Notebook 服务器。

[!INCLUDE [aml-your-server](../../../includes/aml-your-server.md)]

安装说明会安装运行快速入门和教程笔记本所需的程序包。  其他示例笔记本可能需要安装附加组件。  有关这些组件的详细信息，请参阅[安装适用于 Python 的 Azure 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install)。

## <a name="azure-notebooks"></a>Azure Notebook

已在 [Azure Notebooks](https://notebooks.azure.com/) 上为你安装和配置示例笔记本，以及[用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk)。 安装和未来的更新通过 Azure 服务自动管理。

使用 [Azure 门户](https://portal.azure.com)完成 Azure Notebooks 入门。  打开工作区，然后从“概览”部分选择“Azure Notebooks 入门”。

## <a name="next-steps"></a>后续步骤

+ 在此 GitHub 存储库中浏览适用于 Azure 机器学习服务的示例笔记本： https://aka.ms/aml-notebooks

试用以下教程：
+ [通过 MNIST 训练并部署图像分类模型](tutorial-train-models-with-aml.md)

+ [准备数据并使用纽约市出租车数据集通过自动化机器学习来训练回归模型](tutorial-data-prep.md)