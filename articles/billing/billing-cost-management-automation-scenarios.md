---
title: Azure 计费和成本管理自动化方案 | Microsoft Docs
description: 了解如何将常用的计费和成本管理方案映射到不同的 API。
services: billing
documentationcenter: ''
author: Erikre
manager: dougeby
editor: ''
tags: billing
ms.assetid: 204b15b2-6667-4b6c-8ea4-f32c06f287fd
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 6/13/2018
ms.author: erikre
ms.openlocfilehash: cb7a13d9abcf7c677d51f03df002ea06b543014e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232476"
---
# <a name="automation-scenarios-for-billing-and-cost-management"></a>计费和成本管理自动化方案

本文列出了常见的 Azure 计费和成本管理方案。 它将这些方案映射到可以使用的 API。 在每个方案到 API 的映射下，可以找到 API 摘要及其功能。

## <a name="common-scenarios"></a>常见方案

可以在多种方案中使用计费和成本管理 API 来解答与费用和使用情况相关的问题。 下面概述了常见方案：

- **发票对帐**：Microsoft 对我的收费是否正确？  我的帐单情况如何？能否自行计算？

- **交叉记帐**：既然我已知道费用有多少，需由组织中的谁来付费？

- **成本优化**：我已知道自己的费用。 如何才能更充分地利用我的 Azure 投资？

- **成本跟踪**：我想要了解自己在一段时间内的支出情况和 Azure 使用情况。 其趋势如何？ 如何才能改进？

- **本月 Azure 支出**：我的本月至今累计支出是多少？ 我是否需要对自己的 Azure 开销和/或使用情况进行更改？ 这个月的哪个时间我使用 Azure 最多？

- **警报**：如何设置基于资源的消耗警报或基于资金的警报？

## <a name="scenario-to-api-mapping"></a>方案到 API 的映射

|         API        | 发票对帐    | 交叉记帐    | 成本优化    | 成本跟踪    | 月中支出    | 警报    |
|:---------------------------:|:-------------------------:|:----------------:|:--------------------:|:----------------:|:------------------:|:---------:|
| 预算                     |                           |                  |           X          |                  |                    |     X     |
| 市场费用                |             X             |         X        |           X          |         X        |          X         |     X     |
| 价目表                 |             X             |         X        |           X          |         X        |          X         |           |
| 预留建议 |                           |                  |           X          |                  |                    |           |
| 预留详细信息         |                           |                  |           X          |         X        |                    |           |
| 预留摘要       |                           |                  |           X          |         X        |                    |           |
| 使用情况详细信息               |             X             |         X        |           X          |         X        |          X         |     X     |
| 计费周期             |             X             |         X        |           X          |         X        |                    |           |
| 发票                    |             X             |         X        |           X          |         X        |                    |           |
| RateCard                    |             X             |                  |           X          |         X        |          X         |           |
| 未分级的使用情况               |             X             |                  |           X          |                  |          X         |           |

> [!NOTE]
> 方案到 API 的映射不包括企业消耗 API。 请尽可能使用常规使用情况 API 来解决全新开发方案的问题。

## <a name="api-summaries"></a>API 摘要

### <a name="consumption"></a>消耗
除非另有说明，否则 Web 直连客户和企业客户可使用下面的所有 API：

