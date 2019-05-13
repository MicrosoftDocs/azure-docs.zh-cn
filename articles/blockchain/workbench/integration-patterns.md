---
title: Azure Blockchain Workbench 中的智能合约集成模式
description: Azure Blockchain Workbench 中的智能合约集成模式概述。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 6be1d2b1e68c52d45a42027be5f6d69cc2c25adc
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509928"
---
# <a name="smart-contract-integration-patterns"></a>智能合约集成模式

智能合同通常表示需要与外部系统和设备集成的业务工作流。

这些工作流的要求包括需要对分布式账本（包含来自外部系统、服务或设备的数据）发起事务。 它们还需要让外部系统对源自分布式账本中的智能合约的事件做出反应。

REST API 和消息集成将事务从外部系统发送到 Azure Blockchain Workbench 应用程序中包含的智能合同。 它还会根据应用程序中发生的更改向外部系统发送事件通知。

针对数据集成方案，Azure Blockchain Workbench 包含一组数据库视图，这些视图合并了区块链中的事务数据以及有关应用程序和智能合约的元数据的组合。

此外，在某些情况下（例如与供应链或媒体相关的情况），可能还需要集成文档。 尽管 Azure Blockchain Workbench 不提供 API 调用用于直接处理文档，但可将文档合并到区块链应用程序中。 本部分也会介绍该模式。

本部分包括在端到端解决方案中实施上述每种集成可用的模式。

## <a name="rest-api-based-integration"></a>基于 REST API 的集成

Azure Blockchain Workbench 生成的 Web 应用程序中的功能是通过 REST API 公开的。 这些功能包括 Azure Blockchain Workbench 对应用程序的上传、配置和管理，将事务发送到分布式账本，以及查询应用程序元数据和账本数据。

REST API 主要用于 Web、移动和 bot 应用程序等交互式客户端。

本节重点介绍将事务发送到分布式账本的 REST API 方面的模式，以及 Azure Blockchain Workbench 的链外 SQL 数据库查询事务数据的模式。

### <a name="sending-transactions-to-a-distributed-ledger-from-an-external-system"></a>将事务从外部系统发送到分布式账本

Azure Blockchain Workbench REST API 发送经身份验证的请求以在分布式账本上执行事务。

![将事务发送到分布式账本](./media/integration-patterns/send-transactions-ledger.png)

使用前面描述的过程执行事务，其中：

-   外部应用程序对预配为 Azure Blockchain Workbench 部署的一部分的 Azure Active Directory 进行身份验证。
-   获得授权的用户接收一个可以连同请求一起发送到 API 的持有者令牌。
-   外部应用程序使用持有者令牌对 REST API 发出调用。
-   REST API 将请求打包为消息，并将其发送到服务总线。 在服务总线中检索消息并签名，然后发送到相应的分布式账本。
-   REST API 向 Azure Blockchain Workbench SQL 数据库发出请求，以记录该请求并建立当前预配状态。
-   SQL 数据库返回预配状态，API 调用向调用它的外部应用程序返回 ID。

### <a name="querying-blockchain-workbench-metadata-and-distributed-ledger-transactions"></a>查询 Blockchain Workbench 元数据和分布式账本事务

Azure Blockchain Workbench REST API 将经过身份验证的请求发送到与分布式账本上的智能合同执行相关的查询详细信息。

![查询元数据](./media/integration-patterns/querying-metadata.png)

使用前面描述的过程进行查询，其中：

1. 外部应用程序对预配为 Azure Blockchain Workbench 部署的一部分的 Azure Active Directory 进行身份验证。
2. 获得授权的用户接收一个可以连同请求一起发送到 API 的持有者令牌。
3. 外部应用程序使用持有者令牌对 REST API 发出调用。
4. REST API 查询来自 SQL 数据库的请求的数据，并将其返回给客户端。

## <a name="messaging-integration"></a>消息传递集成

消息传递集成简化了无法或者不需要进行交互式登录的系统、服务和设备的交互。 消息集成侧重于两种类型的消息：在分布式账本上执行的消息请求事务，以及事务发生时由该账本公开的事件。

