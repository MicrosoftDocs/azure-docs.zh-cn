---
title: Azure Blockchain Workbench 消息集成概述
description: 有关在 Azure Blockchain Workbench 中使用消息的概述。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: brendal
manager: femila
ms.openlocfilehash: 8f63c62cd23fef5565628793379afd8bcc9f447b
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510159"
---
# <a name="azure-blockchain-workbench-messaging-integration"></a>Azure Blockchain Workbench 消息集成

除了提供 REST API 以外，Azure Blockchain Workbench 还提供基于消息的集成。 Workbench 通过 Azure 事件网格发布以账本为中心的事件，使下游使用者能够根据这些事件引入数据或执行操作。 对于需要可靠消息传递的客户端，Azure Blockchain Workbench 还会将消息传送到 Azure 服务总线终结点。

## <a name="input-apis"></a>输入 API

若要从外部系统发起事务以创建用户、创建合同和更新合同，可以使用消息传送输入 API 针对账本执行事务。 有关演示输入 API 的示例，请参阅[消息传送集成示例](https://aka.ms/blockchain-workbench-integration-sample)。

下面是目前可用的输入 API。

### <a name="create-user"></a>创建用户

创建新用户。

该请求需要填写以下字段：

| **名称**             | **说明**                                      |
|----------------------|------------------------------------------------------|
| requestId            | 客户端提供的 GUID                                |
| firstName            | 用户的名字                              |
| lastName             | 用户的姓氏                               |
| emailAddress         | 用户的电子邮件地址                           |
| externalId           | 用户的 Azure AD 对象 ID                      |
| connectionId         | 区块链连接的唯一标识符 |
| messageSchemaVersion | 消息传送架构版本                            |
| messageName          | **CreateUserRequest**                               |

示例：

``` json
{
    "requestId": "e2264523-6147-41fc-bbbb-edba8e44562d",
    "firstName": "Ali",
    "lastName": "Alio",
    "emailAddress": "aa@contoso.com",
    "externalId": "6a9b7f65-ffff-442f-b3b8-58a35abd1bcd",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateUserRequest"
}
```

Blockchain Workbench 返回包含以下字段的响应：

| **名称**              | **说明**                                                                                                             |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------|
| requestId             | 客户端提供的 GUID |
| userId                | 创建的用户的 ID |
| userChainIdentifier   | 在区块链网络上创建的用户的地址。 在 Ethereum 中，地址是用户的 **链上** 地址。 |
| connectionId          | 区块链连接的唯一标识符|
| messageSchemaVersion  | 消息传送架构版本 |
| messageName           | **CreateUserUpdate** |
| status                | 用户创建请求的状态。  如果成功，则值为 **Success**。 如果失败，则值为 **Failure**。     |
| additionalInformation | 基于状态提供的附加信息 |

Blockchain Workbench 返回的 **create user** 成功响应示例：

``` json
{ 
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Success", 
    "additionalInformation": { } 
} 
```

如果请求不成功，则会在附加信息中包含有关失败的详细信息。

``` json
{
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": null, 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Failure", 
    "additionalInformation": { 
        "errorCode": 4000, 
        "errorMessage": "User cannot be provisioned on connection." 
    }
}
```

### <a name="create-contract"></a>创建合同

创建新合同。

该请求需要填写以下字段：

| **名称**             | **说明**                                                                                                           |
|----------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId            | 客户端提供的 GUID |
| userChainIdentifier  | 在区块链网络上创建的用户的地址。 在 Ethereum 中，此地址是用户的**链上**地址。 |
| applicationName      | 应用程序的名称 |
| version              | 应用程序的版本。 在启用了应用程序的多个版本时是必需的。 否则，版本是可选的。 有关应用程序版本控制的详细信息，请参阅 [Azure Blockchain Workbench 应用程序版本控制](version-app.md)。 |
| workflowName         | 工作流的名称 |
| parameters           | 合同创建请求的参数输入 |
| connectionId         | 区块链连接的唯一标识符 |
| messageSchemaVersion | 消息传送架构版本 |
| messageName          | **CreateContractRequest** |

示例：

``` json
{ 
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211", 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "applicationName": "AssetTransfer",
    "version": "1.0",
    "workflowName": "AssetTransfer", 
    "parameters": [ 
        { 
            "name": "description", 
            "value": "a 1969 dodge charger" 
        }, 
        { 
            "name": "price", 
            "value": "12345" 
        } 
    ], 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateContractRequest" 
}
```

Blockchain Workbench 返回包含以下字段的响应：

| **名称**                 | **说明**                                                                   |
|--------------------------|-----------------------------------------------------------------------------------|
| requestId                | 客户端提供的 GUID                                                             |
| contractId               | Azure Blockchain Workbench 中的合同的唯一标识符 |
| contractLedgerIdentifier | 账本中合同的地址                                            |
| connectionId             | 区块链连接的唯一标识符                               |
| messageSchemaVersion     | 消息传送架构版本                                                         |
| messageName              | **CreateContractUpdate**                                                      |
| status                   | 合同创建请求的状态。  可能的值：**Submitted**、**Committed**、**Failure**。  |
| additionalInformation    | 基于状态提供的附加信息                              |

Blockchain Workbench 返回的页面提交 **create contract** 响应示例：

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Submitted"
    "additionalInformation": { }
}
```

Blockchain Workbench 返回的事务提交 **create contract** 响应示例：

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Committed",
    "additionalInformation": { }
}
```

