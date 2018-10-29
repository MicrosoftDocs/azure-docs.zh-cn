---
title: Ethereum 工作量证明联盟解决方案模板
description: 使用 Ethereum 作量证明联盟解决方案模板部署和配置多成员联盟 Ethereum 网络
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 5/21/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: 823bea9bac8ff270d5b5c02e3b76a2f7236c9c99
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240633"
---
# <a name="ethereum-proof-of-work-consortium-solution-template"></a>Ethereum 工作量证明联盟解决方案模板

Ethereum 工作流证明联盟解决方案模板旨在利用最少的 Azure 和 Ethereum 知识使部署和配置多成员联盟 Ethereum 网络变得更加轻松、快捷。

通过 Azure 门户中的少量用户输入和一键式部署，每个成员都可以使用全球范围内的 Microsoft Azure 计算、网络和存储服务来设置其网络内存占用。 每个成员的网络内存占用由一组负载均衡事务节点（应用程序或用户可以通过它们交互来提交事务）、一组记录事务的挖矿节点以及一个 VPN 网关组成。 随后的连接步骤连接网关以创建完全配置的多成员区块链网络。

## <a name="about-blockchain"></a>关于区块链

对于那些刚刚进入区块链社区的人来说，该解决方案的发布是在 Azure 上以简单的可配置方式了解该技术的绝佳机会。 但是，首先我们建议在构建多成员联盟网络之前，按照此指导演练部署并简化独立 Ethereum 网络拓扑结构。

### <a name="mining-node-details"></a>挖矿节点详细信息

挖矿事务的节点与接受事务的节点分离，以确保这两个操作不会竞争相同的资源。

一个联盟成员最多可以设置为五个区域，其中包含一个或多个挖矿节点，并由托管磁盘支持。 该区域中的一个或多个节点配置为引导节点，以支持网络中节点的动态发现功能。 挖矿节点与其他挖矿节点进行通信以就基础分布式账本的状态达成共识。 应用程序无需了解这些节点或与这些节点通信。 通过专注于专用网络，挖矿节点与入站公共 Internet 流量隔离，以添加第二层保护。 允许出站流量，但不允许 Ethereum 发现端口。

所有节点都有 Go Ethereum (Geth) 客户端的稳定版本，并配置为挖矿节点。 如果你没有提供自定义起源块，则所有节点都使用 Ethereum 帐户密码保护的相同 Ethereum 地址和密钥对。 你提供的 Ethereum 密码用于为每个挖矿节点生成默认帐户 (coinbase)。 作为我的挖矿节点，它们收集添加到此帐户的费用。

每个联盟成员的挖矿节点数量取决于所需网络的总体规模以及专用于每个成员的哈希算力的量。 网络规模越大，需要进行泄漏以获得不公平优势的节点就越多。 该模板使用虚拟机规模集支持每个区域最多设置 15 个挖矿节点。

### <a name="transaction-node-details"></a>事务节点详细信息

联盟成员还具有一组负载均衡事务节点。 这些节点可从虚拟网络外部访问，以便应用程序可以使用这些节点提交事务或执行区块链网络中的智能协定。 所有节点都有 Go Ethereum (Geth) 客户端的稳定版本，并配置为维护分布式账本的完整副本。 如果未提供自定义起源块，则这些节点使用由 Ethereum 帐户密码保护的相同 Ethereum 帐户。

事务节点在可用性集内进行负载平衡以保持高可用性。 该模板使用虚拟机规模集支持最多设置 5 个事务节点。

### <a name="log-analytics-details"></a>Log Analytics 详细信息

每个部署还会创建一个新的 Log Analytics 实例或可以加入现有实例。 该操作允许监视组成部署网络的每个虚拟机的各种性能指标。

## <a name="deployment-architecture"></a>部署体系结构

### <a name="description"></a>Description

此解决方案模板可以部署单个或多个基于区域的多成员 Ethereum 联盟网络。 每个区域的虚拟网络使用 VNET 网关和连接资源以链形拓扑连接到其他区域。 它还设置了一个注册器，其中包含每个区域内部署的所有挖矿程序和事务节点所需的信息。

