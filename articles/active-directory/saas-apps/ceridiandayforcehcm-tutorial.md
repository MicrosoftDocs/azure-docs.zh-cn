---
title: 教程：Azure Active Directory 与 Ceridian Dayforce HCM 集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 和 Ceridian Dayforce HCM 之间配置单一登录。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/27/2021
ms.author: jeedes
ms.openlocfilehash: 4468340dbeeeb67b736d9fd2d227d9c7b2ecbeb6
ms.sourcegitcommit: eb546f78c31dfa65937b3a1be134fb5f153447d6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/02/2021
ms.locfileid: "99427712"
---
# <a name="tutorial-azure-active-directory-integration-with-ceridian-dayforce-hcm"></a>教程：Azure Active Directory 与 Ceridian Dayforce HCM 集成

本教程介绍如何将 Ceridian Dayforce HCM 与 Azure Active Directory (Azure AD) 集成。 将 Ceridian Dayforce HCM 与 Azure AD 集成后，可以执行以下操作：

* 可在 Azure AD 中控制谁有权访问 Ceridian Dayforce HCM。
* 可以让用户使用其 Azure AD 帐户自动登录到 Ceridian Dayforce HCM。
* 在一个中心位置（Azure 门户）管理帐户。

## <a name="prerequisites"></a>先决条件

若要开始操作，需备齐以下项目：

