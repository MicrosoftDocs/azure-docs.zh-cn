---
title: Azure 区块链服务安全性
description: Azure 区块链服务数据访问权限和安全性概念
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: dd0a33364ed9395a85478798e47352c533bd47dc
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028195"
---
# <a name="azure-blockchain-service-security"></a>Azure 区块链服务安全性

Azure 区块链服务使用多个 Azure 功能保护数据的安全且可用。 使用隔离、 加密和身份验证保护数据。

## <a name="isolation"></a>隔离

Azure 区块链服务资源上被隔离的专用虚拟网络中。 每个事务和验证节点是虚拟机 (VM)。 一个虚拟网络中的 Vm 不能与不同虚拟网络中的 Vm 直接通信。 隔离可确保通信的虚拟网络中保持私密性。 有关 Azure 虚拟网络隔离的详细信息，请参阅[在 Azure 公有云中的隔离](../../security/azure-isolation.md#networking-isolation)。

![VNET 关系图](./media/data-security/vnet.png)

## <a name="encryption"></a>加密

用户数据存储在 Azure 存储。 用户数据进行加密，动态和静态的安全性和机密性。 有关详细信息，请参阅：[Azure 存储安全指南](../../storage/common/storage-security-guide.md)。

## <a name="authentication"></a>Authentication

事务可以发送到通过 RPC 终结点的区块链节点。 客户端处理用户身份验证与事务节点使用反向代理服务器进行通信，并通过 SSL 加密数据。

![身份验证关系图](./media/data-security/authentication.png)

有三种 RPC 访问的身份验证模式。

### <a name="basic-authentication"></a>基本身份验证

基本身份验证使用 HTTP 身份验证标头包含用户名和密码。 用户名称是区块链节点的名称。 在成员或节点的预配期间设置密码。 可以使用 Azure 门户或 CLI 更改密码。

### <a name="access-keys"></a>访问密钥

访问密钥使用随机生成的字符串包含在终结点 URL。 两个访问密钥帮助启用密钥轮换。 从 Azure 门户和 CLI，可以重新生成密钥。

### <a name="azure-active-directory"></a>Azure Active Directory

Azure Active Directory (Azure AD) 使用基于声明的身份验证机制使用 Azure AD 用户凭据通过 Azure AD 身份验证用户的位置。 Azure AD 提供基于云的标识管理，并使客户能够在整个企业和访问应用程序在云上使用单个标识。 区块链的 azure 服务集成与 Azure AD 启用 ID 联合身份验证、 单一登录和多因素身份验证。 区块链成员和节点访问你组织中分配的用户、 组和应用程序角色。

Azure AD 客户端代理位于[GitHub](https://github.com/Microsoft/azure-blockchain-connector/releases)。 客户端代理将用户定向到 Azure AD 登录页，并获取身份验证成功后的持有者令牌。 随后，用户连接到客户端代理的终结点 Geth 或 Truffle 之类的以太坊客户端应用程序。 最后，当提交事务时，客户端代理将注入 http 标头中的持有者令牌，反向代理验证令牌使用 OAuth 协议。

## <a name="keys-and-ethereum-accounts"></a>密钥和以太坊帐户

如果预配 Azure 区块链服务成员，生成的以太坊帐户和一个公钥和私钥的密钥对。 专用密钥用于将事务发送到区块链。 以太坊帐户是公钥的哈希的最后 20 个字节。 以太坊帐户也称为 wallet。

专用和公共密钥对存储为 JSON 格式密钥文件。 使用创建区块链分类帐服务时输入的密码进行加密的私钥。

私钥用于对事务进行数字签名。 在专用区块链中，智能协定由私钥签名表示签名者的标识。 若要验证签名的有效性，接收方可以与进行比较签名者的公钥计算签名的地址。

星座密钥用于唯一标识一个仲裁节点。 星座密钥在节点预配时生成和仲裁中的专用事务的 privateFor 参数中指定。

## <a name="next-steps"></a>后续步骤

[配置 Azure 区块链服务事务节点](configure-transaction-nodes.md)
