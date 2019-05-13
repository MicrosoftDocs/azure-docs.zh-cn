---
title: 授权访问 Azure 存储 | Microsoft Docs
description: 了解授权访问 Azure 存储的各种方式，包括 Azure Active Directory、共享密钥身份验证或共享访问签名。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 9e8c9755c444ca7b81891f5f83164bc51aa694eb
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147067"
---
# <a name="authorizing-access-to-azure-storage"></a>授权访问 Azure 存储

每次访问存储帐户中的数据时，客户端都会通过 HTTP/HTTPS 向 Azure 存储发出请求。 每个对安全资源的请求都必须经过授权，以便服务确保客户端具有访问数据所需的权限。 Azure 存储提供以下用于授权访问安全资源的选项：

- 用于 Blob 和队列的 **Azure Active Directory (Azure AD) 集成**。 Azure AD 提供基于角色的访问控制 (RBAC)，用于精细地控制客户端对存储帐户中资源的访问权限。 有关详细信息，请参阅[使用 Azure Active Directory 对 Azure 存储请求进行身份验证](storage-auth-aad.md)。
- 用于 blob、文件、队列和表的**共享密钥授权**。 使用共享密钥的客户端会随使用存储帐户访问密钥签名的每个请求传递一个标头。 有关详细信息，请参阅[通过共享密钥进行授权](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-shared-key/)。
- 用于 blob、文件、队列和表的**共享访问签名**。 共享访问签名 (SAS) 针对存储帐户中的资源提供有限的委托访问权限。 通过对签名的有效时间间隔或对它授予的权限添加约束，可灵活地管理访问权限。 有关详细信息，请参阅[使用共享访问签名 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。
- 用于容器和 blob 的**匿名公共读取访问**。 无需授权。 有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](../blobs/storage-manage-access-to-resources.md)。  

默认情况下，Azure 存储中的所有资源都受到保护，并且只能由帐户所有者使用。 虽然你可以使用上述任何授权策略为客户端授予访问存储帐户资源的权限，但 Microsoft 建议尽可能使用 Azure AD，以获得最大的安全性和易用性。 



