---
title: 对于企业协议中了解 Azure 预订使用情况
description: 了解如何读取使用情况，以了解如何应用适用于企业合约的 Azure 预留。
author: bandersmsft
manager: yashar
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/07/2019
ms.author: banders
ms.openlocfilehash: 8d85dd1c21f952261e838c01843e15dafcc0e931
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415735"
---
# <a name="get-enterprise-agreement-reservation-costs-and-usage"></a>获取企业协议保留成本和使用情况

预留成本和使用情况数据则是适用于企业协议客户在 Azure 门户和 REST Api。 本文可帮助你：

- 获取预订购买数据
- 获取保留未得到充分利用数据
- 摊销预订成本
- Chargeback 保留使用率
- 计算保留节省

在使用情况数据合并的 marketplace 费用。 查看第一方使用情况、 marketplace 使用情况和单个数据源中的购买费用。

## <a name="reservation-charges-in-azure-usage-data"></a>Azure 使用情况数据中的预订费用

数据划分为两个单独的数据集：_实际成本_并_摊销成本_。 这两个数据集有何不同：

**实际成本**-提供要与每月的帐单保持一致的数据。 此数据已预订购买成本。 它具有零个 EffectivePrice 接收预订折扣的使用情况。

**摊销成本**-资源获取预订折扣的 EffectiveCost 是保留的实例的每天按比例分摊的成本。 数据集还具有未使用的预订成本。 保留费和未使用的预订的总和提供保留的每日摊销的成本。

两个数据集的比较：

| 数据 | 实际成本数据集 | 摊销的成本数据集 |
| --- | --- | --- |
| 预订购买 | 在此视图中可用。<br>  若要获得此数据筛选器在 ChargeType =&quot;采购&quot;。 <br> 请参阅 ReservationID 或 ReservationName 知道费用是针对哪些保留。  | 此视图不适用于。 <br> 摊销数据中未提供的采购成本。 |
| EffectivePrice | 值为零的使用情况，可获取预订折扣。 | 值为已预订折扣的使用情况的保留的每小时按比例分配成本。 |
| 未使用的预订 （提供的预订未使用在一天中的小时数和浪费的货币值） | 在此视图中不适用。 | 在此视图中可用。<br> 若要获取此数据，筛选 ChargeType = &quot;UnusedReservation&quot;。<br>  请参阅 ReservationID 或 ReservationName 知道哪些保留了未被充分利用。 这是多少预订浪费在上一天。  |
| 单价 （价格表中的资源的价格） | 可用 | 可用 |

Azure 使用情况数据中可用的其他信息已更改：

- 产品和计量信息-Azure 不会替换为最初已使用的计量的 ReservationId 和 ReservationName，就像以前一样。
- ReservationId 和 ReservationName-它们是其自己数据中的字段。 以前，它使用为仅在 AdditionalInfo 下可用。
- ProductOrderId-预订订单 ID，作为其自己的字段添加。
- ProductOrderName-购买的预订的产品名称。
- 术语的 12 个月或 36 个月。
- RINormalizationRatio-AdditionalInfo 下可用。 这是比预订应用于使用情况记录的位置。 如果实例大小灵活性上启用了预订，则它可以将应用于其他大小。 值显示的使用情况记录已应用于保留的比率。

## <a name="get-azure-consumption-and-reservation-usage-data-using-api"></a>获取使用 API 的 Azure 使用情况和预订使用情况数据

可以使用该 API 获取数据或从 Azure 门户下载它。

