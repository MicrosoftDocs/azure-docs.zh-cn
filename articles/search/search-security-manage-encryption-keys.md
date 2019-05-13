---
title: 静态加密 Azure 密钥保管库-Azure 搜索中使用客户托管密钥
description: 对索引和 Azure 搜索中通过创建和管理 Azure 密钥保管库中的键的同义词映射的补充程序服务器端加密。
author: NatiNimni
manager: jlembicz
ms.author: natinimn
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: ''
ms.openlocfilehash: 987b56a9571fd50f605dbe6fb4112ef857021530
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029170"
---
# <a name="azure-search-encryption-using-customer-managed-keys-in-azure-key-vault"></a>使用 Azure Key Vault 中客户托管密钥的 azure 搜索加密

默认情况下，Azure 搜索用户对内容进行加密与静态[服务托管密钥](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest#data-encryption-models)。 可以使用密钥的创建和管理 Azure 密钥保管库中的其他加密层补充默认的加密。 本文将指导您完成的步骤。

通过与集成支持服务器端加密[Azure 密钥保管库](https://docs.microsoft.com/azure/key-vault/key-vault-overview)。 你可以创建自己的加密密钥并将其存储在 Key Vault 中，或使用 Azure Key Vault 的 API 来生成加密密钥。 使用 Azure 密钥保管库，还可以审核密钥用法。 

使用客户托管密钥的加密级别的索引或同义词映射时创建这些对象，而不搜索服务级别配置。 无法加密已存在的内容。 

可以使用不同的密钥保管库中的不同密钥。 这意味着单个搜索服务可以托管多个加密的 indexes\synonym 映射，每个可能使用不同客户托管密钥，与使用客户托管密钥未加密的 indexes\synonym 映射一起加密。 

>[!Note]
> **功能可用性**:使用客户托管密钥的加密是一个预览功能，不适用于免费服务。 对于付费服务，它功能仅适用于搜索服务创建当天或之后 2019年-01-01，使用最新预览版 api-version (api 版本 = 2019年-05-06-预览)。 目前没有任何门户支持此功能。

## <a name="prerequisites"></a>必备组件

在此示例中使用了以下服务。 

[创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可在本教程中使用免费服务。

[创建 Azure 密钥保管库资源](https://docs.microsoft.com/azure/key-vault/quick-create-portal#create-a-vault)或查找你的订阅下现有的保管库。

[Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)或[Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)用于配置任务。

[Postman](search-fiddler.md)， [Azure PowerShell](search-create-index-rest-api.md)并[Azure 搜索 SDK](https://aka.ms/search-sdk-preview)可用于调用的预览版 REST API。 没有门户或.NET SDK 支持这一次在客户托管的加密。

## <a name="1---enable-key-recovery"></a>1-启用密钥恢复

此步骤是可选但强烈推荐。 创建 Azure 密钥保管库资源后，启用**软删除**并**清除保护**通过执行以下 PowerShell 或 Azure CLI 命令在所选密钥保管库：   

```powershell
$resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "<vault_name>").ResourceId

$resource.Properties | Add-Member -MemberType NoteProperty -Name "enableSoftDelete" -Value 'true'

$resource.Properties | Add-Member -MemberType NoteProperty -Name "enablePurgeProtection" -Value 'true'

Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
```

```azurecli-interactive
az keyvault update -n <vault_name> -g <resource_group> --enable-soft-delete --enable-purge-protection
```

>[!Note]
> 由于使用客户托管密钥功能加密的本质，Azure 搜索将不能检索数据，如果删除你的 Azure 密钥保管库密钥。 若要防止引起意外的密钥保管库密钥删除的数据丢失，强烈建议在所选密钥保管库上启用软删除和清除保护。 有关详细信息，请参阅[Azure Key Vault 软删除](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete)。   

## <a name="2---create-a-new-key"></a>2-创建新的密钥

如果使用现有密钥对 Azure 搜索内容进行加密，请跳过此步骤。

1. [登录到 Azure 门户](https://portal.azure.com)并导航到密钥保管库仪表板。

1. 选择**密钥**从左侧的导航窗格中，设置并单击 **+ 生成/导入**。

1. 在“创建密钥”窗格中，从“选项”列表中，选择要用于创建密钥的方法。 可以**生成**新密钥、**上传**现有密钥，或使用“备份还原”选择密钥的备份。

1. 输入**名称**你的密钥，并且根据需要选择其他键属性。

1. 单击**创建**按钮以开始部署。

请记下的密钥标识符 – 这由组成**键值 Uri**，则**密钥名称**，并**密钥版本**。 你将需要这些元素来定义 Azure 搜索中的加密的索引。
 
![创建新的密钥保管库密钥](./media/search-manage-encryption-keys/create-new-key-vault-key.png "创建新的密钥保管库密钥")

## <a name="3---create-a-service-identity"></a>3-创建服务标识

将标识分配给你的搜索服务，你可以授予对你的搜索服务的密钥保管库访问权限。 搜索服务将使用其标识使用 Azure 密钥保管库进行身份验证。

Azure 搜索支持两种方法将分配标识的： 托管的标识或外部托管 Azure Active Directory 应用程序。 

如果可能，请使用托管的标识。 它是将标识分配给你的搜索服务的最简单的方法，在大多数情况下即可正常工作。 如果使用多个键的索引和同义词映射，或者如果你的解决方案是在分布式体系结构中，将使基于标识的身份验证，使用高级[外部托管的 Azure Active Directory 方法](#aad-app)末尾的这篇文章所述。

 一般情况下，托管的标识可以让您搜索服务对 Azure 密钥保管库在代码中存储的凭据进行身份验证。 这种类型的托管标识的生命周期绑定到你的搜索服务，只能有一个托管的标识的生命周期。 [了解有关托管标识的详细信息](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)。

1. 若要创建托管的标识[登录到 Azure 门户](https://portal.azure.com)并打开搜索服务仪表板。 

1. 单击**标识**的左侧的导航窗格中，更改其状态设置为**上**，然后单击**保存**。

![启用托管的标识](./media/search-enable-msi/enable-identity-portal.png "启用托管的标识")

## <a name="4---grant-key-access-permissions"></a>4-授予密钥访问权限

若要启用你的搜索服务以使用你的密钥保管库密钥，你将需要授予你的搜索服务特定的访问权限。

无法在任何给定时间撤销访问权限。 一旦被吊销，使用该密钥保管库任何搜索服务索引或同义词映射将变得不可用。 在以后还原密钥保管库访问权限将还原 index\synonym 映射访问。 有关详细信息，请参阅[保护对密钥保管库的访问](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)。

1. [登录到 Azure 门户](https://portal.azure.com)并打开密钥保管库概述页。 

1. 选择**访问策略**从左侧的导航窗格中，设置并单击 **+ 添加新**。

   ![添加新的密钥保管库访问策略](./media/search-manage-encryption-keys/add-new-key-vault-access-policy.png "添加新的密钥保管库访问策略")

1. 单击**选择主体**，然后选择你的 Azure 搜索服务。 您可以搜索它按名称或启用托管的标识后显示的对象 ID。

   ![选择密钥保管库访问策略主体](./media/search-manage-encryption-keys/select-key-vault-access-policy-principal.png "选择密钥保管库访问策略主体")

1. 单击**密钥权限**，然后选择*获取*， *Unwrap Key*并*Wrap Key*。 可以使用*Azure Data Lake 存储或 Azure 存储*模板以快速选择所需的权限。

   Azure 搜索必须授予以下[访问权限](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-operations):

   * *获取*-允许你搜索的服务来检索 Key Vault 中密钥的公共部分
   * *包装密钥*-允许搜索服务以使用你的密钥来保护内部加密密钥
   * *解包密钥*-允许搜索服务以使用你的密钥进行解包的内部加密密钥

   ![选择密钥保管库访问策略密钥权限](./media/search-manage-encryption-keys/select-key-vault-access-policy-key-permissions.png "选择密钥保管库访问策略密钥权限")

1. 单击**确定**并**保存**访问策略更改。

> [!Important]
> Azure 搜索中的加密的内容配置为使用特定的 Azure 密钥保管库密钥与特定**版本**。 如果更改密钥或版本，必须更新索引或同义词映射以使用新 key\version**之前**删除以前 key\version。 如果不这样做将呈现索引或同义词映射不可用，你将不能以后密钥访问权限都将丢失对内容进行解密。   

## <a name="5---encrypt-content"></a>5-加密内容

创建索引或同义词映射使用客户托管密钥加密尚不可能使用 Azure 门户。 使用 Azure 搜索 REST API 创建此类的索引或同义词映射。

索引和同义词映射支持一个新的顶级**encryptionKey**属性用于指定的密钥。 

使用**密钥保管库 Uri**，**密钥名称**并**密钥版本**的密钥保管库密钥，我们可以创建**encryptionKey**定义：

```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
  }
}
```
> [!Note] 
> 没有任何这些密钥保管库详细信息被视为机密，可以轻松地检索通过浏览到 Azure 门户中的相关 Azure 密钥保管库密钥页。

如果将 AAD 应用程序用于 Key Vault 身份验证而不是使用托管的标识，将添加的 AAD 应用程序**访问凭据**到你的加密密钥： 
```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

## <a name="example-index-encryption"></a>示例：索引加密
在找不到创建新的索引通过 REST API 的详细信息[创建索引 (Azure 搜索服务 REST API)](https://docs.microsoft.com/rest/api/searchservice/create-index)，其中唯一的区别指定的加密密钥的详细信息，作为索引定义的一部分： 

```json
{
 "name": "hotels",  
 "fields": [
  {"name": "HotelId", "type": "Edm.String", "key": true, "filterable": true},
  {"name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false},
  {"name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene"},
  {"name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
  {"name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
  {"name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true},
  {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true},
  {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true},
 ],
 "encryptionKey": {
   "keyVaultUri": "https://demokeyvault.vault.azure.net",
   "keyVaultKeyName": "myEncryptionKey",
   "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
 }
}
```
您可以立即发送索引创建请求，并启动通常使用索引。

## <a name="example-synonym-map-encryption"></a>示例：同义词映射加密

创建新的同义词映射通过 REST API 的详细信息，请参阅[创建同义词映射 (Azure 搜索服务 REST API)](https://docs.microsoft.com/rest/api/searchservice/create-synonym-map)，其中唯一的区别指定的加密密钥的详细信息，作为同义词映射定义的一部分： 

```json
{   
  "name" : "synonymmap1",  
  "format" : "solr",  
  "synonyms" : "United States, United States of America, USA\n
  Washington, Wash. => WA",
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
  }
}
```
你可以现在将发送同义词映射创建请求，并启动正常使用。

>[!Important] 
> 虽然**encryptionKey**无法添加到现有的 Azure 搜索索引或同义词映射，它可能通过提供不同的值 （例如，更新密钥的版本） 的三个密钥保管库详细信息的任何更新。 更改为新的密钥保管库密钥或新的密钥版本时，将使用的密钥的任何 Azure 搜索索引或同义词映射必须首先更新为使用新 key\version**之前**删除以前 key\version。 如果不这样做将呈现索引或者不可用，因为它将无法解密内容一次密钥访问的同义词映射丢失。   
> 在更高版本时还原密钥保管库访问权限会还原内容的访问权限。

## <a name="aad-app"></a> 高级：使用外部托管的 Azure Active Directory 应用程序

当托管的标识不能，可以创建一个 Azure Active Directory 应用程序的安全主体的 Azure 搜索服务。 具体而言，托管的标识不是在这些情况下可行的：

* 您不能直接授予您的搜索服务对密钥保管库的访问权限 （例如，如果搜索服务是在不同的 Active Directory 租户与 Azure 密钥保管库中）。

* 单个搜索服务是以托管多个加密的 indexes\synonym 映射，每个使用不同的密钥从不同的密钥保管库，其中每个密钥保管库必须使用**另一标识**进行身份验证。 如果使用不同的标识管理不同的密钥保管库不是一项要求，请考虑使用上面的托管的标识选项。  

为了适应这种拓扑，Azure 搜索支持你的搜索服务和密钥保管库之间进行身份验证使用 Azure Active Directory (AAD) 应用程序。    
若要在门户中创建 AAD 应用程序：

1. [创建 Azure Active Directory 应用程序](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application)。

1. [获取应用程序 ID 和身份验证密钥](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key)因为那些需要用于创建加密的索引。 将需要提供的值包括**应用程序 ID**并**身份验证密钥**。

>[!Important]
> 在决定要使用 AAD 应用程序的身份验证而不是托管标识，请考虑这一事实的未授权 Azure 搜索管理你的代表 AAD 应用程序，并将由你管理 AAD 应用程序，如定期轮换的应用程序身份验证密钥。
> 时更改 AAD 应用程序或其身份验证密钥，任何使用该应用程序的 Azure 搜索索引或同义词映射必须首先更新为使用新的应用程序 ID\key**之前**删除以前的应用程序或其授权密钥，和之前撤消您对它的密钥保管库访问权限。
> 如果不这样做将呈现索引或者不可用，因为它将无法解密内容一次密钥访问的同义词映射丢失。   

## <a name="next-steps"></a>后续步骤

如果您不熟悉 Azure 安全体系结构，请查看[Azure 安全文档](https://docs.microsoft.com/azure/security/)，和具体而言，这篇文章：

> [!div class="nextstepaction"]
> [静态数据加密](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest)
