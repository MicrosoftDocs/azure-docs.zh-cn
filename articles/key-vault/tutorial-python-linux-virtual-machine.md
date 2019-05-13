---
title: 教程 - 使用 Linux 虚拟机和 Python 应用程序在 Azure 密钥保管库中存储机密 | Microsoft Docs
description: 在本教程中，你会了解如何配置一个 Python 应用程序，以便从 Azure 密钥保管库读取机密。
services: key-vault
author: mbaldwin
manager: rajvijan
ms.service: key-vault
ms.topic: tutorial
ms.date: 09/05/2018
ms.author: pryerram
ms.custom: mvc
ms.openlocfilehash: a5923c3d5ea7888a3737d7976caded8eef9c1661
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64702110"
---
# <a name="tutorial-use-a-linux-vm-and-a-python-app-to-store-secrets-in-azure-key-vault"></a>教程：使用 Linux VM 和 Python 应用在 Azure 密钥保管库中存储机密

Azure Key Vault 用于保护机密，例如访问应用程序、服务和 IT 资源所需的 API 密钥和数据库连接字符串。

在本教程中，你会设置 Azure Web 应用程序以使用 Azure 资源的托管标识从 Azure 密钥保管库读取信息。 学习如何：

> [!div class="checklist"]
> * 创建 key vault
> * 在密钥保管库中存储机密
> * 创建 Linux 虚拟机
> * 为虚拟机启用[托管标识](../active-directory/managed-identities-azure-resources/overview.md)
> * 授予所需的权限，让控制台应用程序从密钥保管库读取数据
> * 从密钥保管库检索机密

