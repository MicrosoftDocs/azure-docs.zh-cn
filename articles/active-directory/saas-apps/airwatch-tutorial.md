---
title: 教程：Azure Active Directory 与 AirWatch 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 AirWatch 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 96a3bb1c-96c6-40dc-8ea0-060b0c2a62e5
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/07/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7ec8b575157dcf2fe8430f554798af62b966c78d
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406701"
---
# <a name="tutorial-azure-active-directory-integration-with-airwatch"></a>教程：Azure Active Directory 与 AirWatch 集成

在本教程中，了解如何将 AirWatch 与 Azure Active Directory (Azure AD) 集成。
将 AirWatch 与 Azure AD 集成提供以下优势：

* 可在 Azure AD 中控制谁有权访问 AirWatch。
* 可以让用户使用其 Azure AD 帐户自动登录到 AirWatch（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 AirWatch 的集成，需要具有以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 AirWatch 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* AirWatch 支持 SP 发起的 SSO

## <a name="adding-airwatch-from-the-gallery"></a>从库添加 AirWatch

若要配置 AirWatch 与 Azure AD 的集成，需要将库中的 AirWatch 添加到托管的 SaaS 应用列表。

若要从库添加 AirWatch，请执行以下步骤：

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“AirWatch”，在结果面板中选择“AirWatch”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 AirWatch](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于一个名为 Britta Simon 的测试用户使用 AirWatch 配置和测试 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 AirWatch 相关用户之间建立链接关系。

若要配置和测试 AirWatch 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 AirWatch 单一登录](#configure-airwatch-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. [创建 AirWatch 测试用户](#create-airwatch-test-user) - 在 AirWatch 中创建 Britta Simon 的对应用户，将其链接到该用户的 Azure AD 表示形式。
5. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 AirWatch 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的 AirWatch 应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![AirWatch 域和 URL 单一登录信息](common/sp-identifier.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL：`https://<subdomain>.awmdm.com/AirWatch/Login?gid=companycode`

    b. 在“标识符(实体 ID)”文本框中，键入值：`AirWatch`

    > [!NOTE]
    > 此值不是真实值。 使用实际登录 URL 更新此值。 若要获取此值，请与 [AirWatch 客户端支持团队](https://www.air-watch.com/company/contact-us/)联系。 还可以参考 Azure 门户中的“基本 SAML 配置”部分中显示的模式。

5. AirWatch 应用程序需要特定格式的 SAML 断言。 请为此应用程序配置以下声明。 可以在应用程序集成页的“用户属性”部分管理这些属性的值。 在“使用 SAML 设置单一登录”页上，单击“编辑”按钮以打开“用户属性”对话框。

    ![image](common/edit-attribute.png)

6. 在“用户属性”对话框的“用户声明”部分中，通过使用“编辑图标”编辑声明或使用“添加新声明”添加声明，按上图所示配置 SAML 令牌属性，并执行以下步骤：

    | 名称 |  源属性|
    |---------------|----------------|
    | UID | user.userprincipalname |
    | | |

    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，并单击“添加引用”。 单击“添加新声明”以打开“管理用户声明”对话框。

    ![图像](common/new-save-attribute.png)

    ![图像](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。

    c. 将“命名空间”留空。

    d. 选择“源”作为“属性”。

    e. 在“源属性”列表中，键入为该行显示的属性值。

    f. 单击“确定”

    g. 单击“ **保存**”。

7. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

8. 在“设置 AirWatch”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-airwatch-single-sign-on"></a>配置 AirWatch 单一登录

1. 在其他 Web 浏览器窗口中，以管理员身份登录 AirWatch 公司站点。

2. 在左侧导航窗格中，单击“帐户”，并单击“管理员”。

   ![管理员](./media/airwatch-tutorial/ic791920.png "管理员")

3. 展开“设置”菜单，并单击“目录服务”。

   ![设置](./media/airwatch-tutorial/ic791921.png "设置")

4. 单击“用户”选项卡，在“基 DN”文本框中键入域名，然后单击“保存”。

   ![用户](./media/airwatch-tutorial/ic791922.png "用户")

5. 单击“服务器”选项卡。

   ![服务器](./media/airwatch-tutorial/ic791923.png "服务器")

6. 执行以下步骤：

    ![上传](./media/airwatch-tutorial/ic791924.png "上传")   

    a. 对于“目录类型”，选择“无”。

    b. 选择“将 SAML 用于身份验证”。

    c. 若要上传下载的证书，请单击“上传”。

7. 在“请求”部分中，执行以下步骤：

    ![请求](./media/airwatch-tutorial/ic791925.png "请求")  

    a. 对于“请求绑定类型”，请选择“POST”。

    b. 在 Azure 门户的“配置 Airwatch 的单一登录”对话框上，复制“登录 URL”值，然后将其粘贴到“标识提供程序单一登录 URL”文本框。

    c. 对于“NameID 格式”，请选择“电子邮件地址”。

    d. 单击“ **保存**”。

8. 再次单击“用户”选项卡。

    ![用户](./media/airwatch-tutorial/ic791926.png "用户")

9. 在“属性”部分中，执行以下步骤：

    ![属性](./media/airwatch-tutorial/ic791927.png "属性")

    a. 在“对象标识符”文本框中，键入 `http://schemas.microsoft.com/identity/claims/objectidentifier`。

    b. 在“用户名”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。

    c. 在“显示名称”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`。

    d. 在“名字”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`。

    e. 在“姓氏”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`。

    f. 在“电子邮件”文本框中，键入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。

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

在本部分中，通过授予 AirWatch 的访问权限，支持 Britta Simon 使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“AirWatch”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，选择“AirWatch”。

    ![应用程序列表中的 AirWatch 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-airwatch-test-user"></a>创建 AirWatch 测试用户

要使 Azure AD 用户能够登录 AirWatch，必须对其进行预配才能使其登录 AirWatch。 就 AirWatch 来说，预配任务需要手动完成。

**若要配置用户设置，请执行以下步骤：**

1. 以管理员身份登录 **AirWatch** 公司站点。

2. 在左侧导航窗格中，单击“帐户”，并单击“用户”。
  
   ![用户](./media/airwatch-tutorial/ic791929.png "用户")

3. 在“用户”菜单中，单击“列表视图”，并依次单击“添加”\>“添加用户”。
  
   ![添加用户](./media/airwatch-tutorial/ic791930.png "添加用户")

4. 在“添加 / 编辑用户”对话框中，执行以下步骤：

   ![添加用户](./media/airwatch-tutorial/ic791931.png "添加用户")

   a. 键入希望在相关文本框中预配的有效 Azure Active Directory 帐户的“用户名”、“密码”、“确认密码”、“名字”、“姓氏”、“电子邮件地址”。

   b. 单击“ **保存**”。

> [!NOTE]
> 可使用其他任何 AirWatch 用户帐户创建工具或 AirWatch 提供的 API 预配 AAD 用户帐户。

### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 AirWatch 磁贴时，应会自动登录到为其设置了 SSO 的 AirWatch。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
