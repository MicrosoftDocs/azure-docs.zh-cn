---
title: 逻辑应用体验版 |Azure Marketplace
description: 介绍如何生成与 Dynamics AX/CRM 实例或 Azure 外部的其他任何资源连接的体验版。
services: Azure, Marketplace, Cloud Partner Portal,
author: pbutlerm
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 3dc5642541f411acc93301f6bb9828e901aba449
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64940010"
---
<a name="logic-app-test-drive"></a>逻辑应用体验版
====================

本文面向已在 AppSource 中发布了套餐，并想要生成与 Dynamics AX/CRM 实例或 Azure 外部的其他任何资源连接的体验版的发布者。

<a name="how-to-build-a-logic-app-test-drive"></a>如何生成逻辑应用体验版
-----------------------------------

用于[运营](https://github.com/Microsoft/AppSource/blob/master/Setup-your-Azure-subscription-for-Dynamics365-Operations-Test-Drives.md)和[客户互动](https://github.com/Microsoft/AppSource/wiki/Setting-up-Test-Drives-for-Dynamics-365-app)的逻辑应用体验版的文档目前仍在 GitHub 上提供，请转到 GitHub 详细阅读。

<a name="how-to-publish-a-test-drive"></a>如何发布体验版
---------------------------

生成体验版后，请阅读本部分，逐步了解成功发布体验版所要填写的每个字段。

![启用体验版功能](./media/azure-resource-manager-test-drive/howtopub1.png)

第一个（也是最重要的）字段是选择是否要显示包含所有必填字段的测试窗体以便填写。如果选择“否”，则会禁用该窗体；在禁用体验版的情况下重新发布时，会从生产环境中删除体验版。

*注意*：如果用户正在活跃地使用任何体验版，则这些体验版将继续运行，直到用户的会话过期。

### <a name="details"></a>详细信息

要填写的下一个部分是体验版产品/服务的详细信息。

![体验版详细信息](./media/azure-resource-manager-test-drive/howtopub2.png)

**说明 -** [必填字段] 编写有关体验版的功能的主要说明。 客户将在此处阅读产品体验版涵盖的方案。 

**用户手册 -** [必填字段] 体验版体验的深入演练。 客户将打开此文档，并完全根据其中的说明演练整个体验版的操作。 此内容必须容易理解和遵循！ （必须是 .pdf 文件）

体验版演示视频 - 建议填写。类似于用户手册，最好是在其中包含体验版体验的视频教程。\[\] 客户在使用体验版之前和使用期间将会观看此视频，并完全根据其中的说明演练整个体验版的操作。 此内容必须容易理解和遵循！

- **名称** - 视频的标题
- **链接** - 必须是 YouTube 或 Vimeo 的嵌入式 URL。 下面是有关如何获取嵌入式 URL 的示例：
- 缩略图 - 必须是优质图像（533x324 像素）。 建议在此处插入体验版体验某些部分的屏幕截图。

下面显示了在客户使用体验版体验期间，如何向他们显示这些字段。

![体验版字段的外观](./media/azure-resource-manager-test-drive/howtopub4.png)

### <a name="technical-configuration"></a>技术配置

要填写的下一个部分是指定在何处配置体验版逻辑应用，并定义体验版实例的具体工作方式。

![体验版技术配置](./media/azure-resource-manager-test-drive/howtopub5_logicapp.png)

- **区域**  -  [必填字段] 将在所选的区域中部署体验版逻辑应用资源。

    *注意：* 如果逻辑应用在某个区域中存储了任何自定义资源，请务必在此处选择该区域。 在此处输入此区域之前，最好是**在门户中将逻辑应用完全部署到 Azure 订阅本地，并验证它是否正常工作**。

- **最大并发体验版数目**  -  [必填字段] 每个选定区域中已部署的和正在等待访问的体验版实例数。 客户无需等待部署，就能立即访问此体验版。

    *注意：* 如果运行的网络研讨会/课程需要所有 N 个学生使用体验版，则我们建议发布 N 个热实例，然后在课程结束后，重新发布为正常数目的热实例。

- **体验版持续时间(小时) –** [必填字段] 体验版保持活动状态的持续时间，以小时为单位。 此时间段结束后，体验版会自动终止。

- **Azure 资源组名称 -** [必填字段] 输入用于保存逻辑应用体验版的资源组名称。

- **分配逻辑应用名称 -** [必填字段] 输入在客户获取体验版之前用于在体验版中分配用户的逻辑应用；请在此处输入该逻辑应用的名称。 请确保此文件保存在上述资源组中。

- **取消预配逻辑应用名称 -** [必填字段] 输入用于取消预配体验版中创建的所有资源的逻辑应用名称。 请确保此文件保存在上述资源组中。

- **访问信息 -** [必填字段] 客户获取体验版后，会向他们显示访问信息。 这些说明旨在共享体验版资源管理器模板中的有用输出参数。 若要包含输出参数，请使用双大括号（例如 {{outputname}}），这样，它们就会正确插入到相应的位置。 （建议在此处使用 HTML 字符串格式，以便在前端正确显示）。

### <a name="test-drive-deployment-subscription-details"></a>体验版部署订阅详细信息

要填写的最后一个部分是确保能够通过连接 Azure 订阅和 Azure Active Directory (AD) 来自动部署体验版。

![体验版部署订阅详细信息](./media/azure-resource-manager-test-drive/subdetails1.png)

**Azure 订阅 ID -** [必填字段] 授予对 Azure 服务和 Azure 门户的访问权限。 将在该订阅中报告资源用量和计收服务费用。 如果你没有一个专门用于体验版的**独立** Azure 订阅，请创建一个订阅。 登录到 Azure 门户并导航到左侧菜单中的“订阅”，即可找到 Azure 订阅 ID。
（示例：“a83645ac-1234-5ab6-6789-1h234g764ghty”）

![Azure 订阅](./media/azure-resource-manager-test-drive/subdetails2.png)

**Azure AD 租户 ID -** [必填字段] 如果已有一个租户 ID，可在“属性”-\>“目录 ID”下面找到它。

![Azure Active Directory](./media/azure-resource-manager-test-drive/subdetails3.png)

否则，请在 Azure Active Directory 中创建一个新租户。

![Azure Active Directory 属性屏幕](./media/azure-resource-manager-test-drive/subdetails4.png)

!Azure Active Directory](./media/azure-resource-manager-test-drive/subdetails5.png)

