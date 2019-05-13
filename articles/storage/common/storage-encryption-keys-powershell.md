---
title: 配置为从 PowerShell 的 Azure 存储加密的客户托管密钥
description: 了解如何使用 PowerShell 来配置用于 Azure 存储加密的客户托管密钥。 客户托管密钥，可以创建、 轮换、 禁用和撤消访问控制。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/16/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: be876b370cd476bee2af7d90a9f0433fd80de3b4
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233677"
---
# <a name="configure-customer-managed-keys-for-azure-storage-encryption-from-powershell"></a>配置为从 PowerShell 的 Azure 存储加密的客户托管密钥

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

本文介绍如何使用客户托管密钥使用 PowerShell 配置密钥保管库。

## <a name="assign-an-identity-to-the-storage-account"></a>将标识分配到存储帐户

若要启用客户托管密钥的存储帐户，首先将系统分配的托管的标识分配给存储帐户。 您将使用此托管的标识授予访问密钥保管库的存储帐户权限。

若要将分配使用 PowerShell 托管的标识，请调用[集 AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount)。 请记住将括号中的占位符值替换为你自己的值。

```powershell
$storageAccount = Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -Name <storage-account> `
    -AssignIdentity
```

有关使用 PowerShell 配置系统分配的托管的标识的详细信息，请参阅[配置托管在使用 PowerShell 为 Azure VM 上的 Azure 资源的标识](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)。

## <a name="create-a-new-key-vault"></a>创建新的密钥保管库

若要创建新的 key vault 使用 PowerShell，请调用[新建 AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault)。 使用 Azure 存储加密必须具有两个密钥保护设置已启用，将客户托管密钥的密钥保管库**软删除**并**不清除**。 

请记住将括号中的占位符值替换为你自己的值。 

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnableSoftDelete `
    -EnablePurgeProtection
```

## <a name="configure-the-key-vault-access-policy"></a>配置密钥保管库访问策略

接下来，配置密钥保管库的访问策略，以便存储帐户有权访问它。 在此步骤中，将使用以前分配给存储帐户的托管的标识。

若要设置密钥保管库的访问策略，请调用[集 AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy)。 请记住将括号中的占位符值替换为你自己的值，然后使用在前面的示例中定义的变量。

```powershell
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get,recover
```

## <a name="create-a-new-key"></a>新建密钥

接下来，在密钥保管库中创建新的密钥。 若要创建新的密钥，请调用[添加 AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey)。 请记住将括号中的占位符值替换为你自己的值，然后使用在前面的示例中定义的变量。

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName -Name <key> -Destination 'Software'
```

## <a name="configure-encryption-with-customer-managed-keys"></a>使用客户托管密钥配置加密

默认情况下，Azure 存储加密使用 Microsoft 托管密钥。 在此步骤中，配置 Azure 存储帐户，以使用客户托管密钥，并指定要将存储帐户相关联的键。

调用[集 AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount)更新存储帐户的加密设置。 请记住将括号中的占位符值替换为你自己的值，然后使用在前面的示例中定义的变量。

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultUri $keyVault.VaultUri
```

## <a name="update-the-key-version"></a>更新密钥的版本

创建密钥的新版本时，你将需要更新存储帐户以使用新版本。 首先，调用[Get AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey)以获取最新版本的密钥。 然后调用[集 AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount)更新存储帐户的加密设置，以使用新版本的密钥，如在上一部分中所示。

## <a name="next-steps"></a>后续步骤

- [静态数据的 azure 存储加密](storage-service-encryption.md) 
- [什么是 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)？
