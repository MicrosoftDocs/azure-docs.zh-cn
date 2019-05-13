---
title: 使用 Azure 资源管理器模板部署虚拟机扩展 | Microsoft Docs
description: 了解如何使用 Azure 资源管理器模板部署虚拟机扩展
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 11/13/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 5657ebb2a5b29e4ec5360480c1fef6cb92dad9c8
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60388518"
---
# <a name="tutorial-deploy-virtual-machine-extensions-with-azure-resource-manager-templates"></a>教程：使用 Azure 资源管理器模板部署虚拟机扩展

了解如何使用 [Azure 虚拟机扩展](../virtual-machines/extensions/features-windows.md)在 Azure VM 上执行部署后配置和自动化任务。 有许多不同的 VM 扩展可与 Azure VM 配合使用。 在本教程中，你将从 Azure 资源管理器模板部署自定义脚本扩展，以便在 Windows VM 上运行 PowerShell 脚本。  此脚本在 VM 上安装 Web 服务器。

本教程涵盖以下任务：

> [!div class="checklist"]
> * 准备 PowerShell 脚本
> * 打开快速入门模板
> * 编辑模板
> * 部署模板
> * 验证部署

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含资源管理器工具扩展的 [Visual Studio Code](https://code.visualstudio.com/)。 请参阅[安装扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)。
* 若要提高安全性，请使用为虚拟机管理员帐户生成的密码。 以下是密码生成示例：

    ```azurecli-interactive
    openssl rand -base64 32
    ```

    Azure Key Vault 旨在保护加密密钥和其他机密。 有关详细信息，请参阅[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md)。 我们还建议每三个月更新一次密码。

## <a name="prepare-a-powershell-script"></a>准备 PowerShell 脚本

包含以下内容的 PowerShell 脚本在[可以公开访问的 Azure 存储帐户](https://armtutorials.blob.core.windows.net/usescriptextensions/installWebServer.ps1)上共享：

```azurepowershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

如果选择将文件发布到你自己的位置，则必须在教程的后面部分更新模板中的 `fileUri` 元素。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

Azure 快速入门模板是资源管理器模板的存储库。 无需从头开始创建模板，只需找到一个示例模板并对其自定义即可。 本教程中使用的模板称为[部署简单的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。

1. 在 Visual Studio Code 中，选择“文件” > “打开文件”。
1. 在“文件名”框中粘贴以下 URL： https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json 

1. 若要打开该文件，请选择“打开”。  
    该模板定义五个资源：

   * **Microsoft.Storage/storageAccounts**。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/Microsoft.Storage/storageAccounts)。
   * **Microsoft.Network/publicIPAddresses**。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/publicipaddresses)。
   * **Microsoft.Network/virtualNetworks**。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks)。
   * **Microsoft.Network/networkInterfaces**。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.network/networkinterfaces)。
   * **Microsoft.Compute/virtualMachines**。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines)。

     在自定义模板之前，不妨对其进行一些基本的了解。

1. 选择“文件” > “另存为”，使用文件名 *azuredeploy.json* 将该文件的副本保存到本地计算机。

## <a name="edit-the-template"></a>编辑模板

将虚拟机扩展资源添加到包含以下内容的现有模板：

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat(variables('vmName'),'/', 'InstallWebServer')]",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/',variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        "autoUpgradeMinorVersion":true,
        "settings": {
            "fileUris": [
                "https://armtutorials.blob.core.windows.net/usescriptextensions/installWebServer.ps1"
            ],
            "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File installWebServer.ps1"
        }
    }
}
```

有关此资源定义的详细信息，请查看[扩展参考](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines/extensions)。 下面是一些重要元素：

* **名称**：由于扩展资源是虚拟机对象的子资源，因此其名称必须有虚拟机名称前缀。 请参阅[子资源](./resource-group-authoring-templates.md#child-resources)。
* **dependsOn**：在创建虚拟机以后创建扩展资源。
* **fileUris**：存储脚本文件的位置。 如果不使用提供的位置，则需更新这些值。
* **commandToExecute**：此命令调用脚本。  

## <a name="deploy-the-template"></a>部署模板

有关部署过程，请参阅“部署模板”部分，文档为[教程：使用依赖的资源创建 Azure 资源管理器模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)中使用的。 建议使用为虚拟机管理员帐户生成的密码。 请参阅本文的[先决条件](#prerequisites)部分。

## <a name="verify-the-deployment"></a>验证部署

1. 在 Azure 门户中选择 VM。
1. 在 VM 概述中，选择“单击进行复制”复制 IP 地址，并将其粘贴到浏览器标签页中。  
   此时会打开默认的 Internet Information Services (IIS) 欢迎页：

![Internet Information Services 欢迎页](./media/resource-manager-tutorial-deploy-vm-extensions/resource-manager-template-deploy-extensions-customer-script-web-server.png)

## <a name="clean-up-resources"></a>清理资源

不再需要部署的 Azure 资源时，请通过删除资源组将其清除。

1. 在 Azure 门户的左窗格中选择“资源组”。
2. 在“按名称筛选”框中输入资源组名称。
3. 选择资源组名称。  
    将显示资源组中的六个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

在本教程中，你已部署虚拟机和虚拟机扩展。 此扩展在虚拟机上安装了 IIS Web 服务器。 若要了解如何使用 Azure SQL 数据库扩展导入 BACPAC 文件，请参阅：

> [!div class="nextstepaction"]
> [部署 SQL 扩展](./resource-manager-tutorial-deploy-sql-extensions-bacpac.md)
