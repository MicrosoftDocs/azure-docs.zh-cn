---
title: 使用 Geth 连接到 Azure 区块链服务
description: 使用 Geth 连接到 Azure 区块链服务网络
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: quickstart
ms.service: azure-blockchain
ms.reviewer: jackyhsu
manager: femila
ms.openlocfilehash: 0716a9326a54ae31d4f355fe5f4c88488339b390
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029975"
---
# <a name="quickstart-use-geth-to-connect-to-a-transaction-node"></a>快速入门：使用 Geth 连接到事务节点

Geth 是一个 Go Ethereum 客户端，可以用来附加到 Azure 区块链服务事务节点上的 Geth 实例。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

* 安装 [Geth](https://github.com/ethereum/go-ethereum/wiki/geth)
* [创建 Azure 区块链成员](create-member.md)

## <a name="get-the-geth-connection-string"></a>获取 Geth 连接字符串

可以在 Azure 门户中找到 Geth 连接字符串。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 导航到你的 Azure 区块链服务成员。 选择“事务节点”和默认事务节点链接。

    ![选择默认事务节点](./media/connect-geth/transaction-nodes.png)

1. 选择“连接字符串”。
1. 从“HTTPS(访问密钥 1)”中复制连接字符串。 下一部分中需要使用此命令。

    ![连接字符串](./media/connect-geth/connection-string.png)

## <a name="connect-to-geth"></a>连接到 Geth

1. 打开命令提示符或 shell。
1. 使用 Geth attach 子命令附加到你的事务节点上正在运行的 Geth 实例。 将连接字符串粘贴为 attach 子命令的参数。 例如，

    ```
    geth attach <connection string>
    ```

1. 连接到事务节点的 Ethereum 控制台后，可以调用 web3 JavaScript Dapp API 或管理 API。

    例如，使用以下 API 来查找 chainId。

    ```bash
    admin.nodeInfo.protocols.istanbul.config.chainId
    ```

    在此示例中，chainId 为 297。

    ![Azure 区块链服务选项](./media/connect-geth/geth-attach.png)

1. 若要从控制台断开连接，请键入 `exit`。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你使用 Geth 客户端附加到了 Azure 区块链服务事务节点上的 Geth 实例。 请尝试下一教程来使用 Truffle 部署并发送事务。

> [!div class="nextstepaction"]
> [发送事务](send-transaction.md)