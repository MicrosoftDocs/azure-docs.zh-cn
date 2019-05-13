---
title: 教程：Azure Active Directory 与 Alibaba Cloud Service (Role-based SSO) 的集成 | Microsoft Docs
description: 了解如何在 Azure Active Directory 与 Alibaba Cloud Service (Role-based SSO)（阿里云服务（基于角色的 SSO））之间配置单一登录。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 3667841e-acfc-4490-acf5-80d9ca3e71e8
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8de2f7cb90e004673c59282a8023d55df364220a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65140862"
---
# <a name="tutorial-azure-active-directory-integration-with-alibaba-cloud-service-role-based-sso"></a>教程：Azure Active Directory 与 Alibaba Cloud Service (Role-based SSO) 的集成

本教程介绍如何将 Alibaba Cloud Service (Role-based SSO) 与 Azure Active Directory (Azure AD) 集成。
将 Alibaba Cloud Service (Role-based SSO) 与 Azure AD 集成可提供以下优势：

* 可以在 Azure AD 中控制谁有权访问 Alibaba Cloud Service (Role-based SSO)。
* 可让用户使用其 Azure AD 帐户自动登录到 Alibaba Cloud Service (Role-based SSO)（单一登录）。
* 可在中心位置（即 Azure 门户）管理帐户。

