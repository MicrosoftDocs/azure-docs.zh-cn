---
title: 教程：将 Azure Active Directory 与 MOVEit Transfer - Azure AD 集成集成 | Microsoft 文档
description: 了解如何配置 Azure Active Directory 和 MOVEit Transfer - Azure AD 集成之间的单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 8ff7102d-be73-4888-ae81-d8e3d01dd534
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/25/2019
ms.author: jeedes
ms.openlocfilehash: 2a491cd303cccd94cbb489654890d55c03912a63
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408047"
---
# <a name="tutorial-azure-active-directory-integration-with-moveit-transfer---azure-ad-integration"></a>教程：将 Azure Active Directory 与 MOVEit Transfer - Azure AD 集成

此教程介绍如何将 MOVEit Transfer - Azure AD 集成与 Azure Active Directory (Azure AD) 集成。
将 MOVEit Transfer - Azure AD 集成与 Azure AD 集成可提供以下优势：

* 可在 Azure AD 中控制谁有权访问 MOVEit Transfer - Azure AD 集成。
* 可以让用户使用其 Azure AD 帐户自动登录到 MOVEit Transfer - Azure AD integration（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 MOVEit Transfer - Azure AD 集成之间的集成，需要具有以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 MOVEit Transfer - Azure AD integration 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* MOVEit Transfer - Azure AD integration 支持 **SP** 发起的 SSO

## <a name="adding-moveit-transfer---azure-ad-integration-from-the-gallery"></a>从库中添加 MOVEit Transfer - Azure AD 集成

要配置 MOVEit Transfer - Azure AD 集成与 Azure AD 的集成，需要从库中将 MOVEit Transfer - Azure AD 集成添加到托管 SaaS 应用列表。

若要从库中添加 MOVEit Transfer - Azure AD 集成，请执行以下步骤：

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“MOVEit Transfer - Azure AD 集成”，从结果面板中选择“MOVEit Transfer - Azure AD 集成”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 MOVEit Transfer - Azure AD 集成](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 MOVEit Transfer - Azure AD integration 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 MOVEit Transfer - Azure AD integration 相关用户之间建立链接关系。

若要配置并测试 MOVEit Transfer - Azure AD 集成的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 MOVEit Transfer - Azure AD integration 单一登录](#configure-moveit-transfer---azure-ad-integration-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 MOVEit Transfer - Azure AD integration 测试用户](#create-moveit-transfer---azure-ad-integration-test-user)** - 在 MOVEit Transfer - Azure AD integration 中创建 Britta Simon 的对应用户，并将其关联到其在 Azure AD 中的表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 MOVEit Transfer - Azure AD integration 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“MOVEit Transfer - Azure AD integration”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分，如果有**服务提供程序元数据文件**，请执行以下步骤：

    a. 单击“上传元数据文件”。

    ![上传元数据文件](common/upload-metadata.png)

    b. 单击“文件夹徽标”来选择元数据文件并单击“上传”。

    ![选择元数据文件](common/browse-upload-metadata.png)

    c. 成功上传元数据文件后，“标识符”和“回复 URL”值会自动填充在“基本 SAML 配置”部分：

    ![MOVEit Transfer - Azure AD integration 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    在“登录 URL”文本框中，使用以下模式键入 URL：`https://contoso.com`

    > [!NOTE]
    > “登录 URL”值不是实际值。 请使用实际登录 URL 更新此值。 请联系 [MOVEit Transfer - Azure AD integration 客户端支持团队](https://community.ipswitch.com/s/support)获取这些值。 可以从“服务提供程序元数据 URL”下载“服务提供程序元数据文件”，本教程稍后的“配置 MOVEit Transfer - Azure AD integration 单一登录”部分将介绍此操作。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

4. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

6. 在“设置 MOVEit Transfer - Azure AD integration”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-moveit-transfer---azure-ad-integration-single-sign-on"></a>配置 MOVEit Transfer - Azure AD integration 单一登录

1. 以管理员身份登录到 MOVEit Transfer 租户。

2. 在左侧导航窗格上，单击“设置”。

    ![应用端上的“设置”部分](./media/moveittransfer-tutorial/tutorial_moveittransfer_000.png)

3. 单击位于“安全策略”->“用户身份验证”下的“单一登录”链接。

    ![应用端的安全策略](./media/moveittransfer-tutorial/tutorial_moveittransfer_001.png)

4. 单击元数据 URL 链接以下载元数据文档。

    ![服务提供程序元数据 URL](./media/moveittransfer-tutorial/tutorial_moveittransfer_002.png)
    
   * 验证“entityID”是否与“基本 SAML 配置”部分的“标识符”相匹配。
   * 验证“AssertionConsumerService”位置 URL 是否与“基本 SAML 配置”部分的“回复 URL”相匹配。
    
     ![在应用端配置单一登录](./media/moveittransfer-tutorial/tutorial_moveittransfer_007.png)

5. 单击“添加标识提供程序”按钮以添加新的联合身份提供程序。

    ![添加标识提供者](./media/moveittransfer-tutorial/tutorial_moveittransfer_003.png)

6. 单击“浏览...”以选择从 Azure 门户中下载的元数据文件，并单击“添加标识提供者”以上传所下载的文件。

    ![SAML 标识提供程序](./media/moveittransfer-tutorial/tutorial_moveittransfer_004.png)

7. 在“编辑联合身份提供程序设置...”页上，对于“启用”选择“是”，并单击“保存”。

    ![联合标识提供程序设置](./media/moveittransfer-tutorial/tutorial_moveittransfer_005.png)

8. 在“编辑联合标识提供者用户设置”页上，执行以下操作：
    
    ![编辑联合标识提供程序设置](./media/moveittransfer-tutorial/tutorial_moveittransfer_006.png)
    
    a. 对于“登录名”，选择“SAML NameID”。
    
    b. 选择“其他”作为“全名”并在“属性名称”文本框中输入以下值：`http://schemas.microsoft.com/identity/claims/displayname`。
    
    c. 选择“其他”作为“电子邮件”并在“属性名称”文本框中输入以下值：`https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。
    
    d. 对于“登录时自动创建帐户”，选择“是”。
    
    e. 单击“保存”按钮。

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

在本部分中，通过授予 Britta Simon 访问 MOVEit Transfer - Azure AD 集成的权限，支持其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“MOVEit Transfer - Azure AD integration”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“MOVEit Transfer - Azure AD 集成”。

    ![应用程序列表中的 MOVEit Transfer - Azure AD 集成链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-moveit-transfer---azure-ad-integration-test-user"></a>创建 MOVEit Transfer - Azure AD integration 测试用户

本部分的目的是在 MOVEit Transfer - Azure AD 集成中创建名为 Britta Simon 的用户。 MOVEit Transfer - Azure AD 集成支持在默认情况下启用的实时预配。 此部分不存在任何操作项。 尝试访问 MOVEit Transfer - Azure AD 集成期间，如果尚不存在用户，则会创建一个新用户。

>[!NOTE]
>如果需要手动创建用户，则需联系 [MOVEit Transfer - Azure AD 集成客户端支持团队](https://community.ipswitch.com/s/support)。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中单击“MOVEit Transfer - Azure AD integration”磁贴时，应会自动登录到设置了 SSO 的 MOVEit Transfer - Azure AD integration。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

