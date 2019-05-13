---
title: Azure Key Vault 托管存储帐户 - CLI
description: 存储帐户密钥在 Azure Key Vault 与 Azure 存储帐户基于密钥的访问方式之间提供无缝集成。
ms.topic: conceptual
services: key-vault
ms.service: key-vault
author: msmbaldwin
ms.author: mbaldwin
manager: barbkess
ms.date: 03/01/2019
ms.openlocfilehash: 190375700f65cf2d3ea47335a646562eb46b2d49
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232560"
---
# <a name="azure-key-vault-managed-storage-account---cli"></a>Azure Key Vault 托管存储帐户 - CLI

> [!NOTE]
> [Azure 存储集成与 Azure Active Directory (Azure AD)] 是 Microsoft 的基于云的标识和访问管理服务。 Azure AD 的集成仅适用于 Blob 和队列服务。(https://docs.microsoft.com/azure/storage/common/storage-auth-aad). 我们建议使用 Azure AD 进行身份验证和授权。与 Azure Key Vault 一样，Azure AD 支持使用 OAuth2 令牌访问 Azure 存储。 这样，便可以：
> - 使用应用程序标识或用户标识（而不是存储帐户凭据）对客户端应用程序进行身份验证。 
> - 在 Azure 上运行时使用 [Azure AD 托管标识](/azure/active-directory/managed-identities-azure-resources/)。 托管标识完全消除了客户端身份验证的需要，并可以在应用程序中存储凭据，或者将凭据与应用程序一同存储。
> - 使用同样受 Key Vault 支持的基于角色的访问控制 (RBAC) 来管理授权。

[Azure 存储帐户](/azure/storage/storage-create-storage-account)使用由帐户名和密钥构成的凭据。 密钥会自动生成，并主要充当“密码”而不是加密密钥。 Key Vault 可以通过将这些存储帐户密钥存储为 [Key Vault 机密](/azure/key-vault/about-keys-secrets-and-certificates#key-vault-secrets)来对其进行管理。 

## <a name="overview"></a>概述

Key Vault 托管存储帐户功能代你执行多种管理功能：

- 列出（同步）Azure 存储帐户的密钥。
- 定期重新生成（轮换）密钥。
- 管理存储帐户和经典存储帐户的密钥。
- 响应调用方时永远不会返回密钥值。

使用托管存储帐户密钥功能时：

- **只允许 Key Vault 管理存储帐户密钥。** 不要尝试自行管理这些密钥，否则会干扰 Key Vault 的流程。
- **不要允许多个 Key Vault 对象管理存储帐户密钥。**
- **不要手动重新生成存储帐户密钥。** 我们建议通过 Key Vault 重新生成这些密钥。
- 要求 Key Vault 来管理你的存储帐户可以通过暂时用户主体和服务主体

以下示例演示如何允许 Key Vault 管理存储帐户密钥。

> [!IMPORTANT]
> Azure AD 租户为每个已注册的应用程序提供充当应用程序标识的[服务主体](/azure/active-directory/develop/developer-glossary#service-principal-object)。 通过基于角色的访问控制 (RBAC) 授权该服务主体访问其他 Azure 资源时，将使用它的应用程序 ID。 由于 Key Vault 是一个 Microsoft 应用程序，因此它已预先注册到每个 Azure 云内同一个应用程序 ID 下的所有 Azure AD 租户中：
> - Azure 政府云中的 Azure AD 租户使用应用程序 ID `7e7c393b-45d0-48b1-a35e-2905ddf8183c`。
> - Azure 公有云及所有其他云中的 Azure AD 租户使用应用程序 ID `cfa8b339-82a2-471a-a3c9-0fc0be7a4093`。

<a name="prerequisites"></a>必备组件
--------------
1. [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 安装 Azure CLI   
2. [创建存储帐户](https://azure.microsoft.com/services/storage/)
    - 请按照此[文档](https://docs.microsoft.com/azure/storage/)中的步骤创建一个存储帐户  
    - **命名指南：** 存储帐户名称必须为 3 到 24 个字符，并且只能包含数字和小写字母。        
      
<a name="step-by-step-instructions-on-how-to-use-key-vault-to-manage-storage-account-keys"></a>有关如何使用 Key Vault 管理存储帐户密钥的分步说明
--------------------------------------------------------------------------------
从概念上讲，步骤列表包括：
- 首先获取（预先存在的）存储帐户
- 然后提取（预先存在的）Key Vault
- 然后在保管库中添加 KeyVault 管理的存储帐户，将 Key1 设置为活动密钥，其重新生成期限为 180 天
- 最后，使用 Key1 为指定的存储帐户设置存储上下文

在以下说明中，我们将 Key Vault 分配为服务，以便为存储帐户授予操作员权限

> [!NOTE]
> 请注意，设置了 Azure Key Vault 托管存储帐户密钥后，除了通过 Key Vault 进行更改外，这些密钥应**不**再进行更改。 托管存储帐户密钥意味着 Key Vault 将管理存储帐户密钥的轮换

> [!IMPORTANT]
> Azure AD 租户为每个已注册的应用程序提供充当应用程序标识的[服务主体](/azure/active-directory/develop/developer-glossary#service-principal-object)。 通过基于角色的访问控制 (RBAC) 授权该服务主体访问其他 Azure 资源时，将使用它的应用程序 ID。 由于 Key Vault 是一个 Microsoft 应用程序，因此它已预先注册到每个 Azure 云内同一个应用程序 ID 下的所有 Azure AD 租户中：
> - Azure 政府云中的 Azure AD 租户使用应用程序 ID `7e7c393b-45d0-48b1-a35e-2905ddf8183c`。
> - Azure 公有云及所有其他云中的 Azure AD 租户使用应用程序 ID `cfa8b339-82a2-471a-a3c9-0fc0be7a4093`。

> - 目前可以使用用户主体要求 Key Vault 管理存储帐户和服务主体


1. 创建存储帐户后，运行以下命令来获取要管理的存储帐户的资源 ID

    ```
    az storage account show -n storageaccountname 
    ```
    复制上面的命令，如下面所示的结果超出 ID 字段
    ```
    /subscriptions/0xxxxxx-4310-48d9-b5ca-0xxxxxxxxxx/resourceGroups/ResourceGroup/providers/Microsoft.Storage/storageAccounts/StorageAccountName
    ```
            "objectId": "93c27d83-f79b-4cb2-8dd4-4aa716542e74"
    
2. 将"存储帐户密钥操作员服务角色"的 RBAC 角色分配到密钥保管库，限制到存储帐户的访问作用域。 对于经典存储帐户，使用"经典存储帐户密钥操作员服务角色。"
    ```
    az role assignment create --role "Storage Account Key Operator Service Role"  --assignee-object-id <ObjectIdOfKeyVault> --scope 93c27d83-f79b-4cb2-8dd4-4aa716542e74
    ```
    
    "93c27d83-f79b-4cb2-8dd4-4aa716542e74"是为公有云中的密钥保管库的对象 ID。 若要获取国家/地区云中的密钥保管库的对象 ID，请参阅上面的重要部分
    
3. 创建 Key Vault 托管存储帐户。     <br /><br />
   下面，我们将重新生成周期设置为 90 天。 90 天后，Key Vault 将重新生成“key1”，并将活动密钥从“key2”交换为“key1”。 它会立即将 Key1 标记为活动密钥。 
   
    ```
    az keyvault storage add --vault-name <YourVaultName> -n <StorageAccountName> --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id <Id-of-storage-account>
    ```

<a name="step-by-step-instructions-on-how-to-use-key-vault-to-create-and-generate-sas-tokens"></a>有关如何使用 Key Vault 创建和生成 SAS 令牌的分步说明
--------------------------------------------------------------------------------
也可以要求 Key Vault 生成 SAS（共享访问签名）令牌。 共享访问签名对存储帐户中的资源提供委托访问。 通过 SAS，可以授予客户端对存储帐户中资源的访问权限，无需共享帐户密钥。 这是在应用程序中使用共享访问签名的关键之处 - SAS 是用于共享存储资源的一种安全方式，它不会危及帐户密钥。

完成上面所列的步骤后，可运行以下命令要求 Key Vault 为你生成 SAS 令牌。 

在以下步骤中完成的任务列表包括：
- 设置一个帐户 SAS 定义名为`<YourSASDefinitionName>`密钥保管库托管存储帐户上`<YourStorageAccountName>`保管库中`<VaultName>`。 
- 为 Blob、文件、表和队列服务以及“服务”、“容器”和“对象”资源类型创建拥有所有权限、通过 HTTPS 访问并指定了开始和结束日期的帐户 SAS 令牌
- 在保管库中设置 KeyVault 管理的存储 SAS 定义，其模板 URI 为上面创建的 SAS 令牌，SAS 类型为“帐户”，有效期为 N 天
- 从对应于 SAS 定义的 KeyVault 机密中检索实际访问令牌

1. 此步骤将创建 SAS 定义。 创建此 SAS 定义后，可以要求 Key Vault 为你生成更多的 SAS 令牌。 此操作需要 storage/setsas 权限。

```
$sastoken = az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name storageacct --account-key 00000000
```
可在[此处](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-generate-sas)查看有关上述操作的更多帮助

成功运行此操作后，应会看到如下所示的输出。 请复制该输出

```console
   "se=2020-01-01&sp=***"
```

1. 此步骤使用上面生成的输出 ($sasToken) 创建 SAS 定义。 [此处](https://docs.microsoft.com/cli/azure/keyvault/storage/sas-definition?view=azure-cli-latest#required-parameters)提供了更多文档   

```
az keyvault storage sas-definition create --vault-name <YourVaultName> --account-name <YourStorageAccountName> -n <NameOfSasDefinitionYouWantToGive> --validity-period P2D --sas-type account --template-uri $sastoken
```
                        

 > [!NOTE] 
 > 如果用户对存储帐户没有权限，我们会先获取用户的对象 ID

 ```
 az ad user show --upn-or-object-id "developer@contoso.com"

 az keyvault set-policy --name <YourVaultName> --object-id <ObjectId> --storage-permissions backup delete list regeneratekey recover     purge restore set setsas update
 ```
    
## <a name="fetch-sas-tokens-in-code"></a>在代码中提取 SAS 令牌

在本部分中，我们将讨论如何通过从密钥保管库中提取 [SAS 令牌](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)对存储帐户执行操作

以下部分演示在按前文所述创建 SAS 定义之后，如何提取 SAS 令牌。

> [!NOTE]
>   有 3 种方式可用于向密钥保管库进行身份验证，如[基本概念](key-vault-whatis.md#basic-concepts)中所述
> - 使用托管服务标识（强烈推荐）
> - 使用服务主体和证书 
> - 使用服务主体和密码（不推荐）

```cs
// Once you have a security token from one of the above methods, then create KeyVaultClient with vault credentials
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(securityToken));

// Get a SAS token for our storage from Key Vault. SecretUri is of the format https://<VaultName>.vault.azure.net/secrets/<ExamplePassword>
var sasToken = await kv.GetSecretAsync("SecretUri");

// Create new storage credentials using the SAS token.
var accountSasCredential = new StorageCredentials(sasToken.Value);

// Use the storage credentials and the Blob storage endpoint to create a new Blob service client.
var accountWithSas = new CloudStorageAccount(accountSasCredential, new Uri ("https://myaccount.blob.core.windows.net/"), null, null, null);

var blobClientWithSas = accountWithSas.CreateCloudBlobClient();
```

如果 SAS 令牌即将过期，则你将再次从密钥保管库中提取 SAS 令牌并更新代码

```cs
// If your SAS token is about to expire, get the SAS Token again from Key Vault and update it.
sasToken = await kv.GetSecretAsync("SecretUri");
accountSasCredential.UpdateSASToken(sasToken);
```

### <a name="relevant-azure-cli-commands"></a>相关的 Azure CLI 命令

[Azure CLI 存储命令](https://docs.microsoft.com/cli/azure/keyvault/storage?view=azure-cli-latest)

## <a name="see-also"></a>另请参阅

- [关于键、密钥和证书](https://docs.microsoft.com/rest/api/keyvault/)
- [Key Vault 团队博客](https://blogs.technet.microsoft.com/kv/)
