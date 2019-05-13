---
title: Azure Cosmos DB 中使用 Azure Active Directory 集成的基于角色的访问控制
description: 了解 Azure Cosmos DB 如何提供与 Active directory 集成 (RBAC) 的数据库保护。
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: e43341e59e807fdc20033d909beadb9d582149b5
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65078935"
---
# <a name="role-based-access-control-in-azure-cosmos-db"></a>Azure Cosmos DB 中的基于角色的访问控制

Azure Cosmos DB 的 Azure Cosmos DB 中的常见管理方案提供内置的基于角色的访问控制 (RBAC)。 Azure Active Directory 中有一个配置文件的人员可以将这些 RBAC 角色分配给用户、 组、 服务主体或托管标识，以授予或拒绝访问的资源和 Azure Cosmos DB 资源执行操作。 角色分配的作用域控制平面的访问，它包括访问 Azure Cosmos 帐户、 数据库、 容器，并提供 （吞吐量）。

## <a name="built-in-roles"></a>内置角色

以下是 Azure Cosmos DB 支持的内置角色：

|**内置角色**  |**说明**  |
|---------|---------|
|[DocumentDB 帐户参与者](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)   | 可管理 Azure Cosmos DB 帐户。  |
|[Cosmos DB 帐户读取器](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)  | 可以读取 Azure Cosmos DB 帐户数据。        |
|[Cosmos 备份操作员](../role-based-access-control/built-in-roles.md#cosmosbackupoperator)     |  可以提交还原请求的 Azure Cosmos 数据库或容器。       |
|[Cosmos DB 运算符](../role-based-access-control/built-in-roles.md#cosmos-db-operator)  | 可以预配 Azure Cosmos 帐户、 数据库和容器，但不能访问所需访问的数据的密钥。         |

> [!IMPORTANT]
> Azure Cosmos DB 中的 RBAC 支持适用于控制平面操作。 数据平面操作，使用主密钥或资源令牌进行保护。 若要了解详细信息，请参阅[安全访问 Azure Cosmos DB 中的数据](secure-access-to-data.md)

## <a name="identity-and-access-management-iam"></a>标识和访问管理 (IAM)

**访问控制 (IAM)** 窗格在 Azure 门户中的用于配置对 Azure Cosmos 资源的基于角色的访问控制。 角色不适用于用户、 组、 服务主体和 Active Directory 中的托管的标识。 对于个人和组，可以使用内置角色或自定义角色。 下面的屏幕截图显示了 Active Directory 集成 (RBAC) 在 Azure 门户中使用访问控制 (IAM):

![Azure 门户中的访问控制 (IAM) - 演示数据库安全性](./media/role-based-access-control/database-security-identity-access-management-rbac.png)

## <a name="custom-roles"></a>自定义角色

除了内置角色中，用户还可以创建[自定义角色](../role-based-access-control/custom-roles.md)在 Azure 中并将这些角色应用到服务主体的 Active Directory 租户中的所有订阅。 自定义角色为用户提供了一种方法来创建具有一组自定义资源提供程序操作的 RBAC 角色定义。 若要了解哪些操作是可用于生成自定义角色的 Azure Cosmos DB，请参阅[Azure Cosmos DB 资源提供程序操作](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb)

## <a name="next-steps"></a>后续步骤

- [什么是 Azure 资源的基于角色的访问控制 (RBAC)](../role-based-access-control/overview.md)
- [Azure 资源的自定义角色](../role-based-access-control/custom-roles.md)
- [Azure Cosmos DB 资源提供程序操作](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb)
