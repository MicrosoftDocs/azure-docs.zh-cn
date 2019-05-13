---
title: 配置 Azure CLI 从 Azure 存储加密的客户托管密钥
description: 了解如何使用 Azure CLI 配置用于 Azure 存储加密的客户托管密钥。 客户托管密钥，可以创建、 轮换、 禁用和撤消访问控制。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/16/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: ca2cfc9369fd6fb001b2a2dc401c33e5ddfd38c8
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142908"
---
# <a name="configure-customer-managed-keys-for-azure-storage-encryption-from-azure-cli"></a>配置 Azure CLI 从 Azure 存储加密的客户托管密钥

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

本文介绍如何使用客户托管密钥使用 Azure CLI 配置密钥保管库。

## <a name="assign-an-identity-to-the-storage-account"></a>将标识分配到存储帐户

若要启用客户托管密钥的存储帐户，首先将系统分配的托管的标识分配给存储帐户。 您将使用此托管的标识授予访问密钥保管库的存储帐户权限。

若要将分配托管的标识使用 Azure CLI，请调用[az 存储帐户更新](/cli/azure/storage/account#az-storage-account-update)。 请记住将括号中的占位符值替换为你自己的值。

```azurecli-interactive
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

有关使用 Azure CLI 配置系统分配的托管的标识的详细信息，请参阅[配置管理使用 Azure CLI 在 Azure VM 上的 Azure 资源的标识](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)。

## <a name="create-a-new-key-vault"></a>创建新的密钥保管库

使用 Azure 存储加密必须具有两个密钥保护设置已启用，将客户托管密钥的密钥保管库**软删除**并**不清除**。 若要创建新的 key vault 启用这些设置中使用 PowerShell 或 Azure CLI，请执行以下命令。 请记住将括号中的占位符值替换为你自己的值。 

若要创建新的 key vault 使用 Azure CLI，请调用[az keyvault 创建](/cli/azure/keyvault#az-keyvault-create)。 请记住将括号中的占位符值替换为你自己的值。

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-soft-delete \
    --enable-purge-protection
```

## <a name="configure-the-key-vault-access-policy"></a>配置密钥保管库访问策略

接下来，配置密钥保管库的访问策略，以便存储帐户有权访问它。 在此步骤中，将使用以前分配给存储帐户的托管的标识。

若要设置密钥保管库的访问策略，请调用[az keyvault 设置策略](/cli/azure/keyvault#az-keyvault-set-policy)。 请记住将括号中的占位符值替换为你自己的值。

```azurecli-interactive
storage_account_principal=$(az storage account show \
    -name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get recover unwrapKey wrapKey
```

## <a name="create-a-new-key"></a>新建密钥

接下来，在密钥保管库中创建一个密钥。 若要创建密钥，请调用[az keyvault 密钥创建](/cli/azure/keyvault/key#az-keyvault-key-create)。 请记住将括号中的占位符值替换为你自己的值。

```azurecli-interactive
az keyvault key create
    --name <key> \
    --vault-name <key-vault>
```

## <a name="configure-encryption-with-customer-managed-keys"></a>使用客户托管密钥配置加密

默认情况下，Azure 存储加密使用 Microsoft 托管密钥。 配置 Azure 存储帐户的客户托管密钥并指定要将存储帐户相关联的键。

若要更新存储帐户的加密设置，请调用[az 存储帐户更新](/cli/azure/storage/account#az-storage-account-update)。 此示例中也会查询密钥保管库 URI 和密钥版本，这两个的值需要将密钥与存储帐户相关联。 请记住将括号中的占位符值替换为你自己的值。

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update 
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

## <a name="update-the-key-version"></a>更新密钥的版本

创建密钥的新版本时，你将需要更新存储帐户以使用新版本。 首先，调用密钥保管库 URI 中查询[az keyvault show](/cli/azure/keyvault#az-keyvault-show)，并通过调用的密钥版本[az keyvault 密钥列表-版本](/cli/azure/keyvault/key#az-keyvault-key-list-versions)。 然后调用[az 存储帐户更新](/cli/azure/storage/account#az-storage-account-update)更新存储帐户的加密设置，以使用新版本的密钥，如在上一部分中所示。

## <a name="next-steps"></a>后续步骤

- [静态数据的 azure 存储加密](storage-service-encryption.md) 
- [什么是 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)？
