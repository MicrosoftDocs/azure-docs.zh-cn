---
title: 创建存储帐户 - Azure 存储
description: 本操作指南文章，您了解如何创建使用 Azure 门户、 Azure PowerShell 或 Azure CLI 的存储帐户。 Azure 存储帐户提供 Microsoft Azure 中的唯一命名空间，用于在 Azure 存储中存储和访问创建的数据对象。
services: storage
author: tamram
ms.custom: mvc
ms.service: storage
ms.topic: article
ms.date: 05/06/2019
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 8375f4c54dc436ecf0694ec5f629c81d3591594d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234181"
---
# <a name="create-a-storage-account"></a>创建存储帐户

Azure 存储帐户包含所有 Azure 存储数据对象：Blob、文件、队列、表和磁盘。 存储帐户为你的 Azure 存储数据位于可从任意位置在世界通过 HTTP 或 HTTPS 提供唯一的命名空间。 Azure 存储帐户中的数据是持久和高度可用、 安全且高度可缩放。

本操作指南文章介绍如何创建存储帐户使用[Azure 门户](https://portal.azure.com/)， [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)， [Azure CLI](https://docs.microsoft.com/cli/azure?view=azure-cli-latest)，或[Azure 资源管理器模板](../../azure-resource-manager/resource-group-overview.md)。  

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>必备组件

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。

# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)

无。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

本操作指南文章需要 Azure PowerShell 模块 Az 0.7 或更高版本。 运行 `Get-Module -ListAvailable Az` 即可查找当前版本。 如果需要进行安装或升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

可以登录到 Azure，然后采用以下两种方式之一运行 Azure CLI 命令：

- Azure Cloud Shell 中，可以运行从 Azure 门户中的 CLI 命令。
- 可以安装 CLI 并在本地运行 CLI 命令。

### <a name="use-azure-cloud-shell"></a>使用 Azure Cloud Shell

Azure Cloud Shell 是可直接在 Azure 门户中运行的免费 Bash shell。 Azure CLI 是预安装和配置为使用与你的帐户。 单击**Cloud Shell**在 Azure 门户的右上部分的菜单上的按钮：

[![Cloud Shell](./media/storage-quickstart-create-account/cloud-shell-menu.png)](https://portal.azure.com)

该按钮会启动交互式 shell，可用于运行本操作指南文章中所述的步骤：

[![显示门户中 Cloud Shell 窗口的屏幕截图](./media/storage-quickstart-create-account/cloud-shell.png)](https://portal.azure.com)

### <a name="install-the-cli-locally"></a>在本地安装 CLI

也可在本地安装和使用 Azure CLI。 本操作指南文章要求运行 Azure CLI 2.0.4 或更高版本。 运行 `az --version` 即可查找版本。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。 

# <a name="templatetabtemplate"></a>[模板](#tab/template)

无。

---

## <a name="sign-in-to-azure"></a>登录 Azure

# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)

登录到 [Azure 门户](https://portal.azure.com)。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

使用 `Connect-AzAccount` 命令登录到 Azure 订阅，然后按照屏幕上的说明进行身份验证。

```powershell
Connect-AzAccount
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

若要启动 Azure Cloud Shell，请登录到[Azure 门户](https://portal.azure.com)。

若要登录到本地安装的 cli，运行[az 登录](/cli/azure/reference-index#az-login)命令：

```cli
az login
```

# <a name="templatetabtemplate"></a>[模板](#tab/template)

不适用

---

## <a name="create-a-storage-account"></a>创建存储帐户

现在可以创建存储帐户。

每个存储帐户都必须属于 Azure 资源组。 资源组是对 Azure 资源进行分组的逻辑容器。 在创建存储帐户时，可以选择创建新的资源组，也可以使用现有资源组。 本文介绍如何创建新的资源组。

可以使用常规用途 v2 存储帐户访问所有 Azure 存储服务：Blob、文件、队列、表和磁盘。 此处所述的步骤创建常规用途 v2 存储帐户，但若要创建任何类型的存储帐户的步骤是类似。

# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

首先，使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 命令，通过 PowerShell 创建新的资源组：

```powershell
# put resource group in a variable so you can use the same group name going forward,
# without hard-coding it repeatedly
$resourceGroup = "storage-resource-group"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

如果不确定为 `-Location` 参数指定哪个区域，可使用 [Get-AzLocation](/powershell/module/az.resources/get-azlocation) 命令检索订阅支持的区域的列表：

```powershell
Get-AzLocation | select Location
$location = "westus"
```

接下来，使用与读取访问异地冗余存储 (RA-GRS) 创建常规用途 v2 存储帐户[新建 AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount)命令。 请记住，你的存储帐户名称必须在 Azure 中是唯一，因此在方括号中的占位符值替换为你自己的唯一值：

```powershell
New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name <account-name> `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2 
```

若要使用不同的复制选项创建常规用途 v2 存储帐户，请替换为下表中所需的值**SkuName**参数。

|复制选项  |SkuName 参数  |
|---------|---------|
|本地冗余存储 (LRS)     |Standard_LRS         |
|区域冗余存储(ZRS)     |Standard_ZRS         |
|异地冗余存储(GRS)     |Standard_GRS         |
|读取访问异地冗余存储 (GRS)     |Standard_RAGRS         |

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

首先，使用 [az group create](/cli/azure/group#az_group_create) 命令，通过 Azure CLI 创建新的资源组。

```azurecli-interactive
az group create \
    --name storage-resource-group \
    --location westus
```

如果不确定为 `--location` 参数指定哪个区域，可使用 [az account list-locations](/cli/azure/account#az_account_list) 命令检索订阅支持的区域的列表。

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

接下来，使用读取访问异地冗余存储与创建常规用途 v2 存储帐户[az 存储帐户创建](/cli/azure/storage/account#az_storage_account_create)命令。 请记住，你的存储帐户名称必须在 Azure 中是唯一，因此在方括号中的占位符值替换为你自己的唯一值：

```azurecli-interactive
az storage account create \
    --name <account-name> \
    --resource-group storage-resource-group \
    --location westus \
    --sku Standard_RAGRS \
    --kind StorageV2
```

若要使用不同的复制选项创建常规用途 v2 存储帐户，请替换为下表中所需的值**sku**参数。

|复制选项  |sku 参数  |
|---------|---------|
|本地冗余存储 (LRS)     |Standard_LRS         |
|区域冗余存储(ZRS)     |Standard_ZRS         |
|异地冗余存储(GRS)     |Standard_GRS         |
|读取访问异地冗余存储 (GRS)     |Standard_RAGRS         |

# <a name="templatetabtemplate"></a>[模板](#tab/template)

可以使用 Azure Powershell 或 Azure CLI 来部署资源管理器模板以创建存储帐户。 在本操作指南文章中所用的模板是从[Azure 资源管理器快速入门模板](https://azure.microsoft.com/resources/templates/101-storage-account-create/)。 若要运行脚本，请选择“试用”打开 Azure Cloud shell。 若要粘贴脚本，请右键单击 shell，然后选择“粘贴”。

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group create --name $resourceGroupName --location "$location" &&
az group deployment create --resource-group $resourceGroupName --template-file "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

若要了解如何创建模板，请参阅：

- [Azure 资源管理器文档](/azure/azure-resource-manager/)。
- [存储帐户模板参考](/azure/templates/microsoft.storage/allversions)。
- [其他存储帐户模板示例](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Storage)。

---

有关可用的复制选项的详细信息，请参阅[存储复制选项](storage-redundancy.md)。

## <a name="clean-up-resources"></a>清理资源

如果你想要清理本操作指南文章创建的资源，则可以删除资源组。 删除资源组也会删除相关联的存储帐户，以及与资源组相关联的任何其他资源。

# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)

若要使用 Azure 门户删除资源组，请执行以下操作：

1. 在 Azure 门户中展开左侧的菜单，打开服务菜单，然后选择“资源组”以显示资源组的列表。
2. 找到要删除的资源组，右键单击列表右侧的“更多”按钮 (**...**)。
3. 选择“删除资源组”并进行确认。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

若要删除资源组及其关联的资源（包括新的存储帐户），请使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) 命令：

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

若要删除资源组及其关联的资源（包括新的存储帐户），请使用 [az group delete](/cli/azure/group#az_group_delete) 命令。

```azurecli-interactive
az group delete --name storage-resource-group
```

# <a name="templatetabtemplate"></a>[模板](#tab/template)

若要删除资源组及其关联的资源（包括新的存储帐户），请使用 Azure PowerShell 或 Azure CLI。

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>后续步骤

在本操作指南文章，已创建常规用途 v2 的标准存储帐户。 若要了解如何上传和下载 blob，与你的存储帐户，继续到 Blob 存储快速入门之一。

# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)

> [!div class="nextstepaction"]
> [通过 Azure 门户使用 Blob](../blobs/storage-quickstart-blobs-portal.md)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

> [!div class="nextstepaction"]
> [通过 PowerShell 使用 Blob](../blobs/storage-quickstart-blobs-powershell.md)

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!div class="nextstepaction"]
> [通过 Azure CLI 使用 Blob](../blobs/storage-quickstart-blobs-cli.md)

# <a name="templatetabtemplate"></a>[模板](#tab/template)

> [!div class="nextstepaction"]
> [通过 Azure 门户使用 Blob](../blobs/storage-quickstart-blobs-portal.md)

---