### <a name="standalone-and-consortium-leader-overview"></a>独立应用程序和联盟领导者概述

![独立应用程序和联盟领导者概述](./media/ethereum-deployment/leader-overview.png)

### <a name="joining-consortium-member-overview"></a>加入联盟成员概述

![加入联盟成员概述](./media/ethereum-deployment/member-overview.png)

## <a name="getting-started"></a>入门

此过程需要支持部署多个虚拟机规模集和托管磁盘的 Azure 订阅。 如有必要，请从创建一个免费的 Azure 帐户开始。

获得订阅后，请转到 Azure 门户。 选择“+ 创建资源”、“市场(查看全部)”，并搜索“Ethereum 工作量证明联盟”。

模板部署将引导你在网络中配置第一个成员的内存占用。 部署流程分为五个步骤：基础、Operations Management Suite、部署区域、网络规模和性能、Ethereum 设置。

### <a name="basics"></a>基础

在“基础”部分，为全部部署指定标准参数的值，例如订阅、资源组和基本虚拟机属性。

![基础](./media/ethereum-deployment/sample-deployment.png)

参数名称|说明| 允许的值|默认值
---|---|---|---
新建网络还是加入现有网络？|新建网络或加入预先存在的联盟网络|新建 加入现有|新建
部署一个将成为联盟组成部分的网络？|联盟网络允许未来的部署加入此网络（在上面选择“新建”时可见）|独立联盟|独立
资源前缀 |用作命名资源基础的字符串（2 至 4 个字母数字字符）。 对于某些资源，字符串前面会追加唯一哈希，同时追加特定资源信息。|长度为 2 到 4 的字母数字字符|NA
VM 用户名| 部署的每个 VM 的管理员用户名（仅限字母数字字符）|1 - 64 个字符 |gethadmin
身份验证类型|对虚拟机进行身份验证的方法。 |密码或 SSH 公钥|密码
密码（身份验证类型 = 密码）|部署的每个虚拟机的管理员帐户密码。 密码必须包含下列要求中的 3 项: 1 个大写字符，1 个小写字符，1 个数字和 1 个特殊字符。 <br />虽然所有 VM 最初都有相同的密码，但你可以在预配后更改密码。|12 - 72 个字符|NA
SSH 密钥（身份验证类型 = 公钥）|用于远程登录的安全 shell 密钥。|| NA
订阅| 部署联盟网络的订阅||NA
资源组| 部署联盟网络的资源组。||NA
位置| 资源组的 Azure 区域。 ||NA



### <a name="operations-management-suite"></a>Operations Management Suite

Operations Management Suite (OMS) 边栏选项卡允许你为网络配置 OMS 资源。 OMS 将从网络收集并显示有用的指标和日志，从而提供快速检查网络运行状况或调试问题的功能。 一旦达到容量，OMS 的免费产品将失败。

![新建 OMS](./media/ethereum-deployment/new-oms.png)

参数名称|说明| 允许的值|默认值
---|---|---|---
连接到现有 OMS|新建 Log Analytics 实例或加入现有实例|新建 加入现有|新建 Log Analytics 位置|将部署新 Log Analytics 的区域（如果选择“新建”，则可见）
现有 OMS 工作区 ID|现有实例 OMS 服务层的工作区 ID（如果选择“加入现有”，则可见）|为新实例选择定价层。 有关详细信息，请参阅 https://azure.microsoft.com/pricing/details/log-analytics/ （如果选择“加入现有”，则可见）|每个节点免费独立|免费
现有的 OMS 主键|用于连接到现有 OMS 实例的主键（如果选择“加入现有”，则可见）

### <a name="deployment-regions"></a>部署区域

接下来，在“部署区域”下，为“区域数量”指定输入，以根据给定的区域数量部署联盟网络及选择 Azure 区域。 用户最多可在五个区域中进行部署。

![部署区域](./media/ethereum-deployment/deployment-regions.png)

