---
title: 使用 Azure CLI 创建 Azure 区块链服务
description: 通过 Azure CLI 使用 Azure 区块链服务创建区块链成员。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: quickstart
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: e1b7558ea83c8948a8984215e15040e4d929cb1b
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141384"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-cli"></a>快速入门：通过 Azure CLI 创建 Azure 区块链服务区块链成员

Azure 区块链服务是一个区块链平台，可以用来在智能合同中执行你的业务逻辑。 本快速入门展示了如何开始使用 Azure CLI 创建区块链成员。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。

若要打开 Cloud Shell，只需要从代码块的右上角选择“试一试”。 也可以通过转到 [https://shell.azure.com/bash](https://shell.azure.com/bash) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

如果希望在本地安装并使用 CLI，则本快速入门需要 Azure CLI 2.0.51 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.microsoft.com/cli/azure/group) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 以下示例在 eastus 位置创建名为 myResourceGroup 的资源组：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-blockchain-member"></a>创建区块链成员

在 Azure 区块链服务中创建一个区块链成员，用以在新联盟中运行仲裁账本协议。

需要传递多个参数和属性。 将以下参数替换为你自己的值。

| 参数 | 说明 |
|---------|-------------|
| **resource-group** | 在其中创建 Azure 区块链服务资源的资源组名称。 请使用在上一部分中创建的资源组。
| name | 用于标识 Azure 区块链服务区块链成员的唯一名称。 此名称用于公共终结点地址。 例如，`myblockchainmember.blockchain.azure.com`。
| **位置** | 在其中创建区块链成员的 Azure 区域。 例如，`eastus`。 选择最靠近用户或其他 Azure 应用程序的位置。
| **password** | 成员帐户密码。 成员帐户密码用来使用基本身份验证向区块链成员的公共终结点进行身份验证。
| **consortium** | 要加入或创建的联盟的名称。
| **consortiumManagementAccountPassword** | 联盟管理密码。 这用于加入联盟。
| **skuName** | 层类型。 对于标准层，请使用 S0，对于基本层，请使用 B0。

```azurecli-interactive
az resource create --resource-group myResourceGroup --name myblockchainmember --resource-type Microsoft.Blockchain/blockchainMembers --is-full-object --properties "{ \"location\": \"eastus\", \"properties\": {\"password\": \"strongMemberAccountPassword@1\", \"protocol\": \"Quorum\", \"consortium\": \"myConsortiumName\", \"consortiumManagementAccountPassword\": \"strongConsortiumManagementPassword@1\" }, \"sku\": { \"name\": \"S0\" } }"
```

创建区块链成员和支持资源需要大约 10 分钟。

下面显示了成功的创建操作的输出示例。

```json
{
  "id": "/subscriptions/<subscriptionId>/resourceGroups/myResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/mymembername",
  "kind": null,
  "location": "eastus",
  "name": "mymembername",
  "properties": {
    "ConsortiumMemberDisplayName": "mymembername",
    "consortium": "myConsortiumName",
    "consortiumManagementAccountAddress": "0xfe5fbb9d1036298abf415282f52397ade5d5beef",
    "consortiumManagementAccountPassword": null,
    "consortiumRole": "ADMIN",
    "dns": "mymembername.blockchain.azure.com",
    "protocol": "Quorum",
    "provisioningState": "Succeeded",
    "userName": "mymembername",
    "validatorNodesSku": {
      "capacity": 2
    }
  },
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "S0",
    "tier": "Standard"
  },
  "type": "Microsoft.Blockchain/blockchainMembers"
}
```

## <a name="clean-up-resources"></a>清理资源

你可以将所创建的区块链成员用于下一快速入门或教程。 当不再需要这些资源时，可以通过删除使用 Azure 区块链服务创建的 `myResourceGroup` 资源组来删除这些资源。

运行以下命令来删除资源组和所有相关资源。

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>后续步骤

现在，你已创建了一个区块链成员，请尝试适用于 [Geth](connect-geth.md)、[MetaMask](connect-metamask.md) 或 [Truffle](connect-truffle.md) 的连接事务节点快速入门之一。

> [!div class="nextstepaction"]
> [使用 Truffle 连接到 Azure 区块链服务网络](connect-truffle.md)
