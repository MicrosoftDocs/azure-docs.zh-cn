---
title: 安全性和数据隐私 - Azure 搜索
description: Azure 搜索符合 SOC 2、HIPAA 和其他认证。 在 Azure 搜索筛选器中通过用户和组安全标识符进行连接和数据加密、身份验证和标识访问。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: f366726f539a817f515a78fbc35bfeaa3b65514e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024500"
---
# <a name="security-and-data-privacy-in-azure-search"></a>Azure 搜索中的安全性和数据隐私

Azure 搜索中内置了全面的安全功能和访问控制，以确保私有内容保持原有状态。 本文列举了 Azure 搜索中内置的安全功能和标准符合性。

Azure 搜索安全体系结构跨越物理安全性、加密传输、加密存储和全平台标准符合性。 在操作上，Azure 搜索仅接受经过身份验证的请求。 可以选择通过安全筛选器针对内容添加基于用户的访问控制。 本文会涉及每个层的安全性，但侧重于有关如何在 Azure 搜索中保护数据和操作。

## <a name="standards-compliance-iso-27001-soc-2-hipaa"></a>标准符合性：ISO 27001、SOC 2、HIPAA

Azure 搜索针对以下标准进行了认证，如 [2018 年 6 月发布的公告](https://azure.microsoft.com/blog/azure-search-is-now-certified-for-several-levels-of-compliance/)所述。

+ [ISO 27001:2013](https://www.iso.org/isoiec-27001-information-security.html) 
+ [SOC 2 类型 2 符合性](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aicpasoc2report.html) 有关完整报告，请转到 [Azure - Azure 政府版 SOC 2 类型 II 报告](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports)。 
+ [健康保险可携性和责任法案 (HIPAA)](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)
+ [GxP（CFR 第 21 篇第·11 部分）](https://en.wikipedia.org/wiki/Title_21_CFR_Part_11)
+ [HITRUST](https://en.wikipedia.org/wiki/HITRUST)
+ [PCI DSS 1 级](https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)
+ [澳大利亚 IRAP 未分类 DLM](https://asd.gov.au/infosec/irap/certified_clouds.htm)

标准符合性应用于正式版功能。 预览版功能在转变为正式版时进行认证，不能用于具有严格标准要求的解决方案中。 符合性认证记录在 [Microsoft Azure 符合性概述](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942)和[信任中心](https://www.microsoft.com/en-us/trustcenter)中。 

## <a name="encrypted-transmission-and-storage"></a>加密的传输和存储

加密扩展到整个索引管道：从连接通过传输向下到存储在 Azure 搜索中的索引数据。

| 安全层 | 描述 |
|----------------|-------------|
| 传输中加密 <br>(HTTPS/SSL/TLS) | Azure 搜索在 HTTPS 端口 443 上侦听。 与 Azure 服务建立的跨平台连接经过加密。 <br/><br/>所有从客户端到服务的 Azure 搜索交互都支持 SSL/TLS 1.2。  请务必为你的服务的 SSL 连接使用 TLSv1.2。|
| 静态加密 <br>Microsoft 托管的密钥 | 加密在索引过程中完全进行内部化处理，而不会显著影响完成索引所需的时间或索引大小。 加密自动对所有索引进行，包括对未完全加密的索引（在 2018 年 1 月前创建）的增量更新。<br><br>在内部，加密基于 [Azure 存储服务加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)，使用 256 位 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)进行。<br><br> 加密在 Azure 搜索内部进行，证书和加密密钥由 Microsoft 内部管理，并广泛应用。 无法在门户中或以编程方式打开或关闭加密、管理或替换为自己的密钥，或者查看加密设置。<br><br>静态加密已于 2018 年 1 月 24 日宣布推出并应用于所有区域中的所有服务层级，包括共享（免费）服务。 对于完全加密，必须删除该日期之前创建的索引并重新生成，以便进行加密。 否则，仅对 1 月 24 日以后添加的新数据进行加密。|
| 静态加密 <br>客户管理的密钥 | 使用客户托管密钥的加密是**预览版**功能未免费提供的服务。 对于付费服务，它才可用于搜索服务上创建或年 1 月 2019年之后使用最新预览 api 版本 (api 版本 = 2019年-05-06-预览)。<br><br>Azure 搜索索引和同义词映射可以现在是静态加密与 Azure Key Vault 中客户托管的密钥密钥。 若要了解详细信息，请参阅[在 Azure 搜索管理加密密钥](search-security-manage-encryption-keys.md)。<br>此功能不会替代默认静态加密，但除了它而不是应用。<br>启用此功能将增加索引的大小，并会降低查询性能。 根据迄今为止观察结果，您有望看到查询时间提高 30%-60%的实际性能将会有所不同，具体取决于索引定义和类型的查询但。 由于这种性能影响，我们建议你仅启用此功能在真正需要的索引。

## <a name="azure-wide-user-access-controls"></a>Azure 范围的用户访问控制

整个 Azure 提供多种安全机制，因此，创建的 Azure 搜索资源也会自动获得这些安全机制。

+ [订阅或资源级别的锁可防止删除](../azure-resource-manager/resource-group-lock-resources.md)
+ [基于角色的访问控制 (RBAC) 可以控制对信息和管理操作的访问](../role-based-access-control/overview.md)

所有 Azure 服务支持使用基于角色的访问控制 (RBAC) 在不同的服务之间以一致的方式设置访问级别。 例如，仅限“所有者”和“参与者”角色查看敏感数据（如管理密钥），而任何角色的成员都可以查看服务状态。 RBAC 提供“所有者”、“参与者”和“读取者”角色。 默认情况下，所有服务管理员是“所有者”角色的成员。

<a name="service-access-and-authentication"></a>

## <a name="service-access-and-authentication"></a>服务访问和身份验证

虽然 Azure 搜索继承了 Azure 平台的安全保护，但它还提供其自己的基于密钥的身份验证。 API 密钥是随机生成的数字和字母所组成的字符串。 密钥的类型（管理员或查询）确定访问的级别。 提交有效密钥被视为请求源自受信任实体的证明。 

有两个搜索服务访问级别，可通过两种类型的密钥启用它们：

* 管理员访问（适用于服务的任何读写操作）
* 查询 （适用于只读操作，例如，对索引的文档集合的查询） 的访问权限

预配服务时会创建*管理员密钥*。 有两个管理密钥，分别指定为*主要*和*辅助*密钥以将它们保持在各自的位置，但事实上是可互换的。 每个服务都有两个管理密钥，以便在转换其中一个时不会丢失服务的访问权限。 你可以[重新生成管理密钥](search-security-api-keys.md#regenerate-admin-keys)定期每个 Azure 安全最佳实践，但您不能将添加到管理密钥总计数。 有最多为每个搜索服务的两个管理密钥。

*查询密钥*根据需要创建并设计用于客户端应用程序发出查询。 最多可以创建 50 个查询密钥。 在应用程序代码中，指定搜索 URL 和查询 api 密钥以允许对特定索引的文档集合的只读访问。 终结点、仅供只读访问的 API 密钥以及目标索引共同定义客户端应用程序连接的作用域和访问级别。

需要对每个请求进行身份验证，而每个请求由必需密钥、操作和对象组成。 链接在一起后，两个权限级别（完全或只读）加上上下文（例如，索引上的查询操作）便足以针对服务操作提供全面的安全性。 有关密钥的详细信息，请参阅[创建和管理 API 密钥](search-security-api-keys.md)。

## <a name="index-access"></a>索引访问

在 Azure 搜索中，单个索引不是安全对象。 对索引的访问权限是根据服务层（读取或写入访问）以及操作上下文确定的。

对于最终用户访问，可以使用查询密钥构建要连接的查询请求，这会将任何请求设置为只读，并包含应用使用的特定索引。 在查询请求中，没有联接索引或同时访问多个索引的概念，所有请求都会根据定义以单个索引为目标。 因此，查询请求本身的结构（密钥加上单个目标索引）定义了安全边界。

管理员和开发人员对索引的访问权限没有区别：两者都需要写访问权限才能创建、删除和更新服务管理的对象。 拥有服务管理密钥的任何人都可以读取、修改或删除同一服务中的任何索引。 为了防止意外删除或恶意删除索引，代码资产的内部源代码管理作为一种补救机制，可以还原意外的索引删除或修改。 Azure 搜索在群集中提供故障转移功能来确保可用性，但它不会存储或执行用于创建或加载索引的专属代码。

需要索引级安全边界的多租户解决方案通常包含一个中间层，客户可以使用它来处理索引隔离。 有关多租户用例的详细信息，请参阅[多租户 SaaS 应用程序与 Azure 搜索的设计模式](search-modeling-multitenant-saas-applications.md)。

## <a name="admin-access"></a>管理员访问权限

[基于角色的访问 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview)确定你是否有权访问控制服务和其内容。 如果是 Azure 搜索服务的所有者或参与者，则可以使用门户或 PowerShell **Az.Search**模块来创建、 更新或删除该服务的对象。 此外可以使用[Azure 搜索管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/search-howto-management-rest-api)。

## <a name="user-access"></a>用户访问权限

默认情况下，由查询请求中的访问密钥确定用户对索引的访问权限。 大部分开发人员会针对客户端搜索请求创建并分配[*查询密钥*](search-security-api-keys.md)。 查询密钥授予对索引内的所有内容的读取访问权限。

如果需要对内容进行精细的基于每个用户的控制，可以在查询中生成安全筛选器，返回与给定安全标识关联的文档。 基于标识的访问控制不是预定义的角色和角色分配，它作为*筛选器*实现，该筛选器可以根据标识修整文档和内容的搜索结果。 下表描述了修整未经授权内容的搜索结果的两种方法。

| 方法 | 描述 |
|----------|-------------|
|[基于标识筛选器的安全修整](search-security-trimming-for-azure-search.md)  | 阐述实现用户标识访问控制的基本工作流。 该工作流包括将安全标识符添加到索引，然后解释如何针对该字段进行筛选，以修整受禁内容的结果。 |
|[Azure Active Directory 标识的安全修整](search-security-trimming-for-azure-search-with-aad.md)  | 此文延伸了前一篇文章的内容，提供了有关从 Azure Active Directory (AAD)（Azure 云平台中的一个[免费服务](https://azure.microsoft.com/free/)）检索标识的步骤。 |

## <a name="table-permissioned-operations"></a>表：权限操作

下表汇总了 Azure 搜索中允许的操作，以及哪个密钥可以解锁特定操作的访问。

| Operation | 权限 |
|-----------|-------------------------|
| 创建服务 | Azure 订阅持有者|
| 缩放服务 | 管理密钥，资源中的 RBAC 所有者或参与者  |
| 删除服务 | 管理密钥，资源中的 RBAC 所有者或参与者 |
| 创建、修改、删除服务中的对象： <br>索引和组件部分（包括分析器定义、评分配置文件、CORS 选项）、索引器、数据源、同义词、建议器。 | 管理密钥，资源中的 RBAC 所有者或参与者  |
| 查询索引 | 管理密钥或查询密钥（RBAC 不适用） |
| 查询系统信息，例如返回统计信息、计数和对象列表。 | 管理密钥，资源的 RBAC（所有者、参与者、读取者） |
| 管理管理密钥 | 管理密钥，资源中的 RBAC 所有者或参与者。 |
| 管理查询密钥 |  管理密钥，资源中的 RBAC 所有者或参与者。  |

## <a name="physical-security"></a>物理安全性

Microsoft 数据中心提供行业领先的物理安全性，符合广泛的标准和法规要求。 若要了解详细信息，请转到[全球数据中心](https://www.microsoft.com/cloud-platform/global-datacenters)页，或观看有关数据中心安全性的简短视频。

> [!VIDEO https://www.youtube.com/embed/r1cyTL8JqRg]


## <a name="see-also"></a>另请参阅

+ [.NET 入门（演示如何使用管理密钥创建索引）](search-create-index-dotnet.md)
+ [REST 入门（演示如何使用管理密钥创建索引）](search-create-index-rest-api.md)
+ [使用 Azure 搜索筛选器进行基于标识的访问控制](search-security-trimming-for-azure-search.md)
+ [使用 Azure 搜索筛选器进行 Active Directory 基于标识的访问控制](search-security-trimming-for-azure-search-with-aad.md)
+ [Azure 搜索中的筛选器](search-filters.md)