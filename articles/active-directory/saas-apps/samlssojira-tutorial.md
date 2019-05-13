---
title: 教程：Azure Active Directory 与 SAML SSO for Jira by Resolution GmbH 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 SAML SSO for Jira by resolution GmbH 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 20e18819-e330-4e40-bd8d-2ff3b98e035f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/03/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 943131bc746b5d2a1fd95a26a6a6c9f3bb6b9e57
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509956"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-jira-by-resolution-gmbh"></a>教程：Azure Active Directory 与 SAML SSO for Jira by resolution GmbH 集成

本教程介绍如何设置 SAML SSO for Jira by resolution GmbH 与 Azure Active Directory (Azure AD) 的集成。
将 SAML SSO for Jira by resolution GmbH 与 Azure AD 集成具有以下优势：

* 可以在 Azure AD 中控制谁能够使用 resolution GmbH 提供的 SAML SSO 插件登录到 Jira。
* 可让用户使用 SAML SSO for Jira by resolution GmbH 通过其 Azure AD 帐户自动登录到 Jira（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 SAML SSO for Jira by resolution GmbH 的集成，需要准备好以下各项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 启用了 SAML SSO for Jira by resolution GmbH 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* SAML SSO for Jira by resolution GmbH 支持 **SP** 和 **IDP** 发起的 SSO

## <a name="adding-an-enterprise-application-for-single-sign-on"></a>添加一个企业应用程序用于单一登录

若要在 Azure AD 中设置单一登录，需要添加一个新的企业应用程序。 库中为此预先配置了一个应用程序：**SAML SSO for Jira by resolution GmbH**。

**若要从库中添加 SAML SSO for Jira by resolution GmbH，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 导航到“企业应用程序”，然后单击“所有应用程序”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入 **SAML SSO for Jira by resolution GmbH**，在结果面板中选择“SAML SSO for Jira by resolution GmbH”，然后单击“添加”按钮添加该应用程序。 还可以更改企业应用的名称。

     ![结果列表中的 SAML SSO for Jira by resolution GmbH](common/search-new-app.png)

## <a name="configure-and-test-single-sign-on-with-the-saml-sso-plugin-and-azure-ad"></a>配置并测试使用 SAML SSO 插件和 Azure AD 进行的单一登录

在本部分，我们将测试并配置 Azure AD 用户在 Jira 中的单一登录。 此操作是使用名为 **Britta Simon** 的测试用户完成的。
若要运行单一登录，需要在 Azure AD 用户与 SAML SSO for Jira by resolution GmbH 相关用户之间建立链接关系。

若要配置并测试单一登录，需要完成以下步骤：

