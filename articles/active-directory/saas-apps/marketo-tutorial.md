---
title: 教程：Azure Active Directory 与 Marketo 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Marketo 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: b88c45f5-d288-4717-835c-ca965add8735
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/19/2019
ms.author: jeedes
ms.openlocfilehash: 810d19a697e32f2d8e389bf58d52e82529bdd66b
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64917442"
---
# <a name="tutorial-azure-active-directory-integration-with-marketo"></a>教程：Azure Active Directory 与 Marketo 的集成

本教程介绍了如何将 Marketo 与 Azure Active Directory (Azure AD) 进行集成。
将 Marketo 与 Azure AD 集成可提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 Marketo。
* 可以让用户使用其 Azure AD 帐户自动登录到 Marketo（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Marketo 的集成，需要具有以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 Marketo 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Marketo 支持 IDP 发起的 SSO

## <a name="adding-marketo-from-the-gallery"></a>从库中添加 Marketo

要配置 Marketo 与 Azure AD 的集成，需要从库中将 Marketo 添加到托管 SaaS 应用列表。

**若要从库中添加 Marketo，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“Marketo”，在结果面板中选择“Marketo”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 Marketo](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，将基于名为“Britta Simon”的测试用户使用 Marketo 配置和测试 Azure AD 单一登录。
若要使单一登录有效，需要在 Azure AD 用户与 Marketo 相关用户之间建立链接关系。

若要配置并测试 Marketo 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Marketo 单一登录](#configure-marketo-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Marketo 测试用户](#create-marketo-test-user)** - 在 Marketo 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Marketo 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Marketo”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“设置 SAML 单一登录”页上，执行以下步骤：

    ![Marketo 域和 URL 单一登录信息](common/idp-intiated.png)

    a. 在“标识符”文本框中，使用以下模式键入 URL：`https://saml.marketo.com/sp`

    b. 在“回复 URL”文本框中，使用以下模式键入 URL：`https://login.marketo.com/saml/assertion/\<munchkinid\>`

    > [!NOTE]
    > 这些不是实际值。 请使用实际标识符和回复 URL 更新这些值。 请联系 [Marketo 客户端支持团队](https://investors.marketo.com/contactus.cfm)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

6. 在“设置 Marketo”部分中，根据要求复制相应 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-marketo-single-sign-on"></a>配置 Marketo 单一登录

1. 若要获取应用程序的 Munchkin Id，请使用管理员凭据登录到 Marketo 并执行以下操作：
   
    a. 使用管理员凭据登录到 Marketo 应用。
   
    b. 单击顶部导航窗格上的“管理员”按钮。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 导航到“集成”菜单，单击“Munchkin 链接”。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_11.png)
   
    d. 复制屏幕上显示的 Munchkin Id，并在 Azure AD 配置向导中填写回复 URL。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_12.png) 

2. 若要在应用程序中配置 SSO，请执行以下步骤：
   
    a. 使用管理员凭据登录到 Marketo 应用。
   
    b. 单击顶部导航窗格上的“管理员”按钮。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 导航到“集成”菜单，然后单击“单一登录”。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_07.png) 
   
    d. 若要启用 SAML 设置，请单击“编辑”按钮。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_08.png) 
   
    e. 已启用的单一登录设置。
   
    f. 在“颁发者 ID”文本框中粘贴“Azure AD 标识符”。
   
    g. 在“实体 ID”文本框中，输入 `http://saml.marketo.com/sp` 作为URL。
   
    h. 对于“名称标识符元素”，选择“用户 ID 位置”。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_09.png)
   
    > [!NOTE]
    > 如果用户标识符不是 UPN 值，则在“属性”选项卡中更改该值。
   
    i. 上传从 Azure AD 配置向导下载的证书。 保存设置。
   
    j. 编辑“重定向页面”设置。
   
    k. 在“登录 URL”文本框中粘贴“登录 URL”。
   
    l. 在“注销 URL”文本框中粘贴“注销 URL”。
   
    m. 在“错误 URL”中，复制 Marketo 实例 URL 并单击“保存”按钮以保存设置。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_10.png)

3. 若要为用户启用 SSO，请完成以下操作：
   
    a. 使用管理员凭据登录到 Marketo 应用。
   
    b. 单击顶部导航窗格上的“管理员”按钮。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. 导航到“安全性”菜单，并单击“登录设置”。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_13.png)
   
    d. 选中“需要 SSO”选项并保存设置。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_14.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户 

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中，按照以下步骤操作。

    ![“用户”对话框](common/user-properties.png)

    a. 在“名称”字段中，输入 BrittaSimon。
  
    b. 在“用户名”字段中，键入 brittasimon\@yourcompanydomain.extension  
    例如： BrittaSimon@contoso.com

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，将通过向 Britta Simon 授予对 Marketo 的访问权限，使其能够使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“Marketo”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Marketo”。

    ![应用程序列表中的 Marketo 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-marketo-test-user"></a>创建一个 Marketo 测试用户

在本部分中，会在 Marketo 中创建一个名为 Britta Simon 的用户。 请按照以下步骤在 Marketo 平台中创建用户。

1. 使用管理员凭据登录到 Marketo 应用。

2. 单击顶部导航窗格上的“管理员”按钮。
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_06.png) 

3. 导航到“安全性”菜单，单击“用户和角色”
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_19.png)  

4. 在“用户”选项卡上单击“邀请新用户”链接
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_15.png) 

5. 在“邀请新用户”向导中，填写以下信息
   
    a. 在文本框中输入用户**电子邮件**地址
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_16.png)
   
    b. 在文本框中输入**名字**
   
    c. 在文本框中输入**姓氏**
   
    d. 点击“下一步”

6. 在“权限”选项卡中，选择“用户角色”并单击“下一步”
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_17.png)
7. 单击“发送”按钮以发送用户邀请
   
    ![配置单一登录](./media/marketo-tutorial/tutorial_marketo_18.png)

8. 用户将收到电子邮件通知，且必须单击链接并更改密码以激活帐户。 

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Marketo 磁贴时，应当会自动登录到已为其设置了 SSO 的 Marketo。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

