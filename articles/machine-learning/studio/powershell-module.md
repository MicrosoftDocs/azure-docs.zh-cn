---
title: 适用于机器学习工作室的 PowerShell 模块
titleSuffix: Azure Machine Learning Studio
description: 使用 PowerShell 创建和管理 Azure 机器学习工作室工作区、实验和 Web 服务等。
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: xiaoharper
ms.author: amlstudiodocs
ms.date: 04/25/2019
ms.openlocfilehash: bee42f8a9582908963c0eef95a2fd04742cd425e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205666"
---
# <a name="powershell-modules-for-azure-machine-learning-studio"></a>Azure 机器学习工作室 PowerShell 模块

使用 PowerShell 模块，可采用编程方式管理工作室资源和资产，例如工作区、数据集和 Web 服务。

你可以使用以下三个 Powershell 模块与工作室资源进行交互：

* 于 2018 年发布的 [Azure PowerShell Az](#az-rm)，包含 AzureRM 的所有功能，尽管使用不同的 cmdlet 名称
* [AzureRM](#az-rm) 2016，替换为 PowerShell Az 发布
* 于 2016 年发布的 [Azure 机器学习 PowerShell（经典）](#classic)

尽管这些 PowerShell 模块具有一些相似之处，但每个专为特定方案。 本文介绍了 PowerShell 模块之间的差异，帮助你决定选择哪些模块。  

检查下面的[支持表](#support-table)，查看每个模块支持哪些资源。 

## <a name="az-rm"></a> Azure PowerShell Az 和 AzureRM

Az 现在是与 Azure 交互的 PowerShell 模块，它包含 AzureRM 以前的所有功能。 AzureRM 将继续接收 bug 修复，但它不会接收任何新的 cmdlet 或功能。  Az 和 AzureRM 都管理使用 Azure 资源管理器部署模型部署的解决方案。 这些资源包括工作室工作区和工作室"新"web 服务。 

PowerShell 经典可以与 Az 或 AzureRM 以覆盖这两个"新"和"经典"资源类型一起安装。 但是，不建议同时安装 Az 和 AzureRM。 若要在 Az 和 AzureRM 之间做出选择，Microsoft 建议在未来所有部署中使用 Az。  了解有关与 AzureRM 和迁移路径中的 Az [Azure PowerShell Az 简介](https://docs.microsoft.com/powershell/azure/new-azureps-module-az)。

若要开始使用 Az，请按照 [Azure Az 的安装说明](https://docs.microsoft.com/powershell/azure/install-az-ps)操作。

## <a name="classic"></a> PowerShell（经典）

通过工作室 [PowerShell 经典模块](https://aka.ms/amlps)，可让你管理使用经典部署模型部署的资源。 这些资源包括 Studio 用户资产、"经典"web 服务和"经典"web 服务终结点。

但是，Microsoft 建议使用将来的所有资源的资源管理器部署模型来简化部署和管理资源。 若要了解有关部署模型的详细信息，请参阅 [Azure 资源管理器与经典部署](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-model)一文。

若要开始使用 PowerShell（经典），请从 GitHub 下载[发行包](https://github.com/hning86/azuremlps/releases)并按照[安装说明](https://github.com/hning86/azuremlps/blob/master/README.md)执行操作。 这些说明解释了如何取消阻止已下载/解压缩的 DLL，然后将它导入到 PowerShell 环境。

PowerShell 经典可以与 Az 或 AzureRM 以覆盖这两个"新"和"经典"资源类型一起安装。

## <a name="support-table"></a> PowerShell 支持表


| | **Az** |  **PowerShell 经典** |
| --- | --- | --- |
| 创建/删除工作区 | [Resource Manager 模板](https://docs.microsoft.com/azure/machine-learning/studio/deploy-with-resource-manager-template) |  |
| 管理工作区承诺计划 | [New-AzMlCommitmentPlan](https://docs.microsoft.com/powershell/module/az.machinelearning/new-azmlcommitmentplan) | |
| 管理工作区用户 |  | [Add-AmlWorkspaceUsers](https://github.com/hning86/azuremlps#add-amlworkspaceusers)|
| 管理 Web 服务 | [New-AzMlWebService](https://docs.microsoft.com/powershell/module/az.machinelearning/new-azmlwebservice) <br>("new"web 服务)|| [New-AmlWebService](https://github.com/hning86/azuremlps#manage-classic-web-service) <br>("经典"web 服务) |
| 管理 web 服务终结点/密钥 |  [Get-AzMlWebServiceKey](https://docs.microsoft.com/powershell/module/az.machinelearning/get-azmlwebservicekey)|  [Add-AmlWebServiceEndpoint](https://github.com/hning86/azuremlps#manage-classic-web-servcie-endpoint)|
| 管理用户数据集/训练模型| | [Get-AmlDataset](https://github.com/hning86/azuremlps#manage-user-assets-dataset-trained-model-transform) |
| 管理用户试验 |  | [Start-AmlExperiment](https://github.com/hning86/azuremlps#manage-experiment) |
| 管理自定义模块 | | [New-AmlCustomModule](https://github.com/hning86/azuremlps#manage-custom-module) |


## <a name="next-steps"></a>后续步骤
查阅这些 PowerShell 模块的完整文档：
* [PowerShell 经典](https://aka.ms/amlps)
* [Azure PowerShell Az](https://docs.microsoft.com/powershell/module/az.machinelearning/#machine_learning)