在调用[使用情况详细信息 API](/rest/api/consumption/usagedetails/list) API 版本&quot;2019年-04-01-预览&quot;获取新数据。 有关术语的详细信息，请参阅[使用条款](billing-understand-your-usage.md)。 调用方应为企业协议使用的企业管理员[EA 门户](https://ea.azure.com)。 只读的企业管理员还可以获取数据。

数据中不可[适用于企业客户的使用情况详细信息的报告 Api](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail)。

下面是对 API 的示例调用：

```
https://consumption.azure.com/providers/Microsoft.Billing/billingAccounts/{enrollmentId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodId}/providers/Microsoft.Consumption/usagedetails?metric={metric}&amp;api-version=2019-04-01-preview&amp;$filter={filter}
```

有关详细信息 {enrollmentId} 和 {billingPeriodId}，请参阅[用法的详细信息 – 列表](https://docs.microsoft.com/rest/api/consumption/usagedetails/list)API 文章。

下表中有关度量值和筛选器的信息可帮助解决有关预订常见问题。

| **API 数据类型** | API 调用操作 |
| --- | --- |
| **所有费用 （使用情况和购买）** | {指标} 替换为 ActualCost |
| **获取预订折扣的使用情况** | {指标} 替换为 ActualCost<br>将 {filter} 替换为： properties/reservationId%20ne%20 |
| **未获取预订折扣的使用情况** | {指标} 替换为 ActualCost<br>Replace {filter} with: properties/reservationId%20eq%20 |
| **摊销的费用 （使用情况和购买）** | {指标} 替换为 AmortizedCost |
| **未使用的预订报表** | {指标} 替换为 AmortizedCost<br>将 {filter} 替换为： properties/ChargeType%20eq%20'UnusedReservation |
| **预订购买** | 将 {指标} 替换为与 ActualCostReplace {filter}: properties/ChargeType%20eq%20'Purchase  |
| **退款** | {指标} 替换为 ActualCost<br>将 {filter} 替换为： properties/ChargeType%20eq%20'Refund |

## <a name="download-the-usage-csv-file-with-new-data"></a>下载包含新数据的使用情况 CSV 文件

如果你是 EA 管理员，可以下载包含新的使用情况数据从 Azure 门户的 CSV 文件。 此数据不会从可用[EA 门户](https://ea.azure.com)。

在 Azure 门户中，导航到[成本管理 + 计费](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/BillingAccounts)。

1. 选择计费帐户。
2. 单击**使用情况 + 费用**。
3. 单击“下载”。  
![显示在何处下载 Azure 门户中的 CSV 使用情况数据文件的示例](./media/billing-understand-reserved-instance-usage-ea/portal-download-csv.png)
4. 在中**下载使用情况 + 费用**下**使用情况详细信息版本 2** ，选择 **（使用情况和购买） 的所有费用**，然后单击下载。 重复这些步骤**分摊费用 （使用情况和购买）**。

所下载的 CSV 文件包含实际成本和摊销的成本。

## <a name="common-cost-and-usage-tasks"></a>成本和使用情况的常见任务

以下各节是大多数人使用，以查看其保留成本和用量数据的常见任务。

### <a name="get-reservation-purchase-costs"></a>获取预订的采购成本

实际成本数据中提供了保留项的采购成本。 筛选_ChargeType = 采购_。 请参阅 ProductOrderID 来确定为用户购买的是哪个预订订单。

### <a name="get-underutilized-reservation-quantity-and-costs"></a>获取未充分利用的预订数量和成本

获取摊销成本数据和筛选_ChargeType_ _= UnusedReservation_。 可获得每日的未使用的预订数量和成本。 可以保留或保留顺序使用的数据进行筛选_ReservationId_并_ProductOrderId_字段，分别。 如果保留利用率达到 100%，记录将具有数量为 0。

### <a name="amortize-reservation-costs"></a>摊销预订成本

获取摊销成本数据，并保留顺序使用的筛选_ProductOrderID_预订获取每日摊销的成本。

### <a name="chargeback-for-a-reservation"></a>预订的退款

您可以用于其他组织的退款预订使用按订阅、 资源组或标记。 摊销的成本数据提供了在以下数据类型的预订的利用率的货币值：

- 资源 （例如 VM)
- 资源组
- 标记
- 订阅

### <a name="get-the-blended-rate-for-chargeback"></a>混合的速率获得退款

若要确定混合的速率，获取摊销的成本数据和聚合的总成本。 对于 Vm，可以使用的计量名称或服务类型信息从 AdditionalInfo JSON 数据。 将划分的总成本由用来获取混合的率的数量。

### <a name="audit-optimum-reservation-use-for-instance-size-flexibility"></a>审核最佳预订实例使用大小大的灵活性

具有多个数量_RINormalizationRatio_，从 AdditionalInfo。 结果表明对使用情况记录应用的预订使用小时数。

### <a name="determine-reservation-savings"></a>确定保留节省

获取 Amortized 成本数据和筛选数据的保留实例。 然后：

1. 获取估计的即用即付成本。 乘_UnitPrice_值替换_Quantity_值，以获取估计的即用即付成本，如果预订折扣不适用于使用情况。
2. 获取保留成本。 总和_成本_值，以获取你为保留的实例付费的货币值。 它包括使用和未使用预订的成本。
3. 减去保留从估计的即用即付成本，以获取估计的节省成本。

## <a name="reservation-purchases-and-amortization-in-azure-cost-analysis"></a>预订购买和摊销在 Azure 成本分析

预订的实例成本现已推出[Azure 成本分析预览模式](https://preview.portal.azure.com/?feature.canmodifystamps=true&amp;microsoft_azure_costmanagement=stage2&amp;Microsoft_Azure_CostManagement_arm_canary=true&amp;Microsoft_Azure_CostManagement_apiversion=2019-04-01-preview&amp;Microsoft_Azure_CostManagement_amortizedCost=true#blade/Microsoft_Azure_CostManagement/Menu/costanalysis)。 默认情况下，成本数据视图为的实际成本。 您可以切换到摊销成本。 下面是一个示例。

![显示在成本分析中选择摊销的成本的位置的示例](./media/billing-understand-reserved-instance-usage-ea/portal-cost-analysis-amortized-view.png)

应用筛选器以查看你的费用由预订或费用类型。 保留名称，可以查看按保留项细分的成本进行分组。

## <a name="need-help-contact-us"></a>需要帮助？ 请联系我们。

如果有疑问或需要帮助，请[创建支持请求](https://go.microsoft.com/fwlink/?linkid=2083458)。

## <a name="next-steps"></a>后续步骤

若要了解有关 Azure 预订的详细信息，请参阅以下文章：

- [什么是 Azure 预订？](billing-save-compute-costs-reservations.md)
- [通过 Azure 虚拟机预留实例为虚拟机预付费](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [通过 Azure SQL 数据库保留容量预付 SQL 数据库计算资源费用](../sql-database/sql-database-reserved-capacity.md)
- [管理 Azure 预留项](billing-manage-reserved-vm-instance.md)
- [了解预留折扣的应用方式](billing-understand-vm-reservation-charges.md)
- [了解即用即付订阅的预留使用情况](billing-understand-reserved-instance-usage.md)
- [预订未包含的 Windows 软件成本](billing-reserved-instance-windows-software-costs.md)