参数名称| 说明| 允许的值 |默认值
---|---|---|---
区域数量| 部署联盟网络的区域数量|1、2、3、4、5| 2
第一个区域| 部署联盟网络的第一个区域|所有允许的 Azure 区域| 美国西部
第二个区域 |部署联盟网络的第二个区域（仅在选择的区域数量为 2 时可见）|所有允许的 Azure 区域| 美国东部
第三个区域| 部署联盟网络的第三个区域（仅在选择的区域数量为 3 时可见）|所有允许的 Azure 区域| 美国中部
第四个区域| 部署联盟网络的第四个区域（仅在选择的区域数量为 4 时可见）|所有允许的 Azure 区域| 美国东部 2
第五个区域| 部署联盟网络的第五个区域（仅在选择的区域数量为 5 时可见）|所有允许的 Azure 区域| 美国西部 2

### <a name="network-size-and-performance"></a>网络规模和性能

接下来，在“网络规模和性能”下指定联盟网络规模的输入值，例如挖矿节点和事务节点的数量和大小。

![网络规模和性能](./media/ethereum-deployment/network-size-performance.png)

参数名称 |说明 |允许的值| 默认值
---|---|---|---
挖矿节点的数量|每个区域部署的挖矿节点的数量|2 - 15| 2
挖矿节点存储性能|支持每个已部署挖矿节点的托管磁盘的类型。|标准或高级|标准
挖矿节点虚拟机大小|用于挖矿节点的虚拟机大小。|标准 A、 <br />标准 D、 <br />标准 D-v2、 <br />标准 F 系列、 <br />标准 DS  <br />和标准 FS|标准 D1v2
负载平衡事务节点数|设置为网络组成部分的交易节点的数量。|1 - 5| 2
事务节点存储性能|支持每个已部署事务节点的托管磁盘的类型。|标准或高级|标准
事务节点虚拟机大小|用于事务节点的虚拟机大小。|标准 A、 <br />标准 D、 <br />标准 D-v2、 <br />标准 F 系列、 <br />标准 DS  <br />和标准 FS|标准 D1v2

### <a name="ethereum-settings"></a>Ethereum 设置

接下来，在“Ethereum 设置”下，指定 Ethereum 相关配置设置，例如网络 ID、Ethereum 帐户密码或起源块。

![Ethereum 设置](./media/ethereum-deployment/standalone-leader-deployment.png)

参数名称 |说明 |允许的值|默认值
---|---|---|---
ConsortiumMember ID|与参与联盟网络的每个成员相关联的 ID，用于配置 IP 地址空间以避免冲突。 <br /><br />成员 ID 在同一网络中的不同组织之间应该是唯一的。 即使同一个组织部署到多个区域，也需要一个唯一的成员 ID。<br /><br />记下此参数的值，因为你需要与其他加入成员共享它。|0 - 255
Ethereum 网络 ID|要部署的 Ethereum 联盟网络的网络 ID。 每个 Ethereum 网络都有自己的网络 ID，其中 1 是公共网络的 ID。 尽管挖矿节点的网络访问受到限制，但我们仍建议使用大量挖矿节点来防止冲突。|5 - 999,999,999| 10101010
自定义起源块|自动生成起源块或提供自定义块的选项。|Yes/No| 否
Ethereum 帐户密码（自定义起源块 = 否）|用于保护导入到每个节点的 Ethereum 帐户的管理员密码。 密码必须包含以下内容：1 个大写字符、1 个小写字符和 1 个数字。|12 个字符或更多|NA
Ethereum 私钥密码（自定义起源块 = 否）|用于生成与生成的默认 Ethereum 帐户关联的 ECC 私钥的密码。 预生成的私钥不需要显式传入。<br /><br />考虑使用具有足够随机性的密码，以确保私钥强度强，并且不会与其他联盟成员重叠。 密码必须至少包含以下内容：1 个大写字符、1 个小写字符和 1 个数字。<br /><br />请注意，如果两个成员使用相同的密码，则所生成的帐户将相同。 如果单个组织试图跨区域部署并希望跨所有节点共享一个帐户 (coin base)，则相同的密码会很有用。|12 个字符或更多|NA
起源块（自定义起源块 = 是）|表示自定义起源块的 JSON 字符串。 你可以在自定义网络下找到有关起源块格式的详细信息。<br /><br />Ethereum 帐户仍在提供自定义起源块时创建。 考虑在起源块中指定一个预先配置好的 Ethereum 帐户，以便无需等待挖矿。|有效的 JSON |NA
用于连接的共享密钥|用于 VNET 网关之间连接的共享密钥。| 12 个字符或更多|NA
联盟数据 URL|指向由其他成员的部署提供的相关联盟配置数据的 URL。 <br /><br />此信息由已进行部署的已连接成员提供。 如果你部署了网络的其余部分，则 URL 是模板部署输出，名为 CONSORTIUM-DATA。||NA
要连接到的 VNet 网关|要连接的 VNet 网关的资源路径。<br />此信息由已进行部署的已连接成员提供。 如果你部署了网络的其余部分，则 URL 位于模板部署输出中，名为 CONSORTIUM_MEMBER_GATEWAY_ID。 注意：必须使用同一成员的联盟数据 URL 和 VNet 网关资源。||NA
对等信息注册机构的终结点|其他成员部署提供的对等信息终结点|联盟中首个成员的有效终结点|NA
对等信息注册机构的密钥|其他成员部署提供的对等信息主键|联盟中首个成员的有效主键|NA

