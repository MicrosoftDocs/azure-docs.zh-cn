---
title: 教程：Azure Active Directory 与 E Sales Manager Remix 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 E Sales Manager Remix 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 89b5022c-0d5b-4103-9877-ddd32b6e1c02
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5f2a3bb666e40f1aafd921d70d93133ca9139b38
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406957"
---
# <a name="integrate-azure-active-directory-with-e-sales-manager-remix"></a>将 Azure Active Directory 与 E Sales Manager Remix 集成

本教程介绍如何将 Azure Active Directory (Azure AD) 与 E Sales Manager Remix 集成。

将 Azure AD 与 E Sales Manager Remix 集成可提供以下优势：

- 可以在 Azure AD 中控制谁有权访问 E Sales Manager Remix。
- 可以让用户使用其 Azure AD 帐户自动登录到 E Sales Manager Remix（单一登录，简称 SSO）。
- 可在一个中心位置（即 Azure 门户）管理帐户。

有关 SaaS 应用与 Azure AD 集成的详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必备组件

若要配置 Azure AD 与 E Sales Manager Remix 的集成，需要以下各项：

- Azure AD 订阅
- 一个启用了 SSO 的 E Sales Manager Remix 订阅

> [!NOTE]
> 不建议使用生产环境测试本教程中的步骤。

若要测试本教程中的步骤，请遵循以下建议：

