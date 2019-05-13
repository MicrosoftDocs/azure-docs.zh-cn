---
title: 教程：Azure Active Directory 与 Zscaler Beta 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 Zscaler Beta 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 56b846ae-a1e7-45ae-a79d-992a87f075ba
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/24/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: f31361dc3d7e24092677f1a78b2c405ae84578ed
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230052"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-beta"></a>教程：Azure Active Directory 与 Zscaler Beta 的集成

本教程介绍如何将 Zscaler Beta 与 Azure Active Directory (Azure AD) 集成。
将 Zscaler Beta 与 Azure AD 集成后，可以：

* 在 Azure AD 中控制谁有权访问 Zscaler Beta。
* 可让用户使用其 Azure AD 帐户自动登录到 Zscaler Beta。 这种访问控制称为单一登录 (SSO)。
* 使用 Azure 门户在一个中心位置管理自己的帐户。

有关服务型软件 (SaaS) 应用与 Azure AD 集成的详细信息，请参阅[什么是使用 Azure Active Directory 的应用程序访问和单一登录？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Zscaler Beta 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 使用单一登录的 Zscaler Beta 订阅。

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Zscaler Beta 支持 SP 发起的 SSO。
* Zscaler Beta 支持实时用户预配。

## <a name="add-zscaler-beta-from-the-azure-marketplace"></a>从 Azure 市场中添加 Zscaler Beta

若要配置 Zscaler Beta 与 Azure AD 的集成，需从 Azure 市场中将 Zscaler Beta 添加到托管 SaaS 应用列表。

若要从 Azure 市场中添加 Zscaler Beta，请执行以下步骤。

1. 在 [Azure 门户](https://portal.azure.com)中，在左侧导航窗格中，选择“Azure Active Directory”。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用程序”，并选择“所有应用程序”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请在对话框顶部选择“新建应用程序”。

    ![“新建应用程序”按钮](common/add-new-app.png)

4. 在搜索框中输入 **Zscaler Beta**。 在结果面板中选择“Zscaler Beta”，然后选择“添加”。

     ![结果列表中的 Zscaler Beta](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于测试用户 Britta Simon 配置并测试 Zscaler Beta 的 Azure AD 单一登录。
若要正常使用单一登录，请在 Azure AD 用户与 Zscaler Beta 中的相关用户之间建立链接关系。

若要配置并测试 Zscaler Beta 的 Azure AD 单一登录，请完成以下构建基块：

- [配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)，使用户能够使用此功能。
- [配置 Zscaler Beta 单一登录](#configure-zscaler-beta-single-sign-on) - 在应用程序端配置单一登录设置。
- [创建 Azure AD 测试用户](#create-an-azure-ad-test-user)，使用 Britta Simon 测试 Azure AD 单一登录。
- [分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)，使 Britta Simon 能够使用 Azure AD 单一登录。
- [创建 Zscaler Beta 测试用户](#create-a-zscaler-beta-test-user) - 在 Zscaler Beta 中创建 Britta Simon 的对应用户，其与用户的 Azure AD 表示形式相关联。
- [测试单一登录](#test-single-sign-on)，以验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Zscaler Beta 的 Azure AD 单一登录，请执行以下步骤。

1. 在 [Azure 门户](https://portal.azure.com/)中的 Zscaler Beta 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在“选择单一登录方法”对话框中，选择“SAML/WS-Fed”模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“设置 SAML 单一登录”页上，选择“编辑”图标打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分执行以下步骤：

    ![Zscaler Beta 域和 URL 单一登录信息](common/sp-intiated.png)

    - 在“登录 URL”框中，键入用户用来登录 Zscaler Beta 应用程序的 URL。

    > [!NOTE]
    > 此值不是真实值。 请使用实际的“登录 URL”值更新此值。 若要获取此值，请联系 [Zscaler Beta 客户端支持团队](https://www.zscaler.com/company/contact)。

5. Zscaler Beta 应用程序需要特定格式的 SAML 断言。 必须将自定义属性映射添加到 SAML 令牌属性配置。 以下屏幕截图显示了默认属性的列表。 选择“编辑”打开“用户属性”对话框。 **** 

    ![“用户属性”对话框](common/edit-attribute.png)

6. Zscaler Beta 应用程序还要求在 SAML 响应中传回其他几个属性。 在“用户属性”对话框的“用户声明”部分执行以下步骤，以添加 SAML 令牌属性，如下表所示。
    
    | 名称 | 源属性 | 
    | ---------------| --------------- |
    | memberOf  | user.assignedroles |

    a. 选择“添加新声明”打开“管理用户声明”对话框。

    ![“用户声明”对话框](common/new-save-attribute.png)

    ![“管理用户声明”对话框](common/new-attribute-details.png)

    b. 在“名称”框中，键入该行显示的属性名称。

    c. 将“命名空间”框留空。

    d. 对于“源”，请选择“属性”。

    e. 在“源属性”列表中，输入该行显示的属性值。

    f. 选择“确定”。

    g. 选择“保存”。

    > [!NOTE]
    > 若要了解如何在 Azure AD 中配置角色，请参阅[配置角色声明](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management)。

7. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，选择“下载”以下载“证书(Base64)”。 然后将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

8. 在“设置 Zscaler Beta”部分，根据要求复制所需的 URL：

    ![复制配置 URL](common/copy-configuration-urls.png)

    - 登录 URL
    - Azure AD 标识符
    - 注销 URL

### <a name="configure-zscaler-beta-single-sign-on"></a>配置 Zscaler Beta 单一登录

1. 若要在 Zscaler Beta 中自动执行配置，需要通过选择“安装扩展”，安装“我的应用安全登录浏览器扩展”。

    ![“我的应用”扩展](common/install-myappssecure-extension.png)

2. 将该扩展添加到浏览器后，选择“设置 Zscaler Beta”会定向到 Zscaler Beta 应用程序。 在此处，请提供管理员凭据以登录到 Zscaler Beta。 浏览器扩展会自动配置应用程序，并自动执行步骤 3 至 6。

    ![设置配置](common/setup-sso.png)

3. 若要手动设置 Zscaler Beta，请打开新的 Web 浏览器窗口。 以管理员身份登录到 Zscaler Beta 公司站点，然后执行以下步骤。

4. 转到“管理” > “身份验证” > “身份验证设置”，然后执行以下步骤。
   
    ![管理](./media/zscaler-beta-tutorial/ic800206.png "Administration")

    a. 在“身份验证类型”下选择“SAML”。

    b. 选择“配置 SAML”。

5. 在“编辑 SAML”窗口中执行以下步骤： 
            
    ![管理用户和身份验证](./media/zscaler-beta-tutorial/ic800208.png "管理用户和身份验证")
    
    a. 在“SAML 门户 URL”框中，粘贴从 Azure 门户复制的“登录 URL”。

    b. 在“登录名属性”框中输入 **NameID**。

    c. 在“公共 SSL 证书”框中，选择“上传”以上传从 Azure 门户下载的 Azure SAML 签名证书。

    d. 切换为“启用 SAML 自动预配”。

    e. 若要为 displayName 属性启用 SAML 自动预配，请在“用户显示名称属性”框中输入 **displayName**。

    f. 若要为 memberOf 属性启用 SAML 自动预配，请在“组名称属性”框中输入 **memberOf**。

    g. 若要为 department 属性启用 SAML 自动预配，请在“部门名称属性”框中输入 **department**。

    h. 选择“保存”。

6. 在“配置用户身份验证”对话框页上执行以下步骤：

    ![“激活”菜单和“激活”按钮](./media/zscaler-beta-tutorial/ic800207.png)

    a. 将鼠标悬停在左下角的“激活”菜单上。

    b. 选择“激活”。

## <a name="configure-proxy-settings"></a>配置代理设置
若要在 Internet Explorer 中配置代理设置，请执行以下步骤。

1. 启动 **Internet Explorer**。

2. 从“工具”菜单中选择“Internet 选项”，打开“Internet 选项”对话框。 
    
     ![“Internet 选项”对话框](./media/zscaler-beta-tutorial/ic769492.png "Internet 选项")

3. 选择“连接”选项卡。 
  
     ![“连接”选项卡](./media/zscaler-beta-tutorial/ic769493.png "连接")

4. 选择“LAN 设置”打开“局域网(LAN)设置”对话框。

5. 在“代理服务器”部分执行以下步骤： 
   
    ![“代理服务器”部分](./media/zscaler-beta-tutorial/ic769494.png "代理服务器")

    a. 选中“对 LAN 使用代理服务器”复选框。

    b. 在“地址”框中输入 **gateway.Zscaler Beta.net**。

    c. 在“端口”框中输入 **80**。

    d. 选中“对本地地址不使用代理服务器”复选框。

    e. 选择“确定”关闭“局域网(LAN)设置”对话框。

6. 选择“确定”关闭“Internet 选项”对话框。

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户 

在 Azure 门户中，创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左窗格中，选择“Azure Active Directory” > “用户” > “所有用户”。

    ![“用户”和“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户”对话框中执行以下步骤：

    ![“用户”对话框](common/user-properties.png)

    a. 在“姓名”框中，输入 **BrittaSimon**。
  
    b. 在“用户名”框中输入 `brittasimon@yourcompanydomain.extension`。 例如 BrittaSimon@contoso.com。

    c. 选中“显示密码”复选框。 记下“密码”框中显示的值。

    d. 选择“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

通过授予 Britta Simon 访问 Zscaler Beta 的权限，使其能够使用 Azure 单一登录。

1. 在 Azure 门户中，选择“企业应用程序” > “所有应用程序” > “Zscaler Beta”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，输入并选择“Zscaler Beta”。

    ![“应用程序”列表中的“Zscaler Beta”链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 选择“添加用户”。 在“添加分配”对话框中选择“用户和组”。

    ![“添加用户”按钮](common/add-assign-user.png)

5. 在“用户和组”对话框中，从列表中选择类似于“Britta Simon”的用户。 然后选择屏幕底部的“选择”。

    ![“用户和组”对话框](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_users.png)

6. 在“选择角色”对话框中，从列表中选择相应的用户角色。 然后选择屏幕底部的“选择”。

    ![“选择角色”对话框](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_roles.png)

7. 在“添加分配”对话框中选择“分配”。

    ![“添加分配”对话框](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_assign.png)

### <a name="create-a-zscaler-beta-test-user"></a>创建 Zscaler Beta 测试用户

在本部分，我们将在 Zscaler Beta 中创建用户 Britta Simon。 Zscaler Beta 支持默认启用的**恰时用户预配**。 在本部分不需要执行任何操作。 如果 Zscaler Beta 中尚不存在用户，则会在身份验证后创建一个新用户。

>[!Note]
>若要手动创建用户，请联系 [Zscaler Beta 支持团队](https://www.zscaler.com/company/contact)。

### <a name="test-single-sign-on"></a>测试单一登录 

使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“Zscaler Beta”磁贴时，应会自动登录到设置了 SSO 的 Zscaler Beta。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

