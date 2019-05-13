---
title: 使用 REST API 查看 Azure 订阅计费数据 | Microsoft Docs
description: 了解如何使用 Azure REST API 查看订阅计费详细信息。
services: billing
documentationcenter: na
author: lleonard-msft
manager: ''
editor: ''
ms.assetid: 82D50B98-40F2-44B1-A445-4391EA9EBBAA
ms.service: billing
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/06/2018
ms.author: erikre
ms.openlocfilehash: 0a73462b7fdbaf6386a3051a72da755f31ff8dd2
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192137"
---
# <a name="review-subscription-billing-using-rest-apis"></a>使用 REST API 查看订阅计费

Azure 报告 API 可帮助查看和管理 Azure 费用。

筛选器可帮助自定义结果以满足需求。

本文介绍如何使用 REST API 返回给定日期范围内的订阅计费详细信息。

``` http
GET https://management.azure.com/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}/providers/Microsoft.Consumption/usageDetails?$filter=properties/usageEnd ge '${startDate}' AND properties/usageEnd le '${endDate}'
Content-Type: application/json
Authorization: Bearer
```

## <a name="build-the-request"></a>生成请求

`{subscriptionID}` 参数是必需的，用于标识目标订阅。

`{billingPeriod}` 参数是必需的，用于指定当前[计费周期](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods)。

`${startDate}` 和 `${endDate}` 参数对于此示例是必需的，但对于终结点是可选的。 它们以 YYYY-MM-DD 形式将日期范围指定为字符串（例如：`'20180501'` 和 `'20180615'`）。

以下标头是必需的：

|请求标头|描述|
|--------------------|-----------------|
|Content-Type：|必需。 设置为 `application/json`。|
|Authorization：|必需。 设置为有效的 `Bearer` [访问令牌](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients)。 |

## <a name="response"></a>响应

系统会返回状态代码 200（正常）以指示一个成功响应，该响应中包含你的帐户的详细费用列表。

``` json
{
  "value": [
    {
      "id": "/subscriptions/{$subscriptionID}/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/{$detailsID}",
      "name": "{$detailsID}",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        "billingPeriodId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}",
        "invoiceId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/invoices/${invoiceID}",
        "usageStart": "${startDate}}",
        "usageEnd": "${endDate}",
        "currency": "USD",
        "usageQuantity": "${usageQuantity}",
        "billableQuantity": "${billableQuantity}",
        "pretaxCost": "${cost}",
        "meterId": "${meterID}",
        "meterDetails": "${meterDetails}"
      }
    }
  ],
  "nextLink": "${nextLinkURL}"
}
```

**值**中的每一项均表示有关服务使用的详细信息：

|响应属性|描述|
|----------------|----------|
|**subscriptionGuid** | 订阅的的全局唯一 ID。 |
|**startDate** | 使用开始日期。 |
|**endDate** | 使用结束日期。 |
|**useageQuantity** | 使用数量。 |
|**billableQuantity** | 实际计费数量。 |
|**pretaxCost** | 在适用税前开发票的费用。 |
|**meterDetails** | 有关使用的详细信息。 |
|**nextLink**| 设置时，指定一个 URL 作为详细信息的下一“页”。 当页面为最后一个页面时为空。 |

此示例已经过缩略；请参阅[列出使用情况详细信息](https://docs.microsoft.com/rest/api/consumption/usagedetails/list#usagedetailslistforbillingperiod)以了解每个响应字段的完整说明。

其他状态代码指示错误条件。 在这些情况下，响应对象解释请求失败的原因。

``` json
{
  "error": [
    {
      "code": "Error type.",
      "message": "Error response describing why the operation failed."
    }
  ]
}
```

## <a name="next-steps"></a>后续步骤
- 查看 [Enterprise Reporting 概述](https://docs.microsoft.com/azure/billing/billing-enterprise-api)
- 详细研究[企业计费 REST API](https://docs.microsoft.com/rest/api/billing/)
- [Azure REST API 入门](https://docs.microsoft.com/rest/api/azure/)
