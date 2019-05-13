---
title: Azure 区块链服务联盟
description: ''
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: e745a4ee4789ef46a61b5cb0bbf806c41ef631ec
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027910"
---
# <a name="azure-blockchain-service-consortium"></a>Azure 区块链服务联盟

使用 Azure 区块链服务，你可以创建专用联盟其中每个区块链网络可将限制为在网络中的特定参与者的区块链网络。 仅专用联盟区块链网络中的参与者可以查看和使用区块链进行交互。 在 Azure 区块链服务中的联盟网络可以包含两种类型的成员的参与者角色：

* **管理员**-特权可以执行联盟管理操作，它可以参与区块链事务的参与者。

* **用户**-不能执行任何联盟管理操作，但可以参与区块链事务的参与者。

联盟网络可以是参与者角色的组合，并且可以包含任意数目的每个角色类型。 必须有至少一个管理员。

下图显示了具有多个参与者的联盟网络：

![专用联盟网络关系图](./media/consortium/network-diagram.png)

借助联盟 Azure 区块链服务中的管理，你可以管理联盟网络中的参与者。 管理联合会的基于网络的一致性模型。 在当前的预览版本中，Azure 区块链服务提供联盟管理集中式的一致性模型。 任何特权的参与者具有管理角色的用户可能需要联盟管理操作，例如添加或从网络删除参与者。

## <a name="roles"></a>角色

联盟中的参与者可以是个人或组织和用户角色或管理员角色可以分配。 下表列出了两个角色之间的大致差异：

| 操作 | 用户角色 | 管理员角色
|--------|:----:|:------------:|
| 创建新的成员 | 是 | 是 |
| 邀请新成员 | 否 | 是 |
| 设置或更改成员参与者角色 | 否 | 是 |
| 更改成员的显示名称 | 仅为自己的成员 | 仅为自己的成员 |
| 删除成员 | 仅为自己的成员 | 是 |
| 参与区块链事务 | 是 | 是 |

### <a name="user-role"></a>用户角色

用户不具有任何管理员功能的联盟参与者。 它们不能参与管理与联盟的成员。 用户可以更改其成员的显示名称，并可以将自己从联盟移除。

### <a name="administrator"></a>管理员

管理员可以管理联盟中的成员。 管理员可以邀请成员、 删除成员，或更新联盟中的成员角色。
必须始终有至少一个管理员联盟中。 离开联盟前，最后一个管理员必须作为管理员角色指定另一个参与者。

## <a name="managing-members"></a>管理成员

只有管理员可以邀请其他参与者加入联盟。 管理员邀请参与者使用其 Azure 订阅 id。

后邀请时，参与者可以通过部署 Azure 区块链服务中的新成员加入区块链联盟。 若要查看并加入受邀的联盟，必须指定网络管理员邀请中使用的同一 Azure 订阅 ID。

管理员可以从联盟，包括其他管理员删除任何参与者。 成员只可以删除自己从联盟。

## <a name="consortium-management-smart-contract"></a>联盟管理智能协定

在 Azure 区块链服务中的联盟管理是通过联盟管理智能协定。 在将新区块链成员时，智能协定会自动部署到你的节点。

可以在 Azure 门户中查看根联盟管理智能协定的地址。 **RootContract 地址**区块链成员的概述部分中。

![RootContract address](./media/consortium/rootcontract-address.png)

可以与使用联盟管理联盟管理智能协定进行交互[PowerShell 模块](manage-consortium-powershell.md)，Azure 门户中，或直接通过智能协定使用生成以太坊 Azure 区块链服务帐户。

## <a name="ethereum-account"></a>以太坊帐户

创建成员时，被创建一个以太坊帐户密钥。 Azure 区块链服务使用该密钥来创建与联盟管理相关的事务。 自动由 Azure 区块链服务管理以太坊帐户密钥。

可以在 Azure 门户中查看成员帐户。 成员帐户是在区块链成员的概述部分中。

![成员帐户](./media/consortium/member-account.png)

您可以通过单击成员帐户，并输入新密码重置以太坊帐户。 将重置以太坊帐户地址和密码。  

## <a name="next-steps"></a>后续步骤

[如何管理使用 PowerShell 的 Azure 区块链服务中的成员](manage-consortium-powershell.md)