- 除非必要，请勿使用生产环境。
- 如果没有 Azure AD 试用环境，可以[获取一个月的试用版](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>方案描述
在本教程中，将在测试环境中测试 Azure AD 单一登录。 

本教程中概述的方案包括两个主要构建基块：

* 从库中添加 E Sales Manager Remix
* 配置和测试 Azure AD 单一登录

## <a name="add-e-sales-manager-remix-from-the-gallery"></a>从库中添加 E Sales Manager Remix
若要配置 E Sales Manager Remix 与 Azure AD 的集成，请执行以下操作，从库中将 E Sales Manager Remix 添加到托管 SaaS 应用列表：

1. 在 [Azure 门户](https://portal.azure.com) 的左窗格中，选择“Azure Active Directory”。 

    ![“Azure Active Directory”按钮][1]

1. 选择“企业应用程序” > “所有应用程序”。

    ![“企业应用程序”窗口][2]
    
1. 若要添加新应用程序，请在窗口顶部选择“新建应用程序”。

    ![“新增应用程序”按钮][3]

1. 在搜索框中，键入 **E Sales Manager Remix**，在结果列表中选择“E Sales Manager Remix”，然后选择“添加”。

    ![结果列表中的 E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于一个名为“Britta Simon”的测试用户配置和测试 E Sales Manager Remix 的 Azure AD 单一登录。

若要运行单一登录，Azure AD 需要知道与 Azure AD 中的用户相对应的 E Sales Manager Remix 用户。 换句话说，必须在 Azure AD 用户与 E Sales Manager Remix 中的相关用户之间建立链接关系。

要配置和测试 E Sales Manager Remix 的 Azure AD 单一登录，请完成下面 5 个部分中的构建基块。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

执行以下操作，在 Azure 门户中启用 Azure AD 单一登录并在 E Sales Manager Remix 应用程序中配置单一登录：

1. 在 Azure 门户中的 **E Sales Manager Remix** 应用程序集成页上，选择“单一登录”。

    ![“单一登录”链接][4]

1. 在“单一登录”窗口中的“单一登录模式”框内，选择“基于 SAML 的登录”。
 
    ![“单一登录”窗口](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_samlbase.png)

1. 在“E Sales Manager Remix 域和 URL”下，执行以下操作：

    ![E Sales Manager Remix 域和 URL 单一登录信息](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_url.png)

    a. 在“登录 URL”文本框中，使用以下格式键入 URL：*https://\<Server-Based-URL>/\<sub-domain>/esales-pc*

    b. 在“标识符”文本框中，使用以下格式键入 URL：*https://\<Server-Based-URL>/\<sub-domain>/*

    c. 记下“标识符”值，以便稍后在本教程中使用。
    
    > [!NOTE] 
    > 上面的值不是实际值。 使用实际登录 URL 和标识符更新这些值。 若要获取这些值，请联系 [E Sales Manager Remix 客户端支持团队](mailto:esupport@softbrain.co.jp)。

1. 在“SAML 签名证书”下，选择“证书(Base64)”，然后将证书文件保存到计算机。

    ![证书 (Base64) 下载链接](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_certificate.png) 

1. 选中“查看和编辑其他所有用户属性”复选框，然后选择“emailaddress”属性。
    
    ![“用户属性”窗口](./media/esalesmanagerremix-tutorial/configure1.png)

    此时会打开“编辑属性”窗口。

1. 复制“Namespace”和“Name”值。 以 *\<Namespace>/\<Name>* 模式生成值，并保存以便稍后在本教程中使用。

    ![“编辑属性”窗口](./media/esalesmanagerremix-tutorial/configure2.png)

1. 在“E Sales Manager Remix 配置”下，选择“配置 E Sales Manager Remix”。

    ![E Sales Manager Remix 配置](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_configure.png) 

    此时会打开“配置登录”窗口。

1. 在“快速参考”部分，复制注销 URL 和 SAML 单一登录服务 URL。

1. 选择“保存”。

    ![点击“保存”按钮](./media/esalesmanagerremix-tutorial/tutorial_general_400.png)

1. 以管理员身份登录到 E Sales Manager Remix 应用程序。

1. 在右上角选择“到管理员菜单”。

    ![“到管理员菜单”命令](./media/esalesmanagerremix-tutorial/configure4.png)

1. 在左侧窗格中，选择“系统设置” > “与外部系统协作”。

    ![“系统设置”和“与外部系统协作”链接](./media/esalesmanagerremix-tutorial/configure5.png)
    
1. 在“与外部系统协作”窗口中，选择“SAML”。

    ![“与外部系统协作”窗口](./media/esalesmanagerremix-tutorial/configure6.png)

1. 在“SAML 身份验证设置”下，执行以下操作：

    ![“SAML 身份验证设置”部分](./media/esalesmanagerremix-tutorial/configure3.png)
    
    a. 选中“电脑版本”复选框。
    
    b. 在“协作项”部分的下拉列表中选择“电子邮件”。

    c. 在“协作项”框中，粘贴先前从 Azure 门户复制的声明值（即：**https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress**）。

    d. 在“颁发者(实体 ID)”框中，粘贴前面从 Azure 门户的”E Sales Manager Remix 域和 URL”部分复制的标识符值。

    e. 若要上传从 Azure 门户下载的证书，请选择“文件选择”。

    f. 在“标识提供者登录 URL”框中，粘贴前面从 Azure 门户复制的 SAML 单一登录服务 URL。

    g. 在“标识提供者注销 URL”框中，粘贴前面从 Azure 门户复制的注销 URL 值。

    h. 选择“设置完成”。

> [!TIP]
> 设置应用时，可以在 [Azure 门户](https://portal.azure.com)中阅读前述教程的简洁版本。 在“Active Directory” > “企业应用程序”部分添加此应用后，请选择“单一登录”选项卡，然后在底部的“配置”部分访问嵌入式文档。 有关嵌入式文档功能的详细信息，请参阅 [Azure AD 嵌入式文档]( https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，我们将执行以下操作，在 Azure 门户中创建一个名为 Britta Simon 的测试用户：

![创建 Azure AD 测试用户][100]

1. 在 Azure 门户的左窗格中，选择“Azure Active Directory”。

    ![“Azure Active Directory”链接](./media/paloaltoadmin-tutorial/create_aaduser_01.png)

1. 若要显示当前用户列表，请选择“用户和组” > “所有用户”。

    ![“用户和组”以及“所有用户”链接](./media/paloaltoadmin-tutorial/create_aaduser_02.png)

1. 在“所有用户”窗口的顶部，选择“添加”。

    ![“添加”按钮](./media/paloaltoadmin-tutorial/create_aaduser_03.png)
    
    此时会打开“用户”窗口。

1. 在“用户”窗口中执行以下操作：

    ![“用户”窗口](./media/paloaltoadmin-tutorial/create_aaduser_04.png)

    a. 在“姓名”框中，键入“BrittaSimon”。

    b. 在“用户名”框中，键入用户 Britta Simon 的电子邮件地址。

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 选择“创建”。
 
### <a name="create-an-e-sales-manager-remix-test-user"></a>创建 E Sales Manager Remix 测试用户

1. 以管理员身份登录到 E Sales Manager Remix 应用程序。

1. 从右上角菜单选择“到管理员菜单”。

    ![E Sales Manager Remix 配置](./media/esalesmanagerremix-tutorial/configure4.png)

1. 选择“公司设置” > “部门和员工维护”，然后选择“注册员工”。

    ![“注册员工”选项卡](./media/esalesmanagerremix-tutorial/user1.png)

1. 在“新员工注册”部分，执行以下操作：
    
    ![“新员工注册”部分](./media/esalesmanagerremix-tutorial/user2.png)

    a. 在“员工姓名”框中，键入用户名（如 **Britta**）。

    b. 填写剩余的必填字段。
    
    c. 如果启用了 SAML，则管理员无法从登录页登录。 选中“管理员登录”复选框，向用户授予管理员登录特权。

    d. 选择“注册”。

1. 将来若要以管理员身份登录，请以具有管理员权限的用户身份登录，然后在右上角选择“到管理员菜单”。

    ![“到管理员菜单”命令](./media/esalesmanagerremix-tutorial/configure4.png)

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分中，通过授予用户 Britta Simon 访问 E Sales Manager Remix 的权限，允许她使用 Azure 单一登录。 为此，请执行以下操作： 

![分配用户角色][200] 

1. 在 Azure 门户中，打开“应用程序”视图，转到“目录”视图，然后选择“企业应用程序” > “所有应用程序”。

    ![“企业应用程序”和“所有应用程序”链接][201] 

1. 在“应用程序”列表中，选择“E Sales Manager Remix”。

    ![“E Sales Manager Remix”链接](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_app.png)  

1. 在左窗格中，选择“用户和组”。

    ![“用户和组”链接][202]

1. 选择“添加”，然后在“添加分配”窗格中选择“用户和组”。

    ![“添加分配”窗格][203]

1. 在“用户和组”窗口中的“用户”列表内，选择“Britta Simon”。

1. 选择“选择”按钮。

1. 在“添加分配”窗口中，选择“分配”。
    
### <a name="test-single-sign-on"></a>测试单一登录

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

在访问面板中选择 E Sales Manager Remix 磁贴时，应会自动登录到 E Sales Manager Remix 应用程序。

有关访问面板的详细信息，请参阅[访问面板简介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他资源

* [有关将 SaaS 应用与 Azure Active Directory 集成的教程列表](tutorial-list.md)
* [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/esalesmanagerremix-tutorial/tutorial_general_01.png
[2]: ./media/esalesmanagerremix-tutorial/tutorial_general_02.png
[3]: ./media/esalesmanagerremix-tutorial/tutorial_general_03.png
[4]: ./media/esalesmanagerremix-tutorial/tutorial_general_04.png

[100]: ./media/esalesmanagerremix-tutorial/tutorial_general_100.png

[200]: ./media/esalesmanagerremix-tutorial/tutorial_general_200.png
[201]: ./media/esalesmanagerremix-tutorial/tutorial_general_201.png
[202]: ./media/esalesmanagerremix-tutorial/tutorial_general_202.png
[203]: ./media/esalesmanagerremix-tutorial/tutorial_general_203.png

