---
title: 教程 - 将 Azure Key Vault 与通过 .NET 编写的 Windows 虚拟机配合使用 | Microsoft Docs
description: 本教程介绍如何将 ASP.NET Core 应用程序配置为从密钥保管库读取机密。
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.topic: tutorial
ms.date: 01/02/2019
ms.author: pryerram
ms.custom: mvc
ms.openlocfilehash: c88977f465de6d9b89bd2d9c4cf67402fe6f563f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228149"
---
# <a name="tutorial-use-azure-key-vault-with-a-windows-virtual-machine-in-net"></a>教程：将 Azure Key Vault 与通过 .NET 编写的 Windows 虚拟机配合使用

Azure Key Vault 用于保护机密，例如访问应用程序、服务和 IT 资源所需的 API 密钥、数据库连接字符串。

本教程介绍如何获取控制台应用程序，以便从 Azure Key Vault 读取信息。 为此，请将托管标识用于 Azure 资源。 

本教程介绍如何：

> [!div class="checklist"]
> * 创建资源组。
> * 创建密钥保管库。
> * 将机密添加到 Key Vault。
> * 从密钥保管库检索机密。
> * 创建一个 Azure 虚拟机。
> * 为虚拟机启用[托管标识](../active-directory/managed-identities-azure-resources/overview.md)。
> * 为 VM 标识分配权限。

在开始之前，请阅读 [Key Vault 的基本概念](key-vault-whatis.md#basic-concepts)。 

如果没有 Azure 订阅，请创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

对于 Windows、Mac 和 Linux：
  * [Git](https://git-scm.com/downloads)
  * 本部分教程要求在本地运行 Azure CLI。 必须安装 Azure CLI 2.0.4 或更高版本。 运行 `az --version` 即可查找版本。 如果需要安装或升级 CLI，请参阅[安装 Azure CLI 2.0](https://review.docs.microsoft.com/cli/azure/install-azure-cli)。

## <a name="about-managed-service-identity"></a>关于托管服务标识

Azure Key Vault 可以安全地存储凭据，因此不需要在代码中显示凭据。 但是，需要对 Azure Key Vault 进行身份验证才能检索密钥。 若要对 Key Vault 进行身份验证，需要提供凭据。 因此，在启动过程中，这是一个难以兼顾的典型问题。 托管服务标识 (MSI) 提供简化该过程的启动标识，可以解决此问题。

为 Azure 服务（例如 Azure 虚拟机、Azure 应用服务或 Azure Functions）启用 MSI 时，Azure 会创建一个[服务主体](key-vault-whatis.md#basic-concepts)。 MSI 针对 Azure Active Directory (Azure AD) 中的服务实例提供启动标识，并将服务主体凭据注入该实例。 

![MSI](media/MSI.png)

接下来，为了获取访问令牌，代码会调用 Azure 资源上提供的本地元数据服务。 代码使用从本地 MSI 终结点获取的访问令牌，以便向 Azure Key Vault 服务进行身份验证。 

## <a name="log-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 登录到 Azure，请输入：

```azurecli
az login
```

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

使用 [az group create](/cli/azure/group#az-group-create) 命令创建资源组。 

接下来，选择一个资源组名称，然后将其填充在占位符中。 以下示例在“美国西部”位置创建一个资源组：

```azurecli
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "West US"
```

本教程通篇使用新建的资源组。

## <a name="create-a-key-vault"></a>创建 key vault

若要在上一步创建的资源组中创建 Key Vault，请提供以下信息：

* Key Vault 名称：由 3 到 24 个字符构成的字符串，可以包含数字 (0-9)、字母 (a-z, A-Z) 和连字符 (-)
* 资源组名称
* 位置：**美国西部**

```azurecli
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "West US"
```
目前，只有你的 Azure 帐户才有权对这个新的 Key Vault 执行操作。

## <a name="add-a-secret-to-the-key-vault"></a>向密钥保管库添加机密

我们将添加机密以帮助说明这是如何工作的。 机密可以是 SQL 连接字符串，或者需要安全保存的、可供应用程序使用的其他任何信息。

若要在名为 **AppSecret** 的 Key Vault 中创建机密，请输入以下命令：

```azurecli
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

此机密将存储值 **MySecret**。

## <a name="create-a-virtual-machine"></a>创建虚拟机
可以使用以下方法之一创建虚拟机：

* [Azure CLI](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-cli)
* [PowerShell](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-powershell)
* [Azure 门户](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal)

## <a name="assign-an-identity-to-the-vm"></a>为 VM 分配标识
在此步骤中，请通过在 Azure CLI 中运行以下命令，为虚拟机创建系统分配的标识：

```azurecli
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

记下以下代码中显示的系统分配的标识。 以上命令的输出为： 

```azurecli
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>为 VM 标识分配权限
现在可以运行以下命令，将前面创建的标识权限分配到 Key Vault：

```azurecli
az keyvault set-policy --name '<YourKeyVaultName>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

## <a name="log-on-to-the-virtual-machine"></a>登录到虚拟机

若要登录到虚拟机，请按[连接并登录到运行 Windows 的 Azure 虚拟机](https://docs.microsoft.com/azure/virtual-machines/windows/connect-logon)中的说明操作。

## <a name="install-net-core"></a>安装 .NET Core

若要安装 .NET Core，请转到 [.NET 下载](https://www.microsoft.com/net/download)页。

## <a name="create-and-run-a-sample-net-app"></a>创建并运行示例 .NET 应用

打开命令提示符。

可以运行以下命令，将“Hello World”输出到控制台：

```batch
dotnet new console -o helloworldapp
cd helloworldapp
dotnet run
```

## <a name="edit-the-console-app"></a>编辑控制台应用

打开 *Program.cs* 文件，添加以下包：

```csharp
using System;
using System.IO;
using System.Net;
using System.Text;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

编辑类文件，使之包含在下面的两步过程中使用的代码：

1. 从 VM 上的本地 MSI 终结点获取一个令牌。 这还会从 Azure AD 获取令牌。
1. 将令牌传递到密钥保管库，然后获取机密。 

```csharp
 class Program
    {
        static void Main(string[] args)
        {
            // Step 1: Get a token from the local (URI) Managed Service Identity endpoint, which in turn fetches it from Azure AD
            var token = GetToken();

            // Step 2: Fetch the secret value from your key vault
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
            WebRequest kvRequest = WebRequest.Create("https://<YourVaultName>.vault.azure.net/secrets/<YourSecretName>?api-version=2016-10-01");
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

以上代码演示了如何在 Windows 虚拟机中通过 Azure Key Vault 执行操作。

## <a name="clean-up-resources"></a>清理资源

不再需要本教程中创建的虚拟机和 Key Vault 时，请将其删除。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Key Vault REST API](https://docs.microsoft.com/rest/api/keyvault/)
