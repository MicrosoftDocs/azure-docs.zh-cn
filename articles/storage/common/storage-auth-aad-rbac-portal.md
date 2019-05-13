---
title: 使用 Azure 门户管理 Azure AD 对使用 RBAC 的 Azure 存储的 blob 和队列数据的访问权限 |Microsoft Docs
description: 在 Azure 门户中使用基于角色的访问控制 (RBAC) 向安全主体分配容器和队列的访问权限。 Azure 存储支持通过 Azure AD 使用内置和自定义的 RBAC 角色进行身份验证。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: ebb67ff295a934b4e125e298018c27a095adc590
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153870"
---
# <a name="grant-access-to-azure-blob-and-queue-data-with-rbac-in-the-azure-portal"></a>授予对 Azure blob 和队列数据使用 RBAC 在 Azure 门户中访问权限

Azure Active Directory (Azure AD) 通过[基于角色的访问控制 (RBAC)](../../role-based-access-control/overview.md) 授权访问受保护的资源。 Azure 存储定义包含常见的用来访问 blob 或队列数据的权限集的内置 RBAC 角色的一组。 

将 RBAC 角色分配到 Azure AD 安全主体后，Azure 会向该安全主体授予对这些资源的访问权限。 可以将访问权限限定于订阅、资源组、存储帐户、单个容器或队列级别。 Azure AD 安全主体可能是用户、 组、 应用程序服务主体，或[管理 Azure 资源的标识](../../active-directory/managed-identities-azure-resources/overview.md)。

本文介绍如何使用 Azure 门户分配 RBAC 角色。 Azure 门户提供一个简单的界面用于分配 RBAC 角色，以及管理对存储资源的访问权限。 也可以使用 Azure 命令行工具或 Azure 存储管理 API 来为 Blob 和队列资源分配 RBAC 角色。 有关存储资源的 RBAC 角色的详细信息，请参阅[使用 Azure Active Directory 验证对 Azure Blob 和队列的访问](storage-auth-aad.md)。 

## <a name="rbac-roles-for-blobs-and-queues"></a>Blob 和队列的 RBAC 角色

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>确定资源范围 

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="assign-rbac-roles-using-the-azure-portal"></a>使用 Azure 门户分配 RBAC 角色

确定适当的作用域的角色分配之后，导航到该资源在 Azure 门户中。 显示**访问控制 (IAM)** 设置对于资源，并按照以下说明用于管理角色分配：

1. 分配相应的 Azure 存储 RBAC 角色，授予对 Azure AD 安全主体访问权限。

