---
title: Azure Blockchain Workbench 中的数据库视图
description: Azure Blockchain Workbench SQL DB 数据库视图概述。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 62f1cbc4f96b3ef522e0fab5529b8b84cb52d9ef
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240665"
---
# <a name="database-views-in-azure-blockchain-workbench"></a>Azure Blockchain Workbench 中的数据库视图

Azure Blockchain Workbench 可将数据从分布式账本传送到链外 SQL DB 数据库。 这样，我们便可以使用 SQL 和现有工具（例如 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017)）来与区块链数据交互。

Azure Blockchain Workbench 提供一组数据库视图用于访问有助于执行查询的数据。 这些视图经过很大力度的非规范化，因此，可以快速轻松地使用它们开始生成报告、分析，或者结合现有工具使用区块链数据，而无需重新培训数据库工作人员。

本部分概述数据库视图及其包含的数据。

> [!NOTE]
> 不支持在这些视图的外部直接使用数据库中的数据库表（如果存在这种情况）。
> 

## <a name="vwapplication"></a>vwApplication

此视图提供有关已上传到 Azure Blockchain Workbench 的**应用程序**的详细信息。

| 名称                             | Type          | 可为 Null | Description                                                                                                                                                                                                                                                   |
|----------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                    | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                      |
| ApplicationName                  | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                  |
| ApplicationDescription           | nvarchar(255) | 是         | 应用程序的说明                                                                                                                                                                                                                              |
| ApplicationDisplayName           | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                 |
| ApplicationEnabled               | bit           | 否          | 标识当前是否已启用应用程序。</br> **注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。 |
| UploadedDtTm                     | datetime2(7)  | 否          | 上传合约的日期和时间。                                                                                                                                                                                                                    |
| UploadedByUserId                 | int           | 否          | 上传应用程序的用户的 ID。                                                                                                                                                                                                              |
| UploadedByUserExternalId         | nvarchar(255) | 否          | 上传应用程序的用户的外部标识符。 默认情况下，这是用于联盟的 Azure Active Directory 中的用户 ID。                                                                                                |
| UploadedByUserProvisioningStatus | int           | 否          | 标识用户的当前预配过程状态。 可能的值包括</br>0 – API 已创建用户<br>1 – 已将密钥与数据库中的用户相关联</br>2 – 已完全预配用户                         |
| UploadedByUserFirstName          | nvarchar(50)  | 是         | 上传合约的用户的名字。                                                                                                                                                                                                         |
| UploadedByUserLastName           | nvarchar(50)  | 是         | 上传合约的用户的姓氏。                                                                                                                                                                                                          |
| UploadedByUserEmailAddress       | nvarchar(255) | 是         | 上传合约的用户的电子邮件地址。                                                                                                                                                                                                      |

## <a name="vwapplicationrole"></a>vwApplicationRole

此视图提供有关 Azure Blockchain Workbench 应用程序中定义的角色的详细信息。

例如，在“资产转让”应用程序中，可以定义“买方”和“卖方”等角色。

| 名称                   | Type             | 可为 Null | Description                                       |
|------------------------|------------------|-------------|---------------------------------------------------|
| ApplicationId          | int              | 否          | 应用程序的唯一标识符           |
| ApplicationName        | nvarchar(50)     | 否          | 应用程序的名称                       |
| ApplicationDescription | nvarchar(255)    | 是         | 应用程序的说明                  |
| ApplicationDisplayName | nvarchar(255)    | 否          | 在用户界面中显示的名称      |
| RoleId                 | int              | 否          | 应用程序中角色的唯一标识符 |
| RoleName               | nvarchar50)      | 否          | 角色的名称                              |
| RoleDescription        | description(255) | 是         | 角色的说明                         |

## <a name="vwapplicationroleuser"></a>vwApplicationRoleUser

此视图提供有关 Azure Blockchain Workbench 应用程序中定义的角色及其关联的用户的详细信息。

例如，在“资产转让”应用程序中，*John Smith* 可能与“买方”角色相关联。

