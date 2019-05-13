---
title: 配置应用程序来公开 Web API - Microsoft 标识平台
description: 了解如何将应用程序配置为公开新的权限/范围和角色，使该应用程序可供客户端应用程序使用。
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1148fb221e82963ea39ece3e18598e38d633238d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413958"
---
# <a name="quickstart-configure-an-application-to-expose-web-apis"></a>快速入门：配置应用程序来公开 Web API

可以开发一个 Web API，并通过公开[权限/范围](developer-glossary.md#scopes)和[角色](developer-glossary.md#roles)，使其可供客户端应用程序使用。 可以像提供其他 Microsoft Web API（包括图形 API 和 Office 365 API）一样提供正确配置的 Web API。

本快速入门介绍如何将应用程序配置为公开新的范围，使其可供客户端应用程序使用。

## <a name="prerequisites"></a>先决条件

若要开始，请确保满足下列先决条件：

* 了解支持的[权限和许可](v2-permissions-and-consent.md)，这是在生成其他用户或应用程序需要使用的应用程序时必须理解的。
* 拥有一个其中已注册了应用程序的租户。
  * 如果尚未注册应用，请[了解如何将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)。

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>登录到 Azure 门户，并选择应用

在配置应用之前，请执行以下步骤：

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务，然后选择“应用注册”。
1. 找到并选择要配置的应用程序。 选择应用以后，会看到应用程序的“概览”页或主注册页。
1. 选择要使用哪个方法、UI 或应用程序清单来公开新的范围：
    * [通过 UI 公开新的范围](#expose-a-new-scope-through-the-ui)
    * [通过应用程序清单公开新的范围或角色](#expose-a-new-scope-or-role-through-the-application-manifest)

## <a name="expose-a-new-scope-through-the-ui"></a>通过 UI 公开新的范围

[![通过 UI 公开 API](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png#lightbox)

若要通过 UI 公开新的范围，请执行以下操作：

1. 在应用的“概览”页中，选择“公开 API”部分。

1. 选择“添加范围”。

1. 如果尚未设置“应用程序 ID URI”，则会看到一个提示你输入它的提示。 输入应用程序 ID URI 或使用已提供的，然后选择“保存并继续”。

1. 出现“添加范围”页时，输入范围的信息：

    | 字段 | 说明 |
    |-------|-------------|
    | **范围名称** | 为范围输入一个有意义的名称。<br><br>例如，`Employees.Read.All`。 |
    | **谁可以许可** | 选择是让用户许可此范围，还是让管理员许可。 若要获得更高特权，请选择“仅管理员”。 |
    | **管理员许可显示名称** | 为范围输入管理员会看到的有意义说明。<br><br>例如： `Read-only access to Employee records` |
    | **管理员许可说明** | 为范围输入管理员会看到的有意义说明。<br><br>例如： `Allow the application to have read-only access to all Employee data.` |

    如果用户可以许可你的范围，另请添加以下字段的值：

    | 字段 | 说明 |
    |-------|-------------|
    | **用户许可显示名称** | 为范围输入一个用户会看到的有意义名称。<br><br>例如： `Read-only access to your Employee records` |
    | **用户许可说明** | 为范围输入用户会看到的有意义说明。<br><br>例如： `Allow the application to have read-only access to your Employee data.` |

1. 完成后，设置“状态”并选择“添加范围”。

1. 按步骤[验证 Web API 是否已公开给其他应用程序](#verify-the-web-api-is-exposed-to-other-applications)。

## <a name="expose-a-new-scope-or-role-through-the-application-manifest"></a>通过应用程序清单公开新的范围或角色

[![使用清单中的 oauth2Permissions 集合公开新的范围](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png#lightbox)

若要通过应用程序清单公开新的范围，请执行以下操作：

1. 在应用的“概览”页中，选择“清单”部分。 此时会打开一个基于 Web 的清单编辑器，可在其中**编辑**门户中的清单。 （可选）可以选择“下载”并在本地编辑清单，然后使用“上传”将清单重新应用到应用程序。
    
    以下示例介绍通过将以下 JSON 元素添加到 `oauth2Permissions` 集合，在资源/API 中公开一个名为 `Employees.Read.All` 的新范围。

      ```json
      {
        "adminConsentDescription": "Allow the application to have read-only access to all Employee data.",
        "adminConsentDisplayName": "Read-only access to Employee records",
        "id": "2b351394-d7a7-4a84-841e-08a6a17e4cb8",
        "isEnabled": true,
        "type": "User",
        "userConsentDescription": "Allow the application to have read-only access to your Employee data.",
        "userConsentDisplayName": "Read-only access to your Employee records",
        "value": "Employees.Read.All"
      }
      ```

   > [!NOTE]
   > 必须通过编程方式或 GUID 生成工具（例如 [guidgen](https://msdn.microsoft.com/library/ms241442%28v=vs.80%29.aspx)）来生成 `id` 值。 `id` 表示 Web API 公开的范围的唯一标识符。 为客户端配置访问 Web API 的适当权限后，Azure AD 将为它颁发 OAuth 2.0 访问令牌。 当客户端调用 Web API 时，会出示该访问令牌，其中的范围 (scp) 声明设置为客户端应用程序注册中请求的权限。
   >
   > 以后可以根据需要公开其他范围。 请考虑 Web API 可能要公开与各种不同功能关联的多个范围。 在运行时，资源可以通过评估所收到的 OAuth 2.0 访问令牌中的范围 (`scp`) 声明，来控制对 Web API 的访问。

1. 完成后，单击“保存”。 现在，Web API 已配置为可供目录中的其他应用程序使用。
1. 按步骤[验证 Web API 是否已公开给其他应用程序](#verify-the-web-api-is-exposed-to-other-applications)。

## <a name="verify-the-web-api-is-exposed-to-other-applications"></a>验证 Web API 是否已公开给其他应用程序

1. 返回到 Azure AD 租户，选择“应用注册”，找到并选择要配置的客户端应用程序。
1. 重复“配置客户端应用程序以访问 Web API”中概述的步骤。
1. 执行到“选择 API”这一步时，请选择资源。 此时会看到可供客户端权限请求使用的新范围。

## <a name="more-on-the-application-manifest"></a>有关应用程序清单的更多信息

应用程序清单充当了用于更新应用程序实体的机制，它定义了 Azure AD 应用程序的标识配置的所有属性。 有关应用程序实体及其架构的详细信息，请参阅[图形 API 应用程序实体文档](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity)。 此文包含有关用于指定 API 权限的应用程序实体成员的完整参考信息，包括：  

* appRoles 成员，用于定义 Web API 的[应用程序权限](developer-glossary.md#permissions)的 [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) 实体集合。
* oauth2Permissions 成员，用于定义 Web API 的[委托权限](developer-glossary.md#permissions)的 [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) 实体集合。

有关应用程序清单概念的一般详细信息，请参阅[了解 Azure Active Directory 应用程序清单](reference-app-manifest.md)。

## <a name="next-steps"></a>后续步骤

了解下述其他相关的应用管理快速入门：

* [将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)
* [将客户端应用程序配置为访问 Web API](quickstart-configure-app-access-web-apis.md)
* [修改应用程序支持的帐户](quickstart-modify-supported-accounts.md)
* [删除注册到 Microsoft 标识平台的应用程序](quickstart-remove-app.md)

了解有关表示已注册应用程序和它们之间的关系的两个 Azure AD 对象的详细信息，请参阅[应用程序对象和服务主体对象](app-objects-and-service-principals.md)。

深入了解使用 Azure Active Directory 开发应用程序时应使用的品牌准则，请参阅[应用程序的品牌准则](howto-add-branding-in-azure-ad-apps.md)。
