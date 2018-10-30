---
title: 使用 Azure Blockchain Workbench REST API
description: 关于如何使用 Azure Blockchain Workbench REST API 的方案
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: 2e9124213181fe32f3492e353b05ace89a9d6992
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240693"
---
# <a name="using-the-azure-blockchain-workbench-rest-api"></a>使用 Azure Blockchain Workbench REST API 

开发人员和信息工作者可以使用 Azure Blockchain Workbench REST API 生成丰富的适用于区块链应用程序的集成。 本文档详细介绍 Workbench REST API 的多种主要方法。 假设这样一种场景：开发人员需要创建一个自定义区块链客户端，以便登录用户查看已分配的区块链应用程序并与之交互。 用户可以通过客户端查看合约实例并对智能合约执行操作。 在用户已登录的情况下，客户端会使用 Workbench REST API 执行以下操作：

* 列出应用程序
* 列出应用程序的工作流
* 列出工作流的智能合约实例
* 列出可用于合约的操作
* 执行适用于合约的操作

上述场景中使用的示例区块链应用程序可以[从 GitHub 下载](https://github.com/Azure-Samples/blockchain)。 

## <a name="list-applications"></a>列出应用程序

当用户登录到区块链客户端以后，第一个任务是为该用户检索所有 Blockchain Workbench 应用程序。 在这种情况下，用户可以访问两个应用程序：

1.  [资产转移](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer/readme.md)
2.  [冷藏运输](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/refrigerated-transportation/readme.md)

使用 [Applications GET API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench/applications/applicationsget)：

``` http
GET /api/v1/applications 
Authorization : Bearer {access token}
```

响应会列出可供用户在 Blockchain Workbench 中访问的所有区块链应用程序。 Blockchain Workbench 管理员可以获取所有区块链应用程序，而非 Workbench 管理员则可获取有它们的至少一个相关应用程序角色或相关智能合约实例角色的所有区块链。

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications?skip=2",
    "applications": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
            "displayName": "Asset Transfer",
            "createdByUserId": 1,
            "createdDtTm": "2018-04-28T05:59:14.4733333",
            "enabled": true,
            "applicationRoles": null
        },
        {
            "id": 2,
            "name": "RefrigeratedTransportation",
            "description": "Application to track end-to-end transportation of perishable goods.",
            "displayName": "Refrigerated Transportation",
            "createdByUserId": 7,
            "createdDtTm": "2018-04-28T18:25:38.71",
            "enabled": true,
            "applicationRoles": null
        }
    ]
}
```

## <a name="list-workflows-for-an-application"></a>列出应用程序的工作流

一旦用户选择了适当的区块链应用程序（在本示例中为“资产转移”），区块链客户端就会检索该特定区块链应用程序的所有工作流。 然后，用户可以在工作流的所有智能合约实例显示之前选择适当的工作流。 每个区块链应用程序有一个或多个工作流，每个工作流有零个或多个智能合约实例。 开发区块链客户端应用程序时，建议在只有一个工作流适用于区块链应用程序的情况下跳过允许用户选择适当工作流的用户体验流。 在本示例中，资产转移只有一个工作流，也称为“资产转移”。

使用 [Applications Workflows GET API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench/applications/workflowsget)：

``` http
GET /api/v1/applications/{applicationId}/workflows
Authorization: Bearer {access token}
```

响应会列出可供用户在 Blockchain Workbench 中访问的指定区块链应用程序的所有工作流。 Blockchain Workbench 管理员可以获取所有区块链工作流，而非 Workbench 管理员则可获取有它们的至少一个相关应用程序角色或相关智能合约实例角色的所有工作流。

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications/1/workflows?skip=1",
    "workflows": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Handles the business logic for the asset transfer scenario",
            "displayName": "Asset Transfer",
            "applicationId": 1,
            "constructorId": 1,
            "startStateId": 1
        }
    ]
}
```

## <a name="list-smart-contract-instances-for-a-workflow"></a>列出工作流的智能合约实例

一旦用户选择了适当的工作流（在本示例中为“资产转移”），区块链客户端就会检索指定工作流的所有智能合约实例。 可以根据此信息来显示工作流的所有智能合约实例，以便用户深入了解任何显示的智能合约实例。 在此示例中，设想用户会与某个需执行操作的智能合约实例交互。

使用 [Contracts GET API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench/contracts/contractsget)：

``` http
GET api/v1/contracts?workflowId={workflowId}
Authorization: Bearer {access token}
```