1. **[配置用于单一登录的 Azure AD 企业应用程序](#configure-the-azure-ad-enterprise-application-for-single-sign-on)** - 配置用于单一登录的 Azure AD 企业应用程序
2. **[配置 Jira 实例的 SAML SSO 插件](#configure-the-saml-sso-plugin-of-your-jira-instance)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 在 Azure AD 中创建测试用户。
1. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使测试用户能够在 Azure 端使用单一登录。
1. **[在 Jira 中创建测试用户](#create-the-test-user-also-in-jira)** - 在 Jira 中为 Azure AD 测试用户创建一个对应的测试用户。
1. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-the-azure-ad-enterprise-application-for-single-sign-on"></a>配置用于单一登录的 Azure AD 企业应用程序

在本部分，我们将在 Azure 门户中设置单一登录。

若要配置 SAML SSO for Jira by resolution GmbH 的单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中，在刚刚创建的 **SAML SSO for Jira by resolution GmbH** 企业应用程序中，选择左面板中的“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 对于“选择单一登录方法”，请选择“SAML”模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 然后，单击“编辑”图标打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 若要在 **IDP** 发起的模式下配置应用程序，请在“基本 SAML 配置”部分执行以下步骤：

    ![SAML SSO for Jira by resolution GmbH 域和 URL 单一登录信息](common/idp-intiated.png)

    a. 在“标识符”文本框中，使用以下模式键入 URL：`https://<server-base-url>/plugins/servlet/samlsso`

    b. 在“回复 URL”文本框中，使用以下模式键入 URL：`https://<server-base-url>/plugins/servlet/samlsso`

    c. 若要在 **SP** 发起的模式下配置应用程序，请单击“设置其他 URL”，并执行以下步骤：

    ![SAML SSO for Jira by resolution GmbH 域和 URL 单一登录信息](common/metadata-upload-additional-signon.png)

    在“登录 URL”文本框中，使用以下模式键入 URL：`https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > 对于“标识符”、“回复 URL”和“登录 URL”，请将 **\<server-base-url>** 替换为 Jira 实例的基 URL。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。 如果遇到问题，请联系 [SAML SSO for Jira by resolution GmbH 客户支持团队](https://www.resolution.de/go/support)。

4. 在“设置 SAML 单一登录”页上的“SAML 签名证书”部分，下载“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

### <a name="configure-the-saml-sso-plugin-of-your-jira-instance"></a>配置 Jira 实例的 SAML SSO 插件 

1. 在另一个 Web 浏览器窗口中，以管理员身份登录到 Jira 实例。

2. 将鼠标悬停在右侧的齿轮图标上，然后单击“管理应用”。
    
    ![配置单一登录](./media/samlssojira-tutorial/addon1.png)

3. 如果已重定向到“管理员访问”页，请输入**密码**，然后单击“确认”按钮。

    ![配置单一登录](./media/samlssojira-tutorial/addon2.png)

4. Jira 通常会将你重定向到 Atlassian 市场。 如果没有，请单击左面板中的“查找新应用”。 搜索 **SAML Single Sign On (SSO) for JIRA**，然后单击“安装”按钮安装 SAML 插件。

    ![配置单一登录](./media/samlssojira-tutorial/store.png)

5. 插件安装随即开始。 完成后，单击“关闭”按钮。

    ![配置单一登录](./media/samlssojira-tutorial/store-2.png)

    ![配置单一登录](./media/samlssojira-tutorial/store-3.png)

6. 然后单击“管理”。

    ![配置单一登录](./media/samlssojira-tutorial/store-4.png)
    
8. 然后，单击“配置”以配置刚刚安装的插件。

    ![配置单一登录](./media/samlssojira-tutorial/store-5.png)

9. 在“SAML 单一登录插件配置”向导中，单击“添加新 IdP”，将 Azure AD 配置为新的标识提供者。

    ![配置单一登录](./media/samlssojira-tutorial/addon4.png) 

10. 在“选择 SAML 标识提供者”页上执行以下步骤：

    ![配置单一登录](./media/samlssojira-tutorial/addon5a.png)
 
    a. 设置“Azure AD”作为 IdP 类型。
    
    b. 添加标识提供者的**名称**（例如 Azure AD）。
    
    c. （可选）添加标识提供者的**说明**（例如 Azure AD）。
    
    d. 单击“下一步”。
    
11. 在“标识提供者配置”页上，单击“下一步”。
 
    ![配置单一登录](./media/samlssojira-tutorial/addon5b.png)

12. 在“导入 SAML IdP 元数据”页上，执行以下步骤：

    ![配置单一登录](./media/samlssojira-tutorial/addon5c.png)

    a. 单击“选择元数据 XML 文件”按钮，然后选择前面下载的“联合身份验证元数据 XML”文件。

    b. 单击“导入”按钮。
     
    c. 等待片刻，让导入成功完成。  
     
    d. 单击“下一步”按钮。
    
13. 在“用户 ID 属性和转换”页上，单击“下一步”按钮。

    ![配置单一登录](./media/samlssojira-tutorial/addon5d.png)
    
14. 在“用户创建和更新”页上，单击“保存”以保存设置，然后单击“下一步”。
    
    ![配置单一登录](./media/samlssojira-tutorial/addon6a.png)
    
15. 在“测试设置”页上，单击“跳过测试并手动配置”以暂时跳过用户测试。 此测试将在下一部分执行，需要在 Azure 门户中进行某些设置。
    
    ![配置单一登录](./media/samlssojira-tutorial/addon6b.png)
    
16. 单击“确定”跳过警告。
    
    ![配置单一登录](./media/samlssojira-tutorial/addon6c.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。 我们将使用该用户测试单一登录。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中执行以下步骤：

    ![“用户”对话框](common/user-properties.png)

    a. 在“名称”字段中输入 **Britta Simon**。
  
    b. 在“用户名”字段中输入 <b>BrittaSimon@contoso.com</b>。

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分，我们将 Britta Simon 添加到企业应用程序，使其能够使用单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。 

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，搜索在本教程开头创建的企业应用程序。 如果你一直在遵循本教程中的步骤，该应用程序名为 **SAML SSO for Jira by resolution GmbH**。 如果你指定了其他名称，请搜索该名称。

    ![应用程序列表中的 SAML SSO for Jira by resolution GmbH 链接](common/all-applications.png)

3. 在左面板中单击“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 选择“添加用户”，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，从“用户”列表中选择“Britta Simon”，然后单击屏幕底部的“选择”按钮。

6. 如果在 SAML 断言中需要任何角色值，请在“选择角色”对话框中，从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-the-test-user-also-in-jira"></a>同样在 Jira 中创建测试用户

要使 Azure AD 用户能够登录到 SAML SSO for Jira by resolution GmbH，必须将其预配到 SAML SSO for Jira by resolution GmbH 中。 对于本教程，必须手动完成预配。 但是，还有其他预配模型可供解决方案的 SAML SSO 插件使用，例如**实时**预配。 请参阅 [SAML SSO by resolution GmbH](https://wiki.resolution.de/doc/saml-sso/latest/all) 提供的文档。 如果遇到相关问题，请联系[解决方案支持](https://www.resolution.de/go/support)。

**若要手动预配用户帐户，请执行以下步骤：**

1. 以管理员身份登录到 Jira 实例。

2. 将鼠标悬停在齿轮图标上，然后选择“用户管理”。

   ![添加员工](./media/samlssojira-tutorial/user1.png)

3. 如果已重定向到“管理员访问”页，请输入**密码**，然后单击“确认”按钮。

    ![添加员工](./media/samlssojira-tutorial/user2.png) 

4. 在“用户管理”选项卡部分，单击“创建用户”。

    ![添加员工](./media/samlssojira-tutorial/user3-new.png) 

5. 在“新建用户”对话框页上执行以下步骤。 必须创建与 Azure AD 中完全相同的用户：

    ![添加员工](./media/samlssojira-tutorial/user4-new.png) 

    a. 在“电子邮件地址”文本框中，键入用户的电子邮件地址：<b>BrittaSimon@contoso.com</b>。

    b. 在“全名”文本框中，键入用户的全名：**Britta Simon**。

    c. 在“用户名”文本框中，键入用户的电子邮件地址：<b>BrittaSimon@contoso.com</b>。 

    d. 在“密码”文本框中，输入用户的密码。

    e. 单击“创建用户”以完成用户创建。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的“SAML SSO for Jira by resolution GmbH”磁贴时，应当会自动登录到已为其设置了 SSO 的 SAML SSO for Jira by resolution GmbH。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

还可以导航到 [https://\<server-base-url>/plugins/servlet/samlsso](https://\<server-base-url>/plugins/servlet/samlsso) 来测试单一登录。 请将 **\<server-base-url>** 替换为 Jira 实例的基 URL。


## <a name="enable-single-sign-on-redirection-for-jira"></a>为 Jira 启用单一登录重定向

如前面的部分中所述，目前可以通过两种方式来触发单一登录： 使用 **Azure 门户**或使用 **Jira 实例的特殊链接**。 使用 resolution GmbH 提供的 SAML SSO 插件，还可以简单地通过**访问指向 Jira 实例的任何 URL**，来触发单一登录。

从本质上讲，访问 Jira 的所有用户在激活该插件中的某个选项后，会重定向到单一登录页。

若要激活 SSO 重定向，请**在 Jira 实例**中执行以下操作：

1. 访问 Jira 中 SAML SSO 插件的配置页。
1. 单击左面板中的“重定向”。
![](./media/samlssojira-tutorial/ssore1.png)

1. 勾选“启用 SSO 重定向”。
![](./media/samlssojira-tutorial/ssore2.png) 

1. 按右上角的“保存设置”按钮。

激活该选项后，如果已勾选“启用 nosso”选项，则仍可以通过导航到 [https://\<server-base-url>/login.jsp?nosso](https://\<server-base-url>/login.jsp?nosso) 来查看用户名/密码提示。 如前所述，请将 **\<server-base-url>** 替换为基 URL。


## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

