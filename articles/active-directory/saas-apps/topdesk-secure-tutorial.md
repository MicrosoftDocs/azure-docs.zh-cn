---
title: 教程：Azure Active Directory 与 TOPdesk - Secure 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 TOPdesk - Secure 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 8e06ee33-18f9-4c05-9168-e6b162079d88
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0d0295162acdf358bd798e86bd7d3479f5e78d72
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407905"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---secure"></a>教程：Azure Active Directory 与 TOPdesk - Secure 集成

本教程介绍如何将 TOPdesk - Secure 与 Azure Active Directory (Azure AD) 集成。
将 TOPdesk - Secure 与 Azure AD 集成提供以下优势：

* 可在 Azure AD 中控制谁有权访问 TOPdesk - Secure。
* 可以让用户使用其 Azure AD 帐户自动登录到 TOPdesk - Secure（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 TOPdesk - Secure 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 TOPdesk - Secure 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* TOPdesk - Secure 支持 **SP** 发起的 SSO

## <a name="adding-topdesk---secure-from-the-gallery"></a>从库中添加 TOPdesk - Secure

要配置 TOPdesk - Secure 与 Azure AD 的集成，需要从库中将 TOPdesk - Secure 添加到托管 SaaS 应用列表。

**若要从库中添加 TOPdesk - Secure，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入 **TOPdesk - Secure**，在结果面板中选择“TOPdesk - Secure”，单击“添加”按钮添加该应用程序。

     ![结果列表中的 TOPdesk - Secure](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，将基于名为“Britta Simon”的测试用户配置和测试 TOPdesk - Secure 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 TOPdesk - Secure 相关用户之间建立链接关系。

若要配置和测试 TOPdesk - Secure 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 TOPdesk - Secure 单一登录](#configure-topdesk---secure-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 TOPdesk - Secure 测试用户](#create-topdesk---secure-test-user)** - 在 TOPdesk - Secure 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 TOPdesk - Secure 的 Azure AD 单一登录，请执行以下步骤：

1. [在 Azure 门户中的](https://portal.azure.com/) “TOPdesk - Secure” 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![TOPdesk - Secure 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<companyname>.topdesk.net`。

    b. 在“标识符”框中，使用以下模式键入 URL：`https://<companyname>.topdesk.net/tas/secure/login/verify`

    c. 在“回复 URL”文本框中，使用以下模式键入 URL：`https://<companyname>.topdesk.net/tas/public/login/saml`

    > [!NOTE]
    > 这些不是实际值。 请使用实际登录 URL、标识符和回复 URL 更新这些值。 请联系 [TOPdesk - Secure 客户端支持团队](https://www.topdesk.com/us/support/)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

6. 在“设置 TOPdesk - Secure”部分中，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-topdesk---secure-single-sign-on"></a>配置 TOPdesk - Secure 单一登录

1. 以管理员身份登录 **TOPdesk - Secure** 公司站点。

2. 在“TOPdesk”菜单中，单击“设置”。

    ![设置](./media/topdesk-secure-tutorial/ic790598.png "设置")

3. 单击“登录设置”。

    ![登录设置](./media/topdesk-secure-tutorial/ic790599.png "登录设置")

4. 展开“登录设置”菜单，并单击“常规”。

    ![常规](./media/topdesk-secure-tutorial/ic790600.png "常规")

5. 在“SAML 登录”配置部分的“安全”部分中，执行以下步骤：

    ![技术设置](./media/topdesk-secure-tutorial/ic790855.png "技术设置")

    a. 单击“下载”下载公共元数据文件，并将该文件保存到本地计算机上。

    b. 打开元数据文件，并找到“AssertionConsumerService”节点。

    ![断言使用者服务](./media/topdesk-secure-tutorial/ic790856.png "断言使用者服务")

    c. 复制 **AssertionConsumerService** 值，并将此值粘贴到“TOPdesk - Secure 域和 URL”部分中的“回复 URL”文本框内。

6. 若要创建证书文件，请执行以下步骤：

    ![证书](./media/topdesk-secure-tutorial/ic790606.png "证书")

    a. 在 Azure 门户中打开下载的元数据文件。

    b. 展开其 **xsi:type** 为 **fed:ApplicationServiceType** 的 **RoleDescriptor** 节点。

    c. 复制 **X509Certificate** 节点的值。

    d. 将复制的 **X509Certificate** 值保存到本地计算机上的文件中。

7. 在“公共”部分中，单击“添加”。

    ![添加](./media/topdesk-secure-tutorial/ic790607.png "Add")

8. 在“SAML 配置助手”对话框页上，执行以下步骤：

    ![SAML 配置助手](./media/topdesk-secure-tutorial/ic790608.png "SAML 配置助手")

    a. 若要从 Azure 门户上传已下载的元数据文件，请在“联合元数据”下单击“浏览”。

    b. 若要上载证书文件，请在“证书(RSA)”下，单击“浏览”。

    c. 对于“私钥(RSA, PKCS8, DER)”，可以上传自己的私钥，也可以联系 [TOPdesk - Secure 客户端支持团队](https://www.topdesk.com/us/support)获取私钥。

    d. 若要上载从 TOPdesk 支持团队获得的徽标文件，请在“徽标图标”下，单击“浏览”。

    e. 在“用户名属性”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。

    f. 在“显示名称”文本框中，键入配置名称。

    g. 单击“ **保存**”。

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

在本部分中，通过授予 Britta Simon 访问 TOPdesk - Secure 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“TOPdesk - Secure”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，键入并选择“TOPdesk - Secure”。

    ![应用程序列表中的 TOPdesk - Secure 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-topdesk---secure-test-user"></a>创建 TOPdesk - Secure 测试用户

要使 Azure AD 用户能够登录 TOPdesk - Secure，必须将这些用户预配到 TOPdesk - Secure 中。  
对于 TOPdesk - Secure，预配是一项手动任务。

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>若要配置用户设置，请执行以下步骤：

1. 以管理员身份登录 **TOPdesk - Secure** 公司站点。

2. 在顶部菜单中，单击“TOPdesk”\>“新建”\>“支持文件”\>“操作员”。

    ![操作员](./media/topdesk-secure-tutorial/ic790610.png "操作员")

3. 在“新建操作员”对话框中，执行以下步骤：

    ![新建操作员](./media/topdesk-secure-tutorial/ic790611.png "新建操作员")

    a. 单击“常规”选项卡。

    b. 在“姓氏”文本框中，键入用户的姓氏（如 Simon）。

    c. 在“位置”部分中选择帐户所在的**站点**。

    d. 在“TOPdesk 登录”部分的“登录名”文本框中，键入用户的登录名。

    e. 单击“ **保存**”。

> [!NOTE]
> 可以使用任何其他 TOPdesk - Secure 用户帐户创建工具或 TOPdesk - Secure 提供的 API 来预配 AAD 用户帐户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 TOPdesk - Secure 磁贴时，应当会自动登录到已为其设置了 SSO 的 TOPdesk - Secure。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

