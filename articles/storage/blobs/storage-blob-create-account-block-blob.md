---
title: 创建 Azure 存储的块 blob 存储帐户-|Microsoft Docs
description: 演示如何创建具有高级性能特征的 Azure 块 blob 存储帐户。
author: tamram
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 03/23/2019
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 9d8fb8f5f470dc47088efb30b7f823a0b8c624c8
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141001"
---
# <a name="create-a-block-blob-storage-account"></a>创建块 Blob 存储帐户

块 blob 存储帐户类型，可以使用高级性能特征创建块 blob。 针对具有高事务率的工作负荷优化此类型的存储帐户，或者要求非常快的访问时间。 本文介绍如何使用 Azure 门户、 Azure CLI 或 Azure PowerShell 创建块 blob 存储帐户。

有关块 blob 存储帐户的详细信息，请参阅[Azure 存储帐户概述](https://docs.microsoft.com/azure/storage/common/storage-account-overview)。

## <a name="create-account-in-the-azure-portal"></a>在 Azure 门户中创建帐户

若要在 Azure 门户中创建块 blob 存储帐户，请执行以下步骤：

1. 在 Azure 门户中，选择**所有服务**>**存储**类别 >**存储帐户**。

1. 下**存储帐户**，选择**添加**。

1. 在中**订阅**字段中，选择要在其中创建存储帐户的订阅。

1. 在中**资源组**字段中，选择现有资源组或选择**新建**，并输入新的资源组的名称。

1. 在中**存储帐户名称**字段中，输入帐户的名称。 请注意以下准则：

   - 名称必须是唯一的 Azure。
   - 名称必须介于 3 和 24 个字符。
   - 该名称可以包含仅限数字和小写字母。

1. 在中**位置**字段中，选择存储帐户的位置或使用默认位置。

1. 对于其余设置，进行下列配置：

   |字段     |值  |
   |---------|---------|
   |**性能**    |  选择**高级**。   |
   |**帐户种类**    | 选择**BlockBlobStorage**。      |
   |**复制**    |  保留默认设置**本地冗余存储 (LRS)**。      |

   ![显示门户 UI 创建块 blob 存储帐户](media/storage-blob-create-account-block-blob/create-block-blob-storage-account.png)

1. 选择**查看 + 创建**若要查看存储帐户设置。

1. 选择“创建”。

## <a name="create-account-using-azure-powershell"></a>使用 Azure PowerShell 创建帐户

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. 打开提升的 Windows PowerShell 会话 （以管理员身份运行）。

1. 运行以下命令以确保最新版本的`Az`安装 PowerShell 模块。

   ```powershell
   Install-Module -Name Az -AllowClobber
   ```

1. 打开一个新的 PowerShell 控制台和使用 Azure 帐户登录。

   ```powershell
   Connect-AzAccount -SubscriptionId <SubscriptionID>
   ```

1. 如果需要创建新的资源组。 替换引号中的值并运行以下命令。

   ```powershell
   $resourcegroup = "new_resource_group_name"
   $location = "region_name"
   New-AzResourceGroup -Name $resourceGroup -Location $location
   ```

1. 创建块 blob 存储帐户。 替换引号中的值并运行以下命令。

   ```powershell
   $resourcegroup = "resource_group_name"
   $storageaccount = "new_storage_account_name"
   $location = "region_name"

   New-AzStorageAccount -ResourceGroupName $resourcegroup -Name $storageaccount -Location $location -Kind "BlockBlobStorage" -SkuName "Premium_LRS"
   ```

## <a name="create-account-using-azure-cli"></a>使用 Azure CLI 创建帐户

若要使用 Azure CLI 创建块 blob 帐户，必须首先安装 Azure CLI v。 2.0.46 或更高版本。 运行 `az --version` 即可查找版本。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

1. 登录到 Azure 订阅。

   ```azurecli
   az login
   ```

1. 如果需要创建新的资源组。 替换为中括号内 （包括括号） 的值并运行以下命令。

   ```azurecli
   az group create \
    --name "<new_resource_group_name>" \
    --location "<location>"
   ```

1. 创建块 blob 存储帐户。 替换为中括号内 （包括括号） 的值并运行以下命令。

   ```azurecli
   az storage account create \
    --location "<location>" \
    --name "<new_storage_account_name>" \
    --resource-group "<resource_group_name>" \
    --kind "BlockBlobStorage" \
    --sku "Premium_LRS"
   ```

## <a name="next-steps"></a>后续步骤

- 有关存储帐户的详细信息，请参阅 [Azure 存储帐户概述](https://docs.microsoft.com/azure/storage/common/storage-account-overview)。

- 有关资源组的详细信息，请参阅 [Azure 资源管理器概述](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。
