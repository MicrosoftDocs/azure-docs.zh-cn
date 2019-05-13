---
title: 教程：Azure Active Directory 与 OpenAthens 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 OpenAthens 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: dd4adfc7-e238-41d5-8b25-1811f08078b6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 1/4/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2cbb1bc64d923add0b1fd38749299714a9093633
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408022"
---
# <a name="tutorial-azure-active-directory-integration-with-openathens"></a>教程：Azure Active Directory 与 OpenAthens 的集成

在本教程中，了解如何将 OpenAthens 与 Azure Active Directory (Azure AD) 集成。
将 OpenAthens 与 Azure AD 集成提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 OpenAthens。
* 可以让用户使用其 Azure AD 帐户自动登录到 OpenAthens（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 OpenAthens 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 启用了单一登录的 OpenAthens 订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* OpenAthens 支持 **IDP** 发起的 SSO

* OpenAthens 支持**实时**用户预配

## <a name="adding-openathens-from-the-gallery"></a>从库中添加 OpenAthens

若要配置 OpenAthens 与 Azure AD 的集成，需要从库中将 OpenAthens 添加到托管 SaaS 应用列表。

**若要从库中添加 OpenAthens，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“OpenAthens”，在结果面板中选择“OpenAthens”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 OpenAthens](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，将基于名为 **Britta Simon** 的测试用户配置和测试 OpenAthens 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 OpenAthens 中相关用户之间建立链接关系。

若要配置并测试 OpenAthens 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 OpenAthens 单一登录](#configure-openathens-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 OpenAthens 测试用户](#create-openathens-test-user)** - 在 OpenAthens 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 OpenAthens 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中，在 **OpenAthens** 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

5. 在“基本 SAML 配置”部分中，上传**服务提供程序元数据文件**，本教程下文中介绍了相关的步骤。

    a. 单击“上传元数据文件”。

    ![Openathens 上传元数据](common/upload-metadata.png)

    b. 单击“文件夹徽标”来选择元数据文件并单击“上传”。

    ![Openathens 浏览上传元数据](common/browse-upload-metadata.png)

    c. 成功上传元数据文件后，**标识符**值会自动填充在“基本 SAML 配置”部分的文本框中：

    ![OpenAthens 域和 URL 单一登录信息](common/idp-identifier.png)

6. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

### <a name="configure-openathens-single-sign-on"></a>配置 OpenAthens 单一登录

1. 在另一个 Web 浏览器窗口中，以管理员身份登录 OpenAthens 公司站点。

2. 在“管理”选项卡下的列表中选择“连接”。 

    ![配置单一登录](./media/openathens-tutorial/tutorial_openathens_application1.png)

3. 选择“SAML 1.1/2.0”，再选择“配置”按钮。

    ![配置单一登录](./media/openathens-tutorial/tutorial_openathens_application2.png)
    
4. 若要添加配置，请选择“浏览”按钮上传已从 Azure 门户下载的元数据 .xml 文件，再选择“添加”。

    ![配置单一登录](./media/openathens-tutorial/tutorial_openathens_application3.png)

5. 在“详细信息”选项卡下执行以下步骤。

    ![配置单一登录](./media/openathens-tutorial/tutorial_openathens_application4.png)

    a. 在“显示名称映射”中，选择“使用属性”。

    b. 在“显示名称属性”文本框中，输入值 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`。
    
    c. 在“唯一用户映射”中，选择“使用属性”。

    d. 在“唯一用户属性”文本框中，输入值 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`。

    e. 在“状态”中，选中所有三个复选框。

    f. 在“创建本地帐户”中，选择“自动”。

    g. 选择“保存更改”。

    h. 在“信赖方”选项卡上，复制**元数据 URL** 并在浏览器中将其打开以下载 **SP 元数据 XML** 文件。 在 Azure AD 中的“基本 SAML 配置”部分中上传此 SP 元数据文件。

    ![配置单一登录](./media/openathens-tutorial/tutorial_openathens_application5.png)

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

在本部分中，通过授予 Britta Simon 访问 OpenAthens 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“OpenAthens”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，键入并选择“OpenAthens”。

    ![应用程序列表中的 OpenAthens 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-openathens-test-user"></a>创建 OpenAthens 测试用户

在本部分中，我们将在 OpenAthens 中创建一个名为 Britta Simon 的用户。 OpenAthens 支持默认情况下启用的**实时用户预配**。 此部分不存在任何操作项。 如果 OpenAthens 中尚不存在用户，则会在身份验证后创建一个新用户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 OpenAthens 磁贴时，应当会自动登录到你为其设置了 SSO 的 OpenAthens。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