消息传递集成侧重于用户创建、合同创建和合同事务执行相关的事务的执行和监视，主要由无外设后端系统使用。

本部分介绍的模式侧重于可将事务发送到分布式账本的基于消息的 API 功能，以及表示基础分布式账本公开的事件消息的模式。

### <a name="one-way-event-delivery-from-a-smart-contract-to-an-event-consumer"></a>从智能合同到事件使用者的单向事件递送 

在此方案中，事件（例如，状态更改或特定类型的事务的执行）发生在智能合同中。 此事件通过事件网格广播到下游使用者，然后，这些使用者采取相应的措施。

此方案的一个例子：发生某个事务时，使用者会收到警报并可以采取措施，例如，在 SQL 数据库或 Common Data Service 中记录信息。 此方案与 Workbench 在填充其链外 SQL DB 时所遵循的模式相同。

另一个例子是，智能合同转换为特定状态，例如，当合同进入 *OutOfCompliance* 状态。 发生这种状态更改时，可能会触发警报并将其发送到管理员的手机。

![单向事件传送](./media/integration-patterns/one-way-event-delivery.png)

使用前面描述的过程会出现这种情况，其中：

-   智能合同转换为新状态，并将事件发送到账本。
-   账本接收事件并将其传送到 Azure Blockchain Workbench。
-   Azure Blockchain Workbench 从账本订阅事件并接收事件。
-   Azure Blockchain Workbench 将事件发布到事件网格中的订阅者。
-   外部系统订阅事件网格、使用消息并采取相应的措施。

## <a name="one-way-event-delivery-of-a-message-from-an-external-system-to-a-smart-contract"></a>从外部系统向智能合同单向传送事件消息

另外还有一种反向流动的方案。 在这种情况下，事件由传感器或外部系统生成，该事件中的数据应发送到智能合同。

一个常见的例子是传送金融市场中的数据（例如商品、股票或债券的价格）传送到智能合同。

### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>以预期的格式在 Azure Blockchain Workbench 中直接传送

某些应用程序旨在与 Azure Blockchain Workbench 集成，可直接以预期的格式生成和发送消息。

![直接传送](./media/integration-patterns/direct-delivery.png)

使用前面描述的过程进行此传送，其中：

-   事件在外部系统中发生，并触发 Azure Blockchain Workbench 的消息创建。
-   外部系统中编写的代码以已知的格式创建此消息，并将其直接发送到服务总线。
-   Azure Blockchain Workbench 从服务总线订阅事件并接收消息。
-   Azure Blockchain Workbench 向账本发起调用，并将数据从外部系统发送到特定的合同。
-   收到消息后，合同将转换为新状态。

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>以未知的格式向 Azure Blockchain Workbench 传送消息

某些系统无法修改，因此无法以 Azure Blockchain Workbench 所用的标准格式传送消息。 在这种情况下，通常可以使用这些系统的现有机制和消息格式。 具体而言，可以使用逻辑应用、Azure Functions 或其他自定义代码将这些系统的本机消息类型映射到预期的某种标准消息格式，从而对其进行转换。

![未知消息格式](./media/integration-patterns/unknown-message-format.png)

使用前面描述的过程会发生此情况，其中：

-   事件在外部系统中发生，并触发消息创建。
-   使用逻辑应用或自定义代码接收该消息，并将其转换为 Azure Blockchain Workbench 的标准格式消息。
-   逻辑应用将已转换的消息直接发送到服务总线。
-   Azure Blockchain Workbench 从服务总线订阅事件并接收消息。
-   Azure Blockchain Workbench 向账本发起调用，并将数据从外部系统发送到合同中的特定函数。
-   函数执行，并且通常会修改状态。 状态更改推动了智能合同中反映的业务工作流，使其他函数现在能够适当地执行。

### <a name="transitioning-control-to-an-external-process-and-await-completion"></a>将控制权转换到外部过程并等待完成

在某些情况下，智能合同必须停止内部执行并转接到外部过程。 然后，外部过程将会完成并向智能合同发送消息，然后执行将在智能合同中继续。

#### <a name="transition-to-the-external-process"></a>转换到外部过程

此模式通常是使用以下方法实现的：

