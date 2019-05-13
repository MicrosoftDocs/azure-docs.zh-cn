---
title: 确定何时使用 Azure Blob、Azure 文件或 Azure 磁盘
description: 了解可通过哪些不同的方式在 Azure 中存储和访问数据，以帮助确定要使用哪种技术。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 11/28/2018
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: f66a2699b6d29f10633b4853801240f0590ff918
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147645"
---
# <a name="deciding-when-to-use-azure-blobs-azure-files-or-azure-disks"></a>确定何时使用 Azure Blob、Azure 文件或 Azure 磁盘
Microsoft Azure 在 Azure 存储中提供多种功能用于在云中存储和访问数据。 本文介绍 Azure 文件、Blob 和磁盘，旨在帮助用户选择合适的功能。

## <a name="scenarios"></a>方案
下表比较了文件、Blob 和磁盘，并显示每种技术适合的示例情景。

| Feature | 描述 | 使用时机 |
|--------------|-------------|-------------|
| Azure 文件 | 提供 SMB 接口、客户端库，以及一个用于从任意位置访问所存储的文件的 [REST 接口](/rest/api/storageservices/file-service-rest-api)。 | 可让你将某个已使用本机文件系统 API 在自身与在 Azure 中运行的其他应用程序之间共享数据的应用程序“即时转移”到云中。<br/><br/>可让你存储需要从许多虚拟机访问的开发和调试工具。 |
| **Azure Blob** | 提供客户端库，以及一个可用于在块 blob 中大规模存储和访问非结构化数据的 [REST 接口](/rest/api/storageservices/blob-service-rest-api)。<br/><br/>还支持 [Azure Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md)，用于企业大数据分析解决方案。 | 使应用程序能够支持流式处理和随机访问方案。<br/><br/>可让你从任意位置访问应用程序数据。<br/><br/>想要在 Azure 上生成企业数据湖并执行大数据分析。 |
| **Azure 磁盘** | 提供客户端库和 [REST 接口](/rest/api/compute/manageddisks/disks/disks-rest-api)，借助该接口可通过附加的虚拟硬盘永久地存储和访问数据。 | 可让你即时转移使用本机文件系统 API 在持久性磁盘中读取和写入数据的应用程序。<br/><br/>可让你存储不需要从磁盘附加到的虚拟机外部访问的数据。 |

## <a name="comparison-files-and-blobs"></a>比较：文件和 Blob
下表将 Azure 文件和 Azure Blob 进行比较。  
  
||||  
|-|-|-|  
|属性|**Azure Blob**|**Azure 文件**|  
|持久性选项|LRS、ZRS、GRS、RA-GRS|LRS、ZRS、GRS|  
|可访问性|REST API|REST API<br /><br /> SMB 2.1 和 SMB 3.0（标准文件系统 API）|  
|连接|REST API - 全球|REST API - 全球<br /><br /> SMB 2.1 - 区域内部<br /><br /> SMB 3.0 - 全球|  
|终结点|`http://myaccount.blob.core.windows.net/mycontainer/myblob`|`\\myaccount.file.core.windows.net\myshare\myfile.txt`<br /><br /> `http://myaccount.file.core.windows.net/myshare/myfile.txt`|  
|目录|平面命名空间|真正目录对象|  
|名称区分大小写|区分大小写|不区分大小写，但保留大小写|  
|容量|最多 2 个 PiB 帐户限制 |5 TiB 文件共享|  
|Throughput|每个块 Blob 高达 60 MiB/秒|每个共享高达 60 MiB/秒|  
|对象大小|每个块 blob 最多大约 4.75 TiB|每个文件最多为 1 TiB|  
|计费容量|基于写入的字节数|基于文件大小|  
|客户端库|多种语言|多语言|  
  
## <a name="comparison-files-and-disks"></a>比较：文件和磁盘
Azure 文件是对 Azure 磁盘的补充。 一个磁盘每次只能附加到一个 Azure 虚拟机。 磁盘是作为页 blob 存储在 Azure 存储中的固定格式 VHD，由虚拟机用来存储持久性数据。 可以像访问本地磁盘一样访问 Azure 文件中的文件共享（使用本机文件系统 API），后者可在多个虚拟机之间共享。  
 
下表将 Azure 文件和 Azure 磁盘进行比较。  
 
||||  
|-|-|-|  
|**属性**|**Azure 磁盘**|**Azure 文件**|  
|范围|专供单个虚拟机使用|可在多个虚拟机之间进行共享访问|  
|快照和复制|是|是|  
|配置|启动虚拟机时进行连接|启动虚拟机后进行连接|  
|Authentication|内置|使用 net use 进行设置|  
|使用 REST 访问|无法访问 VHD 中的文件|可以访问共享中存储的文件|  
|最大大小|4 TiB 磁盘|5 TiB 文件共享，共享中可保存 1 TiB 文件|  
|最大 IOPS|500 IOps|1000 IOps|  
|Throughput|每个磁盘高达 60 MiB/秒|目标是每个文件共享 60 MiB/秒（更高 IO 大小可以达到更高目标）|  

## <a name="next-steps"></a>后续步骤
决定如何存储和访问数据时，还应考虑到相关的成本。 有关详细信息，请参阅 [Azure 存储定价](https://azure.microsoft.com/pricing/details/storage/)。
  
某些 SMB 功能不适用于云。 有关详细信息，请参阅 [Azure 文件服务不支持的功能](/rest/api/storageservices/features-not-supported-by-the-azure-file-service)。
  
有关磁盘的详细信息，请参阅[托管磁盘简介](../../virtual-machines/windows/managed-disks-overview.md)以及[如何将数据磁盘附加到 Windows 虚拟机](../../virtual-machines/windows/attach-managed-disk-portal.md)。
