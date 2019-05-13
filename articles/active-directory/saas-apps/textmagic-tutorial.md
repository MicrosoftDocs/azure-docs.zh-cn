---
title: 教程：Azure Active Directory 与 TextMagic 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 TextMagic 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 3e5b49d2-7096-46bc-a9ce-90e09177ba28
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: c919d02a72565f42c125e6d123b48e2cedaea508
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407982"
---
# <a name="tutorial-azure-active-directory-integration-with-textmagic"></a>教程：Azure Active Directory 与 TextMagic 的集成

本教程介绍了如何将 TextMagic 与 Azure Active Directory (Azure AD) 进行集成。
将 TextMagic 与 Azure AD 集成可提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 TextMagic。
* 可让用户使用其 Azure AD 帐户自动登录到 TextMagic（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 TextMagic 的集成，需要以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 TextMagic 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* TextMagic 支持 IDP 发起的 SSO
* TextMagic 支持“恰时”用户预配

## <a name="adding-textmagic-from-the-gallery"></a>从库中添加 TextMagic

若要配置 TextMagic 与 Azure AD 的集成，需要从库中将 TextMagic 添加到托管 SaaS 应用列表。

**若要从库中添加 TextMagic，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“TextMagic”，在结果面板中选择“TextMagic”，然后单击“添加”按钮添加应用程序。

     ![结果列表中的 TextMagic](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于一个名为“Britta Simon”的测试用户配置和测试 TextMagic 的 Azure AD 单一登录。
若要使单一登录有效，需要在 Azure AD 用户与 TextMagic 相关用户之间建立链接关系。

若要配置和测试 TextMagic 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 TextMagic 单一登录](#configure-textmagic-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 TextMagic 测试用户](#create-textmagic-test-user)** - 在 TextMagic 中创建 Britta Simon 的对应用户，并将其链接到用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 TextMagic 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的 **TextMagic** 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![TextMagic 域和 URL 单一登录信息](common/idp-identifier.png)

    在“标识符”文本框中，键入一个 URL：`https://my.textmagic.com/saml/metadata`

5. TextMagic 应用程序需要特定格式的 SAML 断言，这要求向 SAML 令牌属性配置添加自定义属性映射。 以下屏幕截图显示了默认属性的列表，其中的 **nameidentifier** 通过 **user.userprincipalname** 进行映射。 TextMagic 应用程序要求通过 **user.mail** 对 **nameidentifier** 进行映射，因此需单击“编辑”图标对属性映射进行编辑，然后更改属性映射。

    ![image](common/edit-attribute.png)

6. 除了上述属性，TextMagic 应用程序还要求在 SAML 响应中传递回更多的属性。 在“用户属性”对话框的“用户声明”部分执行以下步骤，以便添加 SAML 令牌属性，如下表所示：

    | 名称 |   源属性| 命名空间  |
    | --------------- | --------------- | --------------- |
    | company | user.companyname | https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | firstName               | user.givenname |  https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | lastName            | user.surname |  https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | phone               | user.telephonenumber |  https://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | | |

    a. 单击“添加新声明”以打开“管理用户声明”对话框。

    ![图像](common/new-save-attribute.png)

    ![图像](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。

    c. 在“命名空间”文本框中，键入为该行显示的命名空间值。

    d. 选择“源”作为“属性”。

    e. 在“源属性”列表中，键入为该行显示的属性值。

    f. 单击“确定”

    g. 单击“ **保存**”。

7. 在“使用 SAML 设置单一登录”页上，在“SAML 签名证书”部分中，单击“下载”以根据要求从给定的选项下载**证书(Base64)** 并将其保存在计算机上。

    ![证书下载链接](common/certificatebase64.png)

8. 在“设置 TextMagic”部分中，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-textmagic-single-sign-on"></a>配置 TextMagic 单一登录

1. 在另一个 Web 浏览器窗口中，以管理员身份登录到 TextMagic 公司站点。

2. 在用户名下选择“帐户设置”。

    ![TextMagic 配置](./media/textmagic-tutorial/config1.png)

3. 单击“单一登录(SSO)”选项卡并填写以下字段：  

    ![TextMagic 配置](./media/textmagic-tutorial/config2.png)

    a. 在“标识提供者实体 ID:”文本框中，粘贴从 Azure 门户复制的“Azure AD 标识符”值 ****  ****。

    b. 在“标识提供者 SSO URL:”文本框中，粘贴从 Azure 门户复制的“登录 URL”值 ****  ****。

    c. 在“标识提供者 SLO URL:”文本框中，粘贴从 Azure 门户复制的“注销 URL”值 ****  ****。

    d. 在记事本中打开从 Azure 门户下载的 base-64 编码证书，将其内容复制到剪贴板，然后粘贴到“公用 x509 证书:”文本框 **** 。

    e. 单击“ **保存**”。

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

在本部分中，通过授予 Britta Simon 访问 TextMagic 的权限，允许她使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“TextMagic”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“TextMagic”。

    ![应用程序列表中的 TextMagic 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-textmagic-test-user"></a>创建 TextMagic 测试用户

应用程序支持实时用户预配，且进行身份验证后，会在应用程序中自动创建用户。 你需要在首次登录时填写一次该信息以在系统中激活子帐户。
此部分不存在任何操作项。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 TextMagic 磁贴时，应当会自动登录到为其设置了 SSO 的 TextMagic。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)