---
title: 在 Azure 资源管理器模板中使用条件 | Microsoft Docs
description: 了解如何根据条件部署 Azure 资源。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 03/04/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: ad7c87161c550c4728978e9c975252cab34f76ec
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60389746"
---
# <a name="tutorial-use-condition-in-azure-resource-manager-templates"></a>教程：在 Azure 资源管理器模板中使用条件

了解如何根据条件部署 Azure 资源。

[设置资源部署顺序](./resource-manager-tutorial-create-templates-with-dependent-resources.md)教程介绍如何创建虚拟机、虚拟网络以及其他一些依赖资源（包括存储帐户）。 无需每次都创建新的存储帐户，可让用户选择是创建新的存储帐户还是使用现有的存储帐户。 为实现此目的，需定义附加的参数。 如果参数值为“new”，则创建新存储帐户。

![资源管理器模板使用条件关系图](./media/resource-manager-tutorial-use-conditions/resource-manager-template-use-condition-diagram.png)

本教程涵盖以下任务：

> [!div class="checklist"]
> * 打开快速入门模板
> * 修改模板
> * 部署模板
> * 清理资源

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含[资源管理器工具扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)的 [Visual Studio Code](https://code.visualstudio.com/)。
* 若要提高安全性，请使用为虚拟机管理员帐户生成的密码。 以下是密码生成示例：

    ```azurecli-interactive
    openssl rand -base64 32
    ```
    Azure Key Vault 旨在保护加密密钥和其他机密。 有关详细信息，请参阅[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md)。 我们还建议你每三个月更新一次密码。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

Azure 快速入门模板是资源管理器模板的存储库。 无需从头开始创建模板，只需找到一个示例模板并对其自定义即可。 本教程中使用的模板称为[部署简单的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。
4. 有五个通过此模板定义的资源：

   * `Microsoft.Storage/storageAccounts`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/Microsoft.Storage/storageAccounts)。
   * `Microsoft.Network/publicIPAddresses`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/publicipaddresses)。
   * `Microsoft.Network/virtualNetworks`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks)。
   * `Microsoft.Network/networkInterfaces`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/networkinterfaces)。
   * `Microsoft.Compute/virtualMachines`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines)。

     在自定义模板之前，不妨对其进行一些基本的了解。
5. 选择“文件”>“另存为”，将该文件的副本保存到名为 **azuredeploy.json** 的本地计算机。

## <a name="modify-the-template"></a>修改模板

对现有模板进行两项更改：

* 添加存储帐户名称参数。 用户可以指定新的存储帐户名称或现有的存储帐户名称。
* 添加名为 **newOrExisting** 的新参数。 部署使用此参数来确定是要创建新存储帐户还是使用现有的存储帐户。

下面是进行更改的过程：

1. 在 Visual Studio Code 中打开 **azuredeploy.json**。
2. 在整个模板中，将 **variables('storageAccountName')** 替换为 **parameters('storageAccountName')**。  **variables('storageAccountName')** 有三种外观。
3. 删除以下变量定义：

    ```json
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",
    ```
4. 将以下两个参数添加到模板：

    ```json
    "storageAccountName": {
      "type": "string"
    },
    "newOrExisting": {
      "type": "string", 
      "allowedValues": [
        "new", 
        "existing"
      ]
    },
    ```
    更新的参数定义如下所示：

    ![在资源管理器中使用条件](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

5. 将以下行添加到存储帐户定义的开头。

    ```json
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    ```

    该条件检查名为 **newOrExisting** 的参数的值。 如果参数值为 **new**，则部署将创建存储帐户。

    更新的存储帐户定义如下所示：

    ![在资源管理器中使用条件](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)
6. 将 **storageUri** 更新为以下值：

    ```json
    "storageUri": "[concat('https://', parameters('storageAccountName'), '.blob.core.windows.net')]"
    ```

    如果使用另一资源组中的现有存储帐户，则此更改是必需的。

7. 保存更改。

## <a name="deploy-the-template"></a>部署模板

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

遵照[部署模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)中的说明部署模板。

使用 Azure PowerShell 部署模板时，需要指定一个附加参数。 若要提高安全性，请使用为虚拟机管理员帐户生成的密码。 请参阅[先决条件](#prerequisites)。

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"
$newOrExisting = Read-Host -Prompt "Create new or use existing (Enter new or existing)"
$location = Read-Host -Prompt "Enter the Azure location (i.e. centralus)"
$vmAdmin = Read-Host -Prompt "Enter the admin username"
$vmPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
$dnsLabelPrefix = Read-Host -Prompt "Enter the DNS Label prefix"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUsername $vmAdmin `
    -adminPassword $vmPassword `
    -dnsLabelPrefix $dnsLabelPrefix `
    -storageAccountName $storageAccountName `
    -newOrExisting $newOrExisting `
    -TemplateFile "$HOME/azuredeploy.json"
```

> [!NOTE]
> 如果 **newOrExisting** 为 **new**，但具有指定存储帐户名称的存储帐户已存在，则部署将会失败。

请尝试创建 **newOrExisting** 设置为“existing”的另一个部署，并指定现有存储帐户。 若要提前创建存储帐户，请参阅[创建存储帐户](../storage/common/storage-quickstart-create-account.md)。

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。  应会看到，该资源组中总共有六个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

在本教程中，我们开发了一个允许用户选择创建新存储帐户或使用现有存储帐户的模板。 若要了解如何从 Azure Key Vault 检索机密并在模板部署中使用这些机密作为密码，请参阅：

> [!div class="nextstepaction"]
> [在模板部署中集成 Key Vault](./resource-manager-tutorial-use-key-vault.md)
