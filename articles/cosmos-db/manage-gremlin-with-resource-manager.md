---
title: Azure Cosmos DB Gremlin api 的 azure 资源管理器模板
description: 使用 Azure 资源管理器模板来创建和配置 Azure Cosmos DB Gremlin API。
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: d1928606a22eba180ebd3f1e979362e75edaf2d7
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077780"
---
# <a name="create-azure-cosmos-db-gremlin-api-resources-from-a-resource-manager-template"></a>从资源管理器模板创建 Azure Cosmos DB Gremlin API 资源

了解如何创建使用 Azure 资源管理器模板 Azure Cosmos DB Gremlin API 资源。 下面的示例创建 Azure Cosmos DB Gremlin API 从[Azure 快速入门模板](https://aka.ms/gremlin-arm-qs)。 此模板会共享在数据库级别的 400 RU/s 吞吐量的两个图表使用 Gremlin api 创建 Azure Cosmos 帐户。

下面是该模板的副本：

[!code-json[create-cosmos-gremlin](~/quickstart-templates/101-cosmosdb-gremlin/azuredeploy.json)]

## <a name="deploy-with-azure-cli"></a>使用 Azure CLI 进行部署

若要部署使用 Azure CLI 资源管理器模板**副本**脚本，并选择**试试**打开 Azure Cloud shell。 若要粘贴脚本，请右键单击 shell，然后选择“粘贴”：

```azurecli-interactive

read -p 'Enter the Resource Group name: ' resourceGroupName
read -p 'Enter the location (i.e. westus2): ' location
read -p 'Enter the account name: ' accountName
read -p 'Enter the primary region (i.e. westus2): ' primaryRegion
read -p 'Enter the secondary region (i.e. eastus2): ' secondaryRegion
read -p 'Enter the database name: ' databaseName
read -p 'Enter the first graph name: ' graph1Name
read -p 'Enter the second graph name: ' graph2Name

az group create --name $resourceGroupName --location $location
az group deployment create --resource-group $resourceGroupName \
   --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-cosmosdb-gremlin/azuredeploy.json \
   --parameters accountName=$accountName primaryRegion=$primaryRegion secondaryRegion=$secondaryRegion databaseName=$databaseName \
   graph1Name=$graph1Name graph2Name=$graph2Name

az cosmosdb show --resource-group $resourceGroupName --name accountName --output tsv
```

`az cosmosdb show`命令显示新创建的 Azure Cosmos 帐户后已预配。 如果选择而不是使用 CloudShell 中使用本地安装的 Azure CLI 版本，请参阅[Azure 命令行接口 (CLI)](/cli/azure/)一文。

在上一示例中，已引用存储在 GitHub 中的模板。 此外可以将模板下载到本地计算机或创建新的模板并指定使用的本地路径`--template-file`参数。

## <a name="next-steps"></a>后续步骤

下面是一些其他资源：

- [Azure 资源管理器文档](/azure/azure-resource-manager/)
- [Azure Cosmos DB 资源提供程序架构](/azure/templates/microsoft.documentdb/allversions)
- [Azure Cosmos DB 快速入门模板](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.DocumentDB&pageNumber=1&sort=Popular)
- [排查常见的 Azure 资源管理器部署错误](../azure-resource-manager/resource-manager-common-deployment-errors.md)