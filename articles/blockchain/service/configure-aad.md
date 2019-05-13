---
title: 如何配置 Azure Active Directory 访问权限
description: 如何使用 Azure Active Directory 访问配置 Azure 区块链服务
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: seal
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: 616e342f1d52179c40c225c5dafc9de13ce85e06
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028210"
---
# <a name="how-to-configure-azure-active-directory-access"></a>如何配置 Azure Active Directory 访问权限

在本文中，您将学习如何授予访问权限，并连接到 Azure 区块链服务节点上使用 Azure Active Directory (Azure AD) 用户、 组或应用程序 Id。

Azure AD 提供基于云的标识管理，并允许您在整个企业和访问应用程序在 Azure 中使用单个标识。 Azure 区块链服务与 Azure AD 集成，并提供了好处，如 ID 联合身份验证、 单一登录和多因素身份验证。

## <a name="prerequisites"></a>必备组件

* [创建使用 Azure 门户的区块链成员](create-member.md)

## <a name="grant-access"></a>授予访问权限

您可以授予成员级别和节点级别的访问权限。 授予成员级别的访问权限又将授予成员下的所有节点访问权限。

### <a name="grant-member-level-access"></a>授予成员级别的访问权限

若要授予成员级别的访问权限。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 导航到**访问控制 (IAM) > 添加 > 添加角色分配**。
1. 选择**区块链成员节点访问 （预览）** 角色并添加你想要授予访问权限的 Azure AD ID 对象。 Azure AD ID 对象可以是：

    | Azure AD 对象 | 示例 |
    |-----------------|---------|
    | Azure AD 用户   | `frank@contoso.onmicrosoft.com` |
    | Azure AD 组  | `sales@contoso.onmicrosoft.com` |
    | 应用程序 ID  | `13925ab1-4161-4534-8d18-812f5ca1ab1e` |

    ![添加角色分配](./media/configure-aad/add-role-assignment.png)

1. 选择“保存”。

### <a name="grant-node-level-access"></a>授予节点级别的访问权限

1. 可以通过导航到节点安全性授予节点级别的访问权限，并单击你想要授予访问权限的节点名称。
1. 选择区块链成员节点访问 （预览） 角色并添加你想要授予访问权限的 Azure AD ID 对象。 

## <a name="connect-using-azure-blockchain-connector"></a>使用 Azure 区块链连接器进行连接

下载或克隆[从 GitHub 的 Azure 区块链连接器](https://github.com/Microsoft/azure-blockchain-connector/)。

```bash
git clone https://github.com/Microsoft/azure-blockchain-connector.git
```

本快速入门主题中，请按照**自述文件**若要生成的源代码从连接器。

### <a name="connect-using-an-azure-ad-user-account"></a>使用 Azure AD 用户帐户进行连接

1. 运行以下命令以使用 Azure AD 用户帐户进行身份验证。 替换\<myAADDirectory\>与 Azure AD 域。 例如，`yourdomain.onmicrosoft.com`。

    ```
    connector.exe -remote <myMemberName>.blockchain.azure.com:3200 -method aadauthcode -tenant-id <myAADDirectory> 
    ```

1. Azure AD 会提示你输入凭据。
1. 使用你的用户名和密码登录。
1. 成功进行身份验证，您的本地代理服务器连接到你区块链的节点。 现在可以将附加 Geth 客户端与本地终结点。

    ```bash
    geth attach http://127.0.0.1:3100
    ```

### <a name="connect-using-an-application-id"></a>使用应用程序 ID 进行连接

许多应用程序而不 Azure AD 用户帐户使用应用程序 ID 与 Azure AD 进行身份验证。

若要连接到你使用应用程序 ID 的节点，替换**aadauthcode**与**aadclient**。

```
connector.exe -remote <myBlockchainEndpoint>  -method aadclient -client-id <myClientID> -client-secret "<myClientSecret>" -tenant-id <myAADDirectory>
```

| 参数 | 描述 |
|-----------|-------------|
| tenant-id | Azure AD 域，例如， `yourdomain.onmicrosoft.com`
| client-id | Azure AD 中注册的应用程序的客户端 ID
| client-secret | Azure AD 中注册的应用程序的客户端机密

有关如何在 Azure AD 中注册应用程序的详细信息，请参阅[如何：使用门户创建可访问资源的 Azure AD 应用程序和服务主体](../../active-directory/develop/howto-create-service-principal-portal.md)

### <a name="connect-a-mobile-device-or-text-browser"></a>将移动设备或文本浏览器连接

对于移动设备或 Azure AD 身份验证弹出窗口中显示不能基于文本的浏览器中，Azure AD 将生成一次性密码。 可以复制密码，并继续进行在另一个环境中的 Azure AD 身份验证。

若要生成代码的更多信息，请替换**aadauthcode**与**aaddevice**。 替换\<myAADDirectory\>与 Azure AD 域。 例如，`yourdomain.onmicrosoft.com`。

```
connector.exe -remote <myBlockchainEndpoint>  -method aaddevice -tenant-id <myAADDirectory>
```

## <a name="next-steps"></a>后续步骤

有关 Azure 区块链服务中的数据安全性的详细信息，请参阅：

> [!div class="nextstepaction"]
> [Azure 区块链服务安全性](data-security.md)