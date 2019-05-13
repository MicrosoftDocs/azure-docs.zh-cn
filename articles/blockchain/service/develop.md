---
title: Azure 区块链服务开发概述
description: 开发 Azure 区块链服务上的解决方案的简介。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: jackyhsu
manager: femila
ms.openlocfilehash: 388a5d8c80c3e2462602959e9d5cbc1452974d1f
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027895"
---
# <a name="azure-blockchain-service-development-overview"></a>Azure 区块链服务开发概述

借助 Azure 区块链服务，可以创建联盟区块链网络，以启用资产跟踪、 数字令牌、 忠诚度和奖励、 金融、 供应链和 provenance 等企业方案。 本文是介绍 Azure 区块链服务开发概述和关键主题来实现企业的区块链。

## <a name="client-connection-to-azure-blockchain-service"></a>客户端连接到 Azure 区块链服务

有不同类型的区块链网络包括完整的节点、 浅节点和远程客户端的客户端。 Azure 区块链服务生成包含节点的区块链网络。 区块链开发的可以为你网关与 Azure 区块链服务使用不同的客户端。 Azure 区块链服务提供基本身份验证或访问密钥与开发终结点。 以下是可以使用的常用客户端连接。

### <a name="metamask"></a>MetaMask

MetaMask 是基于浏览器的钱包 （远程客户端）、 RPC 客户端和基本协定资源管理器。 与其他浏览器证明，不同 MetaMask 注入 web3 实例到浏览器 JavaScript 上下文中，充当连接到各种以太坊区块链的 RPC 客户端 (*mainnet*， *Ropsten testnet*， *Kovan testnet*，本地 RPC 节点，等等。)。 您可以设置自定义 RPC 轻松地以连接到 Azure 区块链服务并开始使用 Remix 区块链开发。

### <a name="geth"></a>Geth

Geth 是运行在 Go 中实现完整以太坊节点的命令行接口。 无需运行完整的节点，但可以启动其交互式控制台，可提供公开了 JavaScript API 进行交互与 Azure 区块链服务的 JavaScript 运行时环境。

## <a name="development-framework-configuration"></a>开发框架配置

若要开发复杂的企业区块链解决方案，开发框架需要连接到不同的区块链网络、 管理智能协定生命周期、 自动执行测试、 部署脚本，具有智能协定和做一个交互式控制台。

Truffle 是常用区块链开发框架编写、 编译、 部署和测试以太坊区块链上分散的应用程序。 您还可以作为尝试无缝集成智能协定开发和传统 web 开发的框架将 Truffle。

即使最小的项目与至少两个区块链节点进行交互：一组在开发人员计算机，另一个表示开发人员在其中部署其应用程序的网络。 例如，主公用以太坊网络或 Azure 区块链服务。 Truffle 用于管理每个网络的编译和部署项目提供系统，并且会简化了最终的应用程序部署的方式。 有关详细信息，请参阅[快速入门：Truffle 用于连接到 Azure 区块链服务网络](connect-truffle.md)。

## <a name="ethereum-quorum-private-transaction"></a>以太坊仲裁专用事务

仲裁是一种基于以太坊的分布式的分类帐协议事务加上合同隐私和新的一致机制。 通过转到以太坊重要的增强功能包括：

* 隐私-仲裁支持专用事务和通过公共和私有状态分离的专用协定，并利用定向专用将数据传输到网络参与者的对等加密的消息交换。
* 替代的一致机制-而无需请求权限的网络中的工作量证明或证明当中共识。 仲裁提供多个一致的机制专为如筏和 IBFT 联盟链。  Azure 区块链服务使用 IBFT 达成一致意见机制。

* 对等互连授予其权限的使用智能合同，确保仅已知参与方的节点和对等方授予其权限可以加入的网络
* 更高的性能-仲裁提供公共 Geth 比更高的性能

有关分步说明，请参阅[教程：使用 Azure 区块链服务的事务发送](send-transaction.md)有关专用事务的示例。

## <a name="block-explorers"></a>块资源管理器

块资源管理器是联机的区块链的浏览器中显示单个块内容、 事务地址数据和历史记录。 通过 Azure 区块链服务中的 Azure Monitor 提供了基本块信息，但是，如果需要在开发过程的更多详细信息，块资源管理器可能会很有用。  有很受欢迎的开源块可以使用的资源管理器。 下面是与 Azure 区块链服务一起使用的块资源管理器的列表：

* [区块链的 azure 服务资源管理器](https://web3labs.com/azure-offer)Web3 labs
* [BlockScout](https://github.com/Azure-Samples/blockchain/blob/master/ledger/template/ethereum-on-azure/technology-samples/blockscout/README.md)

## <a name="tps-measurement"></a>TP 度量

在多个企业方案中使用区块链时，第二个 (TPS) 速度每个事务务必避免瓶颈和系统效率低下。 事务速率较高可能很难维护分散式区块链中。 精确的 TP 测量值可能受不同因素，例如服务器线程、 事务队列大小、 网络延迟和安全性。 如果你需要在开发期间测量 TP 速度，是一个受欢迎的开源工具[ChainHammer](https://github.com/drandreaskrueger/chainhammer)。

## <a name="next-steps"></a>后续步骤

[快速入门：Truffle 用于连接到 Azure 区块链服务网络](connect-truffle.md)
