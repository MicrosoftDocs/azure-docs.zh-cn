---
title: 管理 Azure 存储生命周期
description: 了解如何创建生命周期策略规则，以将陈旧数据从热存储转移到冷存储和存档层。
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: conceptual
ms.date: 4/29/2019
ms.author: mhopkins
ms.reviewer: yzheng
ms.subservice: common
ms.openlocfilehash: 560f7eb8a8809cdd6ef410a610be9806f9709754
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409985"
---
# <a name="manage-the-azure-blob-storage-lifecycle"></a>管理 Azure Blob 存储生命周期

数据集具有独特的生命周期。 在生命周期的早期，人们经常访问某些数据。 但随着数据的老化，访问需求急剧下降。 有些数据在云中保持空闲状态，并且在存储后很少被访问。 有些数据在创建后的数日或者数月即会过期，还有一些数据集在其整个生存期会频繁受到读取和修改。 Azure Blob 存储生命周期管理提供 GPv2 和 Blob 存储帐户的丰富的、 基于规则的策略。 可使用该策略将数据转移到适当的访问层，或在数据的生命周期结束时使数据过期。

生命周期管理策略允许：

- 将 Blob 转移到较冷的存储层（从热到冷、从热到存档，或者从冷到存档），以便针对性能和成本进行优化
- 删除生命周期已结束的 Blob
- 在存储帐户级别定义每天运行一次的规则
- 将规则应用到容器或 Blob 子集（使用前缀作为筛选器）

假设某个数据集在生命周期的早期阶段频繁被访问，两周后只是偶尔被访问。 一个月以后，该数据集很少被访问。 在这种场景下，早期阶段最适合使用热存储。 在偶尔访问阶段最适合使用冷存储，在一个月后数据陈旧时，存档存储便是最佳的层选项。 通过根据数据陈旧程度调整存储层，可根据需求设计出最具性价比的存储选项。 若要实现这种过渡，可以使用生命周期管理策略规则将陈旧数据转移到较冷的存储层。

## <a name="storage-account-support"></a>存储帐户支持

生命周期管理策略适用于常规用途 v2 (GPv2) 帐户和 Blob 存储帐户。 只需在 Azure 门户中执行简单的一键式过程，即可将现有的常规用途 (GPv1) 帐户升级为 GPv2 帐户。 有关存储帐户的详细信息，请参阅 [Azure 存储帐户概述](../common/storage-account-overview.md)。  

## <a name="pricing"></a>定价 

