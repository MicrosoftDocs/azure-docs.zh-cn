---
title: Azure Data Lake Storage Gen2 的已知问题 | Microsoft Docs
description: 了解 Azure Data Lake Storage Gen2 的限制和已知问题
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: normesta
ms.openlocfilehash: 27adc0eeeabed2b1f2e86f301a60604a3d358b82
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464725"
---
# <a name="known-issues-with-azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2 的已知问题

本文列出了功能和工具，尚不支持或仅部分支持与具有层次结构命名空间 （Azure 数据湖存储第 2 代） 的存储帐户。

<a id="blob-apis-disabled" />

## <a name="blob-storage-apis"></a>Blob 存储 API

Blob 的存储 Api 已禁用，以防止意外的数据访问出现问题，因为 Blob 存储 Api 尚不可使用 Azure Data Lake 第 2 代 Api 可互操作。

### <a name="what-to-do-with-existing-tools-applications-and-services"></a>要执行的操作与现有的工具、 应用程序和服务

如果任何这些使用 Blob Api 和您想要使用它们可以处理所有的内容，将上传到你的帐户，然后到 Blob Api 便可与 Azure Data Lake 第 2 代 Api 互操作之前不启用到 Blob 存储帐户上的分层命名空间。

使用存储帐户没有分层命名空间，则意味着您然后不会访问数据湖存储第 2 代特定功能，例如目录和文件系统访问控制列表。

### <a name="what-to-do-with-unmanaged-virtual-machine-vm-disks"></a>要执行的操作与非托管虚拟机 (VM) 磁盘

这取决于已禁用的 Blob 存储 Api，因此如果你想要启用的分层命名空间上的存储帐户，请考虑将它们放置到存储帐户不具有启用了层次结构命名空间功能。

### <a name="what-to-do-if-you-used-blob-apis-to-load-data-before-blob-apis-were-disabled"></a>要执行的操作如果 Blob Api 用于将数据加载之前已禁用，Blob Api

如果你在这些 API 被禁用之前使用了它们来加载数据，并且生产环境需要访问该数据，请联系 Microsoft 支持部门并提供以下信息：

> [!div class="checklist"]
> * 订阅 ID （GUID，而不是名称）。
> * 存储帐户名称。
> * 是否你积极影响生产环境中，如果是，有关哪些存储帐户？。
> * 即使在生产环境中未频繁受到影响，也请告诉我们你是否出于某种原因需要将该数据复制到另一个存储帐户，如果是，为什么？

在这些情况下，我们可以在有限的时间段内恢复对 Blob API 的访问权限，以便你可以将此数据复制到未启用分层命名空间功能的存储帐户。

## <a name="all-other-features-and-tools"></a>所有其他功能和工具

下表列出了所有其他功能和工具，尚不支持或仅部分支持与具有层次结构命名空间 （Azure 数据湖存储第 2 代） 的存储帐户。

| 功能 / 工具    | 详细信息    |
|--------|-----------|
| **数据湖存储第 2 代存储帐户的 Api** | 部分支持 <br><br>可以使用数据湖存储第 2 代**REST** Api，但其他 Blob Sdk，如.NET、 Java、 Python Sdk 中的 Api 尚不可用。|
| **AzCopy** | 特定于版本的支持 <br><br>仅使用最新版本的 AzCopy ([AzCopy v10](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2ftables%2ftoc.json))。 不支持早期版本的 AzCopy，AzCopy v8.1，例如。|
| **Azure Blob 存储生命周期管理策略** | 尚不支持 |
| **Azure 内容分发网络 (CDN)** | 尚不支持|
| **Azure 事件网格** | 尚不支持 |
| **Azure 搜索** |尚不支持|
| **Azure 存储资源管理器** | 特定于版本的支持 <br><br>使用唯一版本`1.6.0`或更高版本。 <br>版本`1.6.0`可用作[免费下载](https://azure.microsoft.com/features/storage-explorer/)。|
| **Blob 容器的 Acl** |尚不支持|
| **Blobfuse** |尚不支持|
| **自定义域** |尚不支持|
| **诊断日志** |尚不支持|
| **文件系统资源管理器** | 有限的支持 |
| **不可变的存储** |尚不支持 <br><br>不可变的存储，可以将数据存储在[（编写一次，曾读过） 蠕虫](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutable-storage)状态。|
| **对象级层** |尚不支持 <br><br>例如:Premium、 热、 冷和存档层。|
| **Powershell 和 CLI 支持** | 有限的功能 <br><br>可以使用 Powershell 或 CLI 来创建一个帐户。 无法执行操作，或在文件系统、 目录和文件上设置访问控制列表。|
| **静态网站** |尚不支持 <br><br>具体而言，能够满足文件复制到[静态网站](https://docs.microsoft.com/azure/storage/blobs/storage-blob-static-website)。|
| **第三方应用程序** | 有限的支持 <br><br>使用 REST Api 处理的第三方应用程序将继续工作，如果它们使用的数据湖存储第 2 代。 <br>如果必须使用 Blob Api 的应用程序，该应用程序很可能会遇到问题，如果该应用程序使用数据湖存储第 2 代。 若要了解详细信息，请参阅[Blob 的存储 Api 的数据湖存储第 2 代存储帐户已禁用](#blob-apis-disabled)本文的部分。|
| **版本控制功能** |尚不支持 <br><br>这包括[快照](https://docs.microsoft.com/rest/api/storageservices/creating-a-snapshot-of-a-blob)并[软删除](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete)。|
|

