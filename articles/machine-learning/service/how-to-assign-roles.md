---
title: 管理 Azure 机器学习工作区中的角色
titleSuffix: Azure Machine Learning service
description: 了解如何访问使用基于角色的访问控制 (RBAC) 的 Azure 机器学习服务工作区。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: larryfr
author: Blackmist
ms.date: 02/20/2019
ms.custom: seodec18
ms.openlocfilehash: f5cc86c40f56cee28ff6a6912acd67961285bffc
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025315"
---
# <a name="manage-access-to-an-azure-machine-learning-workspace"></a>管理对 Azure 机器学习工作区的访问

在本文中，您将了解如何管理对 Azure 机器学习工作区的访问。 [基于角色的访问控制 (RBAC)](/azure/role-based-access-control/overview)用于管理 Azure 资源的访问权限。 Azure Active Directory 中的用户分配特定角色，授予对资源的访问。 Azure 提供的内置角色和创建自定义角色的功能。

## <a name="default-roles"></a>默认角色

Azure 机器学习工作区是一种 Azure 资源。 与其他 Azure 资源一样创建一个新的 Azure 机器学习工作区时，它附带了三个默认角色。 可以将用户添加到工作区并将其分配到这些内置角色之一。

| 角色 | 访问级别 |
| --- | --- |
| **读取者** | 在工作区中的只读操作。 读取器可以列出和查看在工作区中，资产，但无法创建或更新这些资产。 |
| **参与者** | 查看、 创建、 编辑或删除 （如果适用） 的工作区中的资产。 例如，参与者可以创建试验、 创建或附加的计算群集、 提交运行，和部署 web 服务。 |
| **所有者** | 完全访问权限的工作区，包括能够查看、 创建、 编辑或删除 （如果适用） 的工作区中的资产。 此外，还可以更改角色分配。 |

> [!IMPORTANT]
> 可以在 Azure 中的多个级别到作用域角色的访问权限。 例如，具有对工作区所有者访问权限的人员可能不具有到包含的工作区的资源组的所有者访问权限。 有关详细信息，请参阅[如何 RBAC 适用](/azure/role-based-access-control/overview#how-rbac-works)。

有关特定的内置角色的详细信息，请参阅[适用于 Azure 的内置角色](/azure/role-based-access-control/built-in-roles)。

## <a name="manage-workspace-access"></a>管理工作区的访问权限

如果您的工作区所有者，你可以添加和删除工作区的角色。 此外可以向用户分配角色。 使用以下链接来了解如何管理访问权限：
- [Azure 门户 UI](/azure/role-based-access-control/role-assignments-portal)
- [PowerShell](/azure/role-based-access-control/role-assignments-powershell)
- [Azure CLI](/azure/role-based-access-control/role-assignments-cli)
- [REST API](/azure/role-based-access-control/role-assignments-rest)
- [Azure 资源管理器模板](/azure/role-based-access-control/role-assignments-template)

如果已安装[Azure 机器学习 CLI](reference-azure-machine-learning-cli.md)，还可以使用 CLI 命令为用户分配角色。

```azurecli-interactive 
az ml workspace share -n <workspace_name> -g <resource_group_name> --role <role_name> --user <user_corp_email_address>
```

`user`字段是工作区父订阅所在的 Azure Active Directory 的实例中的现有用户的电子邮件地址。 下面是如何使用此命令的示例：

```azurecli-interactive 
az ml workspace share -n my_workspace -g my_resource_group --role Contributor --user jdoe@contoson.com
```

## <a name="create-custom-role"></a>创建自定义角色

如果内置角色的数量不足，可以创建自定义角色。 自定义角色可能具有读取、 写入、 删除和计算该工作区中的资源权限。 您可以使该角色可用在特定工作区级别、 特定资源组级别或特定的订阅级别。

> [!NOTE]
> 您必须是资源的该级别处的所有者才能创建该资源中的自定义角色。

若要创建自定义角色，请先构造指定的权限和角色作用域的角色定义 JSON 文件。 下面的示例定义名为"数据科学家"作用域特定的工作区级别的自定义角色：

`data_scientist_role.json`：
```json
{
    "Name": "Data Scientist",
    "IsCustom": true,
    "Description": "Can run experiment but can't create or delete compute.",
    "Actions": ["*"],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/*/delete",
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace_name>"
    ]
}
```

您可以更改`AssignableScopes`字段，以便在订阅级别、 资源组级别或特定工作区级别设置此自定义角色的作用域。

此自定义角色可以执行以下操作除外的工作区中的所有内容操作：

- 它不能创建或更新的计算资源。
- 它不能删除计算资源。
- 它不能添加、 删除或更改角色分配。
- 它不能删除工作区。

若要部署此自定义角色，请使用以下 Azure CLI 命令：

```azurecli-interactive 
az role definition create --role-definition data_scientist_role.json
```

在部署后，此角色变为可用的指定工作区中。 现在可以添加和分配此角色在 Azure 门户中。 或者，可以向用户分配此角色，通过使用`az ml workspace share`CLI 命令：

```azurecli-interactive
az ml workspace share -n my_workspace -g my_resource_group --role "Data Scientist" --user jdoe@contoson.com
```


有关详细信息，请参阅[Azure 资源的自定义角色](/azure/role-based-access-control/custom-roles)。

## <a name="next-steps"></a>后续步骤

- [企业安全概述](concept-enterprise-security.md)
- [安全地运行试验和虚拟网络中的推断](how-to-enable-virtual-network.md)
- [教程：训练模型](tutorial-train-models-with-aml.md)
