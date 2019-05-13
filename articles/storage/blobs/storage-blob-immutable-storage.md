---
title: Azure 存储 Blob 的不可变存储 | Microsoft Docs
description: Azure 存储提供对 Blob 对象存储的 WORM（一次写入，多次读取）支持，可让用户根据指定的时间间隔，以不可擦除、不可修改的状态存储数据。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 05/06/2019
ms.author: tamram
ms.reviewer: hux
ms.subservice: blobs
ms.openlocfilehash: 60cf37e5f6375d08e73241f6e357ac39ea665e9b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192540"
---
# <a name="store-business-critical-data-in-azure-blob-storage"></a>在 Azure Blob 存储中存储业务关键型数据

Azure Blob 存储的不可变存储使用户能够将业务关键数据对象存储在蠕虫 （写入一次，多次读取） 状态。 此状态可以根据用户指定的时间间隔使数据保持不可擦除且不可修改的状态。 Blob 对象可以创建和读取，但不是修改或删除，在保留间隔持续时间。 为常规用途 v2 和所有 Azure 区域中的 Blob 存储帐户启用了不可变的存储。

## <a name="overview"></a>概述

不可变的存储可以帮助医疗保健组织、 金融机构和相关的行业-尤其是 broker 庄家组织-安全地存储数据。 此外可以利用在任何情况下，若要禁止对关键数据进行修改或删除操作。 

典型的应用程序包含：

