---
title: Hyperledger Fabric 联盟
description: 使用 Hyperledger Fabric 联盟解决方案模板来部署和配置单一成员网络
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 5/21/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: ee8057be98d18db5963a3e5f1ba1f8bd8d76fe05
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240594"
---
# <a name="hyperledger-fabric-single-member-network"></a>Hyperledger Fabric 单一成员网络

可以使用 Hyperledger Fabric 联盟解决方案模板来部署和配置 Hyperledger Fabric 单一成员（多节点）网络。

阅读本文后，可以：

- 获得区块链、Hyperledger Fabric 以及更复杂的联盟网络体系结构的相关知识
- 了解从 Azure 门户部署和配置单一成员 Hyperledger Fabric 联盟网络的方法

## <a name="about-blockchain"></a>关于区块链

对于刚涉足区块链社区的用户而言，这是一个在 Azure 上以简单的可配置方式了解该技术的绝佳机会。 区块链是比特币背后的基础技术，但是，该技术远不止是虚拟货币的促成因素。 它是现有数据库、分布式系统和加密技术的组合，可实现安全的多方计算，并具有不可变性、可验证性、可审计性和攻击复原性的保障。 不同的协议采用不同的机制来提供这些属性。 [Hyperledger Fabric](https://github.com/hyperledger/fabric) 是此类协议之一。

## <a name="consortium-architecture-on-azure"></a>Azure 上的联盟体系结构

此模板部署一个拓扑，有助于进行测试以及为单一组织（单一成员）中的用户模拟生产。 部署包含单一区域中的多个节点网络，很快就会扩展到多个区域。

网络包含三种类型的节点：

1. 成员节点：运行 Fabric 成员身份服务，注册和管理网络成员的节点。 此节点最终可聚集以实现可伸缩性和高可用性，但是在本实验中，将使用单一成员节点。
2. 排序节点：运行通信服务来实现交付保证的节点，例如全序广播或原子事务。
3. 对等节点：提交事务并保持状态和分布式账本副本的节点。

## <a name="getting-started"></a>入门

开始之前，需要一个能够支持部署多个虚拟机和标准存储帐户的 Azure 订阅。 如果没有 Azure 订阅，可以[创建一个免费的 Azure 帐户](https://azure.microsoft.com/free/)。

默认情况下，大多数订阅类型都支持小型部署拓扑，而无需增加配额。 一位成员的最小可能部署需要：

- 5 个虚拟机（5 个核数）
- 1 个 VNet
- 1 个负载均衡器
- 1 个公共 IP 地址

拥有订阅后，请转到 [Azure 门户](https://portal.azure.com)。 选择“+”，选择“区块链”，然后选择“Hyperledger Fabric 单一成员区块链”。

![Hyperledger Fabric 单一成员区块链市场模板](./media/hyperledger-fabric-single-member-blockchain/marketplace-template.png)

## <a name="deployment"></a>部署

若要开始，请选择“Hyperledger Fabric 单一成员区块链”，然后单击“创建”。 这将在向导中打开“基础”边栏选项卡。

模板部署将引导你配置多节点网络。 部署流分为三个步骤：基础、网络配置和 Fabric 配置。

### <a name="basics"></a>基础

在“基础”边栏选项卡中，为全部部署指定标准参数的值，例如订阅、资源组和基本虚拟机属性。

![基础](./media/hyperledger-fabric-single-member-blockchain/basics.png)

参数名称| 说明| 允许的值|默认值
---|---|---|---
**资源前缀**| 用作已部署资源的命名基础的字符串。|6 个字符或更少|NA
**VM 用户名**| 为此成员部署的每个虚拟机的管理员用户名。|1 - 64 个字符|azureuser
**身份验证类型**| 对虚拟机进行身份验证的方法。|密码或 SSH 公钥|密码
**密码（身份验证类型 = 密码）**|部署的每个虚拟机的管理员帐户密码。 密码必须包含下列要求中的 3 项：1 个大写字符、1 个小写字符、1 个数字和 1 个特殊字符。<br /><br />虽然所有 VM 最初都有相同的密码，但可以在预配后更改密码。|12 - 72 个字符|NA
**SSH 密钥（身份验证类型 = 公钥）**|用于远程登录的安全 shell 密钥。||NA
**按 IP 地址限制访问**|用于确定是否限制客户端终结点访问类型的设置。|Yes/No| No
**允许的 IP 地址或子网（按 IP 地址限制访问 = 是）**|启用访问控制时，允许访问客户端终结点的 IP 地址或IP 地址集。||NA
**订阅** |要部署的订阅。
**资源组** |部署联盟网络的资源组。||NA
**位置** |部署第一个成员的网络占用文件的 Azure 区域。

### <a name="network-size-and-performance"></a>网络规模和性能

接下来，在“网络规模和性能”下指定联盟网络规模的输入值，例如成员身份、排序和对等节点的数量。 选择基础结构选项和虚拟机大小。

![网络规模和性能](./media/hyperledger-fabric-single-member-blockchain/network-size-performance.png)

参数名称| 说明| 允许的值|默认值
---|---|---|---
**成员资格节点数**|运行成员资格服务的节点数。 有关成员资格服务的其他详细信息，请查看 Hyperledger [文档](https://media.readthedocs.org/pdf/hyperledger-fabric/latest/hyperledger-fabric.pdf)的安全性与成员资格服务。<br /><br />此值目前限制为 1 个节点，但我们计划在下一个修订版本中通过群集来支持对其进行扩大。|1| 1
**排序节点数** |将事务排序（整理）为块的节点数。--> 此表述冗长且难以理解。 有关排序服务的其他详细信息，请访问 Hyperledger [文档](https://hyperledger-fabric.readthedocs.io/en/release-1.1/ordering-service-faq.html)。<br /><br />此值目前限制为 1 个节点。 |1 |1
**对等节点数**| 归联盟成员所有，执行事务并保持状态和账本副本的节点数。<br /><br />有关排序服务的其他详细信息，请访问 Hyperledger [文档](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html)。|3| 3 - 9
**存储性能**|支持每个已部署的节点的存储类型。 若要了解有关存储的详细信息，请访问 [Microsoft Azure 存储简介](https://docs.microsoft.com/azure/storage/common/storage-introduction)和[高级存储](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)。|标准或高级|标准
**虚拟机大小** |用于网络中所有节点的虚拟机大小|标准 A、<br />标准 D、<br />标准 D-v2、<br />标准 F 系列、<br />标准 DS <br />和标准 FS|标准 D1_v2

### <a name="fabric-specific-settings"></a>Fabric 特定设置

最后，在“Fabric 设置”下，指定与 Fabric 相关的配置设置。

![Fabric 设置](./media/hyperledger-fabric-single-member-blockchain/fabric-settings.png)

参数名称| 说明| 允许的值|默认值
---|---|---|---
**启动用户名**| 通过已部署网络中的成员服务注册的初始授权用户。|9 个字符或更少|admin
**Fabric CA 的启动用户密码**|用于保护导入成员身份节点的 Fabric CA 帐户的管理员密码。<br /><br />密码必须包含一个大写字符、一个小写字符和一个数字。|12 个字符或更多|NA

### <a name="deploy"></a>部署

在“摘要”边栏选项卡中，查看指定的输入并运行基本的部署前验证。

![摘要](./media/hyperledger-fabric-single-member-blockchain/summary.png)

查看法律和隐私条款，然后单击“购买”进行部署。 部署时间可能从几分钟到数十分钟不等，具体取决于正在预配的 VM 数量。

### <a name="accessing-nodes"></a>访问节点

部署完成后，将显示“概述”页。

![部署](./media/hyperledger-fabric-single-member-blockchain/deployments.png)

如果屏幕未自动显示（可能因为在运行部署时浏览管理门户），可始终在左侧导航栏的“资源组”选项卡中找到它。 单击在步骤 1 中输入的资源组名称，转到“概述”页。

“概述”页列出了解决方案模板部署的所有资源。 可随意进行浏览，但还可从此屏幕访问由该模板生成的_输出参数_。 在连接到 Hyperledger Fabric 网络时，这些输出参数可以提供非常有用的信息。

若要访问输出参数，请首先单击“资源组”边栏选项卡中的“部署”选项卡。 部署历史记录随即显示。

![部署历史记录](./media/hyperledger-fabric-single-member-blockchain/deployment-history.png)

从部署历史记录中，单击列表中的第一个部署来查看详细信息。

![部署详细信息](./media/hyperledger-fabric-single-member-blockchain/deployment-details.png)

详细信息屏幕将显示该部署的摘要信息，后接非常有用的三个输出参数：

- 在网络上部署应用程序后，即可使用 _API-ENDPOINT_。
- _PREFIX_，也称为_部署前缀_，唯一标识资源和部署。 使用基于命令行的工具时需使用它。
- _SSH-TO-FIRST-VM_ 提供预构建的 ssh 命令，并附带连接到网络中第一个 VM 所需的所有正确参数；在 Hyperledger Fabric 中为 Fabric-CA 节点。

可使用提供的管理员用户名和密码/SSH 密钥，通过 SSH 密钥远程连接到每个节点的虚拟机。 由于节点 VM 没有自己的公共 IP 地址，因此需要通过负载平衡器并指定端口号。 访问第一个事务节点的 SSH 命令为第三个模板输出 **SSH-TO-FIRST-VM（对于示例部署，则为：`sh -p 3000 azureuser@hlf2racpt.northeurope.cloudapp.azure.com`）。 要获取其他事务节点，请将端口号递增 1（例如，第一个事务节点位于端口 3000 上，第二个事务节点位于端口 3001 上，第三个事务节点位于端口 3002 上）。

## <a name="next-steps"></a>后续步骤

现在你可以专注于针对 Hyperledger 联盟区块链网络的应用程序和链式代码开发。