* 一个 Azure AD 订阅。 如果没有订阅，可以获取一个[免费帐户](https://azure.microsoft.com/free/)。
* 已启用 Ceridian Dayforce HCM 单一登录 (SSO) 的订阅。

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Ceridian Dayforce HCM 支持 SP 发起的 SSO 

## <a name="add-ceridian-dayforce-hcm-from-the-gallery"></a>从库中添加 Ceridian Dayforce HCM

要配置 Ceridian Dayforce HCM 与 Azure AD 的集成，需要从库中将 Ceridian Dayforce HCM 添加到托管 SaaS 应用列表。

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 Azure 门户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  。
1. 导航到“企业应用程序”，选择“所有应用程序”   。
1. 若要添加新的应用程序，请选择“新建应用程序”  。
1. 在“从库中添加”部分的搜索框中，键入“Ceridian Dayforce HCM” 。
1. 在结果面板中选择“Ceridian Dayforce HCM”，然后添加该应用。 在该应用添加到租户时等待几秒钟。

## <a name="configure-and-test-azure-ad-sso-for-ceridian-dayforce-hcm"></a>配置并测试 Ceridian Dayforce HCM 的 Azure AD SSO

使用名为 B.Simon 的测试用户配置并测试 Ceridian Dayforce HCM 的 Azure AD SSO。 若要使 SSO 正常运行，需要在 Azure AD 用户与 Ceridian Dayforce HCM 中的相关用户之间建立链接关系。

若要配置并测试 Ceridian Dayforce HCM 的 Azure AD SSO，请执行以下步骤：

1. **[配置 Azure AD SSO](#configure-azure-ad-sso)** - 使用户能够使用此功能。
    1. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 B. Simon 测试 Azure AD 单一登录。
    1. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 B. Simon 能够使用 Azure AD 单一登录。
1. **[配置 Ceridian Dayforce HCM SSO](#configure-ceridian-dayforce-hcm-sso)** - 在应用程序端配置单一登录设置。
    1. **[创建 Ceridian Dayforce HCM 测试用户](#create-ceridian-dayforce-hcm-test-user)** - 在 Ceridian Dayforce HCM 中创建 B.Simon 的对应用户，并将其链接到该用户的 Azure AD 表示形式。
1. **[测试 SSO](#test-sso)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-sso"></a>配置 Azure AD SSO 

按照下列步骤在 Azure 门户中启用 Azure AD SSO。

1. 在 Azure 门户中的 Ceridian Dayforce HCM 应用程序集成页上，找到“管理”部分，然后选择“单一登录”  。
1. 在“选择单一登录方法”页上选择“SAML” 。
1. 在“设置 SAML 单一登录”页面上，单击“基本 SAML 配置”旁边的铅笔图标以编辑设置 。

   ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”  部分中，按照以下步骤操作：

    ![Ceridian Dayforce HCM 域和 URL 单一登录信息](common/sp-identifier-reply.png)

    a. 在“登录 URL”  文本框中，键入用户用于登录 Ceridian Dayforce HCM 应用程序的 URL。

    | 环境 | 代码 |
    | :-- | :-- |
    | 生产 | `https://sso.dayforcehcm.com/<DayforcehcmNamespace>` |
    | 测试 | `https://ssotest.dayforcehcm.com/<DayforcehcmNamespace>` |

    b. 在“标识符”文本框中，使用以下模式键入 URL：

    | 环境 | 代码 |
    | :-- | :-- |
    | 生产 | `https://ncpingfederate.dayforcehcm.com/sp` |
    | 测试 | `https://fs-test.dayforcehcm.com/sp` |

    c. 在“回复 URL”文本框中，键入 Azure AD 用来发布响应的 URL。 

    | 环境 | 代码 |
    | :-- | :-- |
    | 生产 | `https://ncpingfederate.dayforcehcm.com/sp/ACS.saml2` |
    | 测试 | `https://fs-test.dayforcehcm.com/sp/ACS.saml2` |

    > [!NOTE]
    > 这些不是实际值。 请使用实际登录 URL、标识符和回复 URL 更新这些值。 请联系 [Ceridian Dayforce HCM 客户端支持团队](https://www.ceridian.com/support)获取这些值。 还可以参考 Azure 门户中的“基本 SAML 配置”  部分中显示的模式。

5. Ceridian Dayforce HCM 应用程序需要采用特定格式的 SAML 断言。 请为此应用程序配置以下声明。 可以在应用程序集成页的“用户属性”部分管理这些属性的值。  在“使用 SAML 设置单一登录”  页上，单击“编辑”  按钮以打开“用户属性”  对话框。

    ![屏幕截图显示“用户属性”，并且已选择“编辑”图标。](common/edit-attribute.png)

6. 在“用户属性”对话框的“用户声明”部分中，按上图所示配置 SAML 令牌属性，并执行以下步骤：

    | 名称 | 源属性|
    | ---------| --------- |
    | name  | user.extensionattribute2 |

    a. 单击“添加新声明”  以打开“管理用户声明”  对话框。

    ![屏幕截图显示“用户声明”以及“添加新声明”选项。](common/new-save-attribute.png)

    ![屏幕截图显示“管理用户声明”对话框，可在其中输入所述的值。](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。 

    c. 将“命名空间”留空  。

    d. 选择“源”作为“属性”  。

    e. 从“源属性”列表中，选择要用于实现的用户属性  。 例如，如果想要使用 EmployeeID 作为唯一用户标识符并且已在 ExtensionAttribute2 中存储属性值，则选择 user.extensionattribute2。

    f. 单击“确定” 

    g. 单击“ **保存**”。

7. 在“使用 SAML 设置单一登录”  页的“SAML 签名证书”  部分，单击“下载”  以根据要求下载从给定选项提供的元数据 XML 并将其保存在计算机上。 

    ![证书下载链接](common/metadataxml.png)

8. 在“设置 Ceridian Dayforce HCM”部分，根据要求复制相应 URL  。

    ![复制配置 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户

在本部分，我们将在 Azure 门户中创建名为 B.Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”  。
1. 选择屏幕顶部的“新建用户”。
1. 在“用户”属性中执行以下步骤：
   1. 在“名称”字段中，输入 `B.Simon`。  
   1. 在“用户名”字段中输入 username@companydomain.extension。 例如，`B.Simon@contoso.com` 。
   1. 选中“显示密码”复选框，然后记下“密码”框中显示的值。
   1. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

本部分将通过授予 B.Simon 访问 Ceridian Dayforce HCM 的权限，允许其使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”。 
1. 在应用程序列表中，选择“Ceridian Dayforce HCM”  。
1. 在应用的概述页中，找到“管理”部分，选择“用户和组” 。
1. 选择“添加用户”，然后在“添加分配”对话框中选择“用户和组”。
1. 在“用户和组”对话框中，从“用户”列表中选择“B.Simon”，然后单击屏幕底部的“选择”按钮。
1. 如果你希望将某角色分配给用户，可以从“选择角色”下拉列表中选择该角色。 如果尚未为此应用设置任何角色，你将看到选择了“默认访问权限”角色。
1. 在“添加分配”对话框中，单击“分配”按钮。

### <a name="configure-ceridian-dayforce-hcm-sso"></a>配置 Ceridian Dayforce HCM SSO

若要在 Ceridian Dayforce HCM 端配置单一登录，需要将下载的元数据 XML 以及从 Azure 门户复制的相应 URL 发送给 [Ceridian Dayforce HCM 支持团队](https://www.ceridian.com/support)。 他们会对此进行设置，使两端的 SAML SSO 连接均正确设置。

### <a name="create-ceridian-dayforce-hcm-test-user"></a>创建 Ceridian Dayforce HCM 测试用户

在本部分中，将在 Ceridian Dayforce HCM 中创建一个名为“Britta Simon”的用户。 请与 [Ceridian Dayforce HCM 支持团队](https://www.ceridian.com/support)协作，将用户添加到 Ceridian Dayforce HCM 平台。 使用单一登录前，必须先创建并激活用户。

### <a name="test-sso"></a>测试 SSO 

在本部分，你将使用以下选项测试 Azure AD 单一登录配置。 

* 在 Azure 门户中单击“测试此应用程序”。 这会重定向到 Ceridian Dayforce HCM 登录 URL，你可以从此处启动登录流。 

* 直接转到 Ceridian Dayforce HCM 登录 URL，并从此处启动登录流。

* 你可使用 Microsoft 的“我的应用”。 单击“我的应用”中的“Ceridian Dayforce HCM”磁贴时，这将重定向到 Ceridian Dayforce HCM“登录 URL”。 有关“我的应用”的详细信息，请参阅[“我的应用”简介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="next-steps"></a>后续步骤

配置 Ceridian Dayforce HCM 后，可以强制实施会话控制，实时防止组织的敏感数据外泄和渗透。 会话控制从条件访问扩展而来。 [了解如何通过 Microsoft Cloud App Security 强制实施会话控制](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)。