![Azure Active Directory 租户](./media/azure-resource-manager-test-drive/subdetails6.png)

**Azure AD 应用 ID -** [必填字段] 下一步是创建并注册新应用程序。 我们将使用此应用程序对体验版实例执行操作。

1. 导航到新建的目录或现有目录，然后在筛选器窗格中选择“Azure Active Directory”。
2. 搜索“应用注册”，然后单击“添加”
3. 提供应用程序名称。
4. 选择“Web 应用/API”作为“类型”
5. 在“登录 URL”中提供任意值。我们不会使用该字段。
6. 单击“创建”。
7. 创建应用程序后，转到“属性”-\>“将应用程序设为多租户”，然后点击“保存”。

单击“保存”。 最后一步是获取已注册应用的应用程序 ID，并将其粘贴到此处的体验版字段中。

![Azure Active Directory 应用程序 ID](./media/azure-resource-manager-test-drive/subdetails7.png)

由于我们要使用该应用程序部署到订阅，因此需将该应用程序添加为订阅中的参与者。 下面是操作说明：

1. 导航到“订阅”边栏选项卡，并选择专门用于体验版的相应订阅。
1. 单击“访问控制(IAM)”。
1. 单击“角色分配”选项卡。![在 Azure Active Directory 中添加新的访问控制主体](./media/azure-resource-manager-test-drive/SetupSub7_1.jpg)
1. 单击“添加角色分配”。
1. 将角色设置为“参与者”。
1. 键入 Azure AD 应用程序的名称，并选择要分配该角色的应用程序。
    ![Azure Active Directory 权限](./media/azure-resource-manager-test-drive/SetupSub7_2.jpg)
1. 单击“ **保存**”。

**Azure AD 应用密钥 -** [必填字段] 最终的字段将会生成身份验证密钥。 在密钥下添加密钥说明，将持续时间设置为永不过期，然后选择“保存”。 必须避免使用已过期的密钥，否则，在生产环境中体验版将会中断。 复制此值，并将其粘贴到所需的体验版字段中。

![Azure Active Directory 密钥部分](./media/azure-resource-manager-test-drive/subdetails8.png)

> [!CAUTION]
> 无法使用 Azure 应用程序注册预览版，因为当前生成 base64 编码密钥。


<a name="next-steps"></a>后续步骤
----------

填写所有体验版字段后，请继续重新发布产品/服务。 在体验版通过认证过程后，应该对套餐**预览版**中的客户体验进行广泛的测试。 在 UI 中启动体验版，并验证是否已正确地完整部署了体验版。

请务必注意，不要删除为客户预配的任何体验版组成部分，以便在客户用完体验版之后，体验版服务自动清理这些资源组。

如果对预览版套餐感到满意，现在就可以将它**上线**！ 发布产品/服务后，Microsoft 会完成最终评审过程，以仔细检查完整的端到端体验。 如果出于某种原因拒绝了该套餐，我们会向套餐的工程联系人发送通知，解释需要解决哪些问题。

如果你有其他问题、寻求故障排除建议，或想要让体验版获得更大的成功，请访问[常见问题解答、故障排除和最佳做法](./marketing-and-best-practices.md)。
