---
title: 管理 Azure 区块链服务使用 Azure CLI
description: 如何创建和管理 Azure 区块链服务中使用 Azure CLI
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: d078ca181b2eed4b80d4f12f1c03b42f4e242194
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154442"
---
# <a name="manage-azure-blockchain-service-with-azure-cli"></a>管理 Azure 区块链服务中使用 Azure CLI

除了 Azure 门户中，可以使用 Azure CLI 快速创建并管理你的 Azure 区块链服务的区块链成员和事务节点。

请确保已安装最新[Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)并已登录到 Azure 帐户中的`az login`。

在以下示例中，将为示例`<parameter names>`与你自己的值。

## <a name="create-blockchain-member"></a>创建区块链成员

示例在新联盟中运行的仲裁分类帐协议的 Azure 区块链服务中创建区块链的成员。

```azurecli
az resource create --resource-group <myResourceGroup> --name <myMemberName> --resource-type Microsoft.Blockchain/blockchainMembers --is-full-object --properties "{ \"location\": \"<myBlockchainLocation>\", \"properties\": {\"password\": \"<myStrongPassword>\", \"protocol\": \"Quorum\", \"consortium\": \"<myConsortiumName>\", \"consortiumManagementAccountPassword\": \"<myConsortiumManagementAccountPassword>\", \"firewallRules\": [ { \"ruleName\": \"<myRuleName>\", \"startIpAddress\": \"<myStartIpAddress>\", \"endIpAddress\": \"<myEndIpAddress>\" } ] }, \"sku\": { \"name\": \"<skuName>\" } }"
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | 在其中创建 Azure 区块链服务资源的资源组名称。 |
| **name** | 用于标识 Azure 区块链服务区块链成员的唯一名称。 此名称用于公共终结点地址。 例如，`myblockchainmember.blockchain.azure.com`。 |
| **位置** | 在其中创建区块链成员的 Azure 区域。 例如，`eastus`。 选择最靠近用户或其他 Azure 应用程序的位置。 |
| **password** | 成员帐户密码。 成员帐户密码用来使用基本身份验证向区块链成员的公共终结点进行身份验证。 密码必须满足三个以下四个要求： 长度必须介于 12 和 72 个字符、 1 个小写字符、 1 个大写字符、 1 个数字和 1 个特殊字符不数字 sign(#)、 percent(%)、 逗号 （，）、 star(*)，反引号(\`)，双击 quote(")、 单引号、 dash(-) 和 semicolumn(;)|
| **protocol** | 公共预览版支持仲裁。 |
| **consortium** | 要加入或创建的联盟的名称。 |
| **consortiumManagementAccountPassword** | 联盟管理密码。 密码用于加入联盟。 |
| **ruleName** | 列入允许列表 IP 地址范围的规则名称。 防火墙规则的可选参数。|
| **startIpAddress** | 列入允许列表 IP 地址范围的开始。 防火墙规则的可选参数。 |
| **endIpAddress** | 列入允许列表 IP 地址范围的下限。 防火墙规则的可选参数。 |
| **skuName** | 层类型。 对于标准层，请使用 S0，对于基本层，请使用 B0。 |

## <a name="change-blockchain-member-password"></a>更改区块链成员密码

示例更改了区块链成员的密码。

```azurecli
az resource update --resource-group <myResourceGroup> --name <myMemberName> --resource-type Microsoft.Blockchain/blockchainMembers --set properties.password="<myStrongPassword>" --remove properties.consortiumManagementAccountAddress
```
| 参数 | 描述 |
|---------|-------------|
| **resource-group** | 在其中创建 Azure 区块链服务资源的资源组名称。 |
| **name** | 标识 Azure 区块链服务成员的名称。 |
| **password** | 成员帐户密码。 密码必须满足三个以下四个要求： 长度必须介于 12 和 72 个字符、 1 个小写字符、 1 个大写字符、 1 个数字和 1 个特殊字符不数字 sign(#)、 percent(%)、 逗号 （，）、 star(*)，反引号(\`)，双击 quote(")、 单引号、 dash(-) 和分号 （;）。 |


## <a name="create-transaction-node"></a>创建事务节点

创建现有的区块链成员内部事务节点。 通过添加事务节点，可以提高安全隔离，并将负载分配。 例如，你可以为不同的客户端应用程序的事务节点终结点。

```azurecli
az resource create --resource-group <myResourceGroup> --name <myMemberName>/transactionNodes/<myTransactionNode> --resource-type Microsoft.Blockchain/blockchainMembers  --is-full-object --properties "{ \"location\": \"<myRegion>\", \"properties\": { \"password\": \"<myStrongPassword>\", \"firewallRules\": [ { \"ruleName\": \"<myRuleName>\", \"startIpAddress\": \"<myStartIpAddress>\", \"endIpAddress\": \"<myEndIpAddress>\" } ] } }"
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | 在其中创建 Azure 区块链服务资源的资源组名称。 |
| **name** | 此外包括新的事务节点名称的 Azure 区块链服务区块链成员的名称。 |
| **位置** | 在其中创建区块链成员的 Azure 区域。 例如，`eastus`。 选择最靠近用户或其他 Azure 应用程序的位置。 |
| **password** | 事务节点密码。 密码必须满足三个以下四个要求： 长度必须介于 12 和 72 个字符、 1 个小写字符、 1 个大写字符、 1 个数字和 1 个特殊字符不数字 sign(#)、 percent(%)、 逗号 （，）、 star(*)，反引号(\`)，双击 quote(")、 单引号、 dash(-) 和分号 （;）。 |
| **ruleName** | 列入允许列表 IP 地址范围的规则名称。 防火墙规则的可选参数。 |
| **startIpAddress** | 列入允许列表 IP 地址范围的开始。 防火墙规则的可选参数。 |
| **endIpAddress** | 列入允许列表 IP 地址范围的下限。 防火墙规则的可选参数。|

## <a name="change-transaction-node-password"></a>更改事务节点密码

示例更改事务节点密码。

```azurecli
az resource update --resource-group <myResourceGroup> --name <myMemberName>/transactionNodes/<myTransactionNode> --resource-type Microsoft.Blockchain/blockchainMembers  --set properties.password="<myStrongPassword>"
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | 此外包括新的事务节点名称的 Azure 区块链服务区块链成员的名称。 |
| **password** | 事务节点密码。 密码必须满足三个以下四个要求： 长度必须介于 12 和 72 个字符、 1 个小写字符、 1 个大写字符、 1 个数字和 1 个特殊字符不数字 sign(#)、 percent(%)、 逗号 （，）、 star(*)，反引号(\`)，双击 quote(")、 单引号、 dash(-) 和分号 （;）。 |

## <a name="change-consortium-management-account-password"></a>更改联盟管理帐户密码

联盟管理帐户用于联盟的成员身份管理。 由联盟管理帐户唯一标识每个成员，可以更改此帐户使用以下命令的密码。

```azurecli
az resource update --resource-group <myResourceGroup> --name <myMemberName> --resource-type Microsoft.Blockchain/blockchainMembers --set properties.consortiumManagementAccountPassword="<myConsortiumManagementAccountPassword>" --remove properties.consortiumManagementAccountAddress
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | 在其中创建 Azure 区块链服务资源的资源组名称。 |
| **name** | 标识 Azure 区块链服务成员的名称。 |
| **consortiumManagementAccountPassword** | 联盟管理帐户密码。 密码必须满足三个以下四个要求： 长度必须介于 12 和 72 个字符、 1 个小写字符、 1 个大写字符、 1 个数字和 1 个特殊字符不数字 sign(#)、 percent(%)、 逗号 （，）、 star(*)，反引号(\`)，双击 quote(")、 单引号、 dash(-) 和分号 （;）。 |
  
## <a name="update-firewall-rules"></a>更新防火墙规则

```azurecli
az resource update --resource-group <myResourceGroup> --name <myMemberName> --resource-type Microsoft.Blockchain/blockchainMembers --set properties.firewallRules="[ { \"ruleName\": \"<myRuleName>\", \"startIpAddress\": \"<myStartIpAddress>\", \"endIpAddress\": \"<myEndIpAddress>\" } ]" --remove properties.consortiumManagementAccountAddress
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | Azure 区块链服务区块链成员的名称。 |
| **ruleName** | 列入允许列表 IP 地址范围的规则名称。 防火墙规则的可选参数。|
| **startIpAddress** | 列入允许列表 IP 地址范围的开始。 防火墙规则的可选参数。|
| **endIpAddress** | 列入允许列表 IP 地址范围的下限。 防火墙规则的可选参数。|

## <a name="list-api-keys"></a>列出 API 密钥

API 密钥可以用于节点访问类似于用户名和密码。 有两个 API 密钥，以便支持密钥轮换。 使用以下命令列出你的 API 密钥。

```azurecli
az resource invoke-action --resource-group <myResourceGroup> --name <myMemberName>/transactionNodes/<myTransactionNode> --action "listApiKeys" --resource-type Microsoft.Blockchain/blockchainMembers
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | 此外包括新的事务节点名称的 Azure 区块链服务区块链成员的名称。 |

## <a name="regenerate-api-keys"></a>重新生成 API 密钥

使用以下命令来重新生成 API 密钥。

```azurecli
az resource invoke-action --resource-group <myResourceGroup> --name <myMemberName>/transactionNodes/<myTransactionNode> --action "regenerateApiKeys" --resource-type Microsoft.Blockchain/blockchainMembers --request-body '{"keyName":"<keyValue>"}'
```


| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | 此外包括新的事务节点名称的 Azure 区块链服务区块链成员的名称。 |
| **keyName** | 替换\<keyValue\> key1 或 key2。 |

## <a name="delete-a-transaction-node"></a>删除事务节点

示例将删除区块链成员事务节点。

```azurecli
az resource delete --resource-group <myResourceGroup> --name <myMemberName>/transactionNodes/<myTransactionNode> --resource-type Microsoft.Blockchain/blockchainMembers
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | 此外包括新的事务节点名称，要删除的 Azure 区块链服务区块链成员的名称。 |

## <a name="delete-a-blockchain-member"></a>删除区块链成员

示例将删除区块链成员。

```azurecli
az resource delete --resource-group <myResourceGroup> --name <myMemberName> --resource-type Microsoft.Blockchain/blockchainMembers
```

| 参数 | 描述 |
|---------|-------------|
| **resource-group** | Azure 区块链服务资源所在资源组名称。 |
| **name** | 要删除的 Azure 区块链服务区块链成员的名称。 |

## <a name="azure-active-directory"></a>Azure Active Directory

### <a name="grant-access-for-azure-ad-user"></a>为 Azure AD 用户授予访问权限

```azurecli
az role assignment create --role <role> --assignee <assignee> --scope /subscriptions/<subId>/resourceGroups/<groupName>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>
```

| 参数 | 描述 |
|---------|-------------|
| **role** | Azure AD 角色的名称。 |
| **assignee** | Azure AD 用户 id。 例如： `user@contoso.com` |
| **scope** | 角色分配的范围。 可以是区块链的成员事务节点。 |

**示例：**

授予节点到区块链的 Azure AD 用户的访问权限**成员**:

```azurecli
az role assignment create \
  --role "myRole" \
  --assignee user@contoso.com \
  --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

**示例：**

授予节点到区块链的 Azure AD 用户的访问权限**事务节点**:

```azurecli
az role assignment create \
  --role "MyRole" \
  --assignee user@contoso.com \
  --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1/transactionNodes/contosoTransactionNode1
```

### <a name="grant-node-access-for-azure-ad-group-or-application-role"></a>授予 Azure AD 组或应用程序的节点访问权限角色

```azurecli
az role assignment create --role <role> --assignee-object-id <assignee_object_id>
```
| 参数 | 描述 |
|---------|-------------|
| **role** | Azure AD 角色的名称。 |
| **assignee-object-id** | Azure AD 组 ID 或应用程序 id。 |
| **scope** | 角色分配的范围。 可以是区块链的成员事务节点。 |

**示例：**

节点为授予访问权限**应用程序角色**

```azurecli
az role assignment create \
  --role "myRole" \
  --assignee-object-id 22222222-2222-2222-2222-222222222222 \
  --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

### <a name="remove-azure-ad-node-access"></a>删除 Azure AD 节点访问权限

```azurecli
az role assignment delete --role <myRole> --assignee <assignee> --scope /subscriptions/mySubscriptionId/resourceGroups/<myResourceGroup>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>/transactionNodes/<myTransactionNode>
```

| 参数 | 描述 |
|---------|-------------|
| **role** | Azure AD 角色的名称。 |
| **assignee** | Azure AD 用户 id。 例如： `user@contoso.com` |
| **scope** | 角色分配的范围。 可以是区块链的成员事务节点。 |

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 Azure 门户配置 Azure 区块链服务事务节点](configure-transaction-nodes.md)