如果请求不成功，则会在附加信息中包含有关失败的详细信息。

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": null,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Failure"
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract cannot be provisioned on connection."
    }
}
```

### <a name="create-contract-action"></a>创建合同操作

创建新的合同操作。

该请求需要填写以下字段：

| **名称**                 | **说明**                                                                                                           |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId                | 客户端提供的 GUID |
| userChainIdentifier      | 在区块链网络上创建的用户的地址。 在 Ethereum 中，此地址是用户的**链上**地址。 |
| contractLedgerIdentifier | 账本中合同的地址 |
| version                  | 应用程序的版本。 在启用了应用程序的多个版本时是必需的。 否则，版本是可选的。 有关应用程序版本控制的详细信息，请参阅 [Azure Blockchain Workbench 应用程序版本控制](version-app.md)。 |
| workflowFunctionName     | 工作流函数的名称 |
| parameters               | 合同创建请求的参数输入 |
| connectionId             | 区块链连接的唯一标识符 |
| messageSchemaVersion     | 消息传送架构版本 |
| messageName              | **CreateContractActionRequest** |

示例：

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398",
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "version": "1.0",
    "workflowFunctionName": "modify",
    "parameters": [
        {
            "name": "description",
            "value": "a 1969 dodge charger"
        },
        {
            "name": "price",
            "value": "12345"
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionRequest"
}
```

Blockchain Workbench 返回包含以下字段的响应：

| **名称**              | **说明**                                                                   |
|-----------------------|-----------------------------------------------------------------------------------|
| requestId             | 客户端提供的 GUID|
| contractId            | Azure Blockchain Workbench 中的合同的唯一标识符 |
| connectionId          | 区块链连接的唯一标识符 |
| messageSchemaVersion  | 消息传送架构版本 |
| messageName           | **CreateContractActionUpdate** |
| status                | 合同操作请求的状态。 可能的值：**Submitted**、**Committed**、**Failure**。                         |
| additionalInformation | 基于状态提供的附加信息 |

Blockchain Workbench 返回的页面提交 **create contract action** 响应示例：

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Submitted",
    "additionalInformation": { }
}
```

Blockchain Workbench 返回的事务提交 **create contract action** 响应示例：

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Committed"
    "additionalInformation": { }
}
```