- **法规遵从**：Azure Blob 存储的不可变存储可帮助组织达到 SEC 17a-4(f)、CFTC 1.31(d)、FINRA 和其他法规要求。 通过 Cohasset 将相关联的技术白皮书详细介绍了不可变的存储地址，这些监管要求是通过可下载[Microsoft 服务信任门户](https://aka.ms/AzureWormStorage)。 [Azure 信任中心](https://www.microsoft.com/trustcenter/compliance/compliance-overview)包含有关我们的符合性认证的详细的信息。

- **安全的文档保留**：Azure Blob 存储的不可变存储可确保不能修改或删除的任何用户，包括使用帐户管理权限的用户数据。

- **法定保留**：Azure Blob 存储的不可变的存储，用户可以存储敏感信息很关键的诉讼或企业版中使用防篡改的状态为所需的持续时间，直到被删除。 此功能不是仅限于合法用例，但可以还认为的基于事件的保留或企业锁，需要保护基于事件触发器或企业策略的数据的需要的。

不可变的存储支持以下功能：

- **[基于时间的保留策略支持](#time-based-retention)**:用户可以设置策略以指定的间隔时间存储数据。 基于时间的保留策略时设置，blob 可以创建和读取，但不是修改或删除。 保留期到期后，可以删除 blob，但不是会被覆盖。

- **[法定保留策略支持](#legal-holds)**:如果不知道在保留间隔，用户可以设置法定保留 immutably 合法保留已清除之前存储数据。  当合法保留策略设置时，blob 可以创建和读取，但不是修改或删除。 每个法定保留都具有用户定义的字母数字标记 （如用例 ID，事件名称，等等)，可作为标识符的字符串相关联。 

- **支持所有 Blob 层**：WORM 策略独立于 Azure Blob 存储层，将应用到所有层：热层、冷层和存档层。 用户可将工作负荷的数据转换为最具成本效益的层，同时保持数据的不可变性。

- **容器级配置**：用户可在容器级别配置基于时间的保留策略和法定保留标记。 通过使用简单的容器级别设置，用户可以创建和锁定基于时间的保留策略、 扩展保留间隔、 设置和清除法定保留与的详细信息。 这些策略将应用到容器中的所有 Blob，不管是现有的还是新的 Blob。

- **审核日志记录支持**：每个容器包括策略审核日志。 它显示为 7 基于时间的保留期为锁定的基于时间的保留策略的命令和包含用户 ID、 命令类型、 时间戳和保留间隔。 对于法定保留，日志包含用户 ID、命令类型、时间戳和法定保留标记。 此日志将保留策略，根据秒 17a-4 （f） 法规指导原则的生存期。 [Azure 活动日志](../../azure-monitor/platform/activity-logs-overview.md)显示了更全面的日志的所有控制平面活动; 同时[Azure 诊断日志](../../azure-monitor/platform/diagnostic-logs-overview.md)中保留和显示数据平面操作。 由用户负责根据法规要求或其他要求永久存储这些日志。

## <a name="how-it-works"></a>工作原理

Azure Blob 存储的不可变存储支持两类 WORM 或不可变策略：基于时间的保留和法定保留。 在容器上应用基于时间的保留策略或合法保留时，所有现有 blob 将移到不可变的蠕虫病毒状态在 30 秒内。 上传到该容器的所有新 blob 也会将迁移到不可变状态。 所有 blob 都移动到不可变状态后，确认的不可变的策略，并且所有覆盖或删除后不允许操作不可变的容器中的现有和新对象。

容器和帐户删除也不允许如果受保护的不可变的策略的任何 blob。 如果至少存在一个 Blob 使用锁定的基于时间的保留策略或法定保留，则“删除容器”操作会失败。 如果至少有一个 WORM 容器有法定保留，或者有一个 Blob 有活动的保留时间间隔，则存储帐户删除操作会失败。 

### <a name="time-based-retention"></a>基于时间的保留期

> [!IMPORTANT]
> 根据 SEC 17a-4(f) 和其他法规符合性要求，基于时间的保留策略必须处于锁定状态才能确保 Blob 不可变（不可写入和删除）。 我们建议将策略锁定合理的时间，通常为 24 小时内。 除非是短时功能试用，否则我们不建议使用非锁定状态。

在容器上应用基于时间的保留策略时，容器中的所有 Blob 都会保持在不可变的状态，其持续时间就是有效保留期。 现有 blob 的有效保留期等于 blob 修改时间和用户指定的保留间隔之间的差异。

就新 Blob 来说，有效保持期为用户指定的保留时间间隔。 由于用户可以延长保留时间间隔，因此不可变存储使用用户指定的保留时间间隔的最新值来计算有效保留期。

> [!TIP]
> **示例：** 用户创建了一个基于时间的保留策略，将保留时间间隔设置为五年。
>
> 该容器中的现有 blob _testblob1_，创建一年前。 有效保留期_testblob1_是四年的时间。
>
> 新的 blob _testblob2_，现在上传到容器。 此新 Blob 的有效保留期为五年。

### <a name="legal-holds"></a>法定保留

如果设置了法定保留，所有现有的和新的 Blob 会一直保持在不可变状态，直至法定保留被清除。 有关如何设置和清除法定保留的详细信息，请参阅[入门](#getting-started)部分。

容器可能会同时有法定保留策略和基于时间的保留策略。 该容器中的所有 Blob 会一直保持不可变状态，直至所有法定保留被清除，即使有效保留期已过。 与之相反，即使所有法定保留已被清除，Blob 也会保持不可变状态，直至有效保留期已过。

下表显示了会因各种不可变方案而禁用的 Blob 操作的类型。 有关详细信息，请参阅 [Azure Blob 服务 API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 文档。

|场景  |Blob 状态  |不允许 Blob 操作  |
|---------|---------|---------|
|Blob 的有效保留时间间隔尚未到期，并且/或者法定保留已设置     |不可变：不可删除和写入         | 放置 Blob<sup>1</sup>，将块放<sup>1</sup>，放置块列表<sup>1</sup>、 删除容器，删除 Blob，设置 Blob 元数据、 将页面、 设置 Blob 属性，快照 Blob，增量复制 Blob追加块         |
|Blob 的有效保留时间间隔尚未到期     |仅仅不可写入（允许删除操作）         |放置 Blob<sup>1</sup>、放置块<sup>1</sup>、放置块列表<sup>1</sup>、设置 Blob 元数据、放置页、设置 Blob 属性、快照 Blob、增量复制 Blob、追加块         |
|清除了所有法定保留，未在容器上设置任何基于时间的保留策略     |可变         |无         |
|未创建任何 WORM 策略（基于时间的保留或法定保留）     |可变         |无         |

<sup>1</sup>的应用程序允许这些操作一次创建新的 blob。 所有后续覆盖不允许对现有 blob 路径中的不可变的容器上的操作。

## <a name="supported-values"></a>支持的值

### <a name="time-based-retention"></a>基于时间的保留期
- 对于存储帐户，使用基于时间的不可变的锁定策略的容器的最大数目为 1000。
- 最小保持期间隔为 1 天。 最大值为 146,000 天 （400 年）。
- 对于容器，编辑扩展锁定基于时间的不可变的策略指定的保留间隔的最大数目为 5。
- 对于容器，最多 7 基于时间的保留策略审核日志将保留策略的持续时间。

### <a name="legal-hold"></a>法定保留
- 对于某个存储帐户而言，使用法定保留设置的最大容器数为 1,000。
- 对于某个容器而言，最大法定保留标记数为 10。
- 合法保留标记的最小长度为 3 个字母数字字符。 最大长度为 23 的字母数字字符。
- 对于容器，最多 10 合法保留策略审核日志将保留策略的持续时间。

## <a name="pricing"></a>定价

使用此功能不会产生额外的费用。 不可变数据的定价方式与常规的可变数据相同。 有关 Azure Blob 存储的定价详细信息，请参阅 [Azure 存储定价页](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="getting-started"></a>入门
不可变的存储是仅适用于常规用途 v2 和 Blob 存储帐户。 必须通过管理这些帐户[Azure 资源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。 有关升级现有的常规用途 v1 存储帐户的信息，请参阅[存储帐户升级](../common/storage-account-upgrade.md)。

最新版本的[Azure 门户](https://portal.azure.com)， [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)，并[Azure PowerShell](https://github.com/Azure/azure-powershell/releases)支持 Azure Blob 存储的不可变的存储。 [客户端库支持](#client-libraries)还提供了。

### <a name="azure-portal"></a>Azure 门户

1. 请创建新的容器或选择现有容器，以便存储需要保持不可变状态的 Blob。
 容器必须位于 GPv2 或 Blob 存储帐户中。
2. 在容器设置中选择“访问策略”。 然后选择“不可变 Blob 存储”下的“+ 添加策略”。

    ![门户中的容器设置](media/storage-blob-immutable-storage/portal-image-1.png)

3. 若要启用基于时间的保留，请从下拉菜单中选择“基于时间的保留”。

    ![在“策略类型”下选择“基于时间的保留”](media/storage-blob-immutable-storage/portal-image-2.png)

4. 天 （可接受的值为 1 到 146000 天） 中输入保留间隔。

    ![“将保留期更新为”框](media/storage-blob-immutable-storage/portal-image-5-retention-interval.png)

    该策略的初始状态处于解锁状态使您可以测试此功能和对策略进行更改之前将其锁定。 锁定策略至关重要的 SEC 17a-4 等法规符合性。

5. 锁定策略。 右键单击省略号 (**...**)，并与其他操作出现以下菜单：

    ![菜单中的“锁定策略”](media/storage-blob-immutable-storage/portal-image-4-lock-policy.png)

6. 选择**锁定策略**并确认该锁。 策略现在已被锁定，且不能删除，将允许仅在保留间隔的扩展。 Blob 删除和不允许使用替代。 

    ![确认菜单上的"锁定策略"](media/storage-blob-immutable-storage/portal-image-5-lock-policy.png)

7. 若要启用法定保留，请选择“+ 添加策略”。 从下拉菜单中选择“法定保留”。

    ![菜单中“策略类型”下面的“法定保留”](media/storage-blob-immutable-storage/portal-image-legal-hold-selection-7.png)

8. 使用一个或多个标记创建法定保留。

    ![策略类型下面的“标记名称”框](media/storage-blob-immutable-storage/portal-image-set-legal-hold-tags.png)

9. 若要清除合法保留，只需删除应用的合法保留标识符标记。

### <a name="azure-cli"></a>Azure CLI

以下命令组包含该功能：`az storage container immutability-policy` 和 `az storage container legal-hold`。 对这些命令运行 `-h` 可查看命令。

### <a name="powershell"></a>PowerShell

Az.Storage 模块支持使用不可变的存储。  若要启用该功能，请执行以下步骤：

1. 确保已安装最新版本的 PowerShellGet：`Install-Module PowerShellGet –Repository PSGallery –Force`。
2. 删除以前安装的 Azure PowerShell。
3. 安装 Azure PowerShell：`Install-Module Az –Repository PSGallery –AllowClobber`。

本文稍后的[示例 PowerShell 代码](#sample-powershell-code)部分演示了该功能的用法。

## <a name="client-libraries"></a>客户端库

以下客户端库支持 Azure Blob 存储的不可变存储：

- [.NET 客户端库 7.2.0-preview 和更高版本](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage/7.2.0-preview)
- [Node.js 客户端库 4.0.0 和更高版本](https://www.npmjs.com/package/azure-arm-storage)
- [Python 客户端库 2.0.0 候选发布版 2 和更高版本](https://pypi.org/project/azure-mgmt-storage/2.0.0rc2/)
- [Java 客户端库](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/storage/resource-manager/Microsoft.Storage/preview/2018-03-01-preview)

## <a name="faq"></a>常见问题解答

**你可以提供的蠕虫病毒符合性的文档？**

可以。 文档符合性，Microsoft 保留前导独立评估公司，专注于记录管理和信息管理，Cohasset 相关联，若要评估 Azure 不可变的 Blob 存储和及其是否符合特定要求为金融服务行业。 Cohasset 验证 Azure 不可变的 Blob 存储，用于保留在蠕虫状态下，基于时间的 Blob 时满足的 CFTC 规则 1.31(c)-(d)、 FINRA 规则 4511 和 SEC Rule 17a-4 相关的存储要求。 Microsoft 针对这组规则，因为它们代表全局的金融机构记录保留期的最说明性指南。 Cohasset 报表现已推出[Microsoft 服务信任中心](https://aka.ms/AzureWormStorage)。

**此功能是只适用于块 Blob，还是也适用于页 Blob 和追加 Blob？**

不可变存储可以用于任何 Blob 类型，但我们建议主要将其用于块 Blob。 与块 Blob 不同，页 Blob 和追加 Blob 需先在 WORM 容器外部创建，然后复制到容器中。 复制这些 blob 到蠕虫容器中，不进一步后*追加*追加允许 blob 或页 blob 的更改。

**我是否需要创建新的存储帐户才能使用此功能？**

不可以，你可以与任何现有或新创建常规用途 v2 或 Blob 存储帐户使用不可变的存储。 此功能适用于使用 GPv2 和 Blob 存储帐户中的块 blob 的使用情况。 常规用途 v1 存储帐户不受支持，但可以轻松地升级到常规用途 v2。 有关升级现有的常规用途 v1 存储帐户的信息，请参阅[存储帐户升级](../common/storage-account-upgrade.md)。

**可以将应用的合法保留和基于时间的保留策略？**

是的一个容器可以同时具有合法保留和基于时间的保留策略。 该容器中的所有 Blob 会一直保持不可变状态，直至所有法定保留被清除，即使有效保留期已过。 与之相反，即使所有法定保留已被清除，Blob 也会保持不可变状态，直至有效保留期已过。

**仅为法律诉讼程序是合法保留策略，或是否有其他使用方案？**

否，合法保留是只用于非基于时间的保留策略的一般术语。 它不需要以仅用于诉讼相关会议记录。 合法保留策略可用于禁用覆盖和删除用于保护重要的企业蠕虫数据保留期为未知。 您或者可以使用它作为企业策略来保护任务关键蠕虫工作负荷之前自定义事件触发器需要使用基于时间的保留策略，则会将其用作暂存策略。 

**如果尝试删除某个容器，而该容器使用锁定的基于时间的保留策略或法定保留，会发生什么情况？**

如果至少存在一个 Blob 使用锁定的基于时间的保留策略或法定保留，则“删除容器”操作会失败。 仅当没有 Blob 存在活动的保留时间间隔，也没有法定保留时，“删除容器”操作才会成功。 必须先删除 Blob，然后才能删除容器。

**如果尝试删除的存储帐户有一个 WORM 容器，而该容器使用锁定的基于时间的保留策略或法定保留，会发生什么情况？**

如果至少有一个 WORM 容器有法定保留，或者有一个 Blob 有活动的保留时间间隔，则存储帐户删除操作会失败。 必须先删除所有 WORM 容器，然后才能删除存储帐户。 有关删除容器的信息，请查看上一个问题。

**当 Blob 处于不可变状态时，是否可以跨不同的 Blob 层（热、凉、冷）来移动数据？**

是的设置 Blob 层命令可用于将数据保存在符合的不可变状态的同时跨多个 blob 层移动数据。 热、冷和存档 Blob 层均支持不可变存储。

**如果我无法付款，但我的保留时间间隔尚未到期，会发生什么情况？**

在未付款的情况下，会根据你与 Microsoft 签署的合同条款与条件应用常规的数据保留策略。

**你们是否为只想试用此功能的用户提供试用期或宽限期？**

可以。 首先创建一个基于时间的保留策略后，它是在*解锁*状态。 在这种状态下，可以对保留时间间隔进行所需的更改，例如延长或缩短保留时间间隔，甚至可以删除策略。 策略被锁定后，它将保持锁定状态，直到保留时间间隔到期为止。 此锁定的策略会阻止删除和修改在保留间隔。 我们强烈建议仅在试用的情况下使用未锁定状态，并在 24 小时内锁定策略。 这种做法有助于遵守 SEC 17a-4(f) 及其他法规。

**可以使用不可变的 blob 策略旁边的软删除？**

可以。 [Azure Blob 存储的软删除](storage-blob-soft-delete.md)适用于存储帐户而不考虑为合法保留或基于时间的保留策略中的所有容器。 我们建议启用软删除额外的保护之前任何不可变的蠕虫病毒策略应用并确认。 

**其中提供了功能？**

Azure 公共区域、中国区域和政府区域均提供不可变存储。 如果不可变的存储区域中不可用，请联系支持和电子邮件azurestoragefeedback@microsoft.com。

## <a name="sample-powershell-code"></a>示例 PowerShell 代码

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

以下示例 PowerShell 脚本仅供参考。 此脚本创建新的存储帐户和容器。 然后，它演示如何设置和清除法定保留、创建和锁定基于时间的保留策略（也称为不可变策略），并延长保留时间间隔。

设置并测试 Azure 存储帐户：

```powershell
$ResourceGroup = "<Enter your resource group>”
$StorageAccount = "<Enter your storage account name>"
$container = "<Enter your container name>"
$container2 = "<Enter another container name>”
$location = "<Enter the storage account location>"

# Log in to the Azure Resource Manager account
Login-AzAccount
Register-AzResourceProvider -ProviderNamespace "Microsoft.Storage"

# Create your Azure resource group
New-AzResourceGroup -Name $ResourceGroup -Location $location

# Create your Azure storage account
New-AzStorageAccount -ResourceGroupName $ResourceGroup -StorageAccountName `
    $StorageAccount -SkuName Standard_LRS -Location $location -Kind StorageV2

# Create a new container
New-AzStorageContainer -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -Name $container

# Create Container 2 with a storage account object
$accountObject = Get-AzStorageAccount -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount
New-AzStorageContainer -StorageAccount $accountObject -Name $container2

# Get a container
Get-AzStorageContainer -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -Name $container

# Get a container with an account object
$containerObject = Get-AzStorageContainer -StorageAccount $accountObject -Name $container

# List containers
Get-AzStorageContainer -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount

# Remove a container (add -Force to dismiss the prompt)
Remove-AzStorageContainer -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -Name $container2

# Remove a container with an account object
Remove-AzStorageContainer -StorageAccount $accountObject -Name $container2

# Remove a container with a container object
$containerObject2 = Get-AzStorageContainer -StorageAccount $accountObject -Name $container2
Remove-AzStorageContainer -InputObject $containerObject2
```

设置并清除法定保留：

```powershell
# Set a legal hold
Add-AzRmStorageContainerLegalHold -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -Name $container -Tag <tag1>,<tag2>,...

# with an account object
Add-AzRmStorageContainerLegalHold -StorageAccount $accountObject -Name $container -Tag <tag3>

# with a container object
Add-AzRmStorageContainerLegalHold -Container $containerObject -Tag <tag4>,<tag5>,...

# Clear a legal hold
Remove-AzRmStorageContainerLegalHold -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -Name $container -Tag <tag2>

# with an account object
Remove-AzRmStorageContainerLegalHold -StorageAccount $accountObject -Name $container -Tag <tag3>,<tag5>

# with a container object
Remove-AzRmStorageContainerLegalHold -Container $containerObject -Tag <tag4>
```

创建或更新不可变策略：
```powershell
# with an account name or container name
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -ContainerName $container -ImmutabilityPeriod 10

# with an account object
Set-AzRmStorageContainerImmutabilityPolicy -StorageAccount $accountObject `
    -ContainerName $container -ImmutabilityPeriod 1 -Etag $policy.Etag

# with a container object
$policy = Set-AzRmStorageContainerImmutabilityPolicy -Container `
    $containerObject -ImmutabilityPeriod 7

# with an immutability policy object
Set-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy -ImmutabilityPeriod 5
```

检索不可变策略：
```powershell
# Get an immutability policy
Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $ResourceGroup `
    -StorageAccountName $StorageAccount -ContainerName $container

# with an account object
Get-AzRmStorageContainerImmutabilityPolicy -StorageAccount $accountObject `
    -ContainerName $container

# with a container object
Get-AzRmStorageContainerImmutabilityPolicy -Container $containerObject
```

锁定不可变策略（添加 -Force 以关闭提示）：
```powershell
# with an immutability policy object
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container
$policy = Lock-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy -force

# with an account name or container name
$policy = Lock-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container `
    -Etag $policy.Etag

# with an account object
$policy = Lock-AzRmStorageContainerImmutabilityPolicy -StorageAccount `
    $accountObject -ContainerName $container -Etag $policy.Etag

# with a container object
$policy = Lock-AzRmStorageContainerImmutabilityPolicy -Container `
    $containerObject -Etag $policy.Etag -force
```

扩展不可变策略：
```powershell

# with an immutability policy object
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container

$policy = Set-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy `
    $policy -ImmutabilityPeriod 11 -ExtendPolicy

# with an account name or container name
$policy = Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container `
    -ImmutabilityPeriod 11 -Etag $policy.Etag -ExtendPolicy

# with an account object
$policy = Set-AzRmStorageContainerImmutabilityPolicy -StorageAccount `
    $accountObject -ContainerName $container -ImmutabilityPeriod 12 -Etag `
    $policy.Etag -ExtendPolicy

# with a container object
$policy = Set-AzRmStorageContainerImmutabilityPolicy -Container `
    $containerObject -ImmutabilityPeriod 13 -Etag $policy.Etag -ExtendPolicy
```

删除不可变策略（添加 -Force 以关闭提示）：
```powershell
# with an immutability policy object
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container
Remove-AzStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy

# with an account name or container name
Remove-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $ResourceGroup -StorageAccountName $StorageAccount -ContainerName $container `
    -Etag $policy.Etag

# with an account object
Remove-AzRmStorageContainerImmutabilityPolicy -StorageAccount $accountObject `
    -ContainerName $container -Etag $policy.Etag

# with a container object
Remove-AzRmStorageContainerImmutabilityPolicy -Container $containerObject `
    -Etag $policy.Etag

```
