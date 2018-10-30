---
title: 在 Azure Blockchain Workbench 中创建区块链应用程序
description: 如何在 Azure Blockchain Workbench 中创建区块链应用程序。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: a7ca3f42874bc844bc0036e37a790ffebdc5f8d8
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240740"
---
# <a name="create-a-blockchain-application-in-azure-blockchain-workbench"></a>在 Azure Blockchain Workbench 中创建区块链应用程序

可以使用 Azure Blockchain Workbench 创建区块链应用程序，用于表示由配置和智能合约代码定义的多方工作流。

学习如何：

> [!div class="checklist"]
> * 配置区块链应用程序
> * 创建智能合约代码文件
> * 将区块链应用程序添加到 Blockchain Workbench
> * 将成员添加到区块链应用程序

## <a name="prerequisites"></a>先决条件

* Blockchain Workbench 部署。 有关部署详细信息，请参阅 [Azure Blockchain Workbench 部署](deploy.md)。
* 与 Blockchain Workbench 关联的租户中的 Azure Active Directory 用户。 有关详细信息，请参阅[在 Azure Blockchain Workbench 中添加 Azure AD 用户](manage-users.md#add-azure-ad-users)。
* Blockchain Workbench 管理员帐户。 有关详细信息，请参阅[在 Azure Blockchain Workbench 中添加 Blockchain Workbench 管理员](manage-users.md#manage-blockchain-workbench-administrators)。

## <a name="hello-blockchain"></a>你好，Blockchain！

让我们生成一个基本的应用程序，其中的请求方发送请求，响应方发送对请求的响应。 例如，请求可以是“你好吗？”，而响应可以是“我很好！”。 请求和响应都记录在底层区块链中。 

按照创建应用程序文件的步骤进行操作，也可以[从 GitHub 下载示例](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/hello-blockchain)。 

## <a name="configuration-file"></a>配置文件

配置元数据定义区块链应用程序的高级工作流和交互模型。 配置元数据表示区块链应用程序的工作流阶段和交互模型。

1. 在偏好的编辑器中，创建名为 `HelloBlockchain.json` 的文件。
2. 添加以下 JSON 以定义区块链应用程序的配置。

    ``` json
    {
      "ApplicationName": "HelloBlockchain",
      "DisplayName": "Hello, Blockchain!",
      "Description": "A simple application to send request and get response",
      "ApplicationRoles": [
        {
          "Name": "Requestor",
          "Description": "A person sending a request."
        },
        {
          "Name": "Responder",
          "Description": "A person responding to a request"
        }
      ],
      "Workflows": [
        {
          "Name": "HelloBlockchain",
          "DisplayName": "Request Response",
          "Description": "A simple workflow to send a request and receive a response.",
          "Initiators": [ "Requestor" ],
          "StartState": "Request",
          "Properties": [
            {
              "Name": "State",
              "DisplayName": "State",
              "Description": "Holds the state of the contract.",
              "Type": {
                "Name": "state"
              }
            },
            {
              "Name": "Requestor",
              "DisplayName": "Requestor",
              "Description": "A person sending a request.",
              "Type": {
                "Name": "Requestor"
              }
            },
            {
              "Name": "Responder",
              "DisplayName": "Responder",
              "Description": "A person sending a response.",
              "Type": {
                "Name": "Responder"
              }
            },
            {
              "Name": "RequestMessage",
              "DisplayName": "Request Message",
              "Description": "A request message.",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "ResponseMessage",
              "DisplayName": "Response Message",
              "Description": "A response message.",
              "Type": {
                "Name": "string"
              }
            }
          ],
          "Constructor": {
            "Parameters": [
              {
                "Name": "message",
                "Description": "...",
                "DisplayName": "Request Message",
                "Type": {
                  "Name": "string"
                }
              }
            ]
          },
          "Functions": [
            {
              "Name": "SendRequest",
              "DisplayName": "Request",
              "Description": "...",
              "Parameters": [
                {
                  "Name": "requestMessage",
                  "Description": "...",
                  "DisplayName": "Request Message",
                  "Type": {
                    "Name": "string"
                  }
                }
              ]
            },
            {
              "Name": "SendResponse",
              "DisplayName": "Response",
              "Description": "...",
              "Parameters": [
                {
                  "Name": "responseMessage",
                  "Description": "...",
                  "DisplayName": "Response Message",
                  "Type": {
                    "Name": "string"
                  }
                }
              ]
            }
          ],
          "States": [
            {
              "Name": "Request",
              "DisplayName": "Request",
              "Description": "...",
              "PercentComplete": 50,
              "Value": 0,
              "Style": "Success",
              "Transitions": [
                {
                  "AllowedRoles": ["Responder"],
                  "AllowedInstanceRoles": [],
                  "Description": "...",
                  "Function": "SendResponse",
                  "NextStates": [ "Respond" ],
                  "DisplayName": "Send Response"
                }
              ]
            },
            {
              "Name": "Respond",
              "DisplayName": "Respond",
              "Description": "...",
              "PercentComplete": 90,
              "Value": 1,
              "Style": "Success",
              "Transitions": [
                {
                  "AllowedRoles": [],
                  "AllowedInstanceRoles": ["Requestor"],
                  "Description": "...",
                  "Function": "SendRequest",
                  "NextStates": [ "Request" ],
                  "DisplayName": "Send Request"
                }
              ]
            }
          ]
        }
      ]
    }
    ```

3. 保存 `HelloBlockchain.json` 文件。

配置文件包含多个节。 有关每个节的详细信息如下：

### <a name="application-metadata"></a>应用程序元数据

配置文件的开头包含有关应用程序的信息，包括应用程序名称和说明。

### <a name="application-roles"></a>应用程序角色

应用程序角色节定义可在区块链应用程序中执行操作或参与活动的用户角色。 请根据功能定义一组不同的角色。 在请求-响应方案中，请求方（充当生成请求的实体）与响应方（充当生成响应的实体）的功能没有差别。

### <a name="workflows"></a>工作流

工作流定义合约的一个或多个阶段与操作。 在请求-响应方案中，工作流的第一个阶段（状态）是请求方（角色）执行某个操作（转换）以发送请求（函数）。 下一个阶段（状态）是响应方（角色）执行某个操作（转换）以发送响应（函数）。 应用程序的工作流可能涉及到描述合约流所需的属性、函数和状态。 

有关配置文件内容的详细信息，请参阅 [Azure 区块链工作流配置参考](configuration.md)。

## <a name="smart-contract-code-file"></a>智能合约代码文件

智能合约表示区块链应用程序的业务逻辑。 目前，Blockchain Workbench 支持对区块链账本使用 Ethereum。 Ethereum 使用 [Solidity](https://solidity.readthedocs.io) 作为编程语言来为智能合约编写自我实施的业务逻辑。

Solidity 中的智能合约类似于面向对象的语言中的类。 每个合约包含用于实现智能合约阶段和操作的状态与函数。

在偏好的编辑器中，创建名为 `HelloBlockchain.sol` 的文件。

### <a name="version-pragma"></a>版本杂注

最佳做法是指定 Solidity 目标版本。 指定版本有助于避免与将来的 Solidity 版本不兼容。 

在 `HelloBlockchain.sol` 智能合约代码文件的顶部添加以下版本杂注。


  ``` solidity
  pragma solidity ^0.4.20;
  ```

### <a name="base-class"></a>基类

Blockchain Workbench 可以使用 WorkbenchBase 基类来创建和更新合约。 Blockchain Workbench 特定的智能合约代码需要该基类。 合约需继承自 **WorkbenchBase** 基类。

在 `HelloBlockchain.sol` 智能合约代码文件的开头添加 **WorkbenchBase** 类。 

```
contract WorkbenchBase {
    event WorkbenchContractCreated(string applicationName, string workflowName, address originatingAddress);
    event WorkbenchContractUpdated(string applicationName, string workflowName, string action, address originatingAddress);

    string internal ApplicationName;
    string internal WorkflowName;

    function WorkbenchBase(string applicationName, string workflowName) internal {
        ApplicationName = applicationName;
        WorkflowName = workflowName;
    }

    function ContractCreated() internal {
        WorkbenchContractCreated(ApplicationName, WorkflowName, msg.sender);
    }

    function ContractUpdated(string action) internal {
        WorkbenchContractUpdated(ApplicationName, WorkflowName, action, msg.sender);
    }
}
```
该基类包括两个重要函数：

|基类函数  | 目的  | 何时调用  |
|---------|---------|---------|
| ContractCreated() | 通知 Blockchain Workbench 已创建合约 | 退出合约构造函数之前 |
| ContractUpdated() | 通知 Blockchain Workbench 已更新合约状态 | 退出合约函数之前 |

### <a name="configuration-and-smart-contract-code-relationship"></a>配置与智能合约代码之间的关系

Blockchain Workbench 使用配置文件和智能合约代码文件创建区块链应用程序。 配置中定义的内容与智能合约中的代码之间存在某种关系。 合约详细信息、函数、参数和类型需要匹配才能创建应用程序。 Blockchain Workbench 在创建应用程序之前会验证文件。 

### <a name="contract"></a>合约

对于 Blockchain Workbench 而言，合约需继承自 **WorkbenchBase** 基类。 声明合约时，需将应用程序名称和工作流名称作为参数传递。

将 **contract** 标头添加到 `HelloBlockchain.sol` 智能合约代码文件。 

```
contract HelloBlockchain is WorkbenchBase('HelloBlockchain', 'HelloBlockchain') {
```

合约需继承自 **WorkbenchBase** 基类，并传入配置文件中定义的参数 **ApplicationName** 和工作流 **Name**。 在本例中，应用程序名称和工作流名称是相同的。

### <a name="state-variables"></a>状态变量

状态变量存储每个合约实例的状态值。 合约中的状态变量必须与配置文件中定义的工作流属性匹配。

将状态变量添加到 `HelloBlockchain.sol` 智能合约代码文件中的合约。 

```
    //Set of States
    enum StateType { Request, Respond}
    
    //List of properties
    StateType public  State;
    address public  Requestor;
    address public  Responder;
    
    string public RequestMessage;
    string public ResponseMessage;
```

### <a name="constructor"></a>构造函数

构造函数定义工作流的新智能合约实例的输入参数。 构造函数声明为与合约同名的函数。 构造函数的所需参数定义为配置文件中的构造函数参数。 两个文件中的参数数目、顺序和类型必须匹配。

创建合约之前，请在构造函数中编写想要执行的任何业务逻辑。 例如，使用起始值初始化状态变量。

退出构造函数之前调用 `ContractCreated()` 函数。 此函数通知 Blockchain Workbench 已创建合约。

将构造函数添加到 `HelloBlockchain.sol` 智能合约代码文件中的合约。 

```
    // constructor function
    function HelloBlockchain(string message) public
    {
        Requestor = msg.sender;
        RequestMessage = message;
        State = StateType.Request;
    
        // call ContractCreated() to create an instance of this workflow
        ContractCreated();
    }
```

### <a name="functions"></a>函数

函数是合约中业务逻辑的可执行单元。 函数的所需参数定义为配置文件中的函数参数。 两个文件中的参数数目、顺序和类型必须匹配。 函数关联到配置文件的 Blockchain Workbench 工作流中的转换。 转换是为了转移到合约定义的下一应用程序工作流阶段而执行的操作。

在函数中编写想要执行的任何业务逻辑。 例如，修改状态变量的值。

退出函数之前调用 `ContractUpdated()` 函数。 该函数通知 Blockchain Workbench 已更新合约状态。 如果想要撤消函数中所做的状态更改，请调用 revert()。 Revert 会丢弃自上次调用 ContractUpdated() 以来所做的状态更改。

1. 将以下函数添加到 `HelloBlockchain.sol` 智能合约代码文件中的合约。 

    ```
        // call this function to send a request
        function SendRequest(string requestMessage) public
        {
            if (Requestor != msg.sender)
            {
                revert();
            }
    
            RequestMessage = requestMessage;
            State = StateType.Request;
    
            // call ContractUpdated() to record this action
            ContractUpdated('SendRequest');
        }
    
        // call this function to send a response
        function SendResponse(string responseMessage) public
        {
            Responder = msg.sender;
    
            // call ContractUpdated() to record this action
            ResponseMessage = responseMessage;
            State = StateType.Respond;
            ContractUpdated('SendResponse');
        }
    }
    ```

2. 保存 `HelloBlockchain.sol` 智能合约代码文件。

## <a name="add-blockchain-application-to-blockchain-workbench"></a>将区块链应用程序添加到 Blockchain Workbench

若要将区块链应用程序添加到 Blockchain Workbench，请上传配置和智能合约文件以定义应用程序。

1. 在 Web 浏览器中，导航到 Blockchain Workbench 的 Web 地址。 例如 `https://{workbench URL}.azurewebsites.net/`。该 Web 应用程序是部署 Blockchain Workbench 时创建的。 有关如何查找 Blockchain Workbench Web 地址的信息，请参阅 [Blockchain Workbench Web URL](deploy.md#blockchain-workbench-web-url)
2. 以 [Blockchain Workbench 管理员](manage-users.md#manage-blockchain-workbench-administrators)身份登录。
3. 选择“应用程序” > “新建”。 此时会显示“新建应用程序”窗格。
4. 选择“上传合约配置” > “浏览”，找到创建的 **HelloBlockchain.json** 配置文件。 系统会自动验证该配置文件。 选择“显示”链接以显示验证错误。 请在部署应用程序之前修复验证错误。
5. 选择“上传合约代码” > “浏览”，找到 **HelloBlockchain.sol** 智能合约代码文件。 系统会自动验证该代码文件。 选择“显示”链接以显示验证错误。 请在部署应用程序之前修复验证错误。
6. 选择“部署”，根据配置和智能合约文件创建区块链应用程序。

部署区块链应用程序需要几分钟时间。 完成部署后，新应用程序会显示在“应用程序”中。 

> [!NOTE]
> 也可以使用 [Azure Blockchain Workbench REST API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench) 创建区块链应用程序。 

## <a name="add-blockchain-application-members"></a>添加区块链应用程序成员

将应用程序成员添加到应用程序，以启动合约并对其执行操作。 只有 [Blockchain Workbench 管理员](manage-users.md#manage-blockchain-workbench-administrators)才能添加应用程序成员。

1. 选择“应用程序” > “Hello, Blockchain!”。
2. 页面右上角会显示与应用程序关联的成员数。 对于新应用程序，成员数为零。
3. 选择页面右上角的“成员”链接。 此时会显示应用程序的当前成员列表。
4. 在成员身份列表中，选择“添加成员”。
5. 选择或输入要添加的成员名称。 只会列出 Blockchain Workbench 租户中存在的 Azure AD 用户。 如果找不到用户，则需要[添加 Azure AD 用户](manage-users.md#add-azure-ad-users)。
6. 选择成员的“角色”。 对于第一个成员，请选择“请求方”作为角色。
7. 选择“添加”，将具有关联角色的成员添加到应用程序。
8. 将具有“响应方”角色的另一个成员添加到应用程序。

有关在 Blockchain Workbench 中管理用户的详细信息，请参阅[在 Azure Blockchain Workbench 中管理用户](manage-users.md)

## <a name="next-steps"></a>后续步骤

在本操作指南文章中，已创建了一个基本的请求和响应应用程序。 若要了解如何使用应用程序，请继续学习下一篇操作指南文章。

> [!div class="nextstepaction"]
> [使用区块链应用程序](use.md)