-   智能合同转换到特定的状态。 在此状态下，在外部系统采取所需的措施之前，无法执行任何函数，或者只能执行有限数量的函数。
-   状态更改以事件的形式传递给下游使用者。
-   下游使用者接收事件，并触发外部代码执行。

![将控制权转换到外部过程](./media/integration-patterns/transition-external-process.png)

#### <a name="return-of-control-from-the-smart-contract"></a>从智能合同返回控制权

根据能否自定义外部系统，有时可以（或不可以）使用 Azure Blockchain Workbench 预期需要的标准格式之一传送消息。 外部系统能否生成以下消息之一决定了要采用以下两种返回路径中的哪一种。

##### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>以预期的格式在 Azure Blockchain Workbench 中直接传送

![](./media/integration-patterns/direct-delivery.png)

在此模型中，完成上述过程后，将与合同通信，并发生后续的状态更改，其中 -

-   在即将完成或者抵达外部代码执行的特定里程碑后，会向连接到 Azure Blockchain Workbench 的服务总线发送事件。

-   对于无法直接修改、因而无法编写符合 API 预期要求的消息的系统，将会转换消息。

-   消息内容将被打包，并发送到智能合同中的特定函数。 该发送是代表与外部系统相关联的用户完成的。

-   函数执行，并且通常会修改状态。 状态更改推动了智能合同中反映的业务工作流，使其他函数现在能够适当地执行。

### 

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>以未知的格式向 Azure Blockchain Workbench 传送消息

![未知消息格式](./media/integration-patterns/unknown-message-format.png)

在这种无法直接发送标准格式的消息的模型中，完成上述过程后，将与合同通信，并发生后续的状态更改，其中：

1.  在即将完成或者抵达外部代码执行的特定里程碑后，会向连接到 Azure Blockchain Workbench 的服务总线发送事件。
2.  使用逻辑应用或自定义代码接收该消息，并将其转换为 Azure Blockchain Workbench 的标准格式消息。
3.  逻辑应用将已转换的消息直接发送到服务总线。
4.  Azure Blockchain Workbench 从服务总线订阅事件并接收消息。
5.  Azure Blockchain Workbench 向账本发起调用，并将数据从外部系统发送到特定的合同。
6. 消息内容将被打包，并发送到智能合同中的特定函数。 该发送是代表与外部系统相关联的用户完成的。
7.  函数执行，并且通常会修改状态。 状态更改推动了智能合同中反映的业务工作流，使其他函数现在能够适当地执行。

## <a name="iot-integration"></a>IoT 集成

一种常见的集成方案是在智能合同中包含从传感器检索到的遥测数据。 根据传感器传送的数据，智能合同可以采取明智的措施，并可以更改状态。

例如，如果运送药物的卡车温度飙升到 110 度，同时未在供应链中检测到并消除此问题，则可能会影响药物的效力，并可能造成公共安全问题。 如果驾驶员将其车辆加速到 100 英里/小时，则生成的传感器信息可能会造成保险被保险公司取消。 如果汽车是租赁过来的，GPS 数据可能会指示驾驶员超出了其租赁协议涵盖的地理区域，从而产生罚金。

难点在于，这些传感器可以持续传送数据，但不适合用于将所有这些数据发送到智能合同。 典型的方法是限制发送到区块链的消息数，同时将所有消息传送到辅助存储。 例如，仅当包含的值超出智能合同的议定范围时，才按固定的间隔传送收到的消息（如每小时传送一次）。 检查超出容限的值可以确保接收和执行与合同业务逻辑相关的数据。 按间隔检查值可以确认传感器仍在报告数据。 所有数据发送到辅助报告存储，以进行更广泛的报告、分析和机器学习。 例如，尽管不一定需要每隔一分钟就获取智能合同的 GPS 传感器读数一次，但这些读数能够在报告或地图路线中提供有用的数据。

在 Azure 平台上，与设备的集成通常是使用 IoT 中心实现的。 IoT 中心根据内容提供消息路由，并启用前面描述的功能类型。

![IoT 消息](./media/integration-patterns/iot.png)

该过程描述了一种模式：