如果请求不成功，则会在附加信息中包含有关失败的详细信息。

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Failure"
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract action cannot be provisioned on connection."
    }
}
```

### <a name="input-api-error-codes-and-messages"></a>输入 API 错误代码和消息

**错误代码 4000:错误的请求错误**
- connectionId 无效
- CreateUserRequest 反序列化失败
- CreateContractRequest 反序列化失败
- CreateContractActionRequest 反序列化失败
- 应用程序 {由应用程序名称标识} 不存在
- 应用程序 {由应用程序名称标识} 没有工作流
- UserChainIdentifier 不存在
- 合同 {由账本标识符标识} 不存在
- 合同 {由账本标识符标识} 没有函数 {工作流函数名称}
- UserChainIdentifier 不存在

**错误代码 4090:冲突错误**
- 用户已存在
- 合同已存在
- 合同操作已存在

**错误代码 5000:内部服务器错误**
- 异常消息

## <a name="event-notifications"></a>事件通知

可以使用事件通知将 Blockchain Workbench 中发生的事件及其连接到的区块链网络告知用户和下游系统。 可以直接在代码中使用，或者结合逻辑应用和 Flow 等工具使用事件通知来触发到下游系统的数据流。

有关可接收的各种消息的详细信息，请参阅[通知消息参考](#notification-message-reference)。

### <a name="consuming-event-grid-events-with-azure-functions"></a>配合 Azure Functions 使用事件网格事件

如果用户想要使用事件网格来接收有关 Blockchain Workbench 中所发生的事件的通知，你可以通过 Azure Functions 使用来自事件网格的事件。

1. 在 Azure 门户中创建一个 **Azure 函数应用**。
2. 创建新函数。
3. 找到事件网格的模板。 其中会显示用于读取消息的基本模板代码。 根据需要修改代码。
4. 保存函数。 
5. 从 Blockchain Workbench 的资源组中选择“事件网格”。

### <a name="consuming-event-grid-events-with-logic-apps"></a>配合逻辑应用使用事件网格事件

1. 在 Azure 门户中创建一个新的 **Azure 逻辑应用**。
2. 在门户中打开 Azure 逻辑应用时，系统会提示选择触发器。 选择“Azure 事件网格 - 发生资源事件时”。
3. 显示工作流设计器后，系统会提示登录。
4. 选择订阅。 资源为 **Microsoft.EventGrid.Topics**。 从 Azure Blockchain Workbench 资源组的资源名称中选择“资源名称”。
5. 从 Blockchain Workbench 的资源组中选择“事件网格”。

## <a name="using-service-bus-topics-for-notifications"></a>使用服务总线主题发送通知

可以使用服务总线主题通知用户有关 Blockchain Workbench 中发生的事件。 

1. 浏览到 Workbench 资源组中的“服务总线”。
2. 选择“主题”。
3. 选择“egress-topic”。
4. 创建此主题的新订阅。 获取该订阅的密钥。
5. 创建一个程序，用于订阅此订阅的事件。

### <a name="consuming-service-bus-messages-with-logic-apps"></a>配合逻辑应用使用服务总线消息

1. 在 Azure 门户中创建一个新的 **Azure 逻辑应用**。
2. 在门户中打开 Azure 逻辑应用时，系统会提示选择触发器。 在搜索框中键入“服务总线”，然后选择适合与服务总线之间的交互类型的触发器。 例如，选择“服务总线 - 主题订阅中收到邮件时(自动完成)”。
3. 显示工作流设计器后，指定服务总线的连接信息。
4. 选择订阅并指定 **workbench-external** 主题。
5. 开发应用程序的逻辑，以利用此触发器的消息。

## <a name="notification-message-reference"></a>通知消息参考

具体取决于**messageName**，通知消息具有以下消息类型之一。

### <a name="block-message"></a>块消息

包含有关单个块的信息。 *BlockMessage* 中的某个节包含块级信息，另一个节包含事务信息。

| 名称 | 描述 |
|------|-------------|
| 阻止 | 包含[块信息](#block-information) |
| transactions | 包含块的[事务信息](#transaction-information)集合 |
| connectionId | 连接的唯一标识符 |
| messageSchemaVersion | 消息传送架构版本 |
| messageName | **BlockMessage** |
| additionalInformation | 提供的附加信息 |

#### <a name="block-information"></a>块信息

| 名称              | 描述 |
|-------------------|-------------|
| blockId           | Azure Blockchain Workbench 中的块的唯一标识符 |
| blockNumber       | 账本中块的唯一标识符 |
| blockHash         | 块的哈希 |
| previousBlockHash | 前一个块的哈希 |
| blockTimestamp    | 块的时间戳 |

#### <a name="transaction-information"></a>事务信息

| 名称               | 描述 |
|--------------------|-------------|
| transactionId      | Azure Blockchain Workbench 中的事务的唯一标识符 |
| transactionHash    | 账本中事务的哈希 |
| 来自               | 事务来源账本中的唯一标识符 |
| 至                 | 事务目标账本中的唯一标识符 |
| provisioningStatus | 标识事务的当前预配过程状态。 可能的值包括： </br>0 – API 已在数据库中创建事务</br>1 – 事务已发送到账本</br>2 – 事务已成功提交到账本</br>3 或 4 - 事务无法提交到账本</br>5 - 事务已成功提交到账本 |

Blockchain Workbench 中的 *BlockMessage* 示例：

``` json
{
    "block": {
        "blockId": 123
        "blockNumber": 1738312,
        "blockHash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
        "previousBlockHash": null,
        "blockTimestamp": "2018-10-09T23:35:58Z",
    },
    "transactions": [
        {
            "transactionId": 234
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": null,
            "provisioningStatus": 1
        },
        {
            "transactionId": 235
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xadd97e1e595916e29ea94fda894941574000ffff",
            "to": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff",
            "provisioningStatus": 2
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "BlockMessage",
    "additionalInformation": {}
}
```

### <a name="contract-message"></a>合同消息

包含有关合同的信息。 该消息中的某个节包含合同属性，另一个节包含事务信息。 修改了特定块的合同的所有事务包含在 transaction 节中。

| 名称 | 描述 |
|------|-------------|
| blockId | Azure Blockchain Workbench 中的块的唯一标识符 |
| blockHash | 块的哈希 |
| modifyingTransactions | [修改了合同的事务](#modifying-transaction-information) |
| contractId | Azure Blockchain Workbench 中的合同的唯一标识符 |
| contractLedgerIdentifier | 账本中合同的唯一标识符 |
| contractProperties | [合同的属性](#contract-properties) |
| isNewContract | 指示此合同是否是新建的。 可能的值为：true：此合同是新建的合同。 false：此合同是合同更新。 |
| connectionId | 连接的唯一标识符 |
| messageSchemaVersion | 消息传送架构版本 |
| messageName | **ContractMessage** |
| additionalInformation | 提供的附加信息 |

#### <a name="modifying-transaction-information"></a>修改事务信息

| 名称               | 描述 |
|--------------------|-------------|
| transactionId | Azure Blockchain Workbench 中的事务的唯一标识符 |
| transactionHash | 账本中事务的哈希 |
| 来自 | 事务来源账本中的唯一标识符 |
| 至 | 事务目标账本中的唯一标识符 |

#### <a name="contract-properties"></a>合同属性

| 名称               | 描述 |
|--------------------|-------------|
| workflowPropertyId | Azure Blockchain Workbench 中工作流属性的唯一标识符 |
| 名称 | 工作流属性的名称 |
| value | 工作流属性的值 |

Blockchain Workbench 中的 *ContractMessage* 示例：

``` json
{
    "blockId": 123,
    "blockhash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
    "modifyingTransactions": [
        {
            "transactionId": 234,
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07ffff"
        },
        {
            "transactionId": 235,
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
        }
    ],
    "contractId": 111,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "contractProperties": [
        {
            "workflowPropertyId": 1,
            "name": "State",
            "value": "0"
        },
        {
            "workflowPropertyId": 2,
            "name": "Description",
            "value": "1969 Dodge Charger"
        },
        {
            "workflowPropertyId": 3,
            "name": "AskingPrice",
            "value": "30000"
        },
        {
            "workflowPropertyId": 4,
            "name": "OfferPrice",
            "value": "0"
        },
        {
            "workflowPropertyId": 5,
            "name": "InstanceAppraiser",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 6,
            "name": "InstanceBuyer",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 7,
            "name": "InstanceInspector",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 8,
            "name": "InstanceOwner",
            "value": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
        },
        {
            "workflowPropertyId": 9,
            "name": "ClosingDayOptions",
            "value": "[21,48,69]"
        }
    ],
    "isNewContract": false,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "ContractMessage",
    "additionalInformation": {}
}
```

### <a name="event-message-contract-function-invocation"></a>事件消息：合同函数调用

包含调用合同函数时的信息，例如函数名称、参数输入和函数调用方。

| 名称 | 描述 |
|------|-------------|
| eventName                   | **ContractFunctionInvocation** |
| 调用方                      | [调用方信息](#caller-information) |
| contractId                  | Azure Blockchain Workbench 中的合同的唯一标识符 |
| contractLedgerIdentifier    | 账本中合同的唯一标识符 |
| functionName                | 函数的名称 |
| parameters                  | [参数信息](#parameter-information) |
| transaction                 | 事务信息 |
| inTransactionSequenceNumber | 块中事务的序列号 |
| connectionId                | 连接的唯一标识符 |
| messageSchemaVersion        | 消息传送架构版本 |
| messageName                 | **EventMessage** |
| additionalInformation       | 提供的附加信息 |

#### <a name="caller-information"></a>调用方信息

| 名称 | 描述 |
|------|-------------|
| 类型 | 调用方类型，例如用户或合同 |
| ID | Azure Blockchain Workbench 中调用方的唯一标识符 |
| ledgerIdentifier | 账本中调用方的唯一标识符 |

#### <a name="parameter-information"></a>参数信息

| 名称 | 描述 |
|------|-------------|
| 名称 | 参数名称 |
| value | 参数值 |

#### <a name="event-message-transaction-information"></a>事件消息事务信息

| 名称               | 描述 |
|--------------------|-------------|
| transactionId      | Azure Blockchain Workbench 中的事务的唯一标识符 |
| transactionHash    | 账本中事务的哈希 |
| 来自               | 事务来源账本中的唯一标识符 |
| 至                 | 事务目标账本中的唯一标识符 |

Blockchain Workbench 中的 *EventMessage ContractFunctionInvocation* 示例：

``` json
{
    "eventName": "ContractFunctionInvocation",
    "caller": {
        "type": "User",
        "id": 21,
        "ledgerIdentifier": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60ffff"
    },
    "contractId": 34,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "functionName": "Modify",
    "parameters": [
        {
            "name": "description",
            "value": "a new description"
        },
        {
            "name": "price",
            "value": "4567"
        }
    ],
    "transaction": {
        "transactionId": 234,
        "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
        "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
        "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
    },
    "inTransactionSequenceNumber": 1,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

### <a name="event-message-application-ingestion"></a>事件消息：应用程序引入

包含将应用程序上传到 Workbench 时的信息，例如所上传应用程序的名称和版本。

| 名称 | 描述 |
|------|-------------|
| eventName | **ApplicationIngestion** |
| applicationId | Azure Blockchain Workbench 中应用程序的唯一标识符 |
| applicationName | 应用程序名 |
| applicationDisplayName | 应用程序显示名称 |
| applicationVersion | 应用程序版本: |
| applicationDefinitionLocation | 应用程序配置文件所在的 URL |
| contractCodes | 应用程序的[合同代码](#contract-code-information)集合 |
| applicationRoles | 应用程序的[应用程序角色](#application-role-information)集合 |
| applicationWorkflows | 应用程序的[应用程序工作流](#application-workflow-information)集合 |
| connectionId | 连接的唯一标识符 |
| messageSchemaVersion | 消息传送架构版本 |
| messageName | **EventMessage** |
| additionalInformation | 此处提供的其他信息包括应用程序工作流状态和转换信息。 |

#### <a name="contract-code-information"></a>合同代码信息

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中合同代码文件的唯一标识符 |
| ledgerId | Azure Blockchain Workbench 中账本的唯一标识符 |
| 位置 | 合同代码文件所在的 URL |

#### <a name="application-role-information"></a>应用程序角色信息

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中应用程序角色的唯一标识符 |
| 名称 | 应用程序角色的名称 |

#### <a name="application-workflow-information"></a>应用程序工作流信息

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中应用程序工作流的唯一标识符 |
| 名称 | 应用程序工作流名称 |
| displayName | 应用程序工作流显示名称 |
| 函数 | 集合的[用于应用程序工作流函数](#workflow-function-information)|
| states | [应用程序工作流的状态](#workflow-state-information)集合 |
| properties | 应用程序[工作流属性信息](#workflow-property-information) |

##### <a name="workflow-function-information"></a>工作流函数信息

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中应用程序工作流函数的唯一标识符 |
| 名称 | 函数名 |
| parameters | 函数的参数 |

##### <a name="workflow-state-information"></a>工作流状态信息

| 名称 | 描述 |
|------|-------------|
| 名称 | 状态名称 |
| displayName | 状态显示名称 |
| style | 状态样式（success 或 failure） |

##### <a name="workflow-property-information"></a>工作流属性信息

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中应用程序工作流属性的唯一标识符 |
| 名称 | 属性名称 |
| 类型 | 属性类型 |

Blockchain Workbench 中的 *EventMessage ApplicationIngestion* 示例：

``` json
{
    "eventName": "ApplicationIngestion",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": “1.0”,
    "applicationDefinitionLocation": "http://url"
    "contractCodes": [
        {
            "id": 23,
            "ledgerId": 1,
            "location": "http://url"
        }
    ],
    "applicationRoles": [
            {
                "id": 134,
                "name": "Buyer"
            },
            {
                "id": 135,
                "name": "Seller"
            }
       ],
    "applicationWorkflows": [
        {
            "id": 89,
            "name": "AssetTransfer",
            "displayName": "Asset Transfer",
            "functions": [
                {
                    "id": 912,
                    "name": "",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                },
                {
                    "id": 913,
                    "name": "modify",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                }
            ],
            "states": [ 
                 {
                      "name": "Created",
                      "displayName": "Created",
                      "style" : "Success"
                 },
                 {
                      "name": "Terminated",
                      "displayName": "Terminated",
                      "style" : "Failure"
                 }
            ],
            "properties": [
                {
                    "id": 879,
                    "name": "Description",
                    "type": {
                                "name": "string"
                     }
                },
                {
                    "id": 880,
                    "name": "Price",
                    "type": {
                                "name": "int"
                     }
                }
            ]
        }
    ]
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation":
        {
            "states" :
            [
                {
                    "Name": "BuyerAccepted",
                    "Transitions": [
                        {
                            "DisplayName": "Accept"
                            "AllowedRoles": [ ],
                            "AllowedInstanceRoles": [ "InstanceOwner" ],
                            "Function": "Accept",
                            "NextStates": [ "SellerAccepted" ]
                        }
                    ]
                }
            ]
        }
}
```

### <a name="event-message-role-assignment"></a>事件消息：角色分配

包含在 Workbench 中为用户分配角色时的信息，例如，谁执行了角色分配，以及角色名称和相应的应用程序。

| 名称 | 描述 |
|------|-------------|
| eventName | **RoleAssignment** |
| applicationId | Azure Blockchain Workbench 中应用程序的唯一标识符 |
| applicationName | 应用程序名 |
| applicationDisplayName | 应用程序显示名称 |
| applicationVersion | 应用程序版本: |
| applicationRole        | 有关[应用程序角色](#roleassignment-application-role)的信息 |
| assigner               | 有关[分配者](#roleassignment-assigner)的信息 |
| assignee               | 有关[被分配者](#roleassignment-assignee)的信息 |
| connectionId           | 连接的唯一标识符 |
| messageSchemaVersion   | 消息传送架构版本 |
| messageName            | **EventMessage** |
| additionalInformation  | 提供的附加信息 |

#### <a name="roleassignment-application-role"></a>RoleAssignment 应用程序角色

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中应用程序角色的唯一标识符 |
| 名称 | 应用程序角色的名称 |

#### <a name="roleassignment-assigner"></a>RoleAssignment 分配者

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中用户的唯一标识符 |
| 类型 | 分配者的类型 |
| chainIdentifier | 账本中用户的唯一标识符 |

#### <a name="roleassignment-assignee"></a>RoleAssignment 被分配者

| 名称 | 描述 |
|------|-------------|
| ID | Azure Blockchain Workbench 中用户的唯一标识符 |
| 类型 | 被分配者的类型 |
| chainIdentifier | 账本中用户的唯一标识符 |

Blockchain Workbench 中的 *EventMessage RoleAssignment* 示例：

``` json
{
    "eventName": "RoleAssignment",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": “1.0”,
    "applicationRole": {
        "id": 134,
        "name": "Buyer"
    },
    "assigner": {
        "id": 1,
        "type": null,
        "chainIdentifier": "0xeFFC7766d38aC862d79706c3C5CEEf089564ffff"
    },
    "assignee": {
        "id": 3,
        "type": null,
        "chainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
    },
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

## <a name="next-steps"></a>后续步骤

- [智能合同集成模式](integration-patterns.md)
