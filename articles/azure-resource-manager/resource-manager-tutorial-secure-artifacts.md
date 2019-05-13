---
title: 保护 Azure 资源管理器模板部署中的项目 | Microsoft Docs
description: 了解如何保护 Azure 资源管理器模板中使用的项目。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 02/25/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: fd68edcc727ab08ed9d3ba765bbe795e88de5fc9
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60391263"
---
# <a name="tutorial-secure-artifacts-in-azure-resource-manager-template-deployments"></a>教程：保护 Azure 资源管理器模板部署中的项目

了解如何使用 Azure 存储帐户和共享访问签名 (SAS) 保护 Azure 资源管理器模板中使用的项目。 部署项目是指完成部署所需的任何文件以及主模板文件。 例如，在[教程：使用 Azure 资源管理器模板导入 SQL BACPAC 文件](./resource-manager-tutorial-deploy-sql-extensions-bacpac.md)中，主模板创建 Azure SQL 数据库；它还调用一个 BACPAC 文件来创建表和插入数据。 BACPAC 文件是一个项目。 该项目存储在具有公共访问权限的 Azure 存储帐户中。 在本教程中，你将使用 SAS 来授予对自己 Azure 存储帐户中 BACPAC 文件的有限访问权限。 有关 SAS 的详细信息，请参阅[使用共享访问签名 (SAS)](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

若要了解如何保护链接的模板，请参阅[教程：创建链接的 Azure 资源管理器模板](./resource-manager-tutorial-create-linked-templates.md)。

本教程涵盖以下任务：

> [!div class="checklist"]
> * 准备 BACPAC 文件
> * 打开现有模板
> * 编辑模板
> * 部署模板
> * 验证部署

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含资源管理器工具扩展的 [Visual Studio Code](https://code.visualstudio.com/)。 请参阅[安装扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)。
* 查看[教程：使用 Azure 资源管理器模板导入 SQL BACPAC 文件](./resource-manager-tutorial-deploy-sql-extensions-bacpac.md)。 本教程中使用的模板是在该教程中开发的。 本文提供了已完成模板的下载链接。
* 若要增强安全性，请使用为 SQL Server 管理员帐户生成的密码。 以下是密码生成示例：

    ```azurecli-interactive
    openssl rand -base64 32
    ```
    Azure Key Vault 旨在保护加密密钥和其他机密。 有关详细信息，请参阅[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md)。 我们还建议你每三个月更新一次密码。

## <a name="prepare-a-bacpac-file"></a>准备 BACPAC 文件

在本部分准备 BACPAC 文件，以便在部署资源管理器模板时可以安全访问该文件。 本部分包括五个过程：

* 下载 BACPAC 文件。
* 创建 Azure 存储帐户。
* 创建存储帐户 Blob 容器。
* 将 BACPAC 文件上传到该容器。
* 检索 BACPAC 文件的 SAS 令牌。

若要使用 PowerShell 脚本自动完成这些步骤，请参阅[上传链接的模板](./resource-manager-tutorial-create-linked-templates.md#upload-the-linked-template)中的脚本。

### <a name="download-the-bacpac-file"></a>下载 BACPAC 文件

下载 [BACPAC 文件](https://armtutorials.blob.core.windows.net/sqlextensionbacpac/SQLDatabaseExtension.bacpac)，并以相同的名称 **SQLDatabaseExtension.bacpac** 将其保存到本地计算机。

### <a name="create-a-storage-account"></a>创建存储帐户

1. 选择以下映像在 Azure 门户中打开一个资源管理器模板。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json" target="_blank"><img src="./media/resource-manager-tutorial-secure-artifacts/deploy-to-azure.png" alt="Deploy to Azure"></a>
2. 输入以下属性：

    * **订阅**：选择 Azure 订阅。
    * **资源组**：选择“新建”并指定名称。 资源组是 Azure 资源的容器，用于实现管理目的。 在本教程中，可为存储帐户和 Azure SQL 数据库使用同一个资源组。 请记下此资源组名称，因为稍后在本教程中创建 Azure SQL 数据库时需要用到。
    * **位置**：选择区域。 例如“美国中部”。 
    * **存储帐户类型**：使用默认值“Standard_LRS”。
    * **位置**：使用默认值“[resourceGroup().location]”。 这意味着，要使用存储帐户的资源组位置。
    * **我同意上述条款和条件**：（选中）
3. 选择“购买”。
4. 选择门户右上角的通知图标（钟形图标）查看部署状态。

    ![资源管理器教程门户通知窗格](./media/resource-manager-tutorial-secure-artifacts/resource-manager-tutorial-portal-notifications-pane.png)
5. 成功部署存储帐户后，在通知窗格中选择“转到资源组”打开该资源组。

### <a name="create-a-blob-container"></a>创建 Blob 容器

在上传任何文件之前，需要创建一个 Blob 容器。 

1. 选择存储帐户以将其打开。 应会看到，资源组中只列出了一个存储帐户。 你的存储帐户名称不同于以下屏幕截图所示的名称。

    ![资源管理器教程存储帐户](./media/resource-manager-tutorial-secure-artifacts/resource-manager-tutorial-storage-account.png)

2. 选择“Blob”磁贴。

    ![资源管理器教程 Blob](./media/resource-manager-tutorial-secure-artifacts/resource-manager-tutorial-blobs.png)
3. 选择顶部的“+ 容器”创建新容器。
4. 输入以下值：

    * **名称**：输入 **sqlbacpac**。 
    * **公共访问级别**：使用默认值“专用(不允许匿名访问)”。
5. 选择“确定”。
6. 选择“sqlbacpac”打开新建的容器。

### <a name="upload-the-bacpac-file-to-the-container"></a>将 BACPAC 文件上传到容器

1. 选择“上传”。
2. 输入以下值：

    * **文件**：遵照说明选择前面下载的 BACPAC 文件。 默认名称为 **SQLDatabaseExtension.bacpac**。
    * **身份验证类型**：选择“SAS”。  “SAS”是默认值。
3. 选择“上传”。  成功上传文件后，容器中应会列出其文件名。

### <a name="a-namegenerate-a-sas-token-generate-a-sas-token"></a><a name="generate-a-sas-token" />生成 SAS 令牌

1. 右键单击容器中的“SQLDatabaseExtension.bacpac”，并选择“生成 SAS”。
2. 输入以下值：

    * **权限**：使用默认值“读取”。
    * **开始和过期日期/时间**：默认值为使用 SAS 令牌八小时。 如果需要更多的时间来完成本教程，请更新“过期时间”。
    * **允许的 IP 地址**：将此字段留空。
    * **允许的协议**：使用默认值“HTTPS”。
    * **签名密钥**：使用默认值“密钥 1”。
3. 选择“生成 Blob SAS 令牌和 URL”。
4. 复制“Blob SAS URL”。 URL 的中间是文件名 **SQLDatabaseExtension.bacpac**。  文件名将 URL 划分为三个部分：

   - **项目位置**： https://xxxxxxxxxxxxxx.blob.core.windows.net/sqlbacpac/。 请确保位置以“/”结尾。
   - **BACPAC 文件名**：SQLDatabaseExtension.bacpac。
   - **项目位置 SAS 令牌**：请确保该令牌的前面带有“?”。

     [部署模板](#deploy-the-template)时需要使用这三个值。

## <a name="open-an-existing-template"></a>打开现有模板

在此会话中，修改在[教程：使用 Azure 资源管理器模板导入 SQL BACPAC 文件](./resource-manager-tutorial-deploy-sql-extensions-bacpac.md)中创建的模板，以通过 SAS 令牌调用 BACPAC 文件。  在 SQL 扩展教程中开发的模板已在 [https://armtutorials.blob.core.windows.net/sqlextensionbacpac/azuredeploy.json](https://armtutorials.blob.core.windows.net/sqlextensionbacpac/azuredeploy.json) 位置共享。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://armtutorials.blob.core.windows.net/sqlextensionbacpac/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。

    该模板中定义了五个资源：

   * `Microsoft.Sql/servers`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/2015-05-01-preview/servers)。
   * `Microsoft.SQL/servers/securityAlertPolicies`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/2014-04-01/servers/databases/securityalertpolicies)。
   * `Microsoft.SQL/servers/filewallRules`。 请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/2015-05-01-preview/servers/firewallrules)。
   * `Microsoft.SQL/servers/databases`。  请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/servers/databases)。
   * `Microsoft.SQL/server/databases/extensions`。  请参阅[模板参考](https://docs.microsoft.com/azure/templates/microsoft.sql/2014-04-01/servers/databases/extensions)。

     在自定义模板之前，不妨对其进行一些基本的了解。
4. 选择“文件”>“另存为”，将该文件的副本保存到名为 **azuredeploy.json** 的本地计算机。

## <a name="edit-the-template"></a>编辑模板

添加以下附加参数：

```json
"_artifactsLocation": {
    "type": "string",
    "metadata": {
        "description": "The base URI where artifacts required by this template are located."
    }
},
"_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
        "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
},
"bacpacFileName": {
    "type": "string",
    "defaultValue": "SQLDatabaseExtension.bacpac",
    "metadata": {
        "description": "The bacpac for configure the database and tables."
    }
}
```

![资源管理器教程安全项目参数](./media/resource-manager-tutorial-secure-artifacts/resource-manager-tutorial-secure-artifacts-parameters.png)

更新以下两个元素的值：

```json
"storageKey": "[parameters('_artifactsLocationSasToken')]",
"storageUri": "[uri(parameters('_artifactsLocation'), parameters('bacpacFileName'))]",
```

## <a name="deploy-the-template"></a>部署模板

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

有关部署过程，请参阅[部署模板](./resource-manager-tutorial-create-multiple-instances.md#deploy-the-template)部分。 改用以下 PowerShell 部署脚本：

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the virtual machine admin username"
$adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
$artifactsLocation = Read-Host -Prompt "Enter the artifacts location"
$artifactsLocationSasToken = Read-Host -Prompt "Enter the artifacts location SAS token" -AsSecureString
$bacpacFileName = Read-Host -Prompt "Enter the BACPAC file name"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUser $adminUsername `
    -adminPassword $adminPassword `
    -_artifactsLocation $artifactsLocation `
    -_artifactsLocationSasToken $artifactsLocationSasToken `
    -bacpacFileName $bacpacFileName `
    -TemplateFile "$HOME/azuredeploy.json"
```

使用生成的密码。 请参阅[先决条件](#prerequisites)。
有关 _artifactsLocation、_artifactsLocationSasToken 和 bacpacFileName 的值，请参阅[生成 SAS 令牌](#generate-a-sas-token)。

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

在本教程中，你已部署 SQL Server、SQL 数据库，并已使用 SAS 令牌导入 BACPAC 文件。 若要了解如何跨多个区域部署 Azure 资源，以及如何使用安全部署做法，请参阅

> [!div class="nextstepaction"]
> [使用 Azure 部署管理器](./resource-manager-tutorial-deploy-vm-extensions.md)
