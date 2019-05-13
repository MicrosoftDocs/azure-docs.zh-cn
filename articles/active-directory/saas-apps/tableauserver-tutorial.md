---
title: 教程：Azure Active Directory 与 Tableau Server 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Tableau Server 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: c1917375-08aa-445c-a444-e22e23fa19e0
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/22/2019
ms.author: jeedes
ms.openlocfilehash: 480e799e0fe307b39ce8a2f0b026a04f507a6beb
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408014"
---
# <a name="tutorial-azure-active-directory-integration-with-tableau-server"></a>教程：Azure Active Directory 与 Tableau Server 集成

本教程介绍如何将 Tableau Server 与 Azure Active Directory (Azure AD) 集成。
将 Tableau Server 与 Azure AD 集成提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 Tableau Server。
* 可让用户使用其 Azure AD 帐户自动登录到 Tableau Server（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Tableau Server 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 Tableau Server 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Tableau Server 支持 **SP** 发起的 SSO

## <a name="adding-tableau-server-from-the-gallery"></a>从库中添加 Tableau Server

要配置 Tableau Server 与 Azure AD 的集成，需要从库中将 Tableau Server 添加到托管 SaaS 应用列表。

**若要从库中添加 Tableau Server，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“Tableau Server”，在结果面板中选择“Tableau Server”，然后单击“添加”按钮添加该应用程序。

    ![结果列表中的 Tableau Server](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 Tableau Server 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 Tableau Server 相关用户之间建立链接关系。

若要配置和测试 Tableau Server 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Tableau Server 单一登录](#configure-tableau-server-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Tableau Server 测试用户](#create-tableau-server-test-user)** - 在 Tableau Server 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Tableau Server 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Tableau Server”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![Tableau Server 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://azure.<domain name>.link`。

    b. 在“标识符”框中，使用以下模式键入 URL：`https://azure.<domain name>.link`

    c. 在“回复 URL”文本框中，使用以下模式键入 URL：`https://azure.<domain name>.link/wg/saml/SSO/index.html`

    > [!NOTE]
    > 上面的值不是实际值。 请使用来自 Tableau Server 配置页面的实际 URL 和标识符更新这些值，本教程下文中介绍了此页面。

5. Tableau Server 应用程序需要一个应当如下定义的自定义声明 **username**。 这用作用户标识符，而不是唯一的用户标识符声明。 可以在应用程序集成页的“用户属性和声明”部分管理这些属性的值。 单击“编辑”按钮以打开“用户属性和声明”对话框。

    ![image](common/edit-attribute.png)

6. 在“用户属性和声明”对话框的“用户声明”部分中，按上图所示配置 SAML 令牌属性，并执行以下步骤：

    | 名称 | 源属性 | 命名空间 |
    | ---------------| --------------- | ----------- |
    | username | user.userprincipalname | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | | |

    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，并单击“添加引用”。 单击“添加新声明”以打开“管理用户声明”对话框。

    ![图像](common/new-save-attribute.png)

    ![图像](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。

    c. 输入“命名空间”值。

    d. 选择“源”作为“属性”。

    e. 在“源属性”列表中，键入为该行显示的属性值。

    f. 单击“确定”

    g. 单击“ **保存**”。

7. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

8. 在“设置 Tableau Server”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-tableau-server-single-sign-on"></a>配置 Tableau Server 单一登录

1. 若要为应用程序配置 SSO，需要以管理员身份登录到 Tableau Server 租户。

2. 在“配置”选项卡上，选择“用户标识和访问”，然后选择“身份验证方法”选项卡。

    ![配置单一登录](./media/tableauserver-tutorial/tutorial-tableauserver-auth.png)

3. 在“配置”页上，执行以下步骤：

    ![配置单一登录](./media/tableauserver-tutorial/tutorial-tableauserver-config.png)

    a. 对于**身份验证方法**，请选择“SAML”。

    b. 选中“为服务器启用 SAML 身份验证”复选框。

    c. Tableau Server 返回 URL（Tableau Server 用户将要访问的 URL），例如 <http://tableau_server>。 但不建议使用 `http://localhost`。 不支持使用带尾部反斜杠的 URL（例如 `http://tableau_server/`）。 复制“Tableau Server 返回 URL”，并将其粘贴到 Azure 门户上“基本 SAML 配置”部分的“登录 URL”文本框中。

    d. SAML 实体 ID - 此实体 ID 唯一标识安装到 IdP 的 Tableau Server。 可以在此处再次输入 Tableau Server URL（如果需要），但它不必须是 Tableau Server URL。 复制“SAML 实体 ID”，并将其粘贴到 Azure 门户上“基本 SAML 配置”部分的“标识符”文本框中。

    e. 单击“下载 XML 元数据文件”，并在文本编辑器应用程序中打开该文件。 找到包含 Http Post 和索引 0 的断言使用者服务 URL 并复制该 URL。 现在，将其粘贴到 Azure 门户上“基本 SAML 配置”部分的“回复 URL”文本框中。

    f. 找到从 Azure 门户下载的联合元数据文件，然后在“SAML Idp 元数据文件”中上传它。

    g. 输入 IdP 用来保存用户名、显示名称和电子邮件地址的属性的名称。

    h. 单击“保存”

    > [!NOTE]
    > 客户必须上传 Tableau Server SAML SSO 配置中的任何证书，SSO 流程中会将其忽略。 如果需要帮助在 Tableau Server 上配置 SAML，请参阅此文：[配置 SAML](https://onlinehelp.tableau.com/v2018.2/server/en-us/saml_config_steps_tsm_ui.htm)。

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中，按照以下步骤操作。

    ![“用户”对话框](common/user-properties.png)

    a. 在“名称”字段中，输入 BrittaSimon。
  
    b. 在“用户名”字段中，键入 `brittasimon@yourcompanydomain.extension`  
    例如： BrittaSimon@contoso.com

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予 Britta Simon 访问 Tableau Server 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“Tableau Server”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Tableau Server”。

    ![“应用程序”列表中的“Tableau Server”链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-tableau-server-test-user"></a>创建 Tableau Server 测试用户

本部分的目的是在 Tableau Server 中创建名为“Britta Simon”的用户。 需要在 Tableau Server 中预配所有用户。

用户的用户名应与在 **username** 的 Azure AD 自定义属性中配置的值匹配。 使用正确的映射，此集成应可实现配置 Azure AD 单一登录。

> [!NOTE]
> 如果需要手动创建用户，需要联系组织中的 Tableau Server 管理员。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“Tableau Server”磁贴时，应会自动登录到设置了 SSO 的 Tableau Server。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

