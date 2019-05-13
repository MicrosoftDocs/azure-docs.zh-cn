---
title: 使用 Azure 区块链服务发送事务
description: 有关如何使用 Azure 区块链服务部署智能合同和发送私人事务的教程。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: tutorial
ms.service: azure-blockchain
ms.reviewer: jackyhsu
manager: femila
ms.openlocfilehash: d3ad4cdfe33948c04c278ed3dfef7aa6fda637ab
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027493"
---
# <a name="tutorial-send-transactions-using-azure-blockchain-service"></a>教程：使用 Azure 区块链服务发送事务

在本教程中，你将创建事务节点来测试合同和事务的隐私性。  你将使用 Truffle 创建本地开发环境，部署智能合同并发送私人事务。

将了解如何执行以下操作：

> [!div class="checklist"]
> * 添加事务节点
> * 使用 Truffle 部署智能合同
> * 发送事务
> * 验证事务隐私性

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

* 完成[使用 Azure 门户创建区块链成员](create-member.md)
* 完整[快速入门：使用 Truffle 连接到联盟网络](connect-truffle.md)
* Truffle 要求安装多个工具，包括 [Node.js](https://nodejs.org)、[Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) 和 [Truffle](https://github.com/trufflesuite/truffle)。

    若要快速在 Windows 10 上进行安装，请为 Unix Bash shell 终端安装 [Ubuntu on Windows](https://www.microsoft.com/p/ubuntu/9nblggh4msv6)，然后安装 [Truffle](https://github.com/trufflesuite/truffle)。 Ubuntu on Windows 分发版包括 Node.js 和 Git。

* 安装 [Visual Studio Code](https://code.visualstudio.com/Download)
* 安装 [Visual Studio Code Solidity 扩展](https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity)

## <a name="create-transaction-nodes"></a>创建事务节点

默认情况下，系统已提供一个事务节点。 我们将额外添加两个节点。 其中一个节点参与私人事务。 另一个节点不包括在私人事务中。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 导航到你的 Azure 区块链成员，并选择“事务节点”>“添加”。
1. 完成名为 `alpha` 的新事务节点的设置。

    ![创建事务节点](./media/send-transaction/create-node.png)

    | 设置 | 值 | 说明 |
    |---------|-------|-------------|
    | 名称 | `alpha` | 事务节点名称。 该名称用于创建事务节点终结点的 DNS 地址。 例如，`alpha-mymanagedledger.blockchain.azure.com`。 |
    | 密码 | 强密码 | 该密码用于通过基本身份验证访问事务节点终结点。

1. 选择“创建”。

    预配新的事务节点大约需要 10 分钟时间。

1. 重复步骤 2 至 4，添加名为 `beta` 的事务节点。

可以一边预配节点，一边学习本教程。 预配完成后，将有三个事务节点。

## <a name="open-truffle-project"></a>打开 Truffle 项目

1. 打开 Bash shell 终端。
1. 将路径更改为以下文档的“先决条件”中所述的 Truffle 项目目录：[快速入门：使用 Truffle 连接到联盟网络](connect-truffle.md)。 例如，

    ```bash
    cd truffledemo
    ```

1. 启动 Truffle 的交互式开发控制台。

    ``` bash
    truffle develop
    ```

    Truffle 会创建一个本地开发区块链，并提供一个交互式控制台。

## <a name="connect-to-transaction-node"></a>连接到事务节点

使用 Web3 连接到默认事务节点并创建一个帐户。 可以从 Azure 门户获取 Web3 连接字符串。

1. 在 Azure 门户中，导航到默认事务节点并选择“事务节点”>“示例代码”>“Web3”。
1. 复制“HTTPS(访问密钥 1)”中的 JavaScript ![Web3 示例代码](./media/send-transaction/web3-code.png)

1. 将默认事务节点的 Web3 JavaScript 代码粘贴到 Truffle 交互式开发控制台中。 该代码将创建连接到你的 Azure 区块链服务事务节点的 Web3 对象。

    ```bash
    truffle(develop)> var Web3 = require("Web3");
    truffle(develop)> var provider = new Web3.providers.HttpProvider("https://myblockchainmember.blockchain.azure.com:3200/hy5FMu5TaPR0Zg8GxiPwned");
    truffle(develop)> var web3 = new Web3(provider);
    ```

    可以调用该 Web3 对象中的方法来与事务节点进行交互。

1. 在默认事务节点上创建新帐户。 将 password 参数替换为你自己的强密码。

    ```bash
    web3.eth.personal.newAccount("1@myStrongPassword");
    ```

    请记下返回的帐户地址以及要在下一部分使用的密码。

1. 退出 Truffle 开发环境。

    ```bash
    .exit
    ```

## <a name="configure-truffle-project"></a>配置 Truffle 项目

若要配置 Truffle 项目，需要从 Azure 门户获取一些事务节点信息。

### <a name="transaction-node-public-key"></a>事务节点公钥

每个事务节点都有一个公钥。 使用公钥可将私人事务发送到节点。 若要将默认事务节点中的事务发送到 *alpha* 事务节点，需要获取 *alpha* 事务节点的公钥。

从事务节点列表中可以获取该公钥。 复制 alpha 节点的公钥并保存其值，以便稍后在本教程中使用。

![事务节点列表](./media/send-transaction/node-list.png)

### <a name="transaction-node-endpoint-addresses"></a>事务节点终结点地址

1. 在 Azure 门户中，导航到每个事务节点并选择“事务节点”>“连接字符串”。
1. 复制并保存每个事务节点的“HTTPS (访问密钥 1)”中的终结点 URL。 稍后在本教程中，需要在智能合同配置文件中提供终结点地址。

    ![事务终结点地址](./media/send-transaction/endpoint.png)

### <a name="edit-configuration-file"></a>编辑配置文件

1. 启动 Visual Studio Code，使用“文件”>“打开文件夹”菜单打开 Truffle 项目目录文件夹。
1. 打开 Truffle 配置文件 `truffle-config.js`。
1. 将该文件的内容替换为以下配置信息。 添加包含终结点地址和帐户信息的变量。 将带尖括号的节替换为在前面部分中收集的值。

``` javascript
var defaultnode = "<default transaction node connection string>";
var alpha = "<alpha transaction node connection string>";
var beta = "<beta transaction node connection string>";

var myAccount = "<account address>";
var myPassword = "<account password>";

var Web3 = require("web3");
```

将配置代码添加到配置中的 **module.exports** 节。

```javascript
module.exports = {
  networks: {
    defaultnode: {
      provider:(() =>  {
      const AzureBlockchainProvider = new Web3.providers.HttpProvider(defaultnode);

      const web3 = new Web3(AzureBlockchainProvider);
      web3.eth.personal.unlockAccount(myAccount, myPassword);

      return AzureBlockchainProvider;
      })(),

      network_id: "*",
      gas: 0,
      gasPrice: 0,
      from: myAccount
    },
    alpha: {
      provider: new Web3.providers.HttpProvider(alpha),
      network_id: "*",
      gas: 0,
      gasPrice: 0
    },
    beta: {
      provider: new Web3.providers.HttpProvider(beta),
      network_id: "*",
      gas: 0,
      gasPrice: 0
    }
  }
}
```

## <a name="create-smart-contract"></a>创建智能合同

在 **contracts** 文件夹中，创建名为 `SimpleStorage.sol` 的新文件。 添加以下代码。

```solidity
pragma solidity >=0.4.21 <0.6.0;

contract SimpleStorage {
    string public storedData;

    constructor(string memory initVal) public {
        storedData = initVal;
    }

    function set(string memory x) public {
        storedData = x;
    }

    function get() view public returns (string memory retVal) {
        return storedData;
    }
}
```

在 **migrations** 文件夹中，创建名为 `2_deploy_simplestorage.js` 的新文件。 添加以下代码。

```solidity
var SimpleStorage = artifacts.require("SimpleStorage.sol");

module.exports = function(deployer) {

  // Pass 42 to the contract as the first constructor parameter
  deployer.deploy(SimpleStorage, "42", {privateFor: ["<alpha node public key>"], from:"<Account address>"})  
};
```

替换尖括号中的值。

| 值 | 说明
|-------|-------------
| \<alpha node public key\> | alpha 节点的公钥
| \<Account address\> | 在默认事务节点中创建的帐户地址。

在此示例中，**storeData** 的初始值设置为 42。

**privateFor** 定义合同适用的节点。 在此示例中，默认事务节点的帐户可将私人事务投射到 **alpha** 节点。 需要添加所有私人事务参与者的公钥。 如果不包含 **privateFor:** 和 **from:**，则智能合同事务将是公开的，所有联盟成员都可以看到它们。

选择“文件”>“全部保存”以保存所有文件。

## <a name="deploy-smart-contract"></a>部署智能合同

使用 Truffle 将 `SimpleStorage.sol` 部署到默认事务节点网络。

```bash
truffle migrate --network defaultnode
```

Truffle 会依次编译和部署 **SimpleStorage** 智能合同。

示例输出：

```
pat@DESKTOP:/mnt/c/truffledemo$ truffle migrate --network defaultnode

2_deploy_simplestorage.js
=========================

   Deploying 'SimpleStorage'
   -------------------------
   > transaction hash:    0x3f695ff225e7d11a0239ffcaaab0d5f72adb545912693a77fbfc11c0dbe7ba72
   > Blocks: 2            Seconds: 12
   > contract address:    0x0b15c15C739c1F3C1e041ef70E0011e641C9D763
   > account:             0x1a0B9683B449A8FcAd294A01E881c90c734735C3
   > balance:             0
   > gas used:            0
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0 ETH
```

## <a name="validate-contract-privacy"></a>验证合同隐私性

由于合同隐私性，只能从 **privateFor** 中声明的节点查询合同值。 在此示例中，我们可以查询默认事务节点，因为该节点中存在帐户。 使用 Truffle 控制台连接到默认事务节点。

```bash
truffle console --network defaultnode
```

执行返回合同实例值的命令。

```bash
SimpleStorage.deployed().then(function(instance){return instance.get();})
```

如果查询默认事务节点成功，则会返回值 42。

示例输出：

```
pat@DESKTOP-J41EP5S:/mnt/c/truffledemo$ truffle console --network defaultnode
truffle(defaultnode)> SimpleStorage.deployed().then(function(instance){return instance.get();})
'42'
```

退出控制台。

```bash
.exit
```

由于我们在 **privateFor** 中声明了 **alpha** 节点的公钥，因此可以查询 **alpha** 节点。 使用 Truffle 控制台连接到 **alpha** 节点。

```bash
truffle console --network alpha
```

执行返回合同实例值的命令。

```bash
SimpleStorage.deployed().then(function(instance){return instance.get();})
```

如果查询 **alpha** 节点成功，则会返回值 42。

示例输出：

```
pat@DESKTOP-J41EP5S:/mnt/c/truffledemo$ truffle console --network alpha
truffle(alpha)> SimpleStorage.deployed().then(function(instance){return instance.get();})
'42'
```

退出控制台。

```bash
.exit
```

由于我们未在 **privateFor** 中声明 **beta** 节点的公钥，出于合同隐私性，我们无法查询 **beta** 节点。 使用 Truffle 控制台连接到 **beta** 节点。

```bash
truffle console --network beta
```

执行返回合同实例值的命令。

```bash
SimpleStorage.deployed().then(function(instance){return instance.get();})
```

由于合同是隐私的，因此查询 **beta** 节点失败。

示例输出：

```
pat@DESKTOP-J41EP5S:/mnt/c/truffledemo$ truffle console --network beta
truffle(beta)> SimpleStorage.deployed().then(function(instance){return instance.get();})
Thrown:
Error: Returned values aren't valid, did it run Out of Gas?
    at XMLHttpRequest._onHttpResponseEnd (/mnt/c/truffledemo/node_modules/xhr2-cookies/xml-http-request.ts:345:8)
    at XMLHttpRequest._setReadyState (/mnt/c/truffledemo/node_modules/xhr2-cookies/xml-http-request.ts:219:8)
    at XMLHttpRequestEventTarget.dispatchEvent (/mnt/c/truffledemo/node_modules/xhr2-cookies/xml-http-request-event-target.ts:44:13)
    at XMLHttpRequest.request.onreadystatechange (/mnt/c/truffledemo/node_modules/web3-providers-http/src/index.js:96:13)
```

退出控制台。

```bash
.exit
```

## <a name="send-a-transaction"></a>发送事务

创建名为 `sampletx.js` 的文件。 将此文件保存到项目的根目录中。

此脚本将合同的 **storedData** 变量值设置为 65。 将代码添加到新文件。

```javascript
var SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(done) {
  console.log("Getting deployed version of SimpleStorage...")
  SimpleStorage.deployed().then(function(instance) {
    console.log("Setting value to 65...");
    return instance.set("65", {privateFor: ["<alpha node public key>"], from:"<Account address>"});
  }).then(function(result) {
    console.log("Transaction:", result.tx);
    console.log("Finished!");
    done();
  }).catch(function(e) {
    console.log(e);
    done();
  });
};
```

替换尖括号中的值，然后保存文件。

| 值 | 说明
|-------|-------------
| \<alpha node public key\> | alpha 节点的公钥
| \<Account address\> | 在默认事务节点中创建的帐户地址。

**privateFor** 定义事务适用的节点。 在此示例中，默认事务节点的帐户可将私人事务投射到 **alpha** 节点。 需要添加所有私人事务参与者的公钥。

使用 Truffle 针对默认事务节点执行脚本。

```bash
truffle exec sampletx.js --network defaultnode
```

执行返回合同实例值的命令。

```bash
SimpleStorage.deployed().then(function(instance){return instance.get();})
```

如果事务成功，则会返回值 65。

示例输出：

```
Getting deployed version of SimpleStorage...
Setting value to 65...
Transaction: 0x864e67744c2502ce75ef6e5e09d1bfeb5cdfb7b880428fceca84bc8fd44e6ce0
Finished!
```

退出控制台。

```bash
.exit
```

## <a name="validate-transaction-privacy"></a>验证事务隐私性

由于事务隐私性，只能在 **privateFor** 中声明的节点上执行事务。 在此示例中，我们可以执行事务，因为我们在 **privateFor** 中声明了 **alpha** 节点的公钥。 使用 Truffle 在 **alpha** 节点上执行事务。

```bash
truffle exec sampletx.js --network alpha
```

执行返回合同实例值的命令。

```bash
SimpleStorage.deployed().then(function(instance){return instance.get();})
```

如果事务成功，则会返回值 65。

示例输出：

```
Getting deployed version of SimpleStorage...
Setting value to 65...
Transaction: 0x864e67744c2502ce75ef6e5e09d1bfeb5cdfb7b880428fceca84bc8fd44e6ce0
Finished!
```

退出控制台。

```bash
.exit
```

在本教程中，你已添加两个事务节点来演示合同和事务的隐私性。 你已使用默认节点来部署私人智能合同。 你已通过查询合同值并在区块链中执行事务来测试隐私性。

## <a name="clean-up-resources"></a>清理资源

不再需要本教程中创建的资源时，可以通过删除 Azure 区块链服务创建的 `myResourceGroup` 资源组来删除这些资源。

若要删除资源组，请执行以下操作：

1. 在 Azure 门户中，导航至左侧导航窗格中的“资源组”，然后选择要删除的资源组。
1. 选择“删除资源组”。 输入资源组名称确认删除并选择“删除”。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 Azure 区块链服务开发区块链应用程序](develop.md)