| 名称                       | Type          | 可为 Null | Description                                                                                                                                                                                                                           |
|----------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId              | int           | 否          | 应用程序的唯一标识符                                                                                                                                                                                               |
| ApplicationName            | nvarchar(50)  | 否          | 应用程序的名称                                                                                                                                                                                                           |
| ApplicationDescription     | nvarchar(255) | 是         | 应用程序的说明                                                                                                                                                                                                      |
| ApplicationDisplayName     | nvarchar(255) | 否          | 在用户界面中显示的名称                                                                                                                                                                                          |
| ApplicationRoleId          | int           | 否          | 应用程序中角色的唯一标识符                                                                                                                                                                                     |
| ApplicationRoleName        | nvarchar50)   | 否          | 角色的名称                                                                                                                                                                                                                  |
| ApplicationRoleDescription | nvarchar(255) | 是         | 角色的说明                                                                                                                                                                                                             |
| UserId                     | int           | 否          | 与角色关联的用户的 ID。                                                                                                                                                                                          |
| UserExternalId             | nvarchar(255) | 否          | 与角色关联的用户的外部标识符。 默认情况下，这是用于联盟的 Azure Active Directory 中的用户 ID。                                                                     |
| UserProvisioningStatus     | int           | 否          | 标识用户的当前预配过程状态。 可能的值包括：</br>0 – API 已创建用户</br>1 – 已将密钥与数据库中的用户相关联<br>2 – 已完全预配用户 |
| UserFirstName              | nvarchar(50)  | 是         | 与角色关联的用户的名字。                                                                                                                                                                           |
| UserLastName               | nvarchar(255) | 是         | 与角色关联的用户的姓氏。                                                                                                                                                                            |
| UserEmailAddress           | nvarchar(255) | 是         | 与角色关联的用户的电子邮件地址。                                                                                                                                                                        |

## <a name="vwconnectionuser"></a>vwConnectionUser

此视图提供有关 Azure Blockchain Workbench 中定义的连接及其关联的用户的详细信息。 对于每个连接，此视图包含以下数据：

-   关联的账本详细信息
-   关联的用户信息

| 名称                     | Type          | 可为 Null | Description                                                                                                                                                                                                                           |
|--------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId             | int           | 否          | Azure Blockchain Workbench 中的连接的唯一标识符。                                                                                                                                                                 |
| ConnectionEndpointUrl    | nvarchar(50)  | 否          | 连接的终结点 URL。                                                                                                                                                                                                    |
| ConnectionFundingAccount | nvarchar(255) | 是         | 与连接关联的投资帐户（如果适用）。                                                                                                                                                                      |
| LedgerId                 | int           | 否          | 账本的唯一标识符。                                                                                                                                                                                                   |
| LedgerName               | nvarchar(50)  | 否          | 账本的名称。                                                                                                                                                                                                               |
| LedgerDisplayName        | nvarchar(255) | 否          | 在 UI 中显示的账本名称。                                                                                                                                                                                          |
| UserId                   | int           | 否          | 与连接关联的用户的 ID。                                                                                                                                                                                    |
| UserExternalId           | nvarchar(255) | 否          | 与连接关联的用户的外部标识符。 默认情况下，这是用于联盟的 Azure Active Directory 中的用户 ID。                                                               |
| UserProvisioningStatus   | int           | 否          |标识用户的当前预配过程状态。 可能的值包括：</br>0 – API 已创建用户</br>1 – 已将密钥与数据库中的用户相关联<br>2 – 已完全预配用户 |
| UserFirstName            | nvarchar(50)  | 是         | 与连接关联的用户的名字。                                                                                                                                                                     |
| UserLastName             | nvarchar(255) | 是         | 与连接关联的用户的姓氏。                                                                                                                                                                      |
| UserEmailAddress         | nvarchar(255) | 是         | 与连接关联的用户的电子邮件地址。                                                                                                                                                                  |

## <a name="vwcontract"></a>vwContract

此视图提供有关已部署的合约的详细信息。 对于每个合约，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   关联的函数账本实现
-   启动操作的用户的详细信息
-   区块链块和事务相关的详细信息

