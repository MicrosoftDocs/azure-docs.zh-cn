---
title: 对 Azure 存储使用 Azure PowerShell | Microsoft Docs
description: 了解如何对 Azure 存储使用 Azure PowerShell cmdlet。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/16/2018
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 082033cebc68fc97f7cff9ce80eb02acbbf5f4b0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145878"
---
# <a name="using-azure-powershell-with-azure-storage"></a>对 Azure 存储 使用 Azure PowerShell

Azure PowerShell 用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 针对 Azure 存储，将这些 cmdlet 划分为两个类别 -- 控制平面和数据平面。 控制平面 cmdlet 用于管理存储帐户，即创建存储帐户、设置属性、删除存储帐户、轮换访问密钥等。 数据平面 cmdlet 用于管理存储帐户中存储的数据。 例如，上传 blob、创建文件共享以及将消息添加到队列。

本操作说明文章介绍了使用管理平面 cmdlet 管理存储帐户的常见操作。 学习如何：

> [!div class="checklist"]
> * 列出存储器帐户
> * 获取对现有存储帐户的引用
> * 创建存储帐户
> * 设置存储帐户属性
> * 检索和再生成访问密钥
> * 保护对存储帐户的访问
> * 启用存储分析

本文提供有关存储的其他几篇 PowerShell 文章的链接，例如，如何启用和访问存储分析、如何使用数据平面 cmdlet，以及如何访问中国云、德国云和政府云等 Azure 独立云。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

本演练需要 Azure PowerShell 模块 Az 版本 0.7 或更高版本。 运行 `Get-Module -ListAvailable Az` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。

对于本演练，可以将命令键入到一个常规的 PowerShell 窗口中，也可以使用 [Windows PowerShell 集成脚本环境 (ISE)](/powershell/scripting/getting-started/fundamental/windows-powershell-integrated-scripting-environment--ise-) 并将命令键入到编辑器中，然后在浏览示例时测试一个或多个命令。 可以突出显示想要执行的行，并单击“运行所选项”来仅运行这些命令。

有关存储帐户的详细信息，请参阅[存储简介](storage-introduction.md)和[关于 Azure 存储帐户](storage-create-storage-account.md)。

## <a name="sign-in-to-azure"></a>登录 Azure

运行 `Connect-AzAccount` 命令以登录 Azure 订阅，并按照屏幕上的说明操作。

```powershell
Connect-AzAccount
```

## <a name="list-the-storage-accounts-in-the-subscription"></a>列出订阅中的存储帐户

运行 [Get-AzStorageAccount](/powershell/module/az.storage/Get-azStorageAccount) cmdlet 来检索当前订阅中的存储帐户列表。

```powershell
Get-AzStorageAccount | Select StorageAccountName, Location
```

## <a name="get-a-reference-to-a-storage-account"></a>获取对存储帐户的引用

接下来，需要对存储帐户的引用。 可以创建一个新存储帐户，也可以获取对现有存储帐户的引用。 下列各部分将演示这两种方法。

### <a name="use-an-existing-storage-account"></a>使用现有的存储帐户

若要检索现有的存储帐户，则需要资源组的名称和存储帐户的名称。 为这两个字段设置变量，然后使用 [Get-AzStorageAccount](/powershell/module/az.storage/Get-azStorageAccount) cmdlet。

```powershell
$resourceGroup = "myexistingresourcegroup"
$storageAccountName = "myexistingstorageaccount"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName
```

现在，你已有指向现有存储帐户的 $storageAccount。

### <a name="create-a-storage-account"></a>创建存储帐户

以下脚本将演示如何使用 [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount) 创建常规用途的存储帐户。 创建帐户后，检索其上下文，该操作可以在后续命令中使用，而不针对每次调用指定身份验证。

```powershell
# Get list of locations and select one.
Get-AzLocation | select Location
$location = "eastus"

# Create a new resource group.
$resourceGroup = "teststoragerg"
New-AzResourceGroup -Name $resourceGroup -Location $location

# Set the name of the storage account and the SKU name.
$storageAccountName = "testpshstorage"
$skuName = "Standard_LRS"

# Create the storage account.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName $skuName

# Retrieve the context.
$ctx = $storageAccount.Context
```

该脚本使用以下 PowerShell cmdlet：

*   [Get-AzLocation](/powershell/module/az.resources/get-azlocation) -- 检索有效位置的列表。 该示例使用 `eastus` 作为位置。

*   [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) -- 创建新资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 我们的资源组称为 `teststoragerg`。

*   [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) -- 创建存储帐户。 该示例使用 `testpshstorage`。

