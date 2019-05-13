---
title: 教程：Azure Active Directory 与 Workday 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Workday 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: e9da692e-4a65-4231-8ab3-bc9a87b10bca
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/04/2019
ms.author: jeedes
ms.openlocfilehash: 26a9aa4c32cff17dd157d65dce062309e73eb41f
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407892"
---
# <a name="tutorial-azure-active-directory-integration-with-workday"></a>教程：Azure Active Directory 与 Workday 的集成

本教程介绍如何将 Workday 与 Azure Active Directory (Azure AD) 集成。
将 Workday 与 Azure AD 集成可提供以下优势：

* 可在 Azure AD 中控制谁有权访问 Workday。
* 可让用户使用其 Azure AD 帐户自动登录到 Workday（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Workday 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 Workday 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Workday 支持 **SP** 和 **IDP** 发起的 SSO

## <a name="adding-workday-from-the-gallery"></a>从库中添加 Workday

若要配置 Workday 与 Azure AD 的集成，需要从库中将 Workday 添加到托管 SaaS 应用列表。

若要从库中添加 Workday，请执行以下步骤：

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入“Workday”，在结果面板中选择“Workday”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 Workday](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 Workday 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 Workday 相关用户之间建立链接关系。

若要配置和测试 Workday 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Workday 单一登录](#configure-workday-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Workday 测试用户](#create-workday-test-user)** - 在 Workday 中创建 Britta Simon 的对应用户，并将其关联到其在 Azure AD 中的表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Workday 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Workday”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![Workday 域和 URL 单一登录信息](common/sp-identifier.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https:\//impl.workday.com/<tenant>/login-saml2.flex`。

    b. 在“标识符”文本框中，使用以下模式键入 URL：`https://www.workday.com`

5. 单击“设置其他 URL”并执行以下步骤：

    ![Workday 域和 URL 单一登录信息](./media/workday-tutorial/reply.png)

    在“回复 URL”文本框中，使用以下模式键入 URL：`https:\//impl.workday.com/<tenant>/login-saml.htmld`

    > [!NOTE]
    > 这些不是实际值。 请使用实际的登录 URL 和回复 URL 更新这些值。 回复 URL 必须具有一个子域（例如：www、wd2、wd3、wd3-impl、wd5 和 wd5-impl）。
    > 可以使用类似于 `http://www.myworkday.com` 的内容，但不能使用 `https://myworkday.com`。 请联系 [Workday 客户端支持团队](https://www.workday.com/en-us/partners-services/services/support.html)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

6. Workday 应用程序需要特定格式的 SAML 断言，因此，需要在 SAML 令牌属性配置中添加自定义属性映射。 以下屏幕截图显示了默认属性的列表，其中的 **nameidentifier** 通过 **user.userprincipalname** 进行映射。 Workday 应用程序要求通过 **user.mail**、**UPN** 等对 **nameidentifier** 进行映射，因此需单击“编辑”图标对属性映射进行编辑，然后更改属性映射。

    ![image](common/edit-attribute.png)

    > [!NOTE]
    > 此处我们已将名称 ID 与 UPN (user.userprincipalname) 映射为默认值。 需要在 Workday 帐户（电子邮件、UPN 等）中将名称 ID 与实际用户 ID 进行映射，以便成功运行 SSO。

7. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

8. 在“设置 Workday”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-workday-single-sign-on"></a>配置 Workday 单一登录

1. 在另一 Web 浏览器窗口中，以管理员身份登录 Workday 公司站点。

2. 在主页左上方的“搜索框”中使用名称“编辑租户设置 - 安全性”搜索。

    ![编辑租户安全性](./media/workday-tutorial/IC782925.png "编辑租户安全性")

3. 在“重定向 URL”部分中，执行以下步骤：

    ![重定向 URL](./media/workday-tutorial/IC7829581.png "重定向 URL")

    a. 单机“添加行”。

    b. 在“登录重定向 URL”文本框和“移动重定向 URL”文本框中，键入在 Azure 门户的“基本 SAML 配置”部分中输入的“登录 URL”。

    c. 在 Azure 门户的“设置 Workday”部分复制“注销 URL”，然后将其粘贴到“注销重定向 URL”文本框中。

    d. 在“用于环境”文本框中，选择环境名称。  

   > [!NOTE]
   > “环境”属性的值与租户 URL 的值绑定：  
   > 如果 Workday 租户 URL 的域名以 impl 开头（例如：*https:\//impl.workday.com/\<tenant\>/login-saml2.flex*），则“环境”属性必须设置为“实现”。  
   > 如果域名以其他内容开头，则需要联系 [Workday 客户端支持团队](https://www.workday.com/en-us/partners-services/services/support.html)，获取匹配的“环境”值。

4. 在“SAML 设置”部分中执行以下步骤：

    ![SAML 设置](./media/workday-tutorial/IC782926.png "SAML 设置")

    a.  选择“启用 SAML 身份验证”。

    b.  单击“添加行”。

5. 在“SAML 标识提供者”部分中，执行以下步骤：

    ![SAML 标识提供者](./media/workday-tutorial/IC7829271.png "SAML 标识提供者")

    a. 在“标识提供者名称”文本框中，键入提供者名称（例如：SPInitiatedSSO）。

    b. 在 Azure 门户的“设置 Workday”部分复制“Azure AD 标识符”值，然后将其粘贴到“颁发者”文本框中。

    ![SAML 标识提供者](./media/workday-tutorial/IC7829272.png "SAML 标识提供者")

    c. 在 Azure 门户的“设置 Workday”部分复制“注销 URL”值，然后将其粘贴到“注销响应 URL”文本框中。

    d. 在 Azure 门户的“设置 Workday”部分复制“登录 URL”值，然后将其粘贴到“IdP SSO 服务 URL”文本框中。

    e. 在“用于环境”文本框中，选择环境名称。

    f. 单击“标识提供者公钥证书”，并单击“创建”。

    ![创建](./media/workday-tutorial/IC782928.png "创建")

    g. 单击“创建 x509 公钥”。

    ![创建](./media/workday-tutorial/IC782929.png "创建")

6. 在“查看 x509 公钥”部分中，执行以下步骤：

    ![查看 x509 公钥](./media/workday-tutorial/IC782930.png "查看 x509 公钥")

    a. 在“名称”文本框中，键入证书名称（例如：PPE\_SP）。

    b. 在**有效起始日期**文本框中，键入证书的“有效起始日期”属性值。

    c.  在**有效截止日期**文本框中，键入证书的“有效截止日期”属性值。

    > [!NOTE]
    > 可以双击下载的证书，以从中获取有效起始日期和有效截止日期。  该日期在“详细信息”选项卡下列出。
    >
    >

    d.  在记事本中打开 Base-64 编码的证书，并复制其内容。

    e.  在“证书”文本框中，粘贴剪贴板的内容。

    f.  单击“确定”。

7. 执行以下步骤：

    ![SSO 配置](./media/workday-tutorial/WorkdaySSOConfiguratio.png "SSO 配置")

    a.  在“服务提供商 ID”文本框中，键入 **https://www.workday.com**。

    b. 选择“不削弱 SP 发起的身份验证请求”。

    c. 对于**身份验证请求签名方法**，请选择 **SHA256**。

    ![身份验证请求签名方法](./media/workday-tutorial/WorkdaySSOConfiguration.png "身份验证请求签名方法") 

    d. 单击“确定”。

    ![确定](./media/workday-tutorial/IC782933.png "确定")

    > [!NOTE]
    > 请确保正确设置单一登录。 如果使用不正确的设置启用单一登录，则可能无法使用凭据进入应用程序并被锁定。在这种情况下，Workday 提供备份登录 URL，用户可以使用以下格式的普通用户名和密码登录：[Workday URL]/login.flex?redirect=n

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

本部分中，通过授予 Britta Simon 访问 Workday 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“Workday”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“Workday”。

    ![应用程序列表中的 Workday 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-workday-test-user"></a>创建 Workday 测试用户

在本部分中，将在 Workday 中创建一个名为“Britta Simon”的用户。 请与 [Workday 客户端支持团队](https://www.workday.com/en-us/partners-services/services/support.html)协作来在 Workday 平台中添加用户。 使用单一登录前，必须先创建并激活用户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“Workday”磁贴时，应会自动登录到设置了 SSO 的 Workday。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