如果要了解有关 SaaS 应用与 Azure AD 集成的更多详细信息，请参阅 [Azure Active Directory 的应用程序访问与单一登录是什么](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。
如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要配置 Azure AD 与 Alibaba Cloud Service (Role-based SSO) 的集成，需要准备好以下各项：

* 一个 Azure AD 订阅。 如果没有 Azure AD 环境，可以获取一个[免费帐户](https://azure.microsoft.com/free/)
* 已启用 Alibaba Cloud Service (Role-based SSO) 单一登录的订阅

## <a name="scenario-description"></a>方案描述

本教程会在测试环境中配置和测试 Azure AD 单一登录。

* Alibaba Cloud Service (Role-based SSO) 支持 **IDP** 发起的 SSO

## <a name="adding-alibaba-cloud-service-role-based-sso-from-the-gallery"></a>从库中添加 Alibaba Cloud Service (Role-based SSO)

若要配置 Alibaba Cloud Service (Role-based SSO) 与 Azure AD 的集成，需要从库中将 Alibaba Cloud Service (Role-based SSO) 添加到托管 SaaS 应用列表。

**若要从库中添加 Alibaba Cloud Service (Role-based SSO)，请执行以下步骤：**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中，单击“Azure Active Directory”图标。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用”，并选择“所有应用”选项。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序，请单击对话框顶部的“新建应用程序”按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中键入 **Alibaba Cloud Service (Role-based SSO)**，在结果面板中选择“Alibaba Cloud Service (Role-based SSO)”，然后单击“添加”按钮添加该应用程序。

    ![结果列表中的“Alibaba Cloud Service (Role-based SSO)”](common/search-new-app.png)

5. 在“Alibaba Cloud Service (Role-based SSO)”页上，单击左侧导航窗格中的“属性”，然后复制“对象 ID”并将其保存在计算机上，供以后使用。

    ![属性配置](./media/alibaba-cloud-service-role-based-sso-tutorial/Properties.png)
    
## <a name="configure-and-test-azure-ad-single-sign-on"></a>配置和测试 Azure AD 单一登录

在本部分，我们基于名为 **Britta Simon** 的测试用户来配置并测试 Alibaba Cloud Service (Role-based SSO) 的 Azure AD 单一登录。
若要正常使用单一登录，需要在 Azure AD 用户与 Alibaba Cloud Service (Role-based SSO) 相关用户之间建立链接关系。

若要配置并测试 Alibaba Cloud Service (Role-based SSO) 的 Azure AD 单一登录，需要完成以下构建基块：

1. **[配置 Azure AD 单一登录](#configure-azure-ad-single-sign-on)** - 使用户能够使用此功能。
2. **[在 Alibaba Cloud Service 中配置基于角色的单一登录](#configure-role-based-single-sign-on-in-alibaba-cloud-service)** - 使用户能够使用此功能。
2. **[配置 Alibaba Cloud Service (Role-based SSO) 单一登录](#configure-alibaba-cloud-service-role-based-sso-single-sign-on)** - 在应用程序端配置单一登录设置。
3. **[创建 Azure AD 测试用户](#create-an-azure-ad-test-user)** - 使用 Britta Simon 测试 Azure AD 单一登录。
4. **[分配 Azure AD 测试用户](#assign-the-azure-ad-test-user)** - 使 Britta Simon 能够使用 Azure AD 单一登录。
5. **[创建 Alibaba Cloud Service (Role-based SSO) 测试用户](#create-alibaba-cloud-service-role-based-sso-test-user)** - 在 Alibaba Cloud Service (Role-based SSO) 中创建 Britta Simon 的对应用户，并将其关联到其在 Azure AD 中的表示形式。
6. **[测试单一登录](#test-single-sign-on)** - 验证配置是否正常工作。

### <a name="configure-azure-ad-single-sign-on"></a>配置 Azure AD 单一登录

在本部分中，将在 Azure 门户中启用 Azure AD 单一登录。

若要配置 Alibaba Cloud Service (Role-based SSO) 的 Azure AD 单一登录，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com/)中的“Alibaba Cloud Service (Role-based SSO)”应用程序集成页上，选择“单一登录”。

    ![配置单一登录链接](common/select-sso.png)

2. 在**选择单一登录方法**对话框中，选择 **SAML/WS-Fed**模式以启用单一登录。

    ![单一登录选择模式](common/select-saml-option.png)

3. 在“使用 SAML 设置单一登录”页上，单击“编辑”图标以打开“基本 SAML 配置”对话框。

    ![编辑基本 SAML 配置](common/edit-urls.png)

4. 在“基本 SAML 配置”部分，如果有**服务提供程序元数据文件**，请执行以下步骤：

    >[!NOTE]
    >可以从此 [URL](https://signin.alibabacloud.com/saml-role/sp-metadata.xml) 获取服务提供程序元数据

    a. 单击“上传元数据文件”。

    ![image](common/upload-metadata.png)

    b. 单击“文件夹徽标”来选择元数据文件并单击“上传”。

    ![image](common/browse-upload-metadata.png)

    c. 成功上传元数据文件后，“标识符”和“回复 URL”值会自动填充在“Alibaba Cloud Service (Role-based SSO)”部分的文本框中：

    ![image](common/idp-intiated.png)

    > [!Note]
    > 如果“标识符”和“回复 URL”值未自动填充，请根据要求手动填充这些值。

5. Alibaba Cloud Service (Role-based SSO) 应用程序需要特定格式的 SAML 断言，因此，需要在 SAML 令牌属性配置中添加自定义属性映射。 以下屏幕截图显示了默认属性的列表。 单击“编辑”图标打开“用户属性”对话框。

    ![image](common/edit-attribute.png)

6. 除上述属性以外，Alibaba Cloud Service (Role-based SSO) 应用程序还要求在 SAML 响应中传回其他几个属性。 在“用户属性”对话框的“用户声明”部分执行以下步骤，以便添加 SAML 令牌属性，如下表所示：

    | 名称 | 命名空间 | 源属性|
    | ---------------| ------------| --------------- |
    | 角色 | https://www.aliyun.com/SAML-Role/Attribute | user.assignedroles |
    | RoleSessionName | https://www.aliyun.com/SAML-Role/Attribute | user.userprincipalname |

    > [!NOTE]
    > 若要了解如何在 Azure AD 中配置**角色**，请单击[此处](https://docs.microsoft.com/azure/active-directory/develop/active-directory-enterprise-app-role-management)

    a. 单击“添加新声明”以打开“管理用户声明”对话框。

    ![图像](common/new-save-attribute.png)

    ![图像](common/new-attribute-details.png)

    b. 在“名称”文本框中，键入为该行显示的属性名称。

    c. 将“命名空间”留空。

    d. 选择“源”作为“属性”。

    e. 在“源属性”列表中，键入为该行显示的属性值。

    f. 单击“ **保存**”。

7. 在“使用 SAML 设置单一登录”页的“SAML 签名证书”部分，单击“下载”以根据要求下载从给定选项提供的“联合元数据 XML”并将其保存在计算机上。

    ![证书下载链接](common/metadataxml.png)

8. 在“设置 Alibaba Cloud Service (Role-based SSO)”部分，根据要求复制相应的 URL。

    ![复制配置 URL](common/copy-configuration-urls.png)

    a. 登录 URL

    b. Azure AD 标识符

    c. 注销 URL

### <a name="configure-role-based-single-sign-on-in-alibaba-cloud-service"></a>在 Alibaba Cloud Service 中配置基于角色的单一登录

1. 使用 Account1 登录到 Alibaba Cloud [RAM 控制台](https://account.alibabacloud.com/login/login.htm?oauth_callback=https%3A%2F%2Fram.console.aliyun.com%2F%3Fspm%3Da2c63.p38356.879954.8.7d904e167h6Yg9)。

2. 在左侧导航窗格中选择“SSO”。

3. 在“基于角色的 SSO”选项卡上，单击“创建 IdP”。

4. 在显示的页面上的“IdP 名称”字段中输入 `AAD`，在“说明”字段中输入说明，单击“上传”以上传先前下载的联合元数据文件，然后单击“确定”。

5. 成功创建 IdP 后，单击“创建 RAM 角色”。

6. 在“RAM 角色名称”字段中输入 `AADrole`，从“选择 IdP”下拉列表中选择 `AAD`，然后单击“确定”。

    >[!NOTE]
    >可根据需要向该角色授予权限。 创建 IdP 和相应的角色后，我们建议保存 IdP 和角色的 ARN，供以后使用。 可以在 IdP 信息页和角色信息页上获取 ARN。

7. 将 Alibaba Cloud RAM 角色 (AADrole) 与 Azure AD 用户 (u2) 相关联：若要将 RAM 角色与 Azure AD 用户相关联，必须执行以下步骤在 Azure AD 中创建一个角色：

    a. 登录到 [Azure AD Graph 资源管理器](https://developer.microsoft.com/graph/graph-explorer?spm=a2c63.p38356.879954.9.7d904e167h6Yg9)。

    b. 单击“修改权限”以获取创建角色所需的权限。

    ![Graph 配置](./media/alibaba-cloud-service-role-based-sso-tutorial/graph01.png)

    c. 从列表中选择以下权限，然后单击“修改权限”，如下图所示。

    ![Graph 配置](./media/alibaba-cloud-service-role-based-sso-tutorial/graph02.png)

    >[!NOTE]
    >授予权限后，再次登录到 Graph 资源管理器。

    d. 在“Graph 资源管理器”页上，从第一个下拉列表中选择“GET”，从第二个下拉列表中选择“beta”。 在下拉列表旁边的字段中输入 `https://graph.microsoft.com/beta/servicePrincipals`，然后单击“运行查询”。

    ![Graph 配置](./media/alibaba-cloud-service-role-based-sso-tutorial/graph03.png)

    >[!NOTE]
    >如果使用多个目录，可以在查询字段中输入 `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`。

    e. 在“响应预览”部分，从“服务主体”中提取 appRoles 属性，供以后使用。

    ![Graph 配置](./media/alibaba-cloud-service-role-based-sso-tutorial/graph05.png)

    >[!NOTE]
    >在查询字段中输入 `https://graph.microsoft.com/beta/servicePrincipals/<objectID>` 可以找到 appRoles 属性。 请注意，`objectID` 是从 Azure AD 的“属性”页复制的对象 ID。

    f. 返回 Graph 资源管理器，将方法从 **GET** 更改为 **PATCH**，将以下内容粘贴到“请求正文”部分，然后单击“运行查询”：
    ```
    { 
    "appRoles": [
        { 
        "allowedMemberTypes":[
            "User"
        ],
        "description": "msiam_access",
        "displayName": "msiam_access",
        "id": "41be2db8-48d9-4277-8e86-f6d22d35****",
        "isEnabled": true,
        "origin": "Application",
        "value": null
        },
        { "allowedMemberTypes": [
            "User"
        ],
        "description": "Admin,AzureADProd",
        "displayName": "Admin,AzureADProd",
        "id": "68adae10-8b6b-47e6-9142-6476078cdbce",
        "isEnabled": true,
        "origin": "ServicePrincipal",
        "value": "acs:ram::187125022722****:role/aadrole,acs:ram::187125022722****:saml-provider/AAD"
        }
    ]
    }
    ```
    > [!NOTE]
    > `value` 是在 RAM 控制台中创建的 IdP 和角色的 ARN。 在此处可根据需要添加多个角色。 Azure AD 会在 SAML 响应中将这些角色的值作为声明值发送。 但是，对于修补操作，只能在 `msiam_access` 部分的后面添加新角色。 若要顺利完成创建过程，我们建议使用 ID 生成器（例如 GUID 生成器）实时生成 ID。

    g. 使用所需的角色修补“服务主体”后，请遵循本教程的“分配 Azure AD 测试用户”部分所述的步骤，将角色附加到 Azure AD 用户 (u2)。

### <a name="configure-alibaba-cloud-service-role-based-sso-single-sign-on"></a>配置 Alibaba Cloud Service (Role-based SSO) 单一登录

若要在 **Alibaba Cloud Service (Role-based SSO)** 端配置单一登录，需要将下载的“联合元数据 XML”以及从 Azure 门户复制的相应 URL 发送给 [Alibaba Cloud Service (Role-based SSO) 支持团队](https://www.aliyun.com/service/)。 他们会对此进行设置，使两端的 SAML SSO 连接均正确设置。

### <a name="create-an-azure-ad-test-user"></a>创建 Azure AD 测试用户 

本部分的目的是在 Azure 门户中创建名为 Britta Simon 的测试用户。

1. 在 Azure 门户的左侧窗格中，依次选择“Azure Active Directory”、“用户”和“所有用户”。

    ![“用户和组”以及“所有用户”链接](common/users.png)

2. 选择屏幕顶部的“新建用户”。

    ![“新建用户”按钮](common/new-user.png)

3. 在“用户属性”中，按照以下步骤操作。

    ![“用户”对话框](common/user-properties.png)

    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，并单击“添加引用”。 在“名称”字段中，输入 BrittaSimon。
  
    b. 在“用户名”字段中键入 `brittasimon@yourcompanydomain.extension`。 例如： BrittaSimon@contoso.com

    c. 选中“显示密码”复选框，然后记下“密码”框中显示的值。

    d. 单击“创建”。

### <a name="assign-the-azure-ad-test-user"></a>分配 Azure AD 测试用户

在本部分，我们通过授予 Britta Simon 访问 Alibaba Cloud Service (Role-based SSO) 的权限，使其能够使用 Azure 单一登录。

1. 在 Azure 门户中，依次选择“企业应用程序”、“所有应用程序”、“Alibaba Cloud Service (Role-based SSO)”。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在“应用程序”列表中，选择“Alibaba Cloud Service (Role-based SSO)”。

    ![“应用程序”列表中的“Alibaba Cloud Service (Role-based SSO)”链接](common/all-applications.png)

3. 在左侧菜单中，选择“用户和组”。

    ![“用户和组”链接](common/users-groups-blade.png)

4. 单击“添加用户”按钮，然后在“添加分配”对话框中选择“用户和组”。

    ![“添加分配”窗格](common/add-assign-user.png)

5. 在“用户和组”选项卡上，从用户列表中选择“u2”并单击“选择”。 然后单击“分配”。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test01.png)

6. 查看分配的角色并测试 Alibaba Cloud Service (Role-based SSO)。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test02.png)

    >[!NOTE]
    >分配用户 (u2) 后，创建的角色会自动附加到该用户。 如果创建了多个角色，则需要将相应的角色附加到用户。 若要在多个 Alibaba Cloud 帐户中实现 Azure AD 的基于角色的 SSO，请重复前述步骤。

### <a name="create-alibaba-cloud-service-role-based-sso-test-user"></a>创建 Alibaba Cloud Service (Role-based SSO) 测试用户

在本部分，我们将在 Alibaba Cloud Service (Role-based SSO) 中创建名为 Britta Simon 的用户。 在 [Alibaba Cloud Service (Role-based SSO) 支持团队](https://www.aliyun.com/service/)的配合下，将用户添加到 Alibaba Cloud Service (Role-based SSO) 平台。 使用单一登录前，必须先创建并激活用户。

### <a name="test-single-sign-on"></a>测试单一登录 

完成前面所述的配置后，执行以下步骤测试 Alibaba Cloud Service (Role-based SSO)：

1. 在 Azure 门户中转到“Alibaba Cloud Service (Role-based SSO)”页，选择“单一登录”，然后单击“测试”。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test03.png)

2. 单击“作为当前用户登录”。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test04.png)

3. 在帐户选择页上选择“u2”。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test05.png)

4. 此时会显示以下页面，指出基于角色的 SSO 成功。

    ![测试配置](./media/alibaba-cloud-service-role-based-sso-tutorial/test06.png)

## <a name="additional-resources"></a>其他资源

- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory 的应用程序访问与单一登录是什么？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什么是 Azure Active Directory 中的条件访问？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