SKU 名称指示用于存储帐户的复制类型，如 LRS（本地冗余存储）。 有关复制的详细信息，请参阅 [Azure 存储复制](storage-redundancy.md)。

> [!IMPORTANT]
> 存储帐户的名称在 Azure 中是唯一的，并且必须采用小写。 有关命名约定和限制的信息，请参阅[命名和引用容器、Blob 和元数据](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata)。
>

现在，你有新的存储帐户以及对它的引用。

## <a name="manage-the-storage-account"></a>管理存储帐户

现在，已有对新存储帐户或现有存储帐户的引用，以下部分将介绍一些可用于管理存储帐户的命令。

### <a name="storage-account-properties"></a>存储帐户属性

若要更改存储帐户的设置，请使用 [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount)。 虽然无法更改存储帐户的位置或该帐户所在的资源组，但可以更改许多其他属性。 下面列出一些可使用 PowerShell 更改的属性。

* 分配给存储帐户的自定义域。

* 分配给存储帐户的标记。 标记通常用于分类资源以进行计费。

* SKU 是存储帐户的复制设置，例如 LRS（对于本地冗余存储）。 例如，可能会从标准\_LRS 更改为标准\_GRS 或标准\_RAGRS。 请注意，无法将 Standard\_ZRS 或 Premium\_LRS 更改为其他 SKU，反之亦然。

* Blob 存储帐户的访问层。 将访问层的值设置为“热”或“冷”，并允许用户通过选择符合存储帐户使用方式的访问层来最大限度地降低成本。 有关详细信息，请参阅[热、冷存储层和存档存储层](../blobs/storage-blob-storage-tiers.md)。

* 仅允许 HTTPS 流量。

### <a name="manage-the-access-keys"></a>管理访问密钥

Azure 存储帐户附带了两个帐户密钥。 若要检索密钥，请使用 [Get-AzStorageAccountKey](/powershell/module/az.Storage/Get-azStorageAccountKey)。 此示例将检索第一个密钥。 若要检索另一个密钥，请使用 `Value[1]` 而不是 `Value[0]`。

```powershell
$storageAccountKey = `
    (Get-AzStorageAccountKey `
    -ResourceGroupName $resourceGroup `
    -Name $storageAccountName).Value[0]
