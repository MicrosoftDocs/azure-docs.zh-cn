---
title: 启用对 Azure Blob 存储中容器和 blob 的公共读取访问 | Microsoft Docs
description: 了解如何使容器和 blob 可供匿名访问，以及如何对其进行程序式访问。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/30/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.openlocfilehash: e0f93b0a95a228b26fae266129aea4b595b05e0f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148352"
---
# <a name="manage-anonymous-read-access-to-containers-and-blobs"></a>管理对容器和 blob 的匿名读取访问

可以启用对 Azure Blob 存储中的容器及其 Blob 的匿名公共读取访问。 这样做可以授予对这些资源的只读访问权限，无需共享帐户密钥，也无需共享访问签名 (SAS)。

如果想要始终允许对某些 Blob 进行匿名读取访问，最好的方法是启用公共读取访问。 可以创建共享访问签名，实现更精细的控制。 利用共享访问签名，可以针对特定时间段提供使用不同权限的受限访问。 有关创建共享访问签名的详细信息，请参阅[在 Azure 存储中使用共享访问签名 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

## <a name="grant-anonymous-users-permissions-to-containers-and-blobs"></a>授予对容器和 Blob 的匿名用户权限

默认情况下，可能只能由已被授予适当权限的用户访问容器和其中的所有 blob。 若要授予对容器及其 blob 的匿名用户读取访问权限，可以设置容器公共访问级别。 当授予对容器公共访问权限时，匿名用户可以不授权请求的情况下读取可公开访问容器中的 blob。

可为容器配置以下权限：

* **无公共读取访问：** 只能由存储帐户所有者可以访问容器和其 blob。 这是所有新容器的默认权限。
* **仅限对 blob 的公共读取访问：** 可以通过匿名请求读取容器中的 blob，但容器数据不可用。 匿名客户端无法枚举容器中的 Blob。
* **公共读取访问容器和其 blob:** 可以通过匿名请求读取所有容器和 blob 数据。 客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

可以通过以下方式来设置容器权限：

* [Azure 门户](https://portal.azure.com)
* [Azure PowerShell](../common/storage-powershell-guide-full.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [Azure CLI](../common/storage-azure-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#create-and-manage-blobs)
* 通过使用一个存储客户端库或 REST API 以编程方式设置

### <a name="set-container-public-access-level-in-the-azure-portal"></a>在 Azure 门户中设置容器公共访问级别

从[Azure 门户](https://portal.azure.com)，可以更新一个或多个容器的公共访问级别：

1. 导航到 Azure 门户中的存储帐户。
1. 下**Blob 服务**上的菜单边栏选项卡中，选择**Blob**。
1. 选择你想要设置公共访问级别的容器。
1. 使用**更改访问级别**按钮显示的公共访问权限设置。
1. 选择所需的公共访问级别从**公共访问级别**下拉列表中，单击确定按钮以将更改应用到选择的容器。

以下屏幕截图显示了如何更改所选的容器的公共访问级别。

![显示如何在门户中设置公共访问级别的屏幕截图](./media/storage-manage-access-to-resources/storage-manage-access-to-resources-0.png)

> [!NOTE]
> 不能更改为单个 blob 的公共访问级别。 仅在容器级别设置公共访问级别。

### <a name="set-container-public-access-level-with-net"></a>使用.NET 容器公共访问级别设置

若要使用 C# 和适用于 .NET 的存储客户端库设置容器的权限，请先调用 GetPermissions 方法以检索容器的现有权限。 然后，设置 **GetPermissions** 方法返回的 **BlobContainerPermissions** 对象的 **PublicAccess** 属性。 最后，使用更新的权限调用 **SetPermissions** 方法。

以下示例将容器的权限设置为完全公共读取访问。 要将权限设置为仅针对 blob 的公共读取访问，请将 **PublicAccess** 属性设置为 **BlobContainerPublicAccessType.Blob**。 要删除匿名用户的所有权限，请将该属性设置为 **BlobContainerPublicAccessType.Off**。

```csharp
public static void SetPublicContainerPermissions(CloudBlobContainer container)
{
    BlobContainerPermissions permissions = container.GetPermissions();
    permissions.PublicAccess = BlobContainerPublicAccessType.Container;
    container.SetPermissions(permissions);
}
```

## <a name="access-containers-and-blobs-anonymously"></a>匿名访问容器和 Blob

如果某个客户端需要以匿名方式访问容器和 Blob，该客户端则可以使用不需要凭据的构造函数。 下面的示例演示了多种不同的方式要以匿名方式引用容器和 blob。

### <a name="create-an-anonymous-client-object"></a>创建匿名客户端对象

可以通过为帐户提供 Blob 存储终结点创建新的服务客户端对象进行匿名访问。 但是，也必须要知道该帐户中允许进行匿名访问的容器的名称。

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob storage endpoint.
    CloudBlobClient blobClient = new CloudBlobClient(new Uri(@"https://storagesample.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

### <a name="reference-a-container-anonymously"></a>以匿名方式引用容器

如果拥有可以通过匿名方式使用的容器的 URL，则可使用该 URL 来直接引用容器。

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(new Uri(@"https://storagesample.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

### <a name="reference-a-blob-anonymously"></a>以匿名方式引用 Blob

如果拥有允许进行匿名访问的 Blob 的 URL，则可使用该 URL 来直接引用 Blob：

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(@"https://storagesample.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", System.IO.FileMode.Create);
}
```

## <a name="features-available-to-anonymous-users"></a>对匿名用户可用的功能

下表显示了哪些操作可调用以匿名方式配置容器公共访问权限。

| REST 操作 | 对容器的公共读取访问权限 | 只有 Blob 的公共读取访问权限 |
| --- | --- | --- |
| 列出容器 | 仅授权的请求 | 仅授权的请求 |
| 创建容器 | 仅授权的请求 | 仅授权的请求 |
| 获取容器属性 | 允许匿名请求 | 仅授权的请求 |
| 获取容器元数据 | 允许匿名请求 | 仅授权的请求 |
| 设置容器元数据 | 仅授权的请求 | 仅授权的请求 |
| 获取容器 ACL | 仅授权的请求 | 仅授权的请求 |
| 设置容器 ACL | 仅授权的请求 | 仅授权的请求 |
| 删除容器 | 仅授权的请求 | 仅授权的请求 |
| 列出 Blob | 允许匿名请求 | 仅授权的请求 |
| 放置 Blob | 仅授权的请求 | 仅授权的请求 |
| 获取 Blob | 允许匿名请求 | 允许匿名请求 |
| 获取 Blob 属性 | 允许匿名请求 | 允许匿名请求 |
| 设置 Blob 属性 | 仅授权的请求 | 仅授权的请求 |
| 获取 Blob 元数据 | 允许匿名请求 | 允许匿名请求 |
| 设置 Blob 元数据 | 仅授权的请求 | 仅授权的请求 |
| 放置块 | 仅授权的请求 | 仅授权的请求 |
| 获取块列表（仅提交的块） | 允许匿名请求 | 允许匿名请求 |
| 获取块列表（仅未提交的块或所有块） | 仅授权的请求 | 仅授权的请求 |
| 放置块列表 | 仅授权的请求 | 仅授权的请求 |
| 删除 Blob | 仅授权的请求 | 仅授权的请求 |
| 复制 Blob | 仅授权的请求 | 仅授权的请求 |
| 快照 Blob | 仅授权的请求 | 仅授权的请求 |
| 租用 Blob | 仅授权的请求 | 仅授权的请求 |
| 放置页面 | 仅授权的请求 | 仅授权的请求 |
| 获取页面范围 | 允许匿名请求 | 允许匿名请求 |
| 追加 Blob | 仅授权的请求 | 仅授权的请求 |

## <a name="next-steps"></a>后续步骤

* [Azure 存储服务的授权](https://docs.microsoft.com/rest/api/storageservices/authorization-for-the-azure-storage-services)
* [使用共享访问签名 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)