---
title: Azure 存储简介 - Azure 中的云存储 | Microsoft Docs
description: Azure 存储是 Microsoft 提供的云存储解决方案。 Azure 存储为数据对象提供具有高可用性、安全、持久、可大规模缩放且具有冗余的存储。
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 01/02/2019
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 094c8d45f7f6099b169e25c89dbfc4a7d6522dfc
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510943"
---
# <a name="introduction-to-azure-storage"></a>Azure 存储简介

Azure 存储是 Microsoft 提供的适用于现代数据存储场景的云存储解决方案。 Azure 存储为数据对象提供可大规模缩放的对象存储，为云提供文件系统服务，并且提供用于可靠消息传送的消息传送存储以及 NoSQL 存储。 Azure 存储：

- **持久且具有高可用性。** 冗余可确保数据在发生短暂的硬件故障时是安全的。 还可以选择在各个数据中心或地理区域之间复制数据，从而在发生本地灾难或自然灾害时提供额外的保护。 以此方式复制的数据在发生意外中断时将保持高可用性。 
- **安全。** 该服务将对写入到 Azure 存储的所有数据进行加密。 Azure 存储可以精细地控制谁可以访问你的数据。
- **可缩放。** Azure 存储设计为可大规模缩放以满足当今的应用程序在数据存储和性能方面的需求。 
- **托管的。** Microsoft Azure 为你处理硬件维护、更新和关键问题。
- **易访问。** 可以通过 HTTP 或 HTTPS 从世界上的任何位置访问 Azure 存储中的数据。 Microsoft 以各种语言（NET、Java、Node.js、Python、PHP、Ruby、Go 和其他语言）提供了适用于 Azure 存储的 SDK 以及成熟的 REST API。 Azure 存储支持通过 Azure PowerShell 或 Azure CLI 运行脚本。 而且，Azure 门户和 Azure 存储资源管理器提供了用于处理数据的简单可视化解决方案。  

## <a name="azure-storage-services"></a>Azure 存储服务

Azure 存储包括以下数据服务： 

- [Azure Blob](../blobs/storage-blobs-introduction.md)：适用于文本和二进制数据的可大规模缩放的对象存储。
- [Azure 文件](../files/storage-files-introduction.md)：适用于云或本地部署的托管文件共享。
- [Azure 队列](../queues/storage-queues-introduction.md)：用于在应用程序组件之间进行可靠的消息传送的消息存储。 
- [Azure 表](../tables/table-storage-overview.md)：一种 NoSQL 存储，适合用作结构化数据的无架构存储。

对每个服务的访问都通过存储帐户进行。 若要开始使用，请参阅[创建存储帐户](storage-quickstart-create-account.md)。

## <a name="blob-storage"></a>Blob 存储

Azure Blob 存储是 Microsoft 提供的适用于云的对象存储解决方案。 Blob 存储最适合存储巨量的非结构化数据，例如文本或二进制数据。 

Blob 存储最适合用于：

* 直接向浏览器提供图像或文档。
* 存储文件以供分布式访问。
* 对视频和音频进行流式处理。
* 存储用于备份和还原、灾难恢复及存档的数据。
* 存储数据以供本地或 Azure 托管服务执行分析。