生命周期管理功能是免费。 客户需要支付[列出 Blob](https://docs.microsoft.com/rest/api/storageservices/list-blobs) 和[设置 Blob 层](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier) API 调用的常规操作费用。 删除操作是免费的。 有关定价的详细信息，请参阅[块 Blob 定价](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="regional-availability"></a>区域可用性 
生命周期管理功能目前在所有公共 Azure 区域。 


## <a name="add-or-remove-a-policy"></a>添加或删除策略 

可以添加、 编辑或删除策略通过使用 Azure 门户中， [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)，Azure CLI [REST Api](https://docs.microsoft.com/rest/api/storagerp/managementpolicies)，或客户端工具。 本文介绍如何使用门户和 PowerShell 方法管理策略。  

> [!NOTE]
> 如果为存储帐户启用了防火墙规则，生命周期管理请求可能会被阻止。 可以通过提供例外来取消阻止这些请求。 需要绕过是： `Logging,  Metrics,  AzureServices`。 有关详细信息，请参阅[配置防火墙和虚拟网络](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)中的“例外”部分。

### <a name="azure-portal"></a>Azure 门户

1. 登录到 [Azure 门户](https://portal.azure.com)。

2. 选择**的所有资源**，然后选择你的存储帐户。

3. 下**Blob 服务**，选择**生命周期管理**若要查看或更改你的策略。

### <a name="powershell"></a>PowerShell

```powershell
#Install the latest module
Install-Module -Name Az -Repository PSGallery 

#Create a new action object

$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction Delete -daysAfterModificationGreaterThan 2555
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToArchive -daysAfterModificationGreaterThan 90
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToCool -daysAfterModificationGreaterThan 30
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -SnapshotAction Delete -daysAfterCreationGreaterThan 90

# Create a new filter object
# PowerShell automatically sets BlobType as “blockblob” because it is the only available option currently
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch ab,cd 

#Create a new rule object
#PowerShell automatically sets Type as “Lifecycle” because it is the only available option currently
$rule1 = New-AzStorageAccountManagementPolicyRule -Name Test -Action $action -Filter $filter

#Set the policy 
$policy = Set-AzStorageAccountManagementPolicy -ResourceGroupName $rgname -StorageAccountName $accountName -Rule $rule1

```
## <a name="arm-template-with-lifecycle-management-policy"></a>ARM 模板提供生命周期管理策略

您可以定义和部署 Azure 解决方案部署使用 ARM 模板的一部分的生命周期管理。 以下是示例模板来部署 RA-GRS GPv2 存储帐户与生命周期管理策略。 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "storageAccountName": "[uniqueString(resourceGroup().id)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-04-01",
      "sku": {
        "name": "Standard_RAGRS"
      },
      "kind": "StorageV2",
      "properties": {
        "networkAcls": {}
      }
    },
    {
      "name": "[concat(variables('storageAccountName'), '/default')]",
      "type": "Microsoft.Storage/storageAccounts/managementPolicies",
      "apiVersion": "2019-04-01",
      "dependsOn": [
        "[variables('storageAccountName')]"
      ],
      "properties": {
        "policy": {...}
      }
    }
  ],
  "outputs": {}
}
```

## <a name="policy"></a>策略

生命周期管理策略是 JSON 文档中的规则集合：

```json
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```


策略是一系列规则：

| 参数名称 | 参数类型 | 说明 |
|----------------|----------------|-------|
| 规则          | 规则对象的数组 | 在策略中需要至少一个规则。 您可以在策略中定义最多 100 个规则。|

每个规则在策略中的有几个参数：

| 参数名称 | 参数类型 | 说明 | 需要 |
|----------------|----------------|-------|----------|
| 名称           | String |规则名称可以包含最多 256 个字母数字字符。 规则名称区分大小写。  该名称必须在策略中唯一。 | True |
| enabled | Boolean | 可选的布尔值，以允许规则以将临时禁用。 默认值为 true，如果未设置。 | False | 
| 类型           | 枚举值 | 当前的有效类型是`Lifecycle`。 | True |
| 定义     | 定义生命周期规则的对象 | 每个定义均由筛选器集和操作集组成。 | True |

## <a name="rules"></a>规则

每个规则定义包括筛选器集和操作集。 [筛选器集](#rule-filters)将规则操作限制为容器或对象名称中的某组对象。 [操作集](#rule-actions)对筛选的对象集应用分层或删除操作。

### <a name="sample-rule"></a>示例规则
下面的示例规则筛选器的帐户存在于内的对象上运行操作`container1` **AND**开始`foo`。  

- 在上次修改后的 30 天后，将 Blob 分层到冷层
- 在上次修改后的 90 天后，将 Blob 分层到存档层
- 在上次修改后的 2,555 天（7 年）后，删除 Blob
- 在创建快照后的 90 天后，删除 Blob 快照

```json
{
  "rules": [
    {
      "name": "ruleFoo",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 2555 }
          },
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}