```

若要生成密钥，请使用 [Get-AzStorageAccountKey](/powershell/module/az.Storage/New-azStorageAccountKey)。

```powershell
New-AzStorageAccountKey -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -KeyName key1
```

若要再生成另一个密钥，请将 `key2` 用作密钥名称，而不是 `key1`。

再生成其中一个密钥，然后再次对其进行检索以查看新值。

> [!NOTE]
> 为生产存储帐户再生成密钥之前，应进行仔细的规划。 再生成一个或两个密钥将无法再访问使用已再生成密钥的任何应用程序。 有关详细信息，请参阅[访问密钥](storage-account-manage.md#access-keys)。


### <a name="delete-a-storage-account"></a>删除存储帐户

若要删除存储帐户，请使用 [Remove-AzStorageAccount](/powershell/module/az.storage/Remove-azStorageAccount)。

```powershell
Remove-AzStorageAccount -ResourceGroup $resourceGroup -AccountName $storageAccountName
```

> [!IMPORTANT]
> 在删除存储帐户时，还会删除该帐户中存储的所有资产。 如果意外删除某个帐户，请立即致电支持人员，并创建工单以还原该存储帐户。 不保证数据能得以恢复，但有时上述操作能起作用。 在支持工单得到解决之前，请不要使用相同的旧帐户名创建新的存储帐户。
>

### <a name="protect-your-storage-account-using-vnets-and-firewalls"></a>使用 VNet 和防火墙保护存储帐户

默认情况下，所有存储帐户均可通过任何有权访问 Internet 的网络进行访问。 但是，可以配置网络规则，仅允许来自特定虚拟网络的应用程序访问存储帐户。 有关详细信息，请参阅[配置 Azure 存储防火墙和虚拟网络](storage-network-security.md)。

本文将演示如何使用以下 PowerShell cmdlet 管理这些设置：
* [Add-AzStorageAccountNetworkRule](/powershell/module/az.Storage/Add-azStorageAccountNetworkRule)
* [Update-AzStorageAccountNetworkRuleSet](/powershell/module/az.storage/update-azstorageaccountnetworkruleset)
* [Remove-AzStorageAccountNetworkRule](https://docs.microsoft.com/powershell/module/az.storage/remove-azstorageaccountnetworkrule)

## <a name="use-storage-analytics"></a>使用存储分析  

[Azure 存储分析](storage-analytics.md)由[存储分析度量值](/rest/api/storageservices/about-storage-analytics-metrics)和[存储分析日志记录](/rest/api/storageservices/about-storage-analytics-logging)组成。

存储分析度量值用于收集 Azure 存储帐户的度量值，可用于监视存储帐户的运行情况。 可针对 blob、文件、表和队列启用度量值。

存储分析日志记录在服务器端执行，可用于记录对存储帐户的成功和失败请求的相关详细信息。 使用这些日志，可以查看针对表、队列和 Blob 的读取、写入和删除操作的详细信息，以及请求失败的原因。 日志记录不可用于 Azure 文件。

可以使用 [Azure 门户](https://portal.azure.com)或 PowerShell 配置监视，也可以使用存储客户端库以编程方式配置监视。

> [!NOTE]
> 可以使用 PowerShell 启用分钟分析。 此功能在门户中不可用。
>

* 若要了解如何使用 PowerShell 启用和查看存储指标数据，请参阅[存储分析指标](storage-analytics-metrics.md)。

* 若要了解如何使用 PowerShell 启用和检索存储日志记录数据，请参阅[如何使用 PowerShell 启用存储日志记录](/rest/api/storageservices/Enabling-Storage-Logging-and-Accessing-Log-Data)和[查找存储日志记录的日志数据](/rest/api/storageservices/Enabling-Storage-Logging-and-Accessing-Log-Data)。

* 有关使用“存储度量值”和“存储日志记录”排查存储问题的详细信息，请参阅[对 Microsoft Azure 存储进行监视、诊断和故障排除](storage-monitoring-diagnosing-troubleshooting.md)。

## <a name="manage-the-data-in-the-storage-account"></a>管理存储帐户中的数据

了解如何使用 PowerShell 管理存储帐户后，请参阅以下文章了解如何访问存储帐户中的数据对象。

* [如何使用 PowerShell 管理 blob](../blobs/storage-how-to-use-blobs-powershell.md)
* [如何使用 PowerShell 管理文件](../files/storage-how-to-use-files-powershell.md)
* [如何使用 PowerShell 管理队列](../queues/storage-powershell-how-to-use-queues.md)
* [使用 PowerShell 执行 Azure 表存储操作](../../storage/tables/table-storage-how-to-use-powershell.md)

Azure Cosmos DB 表 API 提供了用于表存储的高级功能，如统包全局分发、低延迟读取和写入、自动辅助索引和专用吞吐量。

* 有关详细信息，请参阅 [Azure Cosmos DB 表 API](../../cosmos-db/table-introduction.md)

## <a name="independent-cloud-deployments-of-azure"></a>Azure 的独立云部署

大多数人为其全球 Azure 部署使用了 Azure 公有云。 但出于主权等方面的原因，还存在一些独立的 Microsoft Azure 部署。 这些独立部署称为“环境”。 可用环境如下：

* [Azure 政府云](https://azure.microsoft.com/features/gov/)
* [由中国 21Vianet 运营的 azure 中国 21Vianet 云](http://www.windowsazure.cn/)
* [Azure 德国云](../../germany/germany-welcome.md)

有关如何使用 PowerShell 访问这些云及其存储的信息，请参阅[使用 PowerShell 管理 Azure 独立云中的存储](storage-powershell-independent-clouds.md)。

## <a name="clean-up-resources"></a>清理资源

如果在本练习中创建了新的资源组和存储帐户，可以通过删除资源组来删除所创建的所有资产。 这会一并删除组中包含的所有资源。 在这种情况下，它会删除创建的存储帐户以及资源组本身。

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```
## <a name="next-steps"></a>后续步骤

本操作说明文章介绍了使用管理平面 cmdlet 管理存储帐户的常见操作。 你已了解如何：

> [!div class="checklist"]
> * 列出存储器帐户
> * 获取对现有存储帐户的引用
> * 创建存储帐户
> * 设置存储帐户属性
> * 检索和再生成访问密钥
> * 保护对存储帐户的访问
> * 启用存储分析

本文还提供了其他几篇参考文章的链接，例如，如何管理数据对象、如何启用存储分析，以及如何访问中国云、德国云和政府云等 Azure 独立云。 下面是一些可供参考的其他相关文章和资源：

* [Azure 存储控制平面 PowerShell cmdlet](/powershell/module/az.storage/)
* [Azure 存储数据平面 PowerShell cmdlet](/powershell/module/azure.storage/)
* [Windows PowerShell Reference](https://msdn.microsoft.com/library/ms714469.aspx)（Windows PowerShell 参考）
