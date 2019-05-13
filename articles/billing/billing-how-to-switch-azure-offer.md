---
title: 更改 Azure 订阅套餐 | Microsoft Docs
description: 了解如何通过 Azure 帐户中心更改 Azure 订阅并切换到其他套餐
services: ''
documentationcenter: ''
author: genlin
manager: adpick
editor: ''
tags: billing,top-support-issue
ms.assetid: aae227b3-6d64-4550-a5b6-d359f53f0a59
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: banders
ms.openlocfilehash: 11d3deb029ef0d8478844b64c54db7982ed1091e
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64570977"
---
# <a name="change-your-azure-pay-as-you-go-subscription-to-a-different-offer"></a>将 Azure 即用即付订阅更改为其他套餐

[即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)客户可以在[帐户中心](https://account.windowsazure.com/Subscriptions)将 Azure 订阅切换到其他套餐。 例如，可以通过此功能充分利用 [Visual Studio 订户的每月信用额度](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)。 

只想从免费试用版升级？ 请参阅[升级到即用即付版](billing-upgrade-azure-subscription.md)。

## <a name="whats-supported"></a>支持的操作：

| 源 | 目标 |
| --- | --- |
| 即用即付 |[即用即付开发/测试](https://azure.microsoft.com/offers/ms-azr-0023p/) |
| 即用即付 |[Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/) |
| 即用即付 |[Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/) |
| 即用即付 |[MSDN 平台](https://azure.microsoft.com/offers/ms-azr-0062p/) |
| 即用即付 |[Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) |
| 即用即付 |[Visual Studio Enterprise (BizSpark)](https://azure.microsoft.com/offers/ms-azr-0064p/) |

> [!NOTE]
> 有关其他产品/服务更改，请[联系支持人员](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
>
>

## <a name="switch-subscription-offer"></a>切换订阅套餐

> [!VIDEO https://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/Switch-to-a-different-Azure-offer/player]
>
>

1. 在 [Azure 帐户中心](https://account.windowsazure.com/Subscriptions)登录。
1. 选择即用即付订阅。
1. 单击“切换到其他套餐”。 仅当用户使用“即用即付”且已完成第一个计费周期的情况下，此按钮才可用。

   ![请注意页面右侧的“切换套餐”按钮](./media/billing-how-to-switch-azure-offer/switchbutton.png)
1. 从订阅可以切换到的套餐的列表中**选择所需套餐**。 此列表因帐户的关联成员资格而异。 如果没有任何可用的产品/服务，请检查 [可以切换到的可用产品/服务的列表](#whats-supported)，并确保具有合适的成员身份。 

   ![选择要切换到的产品/服务](./media/billing-how-to-switch-azure-offer/selectoffer.png)
1. 根据要切换到的产品/服务，可能会看到一条有关切换所造成的影响的备注。 继续操作之前，请仔细查看此列表并遵循相关说明。

   ![查看说明](./media/billing-how-to-switch-azure-offer/thingstonote.png)
1. 可以重命名订阅。 默认情况下，我们会将它设置为新的产品/服务名称。 单击“切换套餐”以完成该过程。

   ![单击绿色按钮](./media/billing-how-to-switch-azure-offer/confirmpage.png)
1. 成功！ 订阅现在已经切换到新的套餐。

## <a name="frequently-asked-questions"></a>常见问题

### <a name="what-is-an-azure-offer"></a>什么是 Azure 产品/服务？

Azure 套餐是用户拥有的 Azure 订阅*类型*。 例如，[即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)、[Azure 开放许可](https://azure.microsoft.com/offers/ms-azr-0111p/)和 [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) 都是 Azure 套餐。 每个产品/服务附带不同的[条款](https://azure.microsoft.com/support/legal/offer-details/)，有些还附带特殊的权益。 可在帐户中心的订阅页中找到订阅的套餐。 单击产品/服务名称可获取更多详细信息。

   ![在帐户中心单击“产品/服务”链接可获取更多详细信息](./media/billing-how-to-switch-azure-offer/offerlink.png)

### <a name="why-dont-i-see-the-button"></a>为什么我看不到按钮？

如果出现以下情况，可能看不到“切换到其他套餐”按钮：

* 没有使用[即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)。 当前只有即用即付订阅可以转换到其他套餐。
  * 如果使用的是[免费试用版](https://azure.microsoft.com/free/)，请了解如何[升级到即用即付](billing-upgrade-azure-subscription.md)。
  * 若要从其他订阅切换套餐，[请与支持人员联系](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
* 仍处于第一个计费周期；必须等到第一个计费周期结束，才能切换套餐。

### <a name="why-do-i-see-there-are-no-offers-available-in-your-region-or-country-at-this-time"></a>为什么会出现“你所在的国家或地区目前没有可用的产品/服务”？

* 你可能没有进行任何产品/服务切换的资格。 请查看[可切换的可用产品/服务列表](#whats-supported)，确保已通过 Visual Studio 或 Bizspark 激活了正确的权益。
* 某些产品/服务可能不可用在所有国家/地区。

### <a name="what-does-switching-azure-offers-do-to-my-service-and-billing"></a>切换 Azure 产品/服务会对我的服务和帐单进行哪些操作？

下面详述了在帐户中心切换 Azure 产品时所发生的情况。

#### <a name="no-service-downtime"></a>服务不会中断

与订阅关联的任何用户都不会出现服务无法使用的情况。 但是，切换到的产品/服务可能会有一些限制。 例如，某些套餐禁止用于生产，因此需要将生产资源移到另一个订阅。

#### <a name="quota-increases-are-reset"></a>配额增加被重置

切换套餐时将重置任何[超过默认限制的限额或配额增加](../azure-supportability/resource-manager-core-quotas-request.md)。 即使更多资源超出默认限制也不会中断服务。 例如，如果订阅使用 200 个内核，那么切换套餐会将内核配额重置回默认的 20 个内核。 使用 200 个内核的 VM 不受影响，将继续运行。 但如果不提出其他增加配额的请求，则将无法预配更多内核。

#### <a name="billing"></a>计费

在切换当天，为所有未付费用生成发票。 之后，按新套餐的定价条款对订阅进行计费。 订阅计费周年会变为更改套餐的日期。 更改产品/服务之前的使用情况与计费数据不会被保留，因此建议在切换之前先下载一个副本。

### <a name="can-i-migrate-from-pay-as-you-go-to-cloud-solution-providerhttpspartnermicrosoftcomsolutionscloud-reseller-overview-csp-or-enterprise-agreementhttpsazuremicrosoftcompricingenterprise-agreement-ea"></a>能否从即用即付转为[云解决方案提供商](https://partner.microsoft.com/Solutions/cloud-reseller-overview) (CSP) 或[企业协议](https://azure.microsoft.com/pricing/enterprise-agreement/) (EA)？

* 若要迁移到 CSP，请参阅[将 Azure 即用即付订阅迁移到 CSP](https://docs.microsoft.com/azure/cloud-solution-provider/migration/migration-from-payg-to-csp)。
* 要迁移到 EA，请让注册管理员将帐户加入 EA。 遵照邀请电子邮件中的说明，根据 EA 注册过程来移动订阅。 若要了解详细信息，请参阅 [Associate an Existing Account in the EA portal](https://ea.azure.com/helpdocs/associateExistingAccount)（在 EA 门户中关联现有帐户）。

### <a name="can-i-migrate-data-and-services-to-a-new-subscription"></a>是否可将数据和服务迁移到新订阅？

* 可以迁移直接将资源迁移到新订阅，具体请参阅[将资源移到新资源组或订阅中](../azure-resource-manager/resource-group-move-resources.md)。
* 要将 Azure 订阅的所有权及其包含的所有内容转让给其他某人，请参阅[转让 Azure 订阅的所有权](billing-subscription-transfer.md)

## <a name="need-help-contact-us"></a>需要帮助？ 请联系我们。

如果有疑问或需要帮助，请[创建支持请求](https://go.microsoft.com/fwlink/?linkid=2083458)。
