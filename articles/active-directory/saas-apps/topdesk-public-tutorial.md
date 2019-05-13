---
title: 教程：Azure Active Directory 与 TOPdesk - Public 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 TOPdesk - Public 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 0873299f-ce70-457b-addc-e57c5801275f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: jeedes
ms.openlocfilehash: d5ecfcd249dd07dc94b3b17ea0a7a7de3559c681
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407940"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---public"></a>教程：Azure Active Directory 与 TOPdesk - Public 集成

本教程介绍如何将 TOPdesk - Public 与 Azure Active Directory (Azure AD) 集成。
将 TOPdesk - Public 与 Azure AD 集成提供以下优势：

* 可在 Azure AD 中控制谁有权访问 TOPdesk - Public
* 可以让用户使用其 Azure AD 帐户自动登录到 TOPdesk - Public（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 TOPdesk - Public 的集成，需准备好以下各项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 启用了单一登录的 TOPdesk - Public 订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* TOPdesk - Public 支持 **SP** 发起的 SSO

## <a name="adding-topdesk---public-from-the-gallery"></a>从库添加 TOPdesk - Public

要配置 TOPdesk - Public 与 Azure AD 的集成，需要从库中将 TOPdesk - Public 添加到托管 SaaS 应用列表。

**若要从库添加 TOPdesk - Public，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入 **TOPdesk - Public**，在结果面板中选择“TOPdesk - Public”，单击“添加”按钮添加该应用程序。

     ![结果列表中的 TOPdesk - Public](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，将基于名为 **Britta Simon** 的测试用户配置和测试 TOPdesk - Public 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 TOPdesk - Public 中相关用户之间建立链接关系。

若要配置和测试 TOPdesk - Public 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 TOPdesk - Public 单一登录](#configure-topdesk---public-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 TOPdesk - Public 测试用户](#create-topdesk---public-test-user)** - 在 TOPdesk - Public 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 TOPdesk - Public 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中，在 **TOPdesk - Public** 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4.  在“基本 SAML 配置”部分，如果有**服务提供程序元数据文件**，请执行以下步骤：

    >[!NOTE]
    >从本教程下文中将介绍的“配置 TOPdesk - Public 单一登录”部分中获取**服务提供商元数据文件**。

    a. 单击“上传元数据文件”。
    
    ![上传元数据文件](common/upload-metadata.png)

    b. 单击“文件夹徽标”来选择元数据文件并单击“上传”。

    ![选择元数据文件](common/browse-upload-metadata.png)

    c. 成功上传元数据文件后，“标识符”和“回复 URL”值会自动填充在“基本 SAML 配置”部分。

    ![TOPdesk - Public 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    d. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<companyname>.topdesk.net`。

    e. 在“标识符”文本框中，使用以下模式键入 URL：`https://<companyname>.topdesk.net/tas/public/login/verify`

    > [!NOTE] 
    > 如果“标识符”和“回复 URL”值未自动填充，则需要手动输入它们。 对于标识符，请遵循上面提到的模式，你可从本教程下文中将介绍的“配置 TOPdesk - Public 单一登录”部分获取回复 URL。 “登录 URL”值不是真实值，因此你需要将该值更新为实际登录 URL。 请联系 [TOPdesk - Public 客户端支持团队](https://help.topdesk.com/saas/enterprise/user/)获取该值。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

6. 在“设置 TOPdesk - Public”部分中，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-topdesk---public-single-sign-on"></a>配置 TOPdesk - Public 单一登录

1. 以管理员身份登录 **TOPdesk - Public** 公司站点。

2. 在“TOPdesk”菜单中，单击“设置”。
   
    ![设置](./media/topdesk-public-tutorial/ic790598.png "设置")

3. 单击“登录设置”。
   
    ![登录设置](./media/topdesk-public-tutorial/ic790599.png "登录设置")

4. 展开“登录设置”菜单，并单击“常规”。
   
    ![常规](./media/topdesk-public-tutorial/ic790600.png "常规")

5. 在“SAML 登录”配置部分的“公共”部分中，执行以下步骤：
   
    ![技术设置](./media/topdesk-public-tutorial/ic790601.png "技术设置")
   
    a. 单击“下载”下载公共元数据文件，并将该文件保存到本地计算机上。
   
    b. 打开下载的元数据文件，找到 **AssertionConsumerService** 节点。

    ![AssertionConsumerService](./media/topdesk-public-tutorial/ic790619.png "AssertionConsumerService")
   
    c. 复制 **AssertionConsumerService** 值，并将此值粘贴到“基本 SAML 配置”部分中的“回复 URL”文本框内。      
   
6. 若要创建证书文件，请执行以下步骤：
    
    ![证书](./media/topdesk-public-tutorial/ic790606.png "证书")
    
    a. 在 Azure 门户中打开下载的元数据文件。
    
    b. 展开其 **xsi:type** 为 **fed:ApplicationServiceType** 的 **RoleDescriptor** 节点。
    
    c. 复制 **X509Certificate** 节点的值。
    
    d. 将复制的 **X509Certificate** 值保存到本地计算机上的文件中。

7. 在“公共”部分中，单击“添加”。
    
    ![SAML 登录](./media/topdesk-public-tutorial/ic790625.png "SAML 登录")

8. 在“SAML 配置助手”对话框页上，执行以下步骤：
    
    ![SAML 配置助手](./media/topdesk-public-tutorial/ic790608.png "SAML 配置助手")
    
    a. 若要从 Azure 门户上传已下载的元数据文件，请在“联合元数据”下单击“浏览”。

    b. 若要上载证书文件，请在“证书(RSA)”下，单击“浏览”。

    c. 若要上载从 TOPdesk 支持团队获得的徽标文件，请在“徽标图标”下，单击“浏览”。

    d. 在“用户名属性”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。

    e. 在“显示名称”文本框中，键入配置名称。

    f. 单击“ **保存**”。

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

在本部分，通过授予 Britta Simon 访问 TOPdesk - Public 的权限，使其能够使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“TOPdesk - Public”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“TOPdesk - Public”。

    ![应用程序列表中的 TOPdesk - Public 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-topdesk---public-test-user"></a>创建 TOPdesk - Public 测试用户

要使 Azure AD 用户能够登录 TOPdesk - Public，必须将这些用户预配到 TOPdesk - Public 中。 对于 TOPdesk - Public，预配是一项手动任务。

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>若要配置用户设置，请执行以下步骤：

1. 以管理员身份登录 **TOPdesk - Public** 公司站点。

2. 在顶部菜单中，单击“TOPdesk”\>“新建”\>“支持文件”\>“人员”。
   
    ![人员](./media/topdesk-public-tutorial/ic790628.png "人员")

3. 在“新建用户”对话框中，执行以下步骤：
   
    ![新建人员](./media/topdesk-public-tutorial/ic790629.png "新建人员")
   
    a. 单击“常规”选项卡。

    b. 在“姓氏”文本框中，键入用户的姓氏（如 Simon）
 
    c. 为该帐户选择**站点**。
 
    d. 单击“ **保存**”。

> [!NOTE]
> 可以使用任何其他 TOPdesk - Public 用户帐户创建工具或 TOPdesk - Public 提供的 API 来预配 Azure AD 用户帐户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 TOPdesk - Public 磁贴时，应当会自动登录到已为其设置了 SSO 的 TOPdesk - Public。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
