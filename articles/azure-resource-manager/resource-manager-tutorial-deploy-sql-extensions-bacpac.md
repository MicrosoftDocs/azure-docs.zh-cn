---
title: 使用 Azure 资源管理器模板导入 SQL BACPAC 文件 | Microsoft Docs
description: 了解如何使用 SQL 数据库扩展，以便通过 Azure 资源管理器模板导入 SQL BACPAC 文件。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 04/08/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 08befabfbd14651475fa56dec95bdf4c2fe54c9c
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60390256"
---
# <a name="tutorial-import-sql-bacpac-files-with-azure-resource-manager-templates"></a>教程：使用 Azure 资源管理器模板导入 SQL BACPAC 文件

了解如何使用 Azure SQL 数据库扩展，以通过 Azure 资源管理器模板导入 BACPAC 文件。 部署项目包括主模板文件以及完成部署所需的任何文件。 BACPAC 文件是一个项目。 在本教程中，你将创建一个模板来部署 Azure SQL Server、SQL 数据库并导入一个 BACPAC 文件。 若要了解如何使用 Azure 资源管理器模板来部署 Azure 虚拟机扩展，请参阅 [# 教程：使用 Azure 资源管理器模板部署虚拟机扩展](./resource-manager-tutorial-deploy-vm-extensions.md)。

本教程涵盖以下任务：

> [!div class="checklist"]
> * 准备 BACPAC 文件
> * 打开快速入门模板
> * 编辑模板
> * 部署模板
> * 验证部署

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含资源管理器工具扩展的 [Visual Studio Code](https://code.visualstudio.com/)。 请参阅[安装扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)。
* 若要增强安全性，请使用为 SQL Server 管理员帐户生成的密码。 以下是密码生成示例：

    ```azurecli-interactive
    openssl rand -base64 32
    ```
    Azure Key Vault 旨在保护加密密钥和其他机密。 有关详细信息，请参阅[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md)。 我们还建议你每三个月更新一次密码。

## <a name="prepare-a-bacpac-file"></a>准备 BACPAC 文件

BACPAC 文件在[可以公开访问的 Azure 存储帐户](https://armtutorials.blob.core.windows.net/sqlextensionbacpac/SQLDatabaseExtension.bacpac)上共享。 若要创建自己的文件，请参阅[将 Azure SQL 数据库导出到 BACPAC 文件](../sql-database/sql-database-export.md)。 如果选择将文件发布到你自己的位置，则必须在教程的后面部分更新模板。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

本教程中使用的模板存储在 [Azure 存储帐户](https://armtutorials.blob.core.windows.net/createsql/azuredeploy.json)中。 

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://armtutorials.blob.core.windows.net/createsql/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。

    有三个在此模板中定义的资源：

   * `Microsoft.Sql/servers`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/servers)。
   * `Microsoft.SQL/servers/securityAlertPolicies`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/servers/securityalertpolicies)。
   * `Microsoft.SQL.servers/databases`。  请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/servers/databases)。

     在自定义模板之前，不妨对其进行一些基本的了解。
4. 选择“文件”>“另存为”，将该文件的副本保存到名为 **azuredeploy.json** 的本地计算机。

## <a name="edit-the-template"></a>编辑模板

向模板添加两个其他资源。

* 若要允许 SQL 数据库扩展导入 BACPAC 文件，需允许访问 Azure 服务。 将以下 JSON 添加到 SQL 服务器定义：

    ```json
    {
        "type": "firewallrules",
        "name": "AllowAllAzureIps",
        "location": "[parameters('location')]",
        "apiVersion": "2015-05-01-preview",
        "dependsOn": [
            "[variables('databaseServerName')]"
        ],
        "properties": {
            "startIpAddress": "0.0.0.0",
            "endIpAddress": "0.0.0.0"
        }
    }
    ```

    模板应如下所示：

    ![Azure 资源管理器部署 sql 扩展 BACPAC](./media/resource-manager-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-firewall.png)

* 使用以下 JSON 将 SQL 数据库扩展资源添加到数据库定义：

    ```json
    "resources": [
        {
            "name": "Import",
            "type": "extensions",
            "apiVersion": "2014-04-01",
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers/databases', variables('databaseServerName'), variables('databaseName'))]"
            ],
            "properties": {
                "storageKeyType": "SharedAccessKey",
                "storageKey": "?",
                "storageUri": "https://armtutorials.blob.core.windows.net/sqlextensionbacpac/SQLDatabaseExtension.bacpac",
                "administratorLogin": "[variables('databaseServerAdminLogin')]",
                "administratorLoginPassword": "[variables('databaseServerAdminLoginPassword')]",
                "operationMode": "Import",
            }
        }
    ]
    ```

    模板应如下所示：

    ![Azure 资源管理器部署 sql 扩展 BACPAC](./media/resource-manager-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac.png)

    若要了解资源定义，请参阅 [SQL 数据库扩展参考](https://docs.microsoft.com/azure/templates/microsoft.sql/servers/databases/extensions)。 下面是一些重要元素：

    * **dependsOn**：必须在创建 SQL 数据库以后才能创建扩展资源。
    * **storageKeyType**：要使用的存储密钥的类型。 值可以是 `StorageAccessKey` 或 `SharedAccessKey`。 由于提供的 BACPAC 文件在可以公开访问的 Azure 存储帐户上共享，因此此处使用“SharedAccessKey”。
    * **storageKey**：要使用的存储密钥。 如果存储密钥类型为 SharedAccessKey，则必须以“?”为前缀。
    * **storageUri**：要使用的存储 URI。 如果选择不使用提供的 BACPAC 文件，则需更新这些值。
    * **administratorLoginPassword**：SQL 管理员的密码。 使用生成的密码。 请参阅[先决条件](#prerequisites)。

## <a name="deploy-the-template"></a>部署模板

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

有关部署过程，请参阅[部署模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)部分。 改用以下 PowerShell 部署脚本：

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the SQL admin username"
$adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUser $adminUsername `
    -adminPassword $adminPassword `
    -TemplateFile "$HOME/azuredeploy.json"
```

使用生成的密码。 请参阅[先决条件](#prerequisites)。

## <a name="verify-the-deployment"></a>验证部署

在门户中，从新部署的资源组中选择 SQL 数据库。 选择“查询编辑器(预览)”，然后输入管理员凭据。 此时会看到两个表导入到数据库中：

![Azure 资源管理器部署 sql 扩展 BACPAC](./media/resource-manager-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-query-editor.png)

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。  应会看到，该资源组中总共有六个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

在本教程中，你部署了 SQL Server、SQL 数据库并导入了 BACPAC 文件。 BACPAC 文件存储在 Azure 存储帐户中。 得到该 URL 的任何人都可以访问该文件。 若要了解如何保护 BACPAC 文件（项目），请参阅

> [!div class="nextstepaction"]
> [保护项目](./resource-manager-tutorial-secure-artifacts.md)
