---
title: 通过集成服务环境 (ISE) 从 Azure 逻辑应用连接到 Azure 虚拟网络
description: 创建集成服务环境 (ISE)，以便逻辑应用和集成帐户可以访问 Azure 虚拟网络，同时保持专用并与公共或“全局”Azure 相隔离
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 8809a2fed5a44910e3a353d9dc5bc41ea964a1ce
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150566"
---
# <a name="connect-to-azure-virtual-networks-from-azure-logic-apps-by-using-an-integration-service-environment-ise"></a>使用集成服务环境 (ISE) 从 Azure 逻辑应用连接到 Azure 虚拟网络

对于逻辑应用和集成帐户需要访问 [ Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)的情况，请创建[集成服务环境 (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)。 ISE 是私有和隔离的环境使用专用的存储和保留独立于公共或"全局"逻辑应用服务的其他资源。 这种分离还减少了其他 Azure 租户可能对应用性能产生的影响。 可以将 ISE 注入到 Azure 虚拟环境，后者然后会将逻辑应用服务部署到虚拟网络中。 创建逻辑应用或集成帐户时，请选择此 ISE 作为其位置。 然后，逻辑应用或集成帐户可以直接访问虚拟网络中的虚拟机 (VM)、服务器、系统和服务等资源。

![选择集成服务环境](./media/connect-virtual-network-vnet-isolated-environment/select-logic-app-integration-service-environment.png)

本文介绍如何完成以下任务：

* 在 Azure 虚拟网络中设置端口，使流量能够跨虚拟网络中的子网遍历集成服务环境 (ISE)。

* 创建集成服务环境 (ISE)。

* 创建可在 ISE 中运行的逻辑应用。

* 在 ISE 中创建逻辑应用的集成帐户。

有关集成服务环境的详细信息，请参阅[从 Azure 逻辑应用访问 Azure 虚拟网络资源](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)。

## <a name="prerequisites"></a>必备组件

* Azure 订阅。 如果没有 Azure 订阅，请<a href="https://azure.microsoft.com/free/" target="_blank">注册一个免费 Azure 帐户</a>。

  > [!IMPORTANT]
  > 运行在 ISE 中的逻辑应用、内置操作和连接器使用不同的定价计划，而不是基于消费的定价计划。 有关详细信息，请参阅[逻辑应用定价](../logic-apps/logic-apps-pricing.md)。

* [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)。 如果没有虚拟网络，请了解如何[创建 Azure 虚拟网络](../virtual-network/quick-create-portal.md)。 

  * 你的虚拟网络必须有四个*空*部署以及在 ISE 中创建资源的子网。 可以提前创建这些子网或可以等待，直到您创建可以在同一时间创建子网在 ISE。 详细了解如何[子网的要求](#create-subnet)。 
  
    > [!NOTE]
    > 如果您使用[ExpressRoute](../expressroute/expressroute-introduction.md)，它提供与 Microsoft 云服务的专用连接，则必须[创建一个路由表](../virtual-network/manage-route-table.md)，具有以下路由并将该表链接由你 ISE 每个子网：
    > 
    > **名称**: <*路由名称*><br>
    > **地址前缀**:0.0.0.0/0<br>
    > **下一跃点**：Internet

  * 请确保你的虚拟网络[使这些端口可](#ports)以便在 ISE 可正常运行且保持可访问。

* 如果想要为 Azure 虚拟网络，使用自定义 DNS 服务器[通过执行以下步骤设置这些服务器](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)在 ISE 部署到你的虚拟网络之前。 否则，每次更改 DNS 服务器时，都必须重启 ISE（ISE 公共预览版提供的一项功能）。

* 有关[如何创建逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知识

<a name="ports"></a>

## <a name="set-up-network-ports"></a>设置网络端口

若要保持正常运行且可供访问，集成服务环境 (ISE) 需要使用虚拟网络中的特定端口。 否则，如果其中的任一端口不可用，ISE 可能会停止运行，使你无法访问 ISE。 在虚拟网络中使用 ISE 时，常见的设置问题是一个或多个端口被阻止。 对于 ISE 与目标系统之间的连接，所用的连接器还可能有其自身的端口要求。 例如，如果使用 FTP 连接器来与 FTP 系统通信，请确保在该 FTP 系统上使用的端口（例如用于发送命令的端口 21）可用。

若要跨虚拟网络的子网，你将部署在 ISE 控制的流量，您可以设置[网络安全组](../virtual-network/security-overview.md)对由这些子网[筛选网络流量跨子网](../virtual-network/tutorial-filter-network-traffic.md)。 下列表格描述了 ISE 在虚拟网络中使用的端口，以及这些端口的使用位置。 [资源管理器服务标记](../virtual-network/security-overview.md#service-tags)表示一组 IP 地址前缀，可帮助创建安全规则时最小化复杂性。

> [!IMPORTANT]
> 对于子网内的内部通信，ISE 需要打开这些子网中的所有端口。

| 目的 | Direction | 端口 | 源服务标记 | 目标服务标记 | 说明 |
|---------|-----------|-------|--------------------|-------------------------|-------|
| 从 Azure 逻辑应用通信 | 出站 | 80 和 443 | VirtualNetwork | Internet | 端口取决于逻辑应用服务与之通信的外部服务 |
| Azure Active Directory | 出站 | 80 和 443 | VirtualNetwork | AzureActiveDirectory | |
| Azure 存储依赖项 | 出站 | 80 和 443 | VirtualNetwork | 存储 | |
| Intersubnet 通信 | 入站和出站 | 80 和 443 | VirtualNetwork | VirtualNetwork | 子网之间的通信 |
| 与 Azure 逻辑应用通信 | 入站 | 443 | Internet  | VirtualNetwork | 计算机或调用任何请求触发器或 webhook 在逻辑应用中存在的服务的 IP 地址。 关闭或阻止此端口可防止对具有请求触发器的逻辑应用的 HTTP 调用。  |
| 逻辑应用运行历史记录 | 入站 | 443 | Internet  | VirtualNetwork | 从中查看逻辑应用的计算机的 IP 地址的运行历史记录。 虽然关闭或阻止此端口不会阻止你查看运行历史记录，但不能查看输入和输出中的每个步骤的运行历史记录。 |
| 连接管理 | 出站 | 443 | VirtualNetwork  | Internet | |
| 发布诊断日志和指标 | 出站 | 443 | VirtualNetwork  | AzureMonitor | |
| 从 Azure 流量管理器的通信 | 入站 | 443 | AzureTrafficManager | VirtualNetwork | |
| 逻辑应用设计器 - 动态属性 | 入站 | 454 | Internet  | VirtualNetwork | 请求来自逻辑应用[访问终结点在该区域中的 IP 地址的入站](../logic-apps/logic-apps-limits-and-config.md#inbound)。 |
| 应用服务管理依赖项 | 入站 | 454 和 455 | AppServiceManagement | VirtualNetwork | |
| 连接器部署 | 入站 | 454 & 3443 | Internet  | VirtualNetwork | 所需的部署和更新连接器。 关闭或阻止此端口将导致 ISE 部署失败，并防止连接器更新或修补程序。 |
| Azure SQL 依赖关系 | 出站 | 1433 | VirtualNetwork | SQL |
| Azure 资源运行状况 | 出站 | 1886 | VirtualNetwork | Internet | 发布到资源运行状况的运行状况状态 |
| API 管理 - 管理终结点 | 入站 | 3443 | APIManagement  | VirtualNetwork | |
| “记录到事件中心”策略和监视代理中的依赖项 | 出站 | 5672 | VirtualNetwork  | EventHub | |
| 访问角色实例之间的 Azure Redis 缓存实例 | 入站 <br>出站 | 6379-6383 | VirtualNetwork  | VirtualNetwork | 此外，对于要使用 Azure 缓存适用于 Redis 的 ISE，则必须打开这些[Redis 常见问题解答针对 Azure 缓存中所述的出站和入站端口](../azure-cache-for-redis/cache-how-to-premium-vnet.md#outbound-port-requirements)。 |
| Azure 负载均衡器 | 入站 | * | AzureLoadBalancer | VirtualNetwork |  |
||||||

<a name="create-environment"></a>

## <a name="create-your-ise"></a>创建 ISE

若要创建集成服务环境 (ISE)，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com)上的 Azure 主菜单上，选择“创建资源”。
在搜索框中，输入“集成服务环境”作为筛选器。

   ![新建资源](./media/connect-virtual-network-vnet-isolated-environment/find-integration-service-environment.png)

1. 在集成服务环境创建窗格中，选择**创建**。

   ![选择“创建”](./media/connect-virtual-network-vnet-isolated-environment/create-integration-service-environment.png)

1. 为环境提供以下详细信息，然后选择“查看 + 创建”，例如：

   ![提供环境详细信息](./media/connect-virtual-network-vnet-isolated-environment/integration-service-environment-details.png)

   | 属性 | 需要 | Value | 描述 |
   |----------|----------|-------|-------------|
   | **订阅** | 是 | <*Azure-subscription-name*> | 用于环境的 Azure 订阅 |
   | **资源组** | 是 | <*Azure-resource-group-name*> | 要在其中创建环境的 Azure 资源组 |
   | **集成服务环境名称** | 是 | <*environment-name*> | 为环境指定的名称 |
   | **位置** | 是 | <*Azure-datacenter-region*> | 要在其中部署环境的 Azure 数据中心区域 |
   | **额外容量** | 是 | 0 到 10 | 要使用此 ISE 资源的其他处理单位数。 若要在创建后添加容量，请参阅[添加 ISE 容量](#add-capacity)。 |
   | **虚拟网络** | 是 | <Azure-virtual-network-name> | 要注入环境以便该环境中的逻辑应用可以访问虚拟网络的 Azure 虚拟网络。 如果您没有网络，[先创建 Azure 虚拟网络](../virtual-network/quick-create-portal.md)。 <p>**重要**：创建 ISE 时可以仅执行此注入。 |
   | **子网** | 是 | <*subnet-resource-list*> | ISE 需要四个空的子网来在环境中创建资源。 要创建每个子网，请[按照此表下方的步骤操作](#create-subnet)。  |
   |||||

   <a name="create-subnet"></a>

   **创建子网**

   若要在你的环境中创建的资源，在 ISE 需要四个*空*不委托给任何服务的子网。 
   您*不能*后创建你的环境更改这些子网地址。 每个子网必须满足以下条件：

   * 已开始以字母字符或下划线，并且也不保证这些字符的名称： `<`， `>`， `%`， `&`， `\\`， `?`， `/`

   * 使用[无类域间路由 (CIDR) 格式](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)和 B 类地址空间。

   * 使用最少`/27`的地址空间，因为每个子网都必须有 32 个地址作为*最小*。 例如：

     * `10.0.0.0/27` 具有 32 个地址，因为 2<sup>(32-27)</sup>为 2<sup>5</sup>或 32。

     * `10.0.0.0/24` 具有 256 个地址，因为 2<sup>(32-24)</sup>为 2<sup>8</sup>或 256。

     * `10.0.0.0/28` 仅有 16 位地址长度且太小，因为 2<sup>(32-28)</sup>为 2<sup>4</sup>或 16。

     若要了解有关计算地址的详细信息，请参阅[IPv4 CIDR 块](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks)。

   * 如果您使用[ExpressRoute](../expressroute/expressroute-introduction.md)，请记住，登录[创建一个路由表](../virtual-network/manage-route-table.md)，具有以下路由并将该表链接由你 ISE 每个子网：

     **名称**: <*路由名称*><br>
     **地址前缀**:0.0.0.0/0<br>
     **下一跃点**：Internet

   1. 在“子网”列表下，选择“管理子网配置”。

      ![管理子网配置](./media/connect-virtual-network-vnet-isolated-environment/manage-subnet.png)

   1. 在“子网”窗格中，选择“子网”。

      ![添加子网](./media/connect-virtual-network-vnet-isolated-environment/add-subnet.png)

   1. 在“添加子网”窗格中，提供此信息。

      * **名称**：子网的名称
      * **地址范围(CIDR 块)**：虚拟网络采用 CIDR 格式的子网范围

      ![添加子网详细信息](./media/connect-virtual-network-vnet-isolated-environment/subnet-details.png)

   1. 完成后，选择“确定”。

   1. 重复这些步骤创建额外三个子网。

      > [!NOTE]
      > 如果你尝试创建的子网不是有效的在 Azure 门户显示一条消息，但不会阻止您的进度。

1. Azure 成功验证 ISE 信息后，请选择“创建”，例如：

   ![成功验证后，选择“创建”](./media/connect-virtual-network-vnet-isolated-environment/ise-validation-success.png)

   Azure 随即开始部署该环境，但此过程最长可能需要两个小时才能完成。 
   若要检查部署状态，请在 Azure 工具栏上选择通知图标打开通知窗格。

   ![检查部署状态](./media/connect-virtual-network-vnet-isolated-environment/environment-deployment-status.png)

   如果部署成功完成，Azure 会显示以下通知：

   ![部署成功](./media/connect-virtual-network-vnet-isolated-environment/deployment-success.png)

   否则，请按照 Azure 门户说明排查部署问题。

   > [!NOTE]
   > 如果部署失败，或者删除在 ISE 中，Azure 可能要需要一小时之前释放您的子网。 这种延迟表示可能需要重复使用在 ISE 中另一个子网之前等待的表示。 
   >
   > 如果删除你的虚拟网络，Azure 时通常需要两个小时前发布了您的子网，但此操作可能需要更长。 
   > 在删除虚拟网络时，请确保没有资源仍连接。 请参阅[删除虚拟网络](../virtual-network/manage-virtual-network.md#delete-a-virtual-network)。

1. 如果部署完成后 Azure 未自动转到你的环境，可以选择“转到资源”来查看该环境。  

有关创建子网的详细信息，请参阅[添加虚拟网络子网](../virtual-network/virtual-network-manage-subnet.md)。

<a name="create-logic-apps-environment"></a>

## <a name="create-logic-app---ise"></a>创建逻辑应用 - ISE

要创建使用集成服务环境 (ISE) 的逻辑应用，请遵循[如何创建逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)中的步骤，但要注意以下差别： 

* 创建逻辑应用时，在“位置”属性下，从“集成服务环境”部分中选择 ISE，例如：

  ![选择集成服务环境](./media/connect-virtual-network-vnet-isolated-environment/create-logic-app-with-integration-service-environment.png)

* 可以使用相同的内置触发器和操作，例如 HTTP、 与逻辑应用在相同 ISE 中运行。 带有“ISE”标签的连接器也在运行逻辑应用的同一 ISE 中运行。 没有 ISE 标签的连接器运行在全局逻辑应用服务中。

  ![选择 ISE 连接器](./media/connect-virtual-network-vnet-isolated-environment/select-ise-connectors.png)

* 将 ISE 注入到 Azure 虚拟网络中后，ISE 中的逻辑应用可以直接访问该虚拟网络中的资源。 对于连接到虚拟网络的本地系统，请将 ISE 注入到该网络，以便逻辑应用可使用这些项中的任意一项直接访问这些系统： 

  * 该系统的 ISE 连接器，例如 SQL Server
  
  * HTTP 操作 
  
  * 自定义连接器

  对于没有在虚拟网络中或没有 ISE 连接器的本地系统，请首先[设置本地数据网关](../logic-apps/logic-apps-gateway-install.md)。

<a name="create-integration-account-environment"></a>

## <a name="create-integration-account---ise"></a>创建集成帐户 - ISE

若要对集成服务环境 (ISE) 中的逻辑应用使用集成帐户，该集成帐户必须使用逻辑应用所在的同一个环境。 ISE 中的逻辑应用只能引用同一 ISE 中的集成帐户。 

要创建使用 ISE 的集成帐户，请按照[如何创建集成帐户](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)中的步骤，但“位置”属性除外，该属性现在会显示“集成服务环境”部分。 请选择 ISE 而不要选择区域，例如：

![选择集成服务环境](./media/connect-virtual-network-vnet-isolated-environment/create-integration-account-with-integration-service-environment.png)

<a name="add-capacity"></a>

## <a name="add-ise-capacity"></a>添加 ISE 容量

在 ISE 基本单位具有固定容量，因此如果需要更大的吞吐量，你可以添加更多缩放单位。 你可以基于性能指标或额外的处理单位数的自动缩放。 如果选择基于指标自动缩放，可以从各种条件中进行选择，并指定满足该条件的阈值条件。

1. 在 Azure 门户中，找到你 ISE。

1. 若要查看使用情况和性能指标的 ISE 中，在 ISE 的主菜单上，选择**概述**。

   ![查看使用情况的 ISE](./media/connect-virtual-network-vnet-isolated-environment/integration-service-environment-usage.png)

1. 若要设置自动缩放下,**设置**，选择**横向扩展**。上**配置**选项卡上，选择**启用自动缩放**。

   ![启用自动缩放](./media/connect-virtual-network-vnet-isolated-environment/scale-out.png)

1. 有关**自动缩放设置名称**，提供你设置的名称。

1. 在中**默认**部分中，选择**基于指标缩放**或**缩放为具体实例数**。

   * 如果选择基于实例的，输入处理单位的数介于 0 和 10 之间 （含）。

   * 如果选择基于指标的，请按照下列步骤：

     1. 在中**规则**部分中，选择**添加规则**。

     1. 上**缩放规则**窗格中，设置你的条件和操作执行时，规则将触发。

     1. 完成后，选择**添加**。

1. 完成后自动缩放设置后，保存所做的更改。

## <a name="next-steps"></a>后续步骤

* 详细了解 [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)
* 了解 [Azure 服务的虚拟网络集成](../virtual-network/virtual-network-for-azure-services.md)
