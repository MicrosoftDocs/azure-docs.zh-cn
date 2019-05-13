---
title: Web Analytics
description: .
services: Azure, Marketplace, Cloud Partner Portal,
author: pbutlerm
manager: Ricardo.Villalobos
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 4b75b2cbd629204e7edbf1196eec9b03cb7a6736
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64943159"
---
<a name="web-analytics"></a>Web Analytics
=============

本文提供有关如何学习和使用 Web Analytics，以便最合理地发展业务的说明。 目前，此“见解”选项卡仅适用于任何 AppSource 套餐。

生成并发布套餐后，旅程的下一个环节是跟踪并衡量该套餐的成功度。 我们通过 **Web Analytics** 添加了相应的功能，让你确切地了解自己的套餐在市场中的表现。 若要启动旅程，请导航到云合作伙伴门户左侧的“见解”页，以查看新的“Analytics”选项卡。

![WebAnalytics 页](./media/si-getting-started/WebAnalytics1.png)

此时会显示发布者 ID 的丰富仪表板，它是使用 Microsoft Power BI 生成的，可在其中查看每个套餐的数据，这些数据每日刷新。

<a name="microsoft-campaigns"></a>**Microsoft 营销活动**
-----------------------

为了扩展套餐以及跟踪套餐的扩展情况，我们在云合作伙伴门户上启用了“Microsoft 营销活动”功能。 营销活动是最新支持的市场功能，可用于跟踪在应用详细信息页中发送客户的不同渠道。

### <a name="how-to-make-a-campaign"></a>**如何创建营销活动**

描述营销活动的最简单方法是将自定义单词/字词添加到在市场中加载应用详细信息页的 URL 中。 Google、必应、LinkedIn 和其他许多站点都鼓励你生成广告，并从其站点链接到所需的站点。

一般情况下，这些工作有助于让新客户接触到你的产品，并且是衡量每个渠道的工作绩效的重要依据。 营销活动在此可以发挥作用。

可通过两种方式生成自己的营销活动。

1. 将查询参数 **mktcmpid** 添加到 URL。该查询参数描述营销活动的具体内容，以及这些客户来自哪个页面/事件。

例如，可以使用：<https://appsource.microsoft.com/product/dynamics-365/contoso.offername?mktcmpid=NewCampaign>

1. （高级）：在 URL 中使用我们支持的通用市场活动 Id 之一。 我们希望能够顺应你需要使用的其他推荐标记，因此我们支持自动识别这些附加标记的约定：
    
    1. **utm\_campaign**
    2. **utm\_source**
    3. **ref**
    4. **src**

例如，可以使用：<https://appsource.microsoft.com/product/dynamics-365/contoso.offername?utm_campaign=NewCampaign>

可以选择使用其中多个营销活动 ID 的组合，以进一步识别驱动营销活动流量的多个源，例如，客户来自何处（电子邮件、博客、社交媒体源等）。

例如：

1. 新闻稿推荐人：<https://appsource.microsoft.com/product/dynamics-365/contoso.offername?mktcmpid=NewCampaign&src=newsletter>
2. LinkedIn 推荐人：<https://appsource.microsoft.com/product/dynamics-365/contoso.offername?mktcmpid=NewCampaign&src=LinkedIn>

### <a name="ensuring-campaigns-pass-through-all-your-pages"></a>**确保在所有页面中传递营销活动**

在某种情况下，有可能营销活动的驱动流量会定向到一个中间页面，然后，通过该页面将客户定向到市场。 必须将初始营销活动 ID 传递到要发送到市场的最终 URL 中。

下面是一个示例：

1. 营销员工从 Google 购买广告，以将流量驱动到公司的登陆页面 <https://contoso.com>。 此登陆页面包含一个指向 <https://appsource.com> 的\"试用我的产品\"链接。
2. 用户单击广告，然后登陆到其公司的登陆页面。
    1.  推荐 URL = google.com
    2.  登陆页面 URL = <https://contoso.com/?utm_campaign=MyCampaignAdName&utm_source=MySourceAdName>
3. 用户单击\"试用我的产品\"链接，然后进入 AppSource。
    1. 推荐 URL = <https://contoso.com/?utm_campaign=MyCampaignAdName&utm_source=MySourceAdName>
    2. 登陆页面 URL（**请确保在此 URL 中添加 utm\_campaign 和 utm\_source**）= [https://appsource.microsoft.com/en-us/product/dynamics-365/contoso.offername?**utm\_campaign=MyCampaignAdName&utm\_source=MySourceAdName**](https://appsource.microsoft.com/en-us/product/dynamics-365/contoso.offername?utm_campaign=MyCampaignAdName&utm_source=MySourceAdName)

<a name="how-to-evaluate-the-success-of-a-campaign"></a>如何评估营销活动的成功度
-----------------------------------------

### <a name="page-visits-by-campaign"></a>**按营销活动列出的页面访问情况**

![WebAnalytics2](./media/si-getting-started/WebAnalytics2.png)

这是每日页面访问情况的明细，按访问时的来源营销活动列出。

### <a name="conversion-rate-by-campaign"></a>**按营销活动列出的转换率**

![WebAnalytics3](./media/si-getting-started/WebAnalytics3.png)

类似于我们显示整个套餐的转换率的方式，在此图表中，可以查看不同营销活动的表现明细。 此图表应有助于识别较高转换率的营销活动来源于何处。

### <a name="distribution-by-campaign"></a>**按营销活动显示的分布情况**

![WebAnalytics4](./media/si-getting-started/WebAnalytics4.png)

类似于我们查看客户所在领域的方式，在此图表中，可以查看用户进入市场时所在的每个营销活动的数据分布。 \_NoCampaign 表示客户导航到市场时，其 URL 中不包含营销活动 ID。

<a name="next-steps"></a>**后续步骤**
--------------

在能够跟踪套餐的成功度之后，我们建议你创建自己的营销活动。

如果有疑问/想要请求功能，请通过右上角的“反馈”告诉我们。

![云合作伙伴门户中的“反馈”](./media/si-getting-started/WebAnalytics5.png)