-   设备直接或通过现场网关与 IoT 中心通信。
-   IoT 中心接收消息，并根据建立的路由评估消息（例如，检查消息的内容）。 传感器是否会报告 50 度以上的温度？
-   IoT 中心将符合条件的消息发送到定义的服务总线以进行路由。
-   逻辑应用或其他代码侦听 IoT 中心为路由建立的服务总线。
-   逻辑应用或其他代码检索消息并其转换为已知的格式。
-   已转换的消息（现在为标准格式）发送到 Azure Blockchain Workbench 的服务总线。
-   Azure Blockchain Workbench 从服务总线订阅事件并接收消息。
-   Azure Blockchain Workbench 向账本发起调用，并将数据从外部系统发送到特定的合同。
-   收到消息后，合同将评估数据，并可以根据评估结果更改状态，例如，如果温度过高，则会将状态更改为“不合规”。

## <a name="data-integration"></a>数据集成

除 REST 和基于消息的 API 以外，在 Azure Blockchain Workbench 中还能访问填充了应用程序和合同元数据以及分布式账本中事务数据的 SQL 数据库。

![数据集成](./media/integration-patterns/data-integration.png)

数据集成是我们所熟知的：

-   Azure Blockchain Workbench 在实施其正常的操作行为过程中存储有关应用程序、工作流、合同和事务的元数据。
-   外部系统或工具提供一个或多个对话框用于简化有关数据库的信息（例如数据库服务器名称、数据库名称、身份验证类型、登录凭据，以及要使用的数据库视图）的收集。
-   针对 SQL 数据库视图编写查询，以便于外部系统、服务、报告、开发人员工具和企业生产力工具在下游的使用。

## <a name="storage-integration"></a>存储集成

许多方案可能需要合并可证实的文件。 出于多种原因，将文件放在区块链中并不合适。 一种常见方法是针对文件执行加密哈希（例如，SHA-256），并在分布式账本中共享该哈希。 将来再次执行该哈希应返回相同的结果。 如果修改了文件（即使只是修改了图像中的一个像素），该哈希将返回不同的值。

![存储集成](./media/integration-patterns/storage-integration.png)

可按如下所述实现此模式：

-   外部系统在 Azure 存储等存储机制中保存某个文件。
-   使用该文件或者该文件与关联的元数据（例如所有者的标识符、文件所在的 URL，等等）生成一个哈希。
-   将该哈希和所有元数据发送到智能合同上的某个函数（例如 *FileAdded*）
-   将来，可以再次对该文件和元数据进行哈希处理，并将其与账本中存储的值进行比较。

## <a name="prerequisites-for-implementing-integration-patterns-using-the-rest-and-message-apis"></a>使用 REST 和消息 API 实现集成模式的先决条件

为了方便外部系统或设备使用 REST 或消息 API 与智能合同进行交互，必须执行以下操作：

1. 在用于联盟的 Azure Active Directory 中，创建一个表示外部系统或设备的帐户。
2. 在 Azure Blockchain Workbench 应用程序的一个或多个适当的智能合同具有定义为接受来自外部系统或设备的事件的功能。
3. 智能合同的应用程序配置文件包含分配给系统或设备的角色。
4. 智能合同的应用程序配置文件确定处于哪些状态时，此函数可由定义的角色调用。
5. 应用程序配置文件及其智能合同上传到 Azure Blockchain Workbench。

上传应用程序后，外部系统的 Azure Active Directory 帐户将分配给合同和关联的角色。

## <a name="testing-external-system-integration-flows-prior-to-writing-integration-code"></a>编写集成代码之前测试外部系统集成流 

与外部系统集成是许多方案的关键要求。 最好是能够在开发代码之前或者在此同时，验证智能合同是否能够与外部系统集成。

使用 Azure Active Directory (Azure AD) 能够大大提高开发人员的工作效率和加速面市。 具体而言，代码与外部系统的集成可能需要经历很长的一段时间。 使用 Azure AD 和 Azure Blockchain Workbench 的 UX 自动生成功能，开发人员可以像登录外部系统一样登录到 Blockchain Workbench，并通过 UX 填充来自外部系统的值。 在为外部系统编写集成代码之前，可以在概念证明环境中快速开发和验证想法。
