---
title: 配置 Azure 区块链服务事务节点
description: 如何配置 Azure 区块链服务事务节点
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: dffeb81ae1eb244c38639a1241c0581e6fcdf94a
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027955"
---
# <a name="configure-azure-blockchain-service-transaction-nodes"></a>配置 Azure 区块链服务事务节点

要与 Azure 区块链服务进行交互，则进行完成连接到区块链成员中的一个或多个事务节点。  若要与事务节点进行交互，需要配置用于访问节点。

## <a name="prerequisites"></a>必备组件

* [创建 Azure 区块链成员](create-member.md)

## <a name="transaction-node-overview"></a>事务节点概述

事务节点用于将区块链事务发送到 Azure 区块链服务，通过公共终结点。 默认事务节点包含注册上区块链，以太坊帐户的私钥，这种情况下无法删除。

若要查看的默认事务节点详细信息：

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 导航到 Azure 区块链服务成员。 选择**事务节点**。

    ![选择的默认事务节点](./media/configure-transaction-nodes/nodes.png)

    概述详细信息包括公共终结点地址和公共密钥。

## <a name="create-transaction-node"></a>创建事务节点

可以向您的区块链成员，总共 10 个事务节点添加多达九个其他事务节点。 通过添加事务节点，你可以增加可伸缩性或分配负载。 例如，你可以为不同的客户端应用程序的事务节点终结点。

若要添加事务节点：

1. 在 Azure 门户中，导航到 Azure 区块链服务成员并选择**事务节点 > 添加**。
1. 完成新的事务节点的设置。

    ![添加事务节点](./media/configure-transaction-nodes/add-node.png)

    | 设置 | 描述 |
    |---------|-------------|
    | 名称 | 事务节点名称。 该名称用于创建事务节点终结点的 DNS 地址。 例如，`newnode-myblockchainmember.blockchain.azure.com`。 在创建后无法更改节点名称。 |
    | 密码 | 设置强密码。 使用密码来访问事务节点终结点使用基本身份验证。

1. 选择“创建”。

    预配新的事务节点需要约 10 分钟。 其他事务节点会产生费用。 有关成本的详细信息，请参阅[Azure 定价](https://aka.ms/ABSPricing)。

## <a name="endpoints"></a>终结点

事务节点具有唯一的 DNS 名称和公共终结点。

若要查看事务节点的终结点详细信息：

1. 在 Azure 门户中，导航到某个 Azure 区块链服务成员事务节点并选择**概述**。

    ![终结点](./media/configure-transaction-nodes/endpoints.png)

事务节点终结点是安全的并要求进行身份验证。 你可以连接到一个事务终结点使用 Azure AD 身份验证，HTTPS 基本身份验证，并通过 SSL 通过 HTTPS 或 Websocket 使用的访问密钥。

### <a name="azure-active-directory-access-control"></a>Azure Active Directory 访问控制

Azure 区块链服务事务节点终结点支持 Azure Active Directory (Azure AD) 身份验证。 可以向你的终结点授予 Azure AD 用户、 组和服务主体访问权限。

若要授予 Azure AD 访问控制到终结点：

1. 在 Azure 门户中，导航到 Azure 区块链服务成员并选择**事务节点 > 的访问控制 (IAM) > 添加 > 添加角色分配**。
1. 创建新的角色分配的用户、 组或服务主体 （应用程序角色）。

    ![添加 IAM 角色](./media/configure-transaction-nodes/add-role.png)

    | 设置 | 操作 |
    |---------|-------------|
    | 角色 | 选择**所有者**，**参与者**，或**读取器**。
    | 将访问权限分配到 | 选择**Azure AD 用户、 组或服务主体**。
    | 选择 | 搜索用户、 组或你想要添加的服务主体。

1. 选择**保存**若要添加的角色分配。

Azure AD 访问控制的详细信息，请参阅[管理使用 RBAC 和 Azure 门户的 Azure 资源的访问权限](../../role-based-access-control/role-assignments-portal.md)

有关如何使用 Azure AD 身份验证进行连接的详细信息，请参阅[连接到你使用 AAD 身份验证的节点](configure-aad.md)。

### <a name="basic-authentication"></a>基本身份验证

对于 HTTPS 基本身份验证，用户名和密码凭据到终结点传递请求的 HTTPS 标头。

可以在 Azure 门户中查看事务节点的基本身份验证终结点详细信息。 导航到某个 Azure 区块链服务成员事务节点并选择**基本身份验证**设置中。

![基本身份验证](./media/configure-transaction-nodes/basic.png)

用户名称是你的节点的名称，并且不能更改。

若要使用的 URL，请替换\<密码\>时预配节点设置的密码。 可以通过选择更新的密码**重置密码**。

### <a name="access-keys"></a>访问密钥

对于访问密钥身份验证，终结点 URL 中包含的访问密钥。 设置事务节点时，会生成两个访问密钥。 这两个访问密钥可以用于身份验证。 两个密钥启用更改和轮换密钥。

您可以查看事务节点的访问密钥的详细信息并复制访问密钥中包含的终结点地址。 导航到某个 Azure 区块链服务成员事务节点并选择**访问密钥**设置中。

### <a name="firewall-rules"></a>防火墙规则

防火墙规则，您可以限制可尝试对事务节点进行身份验证的 IP 地址。  如果事务节点配置防火墙规则，它不能访问的任何一方。  

若要查看事务节点的防火墙规则，请导航到一个 Azure 区块链服务成员事务节点并选择**防火墙规则**设置中。

可以通过输入规则名称添加防火墙规则、 起始 IP 地址和结束 IP 地址中**防火墙规则**网格。

![防火墙规则](./media/configure-transaction-nodes/firewall-rules.png)

若要启用：

* **单个 IP 地址：** 配置的开始和结束 IP 地址的同一 IP 地址。
* **IP 地址范围：** 配置的开始和结束 IP 地址范围。 例如，范围从 10.221.34.0 处开始和结束时间 10.221.34.255 将启用整个 10.221.34.xxx 子网。
* **允许所有 IP 地址：** 配置起始 IP 地址为 0.0.0.0 和 255.255.255.255 的结束 IP 地址。

## <a name="connection-strings"></a>连接字符串

事务节点的连接字符串语法提供基本身份验证或使用访问密钥。 提供连接字符串包括 HTTPS 和 Websocket 的访问密钥。

您可以查看事务节点的连接字符串，并将复制终结点地址。 导航到某个 Azure 区块链服务成员事务节点并选择**连接字符串**设置中。

![连接字符串](./media/configure-transaction-nodes/connection-strings.png)

## <a name="sample-code"></a>代码示例

提供示例代码快速启用连接到通过 Web3、 Nethereum、 Web3js 和 Truffle 事务节点。

您可以查看事务节点的示例连接代码并进行复制，以与流行的开发工具一起使用。 导航到某个 Azure 区块链服务成员事务节点并选择**示例代码**设置中。

选择 Web3 或 Nethereum 选项卡以查看你想要使用的代码示例。

![代码示例](./media/configure-transaction-nodes/sample-code.png)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [配置事务节点上使用 Azure CLI](manage-cli.md)
