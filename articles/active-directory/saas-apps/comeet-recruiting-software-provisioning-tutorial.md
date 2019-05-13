---
title: 教程：配置自动用户预配使用 Azure Active Directory Comeet 招聘软件 |Microsoft Docs
description: 了解如何配置 Azure Active Directory 自动预配和取消其预配用户帐户添加到 Comeet 招聘软件。
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: na
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: zchia
ms.openlocfilehash: e8414b9737e0ee7f847827a432dd9887931a2532
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470394"
---
# <a name="tutorial-configure-comeet-recruiting-software-for-automatic-user-provisioning"></a>教程：配置自动用户预配 Comeet 招聘软件

本教程的目的是为了演示在 Comeet 招聘软件和 Azure Active Directory (Azure AD) 若要配置 Azure AD 自动预配和取消其预配的用户和/或组到 Comeet 招聘软件中执行的步骤。

> [!NOTE]
> 本教程介绍在 Azure AD 用户预配服务之上构建的连接器。 有关此服务的功能、工作原理以及常见问题的重要详细信息，请参阅[使用 Azure Active Directory 自动将用户预配到 SaaS 应用程序和取消预配](../manage-apps/user-provisioning.md)。
>
> 此连接器目前以公共预览版提供。 有关常规 Microsoft Azure 的使用条款预览版功能的详细信息，请参阅[补充使用条款的 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>必备组件

本教程中概述的方案假定你已具有以下先决条件：

* Azure AD 租户
* [Comeet 招聘软件租户](https://www.comeet.co/)
* Comeet 招聘软件中具有管理员权限的用户帐户。

## <a name="add-comeet-recruiting-software-from-the-gallery"></a>从库中添加 Comeet 招聘软件

配置自动用户预配与 Azure AD 的 Comeet 招聘软件之前, 需要将 Comeet 招聘软件从 Azure AD 应用程序库添加到托管 SaaS 应用程序的列表。

**若要从 Azure AD 应用程序库中添加 Comeet 招聘软件，请执行以下步骤：**

1. 在中 **[Azure 门户](https://portal.azure.com)**，在左侧的导航窗格中，选择**Azure Active Directory**。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用程序”，并选择“所有应用程序”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新的应用程序，请选择**新的应用程序**窗格顶部的按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，输入**Comeet 招聘软件**，选择**Comeet 招聘软件**在结果面板中，然后单击**添加**按钮添加该应用程序。

    ![结果列表中的 Comeet Recruiting Software](common/search-new-app.png)

## <a name="assigning-users-to-comeet-recruiting-software"></a>将用户分配到 Comeet 招聘软件

Azure Active Directory 使用称为的概念*分配*来确定哪些用户应收到对所选应用的访问。 在自动用户预配的上下文中，同步仅用户和/或组已分配给 Azure AD 中的应用程序。

在配置和启用自动用户预配前, 应确定哪些用户和/或 Azure AD 中的组需要访问 Comeet 招聘软件。 确定后，您可以将这些用户和/或组到 Comeet 招聘软件分配按照此处的说明：

* [向企业应用分配用户或组](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-comeet-recruiting-software"></a>将用户分配到 Comeet 招聘软件的重要提示

* 建议将单个 Azure AD 用户分配到 Comeet 招聘软件，以测试自动用户预配配置。 其他用户和/或组可以稍后分配。

* 当将用户分配到 Comeet 招聘软件，您必须在分配对话框中选择任何有效的特定于应用程序角色 （如果可用）。 具有“默认访问权限”角色的用户排除在预配之外。

## <a name="configuring-automatic-user-provisioning-to-comeet-recruiting-software"></a>配置自动用户预配到 Comeet 招聘软件 

在 Azure AD 中，本部分指导您完成的步骤配置 Azure AD 预配服务以创建、 更新和禁用用户和/或组 Comeet 招聘软件中的基于用户和/或组分配。

> [!TIP]
> 您还可以选择启用基于 SAML 的单一登录的 Comeet 招聘软件中的说明提供[Comeet 招聘软件单一登录教程](comeetrecruitingsoftware-tutorial.md)。 可以独立于自动用户预配配置单一登录，尽管这两个功能互相补充。

### <a name="to-configure-automatic-user-provisioning-for-comeet-recruiting-software-in-azure-ad"></a>若要配置自动用户在 Azure AD 中预配 Comeet 招聘软件：

1. 登录到 [Azure 门户](https://portal.azure.com)。 选择**企业应用程序**，然后选择**的所有应用程序**。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Comeet Recruiting Software”。

    ![应用程序列表中的 Comeet Recruiting Software 链接](common/all-applications.png)

3. 选择“预配”选项卡。

    ![预配选项卡](common/provisioning.png)

4. 将“预配模式”设置为“自动”。

    ![预配选项卡](common/provisioning-automatic.png)

5. 下**管理员凭据**部分中，输入**租户 URL**并**机密令牌**Comeet 招聘软件的帐户在步骤 6 中所述。

6. 在中[Comeet 招聘软件管理控制台](https://app.comeet.co/)，导航到**Comeet > 设置 > 身份验证 > Microsoft Azure**，并将复制**机密令牌为你的公司**值设为**机密令牌**Azure AD 中的字段。

    ![Comeet 招聘软件设置](./media/comeet-recruiting-software-provisioning-tutorial/secret-token-1.png)

7. 填入步骤 5 中所示的字段后，单击**测试连接**以确保 Azure AD 可以连接到 Comeet 招聘软件。 如果连接失败，请确保你 Comeet 招聘软件的帐户具有管理员权限，然后重试。

    ![令牌](common/provisioning-testconnection-token.png)

8. 在“通知电子邮件”字段中，输入应接收预配错误通知的个人或组的电子邮件地址，并选中复选框“发生故障时发送电子邮件通知”。

    ![通知电子邮件](common/provisioning-notification-email.png)

9. 单击“ **保存**”。

10. 下**映射**部分中，选择**Azure Active Directory 用户同步到 Comeet**。

    ![Comeet 招聘软件用户映射](media/comeet-recruiting-software-provisioning-tutorial/user-mappings.png)

11. 查看从 Azure AD 同步到 Comeet 招聘软件中的用户属性**属性映射**部分。 为所选的属性**匹配**属性用于匹配 Comeet 招聘软件中以执行更新操作的用户帐户。 选择“保存”按钮以提交任何更改。

    ![Comeet 招聘软件组属性](media/comeet-recruiting-software-provisioning-tutorial/user-mapping-attributes.png)

12. 若要配置范围筛选器，请参阅[范围筛选器教程](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md)中提供的以下说明。

13. 若要启用 Azure AD 预配服务 Comeet 招聘软件，请更改**预配状态**到**上**中**设置**部分。

    ![预配状态切换](common/provisioning-toggle-on.png)

14. 选择所需的值中预配到 Comeet 招聘软件中定义的用户和/或组你想**作用域**中**设置**部分。

    ![预配作用域](common/provisioning-scope.png)

15. 已准备好预配时，单击“保存”。

    ![正在保存预配配置](common/provisioning-configuration-save.png)

此操作会对“设置”部分的“范围”中定义的所有用户和/或组启动初始同步。 初始同步执行的时间比后续同步长，只要 Azure AD 预配服务正在运行，大约每隔 40 分钟就会进行一次同步。 可以使用**同步详细信息**部分监视进度并跟踪指向预配活动报告，其中描述了由 Azure AD 预配服务对 Comeet 招聘软件执行所有操作。

若要详细了解如何读取 Azure AD 预配日志，请参阅[有关自动用户帐户预配的报告](../manage-apps/check-status-user-account-provisioning.md)。

## <a name="connector-limitations"></a>连接器限制

* Comeet 招聘软件当前不支持组。

## <a name="additional-resources"></a>其他资源

* [管理企业应用的用户帐户预配](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>后续步骤

* [了解如何查看日志并获取有关预配活动的报告](../manage-apps/check-status-user-account-provisioning.md)

