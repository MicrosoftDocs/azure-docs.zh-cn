---
title: 使用集成服务环境 (ISE) 从 Azure 逻辑应用访问 Azure 虚拟网络
description: 本概述介绍集成服务环境 (ISE) 如何帮助逻辑应用访问 Azure 虚拟网络 (VNET)
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 0206fd2b2ea0a7cfaf79aaf19052e0174645780b
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143115"
---
# <a name="access-to-azure-virtual-network-resources-from-azure-logic-apps-by-using-integration-service-environments-ises"></a>使用集成服务环境 (ISE) 从 Azure 逻辑应用访问 Azure 虚拟网络资源

有时，逻辑应用和集成帐户需要访问 [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)中受保护的资源，例如虚拟机 (VM) 和其他系统或服务。 若要设置此访问权限，可以[创建一个集成服务环境 (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md) 以运行逻辑应用和集成帐户。 当创建 ISE 时，Azure 将部署专用和独立实例的逻辑应用服务，到在 Azure 虚拟网络。 此专用实例使用专用资源（如存储），并与公用“全局”逻辑应用服务分开运行。 分离隔离的专用实例和公共的全局实例还有助于降低影响其他 Azure 租户可能会对应用的性能，也称为["噪声邻居"效果](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors)。

在创建后在 ISE 时要创建你的逻辑应用或集成帐户，, 你可以选择在 ISE 作为逻辑应用或集成帐户的位置：

![选择集成服务环境](./media/connect-virtual-network-vnet-isolated-environment-overview/select-logic-app-integration-service-environment.png)

逻辑应用现在可以直接访问系统内或通过使用任何这些项目连接到虚拟网络：

* 该系统，例如，SQL Server ISE 版本控制型连接器
* 内置的触发器或操作，如 HTTP 触发器或操作
* 自定义连接器

本概述介绍有关 ISE 如何使逻辑应用的更多详细信息和集成帐户直接访问在 Azure 虚拟网络，并将 ISE 和全球的逻辑应用服务之间的差异进行比较。
对于在本地系统不会连接到虚拟网络或不具有 ISE 版本的连接器，你可以连接到通过这些系统[设置和使用的本地数据网关](../logic-apps/logic-apps-gateway-install.md)。

> [!IMPORTANT]
> 运行在 ISE 中的逻辑应用、内置操作和连接器使用不同的定价计划，而不是基于消费的定价计划。 有关详细信息，请参阅[逻辑应用定价](../logic-apps/logic-apps-pricing.md)。

<a name="difference"></a>

## <a name="isolated-versus-global"></a>独立与全局

在 Azure 中创建的集成的服务环境 (ISE) 时，可以选择在要用于 Azure 虚拟网络*注入*在 ISE。 然后，azure 将注入，或部署，到虚拟网络的逻辑应用服务的专用实例。 此操作将创建一个独立环境，可以在专用资源上创建和运行逻辑应用。 创建逻辑应用，请选择你 ISE 作为应用的位置，使逻辑应用直接访问你的虚拟网络和在该网络中的资源。

ISE 中的逻辑应用可提供与全局逻辑应用服务相同的用户体验和类似的功能。 不仅可以使用全局逻辑应用服务中的相同内置操作和连接器，而且可以使用 ISE 特定的连接器。 例如，下面是一些提供在 ISE 中运行的版本的标准连接器：

* Azure Blob 存储、文件存储和表存储
* Azure 队列、Azure 服务总线、Azure 事件中心和 IBM MQ
* FTP 和 SFTP-SSH
* SQL Server、SQL 数据仓库、Azure Cosmos DB
* AS2、X12 和 EDIFACT

ISE 和非 ISE 连接器之间的区别在于触发器和操作运行的位置：

* 在 ISE，内置触发器和操作（例如 HTTP）始终在运行逻辑应用的同一 ISE 中运行。

* 对于提供两个版本的连接器，一个版本运行在 ISE 中，而另一个版本运行在全局逻辑应用服务中。  

  具有 ISE 标签的连接器始终运行在与逻辑应用相同的 ISE 中。 没有 ISE 标签的连接器运行在全局逻辑应用服务中。

  ![选择 ISE 连接器](./media/connect-virtual-network-vnet-isolated-environment-overview/select-ise-connectors.png)

* 在 ISE 中运行的连接器同样可在全局逻辑应用服务中使用。

<a name="create-integration-account-environment"></a>

## <a name="integration-accounts-with-ise"></a>集成帐户与 ISE

可以在集成服务环境 (ISE) 中将集成帐户与逻辑应用结合使用。 但是，这些集成帐户必须使用相同的 ISE 作为链接的逻辑应用。 ISE 中的逻辑应用只能引用同一 ISE 中的这些集成帐户。 创建集成帐户时，可以选择 ISE 作为集成帐户的位置。

## <a name="next-steps"></a>后续步骤

* 了解如何[从独立的逻辑应用连接到 Azure 虚拟网络](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
* 详细了解 [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)
* 了解 [Azure 服务的虚拟网络集成](../virtual-network/virtual-network-for-azure-services.md)