```

### <a name="rule-filters"></a>规则筛选器

筛选器将规则操作限制为存储帐户中的 Blob子集。 如果定义了多个筛选器，将对所有筛选器运行逻辑 `AND`。

有效的筛选器包括：

| 筛选器名称 | 筛选类型 | 说明 | 必需 |
|-------------|-------------|-------|-------------|
| blobTypes   | 预定义枚举值的数组。 | 当前版本支持`blockBlob`。 | “是” |
| prefixMatch | 要匹配的前缀字符串数组。 每个规则可以定义最多 10 个前缀。 前缀字符串必须以容器名称开头。 例如，如果要为某个规则匹配“https://myaccount.blob.core.windows.net/container1/foo/...”下的所有 Blob，则 prefixMatch 为 `container1/foo`。 | 如果未定义 prefixMatch，规则将适用于存储帐户中的所有 blob。  | “否” |

### <a name="rule-actions"></a>规则操作

当满足运行的条件操作应用于已筛选的 blob。

生命周期管理支持分层和删除的 blob 以及删除的 blob 快照。 在 Blob 或 Blob 快照中为每个规则至少定义一个操作。

| 操作        | 基本 Blob                                   | 快照      |
|---------------|---------------------------------------------|---------------|
| tierToCool    | 目前支持位于热层的 Blob         | 不支持 |
| tierToArchive | 目前支持位于热层或冷层的 Blob | 不支持 |
| delete        | 支持                                   | 支持     |

>[!NOTE] 
>如果在同一 Blob 中定义了多个操作，生命周期管理将对该 Blob 应用开销最低的操作。 例如，操作 `delete` 的开销比 `tierToArchive` 更低。 操作 `tierToArchive` 的开销比 `tierToCool` 更低。

运行的条件根据存在时间。 基本 Blob 使用上次修改时间来跟踪陈旧程度，Blob 快照使用快照创建时间来跟踪陈旧程度。

| 操作以运行条件 | 条件值 | 描述 |
|----------------------------|-----------------|-------------|
| daysAfterModificationGreaterThan | 指示陈旧程度（天）的整数值 | 基本 Blob 操作的有效条件 |
| daysAfterCreationGreaterThan     | 指示陈旧程度（天）的整数值 | Blob 快照操作的有效条件 | 

## <a name="examples"></a>示例
以下示例演示如何使用生命周期策略规则满足常见场景。

### <a name="move-aging-data-to-a-cooler-tier"></a>将陈旧的数据移到冷层

此示例演示如何转移前缀为 `container1/foo` 或 `container2/bar` 的块 Blob。 该策略将 30 天以上未修改的 Blob 转移到冷存储，并将 90 天未修改的 Blob 转移到存档层：

```json
{
  "rules": [
    {
      "name": "agingRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo", "container2/bar" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

### <a name="archive-data-at-ingest"></a>引入时存档数据 

某些数据在云中保持空闲状态，并且在存储后很少（如果有）被访问。 下面的生命周期策略被配置为将数据存档后引入。 此示例中转换的块 blob 容器中的存储帐户中`archivecontainer`到存档层。 转换通过对 blob 进行操作上次修改时间之后的 0 天：

```json
{
  "rules": [
    {
      "name": "archiveRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "archivecontainer" ]
        },
        "actions": {
          "baseBlob": {
              "tierToArchive": { "daysAfterModificationGreaterThan": 0 }
          }
        }
      }
    }
  ]
}

```

### <a name="expire-data-based-on-age"></a>基于陈旧程度使数据过期

某些数据应在创建后的数日或数月内过期，以降低成本或遵从政府的要求。 可以将生命周期管理策略配置为：根据数据陈旧程度删除数据，以使数据过期。 以下示例中演示的策略删除超过 365 天的所有块 Blob（未指定前缀）。

```json
{
  "rules": [
    {
      "name": "expirationRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

### <a name="delete-old-snapshots"></a>删除旧快照

对于在整个生存期内频繁修改和访问的数据，通常会使用快照来跟踪数据的旧版本。 可以创建一个策略，用于根据快照的陈旧程度删除旧快照。 可通过评估快照创建时间来确定快照的陈旧程度。 此策略规则删除容器 `activedata` 中自创建快照后达到或超过 90 天的块 Blob 快照。

```json
{
  "rules": [
    {
      "name": "snapshotRule",
      "enabled": true,
      "type": "Lifecycle",      
    "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "activedata" ]
        },
        "actions": {
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```
## <a name="faq"></a>常见问题解答 
**我创建了新的策略，原因操作不立即运行？**  
平台每天运行一次生命周期策略。 一旦配置策略时，它可能需要最多 24 小时的某些操作 （如分层和删除） 以运行适用于第一次。  

## <a name="next-steps"></a>后续步骤

了解如何在意外删除数据后恢复数据：

- [Azure 存储 blob 的软删除](../blobs/storage-blob-soft-delete.md)