可以通过 HTTP 或 HTTPS 从世界上的任何位置访问 Blob 存储中的对象。 用户或客户端应用程序可以通过 URL、[Azure 存储 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api)、[Azure PowerShell](https://docs.microsoft.com/powershell/module/azure.storage)、[Azure CLI](https://docs.microsoft.com/cli/azure/storage) 或 Azure 存储客户端库访问 Blob。 存储客户端库以多种语言提供，包括 [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage/client)、[Java](https://docs.microsoft.com/java/api/overview/azure/storage/client)、[Node.js](https://azure.github.io/azure-storage-node)、[Python](https://azure-storage.readthedocs.io/)、[PHP](https://azure.github.io/azure-storage-php/) 和 [Ruby](https://azure.github.io/azure-storage-ruby)。

有关 Blob 存储的详细信息，请参阅 [Blob 存储简介](../blobs/storage-blobs-introduction.md)。

## <a name="azure-files"></a>Azure 文件
可以通过 [Azure 文件](../files/storage-files-introduction.md)设置可用性高的网络文件共享，以便使用标准的服务器消息块 (SMB) 协议对其进行访问。 这意味着，多个 VM 可以共享启用了读取和写入访问权限的相同文件。 也可使用 REST 接口或存储客户端库来读取文件。

Azure 文件不同于公司文件共享的一点是，可以在全球任何地方使用 URL 来访问文件，只要该 URL 指向文件且包含共享访问签名 (SAS) 令牌即可。 可以生成 SAS 令牌，用于在指定时间内对私有资产进行特定访问。

文件共享适用于许多常用方案：

* 许多本地应用程序使用文件共享。 此功能可以更方便地迁移那些将数据共享到 Azure 的应用程序。 如果将文件共享装载到本地应用程序所使用的驱动器号，则应用程序中访问文件共享的那部分应尽量少做更改（如果必须进行更改的话）。

* 配置文件可以在一个文件共享上存储，从多个 VM 进行访问。 可以将一个组中多个开发人员使用的工具和实用程序存储到文件共享中，确保每个人都能找到它们并使用同一版本。

* 例如，诊断日志、指标和故障转储是三种可以写入到文件共享中供以后处理或分析的数据。

目前不支持基于 Active Directory 的身份验证和访问控制列表 (ACL)，但将来的某个时候会提供此方面的支持。 存储帐户凭据用于提供访问文件共享所需的身份验证。 这意味着，任何人只要装载了共享都具有该共享的完整读/写访问权限。

有关 Azure 文件的详细信息，请参阅 [Azure 文件简介](../files/storage-files-introduction.md)。

## <a name="queue-storage"></a>队列存储

Azure 队列服务用于存储和检索消息。 队列消息最大可以为 64 KB，一个队列可以包含数百万条消息。 队列通常用于存储需要异步处理的消息的列表。

例如，假设你需要客户能够上传图片，并且你需要创建每个图片的缩略图。 可以让客户在上传图片时等待你创建缩略图， 也可以使用队列。 当客户完成上传操作后，向队列写入一条消息。 然后通过 Azure Function 从队列检索该消息并创建缩略图。 此处理过程的每一部分都可以单独进行缩放，让你可以根据使用情况进行调整，加强控制。

有关 Azure 队列的详细信息，请参阅[队列简介](../queues/storage-queues-introduction.md)。

## <a name="table-storage"></a>表存储

Azure 表存储现在是 Azure Cosmos DB 的一部分。 若要查看 Azure 表存储文档，请参阅 [Azure 表存储概述](../tables/table-storage-overview.md)。 除了现有的 Azure 表存储服务，还有新的 Azure Cosmos DB 表 API 产品/服务，后者提供吞吐量优化表、全局分发和自动辅助索引。 若要详细了解和尝试新的高级体验，请查看 [Azure Cosmos DB 表 API](https://aka.ms/premiumtables)。

有关表存储的详细信息，请参阅 [Azure 表存储概述](../tables/table-storage-overview.md)。

## <a name="disk-storage"></a>磁盘存储

Azure 存储还包括虚拟机使用的托管和非托管磁盘功能。 有关这些功能的详细信息，请参阅[计算服务文档](https://docs.microsoft.com/azure/#pivot=products&panel=Compute)。

## <a name="types-of-storage-accounts"></a>存储帐户的类型

[!INCLUDE [storage-account-types-include](../../../includes/storage-account-types-include.md)]

有关存储帐户类型的详细信息，请参阅 [Azure 存储帐户概述](storage-account-overview.md)。 

## <a name="securing-access-to-storage-accounts"></a>保护对存储帐户的访问

到 Azure 存储的每个请求必须进行授权。 Azure 存储支持以下授权方法：

- **Blob 和队列数据的 azure Active Directory (Azure AD) 集成。** Azure 存储支持通过基于角色的访问控制 (RBAC) 的 Blob 和队列服务的身份验证和授权与 Azure AD 凭据。 授权请求与 Azure AD 建议用于卓越的安全性和易用性。 有关详细信息，请参阅[进行身份验证访问 Azure blob 和队列使用 Azure Active Directory](storage-auth-aad.md)。
- **Azure AD 授权 SMB 上 Azure 文件 （预览版）。** 通过 SMB （服务器消息块） 通过 Azure Active Directory 域服务，azure 文件支持基于标识的授权。 已加入域的 Windows 虚拟机 (Vm) 可以访问 Azure 文件共享使用 Azure AD 凭据。 有关详细信息，请参阅[SMB 上的 Azure 文件 （预览版） 的概述的 Azure Active Directory 授权](../files/storage-files-active-directory-overview.md)。
- **使用共享密钥的授权。** Azure 存储 Blob、 队列和表服务和 Azure 文件支持授权对具有共享该客户端使用授权将每个请求都使用存储帐户访问密钥进行签名的标头传递的共享密钥。 有关详细信息，请参阅[通过共享密钥进行授权](https://docs.microsoft.com/rest/api/storageservices/authorize-with-shared-key)。
- **授权使用共享访问签名 (SAS)。** 共享的访问签名 (SAS) 是一个包含可以追加到存储资源的 URI 的安全令牌的字符串。 安全令牌将封装权限和访问的时间间隔等限制。 有关详细信息，请参阅[使用共享访问签名 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。
- **对容器和 blob 的匿名访问。** 容器和其 blob 可能公开可用。 当您指定容器或 blob 是公共的时任何人都可以读取它以匿名方式;无身份验证是必需的。 有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](../blobs/storage-manage-access-to-resources.md)

## <a name="encryption"></a>加密

有两种适用于存储服务的基本加密类型。 若要详细了解安全性和加密，请参阅 [Azure 存储安全指南](storage-security-guide.md)。

### <a name="encryption-at-rest"></a>静态加密

静态 Azure 存储服务加密 (SSE) 可帮助保护数据，使组织能够信守在安全性与符合性方面所做的承诺。 使用此功能，Azure 存储可以先自动加密数据，再将数据保存到存储，并在检索之前解密数据。 加密、解密和密钥管理对于用户而言是完全透明的。


SSE 自动加密所有性能层（标准和高级）、所有部署模型（Azure 资源管理器和经典）、所有 Azure 存储服务（Blob、队列、表和文件）中的数据。 SSE 不影响 Azure 存储性能。

有关 SSE 静态加密的详细信息，请参阅[静态数据的 Azure 存储服务加密](storage-service-encryption.md)。

### <a name="client-side-encryption"></a>客户端加密

可以调用存储客户端库的方法以编程方式加密数据，然后再将数据通过网络从客户端发送到 Azure。 数据以加密方式存储，这意味着数据也是静态加密的。 读回数据时，会在收到数据信息后再将其解密。

有关客户端加密的详细信息，请参阅 [Microsoft Azure 存储的使用 .NET 的客户端加密](storage-client-side-encryption.md)。

## <a name="redundancy"></a>冗余

为了确保数据的持久性，Azure 存储会复制多个数据副本。 设置存储帐户时，可选择冗余选项。 

[!INCLUDE [storage-common-redundancy-options](../../../includes/storage-common-redundancy-options.md)]

有关灾难恢复的详细信息，请参阅 [Azure 存储中的灾难恢复和存储帐户故障转移（预览版）](storage-disaster-recovery-guidance.md)。

## <a name="transferring-data-to-and-from-azure-storage"></a>将数据传输到和移出 Azure 存储

有多个选项用于将数据移入或移出 Azure 存储。 选择哪个选项取决于数据集的大小和网络带宽。 有关详细信息，请参阅[选择 Azure 数据传输解决方案](storage-choose-data-transfer-solution.md)。

## <a name="pricing"></a>定价

有关 Azure 存储定价的详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="storage-apis-libraries-and-tools"></a>存储 API、库和工具
Azure 存储资源可以通过任何发出 HTTP/HTTPS 请求的语言来进行访问。 另外，Azure 存储还为多种主流语言提供了编程库。 这些库通过对细节进行处理简化了使用 Azure 存储的许多方面，这些细节包括同步和异步调用、操作的批处理、异常管理、自动重试、操作行为，等等。 这些库当前可供下列语言和平台以及正在筹备的其他语言和平台使用：

### <a name="azure-storage-data-api-and-library-references"></a>Azure 存储数据 API 和库参考
* [存储服务 REST API](https://docs.microsoft.com/rest/api/storageservices/)
* [适用于 .NET 的存储客户端库](https://docs.microsoft.com/dotnet/api/overview/azure/storage)
* [适用于 Java/Android 的存储空间客户端库](https://docs.microsoft.com/java/api/overview/azure/storage)
* [适用于 Node.js 的存储空间客户端库](https://docs.microsoft.com/javascript/api/azure-storage)
* [适用于 Python 的存储空间客户端库](https://github.com/Azure/azure-storage-python)
* [适用于 PHP 的存储空间客户端库](https://github.com/Azure/azure-storage-php)
* [适用于 Ruby 的存储空间客户端库](https://github.com/Azure/azure-storage-ruby)
* [适用于 C++ 的存储客户端库](https://github.com/Azure/azure-storage-cpp)

### <a name="azure-storage-management-api-and-library-references"></a>Azure 存储管理 API 和库参考
* [存储资源提供程序 REST API](https://docs.microsoft.com/rest/api/storagerp/)
* [适用于 .NET 的存储资源提供程序客户端库](https://docs.microsoft.com/dotnet/api/overview/azure/storage/management)
* [存储服务管理 REST API (Classic)](https://msdn.microsoft.com/library/azure/ee460790.aspx)

### <a name="azure-storage-data-movement-api-and-library-references"></a>Azure 存储数据移动 API 和库参考
* [存储导入/导出服务 REST API](https://docs.microsoft.com/rest/api/storageimportexport/)
* [适用于 .NET 的存储数据移动客户端库](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.datamovement)

### <a name="tools-and-utilities"></a>工具和实用程序
* [适用于存储的 Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/az.storage)
* [适用于存储的 Azure CLI Cmdlet](https://docs.microsoft.com/cli/azure/storage)
* [AzCopy 命令行实用程序](https://aka.ms/downloadazcopy)
* [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)是 Microsoft 免费提供的独立应用，适用于在 Windows、macOS 和 Linux 上以可视方式处理 Azure 存储数据。
* [Azure 存储客户端工具](../storage-explorers.md)
* [Azure 开发人员工具](https://azure.microsoft.com/tools/)

## <a name="next-steps"></a>后续步骤

若要启动并运行 Azure 存储，请参阅[创建存储帐户](storage-quickstart-create-account.md)。
