---
title: Azure 上的 Hyperledger Fabric 联盟网络
description: 用于部署和配置 Hyperledger Fabric 联盟网络的解决方案模板
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: caleteet
manager: femila
ms.openlocfilehash: 80de4e1479fac7296889e45289a5f20e586e3f57
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510750"
---
# <a name="hyperledger-fabric-consortium-network"></a>Hyperledger Fabric 联盟网络

可以使用 Hyperledger Fabric 联盟解决方案模板在 Azure 上部署和配置 Hyperledger Fabric 联盟网络。

阅读本文后，可以：

- 获得区块链、Hyperledger Fabric 以及更复杂的联盟网络体系结构的相关知识
- 了解如何从 Azure 门户部署和配置 Hyperledger Fabric 联盟网络

## <a name="about-blockchain"></a>关于区块链

对于刚涉足区块链社区的用户而言，此解决方案模板是一个在 Azure 上以简单的可配置方式了解该技术的绝佳机会。 区块链是比特币背后的基础技术，但是，该技术远不止是虚拟货币的促成因素。 它是现有数据库、分布式系统和加密技术的组合，可实现安全的多方计算，并具有不可变性、可验证性、可审计性和攻击复原性的保障。 不同的协议采用不同的机制来提供这些属性。 [Hyperledger Fabric](https://github.com/hyperledger/fabric) 是此类协议之一。

## <a name="consortium-architecture-on-azure"></a>Azure 上的联盟体系结构

若要在 Azure 中启用 Hyperledger Fabric，可使用两种受支持的主要部署类型。 这些部署旨在根据所需目标适应不同的拓扑。

- **单个虚拟机，开发人员服务器**：此部署类型设计为开发环境，用于生成和测试在 Hyperledger Fabric 上生成的解决方案。
- **多个虚拟机，横向扩展部署**：此部署类型专门用于为联盟建模的环境，此类联盟由使用共享环境的不同参与者组成。

在任一部署中，构成 Hyperledger Fabric 核心的构建基块是相同的。  这两种部署的不同之处在于如何横向扩展以下组件。

- **CA 节点**：运行用于生成证书的证书颁发机构的节点，这些证书用于网络中的各种标识。
- **排序节点**：运行通信服务来实现交付保证的节点，例如全序广播或原子事务。
- **对等节点**：提交事务并保持状态和分布式账本副本的节点。
- **CouchDB 节点**：可以运行 CouchDB 服务的节点，该服务可以容纳状态数据库，并对链代码数据（从简单的键/值到 JSON 类型存储）执行丰富的查询。

### <a name="single-virtual-machine-architecture"></a>单虚拟机体系结构

如前所述，单虚拟机体系结构是为开发人员构建的，具有一台用于开发应用程序且占用空间较少的服务器。 显示的所有容器均在单个虚拟机中运行。 排序服务使用 [SOLO](https://github.com/hyperledger/fabric/tree/master/orderer) 实现此配置。 此配置并*非*容错排序服务，而是为开发目的而设计的轻量级服务。

![单虚拟机体系结构](./media/hyperledger-fabric-consortium-blockchain/hlf-single-arch.png)

### <a name="multiple-virtual-machine-architecture"></a>多虚拟机体系结构

多虚拟机横向扩展体系结构构建为每个核心组件均具有高可用性和可伸缩性。 此体系结构更适合生产级部署。

![多虚拟机体系结构](./media/hyperledger-fabric-consortium-blockchain/hlf-multi-arch.png)

## <a name="getting-started"></a>入门

开始之前，需要一个能够支持部署多个虚拟机和标准存储帐户的 Azure 订阅。 如果没有 Azure 订阅，可以[创建一个免费的 Azure 帐户](https://azure.microsoft.com/free/)。

拥有订阅后，请转到 [Azure 门户](https://portal.azure.com)。 选择“创建资源”>“区块链”>“Hyperledger Fabric 联盟”。

![Hyperledger Fabric 单一成员区块链市场模板](./media/hyperledger-fabric-consortium-blockchain/marketplace-template.png)

## <a name="deployment"></a>部署

在“Hyperledger Fabric 联盟”模板中，选择“创建”。

模板部署将引导你配置多节点 [Hyperledger 1.3](https://hyperledger-fabric.readthedocs.io/en/release-1.3/) 网络。 部署流程分为四个步骤：基本信息、联盟网络设置、Fabric 配置和可选组件。

### <a name="basics"></a>基本

在“基本信息”中，指定适用于任何部署的标准参数的值。 例如，订阅、资源组和基本的虚拟机属性。

![基本](./media/hyperledger-fabric-consortium-blockchain/basics.png)

| 参数名称 | 描述 | 允许的值 |
|---|---|---|
**资源前缀** | 在部署过程中预配的资源的名称前缀 |6 个字符或更少 |
**用户名** | 为此成员部署的每个虚拟机的管理员用户名 |1 - 64 个字符 |
**身份验证类型** | 向虚拟机进行身份验证的方法 |密码或 SSH 公钥|
**密码（身份验证类型 = 密码）** |部署的每个虚拟机的管理员帐户密码。 密码必须包含以下三种字符类型：1 个大写字符、1 个小写字符、1 个数字和 1 个特殊字符<br /><br />虽然所有 VM 最初都有相同的密码，但可以在预配后更改密码|12 - 72 个字符|
**SSH 密钥（身份验证类型 = SSH 公钥）** |用于远程登录的安全 shell 密钥 ||
**订阅** |要部署到的订阅 ||
**资源组** |要将联盟网络部署到的资源组 ||
**位置** |要部署第一个成员的 Azure 区域 ||

选择“确定”。

### <a name="consortium-network-settings"></a>联盟网络设置

在“网络设置”中，指定用于创建或加入现有联盟网络的输入，并配置组织设置。

![联盟网络设置](./media/hyperledger-fabric-consortium-blockchain/network-settings.png)

| 参数名称 | 描述 | 允许的值 |
|---|---|---|
**网络配置** |可以选择创建新网络或加入现有网络。 如果选择“加入现有网络”，则需要提供其他值。 |新建网络 <br/> 加入现有网络 |
**HLF CA 密码** |用于由证书颁发机构生成的证书的密码，在部署过程中创建。 密码必须包含以下三种字符类型：1 个大写字符、1 个小写字符、1 个数字和 1 个特殊字符。<br /><br />虽然所有虚拟机最初都有相同的密码，但可以在预配后更改密码。|1 - 25 个字符 |
**组织设置** |可以自定义组织的名称和证书，也可以使用默认值。|默认 <br/> 高级 |
**VPN 网络设置** | 预配 VPN 隧道网关以访问 VM | “是” <br/> “否” |

选择“确定”。

### <a name="fabric-specific-settings"></a>Fabric 特定的设置

在“Fabric 配置”中，配置网络大小和性能，并为网络可用性指定输入。 例如，排序节点和对等节点的数量，每个节点使用的持久性引擎，以及 VM 大小。

![Fabric 设置](./media/hyperledger-fabric-consortium-blockchain/fabric-specific-settings.png)

| 参数名称 | 描述 | 允许的值 |
|---|---|---|
**缩放类型** |以下两种部署类型之一：具有多个容器的单个虚拟机或横向扩展模型中的多个虚拟机。|单 VM 或多 VM |
**VM 磁盘类型** |支持每个已部署的节点的存储类型。 <br/> 若要详细了解可用的磁盘类型，请访问[选择磁盘类型](../../virtual-machines/windows/disks-types.md)。|标准 SSD <br/> 高级 SSD |

### <a name="multiple-vm-deployment-additional-settings"></a>多 VM 部署（其他设置）

![有关多 VM 部署的 Fabric 设置](./media/hyperledger-fabric-consortium-blockchain/multiple-vm-deployment.png)

| 参数名称 | 描述 | 允许的值 |
|---|---|---|
**排序节点数** |将事务排序（整理）为块的节点数。 <br />有关排序服务的其他详细信息，请访问 Hyperledger [文档](https://hyperledger-fabric.readthedocs.io/en/release-1.1/ordering-service-faq.html) |1-4 |
**排序节点虚拟机大小** |用于网络中排序节点的虚拟机大小|标准 Bs、<br />标准 Ds、<br />标准 FS |
**对等节点数** | 归联盟成员所有，执行事务并保持状态和账本副本的节点数。<br />有关排序服务的其他详细信息，请访问 Hyperledger [文档](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html)。|1-4 |
**节点状态持久性** |对等节点使用的持久性引擎。 可以为每个对等节点配置此引擎。 请参阅下面有关多对等节点的详细信息。|CouchDB <br />LevelDB |
**对等节点虚拟机大小** |用于网络中所有节点的虚拟机大小|标准 Bs、<br />标准 Ds、<br />标准 FS |

### <a name="multiple-peer-node-configuration"></a>多对等节点配置

此模板允许为每个对等节点选择持久性引擎。 例如，如果有三个对等节点，则可以在一个节点上使用 CouchDB，在另外两个节点上使用 LevelDB。

![多对等节点配置](./media/hyperledger-fabric-consortium-blockchain/multiple-peer-nodes.png)

选择“确定”。

### <a name="deploy"></a>部署

在“摘要”边栏选项卡中，查看指定的输入并运行基本的部署前验证。

![摘要](./media/hyperledger-fabric-consortium-blockchain/summary.png)

查看法律和隐私条款，然后选择“购买”进行部署。 部署时间可能从几分钟到数十分钟不等，具体取决于正在预配的 VM 数量。

## <a name="next-steps"></a>后续步骤

现在你可以专注于针对 Hyperledger 联盟区块链网络的应用程序和链式代码开发。