### <a name="summary"></a>摘要

单击“摘要”边栏选项卡查看指定的输入并运行基本的部署前验证。

![摘要](./media/ethereum-deployment/summary.png)

查看法律和隐私条款，然后单击“购买”进行部署。 如果部署有多个区域，或针对联盟网络进行部署，则该模板将预部署必要的 VPN 网关以支持与其他成员的网络连接。 部署网关可能需要多达 45 到 50 分钟。

## <a name="post-deployment-sanity-checks"></a>部署后健全性检查

### <a name="administrator-page"></a>管理员页

一旦部署成功完成并且所有资源均已配置，你可以转到管理员页面查看区块链网络以及部署状态的健全性检查。 管理页面的 URL 是负载均衡器的 DNS 名称，它存在于名为 ADMIN_SITE 的模板部署的输出部分中。

若要查找它，选择已部署的资源组。 然后，选择“概述”，单击“部署”正下方显示已成功的编号的链接。

![资源组概述](./media/ethereum-deployment/resource-overview.png)

新屏幕显示部署历史记录。 选择第一个部署资源（例如，microsoft-azure-blockchain.azure-blockchain-servi...），然后查找页面下半部分的“输出”部分。 你将看到管理页面的 URL 在模板部署输出参数中作为 ADMIN_SITE 列出。

![资源部署](./media/ethereum-deployment/resource-deployments.png)

![部署输出](./media/ethereum-deployment/deployment-output.png)

若要转到管理页，请复制 ADMIN-SITE 输出并在另一个选项卡中打开它。

在管理页面上，你可以通过查看 Ethereum 节点状态部分获得所部署拓扑的高级概述。 它包括所有节点主机名，它们的对等计数和所看到的最新块。 每个节点的对等计数在最小值小于总节点计数和配置的最大对等计数之间。 请注意，对等计数不会限制可以在网络中部署的节点的数量。
有时候，你会看到对等计数波动并且小于（节点总数 - 1）。 计数的差异并不总是表明节点运行状况不良，因为账本中的分支可能会导致对等计数的轻微变化。 最后，你可以检查网络中每个节点看到的最新块，以确定系统中的分支或延迟。

![节点状态](./media/ethereum-deployment/node-status.png)

节点状态每 10 秒刷新一次。 通过浏览器重新加载页面或“重新加载”按钮来更新视图。

### <a name="oms-portal"></a>OMS 门户

你可以通过遵循部署输出 (OMSPORTALURL) 中的链接或通过选择已部署的资源组中的 OMS 资源来找到你的 OMS 门户。

![OMS URL](./media/ethereum-deployment/oms-url.png)

