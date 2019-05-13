---
title: 创建 Microsoft 开发人员帐户 |Azure Marketplace
description: 创建 Microsoft 开发人员帐户的要求和步骤。
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 4fde5d81fb97bec23fdb46ff53b05874c88d9d67
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64935862"
---
<a name="create-a-microsoft-developer-account"></a>创建 Microsoft 开发人员帐户
====================================

本文介绍如何成为一名批准的 Microsoft 开发人员，以便能够在 Azure 市场中发布产品。

## <a name="create-a-microsoft-account"></a>创建 Microsoft 帐户

若要启动发布过程，需要完成 **Microsoft 开发人员中心**注册。 在**[云合作伙伴门户](https://cloudpartner.azure.com/)** 中，你将使用同一个已注册的帐户来启动发布过程。

### <a name="general-account-guidelines"></a>一般帐户准则

我们建议只对 Azure 市场套餐使用一个 Microsoft 帐户。 此帐户不应特定于服务或套餐。

组成用户名的地址应位于域上，并受 IT 团队控制。 所有与发布有关的活动都应通过此帐户完成。

>[!WARNING]
>Microsoft 帐户注册不支持“Azure”和“Microsoft”等单词。 若要完成帐户创建和注册过程，请避免使用这些单词。

### <a name="company-account-guidelines"></a>公司帐户准则

如果多个用户需要通过使用建立该帐户的 Microsoft 帐户登录来访问该帐户，请遵循以下准则。

>[!IMPORTANT]
>要允许多个用户访问开发人员中心帐户，我们建议使用 Azure Active Directory 为单个用户分配角色。 这些用户可以通过使用其个人 Azure AD 凭据登录来访问该帐户。 有关详细信息，请参阅[管理帐户用户](https://docs.microsoft.com/windows/uwp/publish/manage-account-users)。

-   使用属于公司的域（而不是属于个人）的电子邮件地址创建 Microsoft 帐户。 例如 windowsapps\@fabrikam.com。
-   限制只有尽可能最少数量的开发人员可以访问此 Microsoft 帐户。
-   设置公司电子邮件通讯组列表，其中包括需要访问开发人员帐户的所有人员，并将此电子邮件地址添加到安全信息。 在需要的时候，这会允许列表中的所有员工接收安全代码以及管理 Microsoft 帐户的安全信息。 如果设置通讯组列表不可行，则将需要个人电子邮件帐户的所有者可供访问，并在出现提示时（例如，将新的安全信息添加到该帐户时或必须从新设备访问该帐户时）共享安全代码。
-   添加不需要分机且主要团队成员可以访问的公司电话号码。
-   一般情况下，让开发人员使用受信任的设备登录公司的开发人员帐户。 所有主要团队成员应有权访问这些受信任的设备。 这将减少访问帐户时要发送安全代码的需要。
-   如果需要允许从不受信任的电脑访问该帐户，则限制最多只有五个开发人员可以进行该访问。 理想情况下，这些开发人员应从共用同一地理位置和网络位置的计算机访问该帐户。
-   经常查看[公司的安全信息](https://account.live.com/proofs/Manage)以确保这些信息都是最新的。

>[!IMPORTANT]
>应主要从受信任的电脑访问开发人员帐户。 这很关键，因为对每个帐户每周生成的代码数有限制。 它还确保提供最无缝的登录体验。
>
>有关详细信息，请参阅[其他开发人员帐户准则和安全性](https://msdn.microsoft.com/windows/uwp/publish/opening-a-developer-account#additional-guidelines-for-company-accounts)。

### <a name="to-create-a-microsoft-account"></a>创建 Microsoft 帐户

1.  打开新的 Chrome Incognito 或 Internet Explorer InPrivate 浏览会话，以确保未登录到现有帐户。
2.  使用此[链接](https://signup.live.com/signup.aspx)（按照上述准则）将电子邮件注册为 Microsoft 帐户。 遵照以下注册说明操作：

    - 将帐户注册为 Microsoft 帐户时，需要提供有效电话号码，供系统以文本消息或自动呼叫的形式向你发送帐户验证代码。
    - 将帐户注册为 Microsoft 帐户时，需要提供有效的电子邮件 ID，以接收用于帐户验证的自动发送电子邮件。
    - 验证已发送到 DL 的电子邮件地址。

    现在，可以在 Microsoft 开发人员中心中使用新的 Microsoft 帐户。

## <a name="register-your-account-in-microsoft-developer-center"></a>在 Microsoft 开发人员中心注册帐户

Microsoft 开发人员中心用于注册一次公司信息。 注册者必须是有效的公司代表，并且必须提供其个人信息（验证身份所用方法）。 注册人必须使用公司共享的 Microsoft 帐户，**并且必须在云合作伙伴门户中使用同一帐户。** 应检查以确保公司尚未拥有 Microsoft 开发人员中心帐户，然后再尝试创建该帐户。 在该过程中，我们会收集公司地址信息、银行帐户信息和税务信息。 这些信息通常可从财务或业务联系人处获取。

>[!IMPORTANT]
>必须填写以下开发人员个人资料，才能进入产品/服务创建和部署的各个阶段。

| 开发人员个人资料     | 开始起草    | 过渡       | 发布免费模板和解决方案模板   | 发布商业广告   |
|---------------------- |----------------   |-----------    |-------------------------------------  |---------------------  |
| 公司注册  | 必须具有         | 必须具有     | 必须具有                             | 必须具有             |
| 税务配置文件 ID        | 可选          | 可选      | 可选                              | 必须具有             |
| 银行帐户          | 可选          | 可选      | 可选                              | 必须具有             |

>[!NOTE]
>自带许可 (BYOL) 仅支持用于虚拟机，并视为免费产品。

### <a name="register-your-company-account"></a>注册公司帐户

1. 打开新的 Internet Explorer InPrivate 或 Chrome Incognito 浏览会话，以确保未登录到个人帐户。

2. 转到 [Windows 开发人员中心](https://dev.windows.com/registration?accountprogram=azure)，将自己注册为卖家。 请在继续操作之前，阅读以下重要说明。

   ![Microsoft 帐户验证](./media/cloud-partner-portal-create-dev-center-registration/seller-dashboard-verify.jpg)

    >[!IMPORTANT]
    >确保用于在开发人员中心中注册的电子邮件 ID 或通讯组列表（建议使用通讯组列表，以删除个人的依赖性）是首次注册为 Microsoft 帐户。 如果不是，请使用此链接进行注册。 此外，Microsoft 公司域下的任何电子邮件 ID 都不能用于开发人员中心注册。

   ![开发人员中心登录](./media/cloud-partner-portal-create-dev-center-registration/seller-dashboard-login.jpg)

3. 运行“帮助我们保护你的帐户”向导，以使用电话号码或电子邮件地址验证身份。

4. 在“注册帐户信息”中，从下拉列表中选择“帐户所在国家/地区”，然后选择“下一步”。

   ![选择国家/地区](./media/cloud-partner-portal-create-dev-center-registration/imgRegisterCo_04.png)

    >[!WARNING]
    >"销售 from"国家/地区：若要销售你在 Azure Marketplace 上的服务，注册的实体必须是从一个已批准"sell-from"国家/地区的下拉列表中所示。 此限制是出于付款和税务的原因。 有关详细信息，请参阅“市场参与策略”。

5. 选择“公司”作为“帐户类型”，然后选择“下一步”。

    >[!IMPORTANT]
    >若要更好地了解帐户类型并确定最适合自己的类型，请查看以下屏幕截图中所示的“帐户类型、位置和费用”页

    ![卖方的帐户类型](./media/cloud-partner-portal-create-dev-center-registration/imgRegisterCo_05.png)

6. 输入“发布者显示名称”。 这通常是公司的名称。

    >[!NOTE]
    >套餐列出后，开发人员中心中输入的发布者显示名称不会显示在 Azure 市场中。 但若要完成注册过程，必须填写此信息。

7. 输入用于帐户验证的“联系人信息”。

    >[!IMPORTANT]
    >必须提供准确的联系人信息，因为该信息用于对公司在开发人员中心中获得批准的验证过程。

8. 输入“公司审批者”的联系人信息。 公司审批者是如下人员：可以验证你是否有权代表组织在开发人员中心中创建帐户。 提供此信息后，选择“下一步”转到“付款”部分。

    ![确定公司审批者](./media/cloud-partner-portal-create-dev-center-registration/imgRegisterCo_08.png)

9. 输入帐户的付款信息。 如果有抵扣注册费用的促销代码，可以在此处输入该代码。 否则，请提供信用卡信息（或所在市场支持的 PayPal）。 选择“下一步”转到最终的“检查”屏幕。

   ![付款注册](./media/cloud-partner-portal-create-dev-center-registration/imgRegisterCo_09.png)

10. 检查帐户信息，确认所有内容都正确无误。 阅读并接受 [Microsoft Azure 市场发布者协议](https://go.microsoft.com/fwlink/?LinkID=699560)的条款和条件。 选中该框，即表明已阅读并接受这些条款。

11. 选择“完成”以确认注册。 一封确认邮件将发送到你的电子邮件地址。

12. 如果你只是打算发布免费套餐，请选择“[转到云合作伙伴门户](https://cloudpartner.azure.com/)”，然后跳转到本文的“在云合作伙伴门户中注册帐户”部分。

### <a name="commercial-offers"></a>商业套餐

如果你打算发布商业套餐（例如，带有每小时计费模型的虚拟机套餐），则必须提供税务和银行信息。 为此，请登录到开发人员中心帐户，并选择“更新帐户信息”。 遵照下一部分“添加银行和税务信息”中的说明操作。

>[!IMPORTANT]
>在未提供银行帐户和税务信息的情况下，无法将商业套餐投入生产。

如果以后想要更新银行和税务信息，可以跳转到本文的“在云合作伙伴门户中注册帐户”部分。

>[!NOTE]
>我们建议尽早提供银行帐户和税务信息，因为验证税务信息需要一些时间。

### <a name="add-banking-and-tax-information"></a>添加银行和税务信息

若要发布供购买的商业套餐，需要在开发人员中心添加付款和税务信息，并提交该信息进行验证。

**提供银行信息**

1.  使用 Microsoft 帐户登录到 [Microsoft 开发人员中心](https://dev.windows.com/registration?accountprogram=azure)。
2.  在左侧菜单中选择“付款帐户”，在“选择付款方式”下选择“银行帐户”或“PayPal”。

    >[!NOTE]
    >如果拥有客户在市场购买过的商业产品/服务，该帐户即用于对这些购买进行收款。
3.  输入付款信息，然后选择“保存”。

    >[!IMPORTANT]
    >如果需要更新或更改付款帐户，请遵循上述步骤，将当前信息替换为新信息。
    >
    >更改付款帐户可能会推迟付款（最多一个付款周期）。 出现此推迟的原因是我们需要验证帐户更改，正如首次设置付款帐户时我们会进行验证。 帐户通过验证后，仍会收到全额款项；当前付款周期应得的任何付款将添加到下一个付款周期。

4.  选择“**下一步**”。

**提供税务信息**

1.  必要时使用 Microsoft 帐户登录到 [Microsoft 开发人员中心](https://dev.windows.com/registration?accountprogram=azure)。
2.  在左侧菜单中选择“税务配置文件”。
3.  在“设置税务表单”页上：
    - 选择你永久居留的国家或地区。
    - 选择你拥有主要国籍的国家或地区。
    - 选择“**下一步**”。
4.  输入税务详细信息，然后选择“下一步”。

>[!WARNING]
>如果未在 Microsoft 开发人员中心帐户中提供银行帐户和税务信息，则你无法将商业套餐投入生成。

### <a name="developer-center-registration-issues"></a>开发人员中心注册问题

如果执行开发人员中心注册时遇到问题，请使用以下步骤开具支持票证。

1.  转到[支持链接](https://developer.microsoft.com/windows/support)。
2.  在“联系我们”下，选择“提交事件”。
    ![开具票证](./media/cloud-partner-portal-create-dev-center-registration/imgAddTax_02.png)
3.  选择“开发人员中心帮助”作为“问题类型”，选择“发布和管理应用”作为“类别”。 选择“启动电子邮件”。

    ![确定问题的类型](./media/cloud-partner-portal-create-dev-center-registration/imgAddTax_03.png)

4.  此时会显示登录页。 请使用任何 Microsoft 帐户登录。 如果没有 Microsoft 帐户，请[创建一个](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1)。\

5.  提供问题的详细信息，然后选择“提交”以发送票证。

    ![提交票证](./media/cloud-partner-portal-create-dev-center-registration/imgAddTax_05.png)

## <a name="register-your-account-in-the-cloud-partner-portal"></a>在云合作伙伴门户中注册帐户

使用[云合作伙伴门户](https://cloudpartner.azure.com/)发布和管理套餐。

1.  打开新的 Chrome Incognito 或 Internet Explorer InPrivate 浏览会话，以确保未登录到个人帐户。
2.  转到[云合作伙伴门户](https://cloudpartner.azure.com/)。
3.  如果你是新用户并且是首次登录到[云合作伙伴门户](https://cloudpartner.azure.com/)，则必须使用注册到开发人员中心帐户的同一电子邮件 ID 登录。 这可以确保开发中心帐户和云合作伙伴门户帐户相互链接。

以后可以添加公司中使用该应用程序的其他成员。 可以在云合作伙伴门户中遵循下一部分所述的步骤将他们添加为参与者或所有者。

如果被添加为云合作伙伴门户中的参与者/所有者，则可以使用自己的帐户进行登录。

>[!TIP]
>Azure 网站上对这些参与政策做了介绍。

## <a name="manage-users-as-owners-or-contributors-in-the-cloud-partner-portal"></a>在云合作伙伴门户中管理充当所有者或参与者的用户

[管理云合作伙伴门户中的用户的步骤](./cloud-partner-portal-manage-users.md)


## <a name="next-steps"></a>后续步骤

由于帐户已创建和注册，因此可以启动 Azure 市场发布过程。