在进一步讨论之前，确保了解[有关密钥保管库的基本概念](key-vault-whatis.md#basic-concepts)。

## <a name="prerequisites"></a>先决条件

* [Git](https://git-scm.com/downloads)。
* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [Azure CLI 2.0.4 或更高版本](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)或是 Azure Cloud Shell。

[!INCLUDE [Azure Cloud Shell](../../includes/cloud-shell-try-it.md)]

## <a name="understand-managed-service-identity"></a>了解托管服务标识

Azure Key Vault 可以安全地存储凭据，因此不需要在代码中提供凭据。 若要检索它们，需向 Azure 密钥保管库进行身份验证。 但是，若要向密钥保管库进行身份验证，需提供凭据。 这是经典的启动问题。 通过 Azure 和 Azure Active Directory (Azure AD)，托管服务标识 (MSI) 提供一个“启动标识”，可以简化启动过程。

为 Azure 服务（例如：虚拟机、应用服务或 Functions）启用 MSI 时，Azure 会为 Azure AD 中的服务实例创建一个服务主体。 它会将服务主体的凭据注入服务的实例。

![MSI](media/MSI.png)

接下来，代码会调用 Azure 资源上提供的本地元数据服务，以获取访问令牌。 代码使用从本地 MSI 终结点获取的访问令牌，以便向 Azure Key Vault 服务进行身份验证。

## <a name="sign-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 登录到 Azure，请输入：

```azurecli-interactive
az login
```

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

通过以下代码在美国西部位置，使用 `az group create` 命令创建资源组。 将 `YourResourceGroupName` 替换所选名称。

```azurecli-interactive
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "West US"
```

本教程通篇使用此资源组。

## <a name="create-a-key-vault"></a>创建 key vault

接下来，在上一步骤创建的资源组中创建密钥保管库。 提供以下信息：

* Key Vault 名称：名称必须是 3-24 个字符的字符串，并且只能包含 0-9、a-z、A-Z 和连字符 (-)。
* 资源组名称。
* 位置：**美国西部**。

```azurecli-interactive
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "West US"
```

目前，只有你的 Azure 帐户才有权对这个新保管库执行任何操作。

## <a name="add-a-secret-to-the-key-vault"></a>向密钥保管库添加机密

我们将添加机密以帮助说明这是如何工作的。 可能要存储 SQL 连接字符串，或者需要保持安全并可供应用程序使用的其他任何信息。

键入以下命令，在名为 *AppSecret* 的密钥保管库中创建机密。 此机密将存储值“MySecret”。

```azurecli-interactive
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

## <a name="create-a-linux-virtual-machine"></a>创建 Linux 虚拟机

使用 `az vm create` 命令创建 VM。

以下示例创建一个名为 **myVM** 的 VM 并添加一个名为 **azureuser** 的用户帐户。 `--generate-ssh-keys` 参数自动生成一个 SSH 密钥，并将其放置在默认密钥位置 (**~/.ssh**) 中。 若要改为创建一组特定的密钥，请使用 `--ssh-key-value` 选项。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

创建 VM 和支持资源需要几分钟时间。 以下示例输出表明 VM 创建操作已成功：

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-00-00-00-00-00",
  "powerState": "VM running",
  "privateIpAddress": "XX.XX.XX.XX",
  "publicIpAddress": "XX.XX.XXX.XXX",
  "resourceGroup": "myResourceGroup"
}
```

记下 VM 输出中自己的 `publicIpAddress`。 在后续步骤中，将使用此地址访问 VM。

## <a name="assign-an-identity-to-the-vm"></a>为 VM 分配标识

为虚拟机创建一个系统分配标识，方法是运行以下命令：

```azurecli-interactive
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

该命令的输出如下所示。

```azurecli
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

记下 `systemAssignedIdentity`。 下一步会使用它。

## <a name="give-the-vm-identity-permission-to-key-vault"></a>为 VM 标识提供密钥保管库访问权限

现在可以向所创建的标识提供密钥保管库权限。 运行以下命令：

```azurecli-interactive
az keyvault set-policy --name '<YourKeyVaultName>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

## <a name="log-in-to-the-vm"></a>登录 VM

使用终端登录虚拟机。

```terminal
ssh azureuser@<PublicIpAddress>
```

## <a name="install-python-library-on-the-vm"></a>在 VM 上安装 Python 库

下载并安装[请求](https://pypi.org/project/requests/2.7.0/) Python 库以进行 HTTP GET 调用。

## <a name="create-edit-and-run-the-sample-python-app"></a>创建、编辑并运行示例 Python 应用

创建名为 Sample.py 的 Python 文件。

打开 Sample.py 并编辑它以包含以下代码：

```python
# importing the requests library
  import requests
  
# Step 1: Fetch an access token from an MSI-enabled Azure resource      
  # Note that the resource here is https://vault.azure.net for the public cloud, and api-version is 2018-02-01
  MSI_ENDPOINT = "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net"
  r = requests.get(MSI_ENDPOINT, headers = {"Metadata" : "true"})

# Extracting data in JSON format 
  # This request gets an access token from Azure Active Directory by using the local MSI endpoint
  data = r.json()

# Step 2: Pass the access token received from the previous HTTP GET call to the key vault
  KeyVaultURL = "https://prashanthwinvmvault.vault.azure.net/secrets/RandomSecret?api-version=2016-10-01"
  kvSecret = requests.get(url = KeyVaultURL, headers = {"Authorization": "Bearer " + data["access_token"]})

print(kvSecret.json()["value"])
```

前面的代码执行一个两步过程：

   1. 从 VM 上的本地 MSI 终结点提取令牌。 该终结点转而从 Azure Active Directory 获取令牌。
   1. 将令牌传递到密钥保管库，提取机密。

运行以下命令。 应看到机密值。

```console
python Sample.py
```

在本教程中，你学习了如何将 Azure 密钥保管库与 Linux 虚拟机上运行的 Python 应用结合使用。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组、虚拟机和所有相关的资源时，可将其删除。 为此，请选择适用于 VM 的资源组，然后选择“删除”。

使用 `az keyvault delete` 命令删除密钥保管库：

```azurecli-interactive
az keyvault delete --name
                   [--resource-group]
                   [--subscription]
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Key Vault REST API](https://docs.microsoft.com/rest/api/keyvault/)
