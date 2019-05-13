---
title: 在 Azure 内容分发网络中管理 Azure Blob 存储的到期时间 | Microsoft Docs
description: 了解用于控制 Azure CDN 缓存中 blob 生存时间的选项。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ad4801e9-d09a-49bf-b35c-efdc4e6034e8
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 02/1/2018
ms.author: mazha
ms.openlocfilehash: a6dcd57591dcc6aa09ae2cb62f4b6dfe964c979f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191143"
---
# <a name="manage-expiration-of-azure-blob-storage-in-azure-cdn"></a>在 Azure CDN 中管理 Azure Blob 存储的到期时间
> [!div class="op_single_selector"]
> * [Azure Web 内容](cdn-manage-expiration-of-cloud-service-content.md)
> * [Azure Blob 存储](cdn-manage-expiration-of-blob-content.md)
> 
> 

在多个与 Azure 内容分发网络 (CDN) 集成的基于 Azure 的源中，Azure 存储中的 [Blob 存储服务](../storage/common/storage-introduction.md#blob-storage)是其中一个。 任何可公开访问的 blob 内容均可在 Azure CDN 中进行缓存，直到其生存时间 (TTL) 结束。 TTL 由来自源服务器的 HTTP 响应中的 `Cache-Control` 标头决定。 本文介绍了几种可以在 Azure 存储中的 Blob 上设置 `Cache-Control` 标头的方式。

此外，还可以通过设置 CDN 缓存规则，从 Azure 门户控制缓存设置。 如果创建了一个缓存规则并将其缓存行为设置为“替代”或“绕过缓存”，则将忽略本文中讨论的源提供的缓存设置。 有关一般缓存概念的信息，请参阅[缓存工作原理](cdn-how-caching-works.md)。

> [!TIP]
> 可以选择不对 blob 设置 TTL。 在这种情况下，Azure CDN 将自动应用默认 TTL（七天），除非已在 Azure 门户中设置了缓存规则。 此默认 TTL 仅适用于常规 Web 交付优化。 对于大型文件优化，默认 TTL 为一天；对于媒体流优化，默认 TTL 为一年。
> 
> 有关 Azure CDN 如何加速访问 blob 和其他文件的详细信息，请参阅 [Azure 内容分发网络概述](cdn-overview.md)。
> 
> 有关 Azure Blob 存储的详细信息，请参阅 [Blob 存储简介](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)。
 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>使用 CDN 缓存规则设置 Cache-Control 标头
设置 blob 的 `Cache-Control` 标头的首选方法是使用 Azure 门户中的缓存规则。 有关 CDN 缓存规则的详细信息，请参阅[使用缓存规则控制 Azure CDN 缓存行为](cdn-caching-rules.md)。

> [!NOTE] 
> 缓存规则仅适用于 **Verizon 提供的标准 Azure CDN** 和 **Akamai 提供的标准 Azure CDN** 配置文件。 对于 **Verizon 提供的高级 Azure CDN** 配置文件，必须在**管理**门户中使用 [Azure CDN 规则引擎](cdn-rules-engine.md)来获得类似的功能。

**导航到 CDN 缓存规则页**：

1. 在 Azure 门户中，选择一个 CDN 配置文件，然后选择 blob 的终结点。

2. 在左窗格中的“设置”下，选择“缓存规则”。

   ![CDN 缓存规则按钮](./media/cdn-manage-expiration-of-blob-content/cdn-caching-rules-btn.png)

   “缓存规则”页随即出现。

   ![CDN 缓存页](./media/cdn-manage-expiration-of-blob-content/cdn-caching-page.png)


**使用全局缓存规则设置 Blob 存储服务的 Cache-Control 标头：**

1. 在“全局缓存规则”下，将“查询字符串缓存行为”设置为“忽略查询字符串”，将“缓存行为”设置为“覆盖”。
      
2. 对于“缓存过期持续时间”，在“秒”框中输入 3600，或者在“小时”框中输入 1。 

   ![CDN 全局缓存规则示例](./media/cdn-manage-expiration-of-blob-content/cdn-global-caching-rules-example.png)

   此全局缓存规则设置为期一小时的缓存持续时间，并会影响发送到终结点的所有请求。 它会替代由终结点指定的源服务器发送的所有 `Cache-Control` 或 `Expires` HTTP 标头。   

3. 选择“保存”。
 
**使用自定义缓存规则设置 Blob 文件的 Cache-Control 标头：**

1. 在“自定义缓存规则”下，创建两个匹配条件：

     A. 对于第一个匹配条件，将“匹配条件”设置为“路径”，对于“匹配值”输入 `/blobcontainer1/*`。 将“缓存行为”设置为“替代”，并在“小时”框中输入 4。

    B. 对于第二个匹配条件，将“匹配条件”设置为“路径”，对于“匹配值”输入 `/blobcontainer1/blob1.txt`。 将“缓存行为”设置为“替代”，并在“小时”框中输入 2。

    ![CDN 自定义缓存规则示例](./media/cdn-manage-expiration-of-blob-content/cdn-custom-caching-rules-example.png)

    第一个自定义缓存规则为终结点指定的源服务器上的 `/blobcontainer1` 文件夹中的所有 blob 文件设置为期四小时的缓存持续时间。 第二个规则仅替代 `blob1.txt` blob 文件的第一个规则，并且为它设置为期两小时的缓存持续时间。

2. 选择“保存”。


## <a name="setting-cache-control-headers-by-using-azure-powershell"></a>使用 Azure PowerShell 设置 Cache-Control 标头

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[Azure PowerShell](/powershell/azure/overview) 是管理 Azure 服务的最快捷且最强大的方式之一。 使用 `Get-AzStorageBlob`cmdlet 来获取对 blob 的引用，并设置 `.ICloudBlob.Properties.CacheControl` 属性。 

例如：

```powershell
# Create a storage context
$context = New-AzStorageContext -StorageAccountName "<storage account name>" -StorageAccountKey "<storage account key>"

# Get a reference to the blob
$blob = Get-AzStorageBlob -Context $context -Container "<container name>" -Blob "<blob name>"

# Set the CacheControl property to expire in 1 hour (3600 seconds)
$blob.ICloudBlob.Properties.CacheControl = "max-age=3600"

# Send the update to the cloud
$blob.ICloudBlob.SetProperties()
```

> [!TIP]
> 还可以使用 PowerShell 来[管理 CDN 配置文件和终结点](cdn-manage-powershell.md)。
> 
>

## <a name="setting-cache-control-headers-by-using-net"></a>使用 .NET 设置 Cache-Control 标头
要使用 .NET 代码指定 Blob 的 `Cache-Control` 标头，请使用[适用于 .NET 的 Azure 存储客户端库](../storage/blobs/storage-dotnet-how-to-use-blobs.md)来设置 [CloudBlob.Properties.CacheControl](/dotnet/api/microsoft.azure.storage.blob.blobproperties.cachecontrol#Microsoft_WindowsAzure_Storage_Blob_BlobProperties_CacheControl) 属性。

例如：

```csharp
class Program
{
    const string connectionString = "<storage connection string>";
    static void Main()
    {
        // Retrieve storage account information from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);

        // Create a blob client for interacting with the blob service.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create a reference to the container
        CloudBlobContainer <container name> = blobClient.GetContainerReference("<container name>");

        // Create a reference to the blob
        CloudBlob <blob name> = container.GetBlobReference("<blob name>");

        // Set the CacheControl property to expire in 1 hour (3600 seconds)
        blob.Properties.CacheControl = "max-age=3600";

        // Update the blob's properties in the cloud
        blob.SetProperties();
    }
}
```

> [!TIP]
> 更多 .NET 代码示例可在[适用于.NET 的 Azure Blob 存储示例](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/)中找到。
> 

## <a name="setting-cache-control-headers-by-using-other-methods"></a>使用其他方法设置 Cache-Control 标头

### <a name="azure-storage-explorer"></a>Azure 存储资源管理器
借助 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)，可以查看和编辑 Blob 存储资源，包括 CacheControl 等属性。 

若要使用 Azure 存储资源管理器更新 blob 的 *CacheControl* 属性：
   1. 选择 blob，然后从上下文菜单中选择“属性”。 
   2. 向下滚动到 *CacheControl* 属性。
   3. 输入一个值，然后选择“保存”。


![Azure 存储资源管理器属性](./media/cdn-manage-expiration-of-blob-content/cdn-storage-explorer-properties.png)

### <a name="azure-command-line-interface"></a>Azure 命令行接口
使用 [Azure 命令行接口](https://docs.microsoft.com/cli/azure) (CLI)，可以从命令行管理 Azure blob 资源。 若要在使用 Azure CLI 上传 Blob 时设置 cache-control 标头，可使用 `-p` 开关设置 *cacheControl* 属性。 以下示例说明如何将 TTL 设置为 1 小时（3600 秒）：
  
```azurecli
azure storage blob upload -c <connectionstring> -p cacheControl="max-age=3600" .\<blob name> <container name> <blob name>
```

### <a name="azure-storage-services-rest-api"></a>Azure 存储空间服务 REST API
通过对请求执行以下操作，可以使用 [Azure 存储服务 REST API](/rest/api/storageservices/) 来显式设置 *x-ms-blob-cache-control* 属性：
  
   - [放置 Blob](/rest/api/storageservices/Put-Blob)
   - [放置块列表](/rest/api/storageservices/Put-Block-List)
   - [设置 Blob 属性](/rest/api/storageservices/Set-Blob-Properties)

## <a name="testing-the-cache-control-header"></a>测试 Cache-Control 标头
可以轻松验证 Blob 的 TTL 设置。 使用浏览器的[开发人员工具](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/)，测试 Blob 是否包含 `Cache-Control` 响应标头。 还可使用 [Wget](https://www.gnu.org/software/wget/)、[Postman](https://www.getpostman.com/) 或 [Fiddler](https://www.telerik.com/fiddler) 等工具检查响应标头。

## <a name="next-steps"></a>后续步骤
* [了解如何管理 Azure CDN 中云服务内容的过期问题](cdn-manage-expiration-of-cloud-service-content.md)
* [了解缓存概念](cdn-how-caching-works.md)

