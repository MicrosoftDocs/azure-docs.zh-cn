---
title: 在 Azure 中防止意外成本，管理计费 | Microsoft Docs
description: 了解如何在 Azure 帐单上避免意外费用。 将成本跟踪和管理功能用于 Microsoft Azure 订阅。
services: ''
documentationcenter: ''
author: bandersmsft
manager: alherz
editor: ''
tags: billing
ms.assetid: 482191ac-147e-4eb6-9655-c40c13846672
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/24/2018
ms.author: banders
ms.openlocfilehash: 146c74fe751e75fb85563378be6f812802928fe2
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64918940"
---
# <a name="prevent-unexpected-charges-with-azure-billing-and-cost-management"></a>通过 Azure 计费和成本管理来防止意外费用

当注册 azure 时，有的支出更好地了解您可以做几件事情。 [定价计算器](https://azure.microsoft.com/pricing/calculator/)可在创建 Azure 资源之前提供成本估计。 [Azure 门户](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)可提供订阅的当前成本细分和预测。 如果要对不同项目或团队的成本进行分组和了解，请查看[资源标记](../azure-resource-manager/resource-group-using-tags.md)。 如果组织有希望使用的报告系统，请查看[计费 API](billing-usage-rate-card-overview.md)。

- 如果订阅是企业协议 (EA)，则可在 Azure 门户中通过公共预览查看费用。 如果订阅通过云解决方案提供商 (CSP) 或 Azure 赞助，则以下某些功能可能不适用。 有关详细信息，请参阅[适用于 EA、CSP 和赞助的其他资源](#other-offers)。

- 如果订阅是免费试用版 [Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)、Azure 开放许可 (AIO) 或 BizSpark，则使用所有信用额度后，订阅将被自动禁用。 了解[支出限制](#spending-limit)，避免意外禁用订阅。

- 如果已注册了 [Azure 免费帐户](https://azure.microsoft.com/free/)，[则可免费使用某些最常用的 Azure 服务 12 个月](billing-create-free-services-included-free-account.md)。 请参考下面列出的建议，同时请参阅[避免为免费帐户付费](billing-avoid-charges-free-account.md)。

## <a name="get-estimated-costs-before-adding-azure-services"></a>在添加 Azure 服务之前获取估计成本

### <a name="estimate-cost-online-using-the-pricing-calculator"></a>使用定价计算器在线估计成本

查看[定价计算器](https://azure.microsoft.com/pricing/calculator/)，获取感兴趣的服务的每月估计成本。 可以添加任何第一方 Azure 资源，以获取估计成本。

![定价计算器菜单的屏幕截图](./media/billing-getting-started/pricing-calc.png)

例如，如果 A1 Windows 虚拟机 (VM) 保持一直运行，则其计算小时数预计花费 66.96 美元/月：

![定价计算器的屏幕截图，图中 A1 Windows VM 预计每月花费 66.96 美元](./media/billing-getting-started/pricing-calcVM.png)

有关定价的详细信息，请参阅此[常见问题解答](https://azure.microsoft.com/pricing/faq/)。 或者，如果想要与 Azure 销售人员进行交流，请联系 1-800-867-1389。

### <a name="review-the-estimated-cost-in-the-azure-portal"></a>在 Azure 门户中查看估计成本

通常情况下，在 Azure 门户中添加服务时，会出现一个视图显示大概的每月预计成本。 例如，选择 Windows VM 的大小时，可看到计算小时数的每月估计成本：

![示例：一台 A1 Windows VM 预计每月花费 66.96 美元](./media/billing-getting-started/vm-size-cost.PNG)

### <a name="spending-limit"></a>检查支出限制是否为开启状态

如果有使用信用额度的订阅，则将默认开启支出限制。 这样一来，当信用额度花完时，不再对信用卡计费。 请参阅 [Azure 产品/服务以及支出限制可用性的完整列表](https://azure.microsoft.com/support/legal/offer-details/)。

但是，在达到支出限制时，服务会被禁用。 这意味着 VM 会被解除分配。 若要避免服务中断，必须关闭支出限制。 任何超额都将对保存的信用卡收费。 

若要查看支出限制是否为开启状态，请转到[帐户中心的订阅视图](https://account.windowsazure.com/Subscriptions)。 如果支出限制为开启状态，会显示以下标题：

![帐户中心中，显示支出限制为开启状态的警告信息的屏幕截图](./media/billing-getting-started/spending-limit-banner.PNG)

单击该标题，并按照提示移除支出限制。 如果注册时未输入信用卡信息，则必须输入信息才能移除支出限制。 有关详细信息，请参阅 [Azure 支出限制 - 其工作原理以及如何将其启用或移除](https://azure.microsoft.com/pricing/spending-limits/)。

可以使用 [Cloudyn](https://www.cloudyn.com/) 服务创建警报，自动通知利益干系人支出异常和超支风险。 可以使用支持基于预算和成本阈值的警报的报表创建警报。 有关使用 Cloudyn 的更多信息，请参阅[教程：查看使用情况和成本](../cost-management/tutorial-review-usage.md)。

当你在 Azure VM 上的支出接近总预算时，此示例使用“一段时间的实际成本”报表来发送通知。 在此场景中，你的总预算为 20,000 美元，你希望在成本接近预算的一半（9,000 美元）时收到通知，在成本为 10,000 美元时再次收到警报。

1. 在 Cloudyn 门户顶部的菜单中，选择“成本” > “成本分析” > “时段实际成本”。 
2. 将“组”设置为“服务”，并将“按服务筛选”设置为“Azure/VM”。 
3. 在报表的右上角，选择“操作”，然后选择“计划报表”。
4. 若要按计划的时间间隔向自己发送有关报表的电子邮件，请在“保存或计划此报表”对话框中选择“计划”选项卡。 请务必选择“通过电子邮件发送”。 你使用的任何标记、分组和筛选均包含在通过电子邮件发送的报表中。 
5. 选择“阈值”选项卡，然后选择“实际成本与阈值”。 
   1. 在“红色警报”阈值框中，输入 10000。 
   2. 在“黄色警报”阈值框中，输入 9000。 
   3. 在“连续警报数”框中，输入要接收的连续警报的数目。 当你收到的警报总数达到指定数目时，就不会再发送任何其他警报。 
6. 选择“保存”。

    ![基于支出阈值显示红色和黄色警报的示例](./media/billing-getting-started/schedule-alert01.png)

还可以选择“成本百分比与预算”阈值指标来创建警报**。 这样就可以将阈值指定为预算的百分比而不是货币值。

## <a name="ways-to-monitor-your-costs-when-using-azure-services"></a>使用 Azure 服务时监视成本的方法

### <a name="tags"></a>对资源添加标记以便对计费数据进行分组

可以使用标记来对受支持服务的计费数据进行分组。 例如，当运行不同团队的多个 VM 时，则可以使用标记按成本中心（人力资源、市场营销、财务）或环境（生产，预生产、测试）对成本进行分类。 

![显示在门户中设置标记的屏幕截图](./media/billing-getting-started/tags.PNG)

各标记显示了完全不同的成本报告视图。 例如，在[成本分析视图](#costs)中可立即查看，而在详细使用情况 .csv 中则需在第一个计费期之后才能查看。

有关详细信息，请参阅[使用标记来组织 Azure 资源](../azure-resource-manager/resource-group-using-tags.md)。

### <a name="costs"></a>定期查看门户中的成本明细和消耗率

服务开始运行后，请定期查看其费用。 可在 Azure 门户中查看当前费用和消耗率。

1. 访问 [Azure 门户中的“订阅”](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)并选择一个订阅。

2. 如果你的订阅支持该功能，则可以查看成本明细和消耗率。

    ![Azure 门户中消耗率和明细的屏幕截图](./media/billing-getting-started/burn-rate.PNG)

3. 将列表中的“成本分析”单击到左侧，以便按资源查看成本明细。 添加服务后请等待 24 小时让数据进行填充。

    ![Azure 门户中成本分析视图的屏幕截图](./media/billing-getting-started/cost-analysis.PNG)

4. 可按[标记](#tags)、资源类型、资源组和时间跨度等不同的属性进行筛选。 单击“应用”确认筛选条件，如果想要将视图导出为逗号分隔值 (.csv) 文件，请单击“下载”。

5. 此外，还可以单击资源，查看每日费用历史记录以及每天的资源成本。

    ![Azure 门户中费用历史记录视图的屏幕截图](./media/billing-getting-started/costhistory.PNG)

建议将你看到的成本与你之前选择了服务时看到的估计值进行对比。 如果成本和估计值相差很大，请再次查看之前为资源选择的定价计划。

### <a name="consider-enabling-cost-cutting-features-like-auto-shutdown-for-vms"></a>对于 VM，建议启用自动关闭等降低成本的功能

可在 Azure 门户中为 VM 配置自动关闭，具体视方案而定。 有关详细信息，请参阅[使用 Azure 资源管理器为 VM 设置自动关闭功能](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/)。

![门户中的自动关闭选项的屏幕截图](./media/billing-getting-started/auto-shutdown.PNG)

自动关闭与在 VM 中使用电源选项进行关闭不同。 自动关闭会停止并解除分配 VM，以便停止额外的使用费。 有关详细信息，请参阅 [Linux VM](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) 和 [Windows VM](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) 有关 VM 状态的定价常见问题解答。

有关开发和测试环境的更多降低成本功能，请参阅 [Azure 开发测试实验室](https://azure.microsoft.com/services/devtest-lab/)。

### <a name="turn-on-and-check-out-azure-advisor-recommendations"></a>打开和查看 Azure 顾问建议

[Azure 顾问](../advisor/advisor-overview.md)是一项功能，通过识别很少使用的资源来帮助降低成本。 在 Azure 门户访问顾问：

![Azure 门户中的 Azure 顾问按钮的屏幕截图](./media/billing-getting-started/advisor-button.PNG)

然后可在顾问仪表板的“成本”选项卡中获取可行性建议：

![顾问成本建议示例的屏幕截图](./media/billing-getting-started/advisor-action.PNG)

有关详细信息，请参阅[顾问成本建议](../advisor/advisor-cost-recommendations.md)。

## <a name="reviewing-costs-at-the-end-of-your-billing-cycle"></a>在计费周期结束时查看成本

在计费周期结束后，发票将变为可用。 还可以[下载过去的发票和详细使用情况文件](billing-download-azure-invoice-daily-usage-date.md)，确保收费正确。 有关比较每日使用情况和发票的详细信息，请参阅[了解 Microsoft Azure 帐单](billing-understand-your-bill.md)。

### <a name="billing-api"></a>计费 API

使用计费 API 以编程方式获取使用情况数据。 同时使用 RateCard API 和使用情况 API 获取计费使用情况。 有关详细信息，请参阅[深入了解 Microsoft Azure 资源消耗](billing-usage-rate-card-overview.md)。

## <a name="other-offers"></a>其他资源和特殊情况

### <a name="ea-csp-and-sponsorship-customers"></a>EA、CSP 和赞助客户
请咨询帐户管理员或 Azure 合作伙伴以开始使用。

| 产品/服务 | 资源 |
|-------------------------------|-----------------------------------------------------------------------------------|
| 企业协议 (EA) | [EA 门户](https://ea.azure.com/)、[帮助文档](https://ea.azure.com/helpdocs)和[ Power BI 报表](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-enterprise/) |
| 云解决方案提供商 (CSP) | 咨询提供商 |
| Azure 赞助 | [赞助门户](https://www.microsoftazuresponsorships.com/) |

如果管理大型组织的 IT，建议阅读 [Azure 企业基架](/azure/architecture/cloud-adoption-guide/subscription-governance)和[企业 IT 白皮书](https://download.microsoft.com/download/F/F/F/FFF60E6C-DBA1-4214-BEFD-3130C340B138/Azure_Onboarding_Guide_for_IT_Organizations_EN_US.pdf)（.pdf 下载，仅英文版）。

#### <a name="EA"></a>在 Azure 门户中预览“企业协议”成本视图 

企业成本视图目前为公共预览版。 注意事项：

- 订阅费用基于使用情况，并且不包括预付金额、超额用量、已包含数量、调整和税。 实际费用按注册级计算。
- Azure 门户中显示的金额可能不同于企业门户中显示的金额。 企业门户中的更新可能要花费几分钟时间来完成，之后，更改才会显示在 Azure 门户中。
- 如果未看到成本，可能是以下几种原因之一所致：
    - 你在订阅级别没有权限。 若要查看企业成本视图，必须在订阅级别是账单读者、读者、参与者或所有者。
    - 你是帐户所有者且你的注册管理员已禁用“AO 查看费用”设置。  请联系你的注册管理员以获取费用访问权限。 
    - 你是部门管理员且你的注册管理员已禁用“DA 查看费用”设置。  联系你的注册管理员以获取访问权限。
    - 你是从某个渠道合作伙伴购买的 Azure，而该合作伙伴未发布定价信息。  
- 如果你在企业门户中更新与成本访问相关的设置，则更改过几分钟后才会显示在 Azure 门户中。
- 支出限制和发票指南不适用于 EA 订阅。

### <a name="check-your-subscription-and-access"></a>查看订阅和访问权限

若要查看成本，必须具有[对计费信息的订阅级访问权限](billing-manage-access.md)。 只有帐户管理员可以访问[帐户中心](https://account.azure.com/Subscriptions)、更改计费信息以及管理订阅。 帐户管理员是完成注册过程的人员。 有关详细信息，请参阅[添加或更改管理订阅或服务的 Azure 管理员角色](billing-add-change-azure-subscription-administrator.md)。

若要查看你是否为帐户管理员，请转到 [Azure 门户中的“订阅”](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。 查看你有权访问的订阅的列表。 在“我的角色”下查看。 如果显示*帐户管理员*，则可以进行操作。 如果显示所有者等其他信息，则表示没有完全权限。

![Azure 门户中订阅视图中的角色的屏幕截图](./media/billing-getting-started/sub-blade-view.PNG)

如果你不是帐户管理员，则某人也许可以使用 [Azure Active Directory 基于角色的访问控制](../role-based-access-control/role-assignments-portal.md) (RBAC) 向你授予部分访问权限。 若要管理订阅以及更改计费信息，请[查找帐户管理员](billing-subscription-transfer.md#whoisaa)。请求帐户管理员来执行任务，或者[将订阅转让给你](billing-subscription-transfer.md)。

当帐户管理员不再属于组织，并且需要管理帐单时，请[联系我们](https://go.microsoft.com/fwlink/?linkid=2083458)。


### <a name="how-to-request-a-service-level-agreement-credit-for-a-service-incident"></a>如何请求服务事件的服务级别协议信用额度

服务级别协议 (SLA) 描述 Microsoft 关于运行时间和连接性方面的承诺。 Azure 服务的影响正常运行时间或连接，通常称为"中断"。 遇到的问题时，会报告服务事件 如果我们不是实现 SLA 中所述为每个服务维护服务级别，然后你可能符合用于你的每月服务费用的一部分的信用额度。

若要请求信用额度：

1. 登录到 [Azure 门户](https://portal.azure.com/)。 如果有多个帐户，请确保您使用一个受 Azure 停机时间。 这有助于支持自动收集必要的背景信息和更快地解决这种情况。
2. 创建新的支持请求。
3. 下**问题类型**，选择**计费**。
4. 下**问题类型**，选择**退款请求**。
5. 添加详细信息，以指定你要让 for SLA 信用额度，提到日期/时间/时区，以及受影响的服务 （如 Vm、 网站等）。
6. 验证你的联系人详细信息，并选择**创建**按钮以提交你的请求。

SLA 阈值因服务而异。 例如，SQL Web 层提供 99.9%的 SLA、 虚拟机具有 99.95%的 SLA 和 SQL 标准层提供 99.99%的 SLA。

对于某些服务，没有要应用的 SLA 的先决条件。 例如，虚拟机必须具有同一可用性集中部署两个或多个实例。

有关详细信息，请参阅[服务级别协议](https://azure.microsoft.com/support/legal/sla/)文档并[的 Azure 服务的 SLA 摘要](https://azure.microsoft.com/support/legal/sla/summary/)文档。

## <a name="need-help-contact-us"></a>需要帮助？ 请联系我们。

如果有疑问或需要帮助，请[创建支持请求](https://go.microsoft.com/fwlink/?linkid=2083458)。
