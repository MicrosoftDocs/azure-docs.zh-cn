---
title: Azure 区块链服务概述
description: Azure 区块链服务概述
services: azure-blockchain
keywords: 区块链
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: janders
manager: femila
ms.openlocfilehash: a200649493354f1264afb0df4cf74acb4a274017
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406408"
---
# <a name="what-is-azure-blockchain-service"></a>什么是 Azure 区块链服务？

Azure 区块链服务是一个完全托管的分类帐服务，使用户能够增长并运行在 Azure 中大规模的区块链网络。 通过为基础结构管理和区块链网络监管提供统一的控制，Azure 区块链服务提供：

* 简单网络部署和操作
* 内置联盟管理
* 开发智能协定与熟悉的开发工具

Azure 区块链服务设计为支持多个分类帐协议。 目前，它提供了支持以太坊[仲裁](https://www.jpmorgan.com/Quorum)分类帐使用[IBFT](https://github.com/jpmorganchase/quorum/wiki/Quorum-Consensus)达成一致意见的机制。

这些功能几乎都不需要进行任何管理，并且都是在不另外收费的情况下提供的。 你可以专注于应用程序开发和业务逻辑，而不需要的时间和资源来管理虚拟机和基础结构。 此外，您可以继续开发的开源工具和平台，而无需学习新技能实现你的解决方案所选的应用程序。

## <a name="network-deployment-and-operations"></a>网络部署和操作

通过 Azure 门户、 Azure CLI，也可通过使用 Azure 区块链扩展 Visual Studio code，可以完成 Azure 区块链服务部署。  简化部署，包括事务和验证程序节点，为安全隔离和服务管理的存储的 Azure 虚拟网络的预配。  此外，在部署新的区块链成员时，用户还创建或加入联盟。  联盟链启用多个参与方在不同 Azure 订阅中的能够安全地共享区块链上相互通信。  此简化的部署可减少从几天到几分钟的区块链网络部署。

### <a name="performance-and-service-tiers"></a>性能和服务层

Azure 区块链服务提供两个服务层：*基本*并*标准*。 每个层提供不同的性能和功能来支持轻型开发和测试最多在大规模工作负荷缩放生产区块链部署。 这两个层包括至少一个事务节点和一个验证程序节点 (Basic) 或两个验证程序节点 （标准）。

![定价层](./media/overview/pricing-tiers.png)

提供两个验证程序节点，除了*标准*层提供的 2 *Vcore*有关事务和验证程序的每个节点而基本级别提供 1 vCore 配置。  通过提供有关事务和验证程序节点的 2 个 Vcore，1 个 vCore 可以将专用于仲裁分类帐的而其余 1 个 vCore 可以用于其他基础结构相关的服务，确保区块链工作负荷的生产环境的最佳性能。 有关定价详细信息的详细信息，请参阅[Azure 区块链服务定价](https://azure.microsoft.com/pricing/details/blockchain-service)。

### <a name="security-and-maintenance"></a>安全和维护

预配后应用第一个区块链成员，您可以将其他事务节点添加到您的成员。  默认情况下，事务节点保护通过防火墙规则，将需要配置为访问。  此外，事务的所有节点都加密通过 TLS 的动态数据。  保护事务节点访问权限，包括防火墙规则、 基本身份验证、 访问密钥，以及 Azure Active Directory 集成的存在多个选项。 有关详细信息，请参阅[配置事务节点](configure-transaction-nodes.md)并[配置 Azure Active Directory 访问](configure-aad.md)。

作为托管服务，Azure 区块链服务确保使用最新的主机操作系统系统和会计软件堆栈更新、 配置高可用性 （仅适用于标准层），消除了 DevOps 修补区块链成员的节点所需的传统 IaaS 区块链节点。  修补和更新的详细信息，请参阅[受支持的 Azure 区块链服务分类帐版本](ledger-versions.md)。

### <a name="monitoring-and-logging"></a>监视和日志记录

此外，Azure 区块链服务提供了丰富的指标通过 Azure Monitor 服务提供节点的 CPU、 内存和存储使用情况的见解以及事务和块挖掘，等区块链网络活动的有用见解事务队列深度以及活动连接。  可以自定义指标，以提供对区块链应用程序很重要的见解。  此外，可以通过使用户能够触发的操作，例如发送电子邮件或短消息、 运行逻辑应用，Azure 函数或将发送到自定义 webhook 的警报定义阈值。

![度量值](./media/overview/metrics.png)

通过 Azure Log Analytics，用户可以查看与仲裁分类帐中或其他重要信息，例如尝试连接到事务节点相关的日志。

## <a name="built-in-consortium-management"></a>内置联盟管理

在部署您的第一个区块链成员时，您可以加入或创建新的联盟。  联盟是用来管理公司治理和 transact 多方进程中的区块链成员之间的连接的逻辑组。  Azure 区块链服务提供了通过预定义的智能合同，确定在联盟中的操作成员可以执行的内置管理控制。  这些监管控件可以根据需要由联盟管理员自定义。 当创建新的联盟时，区块链成员是联盟的默认管理员，使你能够邀请以加入你联盟第三方。  仅当您已被以前的邀请，你可以加入联盟。  当加入联盟，区块链成员受到实施相应的服务公司的管理员管理控制。

![联盟管理](./media/overview/consortium.png)

可以通过 PowerShell 和 REST API 访问联盟管理操作，例如添加和删除从联盟的成员。 你可以以编程方式管理联盟使用通用接口而不是修改和提交基于密度的智能协定。 有关详细信息，请参阅[联盟管理](consortium.md)。

## <a name="develop-using-familiar-development-tools"></a>使用熟悉的开发工具进行开发

基于开放源代码仲裁以太坊分类帐，你可以开发适用于 Azure 区块链服务应用程序相同的方式与现有以太坊应用程序一样的方式。 使用领先的行业合作伙伴，Azure 区块链开发工具包 Visual Studio Code 扩展允许开发人员可以使用熟悉的工具，如 Truffle 套件来构建智能协定。 使用 Azure 区块链开发工具包扩展插件，开发人员可以创建或连接到和现有联盟，以便可以生成和部署您的智能合同，所有从一个 IDE。 使用 Visual Studio Code 的 Azure 区块链扩展插件，可以创建或连接到现有联盟，以便可以生成和部署智能合约一个 IDE 中的所有内容。 有关详细信息，请参阅[VS Code marketplace 中的 Azure 区块链开发工具包](https://aka.ms/vscodebcextension)并[Azure 区块链开发工具包用户指南](https://aka.ms/vscodebcextensionwiki )。

## <a name="support-and-feedback"></a>支持和反馈

需要帮助或有任何反馈？

* 请访问[Azure 区块链博客](https://azure.microsoft.com/blog/topics/blockchain/)， [Microsoft 技术社区](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)，并[Azure 区块链论坛](https://social.msdn.microsoft.com/Forums/home?forum=azureblockchain)。
* 若要提供反馈或请求新功能，请通过 [UserVoice](https://feedback.azure.com/forums/921130-azure-blockchain-service) 创建条目。

## <a name="next-steps"></a>后续步骤

若要开始，请尝试快速入门或了解从这些资源的更多详细信息。
* [创建使用 Azure 门户的区块链成员](create-member.md)或[创建区块链成员使用 Azure CLI]()
* 有关成本比较与计算器，请参阅[定价页](https://azure.microsoft.com/pricing/details/blockchain-service)。
* 生成你的第一个应用程序使用[Azure 区块链开发工具包](https://github.com/Azure-Samples/blockchain-devkit)
* Azure 区块链 VSCode 扩展[用户指南](https://github.com/Microsoft/vscode-azure-blockchain-ethereum/wiki)
