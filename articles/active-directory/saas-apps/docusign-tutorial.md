---
title: 教程：Azure Active Directory 与 DocuSign 的集成 | Microsoft 文档
description: 了解如何在 Azure Active Directory 和 DocuSign 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: a691288b-84c1-40fb-84bd-5b06878865f0
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/01/2019
ms.author: jeedes
ms.openlocfilehash: 35fc4e855ef53bd7e667fc8de7146916b4d05acb
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407080"
---
# <a name="tutorial-azure-active-directory-integration-with-docusign"></a>教程：Azure Active Directory 与 DocuSign 的集成

本教程介绍如何将 DocuSign 与 Azure Active Directory (Azure AD) 集成。
将 DocuSign 与 Azure AD 集成具有优势：

* 可在 Azure AD 中控制谁有权访问 DocuSign。
* 可让用户使用其 Azure AD 帐户自动登录到 DocuSign（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 DocuSign 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 DocuSign 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* DocuSign 支持 **SP** 发起的 SSO

* DocuSign 支持**实时**用户预配

## <a name="adding-docusign-from-the-gallery"></a>从库中添加 DocuSign

若要配置 DocuSign 与 Azure AD 的集成，需要从库中将 DocuSign 添加到托管 SaaS 应用列表。

**若要从库中添加 DocuSign，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“DocuSign”，在结果面板中选择“DocuSign”，然后单击“添加”按钮添加应用程序。

     ![结果列表中的 DocuSign](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 DocuSign 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 DocuSign 相关用户之间建立链接关系。

若要配置和测试 DocuSign 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 DocuSign 单一登录](#configure-docusign-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 DocuSign 测试用户](#create-docusign-test-user)** - 在 DocuSign 中创建 Britta Simon 的对应用户，并将其关联到其在 Azure AD 中的表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 DocuSign 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“DocuSign”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![DocuSign 域和 URL 单一登录信息](common/sp-identifier.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<subdomain>.docusign.com/organizations/<OrganizationID>/saml2/login/sp/<IDPID>`

    b. 在“标识符(实体 ID)”文本框中，使用以下模式键入 URL：`https://<subdomain>.docusign.com/organizations/<OrganizationID>/saml2`

    > [!NOTE]
    > 这些不是实际值。 本教程的“查看 SAML 2.0 终结点”部分稍后将介绍如何使用实际的登录 URL 和标识符来更新该值。

5. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

6. 在“设置 DocuSign”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-docusign-single-sign-on"></a>配置 DocuSign 单一登录

1. 在另一个 Web 浏览器窗口中，以管理员身份登录到 **DocuSign 管理门户**。

2. 在页面右上角单击个人资料徽标，然后单击“转到管理员”。
  
    ![配置单一登录][51]

3. 在域解决方案页单击“域”

    ![配置单一登录][50]

4. 在“域”部分下，单击“声明域”。

    ![配置单一登录][52]

5. 在“声明域”对话框中，在“域名”文本框中键入公司域，并单击“声明”。 确保对域进行验证并且其状态为活动。

    ![配置单一登录][53]

6. 在域解决方案页单击“标识提供者”。
  
    ![配置单一登录][54]

7. 在“标识提供者”部分下，单击“添加标识提供者”。 

    ![配置单一登录][55]

8. 在“标识提供者设置”页上，执行以下步骤：

    ![配置单一登录][56]

    a. 在“名称”文本框中，为配置键入一个唯一的名称。 请不要使用空格。

    b. 在“标识提供者颁发者”文本框中，粘贴从 Azure 门户复制的“Azure AD 标识符”值。

    c. 在“标识提供者登录 URL”文本框中，粘贴从 Azure 门户复制的“登录 URL”值。

    d. 在“标识提供者注销 URL”文本框中，粘贴从 Azure 门户复制的“注销 URL”值。

    e. 选择“对身份验证请求签名”。

    f. 对于“身份验证请求发送方式”，选择“POST”。

    g. 对于“注销请求发送方式”，选择“GET”。

    h. 在“自定义属性映射”部分中，单击“添加新映射”。

    ![配置单一登录][62]

    i. 选择要通过 Azure AD 声明映射的字段。 在此示例中，**emailaddress** 声明使用 **https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress** 的值映射。 这是 Azure AD 中用于电子邮件声明的默认声明名称，然后单击“保存”。

    ![配置单一登录][57]

    > [!NOTE]
    > 请使用合适的“用户标识符”将用户从 Azure AD 映射到 Docusign 用户映射。 根据组织设置选择正确的字段并输入合适的值。

    j. 在“标识提供者证书”部分中，单击“添加证书”，然后上传已从 Azure AD 门户下载的证书并单击“保存”。

    ![配置单一登录][58]

    k. 在“标识提供者”部分中，单击“操作”，然后单击“终结点”。

    ![配置单一登录][59]

    l. 在“DocuSign 管理门户”上的“查看 SAML 2.0 终结点”部分中，执行以下步骤：

    ![配置单一登录][60]

    * 复制“服务提供程序颁发者 URL”，然后将其粘贴到 Azure 门户上的“基本 SAML 配置”部分的“标识符”文本框中。

    * 复制“服务提供程序登录 URL”，然后将其粘贴到 Azure 门户上的“基本 SAML 配置”部分的“登录 URL”文本框中。

    * 单击“关闭”。

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

在本部分中，通过授予 Britta Simon 访问 DocuSign 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“DocuSign”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“DocuSign”。

    ![“应用程序”列表中的“DocuSign”链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-docusign-test-user"></a>创建 DocuSign 测试用户

在本部分，我们将在 DocuSign 中创建名为 Britta Simon 的用户。 DocuSign 支持默认启用的实时用户预配。 此部分不存在任何操作项。 如果 DocuSign 中尚不存在用户，身份验证后会创建一个新用户。

>[!Note]
>如果需要手动创建用户，请联系 [DocuSign 支持团队](https://support.docusign.com/)。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“DocuSign”磁贴时，应会自动登录到设置了 SSO 的 DocuSign。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

<!--Image references-->

[50]: ./media/docusign-tutorial/tutorial_docusign_18.png
[51]: ./media/docusign-tutorial/tutorial_docusign_21.png
[52]: ./media/docusign-tutorial/tutorial_docusign_22.png
[53]: ./media/docusign-tutorial/tutorial_docusign_23.png
[54]: ./media/docusign-tutorial/tutorial_docusign_19.png
[55]: ./media/docusign-tutorial/tutorial_docusign_20.png
[56]: ./media/docusign-tutorial/tutorial_docusign_24.png
[57]: ./media/docusign-tutorial/tutorial_docusign_25.png
[58]: ./media/docusign-tutorial/tutorial_docusign_26.png
[59]: ./media/docusign-tutorial/tutorial_docusign_27.png
[60]: ./media/docusign-tutorial/tutorial_docusign_28.png
[61]: ./media/docusign-tutorial/tutorial_docusign_29.png
[62]: ./media/docusign-tutorial/tutorial_docusign_30.png
