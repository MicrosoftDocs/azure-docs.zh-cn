---
title: 教程：Azure Active Directory 与 XaitPorter 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 XaitPorter 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: d33c7cb7-0550-425b-882a-619a713a71b7
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/03/2019
ms.author: jeedes
ms.openlocfilehash: ada93903113cedd68e23ec66a36dceef99ec1233
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192094"
---
# <a name="tutorial-azure-active-directory-integration-with-xaitporter"></a>教程：Azure Active Directory 与 XaitPorter 集成

本教程介绍如何将 XaitPorter 与 Azure Active Directory (Azure AD) 集成。
将 XaitPorter 与 Azure AD 集成提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 XaitPorter
* 可让用户使用其 Azure AD 帐户自动登录到 XaitPorter（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 XaitPorter 的集成，需要准备好以下各项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 已启用 XaitPorter 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* XaitPorter 支持“SP”发起的 SSO

## <a name="adding-xaitporter-from-the-gallery"></a>从库中添加 XaitPorter

要配置 XaitPorter 与 Azure AD 的集成，需要从库中将 XaitPorter 添加到托管 SaaS 应用列表。

**若要从库中添加 XaitPorter，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“XaitPorter”，在结果面板中选择“XaitPorter”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 XaitPorter](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，将基于名为“Britta Simon”的测试用户配置和测试 XaitPorter 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 XaitPorter 相关用户之间建立链接关系。

若要配置和测试 XaitPorter 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 XaitPorter 单一登录](#configure-xaitporter-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 XaitPorter 测试用户](#create-xaitporter-test-user)** - 在 XaitPorter 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 XaitPorter 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的 **XaitPorter** 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![XaitPorter 域和 URL 单一登录信息](common/sp-identifier.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<subdomain>.xaitporter.com/saml/login`

    b. 在“标识符(实体 ID)”文本框中，使用以下模式键入 URL：`https://<subdomain>.xaitporter.com`

    > [!NOTE]
    > 这些不是实际值。 使用实际登录 URL 和标识符更新这些值。 请联系 [XaitPorter 客户端支持团队](https://www.xait.com/support/)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. 在“设置 SAML 单一登录”页的“SAML 签名证书”部分中，单击“复制”按钮，以复制“应用联合元数据 URL”，并将它保存在计算机上。

    ![证书下载链接](common/copy-metadataurl.png)

6. 向 [SmartRecruiters 支持团队](https://www.smartrecruiters.com/about-us/contact-us/)提供 **IP 地址**或**应用联合元数据 URL**，使 XaitPorter 能够确保可从其配置允许列表的 XaitPorter 实例访问该 IP 地址。 

### <a name="configure-xaitporter-single-sign-on"></a>配置 XaitPorter 单一登录

1. 若要在 XaitPorter 中自动执行配置，需要通过单击“安装扩展”，安装“我的应用安全登录浏览器扩展”。

    ![我的应用扩展](common/install-myappssecure-extension.png)

2. 将扩展添加到浏览器后，单击“设置 XaitPorter”会将你定向到 XaitPorter 应用程序。 在此处，请提供管理员凭据以登录到 XaitPorter。 浏览器扩展会自动配置应用程序，并自动执行第 3 步到第 6 步。

    ![设置配置](common/setup-sso.png)

3. 若要手动设置 XaitPorter，请打开新的 Web 浏览器窗口，以管理员身份登录 XaitPorter 公司网站，并执行以下步骤：

4. 单击“管理”。

    ![配置单一登录](./media/xaitporter-tutorial/user1.png)

5. 从“系统设置”下拉列表中选择“管理单一登录”。

    ![配置单一登录](./media/xaitporter-tutorial/user2.png)

6. 在“管理单一登录”部分中执行以下步骤：

    ![配置单一登录](./media/xaitporter-tutorial/user3.png)

    a. 选择“启用单一登录身份验证”。

    b. 在“标识提供者设置”文本框中，粘贴从 Azure 门户复制的**应用联合元数据 URL**并单击“提取”。

    c. 选择“启用用户自动创建”。

    d. 单击“确定”。

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户 

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中，按照以下步骤操作。

    ![“用户”对话框](common/user-properties.png)

    a. 在“名称”字段中，输入 BrittaSimon。
  
    b. 在“用户名”字段中键入 brittasimon@yourcompanydomain.extension。 例如： BrittaSimon@contoso.com

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予 Britta Simon 访问 XaitPorter 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“XaitPorter”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“XaitPorter”。

    ![应用程序列表中的 XaitPorter 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-xaitporter-test-user"></a>创建 XaitPorter 测试用户

本部分的内容是在 XaitPorter 中创建名为“Britta Simon”的用户。 请与 [XaitPorter 客户端支持团队](https://www.xait.com/support/)协作，在 XaitPorter 平台中添加用户。 使用单一登录前，必须先创建并激活用户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 XaitPorter 磁贴时，应会自动登录到为其设置了 SSO 的 XaitPorter。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

