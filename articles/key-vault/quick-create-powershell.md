---
title: Azure 快速入门 - 使用 PowerShell 在 Key Vault 中设置和检索机密 | Microsoft Docs
description: ''
services: key-vault
author: barclayn
manager: barbkess
tags: azure-resource-manager
ms.service: key-vault
ms.topic: quickstart
ms.custom: mvc
ms.date: 01/07/2019
ms.author: barclayn
ms.openlocfilehash: eb9a18ad0fb00fc9f648e3b22b0db1ea4eda479d
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64712752"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-powershell"></a>快速入门：使用 PowerShell 在 Azure Key Vault 中设置和检索机密

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Azure Key Vault 是一项云服务，用作安全的机密存储。 可以安全地存储密钥、密码、证书和其他机密。 有关 Key Vault 的详细信息，可以参阅[概述](key-vault-overview.md)。 在本快速入门中，请使用 PowerShell 创建一个密钥保管库， 然后即可在新创建的保管库中存储机密。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果选择在本地安装并使用 PowerShell，则本教程需要 Azure PowerShell 模块 1.0.0 或更高版本。 键入 `$PSVersionTable.PSVersion` 即可查找版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-az-ps)。 如果在本地运行 PowerShell，则还需运行 `Login-AzAccount` 来创建与 Azure 的连接。

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

```azurepowershell-interactive
New-AzResourceGroup -Name ContosoResourceGroup -Location EastUS
```

## <a name="create-a-key-vault"></a>创建密钥保管库

接下来创建 Key Vault。 执行此步骤时，需要一些信息：

虽然在本快速入门中，我们始终使用“Contoso KeyVault2”作为 Key Vault 的名称，但你必须使用唯一的名称。

- **保管库名称** Contoso-Vault2。
- 资源组名称 **ContosoResourceGroup**。
- **位置**“美国东部”。

```azurepowershell-interactive
New-AzKeyVault -Name 'Contoso-Vault2' -ResourceGroupName 'ContosoResourceGroup' -Location 'East US'
```

此 cmdlet 的输出显示新创建的密钥保管库的属性。 请记下下面列出的两个属性：

* **保管库名称**：在本示例中为 **Contoso-Vault2**。 将在其他密钥保管库 cmdlet 中使用此名称。
* **保管库 URI**：在本示例中为 https://contosokeyvault.vault.azure.net/。 通过其 REST API 使用保管库的应用程序必须使用此 URI。

创建保管库以后，你的 Azure 帐户是唯一能够对这个新的保管库执行任何操作的帐户。

![Key Vault 创建命令完成后的输出](./media/quick-create-powershell/output-after-creating-keyvault.png)

## <a name="adding-a-secret-to-key-vault"></a>向 Key Vault 添加机密

只需执行几个步骤即可向保管库添加机密。 在此示例中，请添加可供应用程序使用的密码。 此密码名为 **ExamplePassword**，其中存储的值为 **hVFkk965BuUv**。

先键入以下内容，将 **hVFkk965BuUv** 值转换成安全字符串：

```azurepowershell-interactive
$secretvalue = ConvertTo-SecureString 'hVFkk965BuUv' -AsPlainText -Force
```

然后键入以下 PowerShell 命令，在 Key Vault 中创建名为 **ExamplePassword** 且值为 **hVFkk965BuUv** 的机密：

```azurepowershell-interactive
$secret = Set-AzKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'ExamplePassword' -SecretValue $secretvalue
```

若要查看机密中包含的纯文本形式的值，请执行以下命令：

```azurepowershell-interactive
(Get-AzKeyVaultSecret -vaultName "Contosokeyvault" -name "ExamplePassword").SecretValueText
```

现在，你已创建 Key Vault 并存储和检索了机密。

## <a name="clean-up-resources"></a>清理资源

 本系列中的其他快速入门和教程是在本快速入门的基础上制作的。 如果打算继续使用其他快速入门和教程，则可能需要保留这些资源。

如果不再需要资源组、Key Vault 和所有相关的资源，可以使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurepowershell-interactive
Remove-AzResourceGroup -Name ContosoResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已创建 Key Vault 并在其中存储了一个软件密钥。 若要详细了解 Key Vault 以及如何将它与应用程序配合使用，请继续学习有关如何将 Web 应用程序与 Key Vault 配合使用的教程。

若要了解如何使用 Web 应用程序通过 Azure 资源的托管标识从 Key Vault 读取机密，请继续学习以下教程

> [!div class="nextstepaction"]
> [将 Azure Web 应用程序配置为从 Key Vault 读取机密](quick-create-net.md)。
