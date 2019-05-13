---
title: 层和 SKU 的服务限制 - Azure 搜索
description: 用于容量计划的服务限制以及请求和响应 Azure 搜索的最大限制。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 419c533aabd67637efa64777387c491dd890596e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024642"
---
# <a name="service-limits-in-azure-search"></a>Azure 搜索中的服务限制
最大限制对存储、 工作负荷，以及数量的索引、 文档和其他对象依赖于是否您[预配 Azure 搜索](search-create-service-portal.md)处**免费**，**基本**， **标准**，或**存储优化**定价层。

+ **免费**层是 Azure 订阅随附的多租户共享服务。

+ **基本**层为小规模生产工作负荷提供专用计算资源。

+ **标准**层在专用计算机上运行，在每个级别上都具有更多存储和处理容量。 标准层共有四个级别：S1、S2、S3 和 S3 HD。

+ **存储优化**具有更多的总存储、 存储带宽和内存比专用计算机上运行**标准**。 存储优化有两个级别：L1 和 L2

> [!NOTE]
> 当前以折扣价格，目的是收集反馈的测试和实验目的在预览版的形式提供存储优化服务层。 确定最终定价将在以后宣布这些层通常不可用时。 建议不要对生产应用程序使用这些层。

  S3 高密度 (S3 HD) 是针对特定工作负荷设计的：[多租户](search-modeling-multitenant-saas-applications.md)和大量的小索引（每个索引一百万个文档，每个服务三千个索引）。 此层未提供[索引器功能](search-indexer-overview.md)。 在 S3 HD 上，数据引入必须利用推送方式，使用 API 调用将数据从源推送到索引。 

> [!NOTE]
> 服务已在特定层上进行预配。 跳转层以获取容量涉及预配新的服务（不会就地升级）。 有关详细信息，请参阅[选择 SKU 或层](search-sku-tier.md)。 若要详细了解在已预配的服务内调整容量，请参阅[缩放查询和为工作负荷编制索引的资源级别](search-capacity-planning.md)。
>

## <a name="subscription-limits"></a>订阅限制
[!INCLUDE [azure-search-limits-per-subscription](../../includes/azure-search-limits-per-subscription.md)]

## <a name="storage-limits"></a>存储限制
[!INCLUDE [azure-search-limits-per-service](../../includes/azure-search-limits-per-service.md)]

<a name="index-limits"></a>

## <a name="index-limits"></a>索引限制

