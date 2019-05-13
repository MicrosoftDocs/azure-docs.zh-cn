---
title: 教程：Azure Active Directory 与 Mozy Enterprise 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Mozy Enterprise 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 489b5e62-85c2-45c9-8766-326632d48114
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: jeedes
ms.openlocfilehash: 964d7af92b087bd9905113a25513e2b2f7a3df17
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64922340"
---
# <a name="tutorial-azure-active-directory-integration-with-mozy-enterprise"></a>教程：Azure Active Directory 与 Mozy Enterprise 集成

本教程介绍了如何将 Mozy Enterprise 与 Azure Active Directory (Azure AD) 集成。
将 Mozy Enterprise 与 Azure AD 集成具有以下优势：

* 可在 Azure AD 中控制谁有权访问 Mozy Enterprise。
* 可以让用户使用其 Azure AD 帐户自动登录到 Mozy Enterprise（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Mozy Enterprise 的集成，需备齐以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 Mozy Enterprise 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Mozy Enterprise 支持 SP 发起的 SSO

## <a name="adding-mozy-enterprise-from-the-gallery"></a>从库中添加 Mozy Enterprise

若要配置 Mozy Enterprise 与 Azure AD 的集成，需要从库中将 Mozy Enterprise 添加到托管 SaaS 应用列表。

若要从库中添加 Mozy Enterprise，请执行以下步骤：

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入“Mozy Enterprise”，从结果面板中选择“Mozy Enterprise”，再单击“添加”按钮以添加此应用程序。

     ![结果列表中的 Mozy Enterprise](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，将基于名为“Britta Simon”的测试用户配置并测试 Mozy Enterprise 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 Mozy Enterprise 相关用户之间建立链接关系。

若要配置并测试 Mozy Enterprise 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Mozy Enterprise 单一登录](#configure-mozy-enterprise-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. [创建 Mozy Enterprise 测试用户](#create-mozy-enterprise-test-user) - 在 Mozy Enterprise 中有一个与 Azure AD 中的 Britta Simon 相对应的关联用户。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Mozy Enterprise 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的 **Mozy Enterprise** 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![Mozy Enterprise 域和 URL 单一登录信息](common/sp-signonurl.png)

    在“登录 URL”文本框中，使用以下模式键入 URL：`https://<tenantname>.Mozyenterprise.com`

    > [!NOTE]
    > 此值不是真实值。 请使用实际登录 URL 更新此值。 请联系 [Mozy Enterprise 客户端支持团队](https://support.mozy.com/)获取此值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

6. 在“设置 Mozy Enterprise”部分中，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-mozy-enterprise-single-sign-on"></a>配置 Mozy Enterprise 单一登录

1. 在另一个 Web 浏览器窗口中，以管理员身份登录到 Mozy Enterprise 公司站点。

2. 在“配置”部分中，单击“身份验证策略”。
   
    ![身份验证策略](./media/mozy-enterprise-tutorial/ic777314.png "身份验证策略")

3. 在“身份验证策略”部分中，执行以下步骤：
   
    ![身份验证策略](./media/mozy-enterprise-tutorial/ic777315.png "身份验证策略")
   
    a. 对于“提供程序”，选择“目录服务”。
   
    b. 选择“使用 LDAP 推送”。
   
    c. 单击“SAML 身份验证”选项卡。
   
    d. 在“登录 URL”文本框中，粘贴从 Azure 门户复制的“身份验证 URL”。
   
    e. 在“SAML 终结点”文本框中，粘贴从 Azure 门户复制的“Azure AD 标识符”值。
   
    f. 在记事本中打开下载的 base-64 编码的证书，将其内容复制到剪贴板，然后将整个证书粘贴到“SAML 证书”文本框。
   
    g. 选择“为使用其网络凭据登录的管理员启用 SSO”。
   
    h. 单击“保存更改”。

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

在本部分中，通过授予 Britta Simon 访问 Mozy Enterprise 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“Mozy Enterprise”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Mozy Enterprise”。

    ![应用程序列表中的 Mozy Enterprise 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-mozy-enterprise-test-user"></a>创建 Mozy Enterprise 测试用户

为了使 Azure AD 用户能够登录到 Mozy Enterprise，必须将其预配到 Mozy Enterprise 中。 对于 Mozy Enterprise，需要手动执行预配。

>[!NOTE]
>可以使用 Mozy Enterprise 提供的任何其他 Mozy Enterprise 用户帐户创建工具或 API 来预配 AAD 用户帐户。

**若要预配用户帐户，请执行以下步骤：**

1. 登录到 **Mozy Enterprise** 租户。

2. 单击“用户”，并单击“添加新用户”。
   
    ![用户](./media/mozy-enterprise-tutorial/ic777317.png "用户")
   
    >[!NOTE]
    >只有当在“身份验证策略”下选择了“Mozy”作为提供程序时才会显示“添加新用户”选项。 如果配置了 SAML 身份验证，则当用户首次通过单一登录进行登录时会自动添加用户。
    
3. 在“新建用户”对话框中，执行以下步骤：
   
    ![添加用户](./media/mozy-enterprise-tutorial/ic777318.png "添加用户")
   
    a. 从“选择组”列表中，选择一个组。
   
    b. 从“用户类型”列表中，选择一个类型。
   
    c. 在“用户名”文本框中，键入 Azure AD 用户的名称。
   
    d. 在“电子邮件”文本框中，键入 Azure AD 用户的电子邮件地址。
   
    e. 选择“发送用户说明电子邮件”。
   
    f. 单击“添加用户”。

     >[!NOTE]
     > 在创建用户后，将向 Azure AD 用户发送一封电子邮件，其中包含在激活帐户前用来确认帐户的链接。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Mozy Enterprise 磁贴时，应当会自动登录到已为其设置了 SSO 的 Mozy Enterprise。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

