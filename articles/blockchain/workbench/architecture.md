---
title: Azure Blockchain Workbench 体系结构
description: Azure Blockchain Workbench 体系结构及其组件的概述。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: 8dc8304e97ff66356a8874a146231f0042e5f509
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510146"
---
# <a name="azure-blockchain-workbench-architecture"></a>Azure Blockchain Workbench 体系结构

Azure Blockchain Workbench 使用多个 Azure 组件提供解决方案，从而简化区块链应用程序的开发。 可以使用 Azure 市场中的解决方案模板部署 Blockchain Workbench。 该模板可用于选择要部署的模块和组件，包括区块链堆栈、客户端应用程序类型，以及对 IoT 集成的支持。 部署后，Blockchain Workbench 会提供对 Web 应用、iOS 应用和 Android 应用的访问权限。

![Blockchain Workbench 体系结构](./media/architecture/architecture.png)

## <a name="identity-and-authentication"></a>标识和身份验证

使用 Blockchain Workbench，联盟可以通过 Azure Active Directory (Azure AD) 联合其企业标识。 Workbench 使用 Azure AD 中存储的企业标识为链中标识生成新用户帐户。 标识映射简化了客户端 API 和应用程序的身份验证登录，并使用组织的身份验证策略。 Workbench 还提供相应的功能用于将企业标识关联到给定智能合同中的特定角色。 此外，Workbench 还提供一个机制用于标识这些角色可以执行的操作，以及何时可以执行这些操作。

部署 Blockchain Workbench 后，用户可以通过客户端应用程序、基于 REST 的客户端 API 或消息传递 API 来与 Blockchain Workbench 交互。 在任何情况下，都必须通过 Azure Active Directory (Azure AD) 或特定于设备的凭据对交互进行身份验证。

用户通过向参与者的电子邮件地址发送电子邮件邀请，将其自己的标识联合到联盟 Azure AD。 用户登录时，系统会使用名称、密码和策略对这些用户进行身份验证。 例如，使用其组织的双重身份验证。

Azure AD 用于管理有权访问 Blockchain Workbench 的所有用户。 连接到智能合同的每台设备也与 Azure AD 相关联。

Azure AD 还用于将用户分配到特殊的管理员组。 与管理员组关联的用户将被授予对 Blockchain Workbench 中权限和操作的访问权限，包括部署合同以及向用户授予访问合同的权限。 此组外部的用户无权访问管理员操作。

## <a name="client-applications"></a>客户端应用程序

Workbench 为可用于验证、测试和查看区块链应用程序的 Web 和移动应用（iOS、Android）提供自动生成的客户端应用程序。 应用程序接口根据智能合同元数据动态生成，并可适应任何用例。 客户端应用程序向 Blockchain Workbench 生成的完整区块链应用程序提供面向用户的前端。 客户端应用程序通过 Azure Active Directory (Azure AD) 对用户进行身份验证，然后提供根据智能合同业务上下文定制的用户体验。 用户体验可让获得授权的个人创建新的智能合同实例，然后提供所需的功能用于在智能合同表示的业务流程中的相应阶段执行特定类型的事务。

在 Web 应用程序中，获得授权的用户可以访问管理员控制台。 控制台可供 Azure AD 管理员组中的用户使用，并提供以下功能的访问权限：

* 为常见方案（例如资产转让方案）部署 Microsoft 提供的 智能合同。
* 上传并部署用户自己的智能合约。
* 在特定角色的上下文中为用户分配智能合约的访问权限。

有关详细信息，请参阅 [GitHub 上的 Azure Blockchain Workbench 示例客户端应用程序](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-development-kit/connect/mobile/blockchain-workbench/workbench-client)。

## <a name="gateway-service-api"></a>网关服务 API

Blockchain Workbench 包括基于 REST 的网关服务 API。 写入区块链时，该 API 会生成消息并将其传送到事件中转站。 当 API 请求数据时，会向链外 SQL 数据库发送查询。 SQL 数据库包含链中数据和元数据的副本，这些数据提供受支持智能合同的上下文和配置信息。 查询以合同元数据指定的格式从链外副本返回所需的数据。

开发人员可以访问网关服务 API 来生成或集成区块链解决方案，而无需依赖于 Blockchain Workbench 客户端应用。

> [!NOTE]
> 若要启用 API 的身份验证访问，必须在 Azure Active Directory 中注册两个客户端应用程序。 Azure Active Directory 要求对每种应用程序类型（本机和 Web）使用不同的应用程序注册。 

## <a name="message-broker-for-incoming-messages"></a>传入消息的消息中转站

想要直接向 Blockchain Workbench 发送消息的开发人员可将消息直接发送到服务总线。 例如，可对系统到系统的集成或 IoT 设备使用消息 API。

