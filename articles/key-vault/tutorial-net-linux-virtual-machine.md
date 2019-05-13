---
title: 教程 - 如何将 Azure Key Vault 与通过 .NET 编写的 Azure Linux 虚拟机配合使用 - Azure Key Vault | Microsoft Docs
description: 教程：将 ASP.NET Core 应用程序配置为从 Key Vault 读取机密
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.topic: tutorial
ms.date: 12/21/2018
ms.author: pryerram
ms.custom: mvc
ms.openlocfilehash: 88d06518c6cabe1f796dbdef6d954db83bc5ae28
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234015"
---
# <a name="tutorial-how-to-use-azure-key-vault-with-azure-linux-virtual-machine-in-net"></a>教程：如何将 Azure Key Vault 与通过 .NET 编写的 Azure Linux 虚拟机配合使用

Azure Key Vault 用于保护机密，例如访问应用程序、服务和 IT 资源所需的 API 密钥、数据库连接字符串。

本教程演练将控制台应用程序配置为使用 Azure 资源的托管标识，以从 Azure Key Vault 读取信息所要执行的步骤。 下面介绍如何：

> [!div class="checklist"]
> * 创建密钥保管库。
> * 在密钥保管库中存储机密。
> * 从密钥保管库检索机密。
> * 创建一个 Azure 虚拟机。
> * 为虚拟机启用[托管标识](../active-directory/managed-identities-azure-resources/overview.md)。
> * 授予所需的权限，让控制台应用程序从密钥保管库读取数据。
> * 从 Key Vault 检索机密

