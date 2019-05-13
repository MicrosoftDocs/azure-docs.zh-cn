---
title: Azure Cosmos DB 中的静态加密
description: 了解 Azure Cosmos DB 如何提供静态数据加密及其实现方式。
author: rimman
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: 49fe9c0921579802b726e02a074bb0a911d88231
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204194"
---
# <a name="data-encryption-in-azure-cosmos-db"></a>Azure Cosmos DB 中的数据加密 

“静态加密”这个短语通常指非易失性存储设备（例如固态硬盘 (SSD) 和机械硬盘 (HDD)）上的数据的加密。 Cosmos DB 将其主数据库存储在 SSD 上。 其媒体附件和备份存储在通常由 HDD 备份的 Azure Blob 存储中。 随着 Cosmos DB 的静态加密功能的发布，所有数据库、媒体附件以及备份都是加密的。 现在数据在传输中（通过网络）以及在静态时（非易失性存储）都处于加密状态，从而实现端到端加密。

作为一种 PaaS 服务，Cosmos DB 非常易于使用。 由于 Cosmos DB 中存储的所有用户数据都会进行静态加密和传输中加密，因此你无需采取任何措施。 实现这一理念的另一方式是默认“开启”静态加密。 没有任何控件可以关闭或打开它。 Azure Cosmos DB 帐户正在其中运行的所有区域上使用 AES-256 加密。 我们在提供此功能的同时也会遵守[可用性和性能 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db)。

## <a name="implementation-of-encryption-at-rest-for-azure-cosmos-db"></a>实现针对 Azure Cosmos DB 的静态加密

静态加密是使用许多安全技术实现的，其中包括安全的密钥存储系统、加密的网络以及加密 API。 对数据进行解密和处理的系统必须与管理密钥的系统通信。 下图展示了加密数据的存储和密钥的管理是如何隔离的。 

![设计图](./media/database-encryption-at-rest/design-diagram.png)

用户请求的基本流程如下：
- 用户数据库帐户准备就绪以后，通过向管理服务资源提供程序发出请求来检索存储密钥。
- 用户通过 HTTPS/安全传输创建与 Cosmos DB 的连接。 （SDK 将详细信息抽象化。）
- 用户通过之前创建的安全连接发送要存储的 JSON 文档。
- 为 JSON 文档编制索引，除非用户已关闭索引编制功能。
- 将 JSON 文档和索引数据都写入到安全存储中。
- 定期从安全存储中读取数据并将其备份到 Azure 加密 Blob 存储中。

## <a name="frequently-asked-questions"></a>常见问题

### <a name="q-how-much-more-does-azure-storage-cost-if-storage-service-encryption-is-enabled"></a>问：如果启用存储服务加密，需另付多少 Azure 存储费用？
答：没有任何额外费用。

### <a name="q-who-manages-the-encryption-keys"></a>问：加密密钥由谁管理？
答：密钥由 Microsoft 管理。

### <a name="q-how-often-are-encryption-keys-rotated"></a>问：加密密钥多久轮换一次？
答：Microsoft 有一套关于加密密钥轮换的内部指导，Cosmos DB 按该指导执行。 未发布具体的指导原则。 Microsoft 发布了[安全开发生命周期 (SDL)](https://www.microsoft.com/sdl/default.aspx)，可以将其视为内部指导原则的一部分，其中提供的最佳做法对开发人员很有用。

### <a name="q-can-i-use-my-own-encryption-keys"></a>问：我可以使用自己的加密密钥吗？
答：Cosmos DB 是一项 PaaS 服务，我们一直在努力使该服务易于使用。 我们注意到此问题经常被作为一个与满足符合性要求（例如 PCI-DSS）相关的代理问题问起。 在构建此功能时，我们一直与符合性审核机构合作，确保使用 Cosmos DB 的客户满足相关要求，不需要客户自己管理密钥。

### <a name="q-what-regions-have-encryption-turned-on"></a>问：哪些区域已开启了此加密？
答：所有 Azure Cosmos DB 区域都已针对所有用户数据开启了此加密。

### <a name="q-does-encryption-affect-the-performance-latency-and-throughput-slas"></a>问：加密是否会影响性能延迟和吞吐量 SLA？
答：现已为所有现有帐户和新帐户启用了静态加密，对性能 SLA 没有影响或更改。 可以在 [Cosmos DB SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) 页上阅读详细信息，查看最新保证。

### <a name="q-does-the-local-emulator-support-encryption-at-rest"></a>问：本地模拟器是否支持静态加密？
答：模拟器是独立的开发/测试工具，不使用托管 Cosmos DB 服务使用的密钥管理服务。 建议在要存储敏感的模拟器测试数据的驱动器上启用 BitLocker。 [模拟器支持更改默认数据目录](local-emulator.md)以及使用已知位置。

## <a name="next-steps"></a>后续步骤

有关 Cosmos DB 安全性和最新改进的概述，请参阅 [Azure Cosmos DB 数据库安全性](database-security.md)。
有关 Microsoft 认证的详细信息，请参阅 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/)。
