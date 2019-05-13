---
title: 教程：Azure Active Directory 与 Templafy 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Templafy 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 60c6153e-2d9d-4a56-809d-4a7a9d677c43
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2ea2ead29ba1945213871ab76a65e33a4d7b27cc
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441504"
---
# <a name="tutorial-azure-active-directory-integration-with-templafy"></a>教程：Azure Active Directory 与 Templafy 集成

在本教程中，了解如何将 Templafy 与 Azure Active Directory (Azure AD) 集成。
将 Templafy 与 Azure AD 集成具有以下优势：

* 可在 Azure AD 中控制谁有权访问 Templafy。
* 可让用户使用其 Azure AD 帐户自动登录到 Templafy（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Templafy 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 Templafy 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Templafy 支持 SP 发起的 SSO
* Templafy 支持**恰时**用户预配

## <a name="adding-templafy-from-the-gallery"></a>从库中添加 Templafy

要配置 Templafy 与 Azure AD 的集成，需要从库中将 Templafy 添加到托管 SaaS 应用列表。

**若要从库中添加 Templafy，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“Templafy”，在结果面板中选择“Templafy”，然后单击“添加”按钮添加该应用程序。

    ![结果列表中的 Templafy](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于一个名为“Britta Simon”的测试用户使用 Templafy 配置和测试 Azure AD 单一登录。
若要使单一登录有效，需要在 Azure AD 用户与 Templafy 相关用户之间建立链接关系。

若要配置和测试 Templafy 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Templafy 单一登录](#configure-templafy-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. [创建 Templafy 测试用户](#create-templafy-test-user) - 在 Templafy 中创建 Britta Simon 的对应用户，并将其链接到用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Templafy 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Templafy”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![Templafy 域和 URL 单一登录信息](common/sp-signonurl.png)

    在“登录 URL”文本框中，使用以下模式键入 URL：`https://<CLIENTSUBDOMAIN>.templafy.com`

    > [!NOTE]
    > 此值不是真实值。 请使用实际登录 URL 更新此值。 若要获取该值，请与 [Templafy 客户端支持团队](mailto:support@templafy.com)联系。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. Templafy 应用程序需要特定格式的 SAML 断言。 请为此应用程序配置以下声明。 可以在应用程序集成页的“用户属性”部分管理这些属性的值。 在“使用 SAML 设置单一登录”页上，单击“编辑”按钮以打开“用户属性”对话框。

    ![image](common/edit-attribute.png)

6. 在“用户属性”对话框的“用户声明”部分中，通过使用“编辑图标”编辑声明或使用“添加新声明”添加声明，按上图所示配置 SAML 令牌属性，并执行以下步骤：

    | 名称 | 源属性| 命名空间  |
    | ---------------| --------------- | --------- |
    | givenname | user.givenname | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | surname | user.surname | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | emailaddress | user.mail | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`
    | streetaddress | user.streetaddress | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | city | user.city | `https://schemas.templafy.com/2016/06/identity/claims`|
    | postalcode | user.postalcode | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | stateorprovince | user.state | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | country | user.country | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | jobtitle | user.jobtitle | `https://schemas.templafy.com/2016/06/identity/claims`|
    | department | user.department | `https://schemas.templafy.com/2016/06/identity/claims`|
    | phonenumber | user.telephonenumber | `https://schemas.templafy.com/2016/06/identity/claims` |
    | facsimilenumber | user.facsimiletelephonenumber | `https://schemas.templafy.com/2016/06/identity/claims`|
    | upn | user.userprincipalname | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | NameIdentifier | user.mail | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims`|
    | | |

    a. 单击“添加新声明”以打开“管理用户声明”对话框。

    ![图像](common/new-save-attribute.png)

    ![图像](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。

    c. 在“命名空间”文本框中，输入为该行显示的命名空间值。

    d. 选择“源”作为“属性”。

    e. 在“源属性”列表中，键入为该行显示的属性值。

    f. 单击“确定”

    g. 单击“ **保存**”。

7. 在“设置 SAML 单一登录”页的“SAML 签名证书”部分中，单击“复制”按钮，以复制“应用联合元数据 URL”，并将它保存在计算机上。

    ![证书下载链接](common/copy-metadataurl.png)

### <a name="configure-templafy-single-sign-on"></a>配置 Templafy 单一登录

若要在 **Templafy** 端配置单一登录，需要将“应用联合元数据 URL”发送给 [Templafy 支持团队](mailto:support@templafy.com)。 他们会对此进行设置，使两端的 SAML SSO 连接均正确设置。

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中，按照以下步骤操作。

    ![“用户”对话框](common/user-properties.png)

    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，并单击“添加引用”。 在“名称”字段中，输入 BrittaSimon。
  
    b. 在“用户名”字段中键入 `brittasimon@yourcompanydomain.extension`。 例如： BrittaSimon@contoso.com

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予 Britta Simon 访问 Templafy 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“Templafy”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Templafy”。

    ![应用程序列表中的 Templafy 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的“Britta Simon”，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-templafy-test-user"></a>创建 Templafy 测试用户

在本部分中，我们会在 Templafy 中创建一个名为 Britta Simon 的用户。 Templafy 支持默认启用的恰时用户预配。 此部分不存在任何操作项。 如果 Templafy 中不存在用户，则会在身份验证后创建一个新用户。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Templafy 磁贴时，应当会自动登录到为其设置了 SSO 的 Templafy。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