在我们进一步讨论之前，请阅读[基本概念](key-vault-whatis.md#basic-concepts)。

## <a name="prerequisites"></a>先决条件
* 所有平台：
  * Git（[下载](https://git-scm.com/downloads)）。
  * Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
  * [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 2.0.4 或更高版本。 适用于 Windows、Mac 和 Linux。

本教程使用托管服务标识

## <a name="what-is-managed-service-identity-and-how-does-it-work"></a>什么是托管服务标识？其工作原理是什么？

在进一步讨论之前，让我们了解 MSI。 Azure Key Vault 可以安全地存储凭据，因此不需将凭据置于代码中，但若要检索这些凭据，需向 Azure Key Vault 进行身份验证。 若要向 Key Vault 进行身份验证，需提供凭据！ 经典的启动问题。 通过 Azure 和 Azure AD，MSI 提供一个“启动标识”，可以大为简化启动过程。

工作方式如下！ 为 Azure 服务（例如虚拟机、应用服务或 Functions）启用 MSI 时，Azure 会为 Azure Active Directory 中的服务实例创建一个[服务主体](key-vault-whatis.md#basic-concepts)，并将服务主体的凭据注入服务实例中。 

![MSI](media/MSI.png)

接下来，代码会调用 Azure 资源上提供的本地元数据服务，以便获取访问令牌。
代码使用从本地 MSI_ENDPOINT 获取的访问令牌，以便向 Azure Key Vault 服务进行身份验证。 

## <a name="sign-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 登录到 Azure，请输入：

```azurecli
az login
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/azure/group#az-group-create) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

选择一个资源组名称，然后将其填充在占位符中。
以下示例在“美国西部”位置创建一个资源组：

```azurecli
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "West US"
```

刚刚创建的资源组将在整篇文章中使用。

## <a name="create-a-key-vault"></a>创建 key vault

接下来，在上一步骤创建的资源组中创建密钥保管库。 提供以下信息：

* Key Vault 名称：名称必须为 3-24 个字符的字符串，并且只能包含 0-9、a-z、A-Z 和 -。
* 资源组名称。
* 位置：**美国西部**。

```azurecli
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "West US"
```
目前，只有你的 Azure 帐户才有权对这个新保管库执行任何操作。

## <a name="add-a-secret-to-the-key-vault"></a>向密钥保管库添加机密

我们将添加机密以帮助说明这是如何工作的。 可以存储需要安全保存的，但同时也要提供给应用程序使用的 SQL 连接字符串或其他任何信息。

键入以下命令，在名为 **AppSecret** 的密钥保管库中创建机密。 此机密将存储值“MySecret”。

```azurecli
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

## <a name="create-a-virtual-machine"></a>创建虚拟机

使用 [az vm create](/cli/azure/vm) 命令创建 VM。

以下示例创建一个名为 *myVM* 的 VM 并添加一个名为 *azureuser* 的用户帐户。 `--generate-ssh-keys` 参数用来自动生成一个 SSH 密钥，并将其放置在默认密钥位置 (*~/.ssh*) 中。 若要改为使用一组特定的密钥，请使用 `--ssh-key-value` 选项。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

创建 VM 和支持资源需要几分钟时间。 以下示例输出表明 VM 创建操作已成功。

```
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

## <a name="assign-identity-to-virtual-machine"></a>为虚拟机分配标识

在此步骤中，我们将为虚拟机创建一个系统分配标识，方法是运行以下命令

```
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

请记下下面显示的 systemAssignedIdentity。 以上命令的输出将为 

```
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="give-vm-identity-permission-to-key-vault"></a>为 VM 标识提供 Key Vault 访问权限

现在，我们可以运行以下命令，为上面创建的标识授予 Key Vault 访问权限

```
az keyvault set-policy --name '<YourKeyVaultName>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

## <a name="sign-in-to-the-virtual-machine"></a>登录到虚拟机

现在，使用终端登录到虚拟机

```
ssh azureuser@<PublicIpAddress>
```

## <a name="install-dot-net-core-on-linux"></a>在 Linux 上安装 Dot Net Core

### <a name="register-the-microsoft-product-key-as-trusted-run-the-following-two-commands"></a>将 Microsoft 产品密钥注册为受信任的密钥。 运行以下两个命令

```
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
```

### <a name="set-up-desired-version-host-package-feed-based-on-operating-system"></a>根据操作系统设置所需版本主机包源

```
 
# Ubuntu 16.04 / Linux Mint 18
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update
 
# Ubuntu 14.04 / Linux Mint 17
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-trusty-prod trusty main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update
```

### <a name="install-net-core"></a>安装 .NET Core

查看 .NET 版本

```
sudo apt-get install dotnet-sdk-2.1.4
dotnet --version
```

## <a name="create-and-run-sample-dot-net-app"></a>创建并运行示例 Dot Net 应用

运行以下命令时，会看到“Hello World”输出到控制台

```
dotnet new console -o helloworldapp
cd helloworldapp
dotnet run
```

## <a name="edit-console-app"></a>编辑控制台应用

打开 Program.cs 文件，添加以下包
```
using System;
using System.IO;
using System.Net;
using System.Text;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

然后，更改类文件，使之包含以下代码。 此过程由两个步骤组成。

1. 从 VM 上的本地 MSI 终结点获取一个令牌，该终结点会转而从 Azure Active Directory 获取令牌
2. 将令牌传递到 Key Vault，获取机密 

```
 class Program
    {
        static void Main(string[] args)
        {
            // Step 1: Get a token from local (URI) Managed Service Identity endpoint which in turn fetches it from Azure Active Directory
            var token = GetToken();

            // Step 2: Fetch the secret value from Key Vault
            System.Console.WriteLine(FetchSecretValueFromKeyVault(token));
        }

        static string GetToken()
        {
            WebRequest request = WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net");
            request.Headers.Add("Metadata", "true");
            WebResponse response = request.GetResponse();
            return ParseWebResponse(response, "access_token");
        }

        static string FetchSecretValueFromKeyVault(string token)
        {
            WebRequest kvRequest = WebRequest.Create("https://prashanthwinvmvault.vault.azure.net/secrets/RandomSecret?api-version=2016-10-01");
            kvRequest.Headers.Add("Authorization", "Bearer "+  token);
            WebResponse kvResponse = kvRequest.GetResponse();
            return ParseWebResponse(kvResponse, "value");
        }

        private static string ParseWebResponse(WebResponse response, string tokenName)
        {
            string token = String.Empty;
            using (Stream stream = response.GetResponseStream())
            {
                StreamReader reader = new StreamReader(stream, Encoding.UTF8);
                String responseString = reader.ReadToEnd();

                JObject joResponse = JObject.Parse(responseString);    
                JValue ojObject = (JValue)joResponse[tokenName];   
                token = ojObject.Value.ToString();
            }
            return token;
        }
    }
```

以上代码演示了如何在 Azure Linux 虚拟机中通过 Azure Key Vault 执行操作。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Key Vault REST API](https://docs.microsoft.com/rest/api/keyvault/)
