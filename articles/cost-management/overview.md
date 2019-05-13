---
title: Azure 中的 Cloudyn 概述 | Microsoft Docs
description: Cloudyn 是一个多云成本管理解决方案，可帮助你使用 Azure 和其他云资源。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 03/13/2019
ms.topic: overview
ms.service: cost-management
manager: benshy
ms.custom: seodec18
ms.openlocfilehash: 0bdf47991435a9a28a4db12f3aa89478b18bb9f1
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61020242"
---
# <a name="what-is-the-cloudyn-service"></a>什么是 Cloudyn 服务？

Cloudyn（一家 Microsoft 子公司）可以跟踪 Azure 资源和其他云提供商（包括 AWS 和 Google）的云使用量和开支。 便于理解的仪表板报告也有助于成本分配和 Showback/Chargeback。 Cloudyn 通过确定可供稍后管理和调整的未充分利用的资源，帮助优化云支出。

若要观看介绍性视频，请参阅 [Azure Cloudyn 简介](https://azure.microsoft.com/resources/videos/azure-cost-management-overview-and-demo)。

Azure 成本管理提供了与 Cloudyn 类似的功能。 Azure 成本管理是一个本机 Azure 成本管理解决方案。 借助它，可以分析成本、创建和管理预算、导出数据，并能查看和采纳优化建议，从而节省资金。 有关详细信息，请参阅 [Azure 成本管理](overview-cost-mgt.md)。

## <a name="cloudyn-features-moving-to-azure-cost-management"></a>Cloudyn 功能移动到 Azure 成本管理

Microsoft 收购了 Cloudyn，并将其成本管理功能从 Cloudyn 门户本机迁移到 Azure。 若要使用新功能，请登录到 Azure 门户并导航到 Azure 服务列表中的[成本管理和账单](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview)。 与 Cloudyn 相比，本机体验提高了性能并减少了大约八小时的数据延迟。

将企业协议、即用即付和 MSDN 产品/服务类别的主要功能迁移到 Azure 成本管理的过程已完成。 CSP 订阅正在迁移到 Azure 成本管理。

如果有一个产品/服务类别尚未迁移，应继续使用 Cloudyn 门户。 其他所有人都可以使用 Azure 成本管理。

| Microsoft Azure 产品/服务和功能 | 推荐的成本管理服务 |
| --- | --- |
| Azure 企业协议 | [Azure 成本管理](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Azure Web Direct (PAYG/MSDN) | [Azure 成本管理](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Azure Government  | [Azure 成本管理](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Azure CSP | [Cloudyn](https://azure.cloudyn.com) |
| 对 AWS 的跨云成本分析支持（预览版） | [Azure 成本管理](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| AWS 建议 | [Cloudyn](https://azure.cloudyn.com) |

Cloudyn 中提供了以下某些功能，但所有这些功能现在都可在 Azure 成本管理中使用。

- API
- Azure 计算建议
- Azure 预留建议
- 预算
- 成本分析
- 将数据导出到 Azure 存储帐户
- 更低的延迟
- Power BI 内容包和连接器
- 资源标记支持

## <a name="monitor-usage-and-spending"></a>监视使用量和支出

监视使用量和支出对云基础结构而言至关重要，因为组织按使用时间支付资源费用。 当使用量超出协议阈值时，会快速产生意外的成本超额。 几个重要的因素使临时监视变得困难。 首先，根据平均使用量预测成本需假定在给定计费时段内，使用情况保持一致。 其次，当成本接近或超过预算时，主动获得通知以调整支出这一点很重要。 另外，云服务提供商可能不会提供成本预测与阈值或各个时期的对比报告。

报告有助于监视开支以分析和跟踪云使用量、成本和趋势。 通过历时报告，可以检测不同于正常趋势的异常现象。 在优化报告中可以查看云部署的低效情况。 还可以在成本分析报告中注意到低效情况。

## <a name="manage-costs"></a>管理成本

当分析一段时间的使用量和成本以确定趋势时，历史数据可帮助管理成本。 然后，可使用趋势预测将来的支出。 Cloudyn 还包括有用的预测成本报告。

根据标记策略，成本分配通过分析成本来管理成本。 可以使用自定义帐户、资源和实体上的标记优化成本分配。 类别管理器可组织标记，以帮助提供其他治理。 另外，可将成本分配用于 Showback/Chargeback 以显示资源利用率和相关成本，从而影响使用行为或向租户客户收费。

访问控制通过确保用户和团队仅访问他们所需的成本管理数据帮助管理成本。 通过将实体结构、用户管理和计划报告与收件人列表结合使用来分配访问权限。

警报功能可在发生异常支出或超支时自动通知用户，从而帮助管理成本。 出现支出异常和超支风险时，警报功能也会自动通知其他利益相关者。 各种报告根据预算和成本阈值支持警报。 但是，目前 CSP 合作伙伴帐户或订阅不支持警报。

## <a name="improve-efficiency"></a>提升效率

借助 Cloudyn，可以确定最佳 VM 使用情况并识别空闲 VM，或者删除空闲 VM 和未附加的磁盘。 使用大小优化和低效报告中的信息，可以创建一个计划以缩小或删除空闲 VM。 但是，目前 CSP 合作伙伴帐户或订阅不支持优化报告。

如果已预配 AWS 预留实例，可以使用优化报告提升预留实例利用率，使用该报告还可以查看购买建议、修改未使用的预订并计划预配。


## <a name="next-steps"></a>后续步骤

熟悉 Cloudyn 后，下一步就是注册云环境并开始探索你的数据。

- [注册一个单独的 Azure 订阅](quick-register-azure-sub.md)
