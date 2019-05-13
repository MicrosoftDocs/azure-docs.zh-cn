---
title: 使用 Azure 门户管理 Azure 资源管理器组 |Microsoft Docs
description: 使用 Azure 门户管理 Azure 资源管理器组。
services: azure-resource-manager,azure-portal
documentationcenter: ''
author: mumian
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2019
ms.author: jgao
ms.openlocfilehash: 559c1874c119eabef2c35a954961c1e669df3c06
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507232"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-the-azure-portal"></a>使用 Azure 门户管理 Azure 资源管理器资源组

了解如何将 [Azure 门户](https://portal.azure.com)与 [Azure 资源管理器](resource-group-overview.md)配合使用来管理 Azure 资源组。 若要管理 Azure 资源，请参阅[使用 Azure 门户管理 Azure 资源](./manage-resources-portal.md)。

有关资源组管理的其他文章：

- [使用 Azure CLI 管理 Azure 资源组](./manage-resources-cli.md)
- [使用 Azure PowerShell 管理 Azure 资源组](./manage-resources-powershell.md)

[!INCLUDE [Handle personal data](../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-resource-group"></a>什么是资源组

资源组是保管 Azure 解决方案的相关资源的容器。 资源组可以包含解决方案的所有资源，也可以只包含想要作为组来管理的资源。 根据对组织有利的原则，决定如何将资源分配到资源组。 通常可将共享相同生命周期的资源添加到同一资源组，以便将其作为一个组轻松部署、更新和删除。

资源组存储与资源有关的元数据。 因此，当指定资源组的位置时，也就指定了元数据的存储位置。 出于合规性原因，可能需要确保数据存储在某一特定区域。

” 资源组存储有关资源的元数据。 指定资源组的位置时，也会指定元数据的存储位置。

## <a name="create-resource-groups"></a>创建资源组

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 选择“资源组”

    ![添加资源组](./media/manage-resource-groups-portal/manage-resource-groups-add-group.png)
3. 选择 **添加** 。
4. 输入以下值：

   - **订阅**：选择 Azure 订阅。 
   - **资源组**：输入新的资源组名称。 
   - **区域**：选择某一 Azure 位置，如**美国中部**。

     ![创建资源组](./media/manage-resource-groups-portal/manage-resource-groups-create-group.png)
5. 选择“查看 + 创建”
6. 选择“创建”。 创建资源组需要几秒钟的时间。
7. 在顶部菜单中选择“刷新”以刷新资源组列表，然后选择新建的资源组将其打开。 或者，选择顶部的“通知”（钟形图标），然后选择“转到资源组”打开新建的资源组

    ![转到资源组](./media/manage-resource-groups-portal/manage-resource-groups-add-group-go-to-resource-group.png)

## <a name="list-resource-groups"></a>列出资源组

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 若要列出资源组，请选择“资源组”

    ![浏览资源组](./media/manage-resource-groups-portal/manage-resource-groups-list-groups.png)

3. 若要自定义显示的资源组信息，请选择“编辑列”。 以下屏幕截图显示了可添加到画面中的附加列：

## <a name="open-resource-groups"></a>打开资源组

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 选择“资源组”。
3. 选择要打开的资源组。

## <a name="delete-resource-groups"></a>删除资源组

1. 打开要删除的资源组。  参阅[打开资源组](#open-resource-groups)。
2. 选择“删除资源组”。

    ![删除 Azure 资源组](./media/manage-resource-groups-portal/delete-group.png)

若要详细了解 Azure 资源管理器如何控制资源的删除，请参阅 [Azure 资源管理器资源组的删除](./resource-group-delete.md)。

## <a name="deploy-resources-to-a-resource-group"></a>将资源部署到资源组

创建资源管理器模板后，可以使用 Azure 门户部署 Azure 资源。 有关如何创建模板，请参阅[快速入门：使用 Azure 门户创建和部署 Azure 资源管理器模板](./resource-manager-quickstart-create-templates-use-the-portal.md)。 有关如何使用门户部署模板，请参阅[使用资源管理器模板和 Azure 门户部署资源](resource-group-template-deploy-portal.md)。

## <a name="move-to-another-resource-group-or-subscription"></a>移到另一个资源组或订阅

可将组中的资源移到另一个资源组。 有关详细信息，请参阅[将资源移到新资源组或订阅](resource-group-move-resources.md)。

## <a name="lock-resource-groups"></a>锁定资源组

锁定可以防止组织中的其他用户意外删除或修改关键资源，例如 Azure 订阅、资源组或资源。 

1. 打开要删除的资源组。  参阅[打开资源组](#open-resource-groups)。
2. 在左窗格中选择“锁定”。
3. 若要将锁添加到资源组，请选择“添加”。
4. 输入“锁定名”、“锁类型”和“备注”。 锁类型包括“只读”和“删除”。

    ![锁定 Azure 资源组](./media/manage-resource-groups-portal/manage-resource-groups-add-lock.png)

有关详细信息，请参阅[锁定资源以防止意外更改](./resource-group-lock-resources.md)。

## <a name="tag-resource-groups"></a>标记资源组

可以将标记应用到资源组和资源，以按照逻辑组织资产。 有关信息，请参阅[使用标记组织 Azure 资源](./resource-group-using-tags.md#portal)。

## <a name="export-resource-groups-to-templates"></a>将资源组导出到模板

有关导出模板的信息，请参阅[单个和多资源导出到模板中-门户](export-template-portal.md)。

### <a name="fix-export-issues"></a>修复导出问题

并非所有资源类型都支持导出模板功能。 仅当从资源组导出（而不是从部署历史记录导出）时，才会看到导出问题。 如果上一个部署能够准确地代表资源组的当前状态，则应从部署历史记录而非资源组中导出模板。 只有在已更改资源组且更改未在单个模板中定义时，才应从资源组导出。

若要解决导出问题，请手动将缺少的资源添加回模板中。 错误消息内指出了无法导出的资源类型。 请在[模板引用](/azure/templates/)中查找该资源类型。 例如，若要手动添加虚拟网关，请参阅 [Microsoft.Network/virtualNetworkGateways 模板引用](/azure/templates/microsoft.network/virtualnetworkgateways)。 模板参考文档中提供了用于将资源添加到模板的 JSON。

获取资源的 JSON 格式后，需要获取资源值。 可以在 REST API 中对资源类型使用 GET 操作，从而查看资源值。 例如，若要获取虚拟网络网关值，请参阅[虚拟网络网关 - Get](/rest/api/network-gateway/virtualnetworkgateways/get)。

## <a name="manage-access-to-resource-groups"></a>管理对资源组的访问

可以在 Azure 中通过[基于角色的访问控制 (RBAC)](../role-based-access-control/overview.md) 这种方式管理对资源的访问。 有关详细信息，请参阅[使用 RBAC 和 Azure 门户管理访问权限](../role-based-access-control/role-assignments-portal.md)。

## <a name="next-steps"></a>后续步骤

- 若要了解 Azure 资源管理器，请参阅 [Azure 资源管理器概述](./resource-group-overview.md)。
- 若要了解资源管理器模板语法，请参阅[了解 Azure 资源管理器模板的结构和语法](./resource-group-authoring-templates.md)。
- 若要了解如何开发模板，请参阅[分步教程](/azure/azure-resource-manager/)。
- 若要查看 Azure 资源管理器模板架构，请参阅[模板参考](/azure/templates/)。