| 名称                                     | Type           | 可为 Null | Description                                                                                                                                                                                                                                                   |
|------------------------------------------|----------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId                             | int            | 否          | Azure Blockchain Workbench 中的连接的唯一标识符。                                                                                                                                                                                         |
| ConnectionEndpointUrl                    | nvarchar(50)   | 否          | 连接的终结点 URL。                                                                                                                                                                                                                            |
| ConnectionFundingAccount                 | nvarchar(255)  | 是         | 与连接关联的投资帐户（如果适用）。                                                                                                                                                                                              |
| LedgerId                                 | int            | 否          | 账本的唯一标识符。                                                                                                                                                                                                                           |
| LedgerName                               | nvarchar(50)   | 否          | 账本的名称。                                                                                                                                                                                                                                       |
| LedgerDisplayName                        | nvarchar(255)  | 否          | 在 UI 中显示的账本名称。                                                                                                                                                                                                                  |
| ApplicationId                            | int            | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                      |
| ApplicationName                          | nvarchar (50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                  |
| ApplicationDisplayName                   | nvarchar (255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                 |
| ApplicationEnabled                       | bit            | 否          | 标识当前是否已启用应用程序。</br> **注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。  |
| WorkflowId                               | int            | 否          | 与合约关联的工作流的唯一标识符。                                                                                                                                                                                              |
| WorkflowName                             | nvarchar(50)   | 否          | 与合约关联的工作流的名称。                                                                                                                                                                                                          |
| WorkflowDisplayName                      | nvarchar(255)  | 否          | 在用户界面中显示的、与合约关联的工作流的名称。                                                                                                                                                                     |
| WorkflowDescription                      | nvarchar(255)  | 是         | 与合约关联的工作流的说明。                                                                                                                                                                                                   |
| ContractCodeId                           | int            | 否          | 与合约关联的合约代码的唯一标识符。                                                                                                                                                                                       |
| ContractFileName                         | int            | 否          | 包含此工作流的智能合约代码的文件的名称。                                                                                                                                                                                    |
| ContractUploadedDtTm                     | int            | 否          | 上传合约代码的日期和时间。                                                                                                                                                                                                             |
| ContractId                               | int            | 否          | 合约的唯一标识符。                                                                                                                                                                                                                       |
| ContractProvisioningStatus               | int            | 否          | 部署合约的用户的姓氏。                                                                                                                                                                                                          |
| ContractLedgerIdentifier                 | nvarchar (255) |             | 部署合约的用户的电子邮件地址。                                                                                                                                                                                                      |
| ContractDeployedByUserId                 | int            | 否          | 部署合约的用户的外部标识符。 默认情况下，这是表示用户 Azure Active Directory ID 的 GUID。                                                                                                          |
| ContractDeployedByUserExternalId         | nvarchar(255)  | 否          | 部署合约的用户的外部标识符。 默认情况下，这是表示用户 Azure Active Directory ID 的 GUID。                                                                                                         |
| ContractDeployedByUserProvisioningStatus | int            | 否          | 标识用户的当前预配过程状态。 可能的值包括：</br>0 – API 已创建用户</br>1 – 已将密钥与数据库中的用户相关联 </br>2 – 已完全预配用户                     |
| ContractDeployedByUserFirstName          | nvarchar(50)   | 是         | 部署合约的用户的名字。                                                                                                                                                                                                         |
| ContractDeployedByUserLastName           | nvarchar(255)  | 是         | 部署合约的用户的姓氏。                                                                                                                                                                                                          |
| ContractDeployedByUserEmailAddress       | nvarchar(255)  | 是         | 部署合约的用户的电子邮件地址。                                                                                                                                                                                                      |

## <a name="vwcontractaction"></a>vwContractAction

此视图呈现针对合约执行的操作相关的大部分信息，方便促成常见的报告方案。 对于执行的每项操作，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   关联的智能合约函数和参数定义
-   关联的函数账本实现
-   为参数提供的特定实例值
-   启动操作的用户的详细信息
-   区块链块和事务相关的详细信息

| 名称                                     | Type          | 可为 Null | Description                                                                                                                                                                                                                                                                                                    |
|------------------------------------------|---------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                            | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                                                                       |
| ApplicationName                          | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                                                                   |
| ApplicationDisplayName                   | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                                                                  |
| ApplicationEnabled                       | bit           | 否          | 此字段标识当前是否已启用应用程序。 注意 – 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。                                                  |
| WorkflowId                               | int           | 否          | 工作流的唯一标识符。                                                                                                                                                                                                                                                                            |
| WorkflowName                             | nvarchar(50)  | 否          | 工作流的名称。                                                                                                                                                                                                                                                                                      |
| WorkflowDisplayName                      | nvarchar(255) | 否          | 在用户界面中显示的工作流名称。                                                                                                                                                                                                                                                       |
| WorkflowDescription                      | nvarchar(255) | 是         | 工作流的说明。                                                                                                                                                                                                                                                                               |
| ContractId                               | int           | 否          | 合约的唯一标识符。                                                                                                                                                                                                                                                                          |
| ContractProvisioningStatus               | int           | 否          | 标识合约的当前预配过程状态。 可能的值包括：</br>0 – API 已在数据库中创建合约</br>1 – 合约已发送到账本</br>2 – 合约已成功部署到账本                             |
| ContractCodeId                           | int           | 否          | 合约代码实现的唯一标识符。                                                                                                                                                                                                                                               |
| ContractLedgerIdentifier                 | nvarchar(255) | 是         | 与特定分布式账本的已部署智能合约版本关联的唯一标识符。 例如 Ethereum。                                                                                                                                                                                 |
| ContractDeployedByUserId                 | int           | 否          | 部署合约的用户的唯一标识符。                                                                                                                                                                                                                                                  |
| ContractDeployedByUserFirstName          | nvarchar(50)  | 是         | 部署合约的用户的名字。                                                                                                                                                                                                                                                              |
| ContractDeployedByUserLastName           | nvarchar(255) | 是         | 部署合约的用户的姓氏。                                                                                                                                                                                                                                                               |
| ContractDeployedByUserExternalId         | nvarchar(255) | 否          | 部署合约的用户的外部标识符。 默认情况下，这是表示用户在联盟 Azure Active Directory 中的标识的 GUID。                                                                                                                                                |
| ContractDeployedByUserEmailAddress       | nvarchar(255) | 是         | 部署合约的用户的电子邮件地址。                                                                                                                                                                                                                                                       |
| WorkflowFunctionId                       | int           | 否          | 工作流函数的唯一标识符。                                                                                                                                                                                                                                                                   |
| WorkflowFunctionName                     | nvarchar(50)  | 否          | 函数的名称。                                                                                                                                                                                                                                                                                      |
| WorkflowFunctionDisplayName              | nvarchar(255) | 否          | 在用户界面中显示的函数名称。                                                                                                                                                                                                                                                  |
| WorkflowFunctionDescription              | nvarchar(255) | 否          | 函数的说明。                                                                                                                                                                                                                                                                               |
| ContractActionId                         | int           | 否          | 合约操作的唯一标识符。                                                                                                                                                                                                                                                                   |
| ContractActionProvisioningStatus         | int           | 否          | 标识合约操作的当前预配过程状态。 可能的值包括：</br>0 – API 已在数据库中创建合约操作</br>1 – 合约操作已发送到账本</br>2 – 合约操作已成功部署到账本 |
| ContractActionTimestamp                  | datetime(2,7) | 否          | 合约操作的时间戳。                                                                                                                                                                                                                                                                          |
| ContractActionExecutedByUserId           | int           | 否          | 执行合约操作的用户的唯一标识符。                                                                                                                                                                                                                                               |
| ContractActionExecutedByUserFirstName    | int           | 是         | 执行合约操作的用户的名字。                                                                                                                                                                                                                                                       |
| ContractActionExecutedByUserLastName     | nvarchar(50)  | 是         | 执行合约操作的用户的姓氏。                                                                                                                                                                                                                                                        |
| ContractActionExecutedByUserExternalId   | nvarchar(255) | 是         | 执行合约操作的用户的外部标识符。  默认情况下，这是表示用户在联盟 Azure Active Directory 中的标识的 GUID                                                                                                                                                                                                            |
| ContractActionExecutedByUserEmailAddress | nvarchar(255) | 是         | 执行合约操作的用户的电子邮件地址。                                                                                                                                                                                                                                                |
| WorkflowFunctionParameterId              | int           | 否          | 函数的参数的唯一标识符。                                                                                                                                                                                                                                                           |
| WorkflowFunctionParameterName            | nvarchar(50)  | 否          | 函数的参数的名称。                                                                                                                                                                                                                                                                       |
| WorkflowFunctionParameterDisplayName     | nvarchar(255) | 否          | 在用户界面中显示的函数参数名称。                                                                                                                                                                                                                                        |
| WorkflowFunctionParameterDataTypeId      | int           | 否          | 与工作流函数参数关联的数据类型的唯一标识符。                                                                                                                                                                                                                         |
| WorkflowParameterDataTypeName            | nvarchar(50)  | 否          | 与工作流函数参数关联的数据类型的名称。                                                                                                                                                                                                                                         |
| ContractActionParameterValue             | nvarchar(255) | 否          | 存储在智能合约中的参数值。                                                                                                                                                                                                                                                      |
| BlockHash                                | nvarchar(255) | 是         | 块的哈希。                                                                                                                                                                                                                                                                                         |
| BlockNumber                              | int           | 是         | 账本中的块数。                                                                                                                                                                                                                                                                         |
| BlockTimestamp                           | datetime(2,7) | 是         | 块的时间戳。                                                                                                                                                                                                                                                                                   |
| TransactionId                            | int           | 否          | 事务的唯一标识符。                                                                                                                                                                                                                                                                       |
| TransactionFrom                          | nvarchar(255) | 是         | 发起事务的一方。                                                                                                                                                                                                                                                                     |
| TransactionTo                            | nvarchar(255) | 是         | 事务的另一参与方。                                                                                                                                                                                                                                                                            |
| TransactionHash                          | nvarchar(255) | 是         | 事务的哈希。                                                                                                                                                                                                                                                                                     |
| TransactionIsWorkbenchTransaction        | bit           | 是         | 一个标识事务是否为 Azure Blockchain Workbench 事务的位。                                                                                                                                                                                                                          |
| TransactionProvisioningStatus            | int           | 是         | 标识事务的当前预配过程状态。 可能的值包括：</br>0 – API 已在数据库中创建事务</br>1 – 事务已发送到账本</br>2 – 事务已成功部署到账本                 |
| TransactionValue                         | decimal(32,2) | 是         | 事务的值。                                                                                                                                                                                                                                                                                  |

## <a name="vwcontractproperty"></a>vwContractProperty

此视图呈现与合约关联的属性相关的大部分信息，方便促成常见的报告方案。 对于采用的每个属性，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   部署工作流的用户的用户详细信息
-   关联的智能合约属性定义
-   属性的特定实例值
-   合约的状态属性的详细信息

| 名称                               | Type          | 可为 Null | Description                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                                           |
| ApplicationName                    | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                                       |
| ApplicationDisplayName             | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                                      |
| ApplicationEnabled                 | bit           | 否          | 标识当前是否已启用应用程序。</br>**注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。                      |
| WorkflowId                         | int           | 否          | 工作流的唯一标识符。                                                                                                                                                                                                                                            |
| WorkflowName                       | nvarchar(50)  | 否          | 工作流的名称。                                                                                                                                                                                                                                                          |
| WorkflowDisplayName                | nvarchar(255) | 否          | 在用户界面中显示的工作流名称。                                                                                                                                                                                                                       |
| WorkflowDescription                | nvarchar(255) | 是         | 工作流的说明。                                                                                                                                                                                                                                                   |
| ContractId                         | int           | 否          | 合约的唯一标识符。                                                                                                                                                                                                                                            |
| ContractProvisioningStatus         | int           | 否          | 标识合约的当前预配过程状态。 可能的值包括：</br>0 – API 已在数据库中创建合约</br>1 – 合约已发送到账本</br>2 – 合约已成功部署到账本 |
| ContractCodeId                     | int           | 否          | 合约代码实现的唯一标识符。                                                                                                                                                                                                                   |
| ContractLedgerIdentifier           | nvarchar(255) | 是         | 与特定分布式账本的已部署智能合约版本关联的唯一标识符。 例如 Ethereum。                                                                                                                                                     |
| ContractDeployedByUserId           | int           | 否          | 部署合约的用户的唯一标识符。                                                                                                                                                                                                                      |
| ContractDeployedByUserFirstName    | nvarchar(50)  | 是         | 部署合约的用户的名字。                                                                                                                                                                                                                                  |
| ContractDeployedByUserLastName     | nvarchar(255) | 是         | 部署合约的用户的姓氏。                                                                                                                                                                                                                                   |
| ContractDeployedByUserExternalId   | nvarchar(255) | 否          | 部署合约的用户的外部标识符。 默认情况下，这是表示用户在联盟 Azure Active Directory 中的标识的 GUID                                                                                                                     |
| ContractDeployedByUserEmailAddress | nvarchar(255) | 是         | 部署合约的用户的电子邮件地址。                                                                                                                                                                                                                           |
| WorkflowPropertyId                 | int           |             | 工作流的属性的唯一标识符。                                                                                                                                                                                                                                  |
| WorkflowPropertyDataTypeId         | int           | 否          | 属性的数据类型的 ID。                                                                                                                                                                                                                                           |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | 否          | 属性的数据类型的名称。                                                                                                                                                                                                                                         |
| WorkflowPropertyName               | nvarchar(50)  | 否          | 工作流属性的名称。                                                                                                                                                                                                                                                 |
| WorkflowPropertyDisplayName        | nvarchar(255) | 否          | 工作流属性的显示名称。                                                                                                                                                                                                                                         |
| WorkflowPropertyDescription        | nvarchar(255) | 是         | 属性的说明。                                                                                                                                                                                                                                                     |
| ContractPropertyValue              | nvarchar(255) | 否          | 合约中属性的值。                                                                                                                                                                                                                                          |
| StateName                          | nvarchar(50)  | 是         | 如果此属性包含合约状态，则此值为此状态的显示名称。 如果此属性不与状态相关联，则值为 null。                                                                                                                                                                                                                    |
| StateDisplayName                   | nvarchar(255) | 否          | 如果此属性包含状态，则此值为此状态的显示名称。 如果此属性不与状态相关联，则值为 null。  |
| StateValue                         | nvarchar(255) | 是         | 如果此属性包含状态，则此值为状态值。 如果此属性不与状态相关联，则值为 null。                          |

## <a name="vwcontractstate"></a>vwContractState

此视图呈现与特定合约状态相关的大部分信息，方便促成常见的报告方案。 此视图中的每条记录包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   部署工作流的用户的用户详细信息
-   关联的智能合约属性定义
-   合约的状态属性的详细信息

| 名称                               | Type          | 可为 Null | Description                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                                           |
| ApplicationName                    | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                                       |
| ApplicationDisplayName             | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                                      |
| ApplicationEnabled                 | bit           | 否          | 标识当前是否已启用应用程序。</br>**注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。                     |
| WorkflowId                         | int           | 否          | 工作流的唯一标识符。                                                                                                                                                                                                                                              |
| WorkflowName                       | nvarchar(50)  | 否          | 工作流的名称。                                                                                                                                                                                                                                                          |
| WorkflowDisplayName                | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                                       |
| WorkflowDescription                | nvarchar(255) | 是         | 工作流的说明。                                                                                                                                                                                                                                                   |
| ContractLedgerImplementationId     | nvarchar(255) | 是         | 与特定分布式账本的已部署智能合约版本关联的唯一标识符。 例如 Ethereum。                                                                                                                                                     |
| ContractId                         | int           | 否          | 合约的唯一标识符。                                                                                                                                                                                                                                              |
| ContractProvisioningStatus         | int           | 否          |标识合约的当前预配过程状态。 可能的值包括：</br>0 – API 已在数据库中创建合约</br>1 – 合约已发送到账本</br>2 – 合约已成功部署到账本 |
| ConnectionId                       | int           | 否          | 工作流部署到的区块链实例的唯一标识符                                                                                                                                                                                                        |
| ContractCodeId                     | int           | 否          | 合约代码实现的唯一标识符。                                                                                                                                                                                                                   |
| ContractDeployedByUserId           | int           | 否          | 部署合约的用户的唯一标识符。                                                                                                                                                                                                                          |
| ContractDeployedByUserExternalId   | nvarchar(255) | 否          | 部署合约的用户的外部标识符。 默认情况下，这是表示用户在联盟 Azure Active Directory 中的标识的 GUID。                                                                                                                    |
| ContractDeployedByUserFirstName    | nvarchar(50)  | 是         | 部署合约的用户的名字。                                                                                                                                                                                                                                  |
| ContractDeployedByUserLastName     | nvarchar(255) | 是         | 部署合约的用户的姓氏。                                                                                                                                                                                                                                   |
| ContractDeployedByUserEmailAddress | nvarchar(255) | 是         | 部署合约的用户的电子邮件地址。                                                                                                                                                                                                                           |
| WorkflowPropertyId                 | int           | 否          | 工作流属性的唯一标识符。                                                                                                                                                                                                                                       |
| WorkflowPropertyDataTypeId         | int           | 否          | 工作流属性的数据类型的 ID。                                                                                                                                                                                                                                  |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | 否          | 工作流属性的数据类型的名称。                                                                                                                                                                                                                                |
| WorkflowPropertyName               | nvarchar(50)  | 否          | 工作流属性的名称。                                                                                                                                                                                                                                                 |
| WorkflowPropertyDisplayName        | nvarchar(255) | 否          | 在 UI 中显示的属性名称。                                                                                                                                                                                                                                  |
| WorkflowPropertyDescription        | nvarchar(255) | 是         | 属性的说明。                                                                                                                                                                                                                                                   |
| ContractPropertyValue              | nvarchar(255) | 否          | 合约中存储的属性值。                                                                                                                                                                                                                                   |
| StateName                          | nvarchar(50)  | 是         | 如果此属性包含状态，则此值为此状态的显示名称。 如果此属性不与状态相关联，则值为 null。                                                                                                                                       |
| StateDisplayName                   | nvarchar(255) | 否          | 如果此属性包含状态，则此值为此状态的显示名称。 如果此属性不与状态相关联，则值为 null。                                                                                                                                       |
| StateValue                         | nvarchar(255) | 是         | 如果此属性包含状态，则此值为状态值。 如果此属性不与状态相关联，则值为 null。                                                                                                                                                      |

## <a name="vwuser"></a>vwUser

此视图提供有关预配为使用 Azure Blockchain Workbench 的联盟成员的详细信息。 默认情况下，会通过用户的初始预配填充数据。

| 名称               | Type          | 可为 Null | Description                                                                                                                                                                                                                               |
|--------------------|---------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID                 | int           | 否          | 用户的唯一标识符。                                                                                                                                                                                                           |
| ExternalID         | nvarchar(255) | 否          | 用户的外部标识符。 默认情况下，这是表示用户 Azure Active Directory ID 的 GUID。                                                                                                                  |
| ProvisioningStatus | int           | 否          |标识用户的当前预配过程状态。 可能的值包括：</br>0 – API 已创建用户</br>1 – 已将密钥与数据库中的用户相关联<br>2 – 已完全预配用户 |
| FirstName          | nvarchar(50)  | 是         | 用户的名字。                                                                                                                                                                                                               |
| LastName           | nvarchar(50)  | 是         | 用户的姓氏。                                                                                                                                                                                                                |
| EmailAddress       | nvarchar(255) | 是         | 用户的电子邮件地址。                                                                                                                                                                                                            |

## <a name="vwworkflow"></a>vwWorkflow

此视图呈现有关核心工作流元数据以及工作流函数和参数的详细信息。 此视图旨在用于报告，其中还包含有关与工作流关联的应用程序的元数据。 此视图包含来自多个基础表的数据，以促成工作流报告。 对于每个工作流，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   关联的工作流启动状态信息

| 名称                              | Type          | 可为 Null | Description                                                                                                                                |
|-----------------------------------|---------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                     | int           | 否          | 应用程序的唯一标识符                                                                                                    |
| ApplicationName                   | nvarchar(50)  | 否          | 应用程序的名称                                                                                                                |
| ApplicationDisplayName            | nvarchar(255) | 否          | 在用户界面中显示的名称                                                                                               |
| ApplicationEnabled                | bit           | 否          | 标识是否已启用应用程序                                                                                                   |
| WorkflowId                        | int           | 是         | 工作流的唯一标识符                                                                                                         |
| WorkflowName                      | nvarchar(50)  | 否          | 工作流的名称                                                                                                                   |
| WorkflowDisplayName               | nvarchar(255) | 否          | 在用户界面中显示的名称                                                                                                |
| WorkflowDescription               | nvarchar(255) | 是         | 工作流的说明。                                                                                                           |
| WorkflowConstructorFunctionId     | int           | 否          | 充当工作流构造函数的工作流函数的标识符。                                                   |
| WorkflowStartStateId              | int           | 否          | 状态的唯一标识符。                                                                                                         |
| WorkflowStartStateName            | nvarchar(50)  | 否          | 状态的名称。                                                                                                                     |
| WorkflowStartStateDisplayName     | nvarchar(255) | 否          | 在用户界面中显示的状态名称。                                                                              |
| WorkflowStartStateDescription     | nvarchar(255) | 是         | 工作流状态的说明。                                                                                                       |
| WorkflowStartStateStyle           | nvarchar(50)  | 是         | 此值标识处于此状态的工作流的完成百分比。                                                     |
| WorkflowStartStateValue           | int           | 否          | 这是状态值。                                                                                                            |
| WorkflowStartStatePercentComplete | int           | 否          | 一段文本说明，提示客户端如何在 UI 中呈现此状态。 支持的状态包括 *Success* 和 *Failure* |

## <a name="vwworkflowfunction"></a>vwWorkflowFunction

此视图呈现有关核心工作流元数据以及工作流函数和参数的详细信息。 此视图旨在用于报告，其中还包含有关与工作流关联的应用程序的元数据。 此视图包含来自多个基础表的数据，以促成工作流报告。  对于每个工作流函数，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   工作流函数详细信息

| 名称                                 | Type          | 可为 Null | Description                                                                          |
|--------------------------------------|---------------|-------------|--------------------------------------------------------------------------------------|
| ApplicationId                        | int           | 否          | 应用程序的唯一标识符                                              |
| ApplicationName                      | nvarchar(50)  | 否          | 应用程序的名称                                                          |
| ApplicationDisplayName               | nvarchar(255) | 否          | 在用户界面中显示的名称                                         |
| ApplicationEnabled                   | bit           | 否          | 标识是否已启用应用程序                                             |
| WorkflowId                           | int           | 否          | 工作流的唯一标识符。                                                  |
| WorkflowName                         | nvarchar(50)  | 否          | 工作流的名称。                                                            |
| WorkflowDisplayName                  | nvarchar(255) | 否          | 在用户界面中显示的工作流名称。                         |
| WorkflowDescription                  | nvarchar(255) | 是         | 工作流的说明。                                                     |
| WorkflowFunctionId                   | int           | 否          | 函数的唯一标识符。                                                  |
| WorkflowFunctionName                 | nvarchar(50)  | 是         | 函数的名称。                                                            |
| WorkflowFunctionDisplayName          | nvarchar(255) | 否          | 在用户界面中显示的函数名称                         |
| WorkflowFunctionDescription          | nvarchar(255) | 是         | 工作流函数的说明。                                            |
| WorkflowFunctionIsConstructor        | bit           | 否          | 此值标识工作流函数是否为工作流的构造函数。        |
| WorkflowFunctionParameterId          | int           | 否          | 函数的参数的唯一标识符。                                   |
| WorkflowFunctionParameterName        | nvarchar(50)  | 否          | 函数的参数的名称。                                             |
| WorkflowFunctionParameterDisplayName | nvarchar(255) | 否          | 在用户界面中显示的函数参数名称               |
| WorkflowFunctionParameterDataTypeId  | int           | 否          | 与工作流函数参数关联的数据类型的唯一标识符。 |
| WorkflowParameterDataTypeName        | nvarchar(50)  | 否          | 与工作流函数参数关联的数据类型的名称。               |

## <a name="vwworkflowproperty"></a>vwWorkflowProperty

此视图呈现针对工作流定义的属性。 对于每个属性，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   工作流属性详细信息

| 名称                         | Type          | 可为 Null | Description                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                      |
| ApplicationName              | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                  |
| ApplicationDisplayName       | nvarchar(255) | 否          | 在用户界面中显示的名称。                                                                                                                                                                                                                 |
| ApplicationEnabled           | bit           | 否          | 标识当前是否已启用应用程序。</br>**注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。 |
| WorkflowId                   | int           | 否          | 工作流的唯一标识符。                                                                                                                                                                                                                         |
| WorkflowName                 | nvarchar(50)  | 否          | 工作流的名称。                                                                                                                                                                                                                                     |
| WorkflowDisplayName          | nvarchar(255) | 否          | 在用户界面中显示的工作流名称。                                                                                                                                                                                                |
| WorkflowDescription          | nvarchar(255) | 是         | 工作流的说明。                                                                                                                                                                                                                                |
| WorkflowPropertyID           | int           | 否          | 工作流的属性的唯一标识符。 |
| WorkflowPropertyName         | nvarchar(50)  | 否          | 属性的名称                                                                                                                                                                                                                                      |
| WorkflowPropertyDescription  | nvarchar(255) | 是         | 属性的说明                                                                                                                                                                                                                                 |
| WorkflowPropertyDisplayName  | nvarchar(255) | 否          | 在用户界面中显示的名称                                                                                                                                                                                                                  |
| WorkflowPropertyWorkflowId   | int           | 否          | 与此属性关联的工作流的 ID                                                                                                                                                                                                   |
| WorkflowPropertyDataTypeId   | int           | 否          | 为属性定义的数据类型的 ID                                                                                                                                                                                                              |
| WorkflowPropertyDataTypeName | nvarchar(50)  | 否          | 为属性定义的数据类型的名称                                                                                                                                                                                                            |
| WorkflowPropertyIsState      | bit           | 否          | 此字段标识此工作流属性是否包含工作流的状态。                                                                                                                                                                           |

## <a name="vwworkflowstate"></a>vwWorkflowState

此视图呈现与工作流关联的属性。 对于每个合约，此视图包含以下数据：

-   关联的应用程序定义
-   关联的工作流定义
-   工作流状态信息

| 名称                         | Type          | 可为 Null | Description                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | 否          | 应用程序的唯一标识符。                                                                                                                                                                                                                      |
| ApplicationName              | nvarchar(50)  | 否          | 应用程序的名称。                                                                                                                                                                                                                                  |
| ApplicationDisplayName       | nvarchar(255) | 否          | 应用程序的说明                                                                                                                                                                                                                              |
| ApplicationEnabled           | bit           | 否          | 标识当前是否已启用应用程序。</br>**注意：** 即使可以在数据库中将某个应用程序表示为已禁用，但关联的合约会保留在区块链中，并且有关这些合约的数据会保留在数据库中。 |
| WorkflowId                   | int           | 否          | 工作流的唯一标识符。                                                                                                                                                                                                                       |
| WorkflowName                 | nvarchar(50)  | 否          | 工作流的名称。                                                                                                                                                                                                                                     |
| WorkflowDisplayName          | nvarchar(255) | 否          | 在用户界面中显示的工作流名称。                                                                                                                                                                                                 |
| WorkflowDescription          | nvarchar(255) | 是         | 工作流的说明。                                                                                                                                                                                                                              |
| WorkflowStateID              | int           | 否          | 状态的唯一标识符。                                                                                                                                                                                                                          |
| WorkflowStateName            | nvarchar(50)  | 否          | 状态的名称。                                                                                                                                                                                                                                        |
| WorkflowStateDisplayName     | nvarchar(255) | 否          | 在用户界面中显示的状态名称。                                                                                                                                                                                                 |
| WorkflowStateDescription     | nvarchar(255) | 是         | 工作流状态的说明。                                                                                                                                                                                                                          |
| WorkflowStatePercentComplete | int           | 否          | 此值标识处于此状态的工作流的完成百分比。                                                                                                                                                                        |
| WorkflowStateValue           | nvarchar(50)  | 否          | 这是状态值。                                                                                                                                                                                                                               |
| WorkflowStateStyle           | nvarchar(50)  | 否          | 一段文本说明，提示客户端如何在 UI 中呈现此状态。 支持的状态包括 *Success* 和 *Failure*                                                                                                                    |