响应会列出指定工作流的所有智能合约实例。 Workbench 管理员可以获取所有智能合约实例，而非 Workbench 管理员则可获取有它们的至少一个相关应用程序角色或相关智能合约实例角色的所有智能合约实例。

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts?skip=3&workflowId=1",
    "contracts": [
        {
            "id": 1,
            "provisioningStatus": 2,
            "connectionID": 1,
            "ledgerIdentifier": "0xbcb6127be062acd37818af290c0e43479a153a1c",
            "deployedByUserId": 1,
            "workflowId": 1,
            "contractCodeId": 1,
            "contractProperties": [
                {
                    "workflowPropertyId": 1,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 2,
                    "value": "My first car"
                },
                {
                    "workflowPropertyId": 3,
                    "value": "54321"
                },
                {
                    "workflowPropertyId": 4,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 5,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 6,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 7,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 8,
                    "value": "0xd882530eb3d6395e697508287900c7679dbe02d7"
                }
            ],
            "transactions": [
                {
                    "id": 1,
                    "connectionId": 1,
                    "transactionHash": "0xf3abb829884dc396e03ae9e115a770b230fcf41bb03d39457201449e077080f4",
                    "blockID": 241,
                    "from": "0xd882530eb3d6395e697508287900c7679dbe02d7",
                    "to": null,
                    "value": 0,
                    "isAppBuilderTx": true
                }
            ],
            "contractActions": [
                {
                    "id": 1,
                    "userId": 1,
                    "provisioningStatus": 2,
                    "timestamp": "2018-04-29T23:41:14.9333333",
                    "parameters": [
                        {
                            "name": "Description",
                            "value": "My first car"
                        },
                        {
                            "name": "Price",
                            "value": "54321"
                        }
                    ],
                    "workflowFunctionId": 1,
                    "transactionId": 1,
                    "workflowStateId": 1
                }
            ]
        }
    ]
}
```

## <a name="list-available-actions-for-a-contract"></a>列出可用于合约的操作

一旦用户决定深入了解某个合约，区块链客户端就会根据合约的状态向该用户显示所有可用的操作。 在此示例中，用户将查看适用于新建智能合约的所有操作：

* 修改：允许用户修改资产的说明和价格。
* 终止：允许用户终止资产的合约。

使用 [Contract Action GET API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench/contracts/contractactionget)：

``` http
GET /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
```

响应会列出用户可以根据指定智能合约实例的当前状态执行的所有操作。 如果用户有一个相关联的应用程序角色，或者与某个智能合约实例角色相关联，则用户可以获取所有适用于指定智能合约实例当前状态的操作。

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts/1/actions?skip=2",
    "workflowFunctions": [
        {
            "id": 2,
            "name": "Modify",
            "description": "Modify the description/price attributes of this asset transfer instance",
            "displayName": "Modify",
            "parameters": [
                {
                    "id": 1,
                    "name": "description",
                    "description": "The new description of the asset",
                    "displayName": "Description",
                    "type": {
                        "id": 2,
                        "name": "string",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                },
                {
                    "id": 2,
                    "name": "price",
                    "description": "The new price of the asset",
                    "displayName": "Price",
                    "type": {
                        "id": 3,
                        "name": "money",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                }
            ],
            "workflowId": 1
        },
        {
            "id": 3,
            "name": "Terminate",
            "description": "Used to cancel this particular instance of asset transfer",
            "displayName": "Terminate",
            "parameters": [],
            "workflowId": 1
        }
    ]
}
```

## <a name="execute-an-action-for-a-contract"></a>执行适用于合约的操作

然后，用户可以根据指定的智能合约实例来决定要采取的操作。 在此示例中，假设这样一种场景：用户要将资产的说明和价格修改如下：

* 说明：“My updated car”
* 价格：54321

使用 [Contract Action POST API](https://docs.microsoft.com/rest/api/azure-blockchain-workbench/contracts/contractactionpost)：

``` http
POST /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
actionInformation: {
    "workflowFunctionId": 2,
    "workflowActionParameters": [
        {
            "name": "description",
            "value": "My updated car"
        },
        {
            "name": "price",
            "value": "54321"
        }
    ]
}
```

用户只能根据指定智能合约实例的当前状态以及用户的关联应用程序角色或智能合约实例角色来执行操作。 如果 POST 成功，则会返回没有响应正文的“HTTP 200 正常”响应。

``` http
HTTP/1.1 200 OK
Content-type: application/json
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench REST API 参考](https://docs.microsoft.com/rest/api/azure-blockchain-workbench)