![OMS 资源](./media/ethereum-deployment/oms-resource.png)

门户将首先将高级网络统计信息作为概述显示。

![OMS 概述](./media/ethereum-deployment/oms-overview.png)

单击概述将重定向到门户，以查看每个节点统计信息。

![每个节点统计信息](./media/ethereum-deployment/per-node-stats.png)

### <a name="accessing-nodes"></a>访问节点

你可以使用所提供的管理员用户名和密码/SSH 密钥，通过 SSH 密钥远程连接到交易节点的虚拟机。 由于事务节点 VM 没有自己的公共 IP 地址，因此需要通过负载平衡器并指定端口号。 运行以访问第一个事务节点的 SSH 命令在模板部署输出参数中列为 SSH_TO_FIRST_TX_NODE（用于示例部署：ssh -p 4000 gethadmin@leader4vb.eastus.cloudapp.azure.com）。 要获得其他事务节点，请将端口号增加 1（例如，第一个事务节点位于端口 4000 上）。

由于运行挖矿节点的虚拟机不能从外部访问，因此你需要通过其中一个事务节点。 一旦与事务节点建立了 SSH 会话，请将私钥安装在事务节点上或使用密码将 SSH 会话启动到任何挖矿节点中。

**注意**

主机名可以从管理站点或从 Azure 门户获得。 在 Azure 门户中，虚拟机规模集 (VMSS) 资源中存在的节点的主机名在“实例”下列出，它与实际主机名不同。 例如，Azure 门户中的主机名可能类似于 mn-asdfmv-reg1_0，，但实际主机名与 mn-asdfmv-reg 相似。

例如：

Azure 门户主机名| 实际主机名
---|---
mn-ethwvu-reg1_0| mn-ethwvu-reg1000000
mn-ethwvu-reg1_1 |mn-ethwvu-reg1000001
mn-ethwvu-reg1_2 |mn-ethwvu-reg1000002

## <a name="adding-a-new-consortium-member"></a>添加新联盟成员

### <a name="sharing-data"></a>共享数据

作为联盟的第一名成员（或已连接的成员），你需要向其他成员提供几条信息，以便他们可以加入并建立其连接。 具体而言：

1. 共享联盟配置数据：有一组用于组织两个成员之间 Ethereum 连接的数据。 将必要的信息（包括起源块、联盟网络 ID 和引导节点）写入到领导者或其他已部署成员的事务节点上的文件中。 该文件的位置在名为 CONSORTIUM-DATA 的模板部署输出参数中列出。
2. 对等信息终结点：对等信息注册器终结点用于从领导者或其他成员的部署中获取已连接到 Ethereum 网络的所有节点的信息。 DB 存储关于网络中每个已连接的节点的一组信息，如节点的主机名、专用 IP 地址等信息。这是名为 PEER_INFO_ENDPOINT 的模板部署输出参数。
3. 对等信息主键：对等信息注册器主键用于访问领导者或其他成员的对等信息主键。 这是名为 PEER_INFO_PRIMARY_KEY 的模板部署输出参数。


4. VNET 网关：每个成员通过现有成员建立到整个区块链网络的连接。 要连接 VNET，你需要到要连接的成员的 VNET 网关的资源路径。 这是名为 CONSORTIUM_MEMBER_GATEWAY_ID 的模板部署输出参数。
5. 共享密钥：联盟网络中两个正在建立连接的成员之间预先建立的机密。 这是在部署上下文之外同意的字母数字字符串（1 到 128 个字符之间）。 （例如，MySharedKeyAbc123）

### <a name="acceptance-of-new-member"></a>接受新成员

此步骤应在加入的成员成功部署其网络后完成。 在成员可以加入网络并查看事务流量之前，现有成员必须在其 VPN 网关上执行最终配置才能接受此连接。 这意味着连接成员的 Ethereum 节点在建立连接前不会运行。 此配置可以通过 PowerShell 或 xPlat CLI 完成。 PowerShell 模块和 xPlat CLI 脚本也与联盟数据一起存储在事务节点上。 脚本位置分别是名为 PAIR-GATEWAY-PS-MODULE 和 PAIR-GATEWAY-AZURE-CLISCRIPT 的部署输出参数。

