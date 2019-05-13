---
title: Azure PowerShell 脚本-删除 Azure Cosmos 帐户中的数据库
description: Azure PowerShell 脚本示例-删除 Azure Cosmos 帐户中的数据库
author: markjbrown
ms.service: cosmos-db
ms.topic: samples
ms.date: 05/08/2019
ms.author: mjbrown
ms.openlocfilehash: a8ecdb7456b13b21f814a3c2eda924aa9b3f48fa
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077720"
---
# <a name="delete-a-database-in-an-azure-cosmos-account-in-an-azure-cosmos-account-using-powershell"></a>删除 Azure Cosmos 帐户中使用 PowerShell 的 Azure Cosmos 帐户数据库

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-database-delete.ps1 "Delete a database in an Azure Cosmos account")]

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
|**Azure 资源**| |
| [New-AzResource](https://docs.microsoft.com/powershell/module/az.resources/new-azresource) | 创建资源。 |
| [Get-AzResource](https://docs.microsoft.com/powershell/module/az.resources/get-azresource) | 获取资源。 |
| [Set-AzResource](https://docs.microsoft.com/powershell/module/az.resources/set-azresource) | 更新资源。 |
| [Invoke-AzResourceAction](https://docs.microsoft.com/powershell/module/az.resources/invoke-azresourceaction) | 调用资源的操作。 |
| [Remove-AzResource](https://docs.microsoft.com/powershell/module/az.resources/remove-azresource) | 删除资源。 |
|**Azure 资源组**| |
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) | 创建用于存储所有资源的资源组。 |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

可以在 [Azure Cosmos DB PowerShell 脚本](../../../powershell-samples.md)中找到其他 Azure Cosmos DB PowerShell 脚本示例。