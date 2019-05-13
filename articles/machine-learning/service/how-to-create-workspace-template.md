---
title: 使用 Azure 资源管理器模板来创建工作区
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 资源管理器模板创建新的 Azure 机器学习服务工作区。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: Blackmist
ms.date: 04/16/2019
ms.custom: seoapril2019
ms.openlocfilehash: abe497ed96515e8194fb2ddefd8e7f4cb9908758
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205128"
---
# <a name="use-an-azure-resource-manager-template-to-create-a-workspace-for-azure-machine-learning-service"></a>使用 Azure 资源管理器模板来创建用于 Azure 机器学习服务的工作区

本文介绍几种使用 Azure 资源管理器模板创建 Azure 机器学习服务工作区的方法。 使用资源管理器模板可以轻松地通过单个协调操作创建资源。 模板是一个 JSON 文档，定义部署所需的资源。 它还可以指定部署参数。 使用模板时，参数用于提供输入值。

有关详细信息，请参阅[使用 Azure Resource Manager 模板部署应用程序](../../azure-resource-manager/resource-group-template-deploy.md)。

## <a name="prerequisites"></a>必备组件

* 一个 **Azure 订阅**。 如果没有订阅，可试用 [Azure 机器学习服务免费版或付费版](https://aka.ms/AMLFree)。

* 若要在 CLI 中使用模板，需要安装 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azps-1.2.0) 或 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。

## <a name="resource-manager-template"></a>资源管理器模板

可以使用以下资源管理器模板创建 Azure 机器学习服务工作区和关联的 Azure 资源：

[!code-json[create-azure-machine-learning-service-workspace](~/quickstart-templates/101-machine-learning-create/azuredeploy.json)]

此模板创建以下 Azure 服务：

* Azure 资源组
* Azure 存储空间帐户
* Azure 密钥保管库
* Azure Application Insights
* Azure 容器注册表
* Azure 机器学习工作区

资源组是保存服务的容器。 Azure 机器学习工作区需要多种服务。

示例模板包含两个参数：

* **位置**：将在其中创建资源组和服务。

    模板将使用你为大多数资源选择的位置。 例外的情况是 Application Insights 服务，它不像其他所有服务一样在所有位置都可用。 如果选择了 Application Insights 服务不可用的位置，将在美国中南部位置创建该服务。

* **工作区名称**：Azure 机器学习工作区的友好名称。

    其他服务的名称将随机生成。

有关模板的详细信息，请参阅以下文章：

* [创作 Azure Resource Manager 模板](../../azure-resource-manager/resource-group-authoring-templates.md)
* [使用 Azure Resource Manager 模板部署应用程序](../../azure-resource-manager/resource-group-template-deploy.md)
* [Microsoft.MachineLearningServices 资源类型](https://docs.microsoft.com/azure/templates/microsoft.machinelearningservices/allversions)

## <a name="use-the-azure-portal"></a>使用 Azure 门户

1. 遵循[从自定义模板部署资源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal#deploy-resources-from-custom-template)中的步骤。 显示“编辑模板”屏幕后，请粘贴本文档中所述的模板。
1. 选择“保存”以使用该模板。 提供以下信息并同意列出的条款和条件：

   * 订阅：选择用于这些资源的 Azure 订阅。
   * 资源组：选择或创建一个用于包含服务的资源组。
   * 工作区名称：要创建的 Azure 机器学习工作区所用的名称。 工作区名称的长度必须为 3 到 33 个字符。 只能包含字母数字字符和“-”。
   * 位置：选择要在其中创建资源的位置。

     ![Azure 门户中的模板参数](media/how-to-create-workspace-template/template-parameters.png)

有关详细信息，请参阅[从自定义模板部署资源](../../azure-resource-manager/resource-group-template-deploy-portal.md#deploy-resources-from-custom-template)。

## <a name="use-azure-powershell"></a>使用 Azure PowerShell

此示例假设已将模板保存到当前目录中名为 `azuredeploy.json` 的文件：

```powershell
New-AzResourceGroup -Name examplegroup -Location "East US"
new-azresourcegroupdeployment -name exampledeployment `
  -resourcegroupname examplegroup -location "East US" `
  -templatefile .\azuredeploy.json -workspaceName "exampleworkspace"
```

有关详细信息，请参阅[使用资源管理器模板和 Azure PowerShell 部署资源](../../azure-resource-manager/resource-group-template-deploy.md)和[使用 SAS 令牌和 Azure PowerShell 部署专用资源管理器模板](../../azure-resource-manager/resource-manager-powershell-sas-token.md)。

## <a name="use-azure-cli"></a>使用 Azure CLI

此示例假设已将模板保存到当前目录中名为 `azuredeploy.json` 的文件：

```azurecli-interactive
az group create --name examplegroup --location "East US"
az group deployment create \
  --name exampledeployment \
  --resource-group examplegroup \
  --template-file azuredeploy.json \
  --parameters workspaceName=exampleworkspace
```

有关详细信息，请参阅[使用资源管理器模板和 Azure CLI 部署资源](../../azure-resource-manager/resource-group-template-deploy-cli.md)和[使用 SAS 令牌和 Azure CLI 部署专用资源管理器模板](../../azure-resource-manager/resource-manager-cli-sas-token.md)。

## <a name="next-steps"></a>后续步骤

* [使用资源管理器模板和资源管理器 REST API 部署资源](../../azure-resource-manager/resource-group-template-deploy-rest.md)。
* [通过 Visual Studio 创建和部署 Azure 资源组](../../azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)。
