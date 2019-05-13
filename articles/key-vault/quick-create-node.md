---
title: 快速入门 - 使用 Node Web 应用在 Azure Key Vault 中设置和检索机密 | Microsoft Docs
description: 在本快速入门中，你将使用 Node Web 应用在 Azure Key Vault 中设置和检索机密
services: key-vault
author: msmbaldwin
manager: sumedhb
ms.service: key-vault
ms.topic: quickstart
ms.date: 09/05/2018
ms.author: barclayn
ms.custom: mvc
ms.openlocfilehash: 5e8c29e033d895e24047754e686420fb4db86142
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236637"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-by-using-a-node-web-app"></a>快速入门：使用 Node Web 应用在 Azure Key Vault 中设置和检索机密 

本快速入门介绍如何在 Azure Key Vault 中存储机密，以及如何使用 Web 应用来检索该机密。 可以通过 Key Vault 来确保信息的安全。 若要查看机密值，必须在 Azure 上运行本快速入门。 本快速入门使用 Node.js 和 Azure 资源的托管标识。 学习如何：

* 创建密钥保管库。
* 在密钥保管库中存储机密。
* 从密钥保管库检索机密。
* 创建 Azure Web 应用程序。
* 为 Web 应用启用[托管标识](https://docs.microsoft.com/azure/active-directory/managed-service-identity/overview)。
* 授予所需的权限，让 Web 应用程序从密钥保管库读取数据。

在继续操作之前，请确保熟悉 [Key Vault 的基本概念](key-vault-whatis.md#basic-concepts)。

> [!NOTE]
> Key Vault 是一个以编程方式存储机密的中央存储库。 但要这样做，应用程序和用户需要首先向 Key Vault 进行身份验证，即提供机密。 为了遵循安全最佳做法，第一个机密需要定期轮换。 
>
> 使用 [Azure 资源的托管服务标识](../active-directory/managed-identities-azure-resources/overview.md)，在 Azure 中运行的应用程序将获得由 Azure 自动管理的标识。 这有助于解决*机密采用问题*，这样用户和应用程序就可以遵循最佳做法，而不必担心轮换第一个机密。

## <a name="prerequisites"></a>先决条件

* [Node.js](https://nodejs.org/en/)
* [Git](https://www.git-scm.com/)
* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 2.0.4 或更高版本。 本快速入门要求在本地运行 Azure CLI。 运行 `az --version` 即可查找版本。 如果需要安装或升级 CLI，请参阅[安装 Azure CLI 2.0](https://review.docs.microsoft.com/en-us/cli/azure/install-azure-cli?branch=master&view=azure-cli-latest)。
* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="log-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 登录到 Azure，请输入：

```azurecli
az login
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/azure/group#az-group-create) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

选择一个资源组名称，然后将其填充在占位符中。
以下示例在“美国东部”位置创建一个资源组。

```azurecli
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "East US"
```

刚刚创建的资源组将在整篇文章中使用。

## <a name="create-a-key-vault"></a>创建 key vault

接下来，使用在上一步骤创建的资源组创建一个 Key Vault。 必须对 Key Vault 使用唯一的名称。本文使用“ContosoKeyVault”作为名称。 提供以下信息：

* Key Vault 名称。
* 资源组名称。 名称必须是 3-24 个字符的字符串，并且只能包含 0-9、a-z、A-Z 和连字符 (-)。
* 位置：**美国东部**。

```azurecli
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "East US"
```

目前，只有你的 Azure 帐户才有权对这个新保管库执行任何操作。

## <a name="add-a-secret-to-the-key-vault"></a>向密钥保管库添加机密

我们将添加机密以帮助说明这是如何工作的。 可以存储需要安全保存的，但同时也要提供给应用程序使用的 SQL 连接字符串或其他任何信息。 在本教程中，密码名为 **AppSecret**，将在其中存储 **MySecret** 的值。

键入以下命令，在名为 **AppSecret** 的密钥保管库中创建机密。 此机密将存储值“MySecret”。

```azurecli
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

若要查看机密中包含的纯文本形式的值，请执行以下命令：

```azurecli
az keyvault secret show --name "AppSecret" --vault-name "<YourKeyVaultName>"
```

此命令显示机密信息，包括 URI。 完成这些步骤后，密钥保管库中会出现某个机密的 URI。 请记下此信息， 后面的步骤需要用到。

## <a name="clone-the-repo"></a>克隆存储库

克隆存储库，以便创建一个可以在其中编辑源的本地副本。 运行以下命令：

```
git clone https://github.com/Azure-Samples/key-vault-node-quickstart.git
```

## <a name="install-dependencies"></a>安装依赖项

运行以下命令安装依赖项：

```
cd key-vault-node-quickstart
npm install
```

此项目使用两个 Node 模块：[ms-rest-azure](https://www.npmjs.com/package/ms-rest-azure) 和 [azure-keyvault](https://www.npmjs.com/package/azure-keyvault)。

## <a name="publish-the-web-app-to-azure"></a>将 Web 应用发布到 Azure

创建 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)计划。 可以在此计划中存储多个 Web 应用。

    ```
    az appservice plan create --name myAppServicePlan --resource-group myResourceGroup
    ```
接下来创建 Web 应用。 在以下示例中，请将 `<app_name>` 替换为全局唯一的应用名称（有效字符为 a-z、0-9 和 -）。 运行时设置为 NODE|6.9。 若要查看所有受支持的运行时，请运行 `az webapp list-runtimes`。

    ```
    # Bash
    az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --runtime "NODE|6.9" --deployment-local-git
    ```
创建 Web 应用后，Azure CLI 会显示类似于以下示例的输出：

    ```
    {
      "availabilityState": "Normal",
      "clientAffinityEnabled": true,
      "clientCertEnabled": false,
      "cloningInfo": null,
      "containerSize": 0,
      "dailyMemoryTimeQuota": 0,
      "defaultHostName": "<app_name>.azurewebsites.net",
      "enabled": true,
      "deploymentLocalGitUrl": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git"
      < JSON data removed for brevity. >
    }
    ```
浏览到新建的 Web 应用，应会看到它正在运行。 将 `<app_name>` 替换为唯一应用名称。

    ```
    http://<app name>.azurewebsites.net
    ```
以上命令还会创建一个支持 Git 的应用，可将该应用从本地 Git 存储库部署到 Azure。 本地 Git 存储库是使用此 URL 配置的：`https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git`。

完成上述命令后，可将 Azure 远程功能添加到本地 Git 存储库。 请将 `<url>` 替换为 Git 存储库的 URL。

    ```
    git remote add azure <url>
    ```

## <a name="enable-a-managed-identity-for-the-web-app"></a>为 Web 应用启用托管标识

虽然 Azure Key Vault 可用于安全存储凭据以及其他密钥和机密，但代码需要通过 Key Vault 的身份验证才能检索它们。 [Azure 资源的托管标识概述](../active-directory/managed-identities-azure-resources/overview.md)为 Azure 服务提供了 Azure Active Directory (Azure AD) 中的自动托管标识，更巧妙地解决了这个问题。 此标识可用于通过支持 Azure AD 身份验证的任何服务（包括 Key Vault）的身份验证，这样就无需在代码中插入任何凭据了。

运行 assign-identity 命令，为此应用程序创建标识：

```azurecli
az webapp identity assign --name <app_name> --resource-group "<YourResourceGroupName>"
```

此命令等同于转到门户并在 Web 应用程序属性中将“标识/系统分配”设置切换为“打开”。

### <a name="assign-permissions-to-your-application-to-read-secrets-from-key-vault"></a>为应用程序分配从 Key Vault 读取机密的权限

记下上述命令的输出。 它应采用以下格式：
        
        {
          "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "type": "SystemAssigned"
        }
        
然后，使用 Key Vault 的名称和 **principalId** 的值运行以下命令：

```azurecli
az keyvault set-policy --name '<YourKeyVaultName>' --object-id <PrincipalId> --secret-permissions get set
```

## <a name="deploy-the-node-app-to-azure-and-retrieve-the-secret-value"></a>将 Node 应用部署到 Azure 并检索机密值

运行以下命令，将应用部署到 Azure：

```
git push azure master
```

然后，在浏览到 `https://<app_name>.azurewebsites.net` 时，即可看到机密值。 确保已将名称 `<YourKeyVaultName>` 替换为保管库名称。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure SDK for Node](https://docs.microsoft.com/javascript/api/overview/azure/key-vault)
