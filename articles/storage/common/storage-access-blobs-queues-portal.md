---
title: 使用 Azure 门户来访问 blob 或队列数据-Azure 存储
description: 访问 blob 或队列数据使用 Azure 门户，门户会进行请求到 Azure 存储在后台时。 对 Azure 存储这些请求进行身份验证和授权使用你的 Azure AD 帐户或存储帐户访问密钥。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/19/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 00f34fa9a1932aebd467163e0ed7441c993387df
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154012"
---
# <a name="use-the-azure-portal-to-access-blob-or-queue-data"></a>使用 Azure 门户来访问 blob 或队列数据

当访问 blob 或队列数据使用[Azure 门户](https://portal.azure.com)，门户在后台向 Azure 存储器发出请求。 对 Azure 存储这些请求进行身份验证和授权使用你的 Azure AD 帐户或存储帐户访问密钥。 门户指示哪种身份验证方法使用的，并使你能够两者之间切换，如果您具有相应权限。  

## <a name="permissions-needed-to-access-blob-or-queue-data"></a>访问 blob 或队列的数据所需的权限

具体取决于如何进行身份验证访问 Azure 门户中的 blob 或队列数据，你将需要特定权限。 在大多数情况下，通过基于角色的访问控制 (RBAC) 提供这些权限。 有关 RBAC 的详细信息，请参阅[什么是基于角色的访问控制 (RBAC)？](../../role-based-access-control/overview.md)。

### <a name="account-access-key"></a>帐户访问密钥

若要使用的帐户访问密钥访问 blob 和队列数据，必须具有包括 RBAC 操作的 RBAC 角色分配给你**Microsoft.Storage/storageAccounts/listkeys/action**。 此 RBAC 角色可以是内置或自定义角色。 支持的内置角色**Microsoft.Storage/storageAccounts/listkeys/action**包括：

- Azure 资源管理器[所有者](../../role-based-access-control/built-in-roles.md#owner)角色
- Azure 资源管理器[参与者](../../role-based-access-control/built-in-roles.md#contributor)角色
- [存储帐户参与者](../../role-based-access-control/built-in-roles.md#storage-account-contributor)角色

当您尝试访问在 Azure 门户中的 blob 或队列数据时，在门户首先检查是否已分配的角色**Microsoft.Storage/storageAccounts/listkeys/action**。 如果您分配了具有此操作的角色，在门户对用于访问 blob 和队列数据使用的帐户密钥。 如果您不被赋予此操作的角色，门户会尝试使用你的 Azure AD 帐户访问的数据。

> [!NOTE]
> 服务管理员和共同管理员的经典订阅管理员角色包括的 Azure 资源管理器的等效[所有者](../../role-based-access-control/built-in-roles.md#owner)角色。 **所有者**角色包括所有操作，包括**Microsoft.Storage/storageAccounts/listkeys/action**，因此具有其中一种管理角色的用户还可以访问与 blob 和队列数据帐户密钥。 有关详细信息，请参阅[经典订阅管理员角色](../../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles)。

### <a name="azure-ad-account"></a>Azure AD 帐户

若要从 Azure 门户中使用 Azure AD 帐户访问 blob 或队列数据，必须为你满足这两个以下语句：

- 你已分配了 Azure 资源管理器[读取器](../../role-based-access-control/built-in-roles.md#reader)角色，至少，作用域内的存储帐户级别或更高版本。 **读取器**角色授予的最受限制的权限，但还可授予对存储帐户管理资源的访问权限的另一个 Azure 资源管理器角色。
- 你分配了可以提供对 blob 或队列数据的访问的内置或自定义角色。

**读取器**角色分配或另一个 Azure 资源管理器角色分配是必需，以便用户可以查看和导航 Azure 门户中的存储帐户管理资源。 授予对 blob 或队列数据的访问的 RBAC 角色不授予对存储帐户管理资源的访问。 若要访问门户中的 blob 或队列数据，用户需要有权导航存储帐户资源。 有关此要求的详细信息，请参阅[分配读者角色以访问门户](../common/storage-auth-aad-rbac-portal.md#assign-the-reader-role-for-portal-access)。

支持对 blob 或队列数据的访问的内置角色包括：

- [存储 Blob 数据所有者](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner):Azure 数据湖存储第 2 代的 POSIX 访问控制。
- [存储 Blob 数据参与者](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor):Blob 的读取/写入/删除权限。
- [存储 Blob 数据读取器](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader):Blob 的只读权限。
- [存储队列数据参与者](../../role-based-access-control/built-in-roles.md#storage-queue-data-contributor):队列的读取/写入/删除权限。
- [存储队列数据读取器](../../role-based-access-control/built-in-roles.md#storage-queue-data-reader):队列的只读权限。
    
自定义角色可以支持相同的权限提供的内置角色的不同组合。 有关创建自定义 RBAC 角色的详细信息，请参阅[Azure 资源的自定义角色](../../role-based-access-control/custom-roles.md)并[了解有关 Azure 资源的角色定义](../../role-based-access-control/role-definitions.md)。

> [!NOTE]
> 不支持列出经典订阅管理员角色的队列。 若要列出队列，用户必须具有分配给他们的 Azure 资源管理器**读取器**角色**存储队列数据读取器**角色，或**存储队列数据参与者**角色。

## <a name="navigate-to-blobs-or-queues-in-the-portal"></a>导航到 blob 或在门户中的队列

若要在门户中查看 blob 或队列数据，请导航到**概述**存储帐户，然后单击链接获得**Blob**或**队列**。 或者您可以导航到**Blob 服务**并**队列服务**菜单中的部分。 

![导航到 Azure 门户中的 blob 或队列数据](media/storage-access-blobs-queues-portal/blob-queue-access.png)

## <a name="determine-the-current-authentication-method"></a>确定当前的身份验证方法

当你导航到容器或队列时，Azure 门户将指示是否当前使用的帐户访问密钥或 Azure AD 帐户进行身份验证。

在本部分中的示例演示访问容器和其 blob，但访问队列和其消息，或列出队列时，门户会显示相同的消息。

### <a name="account-access-key"></a>帐户访问密钥

如果要进行身份验证使用的帐户访问密钥，则会看到**访问密钥**指定为在门户中的身份验证方法：

![当前正在使用的帐户密钥访问容器数据](media/storage-access-blobs-queues-portal/auth-method-access-key.png)

若要切换到使用 Azure AD 帐户，请单击图中突出显示的链接。 如果必须通过分配给你的 RBAC 角色的相应权限，您可以继续。 但是，如果您没有适当的权限，您将看到类似于以下一条错误消息：

![显示 Azure AD 帐户不支持访问错误](media/storage-access-blobs-queues-portal/auth-error-azure-ad.png)

请注意，任何 blob 出现在列表中，是否你的 Azure AD 帐户缺少权限来查看它们。 单击**切换到访问密钥**链接以访问密钥用于再次身份验证。

### <a name="azure-ad-account"></a>Azure AD 帐户

如果要进行身份验证使用 Azure AD 帐户，则会看到**Azure AD 用户帐户**指定为在门户中的身份验证方法：

![当前正在访问使用 Azure AD 帐户的容器数据](media/storage-access-blobs-queues-portal/auth-method-azure-ad.png)

若要切换到使用帐户访问密钥，请单击图中突出显示的链接。 如果你有权访问的帐户密钥，然后你可以继续。 但是，如果您没有对帐户密钥的访问，将看到一条错误消息如下：

![显示是否没有帐户密钥访问错误](media/storage-access-blobs-queues-portal/auth-error-access-key.png)

请注意，任何 blob 出现在列表中，是否你没有帐户密钥的访问权限。 单击**切换到 Azure AD 用户帐户**链接用于你的 Azure AD 帐户进行重新身份验证。

## <a name="next-steps"></a>后续步骤

- [访问 Azure blob 和队列使用 Azure Active Directory 进行身份验证](storage-auth-aad.md)
- [授予对 Azure 容器和 Azure 门户中使用 RBAC 的队列的访问权限](storage-auth-aad-rbac-portal.md)
- [授予对 Azure blob 和队列数据与使用 Azure CLI 的 RBAC 访问权限](storage-auth-aad-rbac-cli.md)
- [授予对 Azure blob 和队列数据与使用 PowerShell 的 RBAC 访问权限](storage-auth-aad-rbac-powershell.md)
