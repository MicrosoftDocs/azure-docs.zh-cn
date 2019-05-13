---
title: 使用 PowerShell 的 azure 区块链服务联盟管理
description: 如何管理使用 PowerShell 的 Azure 区块链服务联盟成员
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: bef0c5d776e8ab6424b8604a49782088c45b0538
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028225"
---
# <a name="manage-consortium-members-in-azure-blockchain-service-using-powershell"></a>管理使用 PowerShell 的 Azure 区块链服务中的联盟成员

可以使用 PowerShell 为 Azure 区块链服务管理区块链联盟成员。 具有管理员权限的成员可以邀请、 添加、 删除和更改的区块链联盟中的所有参与者角色。 使用用户权限的成员可以查看区块链联盟中的所有参与者，并可以更改其成员的显示名称。

## <a name="prerequisites"></a>必备组件

* [创建使用 Azure 门户的区块链成员](create-member.md)
* 有关财团、 成员和节点的详细信息，请参阅[Azure 区块链服务联盟](consortium.md)

## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。 

也可以通过转到 [https://shell.azure.com/powershell](https://shell.azure.com/powershell) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

## <a name="install-powershell-module"></a>安装 PowerShell 模块

从 PowerShell 库安装 Microsoft.AzureBlockchainService.ConsortiumManagement.PS 包。

```powershell
Install-Module -Name Microsoft.AzureBlockchainService.ConsortiumManagement.PS -Scope CurrentUser
Import-Module Microsoft.AzureBlockchainService.ConsortiumManagement.PS
```

## <a name="establish-a-web3-connection"></a>建立 Web3 连接

若要管理联盟成员，需要建立 Web3 连接到 Azure 区块链服务成员终结点。 此脚本可用于设置调用联盟管理 cmdlet 时可以使用的全局变量。

```powershell
$Connection = New-Web3Connection -RemoteRPCEndpoint '<Endpoint address>'
$MemberAccount = Import-Web3Account -ManagedAccountAddress '<Member account address>' -ManagedAccountPassword '<Member account password>'
$ContractConnection = Import-ConsortiumManagementContracts -RootContractAddress '<RootContract address>' -Web3Client $Connection
```

替换\<成员帐户密码\>创建成员时使用的成员帐户密码。

在 Azure 门户中找到其他值：

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 导航到您的默认 Azure 区块链服务成员**概述**页。

    ![成员概述](./media/manage-consortium-powershell/member-overview.png)

    替换\<会员帐户\>，并\<RootContract 地址\>在门户中的值。

1. 终结点地址中，选择**事务节点**选择事务的节点。
1. 选择**连接字符串**。

    ![连接字符串](./media/manage-consortium-powershell/connection-strings.png)

    替换\<终结点地址\>中的值**HTTPS （访问键 1）** 或**HTTPS （访问键 2）**。

## <a name="network-and-smart-contract-management"></a>网络和智能协定管理

使用网络和智能协定 cmdlet 建立与负责联盟管理智能协定将区块链终结点的连接。

### <a name="import-consortiummanagementcontracts"></a>Import-ConsortiumManagementContracts

连接到联盟管理智能协定，用于管理和实施联盟中的成员。

```powershell
Import-ConsortiumManagementContracts -RootContractAddress <String> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| RootContractAddress | 根协定地址的联盟管理智能协定 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
Import-ConsortiumManagementContracts -RootContractAddress '<RootContract address>'  -Web3Client $Connection
```

### <a name="import-web3account"></a>Import-Web3Account

使用此 cmdlet 以创建对象来保存信息远程节点管理帐户。

```powershell
Import-Web3Account -ManagedAccountAddress <String> -ManagedAccountPassword <String>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| ManagedAccountAddress | 区块链成员帐户地址 | 是 |
| ManagedAccountPassword | 帐户地址密码 | 是 |

**示例**

```powershell
Import-Web3Account -ManagedAccountAddress '<Member account address>'  -ManagedAccountPassword '<Member account password>'
```

### <a name="new-web3connection"></a>New-Web3Connection

建立连接到 RPC 终结点的事务节点。

```powershell
New-Web3Connection [-RemoteRPCEndpoint <String>]
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| RemoteRPCEndpoint | 区块链成员终结点地址 | 是 |


**示例**

```powershell
New-Web3Connection -RemoteRPCEndpoint '<Endpoint address>'
```

## <a name="consortium-member-management"></a>联盟成员管理

联盟成员管理 cmdlet 用于管理联盟中的成员。 可用操作取决于你联盟的角色。

### <a name="get-blockchainmember"></a>Get-BlockchainMember

获取成员的详细信息或列出联盟的成员。

```powershell
Get-BlockchainMember [[-Name] <String>] -Members <IContract> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| 名称 | 你想要检索的详细信息上的 Azure 区块链服务成员的名称。 如果你提供成员名称，则返回该成员的详细信息。 如果省略名称时，则返回所有联盟成员的列表。 | 否 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | Get-BlockchainMember -Name <Member Name>
```

**示例输出**

```
Name           : myblockchainmember
CorrelationId  : 0
DisplayName    : myCompany
SubscriptionId : <Azure subscription ID>
AccountAddress : 0x85b911c9e103d6405573151258d668479e9ebeef
Role           : ADMIN
```

### <a name="remove-blockchainmember"></a>Remove-BlockchainMember

删除区块链成员。

```powershell
Remove-BlockchainMember -Name <String> -Members <IContract> -Web3Account <IAccount> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| 名称 | 若要删除的成员名称 | 是 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Account | 获取从导入 Web3Account Web3Account 对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | Remove-BlockchainMember -Name <Member Name> -Web3Account $MemberAccount
```

### <a name="set-blockchainmember"></a>Set-BlockchainMember

将区块链设置包括显示名称和联盟角色的成员属性。

联盟管理员可以设置**DisplayName**并**角色**所有成员。 与用户角色的联盟成员才可以更改其自己的成员的显示名称。

```powershell
Set-BlockchainMember -Name <String> [-DisplayName <String>] [-AccountAddress <String>] [-Role <String>]
 -Members <IContract> -Web3Account <IAccount> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| 名称 | 区块链成员的名称 | 是 |
| DisplayName | 新的显示名称 | 否 |
| AccountAddress | 帐户地址 | 否 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Account | 获取从导入 Web3Account Web3Account 对象 | 是 |
| Web3Client |  获取从新建 Web3Connection Web3Client 对象| 是 |

**示例**

```powershell
$ContractConnection | Set-BlockchainMember -Name <Member Name> -DisplayName <Display name> -Web3Account $MemberAccount
```

## <a name="consortium-member-invitation-management"></a>联盟成员邀请管理

联盟成员邀请管理 cmdlet 用于管理联盟成员邀请。 可用操作取决于你联盟的角色。

### <a name="new-blockchainmemberinvitation"></a>New-BlockchainMemberInvitation

邀请到联盟的新成员。

```powershell
New-BlockchainMemberInvitation -SubscriptionId <String> -Role <String> -Members <IContract>
 -Web3Account <IAccount> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| SubscriptionId | Azure 订阅 ID 的已邀请成员 | 是 |
| 角色 | 联盟角色。 值可以是管理员或用户。 管理员是联盟管理员角色。 用户是联盟成员角色。 | 是 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Account | 获取从导入 Web3Account Web3Account 对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | New-BlockchainMemberInvitation -SubscriptionId <Azure Subscription ID> -Role USER -Web3Account $MemberAccount
```

### <a name="get-blockchainmemberinvitation"></a>Get-BlockchainMemberInvitation

检索或列出联盟成员邀请状态。

```powershell
Get-BlockchainMemberInvitation [[-SubscriptionId] <String>] -Members <IContract> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| SubscriptionId | 已邀请成员的 azure 订阅 ID。 如果提供订阅 Id，则返回邀请的订阅 ID 的详细信息。 如果省略订阅 Id，则返回的所有成员邀请列表。 | 否 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | Get-BlockchainMemberInvitation – SubscriptionId <Azure subscription ID>
```

**示例输出**

```
SubscriptionId                       Role CorrelationId
--------------                       ---- -------------
<Azure subscription ID>              USER             2
```

### <a name="remove-blockchainmemberinvitation"></a>Remove-BlockchainMemberInvitation

撤消联盟成员邀请。

```powershell
Remove-BlockchainMemberInvitation -SubscriptionId <String> -Members <IContract> -Web3Account <IAccount>
 -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| SubscriptionId | 若要吊销的成员的 azure 订阅 ID | 是 |
| 成员 | 获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Account | 获取从导入 Web3Account Web3Account 对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | Remove-BlockchainMemberInvitation -SubscriptionId <Subscription ID> -Web3Account $MemberAccount
```

### <a name="set-blockchainmemberinvitation"></a>Set-BlockchainMemberInvitation

集**角色**现有邀请。 只有联盟管理员可以更改的邀请。

```powershell
Set-BlockchainMemberInvitation -SubscriptionId <String> -Role <String> -Members <IContract>
 -Web3Account <IAccount> -Web3Client <IClient>
```

| 参数 | 描述 | 需要 |
|-----------|-------------|:--------:|
| SubscriptionId | Azure 订阅 ID 的已邀请成员 | 是 |
| 角色 | 邀请新联盟的角色。 值可以是**用户**或**管理员** | 是 |
| 成员 |  获取从导入 ConsortiumManagementContracts 成员对象 | 是 |
| Web3Account | 获取从导入 Web3Account Web3Account 对象 | 是 |
| Web3Client | 获取从新建 Web3Connection Web3Client 对象 | 是 |

**示例**

```powershell
$ContractConnection | Set-BlockchainMemberInvitation -SubscriptionId <Azure subscription ID> -Role USER -Web3Account $MemberAccount
```

## <a name="next-steps"></a>后续步骤

有关财团、 成员和节点的详细信息，请参阅：

> [!div class="nextstepaction"]
> [Azure 区块链服务联盟](consortium.md)