| 资源 | 免费 | 基本&nbsp;<sup>1</sup>  | S1 | S2 | S3 | S3&nbsp;HD | L1 | L2 |
| -------- | ---- | ------------------- | --- | --- | --- | --- | --- | --- |
| 最大索引数 |3 |5 或 15 |50 |200 |200 |每个分区 1000，或者每个服务 3000 |10 |10 |
| 每个索引的最大简单字段 |1000 |100 |1000 |1000 |1000 |1000 |1000 |1000 |
| 每个索引的最大的复杂集合字段 |40 |40 |40 |40 |40 |40 |40 |40 |
| 每个文档的所有复杂集合中的最大元素 |3000 |3000 |3000 |3000 |3000 |3000 |3000 |3000 |
| 复杂的字段的最大深度 |10 |10 |10 |10 |10 |10 |10 |10 |
| 每个索引最大[建议器](https://docs.microsoft.com/rest/api/searchservice/suggesters)数 |第 |1 |1 |1 |1 |1 |1 |第 |
| 每个索引的最大[计分配置文件](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)数 |100 |100 |100 |100 |100 |100 |100 |100 |
| 每个配置文件的最大函数数量 |8 |8 |8 |8 |8 |8 |8 |8 |

<sup>1</sup>在 2017 年 12 月之前创建的基本服务在索引上具有较低的限制 (而不是 15 5)。 基本层是唯一具有下限（每个索引 100 个字段）的 SKU。

<a name="document-limits"></a>

## <a name="document-limits"></a>文档限制 

自 2018 年 10 月起，在任何区域的任何可计费层（基本、S1，S2、S3、S3 HD）创建的任何新服务都不再有任何文档数限制。 虽然自 2017 年 11 月/12 月以来大多数区域的文档数量不受限制，但仍有五个区域继续实施文档数限制。 根据你创建搜索服务的时间和地点，你可能正在运行仍受文档数限制的服务。

若要确定你的服务是否具有文档数限制，请检查你的服务的概述页中的“使用情况”磁贴。 文档数可能不受限制，也可能基于层受限于某个限制。

  ![“使用情况”磁贴](media/search-limits-quotas-capacity/portal-usage-tile.png)

### <a name="regions-previously-having-document-limits"></a>以前具有文档数限制的区域

如果门户指示文档数限制，则你的服务要么是在 2017 年底之前创建的，要么是在使用容量较低的群集托管 Azure 搜索服务的数据中心上创建的：

+ 澳大利亚东部
+ 东亚
+ 印度中部
+ 日本西部
+ 美国中西部

对于具有文档数限制的服务，将应用以下上限：

|  免费 | 基本 | S1 | S2 | S3 | S3&nbsp;HD |
|-------|-------|----|----|----|-------|
|  10,000 |1&nbsp;100 万条 |每个分区 1500 万，或者每个服务 1 亿 8 千万 |每个分区 6 千万，或者每个服务 7 亿 2 千万 |每个分区 1 亿 2 千万，或者每个服务 14 亿 |每个索引 1 百万，或者每个分区 2 亿 |

如果你的服务具有阻止你的限制，请创建一个新服务，然后将所有内容重新发布到该服务。 没有任何机制可以在后台无缝地将你的服务重新预配到新硬件上。

> [!Note] 
> 对于 2017 年底后创建的 S3 高密度服务，已去除每个分区 2 亿个文档的限制，但每个索引 100 万个文档的限制保留。


### <a name="document-size-limits-per-api-call"></a>每个 API 调用的文档大小限制

调用索引 API 时的最大文档大小大约为 16 MB。

文档大小实际上是索引 API 请求主体大小的限制。 由于可以将包含多个文档的批次传递给索引 API，因此大小限制实际上取决于批次中的文档数。 对于具有单个文档的批次，最大文档大小是 16 MB JSON。

要保持较低的文档大小，务必将不可查询数据从请求中排除。 图像和其他二进制数据不可直接查询，并且不应存储在索引中。 要将不可查询的数据集成到搜索结果中，请定义用于存储资源的 URL 引用的不可搜索字段。

## <a name="indexer-limits"></a>索引器限制

最大运行时间存在是为了向余额和稳定性到服务作为一个整体，但更大的数据集可能需要更多索引的时间超出允许的最大值。 如果在允许的最长时间内无法完成索引作业，请尝试按计划运行。 计划程序将跟踪索引的状态。 如果计划的索引作业因某种原因而中断，则索引器可以在下一次计划运行时从它上次停止的位置重新开始。


| 资源 | 免费&nbsp;<sup>1</sup> | 基本&nbsp;<sup>2</sup>| S1 | S2 | S3 | S3&nbsp;HD&nbsp;<sup>3</sup>|L1 |L2 |
| -------- | ----------------- | ----------------- | --- | --- | --- | --- | --- | --- |
| 最大索引器数 |3 |5 或 15|50 |200 |200 |不适用 |10 |10 |
| 最大数据源数 |3 |5 或 15 |50 |200 |200 |不适用 |10 |10 |
| 最大技能集数<sup>4</sup> |3 |5 或 15 |50 |200 |200 |不适用 |10 |10 |
| 每次调用的最大索引编制负载 |10,000 个文档 |仅受最大文档的限制 |仅受最大文档的限制 |仅受最大文档的限制 |仅受最大文档的限制 |不适用 |无限制 |无限制 |
| 最小的计划 | 5 分钟 |5 分钟 |5 分钟 |5 分钟 |5 分钟 |5 分钟 |5 分钟 | 5 分钟 |
| 最长运行时间 <sup>5</sup> | 1-3 分钟 |24 小时 |24 小时 |24 小时 |24 小时 |不适用  |24 小时 |24 小时 |
| 认知搜索技能集的最长运行时间或具有图像分析的 blob 索引 <sup>5</sup> | 3-10 分钟 |2 小时 |2 小时 |2 小时 |2 小时 |不适用  |2 小时 |2 小时 |
| Blob 索引器：最大 blob 大小，MB |16 |16 |128 |256 |256 |不适用  |256 |256 |
| Blob 索引器：从 blob 中提取的内容的最大字符数 |32,000 |64,000 |4&nbsp;100 万条 |4&nbsp;100 万条 |4&nbsp;100 万条 |不适用 |4&nbsp;100 万条 |4&nbsp;100 万条 |

<sup>1</sup> 对于免费服务，对于 blob 源，索引器最长执行时间为 3 分钟；对于所有其他数据源，索引器最长执行时间为为 1 分钟。 用于 AI 索引到认知服务的调用，免费服务仅限于每日 20 免费事务处理事务指已成功通过扩充管道的文档。

<sup>2</sup>在 2017 年 12 月之前创建的基本服务对索引器、 数据源和技能集具有较低的限制 (而不是 15 5)。

<sup>3</sup> S3 HD 服务未包括索引器支持。

<sup>4</sup> 每个技能集最多拥有 30 项技能。

<sup>5</sup> 认知搜索工作负载和 Azure blob 索引中的 图像分析的运行时间比常规文本索引运行时间短。 图像分析和自然语言处理属于计算密集型，并且消耗了过多的可用处理能力。 减少运行时间，以便队列中的其他作业能够运行。  

## <a name="queries-per-second-qps"></a>每秒查询次数 (QPS)

每个客户必须独立制定 QPS 估计值。 索引大小和复杂性、查询大小和复杂性以及流量大小是 QPS 的主要决定因素。 当此类因素未知时，没有方法能提供有意义的估计值。

针对专用资源（基本层和标准层）上运行的服务进行计算时，估计值更可预测。 由于能够控制更多参数，因此可以更精确地评估 QPS。 有关如何方法估算的指南，请参阅 [Azure 搜索的性能和优化](search-performance-optimization.md)。

对于存储优化层中，应该会较低的查询吞吐量和更高的延迟比标准层。  用于估计您会遇到的查询性能的方法是与标准层相同。

## <a name="data-limits-cognitive-search"></a>数据限制（认知搜索）

用于调用文本分析资源进行[实体识别](cognitive-search-skill-entity-recognition.md)、[关键短语提取](cognitive-search-skill-keyphrases.md)、[情绪分析](cognitive-search-skill-sentiment.md)、[语言检测](cognitive-search-skill-language-detection.md)的[认知搜索管道](cognitive-search-concept-intro.md)会受到数据限制。 记录的最大大小应为 50,000 个字符，通过 `String.Length` 进行测量。 如果需要在将数据发送到情绪分析器之前拆分数据，请使用[文本拆分技能](cognitive-search-skill-textsplit.md)。

## <a name="api-request-limits"></a>API 请求限制
* 每个请求最大 16 MB <sup>1</sup>
* 最大 8 KB URL 长度
* 每个索引上传、合并或删除的批次最多包含 1000 个文档
* $Orderby 子句中最多 32 字段
* 最大搜索词大小为 UTF-8 编码文本的 32,766 字节（32 KB 减 2 个字节）

<sup>1</sup> 在 Azure 搜索中，请求主体受 16 MB 上限的约束，这会针对不受理论限制约束的单个字段或集合的内容施加实际限制（有关字段组合和限制的详细信息，请参阅[支持的数据类型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)）。

## <a name="api-response-limits"></a>API 响应限制
* 每页搜索结果最多返回 1000 个文档
* 每个建议 API 请求最多返回 100 条建议

## <a name="api-key-limits"></a>API 密钥限制
API 密钥用于服务身份验证。 有两种类型。 管理密钥在请求标头中指定，并授予对服务的完全读写访问权限。 查询密钥是只读密钥并在 URL 上指定，并且通常分发给客户端应用程序。

* 每个服务最多 2 个管理密钥
* 每个服务最多 50 个查询密钥
