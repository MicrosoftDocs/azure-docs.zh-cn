---
title: 将 Azure Blob 存储事件发送到 Web 终结点 - Powershell | Microsoft Docs
description: 使用 Azure 事件网格订阅 Blob 存储事件。
services: storage,event-grid
author: normesta
ms.author: normesta
ms.reviewer: dastanfo
ms.date: 08/23/2018
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.custom: seodec18
ms.openlocfilehash: cf1b0ba5d70ed0934418a147c09791725b5465bb
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143370"
---
# <a name="quickstart-route-storage-events-to-web-endpoint-with-powershell"></a>快速入门：利用 PowerShell 将存储事件路由到 Web 终结点

Azure 事件网格是针对云的事件处理服务。 在本文中，请使用 Azure PowerShell 订阅 Blob 存储事件，然后触发事件并查看结果。 

通常，你会将事件发送到处理事件数据并执行操作的终结点。 但是，为了简化本文，你将事件发送到收集并显示消息的 Web 应用。

完成后即可看到事件数据已发送到 Web 应用。

![查看结果](./media/storage-blob-event-quickstart-powershell/view-results.png)

## <a name="setup"></a>设置

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

本文要求运行最新版本的 Azure PowerShell。 如需进行安装或升级，请参阅[安装和配置 Azure PowerShell](/powershell/azure/install-Az-ps)。

## <a name="sign-in-to-azure"></a>登录 Azure

使用 `Connect-AzAccount` 命令登录到 Azure 订阅，然后按照屏幕上的说明进行身份验证。

```powershell
Connect-AzAccount
```

此示例使用 westus2，并将所选内容存储在变量中以供整个过程使用。

```powershell
$location = "westus2"
```

## <a name="create-a-resource-group"></a>创建资源组

事件网格主题是 Azure 资源，必须放置在 Azure 资源组中。 该资源组是在其中部署和管理 Azure 资源的逻辑集合。

使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 命令创建资源组。

以下示例在“westus2”位置创建名为“gridResourceGroup”的资源组。  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>创建存储帐户

可在常规用途 v2 存储帐户和 Blob 存储帐户中使用 Blob 存储事件。 常规用途 v2 存储帐户支持所有存储服务（包括 Blob、文件、队列和表）的所有功能。 Blob 存储帐户是一个专用存储帐户，用于将非结构化数据作为 Blob（对象）存储到 Azure 存储中。 Blob 存储帐户类似于常规用途存储帐户，并且具有现在使用的所有卓越的耐用性、可用性、伸缩性和性能功能，包括用于块 blob 和追加 blob 的 100% API 一致性。 有关详细信息，请参阅 [Azure 存储帐户概述](../common/storage-account-overview.md)。

使用 [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount) 创建具有 LRS 复制的 Blob 存储帐户，然后检索定义要使用的存储帐户的存储帐户上下文。 对存储帐户执行操作时，引用上下文而不是重复提供凭据。 本示例创建一个名为 gridstorage 的存储帐户，其中启用了本地冗余存储 (LRS)。 

> [!NOTE]
> 存储帐户名称位于全局命名空间，因此需要向此脚本中提供的名称追加某些随机字符。

```powershell
$storageName = "gridstorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind BlobStorage `
  -AccessTier Hot

$ctx = $storageAccount.Context
```

## <a name="create-a-message-endpoint"></a>创建消息终结点

在订阅主题之前，让我们创建事件消息的终结点。 通常情况下，终结点基于事件数据执行操作。 为了简化此快速入门，将部署用于显示事件消息的[预建的 Web 应用](https://github.com/Azure-Samples/azure-event-grid-viewer)。 所部署的解决方案包括应用服务计划、应用服务 Web 应用和 GitHub 中的源代码。

将 `<your-site-name>` 替换为 Web 应用的唯一名称。 Web 应用名称必须唯一，因为它是 DNS 条目的一部分。

```powershell
$sitename="<your-site-name>"

New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" `
  -siteName $sitename `
  -hostingPlanName viewerhost
```

部署可能需要几分钟才能完成。 部署成功后，请查看 Web 应用以确保它正在运行。 在 Web 浏览器中导航到 `https://<your-site-name>.azurewebsites.net`

应会看到站点上当前未显示任何消息。

[!INCLUDE [event-grid-register-provider-powershell.md](../../../includes/event-grid-register-provider-powershell.md)]

## <a name="subscribe-to-your-storage-account"></a>订阅存储帐户

订阅主题是为了告知事件网格要跟踪哪些事件。以下示例订阅所创建的存储帐户，并将 Web 应用中的 URL 作为事件通知的终结点传递。 Web 应用的终结点必须包括后缀 `/api/updates/`。

```powershell
$storageId = (Get-AzStorageAccount -ResourceGroupName $resourceGroup -AccountName $storageName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzEventGridSubscription `
  -EventSubscriptionName gridBlobQuickStart `
  -Endpoint $endpoint `
  -ResourceId $storageId
```

再次查看 Web 应用，并注意现已向该应用发送了订阅验证事件。 选择眼睛图标以展开事件数据。 事件网格发送验证事件，以便终结点可以验证它是否想要接收事件数据。 Web 应用包含用于验证订阅的代码。

![查看订阅事件](./media/storage-blob-event-quickstart-powershell/view-subscription-event.png)

## <a name="trigger-an-event-from-blob-storage"></a>触发 Blob 存储中的事件

现在，让我们触发一个事件，看事件网格如何将消息分发到终结点。 首先，让我们创建容器和对象。 然后，让我们将该对象上传到容器。

```powershell
$containerName = "gridcontainer"
New-AzStorageContainer -Name $containerName -Context $ctx

echo $null >> gridTestFile.txt

Set-AzStorageBlobContent -File gridTestFile.txt -Container $containerName -Context $ctx -Blob gridTestFile.txt
```

现已触发事件，并且事件网格已将消息发送到订阅时配置的终结点。 查看 Web 应用以查看刚刚发送的事件。

```json
[{
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Storage/storageAccounts/myblobstorageaccount",
  "subject": "/blobServices/default/containers/gridcontainer/blobs/gridTestFile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T20:33:51.0595757Z",
  "id": "4d96b1d4-0001-00b3-58ce-16568c064fab",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "Azure-Storage-PowerShell-d65ca2e2-a168-4155-b7a4-2c925c18902f",
    "requestId": "4d96b1d4-0001-00b3-58ce-16568c000000",
    "eTag": "0x8D4E4E61AE038AD",
    "contentType": "application/octet-stream",
    "contentLength": 0,
    "blobType": "BlockBlob",
    "url": "https://myblobstorageaccount.blob.core.windows.net/gridcontainer/gridTestFile.txt",
    "sequencer": "00000000000000EB0000000000046199",
    "storageDiagnostics": {
      "batchId": "dffea416-b46e-4613-ac19-0371c0c5e352"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

## <a name="clean-up-resources"></a>清理资源
如果计划继续使用此存储帐户和事件订阅，请不要清理在本文中创建的资源。 如果不打算继续学习，请使用以下命令删除本文中创建的资源。

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>后续步骤

了解如何创建主题和事件订阅以后，即可进一步学习 Blob 存储事件以及事件网格的功能：

- [响应 Blob 存储事件](storage-blob-event-overview.md)
- [关于事件网格](../../event-grid/overview.md)