-   [预算 API](https://docs.microsoft.com/rest/api/consumption/budgets)（仅限企业客户）：创建资源、资源组或计费计量的成本预算或使用情况预算。 创建预算后，可以将警报配置为，在超出定义的预算阈值时发出通知。 还可以配置在达到预算金额时采取的操作。

-   [市场费用 API](https://docs.microsoft.com/rest/api/consumption/marketplaces)：获取所有 Azure 市场资源（Azure 合作伙伴产品/服务）的费用和使用情况数据。 使用此数据，可以计算所有市场资源的费用总和，也可以调查特定资源的费用/使用情况。

-   [价目表 API](https://docs.microsoft.com/rest/api/consumption/pricesheet)（仅限企业客户）：获取所有计量的自定义定价。 企业可以将此数据与使用情况详细信息和市场使用情况信息相结合，从而根据使用情况和市场数据来计算费用。 

-   [预留建议 API](https://docs.microsoft.com/rest/api/consumption/reservationrecommendations)：获取与购买虚拟机预留实例相关的建议。 建议有助于分析预期费用节省情况和购买金额。 有关详细信息，请参阅[适用于 Azure 预留自动化的 API](billing-reservation-apis.md)。

-   [预留详细信息 API](https://docs.microsoft.com/rest/api/consumption/reservationsdetails)：查看以前购买的虚拟机预留实例的相关信息，如预留消耗与实际使用量。 可以按 VM 级别详细程度查看数据。 有关详细信息，请参阅[适用于 Azure 预留自动化的 API](billing-reservation-apis.md)。

-   [预留摘要 API](https://docs.microsoft.com/rest/api/consumption/reservationssummaries)：查看组织购买的虚拟机预留实例的汇总信息，如汇总形式的预留消耗与实际使用量。 有关详细信息，请参阅[适用于 Azure 预留自动化的 API](billing-reservation-apis.md)。

-   [使用情况详细信息 API](https://docs.microsoft.com/rest/api/consumption/usagedetails)：从 Microsoft 获取所有 Azure 资源的费用和使用情况信息。 信息采用使用情况详细信息记录的形式，这些记录目前会每天按计量发出。 使用此信息，可以计算所有资源的费用总和，也可以调查特定资源的费用/使用情况。

-   [RateCard API](/previous-versions/azure/reference/mt219005(v=azure.100))：如果你是 Web 直连客户，可获取计量费率。 然后，可以结合使用返回的信息和资源使用情况信息，手动计算预期帐单。 

-   [未计费的使用情况 API](/previous-versions/azure/reference/mt219003(v=azure.100))：获取在 Azure 进行任何计量/计费前的原始使用情况信息。

### <a name="billing"></a>计费
-   [计费周期 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods)：确定要分析的计费周期，以及此周期的发票 ID。 可以将发票 ID 与发票 API 结合使用。

-   [发票 API](https://docs.microsoft.com/rest/api/billing/2018-11-01-preview/invoices)：获取计费周期的发票（PDF 格式）的下载 URL。

### <a name="enterprise-consumption"></a>企业使用情况
以下 API 仅适用于企业：

-   [余额摘要 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-balance-summary)：获取关于余额、新购买、Azure 市场服务费用、调整和超额费用的每月信息摘要。 可以获取当前计费周期或过去的任何计费周期的此信息。 企业可以使用此数据，与手动计算的汇总费用进行比较。 此 API 不提供特定于资源的信息或成本的聚合视图。

-   [使用情况详细信息 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail)：获取当前月份、特定计费周期或自定义日期段的（Microsoft 产品/服务）Azure 使用情况相关信息。 使用此数据，企业可以根据费率和消耗来手动计算帐单。 企业还可以使用部门/组织信息来跨组织确定费用出处。 可以根据这些数据按资源查看使用情况/成本。

-   [市场存储费用 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)：获取当前月份、特定计费周期或自定义日期段的（合作伙伴产品/服务）Azure 使用情况相关信息。 使用此数据，企业可以根据费率和消耗来手动计算帐单。 企业还可以使用部门/组织信息来跨组织确定费用出处。 此 API 提供资源专用使用情况/费用视图。

-   [价目表 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-pricesheet)：获取给定注册和计费周期的每个计量的适用费率。 可以将此费率信息与使用情况详细信息和市场使用情况信息相结合，从而手动计算预期帐单。

-   [计费周期 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods)：获取一系列计费周期。 此 API 还提供一个属性，指向四组与计费周期相关的企业 API 数据的 API 路由：BalanceSummary、UsageDetails、MarketplaceCharges 和 PriceSheet。

-   [预留实例建议 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation)：查看 7 天、30 天或 60 天的虚拟机使用情况，并获取单一购买和共享购买建议。 使用此 API，可以分析预期费用节省情况和建议的购买金额。 有关详细信息，请参阅[适用于 Azure 预留自动化的 API](billing-reservation-apis.md)。

## <a name="frequently-asked-questions"></a>常见问题

### <a name="whats-the-difference-between-the-enterprise-reporting-apis-and-the-consumption-apis-when-should-i-use-each"></a>企业报告 API 与消耗 API 有何区别？ 何时应使用其中的哪一个？
这些 API 具有类似的功能集，都可以解答计费和成本管理方面的各种相同问题。 不同之处在于，它们面向不同的受众： 

- 企业报告 API 适用于与 Microsoft 签署了企业协议、有权访问协商的货币承诺付款和自定义定价的客户。 这些 API 需要密钥，可通过 [Enterprise Portal](https://ea.azure.com) 获取这些密钥。 有关这些 API 的说明，请参阅[面向企业客户的报告 API 概述](billing-enterprise-api.md)。

- 消耗 API 适用于所有客户，但存在几种例外情况。 有关详细信息，请参阅 [Azure 消耗 API 概述](billing-consumption-api-overview.md)和 [Azure 消耗 API 参考](https://docs.microsoft.com/rest/api/consumption/)。 建议将提供的 API 用作最新开发方案的解决方案。 

### <a name="whats-the-difference-between-the-usage-details-api-and-the-usage-api"></a>使用情况详细信息 API 与使用情况 API 有何区别？
这些 API 提供本质不同的数据：

- [使用情况详细信息 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 提供每个计量实例的 Azure 使用情况和成本信息。 提供的数据已通过 Azure 成本计量系统进行传递，其中已应用费用和其他可能的更改：

   - 用于预付货币承诺款项的帐户的更改
   - Azure 发现的用量差异帐户的更改

- [使用情况 API](/previous-versions/azure/reference/mt219003(v=azure.100)) 提供通过 Azure 成本计量系统传递前的原始 Azure 使用情况信息。 此数据可能与通过 Azure 成本计量系统传递后显示的使用情况或费用金额没有任何关联。

### <a name="whats-the-difference-between-the-invoice-api-and-the-usage-details-api"></a>发票 API 与使用情况详细信息 API 有何区别？
这些 API 提供包含相同数据的不同视图：

- [发票 API](https://docs.microsoft.com/rest/api/billing/2018-11-01-preview/invoices) 仅适用于 Web 直连客户。 它根据每种计量类型的汇总费用提供每月帐单汇总。 

- [使用情况详细信息 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 提供每天使用情况/费用记录的具体视图。 企业客户和 Web 直连客户都可以使用它。

### <a name="whats-the-difference-between-the-price-sheet-api-and-the-ratecard-api"></a>价目表 API 与 RateCard API 有何区别？
这些 API 提供类似的数据集，但受众不同：

- [价目表 API](https://docs.microsoft.com/rest/api/consumption/pricesheet) 提供与企业客户协商好的自定义定价。

- [RateCard API](/previous-versions/azure/reference/mt219005(v=azure.100)) 提供适用于 Web 直连客户的面向公众的定价。

## <a name="next-steps"></a>后续步骤

- 若要了解如何使用 Azure API 以编程方式深入了解 Azure 使用情况，请参阅 [Azure 消耗 API 概述](billing-consumption-api-overview.md)和 [Azure 计费 API 概述](billing-usage-rate-card-overview.md)。

- 若要比较发票与 Azure 门户中的每日使用情况明细文件和成本管理报告，请参阅[了解 Microsoft Azure 帐单](billing-understand-your-bill.md)。

- 如果有疑问或需要帮助，请[创建支持请求](https://go.microsoft.com/fwlink/?linkid=2083458)。
