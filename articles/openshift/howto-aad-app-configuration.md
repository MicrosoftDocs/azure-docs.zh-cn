---
title: 创建 Azure AD 应用注册和用户的 Azure Red Hat OpenShift |Microsoft Docs
description: 了解如何创建服务主体、 生成客户端机密和身份验证回调 URL，并用于测试 Microsoft Azure Red Hat OpenShift 群集上的应用创建新的 Active Directory 用户。
author: tylermsft
ms.author: twhitney
ms.service: openshift
manager: jeconnoc
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/06/2019
ms.openlocfilehash: de3f3c30848d26ea399bcccc29a6149a149f6a55
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65078515"
---
# <a name="create-an-azure-ad-app-registration-and-user-for-azure-red-hat-openshift"></a>创建 Azure AD 应用注册和用户的 Azure Red Hat OpenShift

Microsoft Azure Red Hat OpenShift 需要有权代表你的群集执行任务。 如果你的组织还没有 Azure Active Directory (Azure AD) 应用注册要用作服务主体，请按照这些说明来创建一个。

本主题创建一个新的说明来结尾将需要访问 Azure Red Hat OpenShift 群集上运行的应用的 Azure AD 用户。

如果你尚未创建 Azure AD 租户，请按照中的说明操作[创建 Azure AD 租户的 Azure Red Hat OpenShift](howto-create-tenant.md)按照说明继续操作之前。

## <a name="create-a-new-app-registration"></a>创建新的应用注册

首先必须在 Azure AD 租户中注册的应用程序想要使用 Azure AD 的功能。 此注册过程涉及到提供有关应用程序的 URL、 用户进行身份验证后用于发送答复的 URL、 用于标识应用程序中，依次类推的 URI 等应用程序的 Azure AD 详细信息。

1. 在[Azure 门户](https://portal.azure.com)，确保在你的用户名称将显示你的租户门户的权限：

    ![右上角中列出的与租户的门户屏幕截图][tenantcallout]如果显示了错误的租户，请单击右上方，您的用户名称，然后单击**切换目录**，然后选择从正确的租户**所有目录**列表。

2. 打开[应用注册边栏选项卡](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)然后单击**新应用程序注册**。
3. 在中**创建**窗格中，输入您的应用程序对象的友好名称。
4. 絋粄**应用程序类型**设置为*Web 应用 /API*。
5. 创建**单一登录 URL**使用以下模式：

    ```
    https://<cluster-name>.<azure-region>.cloudapp.azure.com
    ```

    . . . 其中`<cluster-name>`是要使用 Azure Red Hat OpenShift 群集的名称和`<azure-region>`是[托管 Azure Red Hat OpenShift 群集的 Azure 区域](supported-resources.md#azure-regions)。 例如，如果你的群集名称将成为`contoso`，并你将需要创建在`eastus`区域中，你将输入的完全限定的域名称 (FQDN)**单一登录 URL**是 `https://contoso.eastus.cloudapp.azure.com`

    > [!IMPORTANT]
    > 群集名称必须全部为小写，和 FQDN URL 必须是唯一的。
    > 请确保选择足够不同群集的名称。

    请记下你的群集名称和登录 URL-你需要将它们更高版本为在群集上运行的应用程序访问。 我们会提到群集名称作为`CLUSTER_NAME`，并为此登录 URL`FQDN`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

6. 请确保你**单一登录 URL**值验证与绿色复选标记。 (按 Tab 键为带焦点*单一登录 URL*字段并运行验证检查。)
7. 单击**创建**按钮以创建 Azure AD 应用程序对象。
8. 上**已注册应用**页上显示的向下复制**应用程序 ID**。 我们将把此值作为`APPID`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

    ![应用程序 ID 文本框中的屏幕截图][appidimage]
    
Azure 应用程序对象的详细信息，请参阅[应用程序和 Azure Active Directory 中的服务主体对象](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)。

详细了解如何创建一个新的 Azure AD 应用程序，请参阅[使用 Azure Active Directory v1.0 终结点注册应用](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)。

### <a name="create-a-client-secret"></a>创建客户端机密

现在你已准备好生成您的应用程序到 Azure Active Directory 进行身份验证的客户端机密。

1. 从**已注册应用**页上，单击**设置**若要打开已注册应用程序的设置。
2. 上**设置**窗格中，单击**密钥**。  **密钥**窗格会显示。
![在门户中创建密钥页的屏幕截图][createkeyimage]
3. 提供一个密钥**说明**。
4. 设置的值**Expires**，例如*2 年内*。
5. 单击**保存**和密钥值将出现。
6. 复制密钥值。 我们将把此值作为`SECRET`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

### <a name="create-a-reply-url"></a>创建回复 URL

Azure AD 使用应用程序对象*回复 URL*指定授权应用程序时要使用的回调。 对于 web API 或 web 应用程序，回复 URL 是 Azure AD 将身份验证响应，包括令牌，如果身份验证是成功的位置。

即使最小的不匹配 (尾随斜杠缺失，大小写不同) 将导致令牌颁发操作失败并且你将无法登录。

1. 返回到**设置**窗格中 (可以单击**设置**在门户顶部痕迹导航中)，然后单击**答复 Url**在右侧。  **答复 Url**窗格会显示。
2. 在列表中的第一个回复 URL 将为`FQDN`值中的步骤 6[创建新的应用程序注册](#create-a-new-app-registration)。 对其进行编辑和追加`/oauth2callback/Azure%20AD`。  例如，你的答复 URL 应现在如下所示 `https://mycluster.eastus.cloudapp.azure.com/oauth2callback/Azure%20AD`
3. 单击**保存**保存回复 URL。

## <a name="create-a-new-active-directory-user"></a>创建新的 Active Directory 用户

在要用于登录到 Azure Red Hat OpenShift 群集上运行的应用的 Active Directory 中创建一个新用户。

1. 转到[用户-所有用户](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers)边栏选项卡。
2. 单击“+ 新建用户”。 **用户**窗格会显示。
3. 输入**名称**你想为此用户。
4. 创建**用户名**基于与创建的租户名称`.onmicrosoft.com`附加在后面。 例如，`yourUserName@yourTenantName.onmicrosoft.com`。 记下此用户名。 你将需要它来登录以在群集上使用应用程序。
5. 单击**目录角色**，然后选择**全局管理员**，然后单击**确定**窗格的底部。
6. 中间**用户**窗格中，单击**显示密码**并记录临时密码。 在第一次登录后，将提示您将其重置。
7. 在窗格的底部，单击**创建**以创建用户。

## <a name="resources"></a>资源

* [应用程序和 Azure Active Directory 中的服务主体对象](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)  
* [快速入门：使用 Azure Active Directory v1.0 终结点注册应用](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)  

[appidimage]: ./media/howto-create-tenant/get-app-id.png
[createkeyimage]: ./media/howto-create-tenant/create-key.png
[tenantcallout]: ./media/howto-create-tenant/tenant-callout.png

## <a name="next-steps"></a>后续步骤

如果你已满足所有[Azure Red Hat OpenShift 先决条件](howto-setup-environment.md)，已准备好创建第一个群集 ！

请尝试教程：
> [!div class="nextstepaction"]
> [创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)
