---
title: 使用 Azure 存储安全访问云中的应用程序数据 |Microsoft Docs
description: 使用 SAS 令牌、加密和 HTTPS 保护云中的应用程序数据。
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 05/30/2018
ms.author: tamram
ms.reviewer: cbrooks
ms.custom: mvc
ms.openlocfilehash: 48bd21ad2f78879091419477dcc3c278d6e74711
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236629"
---
# <a name="secure-access-to-an-applications-data-in-the-cloud"></a>安全访问云中的应用程序数据

本教程是一个系列中的第三部分。 你将了解如何安全访问存储帐户。 

在该系列的第三部分中，你会学习如何：

> [!div class="checklist"]
> * 使用 SAS 令牌访问缩略图图像
> * 启用服务器端加密
> * 启用仅 HTTPS 传输

[Azure blob 存储](../common/storage-introduction.md#blob-storage)提供存储应用程序文件的可靠服务。 此教程对[上一个主题][previous-tutorial]进行扩展，演示如何从 Web 应用程序安全访问存储帐户。 完成后，图像会被加密，Web 应用使用安全的 SAS 令牌访问缩略图图像。

## <a name="prerequisites"></a>先决条件

若要完成本教程，必须先完成上一存储教程：[使用事件网格自动调整上传图像的大小][previous-tutorial]。 

## <a name="set-container-public-access"></a>设置容器公共访问权限

在系列教程的此部分中，SAS 令牌用于访问缩略图。 在此步骤中，请将 _thumbnails_ 容器的公共访问权限设置为 `off`。

```azurecli-interactive 
blobStorageAccount=<blob_storage_account>

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
-n $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission \ --account-name $blobStorageAccount \ --account-key $blobStorageAccountKey \ --name thumbnails  \
--public-access off
``` 

## <a name="configure-sas-tokens-for-thumbnails"></a>为缩略图配置 SAS 令牌

在本系列教程的第一部分中，Web 应用程序显示公共容器中的图像。 在此系列教程的这个部分中，请使用[共享访问签名 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md#what-is-a-shared-access-signature) 令牌检索缩略图图像。 通过 SAS 令牌，可以实现基于 IP、协议、时间间隔或允许的权限提供对容器或 blob 的受限访问权限。

此示例中，源代码存储库使用 `sasTokens` 分支，该分支有更新的代码示例。 使用 [az webapp deployment source delete](/cli/azure/webapp/deployment/source) 删除现有的 GitHub 部署。 然后，使用 [az webapp deployment source config](/cli/azure/webapp/deployment/source) 命令配置 Web 应用的 GitHub 部署。  

在下面的命令中，`<web-app>` 是 Web 应用的名称。  

```azurecli-interactive 
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> \
--resource-group myResourceGroup --branch sasTokens --manual-integration \
--repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
``` 

存储库的 `sasTokens` 分支更新了 `StorageHelper.cs` 文件。 它将使用下面的代码示例替换 `GetThumbNailUrls` 任务。 通过设置 [SharedAccessBlobPolicy](/dotnet/api/microsoft.azure.storage.blob.sharedaccessblobpolicy?view=azure-dotnet) 指定 SAS 令牌的开始时间、到期时间和权限，更新的任务会检索缩略图 URL。 部署 Web 应用后，使用 SAS 令牌检索带 URL 的缩略图。 以下示例显示已更新任务：
    
```csharp
public static async Task<List<string>> GetThumbNailUrls(AzureStorageConfig _storageConfig)
{
    List<string> thumbnailUrls = new List<string>();

    // Create storagecredentials object by reading the values from the configuration (appsettings.json)
    StorageCredentials storageCredentials = new StorageCredentials(_storageConfig.AccountName, _storageConfig.AccountKey);

    // Create cloudstorage account by passing the storagecredentials
    CloudStorageAccount storageAccount = new CloudStorageAccount(storageCredentials, true);

    // Create blob client
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get reference to the container
    CloudBlobContainer container = blobClient.GetContainerReference(_storageConfig.ThumbnailContainer);

    BlobContinuationToken continuationToken = null;

    BlobResultSegment resultSegment = null;

    //Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
    //When the continuation token is null, the last page has been returned and execution can exit the loop.
    do
    {
        //This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
        //or by calling a different overload.
        resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);

        foreach (var blobItem in resultSegment.Results)
        {
            CloudBlockBlob blob = blobItem as CloudBlockBlob;
            //Set the expiry time and permissions for the blob.
            //In this case, the start time is specified as a few minutes in the past, to mitigate clock skew.
            //The shared access signature will be valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();

            sasConstraints.SharedAccessStartTime = DateTimeOffset.UtcNow.AddMinutes(-5);

            sasConstraints.SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddHours(24);

            sasConstraints.Permissions = SharedAccessBlobPermissions.Read;

            //Generate the shared access signature on the blob, setting the constraints directly on the signature.
            string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

            //Return the URI string for the container, including the SAS token.
            thumbnailUrls.Add(blob.Uri + sasBlobToken);

        }

        //Get the continuation token.
        continuationToken = resultSegment.ContinuationToken;
    }

    while (continuationToken != null);

    return await Task.FromResult(thumbnailUrls);
}
```

以下是用于前一任务的类、属性和方法：

|类  |属性| 方法  |
|---------|---------|---------|
|[StorageCredentials](/dotnet/api/microsoft.azure.cosmos.table.storagecredentials)    |         |
|[CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount)     | |[CreateCloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.blobaccountextensions.createcloudblobclient?view=azure-dotnet#Microsoft_WindowsAzure_Storage_CloudStorageAccount_CreateCloudBlobClient)        |
|[CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient)     | |[GetContainerReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getcontainerreference?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobClient_GetContainerReference_System_String_)         |
|[CloudBlobContainer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer?view=azure-dotnet)     | |[SetPermissionsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setpermissionsasync?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobContainer_SetPermissionsAsync_Microsoft_WindowsAzure_Storage_Blob_BlobContainerPermissions_) <br> [ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobContainer_ListBlobsSegmentedAsync_System_String_System_Boolean_Microsoft_WindowsAzure_Storage_Blob_BlobListingDetails_System_Nullable_System_Int32__Microsoft_WindowsAzure_Storage_Blob_BlobContinuationToken_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_)       |
|[BlobContinuationToken](/dotnet/api/microsoft.azure.storage.blob.blobcontinuationtoken?view=azure-dotnet)     |         |
|[BlobResultSegment](/dotnet/api/microsoft.azure.storage.blob.blobresultsegment?view=azure-dotnet)    | [结果](/dotnet/api/microsoft.azure.storage.blob.blobresultsegment.results?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobResultSegment_Results)         |
|[CloudBlockBlob](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob?view=azure-dotnet)    |         | [GetSharedAccessSignature](/dotnet/api/microsoft.azure.storage.blob.cloudblob.getsharedaccesssignature?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlob_GetSharedAccessSignature_Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_)
|[SharedAccessBlobPolicy](/dotnet/api/microsoft.azure.storage.blob.sharedaccessblobpolicy?view=azure-dotnet)     | [SharedAccessStartTime](/dotnet/api/microsoft.azure.storage.blob.sharedaccessblobpolicy.sharedaccessstarttime?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_SharedAccessStartTime)<br>[SharedAccessExpiryTime](/dotnet/api/microsoft.azure.storage.blob.sharedaccessblobpolicy.sharedaccessexpirytime?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_SharedAccessExpiryTime)<br>[权限](/dotnet/api/microsoft.azure.storage.blob.sharedaccessblobpolicy.permissions?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_Permissions) |        |

## <a name="server-side-encryption"></a>服务器端加密

[Azure 存储服务加密 (SSE)](../common/storage-service-encryption.md) 可帮助你保护数据。 SSE 加密静态数据，处理加密、解密和密钥管理。 采用 256 位 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)所有数据，它是现在最强有力的分组密码之一。

SSE 自动加密所有性能层（标准和高级）、所有部署模型（Azure 资源管理器和经典）、所有 Azure 存储服务（Blob、队列、表和文件）中的数据。 

## <a name="enable-https-only"></a>仅启用 HTTPS

为了确保对存储帐户数据的请求安全，可以将请求限制为仅 HTTPS。 使用 [az storage account update](/cli/azure/storage/account) 命令，更新协议所需的存储帐户。

```azurecli-interactive
az storage account update --resource-group myresourcegroup --name <storage-account-name> --https-only true
```

通过使用 `HTTP` 协议的 `curl` 测试连接。

```azurecli-interactive
curl http://<storage-account-name>.blob.core.windows.net/<container>/<blob-name> -I
```

由于需要安全的传输，将收到以下消息：

```
HTTP/1.1 400 The account being accessed does not support http.
```

## <a name="next-steps"></a>后续步骤

本系列教程的第三部分介绍了如何安全访问存储帐户，例如如何：

> [!div class="checklist"]
> * 使用 SAS 令牌访问缩略图图像
> * 启用服务器端加密
> * 启用仅 HTTPS 传输

本系列教程第四部分介绍如何对云存储应用程序进行监视和故障排除。

> [!div class="nextstepaction"]
> [对云存储应用程序进行监视和故障排除](storage-monitor-troubleshoot-storage-application.md)

[previous-tutorial]: ../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json
