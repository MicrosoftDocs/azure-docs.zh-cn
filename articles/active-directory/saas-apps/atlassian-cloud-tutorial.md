---
title: 教程：Azure Active Directory 与 Atlassian Cloud 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 Atlassian Cloud 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 729b8eb6-efc4-47fb-9f34-8998ca2c9545
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/03/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 66b8b9076c0a4b1fddda4ab0bcfe9f104d7dcf8a
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191127"
---
# <a name="tutorial-azure-active-directory-integration-with-atlassian-cloud"></a>教程：Azure Active Directory 与 Atlassian Cloud 的集成

本教程介绍如何将 Atlassian Cloud 与 Azure Active Directory (Azure AD) 集成。
将 Atlassian Cloud 与 Azure AD 集成可提供以下优势：

* 可以在 Azure AD 中控制谁有权限访问 Atlassian Cloud。
* 可以让用户使用其 Azure AD 帐户自动登录到 Atlassian Cloud（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Atlassian Cloud 的集成，需要具有以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 Atlassian Cloud 单一登录的订阅
* 若要对 Atlassian Cloud 产品启用安全断言标记语言 (SAML) 单一登录，需要设置 Atlassian Access。 详细了解 [Atlassian Access]( https://www.atlassian.com/enterprise/cloud/identity-manager)。

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Atlassian Cloud 支持 SP 和 IDP 发起的 SSO

## <a name="adding-atlassian-cloud-from-the-gallery"></a>从库中添加 Atlassian Cloud

要配置 Atlassian Cloud 与 Azure AD 的集成，需要从库中将 Atlassian Cloud 添加到托管 SaaS 应用列表。

**若要从库中添加 Atlassian Cloud，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“Atlassian Cloud”，在结果面板中选择“Atlassian Cloud”，然后单击“添加”按钮来添加应用程序。

     ![结果列表中的 Atlassian Cloud](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于名为“Britta Simon”的测试用户配置并测试 Atlassian Cloud 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 Atlassian Cloud 相关用户之间建立链接关系。

若要配置并测试 Atlassian Cloud 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Atlassian Cloud 单一登录](#configure-atlassian-cloud-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Atlassian Cloud 测试用户](#create-atlassian-cloud-test-user)** - 在 Atlassian Cloud 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Atlassian Cloud 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Atlassian Cloud”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 如果要在 **IDP** 发起的模式下配置应用程序，请在“基本 SAML 配置”部分执行以下步骤：

    ![[应用程序名称] 域和 URL 单一登录信息](common/idp-relay.png)

    a. 在“标识符”文本框中，使用以下模式键入 URL：`https://auth.atlassian.com/saml/<unique ID>`

    b. 在“回复 URL”文本框中，使用以下模式键入 URL：`https://auth.atlassian.com/login/callback?connection=saml-<unique ID>`

    c. 单击“设置其他 URL”。

    d. 在“中继状态”文本框中，使用以下模式键入 URL：`https://<instancename>.atlassian.net`

    > [!NOTE]
    > 上面的值不是实际值。 请使用实际标识符和回复 URL 更新这些值。 可以从“Atlassian Cloud SAML 配置”屏幕获取这些实际值，具体将在本教程的“配置 Atlassian Cloud 单一登录”部分中进行介绍。

5. 如果要在 SP 发起的模式下配置应用程序，请单击“设置其他 URL”，并执行以下步骤：

    ![[应用程序名称] 域和 URL 单一登录信息](common/both-signonurl.png)

    在“登录 URL”文本框中，使用以下模式键入 URL：`https://<instancename>.atlassian.net`

    > [!NOTE]
    > 上面的“登录 URL”值不是实际值。 请使用实际的登录 URL 更新此值。 若要获取此值，请与 [Atlassian Cloud 客户端支持团队](https://support.atlassian.com/)联系。

6. Atlassian Cloud 应用程序需要特定格式的 SAML 断言，这需要向“SAML 令牌属性”配置添加自定义属性映射。 以下屏幕截图显示了默认属性的列表，其中的 **nameidentifier** 通过 **user.userprincipalname** 进行映射。 Atlassian Cloud 应用程序要求通过 **user.mail** 对 **nameidentifier** 进行映射，因此需单击“编辑”图标对属性映射进行编辑，然后更改属性映射。

    ![image](common/edit-attribute.png)

7. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

8. 在“设置 Atlassian Cloud”部分中，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-atlassian-cloud-single-sign-on"></a>配置 Atlassian Cloud 单一登录

1. 若要在 Atlassian Cloud 中自动执行配置，需要安装“我的应用安全登录浏览器扩展”，方法是单击“安装扩展”。

    ![我的应用扩展](common/install-myappssecure-extension.png)

2. 将扩展添加到浏览器后，单击“设置 Atlassian Cloud”会将你定向到 Atlassian Cloud 应用程序。 在此处，提供管理员凭据以登录到 Atlassian Cloud。 浏览器扩展会自动配置应用程序，并自动执行第 3-7 步。

    ![设置配置](common/setup-sso.png)

3. 如果想要手动设置 Atlassian Cloud，请打开新的 Web 浏览器窗口并以管理员身份登录到 Atlassian Cloud 公司网站，并执行以下步骤：

4. 在打算配置单一登录前，需要对你的域进行验证。 有关详细信息，请参阅 [Atlassian 域验证](https://confluence.atlassian.com/cloud/domain-verification-873871234.html)文档。

5. 在左窗格中，选择“SAML 单一登录”。 如果尚未订阅 Atlassian Identity Manager，请订阅。

    ![配置单一登录](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_11.png)

6. 在“添加 SAML 配置”窗口中执行以下操作：

    ![配置单一登录](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_12.png)

    a. 在“标识提供者实体 ID”框中，粘贴从 Azure 门户复制的 **Azure AD 标识符**。

    b. 在“标识提供者 SSO URL”框中，粘贴从 Azure 门户复制的**登录 URL**。

    c. 在 Azure 门户中打开下载的证书 .txt 文件，复制其中包含的值（不要复制 *Begin Certificate* 和 *End Certificate* 行），然后将其粘贴到“公共 X509 证书”框中。

    d. 单击“保存配置”。

7. 为了确保设置正确的 URL，请执行以下操作来更新 Azure AD 设置：

    ![配置单一登录](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_13.png)

    a. 在“SAML”窗口中，复制“SP 标识 ID”，然后在 Azure 门户中 Atlassian Cloud 的“基本 SAML 配置”下，将此 ID 粘贴到“标识符”框中。

    b. 在“SAML”窗口中，复制“SP 断言使用者服务 URL”，然后在 Azure 门户中 Atlassian Cloud 的“基本 SAML 配置”下，将此 URL 粘贴到“回复 URL”框中。 “登录 URL”是 Atlassian Cloud 的租户 URL。

    > [!NOTE]
    > 现有客户在 Azure 门户中更新“SP 标识 ID”和“SP 断言使用者服务 URL”值之后，应选择“是，更新配置”。 新客户可以跳过此步骤。

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

在本部分中，通过授予 Britta Simon 访问 Atlassian Cloud 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“Atlassian Cloud”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，键入并选择“Atlassian Cloud”。

    ![应用程序列表中的 Atlassian Cloud 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-atlassian-cloud-test-user"></a>创建 Atlassian Cloud 测试用户

若要使 Azure AD 用户能够登录到 Atlassian Cloud，请执行以下操作，在 Atlassian Cloud 中手动预配用户帐户：

1. 在“管理”窗格中，选择“用户”。

    ![Atlassian Cloud“用户”链接](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_14.png)

2. 若要在 Atlassian Cloud 中创建用户，请选择“邀请用户”。

    ![创建 Atlassian Cloud 用户](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_15.png)

3. 在“电子邮件地址”框中输入该用户的电子邮件地址，然后分配应用程序访问权限。

    ![创建 Atlassian Cloud 用户](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_16.png)

4. 若要向该用户发送电子邮件邀请，请选择“邀请用户”。 随后会将一封电子邮件邀请发送给该用户，接受邀请后，该用户将在系统中激活。

> [!NOTE]
> 还可以通过选择“用户”部分中的“批量创建”按钮来批量创建用户。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Atlassian Cloud 磁贴时，应当会自动登录到你为其设置了 SSO 的 Atlassian Cloud。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
   