1. 可将 Azure 资源管理器[读取者](../../role-based-access-control/built-in-roles.md#reader)角色分配给需要通过 Azure 门户使用其 Azure AD 凭据访问容器或队列的用户。 

以下各部分更详细地说明了其中的每个步骤。

### <a name="assign-a-built-in-rbac-role"></a>分配内置的 RBAC 角色

在将角色分配到安全主体之前，请务必考虑所要授予的权限的范围。 查看[确定资源范围](#determine-resource-scope)部分以确定适当的范围。

此处所示的过程将分配限定于容器的角色，但你可以按照相同的步骤分配限定于队列的角色： 

1. 在 [Azure 门户](https://portal.azure.com)中，导航到存储帐户并显示该帐户的“概述”。
1. 在“服务”下选择“Blob”。 
1. 找到要针对其分配角色的容器，并显示该容器的设置。 
1. 选择“访问控制(IAM)”以显示容器的访问控制设置。 选择“角色分配”选项卡以查看角色分配列表。

    ![显示容器访问控制设置的屏幕截图](media/storage-auth-aad-rbac-portal/portal-access-control-container.png)

1. 单击“添加角色分配”按钮以添加一个新角色。
1. 在“添加角色分配”窗口中，选择要分配的 Azure 存储角色。 然后通过搜索找到要为其分配该角色的安全主体。

    ![显示如何分配 RBAC 角色的屏幕截图](media/storage-auth-aad-rbac-portal/add-rbac-role.png)

1. 单击“ **保存**”。 分配有该角色的标识列出在该角色下。 例如，下图显示添加的用户现在对名为 *sample-container* 的容器中的数据具有读取权限。

    ![显示已分配到某个角色的用户列表的屏幕截图](media/storage-auth-aad-rbac-portal/container-scoped-role.png)

可以遵循类似的步骤来将分配到存储帐户、 资源组或订阅作用域的角色。

> [!NOTE]
> 作为 Azure 存储帐户的所有者，系统不会自动向你分配数据访问权限。 你必须为自己显式分配一个用于 Azure 存储的 RBAC 角色。 可以在订阅、资源组、存储帐户、容器或队列级别分配它。
> 
> 无法分配作用域容器或队列，如果存储帐户已启用的分层命名空间的角色。

### <a name="assign-the-reader-role-for-portal-access"></a>分配“读取者”角色以访问门户

将 Azure 存储的内置或自定义角色分配到某个安全主体时，会向该安全主体授予权限，以便针对存储帐户中的数据执行操作。 内置的“数据读取者”角色提供对容器或队列中的数据的读取权限，而内置的“数据参与者”角色提供对容器或队列的读取、写入和删除权限。 权限范围限定为指定的资源。  

例如，如果你将分配**存储 Blob 数据参与者**用户 Mary 在名为的容器级别的角色**示例容器**、 然后 Mary 授予读取、 写入和删除中的 blob 的所有访问权限该容器。

但是，如果 Mary 希望在 Azure 门户中查看某个 Blob，“存储 Blob 数据参与者”角色本身无法提供足够的权限用于在门户中导航，因此 Mary 无法查看该 Blob。 必须拥有其他 Azure AD 权限才能在门户中导航和查看门户中显示的其他资源。

如果用户需要在 Azure 门户中访问 Blob，请在存储帐户或更高的级别向这些用户分配一个额外的 RBAC 角色：[读取者](../../role-based-access-control/built-in-roles.md#reader)角色。 “读取者”角色是一个 Azure 资源管理器角色，可让用户查看存储帐户资源，但不允许修改这些资源。 该角色不提供对 Azure 存储中的数据的读取权限，而只提供对帐户管理资源的读取权限。

请遵循以下步骤分配“读取者”角色，使用户能够在 Azure 门户中访问 Blob。 在此示例中，分配范围限定为存储帐户：

1. 在 [Azure 门户](https://portal.azure.com)中导航到存储帐户。
1. 选择“访问控制(IAM)”以显示存储帐户的访问控制设置。 选择“角色分配”选项卡以查看角色分配列表。
1. 在“添加角色分配”窗口中，选择“读取者”角色。 
1. 在“将访问权限分配给”字段中，选择“Azure AD 用户、组或服务主体”。
1. 通过搜索找到要为其分配该角色的安全主体。
1. 保存角色分配。

> [!NOTE]
> 只有必要对需要使用 Azure 门户访问 Blob 或队列的用户分配“读取者”角色。 

## <a name="next-steps"></a>后续步骤

- 有关存储资源的 RBAC 角色的详细信息，请参阅[使用 Azure Active Directory 验证对 Azure Blob 和队列的访问](storage-auth-aad.md)。 
- 若要详细了解 RBAC，请参阅[什么是基于角色的访问控制 (RBAC)？](../../role-based-access-control/overview.md)
- 若要了解如何使用 Azure PowerShell、Azure CLI 或 REST API 分配和管理 RBAC 角色分配，请参阅以下文章：
    - [使用 Azure PowerShell 管理基于角色的访问控制 (RBAC)](../../role-based-access-control/role-assignments-powershell.md)
    - [使用 Azure CLI 管理基于角色的访问控制 (RBAC)](../../role-based-access-control/role-assignments-cli.md)
    - [使用 REST API 管理基于角色的访问控制 (RBAC)](../../role-based-access-control/role-assignments-rest.md)
- 若要了解如何从存储应用程序内授权访问容器和队列，请参阅[将 Azure AD 与 Azure 存储应用程序配合使用](storage-auth-aad-app.md)。
