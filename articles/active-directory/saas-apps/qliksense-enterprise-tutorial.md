---
title: 教程：Azure Active Directory 与 Qlik Sense Enterprise 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Qlik Sense Enterprise 之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 8c27e340-2b25-47b6-bf1f-438be4c14f93
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 12/17/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 31df4cb9163e598bfde0c491d8088398c3204119
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407995"
---
# <a name="tutorial-azure-active-directory-integration-with-qlik-sense-enterprise"></a>教程：Azure Active Directory 与 Qlik Sense Enterprise 集成

在本教程中，了解如何将 Qlik Sense Enterprise 与 Azure Active Directory (Azure AD) 集成。
将 Qlik Sense Enterprise 与 Azure AD 集成提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 Qlik Sense Enterprise。
* 可以让用户使用其 Azure AD 帐户自动登录到 Qlik Sense Enterprise（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Qlik Sense Enterprise 的集成，需要具有以下项：

* 一个 Azure AD 订阅。 如果你没有 Azure AD 环境，可以在[此处](https://azure.microsoft.com/pricing/free-trial/)获取一个月的试用版。
* 已启用 Qlik Sense Enterprise 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Qlik Sense Enterprise 支持 SP 发起的 SSO

## <a name="adding-qlik-sense-enterprise-from-the-gallery"></a>从库中添加 Qlik Sense Enterprise

要配置 Qlik Sense Enterprise 与 Azure AD 的集成，需要从库中将 Qlik Sense Enterprise 添加到托管 SaaS 应用列表。

**若要从库中添加 Qlik Sense Enterprise，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中，键入“Qlik Sense Enterprise”，在结果面板中选择“Qlik Sense Enterprise”，然后单击“添加”按钮添加该应用程序。

     ![结果列表中的 Qlik Sense Enterprise](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分中，基于名为“Britta Simon”的测试用户配置和测试 Qlik Sense Enterprise 的 Azure AD 单一登录。
若要运行单一登录，需要在 Azure AD 用户与 Qlik Sense Enterprise 相关用户之间建立链接关系。

若要使用 Qlik Sense Enterprise 配置和测试 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[配置 Qlik Sense Enterprise 单一登录](#configure-qlik-sense-enterprise-single-sign-on)** - 在应用程序端配置单一登录。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Qlik Sense Enterprise 测试用户](#create-qlik-sense-enterprise-test-user)** - 在 Qlik Sense Enterprise 中创建 Britta Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要使用 Qlik Sense Enterprise 配置 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure](https://portal.azure.com/) 门户中，在“Qlik Sense Enterprise”应用程序集成页上，单击“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分中，按照以下步骤操作：

    ![Qlik Sense Enterprise 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    a. 在“登录 URL”文本框中，使用以下模式键入 URL： `https://<Qlik Sense Fully Qualifed Hostname>:4443/azure/hub`

    b. 在“标识符”文本框中，使用以下模式键入 URL：

    | |
    |--|
    | `https://<Qlik Sense Fully Qualifed Hostname>.qlikpoc.com`|
    | `https://<Qlik Sense Fully Qualifed Hostname>.qliksense.com`|
    | |

    c. 在“回复 URL”文本框中，使用以下模式键入 URL：

    `https://<Qlik Sense Fully Qualifed Hostname>:4443/samlauthn/`

    > [!NOTE]
    > 这些不是实际值。 请将这些值更新为实际登录 RUL、标识符和回复 URL（本教程后面部分将进行介绍）或联系 [Qlik Sense Enterprise 客户端支持团队](https://www.qlik.com/us/services/support)来获取这些值。

5. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

### <a name="configure-qlik-sense-enterprise-single-sign-on"></a>配置 Qlik Sense Enterprise 单一登录

1. 准备联合元数据 XML 文件，以便将其上传到 Qlik Sense 服务器。

    > [!NOTE]
    > 将 IdP 元数据上传到 Qlik Sense 服务器之前，需要编辑该文件以删除信息，从而确保在 Azure AD 和 Qlik Sense 服务器之间操作正确。

    ![QlikSense][qs24]

    a. 在文本编辑器中打开从 Azure 门户下载的 FederationMetaData.xml 文件。

    b. 搜索值“RoleDescriptor”。  会出现四个条目（两对开始和结束元素标记）。

    c. 从文件中删除 RoleDescriptor 标记和标记之间的所有信息。

    d. 保存文件，将该文件放在便于取用处，以便稍后在本文档中使用。

2. 作为可以创建虚拟代理配置的用户，导航到 Qlik Sense Qlik 管理控制台 (QMC)。

3. 在 QMC 中，单击“虚拟代理”菜单项。

    ![QlikSense][qs6]

4. 在屏幕的底部，单击“新建”按钮。

    ![QlikSense][qs7]

5. 将显示虚拟代理编辑屏幕。  屏幕右侧是显示配置选项的菜单。

    ![QlikSense][qs9]

6. 选中“标识”菜单选项后，输入 Azure 虚拟代理配置的标识信息。

    ![QlikSense][qs8]  

    a. “说明”字段是虚拟代理配置的友好名称。  输入说明的值。

    b. “前缀”字段标识虚拟代理终结点，用于通过 Azure AD 单一登录连接到 Qlik Sense。  输入此虚拟代理服务器的唯一前缀名称。

    c. “会话非活动超时(分钟)”是使用此虚拟代理进行连接产生的超时。

    d. “会话 Cookie 标头名称”是用于存储用户在成功进行身份验证后接收到的 Qlik Sense 会话的会话标识符的 Cookie 名称。  此名称必须唯一。

7. 单击“身份验证”菜单选项，使其可见。  出现身份验证屏幕。

    ![QlikSense][qs10]

    a. “匿名访问模式”下拉列表决定匿名用户是否可以通过虚拟代理访问 Qlik Sense。  默认选项为非匿名用户。

    b. “身份验证方法”下拉列表决定了虚拟代理要使用的身份验证方案。  从下拉列表中选择“SAML”。  会出现更多选项。

    c. 在“SAML 主机 URI 字段”中，输入用户通过此 SAML 虚拟代理访问 Qlik Sense 时将输入的主机名。  主机名是 Qlik Sense 服务器的 URI。

    d. 在“SAML 实体 ID”中，输入“SAML 主机 URI”字段中的相同值。

    e. “SAML IdP 元数据”是之前在“从 Azure AD 配置编辑联合身份验证元数据”部分中编辑的文件。  **上传 IdP 元数据之前，需要编辑该文件**以删除信息，从而确保在 Azure AD 和 Qlik Sense 服务器之间操作正确。  **如果文件尚未编辑，请参考以上说明进行操作。**  如果文件已编辑，单击“浏览”按钮并选择编辑过的元数据文件，将其上传到虚拟代理配置。

    f. 针对 SAML 属性（该属性表示 Azure AD 要发送到 Qlik Sense 服务器的“UserID”），输入属性名称或架构引用。  可从 Azure 应用屏幕后期配置中获取架构引用信息。  若要使用 name 属性，请输入 `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`。

    g. 输入“用户目录”的值，当用户通过 Azure AD 对 Qlik Sense 服务器进行身份验证时，该值将附加到用户。  硬编码值必须用**方括号 []** 包围。  若要使用 Azure AD SAML 断言中发送的属性，请在此文本框中输入属性名称，**不带**方括号。

    h. **SAML 签名算法**设置服务提供者（此案例中为 Qlik Sense 服务器）证书签名，以实现虚拟代理配置。  如果 Qlik Sense 服务器使用通过 Microsoft 增强 RSA 和 AES 加密提供程序生成的受信任证书，则将 SAML 签名算法更改为 **SHA 256**。

    i. SAML 属性映射部分允许将其他属性（例如，组）发送到 Qlik Sense，以在安全规则中使用。

8. 单击“负载均衡”菜单选项，使其可见。  出现负载均衡屏幕。

    ![QlikSense][qs11]

9. 单击“添加新服务器”节点按钮，选择一个或多个引擎节点，Qlik Sense 将向其发送会话以实现负载均衡，并单击“添加”按钮。

    ![QlikSense][qs12]

10. 单击“高级”菜单选项，使其可见。 出现高级屏幕。

    ![QlikSense][qs13]

    “主机允许列表”标识连接到 Qlik Sense 服务器时接受的主机名。  **输入连接到 Qlik Sense 服务器时用户将指定的主机名。** 主机名是与 SAML 主机 URI 相同的值，但没有 https://。

11. 单击“应用”按钮。

    ![QlikSense][qs14]

12. 单击“确定”以接受警告消息，该警告消息指出将重启链接到虚拟代理的代理。

    ![QlikSense][qs15]

13. 屏幕右侧出现“关联项”菜单。  单击“代理”菜单选项。

    ![QlikSense][qs16]

14. 出现代理屏幕。  单击底部的“链接”按钮，将代理链接到虚拟代理。

    ![QlikSense][qs17]

15. 选择用于支持此虚拟代理连接的代理节点，单击“链接”按钮。  链接后，该代理会显示在关联代理下。

    ![QlikSense][qs18]
  
    ![QlikSense][qs19]

16. 约五到十秒后显示“刷新 QMC”消息。  单击“刷新 QMC”按钮。

    ![QlikSense][qs20]

17. 在 QMC 刷新时，单击“虚拟代理”菜单项。 屏幕上的表中将列出新的 SAML 虚拟代理条目。  单击虚拟代理条目。

    ![QlikSense][qs51]

18. 将激活屏幕底部的“下载 SP 元数据”按钮。  单击“下载 SP 元数据”按钮，将元数据保存到文件。

    ![QlikSense][qs52]

19. 打开 SP 元数据文件。  观察“entityID”条目和“AssertionConsumerService”条目。  这些值等效于Azure AD 应用程序配置中的**标识符**、**登录 URL** 和**回复 URL**。 将这些值粘贴到 Azure AD 应用程序配置中的“Qlik Sense Enterprise 域和 URL”部分中，如果它们不匹配，则应当在 Azure AD 应用配置向导中替换它们。

    ![QlikSense][qs53]

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

在本部分中，通过授予 Britta Simon 访问 Qlik Sense Enterprise 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”和“Qlik Sense Enterprise”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中，键入并选择 Qlik Sense Enterprise。

    ![应用程序列表中的 Qlik Sense Enterprise 链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”对话框中，选择“用户”列表中的 Britta Simon，然后单击屏幕底部的“选择”按钮。

6. 如果你在 SAML 断言中需要任何角色值，请在“选择角色”对话框中从列表中为用户选择合适的角色，然后单击屏幕底部的“选择”按钮。

7. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="create-qlik-sense-enterprise-test-user"></a>创建 Qlik Sense Enterprise 测试用户

在本部分中，会在 Qlik Sense Enterprise 中创建一个名为“Britta Simon”的用户。 请与  [Qlik Sense Enterprise 支持团队](https://www.qlik.com/us/services/support)协作，在 Qlik Sense Enterprise 平台上添加用户。 使用单一登录前，必须先创建并激活用户。

### <a name="test-single-sign-on"></a>测试单一登录 

在本部分中，使用访问面板测试 Azure AD 单一登录配置。

单击访问面板中的 Qlik Sense Enterprise 磁贴时，应当会自动登录到你为其设置了 SSO 的 Qlik Sense Enterprise。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

<!--Image references-->

[qs6]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_06.png
[qs7]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_07.png
[qs8]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_08.png
[qs9]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_09.png
[qs10]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_10.png
[qs11]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_11.png
[qs12]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_12.png
[qs13]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_13.png
[qs14]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_14.png
[qs15]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_15.png
[qs16]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_16.png
[qs17]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_17.png
[qs18]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_18.png
[qs19]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_19.png
[qs20]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_20.png
[qs24]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_24.png
[qs51]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_51.png
[qs52]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_52.png
[qs53]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_53.png

