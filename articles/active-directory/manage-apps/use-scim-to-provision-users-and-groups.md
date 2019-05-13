---
title: 在 Azure Active Directory 中使用 SCIM 自动预配应用 | Microsoft Docs
description: Azure Active Directory 可以使用 SCIM 协议规范中定义的接口，自动将用户和组预配到以 Web 服务为前端的任何应用程序或标识存储
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 5/06/2019
ms.author: celested
ms.reviewer: asmalser
ms.custom: aaddev;it-pro;seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: fba0a9bc0886b9487b0c61b6091bd122fe6e370d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191546"
---
# <a name="using-system-for-cross-domain-identity-management-scim-to-automatically-provision-users-and-groups-from-azure-active-directory-to-applications"></a>使用跨域标识管理系统 (SCIM) 将用户和组从 Azure Active Directory 自动预配到应用程序

## <a name="overview"></a>概述

SCIM 是标准化的协议和架构的目标是为驱动器中标识跨多个系统的管理方式更好的一致性。 在应用程序进行用户管理支持 SCIM 终结点，Azure AD 用户预配服务可以发送请求来创建、 修改或删除已分配的用户和组添加到此终结点。 

Azure AD 支持的应用程序的许多[预先集成的自动用户预配](../saas-apps/tutorial-list.md)实现 SCIM，如接收用户的方法更改通知。  除此之外，客户可以连接支持的特定配置文件的应用程序[SCIM 2.0 协议规范](https://tools.ietf.org/html/rfc7644)Azure 门户中使用泛型"非库"集成选项。 

本文的重点是在 Azure AD 作为非库应用其泛型 SCIM 连接器的一部分实现的 SCIM 2.0 的配置文件。 但是，成功测试的应用程序支持 SCIM 的一般的 Azure ad 连接器是将为支持用户预配 Azure AD 库中列出的应用的步骤。 Azure AD 应用程序库中列出你的应用程序的详细信息，请参阅[如何：列出 Azure AD 应用程序库中的应用程序](../develop/howto-app-gallery-listing.md)。
 

>[!IMPORTANT]
>Azure AD SCIM 实现的行为最近于 2018 年 12 月 18 日更新。 有关更改内容的信息，请参阅 [Azure AD 用户预配服务 SCIM 2.0 协议合规性](application-provisioning-config-problem-scim-compatibility.md)。

![][0]
*图 1：从 Azure Active Directory 预配到实现 SCIM 的应用程序或标识存储*

本文分为四个部分：

* **[预配的用户和组添加到第三方应用程序支持 SCIM 2.0](#provisioning-users-and-groups-to-applications-that-support-scim)**  -如果你的组织使用，实现的 SCIM 2.0 Azure AD 配置文件支持，可以开始自动同时运行的第三方应用程序预配和取消预配的用户和组今天。

* **[了解 Azure AD SCIM 实现](#understanding-the-azure-ad-scim-implementation)** -如果你正在构建支持 SCIM 2.0 用户管理 API 的应用程序，此部分详细介绍了如何实现 Azure AD SCIM 客户端，以及如何应对建模SCIM 协议请求处理和响应。
  
* **[构建使用 Microsoft CLI 库的 SCIM 终结点](#building-a-scim-endpoint-using-microsoft-cli-libraries)** -公共语言基础结构 (CLI) 库以及代码示例显示如何开发 SCIM 终结点和转换 SCIM 消息。  

* **[用户和组架构参考](#user-and-group-schema-reference)** -介绍了支持的非库应用程序的 Azure AD SCIM 实现的用户和组架构。 

## <a name="provisioning-users-and-groups-to-applications-that-support-scim"></a>将用户和组预配到支持 SCIM 的应用程序
可以将 azure AD 配置为自动预配分配用户和组添加到实现的特定配置文件的应用程序[SCIM 2.0 协议](https://tools.ietf.org/html/rfc7644)。 配置文件的详细信息记录在[了解 Azure AD SCIM 实现](#understanding-the-azure-ad-scim-implementation)。

请咨询应用程序提供者，或参阅应用程序提供者文档中的说明，以了解是否符合这些要求。

>[!IMPORTANT]
>Azure AD SCIM 实现基于 Azure AD 用户预配服务，它旨在不断地保留在 Azure AD 之间同步用户和目标应用程序，并实现一组非常特定的标准操作。 请务必了解这些行为，若要了解 Azure AD SCIM 客户端的行为。 有关详细信息，请参阅[用户预配过程中发生？](user-provisioning.md#what-happens-during-provisioning)。

### <a name="getting-started"></a>入门
支持本文所述 SCIM 配置文件的应用程序可以使用 Azure AD 应用程序库中的“非库应用程序”功能连接到 Azure Active Directory。 连接后，Azure AD 将每隔 40 分钟运行同步过程，此过程将为分配的用户和组查询应用程序的 SCIM 终结点，并根据分配详细信息创建或修改这些用户和组。

**连接到支持 SCIM 的应用程序：**

1. 登录到[Azure Active Directory 门户](https://aad.portal.azure.com)。 

1. 选择**企业应用程序**在左窗格中。 显示所有已配置应用的列表，包括那些从库添加的应用。

1. 选择 **+ 新应用程序** > **所有** > **非库应用程序**。

1. 输入你的应用程序的名称并选择**添加**若要创建应用对象。 新的应用添加到企业应用程序的列表，并打开到其应用程序管理屏幕。
    
   ![][1]
   *图 2：Azure AD 应用程序库*
    
1. 在应用程序管理屏幕中，选择**预配**左侧面板中。
1. 在“预配模式”菜单中，选择“自动”。
    
   ![][2]
   *图 3：在 Azure 门户中配置预配*
    
1. 在“租户 URL”字段中，输入应用程序的 SCIM 终结点的 URL。 示例： https://api.contoso.com/scim/v2/
1. 如果 SCIM 终结点需要来自非 Azure AD 颁发者的 OAuth 持有者令牌，可将所需的 OAuth 持有者令牌复制到可选的“密钥令牌”字段。 如果此字段留空，Azure AD 包含从每个请求的 Azure AD 颁发的 OAuth 持有者令牌。 将 Azure AD 用作标识提供者的应用可以验证 Azure AD 颁发的此令牌。
1. 选择**测试连接**，使 Azure Active Directory 尝试连接到 SCIM 终结点。 如果该尝试失败，显示错误信息。  

    >[!NOTE]
    >**测试连接**使用随机 GUID 作为在 Azure AD 配置中选择的匹配属性，针对不存在的用户查询 SCIM 终结点。 预期正确响应为“HTTP 200 正常”以及空的 SCIM ListResponse 消息。 

1. 如果尝试连接到应用程序成功，然后选择**保存**保存管理员凭据。
1. 在“映射”部分中有两个可选的属性映射集：一个用于用户对象，一个用于组对象。 分别选择它们，查看从 Azure Active Directory 同步到应用的属性。 选为“匹配”属性的特性用于匹配应用中的用户和组，以执行更新操作。 选择“保存”，提交所有更改。

    >[!NOTE]
    >也可通过禁用“组”映射来选择性禁用组对象的同步。 

1. “设置”下的“作用域”字段定义同步的用户和组。 选择**仅同步分配的用户和组**（推荐） 以仅同步用户和组中分配**用户和组**选项卡。
1. 你的配置完成后，设置**预配状态**到**上**。
1. 选择**保存**以启动 Azure AD 预配服务。 
1. 如果仅同步分配用户和组 （推荐），请务必选择**用户和组**选项卡并分配用户或你想要同步的组。

启动初始同步后，可以选择**审核日志**在左面板中监视进度，它显示了通过预配服务对您的应用程序的所有操作。 若要详细了解如何读取 Azure AD 预配日志，请参阅[有关自动用户帐户预配的报告](check-status-user-account-provisioning.md)。

> [!NOTE]
> 初始同步长，若要执行的时间比更高版本，只要服务正在运行，大约每隔 40 分钟就会进行同步。 

## <a name="understanding-the-azure-ad-scim-implementation"></a>了解 Azure AD SCIM 实现

如果您正在构建支持 SCIM 2.0 用户管理 API 的应用程序，此部分详细介绍了如何实现 Azure AD SCIM 客户端，以及如何应对建模 SCIM 协议请求处理和响应。 在已实现 SCIM 终结点之后, 可以测试它按照上一部分中所述的过程。

内[SCIM 2.0 协议规范](http://www.simplecloud.info/#Specification)，你的应用程序必须满足以下要求：

* 支持创建用户和 （可选） 还组，部分所述[SCIM 协议 3.3](https://tools.ietf.org/html/rfc7644#section-3.3)。  
* 支持修改具有修补程序请求用户或组作为每个[部分的 SCIM 协议 3.5.2](https://tools.ietf.org/html/rfc7644#section-3.5.2)。  
* 检索已知的资源的用户或组之前创建的支持为每个[部分的 SCIM 协议 3.4.1](https://tools.ietf.org/html/rfc7644#section-3.4.1)。  
* 支持查询的用户或组，部分所述[SCIM 协议第 3.4.2](https://tools.ietf.org/html/rfc7644#section-3.4.2)。  默认情况下，用户通过检索其`id`由和查询其`username`并`externalid`，并对组进行查询的`displayName`。  
* 支持查询用户按 ID 和管理器中，根据 SCIM 协议第 3.4.2 部分。  
* 支持查询组按 ID 和成员，根据 SCIM 协议第 3.4.2 部分。  
* 接受单个持有者令牌进行身份验证和授权的 Azure AD 向应用程序。

实现的 SCIM 终结点，以确保与 Azure AD 的兼容性时，请遵循以下常规准则：

* `id` 为所有资源的必需的属性。 每个资源应确保每个资源将返回的响应具有此属性，除了`ListResponse`与零个成员。
* 应始终为响应查询/筛选器请求`ListResponse`。
* 组是可选的但只有如果 SCIM 实现支持 PATCH 请求支持。
* 它不需要的修补程序响应中包括整个资源。
* Microsoft Azure AD 仅使用以下运算符：  
     - `eq`
     - `and`
* 不需要区分大小写的匹配项中 SCIM，在特定的修补程序的结构化元素上`op`操作的值，如中所定义 https://tools.ietf.org/html/rfc7644#section-3.5.2。 Azure AD 发出操作的值作为`Add`， `Replace`，和`Remove`。
* Microsoft Azure AD 发出请求以提取一个随机的用户和组，以确保终结点和凭据有效。 它还会作为的一部分**测试连接**流入[Azure 门户](https://portal.azure.com)。 
* 可以在查询资源的属性应设置为匹配的属性中的应用程序[Azure 门户](https://portal.azure.com)。 有关详细信息，请参阅[自定义用户预配属性映射](https://docs.microsoft.com/azure/active-directory/active-directory-saas-customizing-attribute-mappings)

### <a name="user-provisioning-and-de-provisioning"></a>用户预配和取消预配
如下图所示，Azure Active Directory 将发送到 SCIM 服务以管理应用程序的标识存储中的用户的生命周期的消息。  

![][4]
*图 4：用户预配和取消预配顺序*

### <a name="group-provisioning-and-de-provisioning"></a>组预配和取消预配
组预配和取消预配是可选的。 实现并启用时下, 图显示的消息，Azure AD 会发送到 SCIM 服务以管理应用程序的标识存储中的组的生命周期。  这些消息通过两种方式中的用户有关的消息不同： 

* 若要检索组的请求指定成员属性是要从在对请求响应中提供的任何资源中排除。  
* 确定引用属性是否具有特定值的请求是有关成员属性的请求。  

![][5]
*图 5：组预配和取消预配顺序*

### <a name="scim-protocol-requests-and-responses"></a>SCIM 协议请求和响应
本部分提供示例 SCIM 请求发出的 Azure AD SCIM 客户端和示例的预期的响应。 为获得最佳结果，因此应将应用以处理这些请求以这种格式，并发出预期的响应。

>[!IMPORTANT]
>若要了解 Azure AD 用户预配服务如何以及何时发出如下所述的操作，请参阅[用户预配过程中发生？](user-provisioning.md#what-happens-during-provisioning)。

- [用户操作](#user-operations)
  - [创建用户](#create-user)
    - [请求](#request)
    - [响应](#response)
  - [获取用户](#get-user)
    - [请求](#request-1)
    - [响应](#response-1)
  - [通过查询获取用户](#get-user-by-query)
    - [请求](#request-2)
    - [响应](#response-2)
  - [获取用户查询的零个结果](#get-user-by-query---zero-results)
    - [请求](#request-3)
    - [响应](#response-3)
  - [更新用户多值属性](#update-user-multi-valued-properties)
    - [请求](#request-4)
    - [响应](#response-4)
  - [更新用户单值属性](#update-user-single-valued-properties)
    - [请求](#request-5)
    - [响应](#response-5)
  - [删除用户](#delete-user)
    - [请求](#request-6)
    - [响应](#response-6)
- [组操作](#group-operations)
  - [创建组](#create-group)
    - [请求](#request-7)
    - [响应](#response-7)
  - [获取组](#get-group)
    - [请求](#request-8)
    - [响应](#response-8)
  - [获取组的显示名称](#get-group-by-displayname)
    - [请求](#request-9)
    - [响应](#response-9)
  - [更新组 [非成员属性]](#update-group-non-member-attributes)
    - [请求](#request-10)
    - [响应](#response-10)
  - [更新组 [添加成员。](#update-group-add-members)
    - [请求](#request-11)
    - [响应](#response-11)
  - [更新组 [删除成员]](#update-group-remove-members)
    - [请求](#request-12)
    - [响应](#response-12)
  - [删除组](#delete-group)
    - [请求](#request-13)
    - [响应](#response-13)

### <a name="user-operations"></a>用户操作

* 用户可以通过查询`userName`或`email[type eq "work"]`属性。  

#### <a name="create-user"></a>创建用户

###### <a name="request"></a>请求
*POST/用户*
```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "active": true,
    "emails": [{
        "primary": true,
        "type": "work",
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com"
    }],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName"
    },
    "roles": []
}
```

##### <a name="response"></a>响应
*HTTP/1.1 201 已创建*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "48af03ac28ad4fb88478",
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```


#### <a name="get-user"></a>获取用户

###### <a name="request"></a>请求
*GET /Users/5d48a0a8e9f04aa38008* 

###### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5d48a0a8e9f04aa38008",
    "externalId": "58342554-38d6-4ec8-948c-50044d0a33fd",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_feed3ace-693c-4e5a-82e2-694be1b39934",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_22370c1a-9012-42b2-bf64-86099c2a1c22@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```
#### <a name="get-user-by-query"></a>通过查询获取用户

##### <a name="request"></a>请求
*GET /Users?filter=userName eq "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081"*

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2441309d85324e7793ae",
        "externalId": "7fce0092-d52e-4f76-b727-3955bd72c939",
        "meta": {
            "resourceType": "User",
            "created": "2018-03-27T19:59:26.000Z",
            "lastModified": "2018-03-27T19:59:26.000Z"
            
        },
        "userName": "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081",
        "name": {
            "familyName": "familyName",
            "givenName": "givenName"
        },
        "active": true,
        "emails": [{
            "value": "Test_User_91b67701-697b-46de-b864-bd0bbe4f99c1@testuser.com",
            "type": "work",
            "primary": true
        }]
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="get-user-by-query---zero-results"></a>获取用户查询的零个结果

##### <a name="request"></a>请求
*GET/用户？ 筛选器 = 用户名 eq"不存在用户"*

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 0,
    "Resources": [],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="update-user-multi-valued-properties"></a>更新用户多值属性

##### <a name="request"></a>请求
*修补程序/用户/6764549bef60420686bc HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
            {
            "op": "Replace",
            "path": "emails[type eq \"work\"].value",
            "value": "updatedEmail@microsoft.com"
            },
            {
            "op": "Replace",
            "path": "name.familyName",
            "value": "updatedFamilyName"
            }
    ]
}
```

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "6764549bef60420686bc",
    "externalId": "6c75de36-30fa-4d2d-a196-6bdcdb6b6539",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_fbb9dda4-fcde-4f98-a68b-6c5599e17c27",
    "name": {
        "formatted": "givenName updatedFamilyName",
        "familyName": "updatedFamilyName",
        "givenName": "givenName"
    },
    "active": true,
    "emails": [{
        "value": "updatedEmail@microsoft.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="update-user-single-valued-properties"></a>更新用户单值属性

##### <a name="request"></a>请求
*修补程序/用户/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "userName",
        "value": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com"
    }]
}
```

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5171a35d82074e068ce2",
    "externalId": "aa1eca08-7179-4eeb-a0be-a519f7e5cd1a",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "userName": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_49dc1090-aada-4657-8434-4995c25a00f7@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="delete-user"></a>删除用户

##### <a name="request"></a>请求
*DELETE /Users/5171a35d82074e068ce2 HTTP/1.1*

##### <a name="response"></a>响应
*HTTP/1.1 204 无内容*

### <a name="group-operations"></a>组操作

* 始终应使用空成员列表创建组。
* 可以通过查询组`displayName`属性。
* 更新组 PATCH 请求应产生*HTTP 204 无内容*在响应中。 返回正文，其中含有列表的所有成员，不建议。
* 但并不需要以支持返回组的所有成员。

#### <a name="create-group"></a>创建组

##### <a name="request"></a>请求
*POST /Groups HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group", "http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/2.0/Group"],
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "id": "c4d56c3c-bf3b-4e96-9b64-837018d6060e",
    "displayName": "displayName",
    "members": [],
    "meta": {
        "resourceType": "Group"
    }
}
```

##### <a name="response"></a>响应
*HTTP/1.1 201 已创建*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "927fa2c08dcb4a7fae9e",
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "displayName": "displayName",
    "members": []
}
```

#### <a name="get-group"></a>获取组

##### <a name="request"></a>请求
*GET/组/40734ae655284ad3abcc？ excludedAttributes = 成员 HTTP/1.1*

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "40734ae655284ad3abcc",
    "externalId": "60f1bb27-2e1e-402d-bcc4-ec999564a194",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "displayName": "displayName",
}
```

#### <a name="get-group-by-displayname"></a>获取组的显示名称

##### <a name="request"></a>请求
*GET /Groups？ excludedAttributes = 成员筛选器 = displayName eq"displayName"HTTP/1.1*

##### <a name="response"></a>响应
*HTTP/1.1 200 确定*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "id": "8c601452cc934a9ebef9",
        "externalId": "0db508eb-91e2-46e4-809c-30dcbda0c685",
        "meta": {
            "resourceType": "Group",
            "created": "2018-03-27T22:02:32.000Z",
            "lastModified": "2018-03-27T22:02:32.000Z",
            
        },
        "displayName": "displayName",
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}
```
#### <a name="update-group-non-member-attributes"></a>更新组 [非成员属性]

##### <a name="request"></a>请求
*PATCH /Groups/fa2ce26709934589afc5 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "displayName",
        "value": "1879db59-3bdf-4490-ad68-ab880a269474updatedDisplayName"
    }]
}
```

##### <a name="response"></a>响应
*HTTP/1.1 204 无内容*

### <a name="update-group-add-members"></a>更新组 [添加成员。

##### <a name="request"></a>请求
*修补/组/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Add",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a>响应
*HTTP/1.1 204 无内容*

#### <a name="update-group-remove-members"></a>更新组 [删除成员]

##### <a name="request"></a>请求
*修补/组/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Remove",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a>响应
*HTTP/1.1 204 无内容*

#### <a name="delete-group"></a>删除组

##### <a name="request"></a>请求
*DELETE /Groups/cdb1ce18f65944079d37 HTTP/1.1*

##### <a name="response"></a>响应
*HTTP/1.1 204 无内容*


## <a name="building-a-scim-endpoint-using-microsoft-cli-libraries"></a>构建使用 Microsoft CLI 库的 SCIM 终结点
通过与 Azure Active Directory 中创建的 SCIM web 服务的接口，可以启用自动用户预配为几乎任何应用程序或标识存储。

工作方式如下：

1. Azure AD 提供一个名为 Microsoft.SystemForCrossDomainIdentityManagement，示例如下所述的代码中包含的公共语言基础结构 (CLI) 库。 系统集成商和开发人员可以使用此库来创建和部署可以将 Azure AD 连接到任何应用程序的标识存储的基于 SCIM 的 web 服务终结点。
2. 将在 Web 服务中实现映射，以将标准化用户架构映射到用户架构和应用程序所需的协议。 
3. 终结点 URL 在 Azure AD 中注册为应用程序库中自定义应用程序的一部分。
4. 用户和组在 Azure AD 中分配到此应用程序。 分配后，它们是放入队列，以同步到目标应用程序。 处理队列的同步过程每隔 40 分钟运行一次。

### <a name="code-samples"></a>代码示例
若要简化此过程[代码示例](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master)提供其创建 SCIM web 服务终结点并演示自动预配。 该示例是维护一个文件，与表示用户和组以逗号分隔值的行的提供程序。    

**先决条件**

* Visual Studio 2013 或更高版本
* [用于 .NET 的 Azure SDK](https://azure.microsoft.com/downloads/)
* 支持将 ASP.NET Framework 4.5 用作 SCIM 终结点的 Windows 计算机。 此计算机必须是可从云访问。
* [具有 Azure AD Premium 试用版或许可版的 Azure 订阅](https://azure.microsoft.com/services/active-directory/)

### <a name="getting-started"></a>入门
实现可以接受来自 Azure AD 的预配请求的 SCIM 终结点的最简单方法是构建和部署将预配的用户输出逗号分隔值 (CSV) 文件的代码示例。

#### <a name="to-create-a-sample-scim-endpoint"></a>创建示例 SCIM 终结点

1. 从 [https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master) 下载代码示例包
1. 将包解压缩，并将其放在 Windows 计算机上的某个位置，例如 C:\AzureAD-BYOA-Provisioning-Samples\。
1. 在此文件夹中，启动 Visual Studio 中的 FileProvisioning\Host\FileProvisioningService.csproj 项目。
1. 选择**工具** > **NuGet 包管理器** > **程序包管理器控制台**，并执行以下命令为FileProvisioningService 项目解析解决方案引用：

   ```
    Update-Package -Reinstall
   ```

1. 构建 FileProvisioningService 项目。
1. 在 Windows 中启动命令提示符应用程序（以管理员身分），并使用 cd 命令将目录切换到 \AzureAD-BYOA-Provisioning-Samples\FileProvisioning\Host\bin\Debug 文件夹。
1. 运行以下命令，替换`<ip-address>`与 Windows 计算机的 IP 地址或域名名：

   ```
    FileSvc.exe http://<ip-address>:9000 TargetFile.csv
   ```

1. 在 Windows 下**Windows 设置** > **网络和 Internet 设置**，选择**Windows 防火墙** >  **高级设置**，并创建**入站规则**，允许对端口 9000 的入站的访问。
1. 如果 Windows 计算机位于路由器后面，路由器需要将配置为在 Windows 上运行其端口 9000 向 internet 公开的端口和端口 9000 之间的网络访问转换。 若要访问此终结点在云中的 Azure AD 需要此配置。

#### <a name="to-register-the-sample-scim-endpoint-in-azure-ad"></a>在 Azure AD 中注册示例 SCIM 终结点

1. 登录到[Azure Active Directory 门户](https://aad.portal.azure.com)。 

1. 选择**企业应用程序**在左窗格中。 显示所有已配置应用的列表，包括那些从库添加的应用。

1. 选择 **+ 新应用程序** > **所有** > **非库应用程序**。

1. 输入你的应用程序的名称并选择**添加**若要创建应用对象。 创建的应用程序对象代表要预配和实现单一登录的目标应用，而不仅是 SCIM 终结点。

1. 在应用程序管理屏幕中，选择**预配**左侧面板中。

1. 在“预配模式”菜单中，选择“自动”。
    
   ![][2]
   *图 6：在 Azure 门户中配置预配*
    
1. 在“租户 URL”字段中，输入面向 Internet 的 URL 和 SCIM 终结点的端口。 该条目类似于 http://testmachine.contoso.com:9000 或 http://\<ip-address>:9000/，其中 \<ip-address> 是 Internet 公开 IP 地址。 

1. 如果 SCIM 终结点需要来自非 Azure AD 颁发者的 OAuth 持有者令牌，可将所需的 OAuth 持有者令牌复制到可选的“密钥令牌”字段。 如果此字段留空，Azure AD 会包含从每个请求的 Azure AD 颁发的 OAuth 持有者令牌。 将 Azure AD 用作标识提供程序的应用可以验证 Azure AD 颁发的此令牌。

1. 选择**测试连接**，使 Azure Active Directory 尝试连接到 SCIM 终结点。 如果该尝试失败，显示错误信息。  

    >[!NOTE]
    >**测试连接**使用随机 GUID 作为在 Azure AD 配置中选择的匹配属性，针对不存在的用户查询 SCIM 终结点。 预期正确响应为“HTTP 200 正常”以及空的 SCIM ListResponse 消息

1. 如果尝试连接到应用程序成功，然后选择**保存**保存管理员凭据。

1. 在“映射”部分中有两个可选的属性映射集：一个用于用户对象，一个用于组对象。 分别选择它们，查看从 Azure Active Directory 同步到应用的属性。 选为“匹配”属性的特性用于匹配应用中的用户和组，以执行更新操作。 选择“保存”，提交所有更改。

1. “设置”下的“作用域”字段定义同步的用户或组。 选择 **"仅同步分配的用户和组**（推荐） 以仅同步用户和组中分配**用户和组**选项卡。

1. 你的配置完成后，设置**预配状态**到**上**。

1. 选择**保存**以启动 Azure AD 预配服务。 

1. 如果仅同步分配用户和组 （推荐），请务必选择**用户和组**选项卡并分配用户或你想要同步的组。

启动初始同步后，可以选择**审核日志**在左面板中监视进度，它显示了通过预配服务对您的应用程序的所有操作。 若要详细了解如何读取 Azure AD 预配日志，请参阅[有关自动用户帐户预配的报告](check-status-user-account-provisioning.md)。

验证此示例的最后一步是打开 Windows 计算机上 \AzureAD-BYOA-Provisioning-Samples\ProvisioningAgent\bin\Debug 文件夹中的 TargetFile.csv 文件。 运行预配过程后，此文件会显示所有已分配和预配的用户与组的详细信息。

### <a name="development-libraries"></a>开发库
若要开发自己的符合 SCIM 规范的 Web 服务，请先熟悉 Microsoft 提供的、有助于加速开发过程的以下库： 

- 提供公共语言基础结构 (CLI) 库以配合基于该基础结构的语言，例如 C#。 其中一个库，Microsoft.SystemForCrossDomainIdentityManagement.Service，声明了一个接口 Microsoft.SystemForCrossDomainIdentityManagement.IProvider，如下图中所示。 使用这些库的开发人员将对某个类（一般称为提供程序）实现该接口。 这些库可让开发人员部署的 web 服务符合 SCIM 规范。 可以在 Internet 信息服务或任何可执行的 CLI 程序集内或者托管 web 服务。 请求将转换为对提供程序方法的调用，这些方法由开发者编程，以便对某些标识存储执行操作。
  
   ![][3]
  
- [快速路由处理程序](https://expressjs.com/guide/routing.html)用于分析代表对 node.js Web 服务的调用（由 SCIM 规范定义）的 node.js 请求对象。   

### <a name="building-a-custom-scim-endpoint"></a>构建自定义 SCIM 终结点
使用 CLI 库的开发人员可以其服务托管在任何可执行文件的 CLI 程序集，或 Internet 信息服务中。 以下代码示例用于将服务托管在地址为 http://localhost:9000: 的可执行程序集中： 

    private static void Main(string[] arguments)
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IProvider and 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  

    Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
      new DevelopersMonitor();
    Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
      new DevelopersProvider(arguments[1]);
    Microsoft.SystemForCrossDomainIdentityManagement.Service webService = null;
    try
    {
        webService = new WebService(monitor, provider);
        webService.Start("http://localhost:9000");

        Console.ReadKey(true);
    }
    finally
    {
        if (webService != null)
        {
            webService.Dispose();
            webService = null;
        }
    }
    }

    public class WebService : Microsoft.SystemForCrossDomainIdentityManagement.Service
    {
    private Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor;
    private Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider;

    public WebService(
      Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitoringBehavior, 
      Microsoft.SystemForCrossDomainIdentityManagement.IProvider providerBehavior)
    {
        this.monitor = monitoringBehavior;
        this.provider = providerBehavior;
    }

    public override IMonitor MonitoringBehavior
    {
        get
        {
            return this.monitor;
        }

        set
        {
            this.monitor = value;
        }
    }

    public override IProvider ProviderBehavior
    {
        get
        {
            return this.provider;
        }

        set
        {
            this.provider = value;
        }
    }
    }

此服务必须具有 HTTP 地址，其服务器身份验证证书的根证书颁发机构是下列名称之一： 

* CNNIC
* Comodo
* CyberTrust
* DigiCert
* GeoTrust
* GlobalSign
* Go Daddy
* VeriSign
* WoSign

可以使用网络 shell 实用程序将服务器身份验证证书绑定到 Windows 主机上的某个端口： 

    netsh http add sslcert ipport=0.0.0.0:443 certhash=0000000000003ed9cd0c315bbb6dc1c08da5e6 appid={00112233-4455-6677-8899-AABBCCDDEEFF}  

此处，为 certhash 参数提供的值为证书指纹，为 appid 参数提供的值为任意全局唯一标识符。  

若要将服务托管在 Internet 信息服务，开发人员将程序集的默认命名空间中使用名为 Startup 的类构建的 CLI 代码库程序集。  以下是这种类的示例： 

    public class Startup
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor and  
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  

    Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter starter;

    public Startup()
    {
        Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
          new DevelopersMonitor();
        Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
          new DevelopersProvider();
        this.starter = 
          new Microsoft.SystemForCrossDomainIdentityManagement.WebApplicationStarter(
            provider, 
            monitor);
    }

    public void Configuration(
      Owin.IAppBuilder builder) // Defined in Owin.dll.  
    {
        this.starter.ConfigureApplication(builder);
    }
    }

### <a name="handling-endpoint-authentication"></a>处理终结点身份验证
来自 Azure Active Directory 的请求包括 OAuth 2.0 持有者令牌。   接收请求的任何服务应将颁发者作为 Azure Active Directory 的所需的 Azure Active Directory 租户，以访问 Azure Active Directory Graph web 服务进行身份验证。  在令牌中，颁发者由一个 iss 声明，例如"iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/"。  在此示例中，声明值的基址 https://sts.windows.net、 Azure Active Directory 标识为颁发者，而相对地址段 cbb1a5ac-f33b-45fa-9bf5-f37db0fed422、 是为 Azure Active Directory 租户的唯一标识符其颁发的令牌。  如果颁发的令牌用于访问 Azure Active Directory Graph Web 服务，该服务的标识符 00000002-0000-0000-c000-000000000000 应在令牌的 aud 声明值中。  每个在单个租户中注册应用程序可能会收到相同`iss`SCIM 请求声明。

使用 CLI 库构建 SCIM 服务由 Microsoft 提供的开发人员可以通过执行以下步骤使用 Microsoft.Owin.Security.ActiveDirectory 包的 Azure Active Directory 中的请求进行身份验证： 

1. 在提供程序中，通过每次启动服务时让服务返回要调用的方法来实现 Microsoft.SystemForCrossDomainIdentityManagement.IProvider.StartupBehavior 属性： 

   ```
     public override Action\<Owin.IAppBuilder, System.Web.Http.HttpConfiguration.HttpConfiguration\> StartupBehavior
     {
       get
       {
         return this.OnServiceStartup;
       }
     }

     private void OnServiceStartup(
       Owin.IAppBuilder applicationBuilder,  // Defined in Owin.dll.  
       System.Web.Http.HttpConfiguration configuration)  // Defined in System.Web.Http.dll.  
     {
     }
   ```

2. 将以下代码添加到该方法，以对所有服务的终结点作为确定它们持有 Azure Active Directory 为指定的租户，以访问 Azure AD Graph web 服务颁发的令牌进行身份验证的任何都请求： 

   ```
     private void OnServiceStartup(
       Owin.IAppBuilder applicationBuilder IAppBuilder applicationBuilder, 
       System.Web.Http.HttpConfiguration HttpConfiguration configuration)
     {
       // IFilter is defined in System.Web.Http.dll.  
       System.Web.Http.Filters.IFilter authorizationFilter = 
         new System.Web.Http.AuthorizeAttribute(); // Defined in System.Web.Http.dll.configuration.Filters.Add(authorizationFilter);

       // SystemIdentityModel.Tokens.TokenValidationParameters is defined in    
       // System.IdentityModel.Token.Jwt.dll.
       SystemIdentityModel.Tokens.TokenValidationParameters tokenValidationParameters =     
         new TokenValidationParameters()
         {
           ValidAudience = "00000002-0000-0000-c000-000000000000"
         };

       // WindowsAzureActiveDirectoryBearerAuthenticationOptions is defined in 
       // Microsoft.Owin.Security.ActiveDirectory.dll
       Microsoft.Owin.Security.ActiveDirectory.
       WindowsAzureActiveDirectoryBearerAuthenticationOptions authenticationOptions =
         new WindowsAzureActiveDirectoryBearerAuthenticationOptions()    {
         TokenValidationParameters = tokenValidationParameters,
         Tenant = "03F9FCBC-EA7B-46C2-8466-F81917F3C15E" // Substitute the appropriate tenant’s 
                                                       // identifier for this one.  
       };

       applicationBuilder.UseWindowsAzureActiveDirectoryBearerAuthentication(authenticationOptions);
     }
   ```

### <a name="handling-provisioning-and-deprovisioning-of-users"></a>处理预配和取消预配的用户

1. Azure Active Directory 会在服务中查询是否有某个用户的 externalId 属性值与 Azure AD 中用户的 mailNickname 属性值匹配。 查询表示为此示例中，jyoung 是 Azure Active Directory 中用户的 mailNickname 示例如超文本传输协议 (HTTP) 请求。

    >[!NOTE]
    > 这只是一个示例。 并非所有用户将都可以 mailNickname 属性和用户具有的值可能不是唯一的目录中。 此外，用于匹配 （这在此情况下是 externalId） 的属性是可在中配置[Azure AD 属性映射](customize-application-attributes.md)。

   ````
    GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
    Authorization: Bearer ...
   ````
   如果使用由 Microsoft 提供用于实现 SCIM 服务的 CLI 库生成服务，则会将请求转换为对服务的提供者的查询方法的调用。  以下是该方法的签名： 
   ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Protocol.  

    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource[]> Query(
      Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters parameters, 
      string correlationIdentifier);
   ````
   以下是 Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters 接口的定义： 
   ````
    public interface IQueryParameters: 
      Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
    {
        System.Collections.Generic.IReadOnlyCollection <Microsoft.SystemForCrossDomainIdentityManagement.IFilter> AlternateFilters 
        { get; }
    }

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
    {
      system.Collections.Generic.IReadOnlyCollection<string> ExcludedAttributePaths 
      { get; }
      System.Collections.Generic.IReadOnlyCollection<string> RequestedAttributePaths 
      { get; }
      string SchemaIdentifier 
      { get; }
    }

   ```
     GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
     Authorization: Bearer ...
   ```

   If the service was built using the Common Language Infrastructure libraries provided by Microsoft for implementing SCIM services, then the request is translated into a call to the Query method of the service’s provider.  Here is the signature of that method: 

   ```
     // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
     // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
     // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
     // Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters is defined in 
     // Microsoft.SystemForCrossDomainIdentityManagement.Protocol.  
 
     System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource[]>  Query(
       Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters parameters, 
       string correlationIdentifier);
   ```

   Here is the definition of the Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters interface: 

   ```
     public interface IQueryParameters: 
       Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
     {
         System.Collections.Generic.IReadOnlyCollection  <Microsoft.SystemForCrossDomainIdentityManagement.IFilter> AlternateFilters 
         { get; }
     }

     public interface Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
     {
       system.Collections.Generic.IReadOnlyCollection<string> ExcludedAttributePaths 
       { get; }
       System.Collections.Generic.IReadOnlyCollection<string> RequestedAttributePaths 
       { get; }
       string SchemaIdentifier 
       { get; }
     }

     public interface Microsoft.SystemForCrossDomainIdentityManagement.IFilter
     {
         Microsoft.SystemForCrossDomainIdentityManagement.IFilter AdditionalFilter 
           { get; set; }
         string AttributePath 
           { get; } 
         Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator FilterOperator 
           { get; }
         string ComparisonValue 
           { get; }
     }

     public enum Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator
     {
         Equals
     }
   ```

   In the following sample of a query for a user with a given value for the externalId attribute, values of the arguments passed to the Query method are: 
   * parameters.AlternateFilters.Count: 1
   * parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
   * parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
   * parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"
   * correlationIdentifier: System.Net.Http.HttpRequestMessage.GetOwinEnvironment["owin.RequestId"] 

2. If the response to a query to the web service for a user with an externalId attribute value that matches the mailNickname attribute value of a user doesn't return any users, then Azure Active Directory requests that the service provision a user corresponding to the one in Azure Active Directory.  Here is an example of such a request: 

   ````
    开机自检 https://.../scim/Users HTTP/1.1 授权：持有者...内容类型： 应用程序/scim + json {"架构": ["urn:ietf:params:scim:schemas:core:2.0:user"，"urn: ietf:params:scim:schemas:extension:enterprise:2.0User"]"externalId":"jyoung"，"userName":"jyoung"，"活动": true，"地址": null，   "displayName":"乐趣 Young"、"电子邮件": [{"type":"工作"，"value":"jyoung@Contoso.com"、"主要": true}]，"meta": {"resourceType":"用户"}"name": {"familyName":"年轻"、"givenName":"乐趣"}，"phoneNumbers": preferredLa 为 null，"nguage": null、"title": null、"部门": null，则"manager": null}
   ````
   The CLI libraries provided by Microsoft for implementing SCIM services would translate that request into a call to the Create method of the service’s provider.  The Create method has this signature: 
   ````
    System.Threading.Tasks.Tasks mscorlib.dll 中定义。  
    在中定义 Microsoft.SystemForCrossDomainIdentityManagement.Resource / / Microsoft.SystemForCrossDomainIdentityManagement.Schemas。  

    System.Threading.Tasks.Task < Microsoft.SystemForCrossDomainIdentityManagement.Resource > 创建 （Microsoft.SystemForCrossDomainIdentityManagement.Resource 资源、 字符串 correlationIdentifier）
   ````
   In a request to provision a user, the value of the resource argument is an instance of the Microsoft.SystemForCrossDomainIdentityManagement. Core2EnterpriseUser class, defined in the Microsoft.SystemForCrossDomainIdentityManagement.Schemas library.  If the request to provision the user succeeds, then the implementation of the method is expected to return an instance of the Microsoft.SystemForCrossDomainIdentityManagement. Core2EnterpriseUser class, with the value of the Identifier property set to the unique identifier of the newly provisioned user.  

3. To update a user known to exist in an identity store fronted by an SCIM, Azure Active Directory proceeds by requesting the current state of that user from the service with a request such as: 
   ````
    GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1  Authorization:持有者...
   ````
   In a service built using the CLI libraries provided by Microsoft for implementing SCIM services, the request is translated into a call to the Retrieve method of the service’s provider.  Here is the signature of the Retrieve method: 
   ````
    System.Threading.Tasks.Tasks mscorlib.dll 中定义。  
    Microsoft.SystemForCrossDomainIdentityManagement.Resource 和 / / Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters / / 在 Microsoft.SystemForCrossDomainIdentityManagement.Schemas 中定义。  
    System.Threading.Tasks.Task < Microsoft.SystemForCrossDomainIdentityManagement.Resource > 检索 （Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 参数、 字符串 correlationIdentifier）

    Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 的公共接口：   
        IRetrievalParameters {Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier ResourceIdentifier {get;}} 公共接口 Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier {string 标识符 {get; set;}字符串 Microsoft.SystemForCrossDomainIdentityManagement.SchemaIdentifier {get; set;}}
   ````
   In the example of a request to retrieve the current state of a user, the values of the properties of the object provided as the value of the parameters argument are as follows: 
  
   * Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
   * SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

4. If a reference attribute is to be updated, then Azure Active Directory queries the service to determine whether the current value of the reference attribute in the identity store fronted by the service already matches the value of that attribute in Azure Active Directory. For users, the only attribute of which the current value is queried in this way is the manager attribute. Here is an example of a request to determine whether the manager attribute of a particular user object currently has a certain value: 

   If the service was built using the CLI libraries provided by Microsoft for implementing SCIM services, then the request is translated into a call to the Query method of the service’s provider. The value of the properties of the object provided as the value of the parameters argument are as follows: 
  
   * parameters.AlternateFilters.Count: 2
   * parameters.AlternateFilters.ElementAt(x).AttributePath: "ID"
   * parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
   * parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
   * parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
   * parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
   * parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
   * parameters.RequestedAttributePaths.ElementAt(0): "ID"
   * parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

   Here, the value of the index x can be 0 and the value of the index y can be 1, or the value of x can be 1 and the value of y can be 0, depending on the order of the expressions of the filter query parameter.   

5. Here is an example of a request from Azure Active Directory to an SCIM service to update a user: 
   ````
    PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1  Authorization:持有者...内容类型： 应用程序/scim + json {"架构": ["urn: ietf:params:scim:api:messages:2.0:PatchOp"]，"操作": [{"op":"添加"、"路径":"manager"、"value": [{"$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646"、"value":"2819c223-7f76-453a-919d-413861904646"}]}]}
   ````
   The Microsoft CLI libraries for implementing SCIM services would translate the request into a call to the Update method of the service’s provider. Here is the signature of the Update method: 
   ````
    System.Threading.Tasks.Tasks 和 / / System.Collections.Generic.IReadOnlyCollection<T> / / 在 mscorlib.dll 中定义。  
    Microsoft.SystemForCrossDomainIdentityManagement.IPatch，/ / Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase，/ / Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier，/ /Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation，/ / Microsoft.SystemForCrossDomainIdentityManagement.OperationName，/ / Microsoft.SystemForCrossDomainIdentityManagement.IPath 和 / /Microsoft.SystemForCrossDomainIdentityManagement.OperationValue / / 是所有中定义的 Microsoft.SystemForCrossDomainIdentityManagement.Protocol。 

    System.Threading.Tasks.Task 更新 （Microsoft.SystemForCrossDomainIdentityManagement.IPatch 修补程序、 字符串 correlationIdentifier）;

    公共接口 Microsoft.SystemForCrossDomainIdentityManagement.IPatch {Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase PatchRequest {get; set;}Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier ResourceIdentifier {get; set;}        
    }

    公共类 PatchRequest2:  Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase {公共 System.Collections.Generic.IReadOnlyCollection < Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation > 操作 {get;}


   如果使用 Microsoft 提供的、用于实现 SCIM 服务的公共语言基础结构库构建了服务，则将请求转换为对服务提供者的 Query 方法调用。 作为参数自变量值提供的对像属性值如下： 
  
* parameters.AlternateFilters.Count:2
* parameters.AlternateFilters.ElementAt(x).AttributePath:“ID”
* parameters.AlternateFilters.ElementAt(x).ComparisonOperator:ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(x).ComparisonValue:"54D382A4-2050-4C03-94D1-E769F1D15682"
* parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
* parameters.AlternateFilters.ElementAt(y).ComparisonOperator:ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(y).ComparisonValue:"2819c223-7f76-453a-919d-413861904646"
* parameters.RequestedAttributePaths.ElementAt(0):“ID”
* parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

  在这里，索引 x 的值可以是 0 并且索引 y 的值可以是 1，或 x 的值可以是 1 和 y 的值可以是 0，具体取决于筛选器查询参数的表达式的顺序。   

1. 以下是从 Azure Active Directory 向 SCIM 服务发出更新用户请求的示例： 

   ```
     PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
     Authorization: Bearer ...
     Content-type: application/scim+json
     {
       "schemas": 
       [
         "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
       "Operations":
       [
         {
           "op":"Add",
           "path":"manager",
           "value":
             [
               {
                 "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
                 "value":"2819c223-7f76-453a-919d-413861904646"}]}]}
   ```

   用于实现 SCIM 服务的 Microsoft 通用语言基础结构库将请求转换为对服务提供者的 Update 方法调用。 以下是 Update 方法的签名： 

   ```
     // System.Threading.Tasks.Tasks and 
     // System.Collections.Generic.IReadOnlyCollection<T>
     // are defined in mscorlib.dll.  
     // Microsoft.SystemForCrossDomainIdentityManagement.IPatch, 
     // Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase, 
     // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
     // Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation, 
     // Microsoft.SystemForCrossDomainIdentityManagement.OperationName, 
     // Microsoft.SystemForCrossDomainIdentityManagement.IPath and 
     // Microsoft.SystemForCrossDomainIdentityManagement.OperationValue 
     // are all defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 

     System.Threading.Tasks.Task Update(
       Microsoft.SystemForCrossDomainIdentityManagement.IPatch patch, 
       string correlationIdentifier);

     public interface Microsoft.SystemForCrossDomainIdentityManagement.IPatch
     {
     Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase 
       PatchRequest 
         { get; set; }
     Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier 
       ResourceIdentifier 
         { get; set; }        
     }

     public class PatchRequest2: 
       Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase
     {
     public System.Collections.Generic.IReadOnlyCollection
       <Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation> 
         Operations
         { get;}

     public void AddOperation(
       Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation operation);
     }

     public class PatchOperation
     {
     public Microsoft.SystemForCrossDomainIdentityManagement.OperationName 
       Name
       { get; set; }

     public Microsoft.SystemForCrossDomainIdentityManagement.IPath 
       Path
       { get; set; }

     public System.Collections.Generic.IReadOnlyCollection
       <Microsoft.SystemForCrossDomainIdentityManagement.OperationValue> Value
       { get; }

     public void AddValue(
       Microsoft.SystemForCrossDomainIdentityManagement.OperationValue value);
     }

     public enum OperationName
     {
       Add,
       Remove,
       Replace
     }

     public interface IPath
     {
       string AttributePath { get; }
       System.Collections.Generic.IReadOnlyCollection<IFilter> SubAttributes { get; }
       Microsoft.SystemForCrossDomainIdentityManagement.IPath ValuePath { get; }
     }

     public class OperationValue
     {
       public string Reference
       { get; set; }

       public string Value
       { get; set; }
     }
   ```

    在更新用户的请求示例中，作为修补参数值提供的对象具有这些属性值： 
  
   * ResourceIdentifier.Identifier:"54D382A4-2050-4C03-94D1-E769F1D15682"
   * ResourceIdentifier.SchemaIdentifier:  "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
   * (PatchRequest as PatchRequest2).Operations.Count:第
   * (PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName:OperationName.Add
   * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath: "manager"
   * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count:第
   * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference: http://.../scim/Users/2819c223-7f76-453a-919d-413861904646
   * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value:2819c223-7f76-453a-919d-413861904646

1. 若要从 SCIM 服务面对的标识存储中取消预配用户，Azure AD 会发送如下请求： 

   ```
     DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
     Authorization: Bearer ...
   ```

   如果使用 Microsoft 提供的、用于实现 SCIM 服务的公共语言基础结构库构建了服务，则将请求转换为对服务提供者的 Delete 方法调用。   该方法具有以下签名： 

   ```
     // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
     // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
     // is defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 
     System.Threading.Tasks.Task Delete(
       Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier  
         resourceIdentifier, 
       string correlationIdentifier);
   ```

   在取消预配用户的请求示例中，作为 resourceIdentifier 参数值提供的对象具有以下属性值： 

1. 若要从 SCIM 服务面对的标识存储中取消预配用户，Azure AD 会发送如下请求： 
   ````
    DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
   ````
   如果使用由 Microsoft 提供用于实现 SCIM 服务的 CLI 库生成服务，则会将请求转换为对服务的提供程序的 Delete 方法的调用。   该方法具有以下签名： 
   ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
    // is defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 
    System.Threading.Tasks.Task Delete(
      Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier  
        resourceIdentifier, 
      string correlationIdentifier);
   ````
   在取消预配用户的请求示例中，作为 resourceIdentifier 参数值提供的对象具有以下属性值： 
  
   * ResourceIdentifier.Identifier:"54D382A4-2050-4C03-94D1-E769F1D15682"
   * ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

## <a name="user-and-group-schema-reference"></a>用户和组架构参考
Azure Active Directory 可将两种类型的资源预配到 SCIM Web 服务。  这些类型的资源是用户和组。  

用户资源由架构标识符`urn:ietf:params:scim:schemas:extension:enterprise:2.0:User`，此协议规范中包括的哪些： https://tools.ietf.org/html/rfc7643。  表 1 中提供了 Azure Active Directory 中用户的用户资源的属性的属性的默认映射。  

组资源由架构标识符 `urn:ietf:params:scim:schemas:core:2.0:Group` 予以标识。 表 2 显示在 Azure Active Directory 中组的属性的默认映射了组中资源的属性。  

### <a name="table-1-default-user-attribute-mapping"></a>表 1：默认用户属性映射

| Azure Active Directory 用户 | “urn:ietf:params:scim:schemas:extension:enterprise:2.0:User” |
| --- | --- |
| IsSoftDeleted |活动 |
| displayName |displayName |
| Facsimile-TelephoneNumber |phoneNumbers[type eq "fax"].value |
| givenName |name.givenName |
| jobTitle |title |
| mail |emails[type eq "work"].value |
| mailNickname |externalId |
| manager |manager |
| mobile |phoneNumbers[type eq "mobile"].value |
| objectId |ID |
| postalCode |addresses[type eq "work"].postalCode |
| proxy-Addresses |emails[type eq "other"].Value |
| physical-Delivery-OfficeName |addresses[type eq "other"].Formatted |
| streetAddress |addresses[type eq "work"].streetAddress |
| surname |name.familyName |
| telephone-Number |phoneNumbers[type eq "work"].value |
| user-PrincipalName |userName |

### <a name="table-2-default-group-attribute-mapping"></a>表 2：默认组属性映射

| Azure Active Directory 组 | urn:ietf:params:scim:schemas:core:2.0:Group |
| --- | --- |
| displayName |externalId |
| mail |emails[type eq "work"].value |
| mailNickname |displayName |
| members |members |
| objectId |ID |
| proxyAddresses |emails[type eq "other"].Value |


## <a name="related-articles"></a>相关文章
* [在 SaaS 应用中自动预配和取消预配用户](user-provisioning.md)
* [为用户预配自定义属性映射](customize-application-attributes.md)
* [为属性映射编写表达式](functions-for-customizing-application-data.md)
* [用于用户预配的作用域筛选器](define-conditional-rules-for-provisioning-user-accounts.md)
* [帐户预配通知](user-provisioning.md)
* [有关如何集成 SaaS 应用的教程列表](../saas-apps/tutorial-list.md)

<!--Image references-->
[0]: ./media/use-scim-to-provision-users-and-groups/scim-figure-1.png
[1]: ./media/use-scim-to-provision-users-and-groups/scim-figure-2a.png
[2]: ./media/use-scim-to-provision-users-and-groups/scim-figure-2b.png
[3]: ./media/use-scim-to-provision-users-and-groups/scim-figure-3.png
[4]: ./media/use-scim-to-provision-users-and-groups/scim-figure-4.png
[5]: ./media/use-scim-to-provision-users-and-groups/scim-figure-5.png