PowerShell/CLI 安装程序

有关如何开始使用 Azure PowerShell cmdlet 和 Azure xPlat CLI 的其他信息可以在 Azure 文档中找到。

你将需要在本地安装最新版本的 Azure cmdlet，并打开一个会话。 确保使用 Azure 订阅凭证登录会话。

PowerShell：建立连接

下载 PowerShell 模块并将其存储在本地。 PowerShell 模块的位置指定为 PAIR-GATEWAY-PS-MODULE 模板部署输出参数。

如果尚未启用，请使用本地会话的 Set-ExecutionPolicy cmdlet，以允许运行未签名的模块。

Set-ExecutionPolicy Unrestricted CurrentUser

接下来，登录你部署领导者部署所使用的 Azure 订阅

Login-AzureRmAccount

接下来，导入该模块：

Import-Module <filepath to downloaded file>

最后，使用相应的输入运行该函数：

- MyGatewayResourceId：网关的资源路径。 这是名为 CONSORTIUM_MEMBER_GATEWAY_ID 的模板部署输出参数。
- OtherGatewayResourceId：加入成员的网关的资源路径。 这由加入成员提供，并且是也称为 CONSORTIUM_MEMBER_GATEWAY_ID 的模板部署输出参数。
- ConnectionName：用于识别此网关连接的名称。
- 共享密钥：联盟网络中两个正在建立连接的成员之间预先建立的机密。

CreateConnection - MyGatewayResourceId <resource path of your Gateway> -OtherGatewayResourceId <加入成员网关的资源路径> -ConnectionName myConnection -SharedKey "MySharedKeyAbc123"

xPlat CLI：建立连接

下载 Azure CLI 脚本并将其存储在本地。 Azure CLI 脚本的位置在名为 PAIR-GATEWAY-AZURE-CLI-SCRIPT 的模板部署参数中指定。

使用相应的输入运行脚本：

- MyGatewayResourceId：网关的资源路径。 这是名为 CONSORTIUM_MEMBER_GATEWAY_ID 的模板部署输出参数。
- OtherGatewayResourceId：加入成员的网关的资源路径。 这由加入成员提供，并且是其部署的模板部署参数，也称为 CONSORTIUM_MEMBER_GATEWAY_ID 。
- ConnectionName：用于识别此网关连接的名称。
- 共享密钥：联盟网络中两个正在建立连接的成员之间预先建立的机密。
- 位置：部署网关资源的 Azure 区域。

``` powershell
az network vpn-connection create --name $ConnectionName --resource-group
$MyResourceGroup --vnet-gateway1 $MyGatewayResourceId --shared-key $SharedKey --vnet-
gateway2 $OtherGatewayResourceId --enable-bgp
```

在成功配置“领导者”和“成员”部署之间的连接后，体系结构将如下所示。

![领导者和成员体系结构](./media/ethereum-deployment/leader-member.png)

## <a name="fund-new-member-account"></a>新成员帐户资金

### <a name="generated-genesis-block"></a>生成的起源块

作为第一个成员，如果部署生成起源块（自定义起源块 = 否），那么你的 Ethereum 帐户将获得 1 万亿 Ether 币。 其他成员将拥有一个未预先筹资的帐户，并且必须等待积累 Ether，因为他们的挖矿节点开始挖矿程序块。 为避免新成员等待获取 Ether，你需要显式资助加入成员的 Ethereum 帐户。

为此，加入成员必须向你提供在其管理员网站上显示的 Ethereum 帐户。 然后，通过输入提供的帐户，你可以使用你的管理员网站将 Ether 从你的帐户转移到他们的帐户。

### <a name="custom-genesis-block"></a>自定义起源块