## <a name="message-broker-for-downstream-consumers"></a>下游使用者的消息中转站

在应用程序的生命周期内会发生事件。 事件可由网关 API 触发，或在账本中触发。 事件通知可以根据事件启动下游代码。

Blockchain Workbench 自动部署两种类型的事件使用者。 一种使用者由区块链事件触发来填充链外 SQL 存储。 另一种使用者用于捕获 API 生成的与文档上传和存储相关的事件元数据。

## <a name="message-consumers"></a>消息使用者

 消息使用者从服务总线提取消息。 消息使用者的底层事件模型允许其他服务和系统的扩展。 例如，可以添加 CosmosDB 填充支持，或使用 Azure 流分析评估消息。 下列部分介绍 Blockchain Workbench 中包含的消息使用者。

### <a name="distributed-ledger-consumer"></a>分布式账本使用者

分布式账本技术 (DLT) 消息包含要写入区块链的事务的元数据。 使用者检索消息，并将数据推送到事务生成器、签名器和路由器。

### <a name="database-consumer"></a>数据库使用者

数据库使用者从服务总线提取消息，并将数据推送到附加的数据库，例如 SQL 数据库。

### <a name="storage-consumer"></a>存储使用者

存储使用者从服务总线提取消息，并将数据推送到附加的存储。 例如，在 Azure 存储中存储经过哈希处理的文档。

## <a name="transaction-builder-and-signer"></a>事务生成器和签名器

如果需要将入站消息中转站中的消息写入区块链，DLT 使用者会处理此操作。 DLT 使用者是一个服务，它会检索包含需要执行的事务的元数据的消息，然后将信息发送到事务生成器和签名器。 事务生成器和签名器根据数据和所需的区块链目标汇编区块链事务。 汇编后，事务将被签名。 私钥存储在 Azure Key Vault 中。

 Blockchain Workbench 从 Key Vault 检索相应的私钥，并对 Key Vault 外部的事务签名。 签名后，事务将发送到事务路由器和账本。

## <a name="transaction-routers-and-ledgers"></a>事务路由器和账本

事务路由器和账本提取已签名的事务，并将其路由到相应的区块链。 目前，Blockchain Workbench 支持将 Ethereum 用作其目标区块链。

## <a name="dlt-watcher"></a>DLT 观察程序

分布式账本技术 (DLT) 观察程序监视已附加到 Blockchain Workbench 的区块链上发生的事件。
事件反映个人和系统相关的信息。 例如，新合同实例的创建、事务执行和状态更改。 系统会捕获事件并将其发送到出站消息中转站，使其可供下游使用者使用。

例如，SQL 使用者会监视事件、使用事件，并在 SQL 数据库中填充包含的值。 使用复制可在链外存储中重新创建链中数据的副本。

## <a name="azure-sql-database"></a>Azure SQL 数据库

附加到 Blockchain Workbench 的 Azure SQL 数据库存储合约定义、配置元数据，以及区块链中存储的数据的副本（可通过 SQL 访问）。 直接访问数据库即可轻松查询、可视化或分析这些数据。 开发人员和其他用户可以使用该数据库进行报告、分析或其他以数据为中心的集成。 例如，用户可以使用 Power BI 将事务数据可视化。

此链外存储可让企业组织查询 SQL 中的数据，但不允许查询区块链账本中的数据。 此外，通过将区块链技术堆栈不可知的架构标准化，链外存储可让用户跨项目、方案和组织重复使用报告和其他项目。

## <a name="azure-storage"></a>Azure 存储

Azure 存储用于存储合约以及与合约关联的元数据。

从采购订单和提单，到新闻和医疗成像使用的图像，再到源自连拍摄像（包括治安摄像头）的视频和大型动画，文档在许多以区块链为中心的方案中发挥作用。 文档不适合直接放入区块链。

Blockchain Workbench 支持使用区块链业务逻辑添加文档或其他媒体内容的功能。 文档或媒体内容的哈希存储在区块链中，实际文档或媒体内容存储在 Azure 存储中。 关联的事务信息将传送到入站消息中转站，然后经过打包、签名，最后路由到区块链。 此过程会触发事件，而事件通过出站消息中转站共享。 SQL 数据库使用此信息，并将其发送到数据库供以后查询。 下游系统也可以使用这些事件，以对其进行相应的处理。

## <a name="monitoring"></a>监视

Workbench 使用 Application Insights 和 Azure Monitor 提供应用程序日志记录。 Application Insights 用于存储来自 Blockchain Workbench 的所有记录信息，其中包括错误、警告和成功操作。 开发人员可以使用 Application Insights 调试 Blockchain Workbench 问题。 

Azure Monitor 提供有关区块链网络运行状况的信息。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [部署 Azure Blockchain Workbench](../../blockchain-workbench/blockchain-workbench-deploy.md)