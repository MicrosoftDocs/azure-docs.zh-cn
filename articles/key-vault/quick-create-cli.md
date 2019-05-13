---
title: Azure 快速入门 - 使用 Azure CLI 在 Key Vault 中设置和检索机密 | Microsoft Docs
description: 快速入门介绍如何使用 Azure CLI 在 Azure Key Vault 中设置和检索机密
services: key-vault
author: barclayn
manager: barbkess
tags: azure-resource-manager
ms.service: key-vault
ms.topic: quickstart
ms.custom: mvc
ms.date: 01/08/2019
ms.author: barclayn
ms.openlocfilehash: 353334ac371ad571f3290a43ce50b55770e06b1b
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64718335"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-azure-cli"></a>快速入门：使用 Azure CLI 在 Azure Key Vault 中设置和检索机密

Azure Key Vault 是一项云服务，用作安全的机密存储。 可以安全地存储密钥、密码、证书和其他机密。 有关 Key Vault 的详细信息，可以参阅[概述](key-vault-overview.md)。 Azure CLI 用于通过命令或脚本创建和管理 Azure 资源。 在本快速入门中，请创建一个密钥保管库。 完成该操作后，即可存储机密。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本快速入门要求 Azure CLI 2.0.4 或更高版本。 运行 `az --version` 即可查找版本。 如需进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。

若要使用 CLI 登录到 Azure，可以键入：

```azurecli
az login
```

若要详细了解通过 CLI 使用的登录选项，请参阅[使用 Azure CLI 登录](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)

## <a name="create-a-resource-group"></a>创建资源组

资源组是在其中部署和管理 Azure 资源的逻辑容器。 以下示例在“eastus”位置创建名为“ContosoResourceGroup”的资源组。

```azurecli
az group create --name "ContosoResourceGroup" --location eastus
```

## <a name="create-a-key-vault"></a>创建密钥保管库

接下来，将在上一步创建的资源组中创建 Key Vault。 需要提供某些信息：

- 对于本快速入门，请使用 **Contoso-vault2**。 必须在测试中提供唯一名称。
- 资源组名称 **ContosoResourceGroup**。
- 位置“美国东部”。

```azurecli
az keyvault create --name "Contoso-Vault2" --resource-group "ContosoResourceGroup" --location eastus
```

此 cmdlet 的输出显示新创建的 Key Vault 的属性。 请记下下面列出的两个属性：

- **保管库名称**：在本示例中，此项为 **Contoso-Vault2**。 将在其他 Key Vault 命令中使用此名称。
- **保管库 URI**：在本示例中，此项为 https://contoso-vault2.vault.azure.net/。 通过其 REST API 使用保管库的应用程序必须使用此 URI。

目前，只有你的 Azure 帐户才有权对这个新保管库执行任何操作。

## <a name="add-a-secret-to-key-vault"></a>向 Key Vault 添加机密

只需再执行几个步骤即可向保管库添加机密。 此密码可供应用程序使用。 此密码将名为 **ExamplePassword**，将在其中存储的值为 **hVFkk965BuUv**。

键入以下命令，在 Key Vault 中创建名为 **ExamplePassword** 的机密，用于存储的值将为 **hVFkk965BuUv**：

```azurecli
az keyvault secret set --vault-name "Contoso-Vault2" --name "ExamplePassword" --value "hVFkk965BuUv"
```

现在，可以通过使用密码的 URI，引用已添加到 Azure Key Vault 的此密码。 使用 **https://ContosoVault.vault.azure.net/secrets/ExamplePassword** 获取当前版本。 

若要查看机密中包含的纯文本形式的值，请执行以下命令：

```azurecli
az keyvault secret show --name "ExamplePassword" --vault-name "Contoso-Vault2"
```

现在，你已创建 Key Vault 并存储和检索了机密。

## <a name="clean-up-resources"></a>清理资源

本系列中的其他快速入门和教程是在本快速入门的基础上制作的。 如果打算继续使用后续的快速入门和教程，则可能需要保留这些资源。
如果不再需要资源组和所有相关的资源，可以使用 [az group delete](/cli/azure/group) 命令将其删除。 可以删除资源，如下所示：

```azurecli
az group delete --name ContosoResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已创建 Key Vault 并在其中存储了一个机密。 若要详细了解 Key Vault 以及如何将它与应用程序配合使用，请继续学习有关如何将 Web 应用程序与 Key Vault 配合使用的教程。

> [!div class="nextstepaction"]
> 若要了解如何使用 Web 应用程序通过 Azure 资源的托管标识从 Key Vault 读取机密，请继续学习以下教程：[将 Azure Web 应用程序配置为从 Key Vault 读取机密](quick-create-net.md)