如果为指定的 Ethereum 帐户提供自定义起源块，则可以使用 MetaMask 或其他工具将 ether 从指定帐户转移到管理员网站中预生成的可见 Ethereum 帐户。 有关如何使用 MetaMask 的说明，请参阅[创建 Ethereum 帐户](#create-ethereum-account)。

如果提供了没有帐户的自定义起源块，或者你无法访问任何预先分配的帐户，则在挖矿节点开始挖矿，以将 Ether 生成到你的帐户 (coin base) 前，你需要等待。 资金生成速度取决于你在自定义起源块中指定的难度级别。

## <a name="create-ethereum-account"></a>创建 Ethereum 帐户

要创建其他帐户，你可以使用各种解决方案。 其中一种解决方案是 MetaMask，它是一种 Chrome 扩展，提供标识保管库和到 Ethereum 网络、公共、测试或自定义的连接。 MetaMask 制定一项交易以在网络中注册帐户。

与其他任何事务一样，此事务将转到其中一个事务节点，并最终被挖矿到某个块中，如下所示。

![事务节点](./media/ethereum-deployment/transaction-node.png)

要在 Chrome 中安装扩展程序，请转到“自定义和控制 Google Chrome(溢出按钮)”>“更多工具”>“扩展程序”>“获取更多扩展程序”，然后搜索 MetaMask。

![MetaMask 扩展](./media/ethereum-deployment/metamask-extension.png)

安装完成后，打开 MetaMask 并创建新的保管库。 默认情况下，保管库将连接到新式测试网络。 你需要将其更改为连接到已部署的专用联盟网络，特别是连接到事务节点前面的负载均衡器。 从模板输出中，检索端口 8545 处已公开的 Ethereum RPC 终结点（名为 `ETHEREUM-RPC-ENDPOINT`），并将其输入到自定义 RPC 中，如下所示。

![MetaMask 设置](./media/ethereum-deployment/metamask-settings.png)

通过创建保管库，你可以创建一个包含帐户的钱包。 要创建其他帐户，请选择“切换帐户”，然后选择“+”按钮。

![MetaMask 创建帐户](./media/ethereum-deployment/metamask-create-account.png)

### <a name="initiate-initial-ether-allocation"></a>启动初始 Ether 分配

通过管理员页面，你可以制定一项事务，将 Ether 从预分配帐户转移至其他 Ethereum 帐户。 此 Ether 转移是发送到事务节点并挖矿到块中的事务，如下所示。

![事务节点](./media/ethereum-deployment/transaction-node-mined.png)

通过 MetaMask 钱包中的剪贴板图标，复制要将 ether 转移到的 Ethereum 帐户的地址，然后返回到管理员页面。 将复制的帐户粘贴到输入域中，以将 1000 ether 从预先分配的 Ethereum 帐户转移到新创建的帐户。 单击“提交”然后等待将事务挖矿到块中。

![节点状态](./media/ethereum-deployment/node-status2.png)

一旦事务提交到挖矿块中，MetaMask 帐户中的帐户余额将反映转移了 1000 Ether。

![MetaMask 转移](./media/ethereum-deployment/metamask-transfer.png)

### <a name="transfer-of-ether-between-accounts"></a>帐户之间的 Ether 转移

此时，你已准备好在专用联盟网络内执行事务。 最简单的事务是将 Ether 从一个帐户转移到另一个帐户。 为构建此类事务，你可以再次使用 MetaMask，将钱从上面使用的第一个帐户转移到第二个帐户。

从 MetaMask 的“钱包 1”中单击“发送”。 复制到创建到“收件人地址”输入域的第二个钱包地址以及转移到“金额”输入域的 Ether 数量。 单击“发送”并接受该事务。

![MetaMask 发送事务](./media/ethereum-deployment/metamask-send.png)

同样，当挖矿事务并将其提交到块时，帐户余额将相应地反映出来。 请注意，钱包 1 的余额会扣除超过 15 个 Ether，因为你必须支付挖矿费来处理事务。

![钱包 1](./media/ethereum-deployment/wallet1.png)

![钱包 2](./media/ethereum-deployment/wallet2.png)

## <a name="next-steps"></a>后续步骤

现在你可以专注于针对专用联盟区块链网络的应用程序和智能协定开发。
