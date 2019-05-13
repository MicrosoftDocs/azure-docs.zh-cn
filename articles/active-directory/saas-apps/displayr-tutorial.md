---
title: 教程：Azure Active Directory 与 Displayr 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Displayr 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: b739b4e3-1a37-4e3c-be89-c3945487f4c1
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/03/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9694790ea02bc778bf3b9db212e61fabb90a28a8
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441431"
---
# <a name="tutorial-azure-active-directory-integration-with-displayr"></a>教程：Azure Active Directory 与 Displayr 集成

本教程介绍如何将 Displayr 与 Azure Active Directory (Azure AD) 集成。
将 Displayr 与 Azure AD 集成具有以下优势：

* 可在 Azure AD 中控制谁有权访问 Displayr。
* 可让用户使用其 Azure AD 帐户自动登录到 Displayr（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Displayr 的集成，需要准备好以下各项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 启用了单一登录的 Displayr 订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Displayr 支持“SP”发起的 SSO

## <a name="adding-displayr-from-the-gallery"></a>从库中添加 Displayr

要配置 Displayr 与 Azure AD 的集成，需要从库中将 Displayr 添加到托管 SaaS 应用列表。

**若要从库中添加 Displayr，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入“Displayr”，在结果面板中选择“Displayr”，然后单击“添加”按钮添加该应用程序。

    ![结果列表中的 Displayr](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，我们基于名为“Britta Simon”的测试用户为 Displayr 配置和测试 Azure AD 单一登录。
若要使单一登录有效，需要在 Azure AD 用户与 Displayr 相关用户之间建立关联。

若要为 Displayr 配置并测试 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Displayr 单一登录](#configure-displayr-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Displayr 测试用户](#create-displayr-test-user)** - 在 Displayr 中创建 Britta Simon 的对应用户，并将其关联到用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

要为 Displayr 配置 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Displayr”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“设置 SAML 单一登录”页上，单击“编辑”图标，打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，执行以下步骤：

    ![Displayr 域和 URL 单一登录信息](common/sp-intiated.png)

    在“登录 URL”文本框中，键入 URL：`https://app.displayr.com/Login`

5. 在“SAML 签名证书”部分，单击“下载”以根据要求通过从给定的选项下载**证书(原始)** 并将其保存在计算机上。

    ![证书下载链接](common/certificateraw.png)

6. Displayr 应用程序需要特定格式的 SAML 断言，这要求向 SAML 令牌属性配置添加自定义属性映射。 以下屏幕截图显示了默认属性的列表。 单击“编辑”图标以打开“用户属性”对话框 **** 。

    ![image](common/edit-attribute.png)

7. 除了上述属性，Displayr 应用程序还要求在 SAML 响应中传递回更多的属性。 在“组声明(预览)”对话框中的“用户属性和声明”部分，执行以下步骤 ****  **** ：

    a. 单击“声明中返回的组”旁边的**笔**。

    ![image](./media/displayr-tutorial/config04.png)

    ![图像](./media/displayr-tutorial/config05.png)

    b. 从单选列表中选择“所有组”。

    c. 选择“组 ID”作为“源属性”。

    d. 勾选“自定义组声明的名称”。

    e. 勾选“将组作为角色声明发出”。

    f. 单击“ **保存**”。

8. 在“设置 Displayr”部分，根据要求复制相应 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-displayr-single-sign-on"></a>配置 Displayr 单一登录

1. 在另一个 Web 浏览器窗口中，以管理员身份登录到 Displayr。

2. 单击“设置”，然后导航到“帐户”。

    ![配置](./media/displayr-tutorial/config01.png)

3. 在顶部菜单中切换到“设置”，然后在该页中向下滚动，以便单击“配置单一登录(SAML)”。

    ![配置](./media/displayr-tutorial/config02.png)

4. 在“单一登录(SAML)”页上，执行以下步骤：

    ![配置](./media/displayr-tutorial/config03.png)

    a. 选中“启用单一登录(SAML)”框。

    b. 在“颁发者”文本框中，粘贴从 Azure 门户复制的“Azure AD 标识符”值。

    c. 在“登录 URL”文本框中，粘贴从 Azure 门户复制的“登录 URL”值。

    d. 在“注销 URL”文本框中，粘贴从 Azure 门户复制的“注销 URL”值。

    e. 在记事本中打开“证书(原始)”，复制其内容，然后将其粘贴到“证书”文本框中。

    f. “组映射”为可选。

    g. 单击“ **保存**”。  

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

在本部分中，通过授予 Britta Simon 访问 Displayr 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“Displayr”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Displayr”。

    ![应用程序列表中的 Displayr 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的“Britta Simon”，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-displayr-test-user"></a>创建 Displayr 测试用户

必须将 Azure AD 用户预配到 Displayr 中，然后他们才能登录到 Displayr。 在 Displayr 中，预配是一项手动任务。

**若要预配用户帐户，请执行以下步骤：**

1. 以管理员身份登录到 Displayr。

2. 单击“设置”，然后导航到“帐户”。

    ![Displayr 配置](./media/displayr-tutorial/config01.png)

3. 在顶部菜单中切换到“设置”，在该页中向下滚动到“用户”部分，然后单击“新建用户”。

    ![Displayr 配置](./media/displayr-tutorial/config07.png)

4. 在“新建用户”页面上，执行以下步骤：

    ![Displayr 配置](./media/displayr-tutorial/config06.png)

    a. 在“名称”文本框中，输入用户姓名，例如 Brittasimon。

    b. 在“电子邮件”文本框中，输入用户的电子邮件，如 `Brittasimon@contoso.com`。

    c. 选择相应的“组成员身份”。

    d. 单击“ **保存**”。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Displayr 磁贴时，应会自动登录到为其设置了 SSO 的 Displayr。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

