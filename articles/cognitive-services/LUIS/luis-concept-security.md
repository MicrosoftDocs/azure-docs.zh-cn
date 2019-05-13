---
title: 协作时的安全性
titleSuffix: Language Understanding - Azure Cognitive Services
description: 所有者和协作者具有创作访问权限。 对于专用应用，所有者和协作者具有终结点访问权限。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 12/18/2018
ms.author: diberry
ms.openlocfilehash: 499854bcf6774c3e4eee350c1dd4a2204885f3b1
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522480"
---
# <a name="authoring-and-endpoint-user-access"></a>创作和终结点用户访问权限
所有者和协作者具有创作访问权限。 对于专用应用，所有者和协作者具有终结点访问权限。 对于公共应用，任何具有 LUIS 帐户和公共应用 ID 的人员均具有终结点访问权限。 

## <a name="access-to-authoring"></a>访问创作
从 [LUIS](luis-reference-regions.md#luis-website) 网站或[创作 API](https://go.microsoft.com/fwlink/?linkid=2092087) 访问应用的权限由应用的所有者控制。 

所有者和所有协作者具有创作应用的权限。 

|创作访问权限包括|说明|
|--|--|
|添加或删除终结点密钥||
|导出版本||
|导出终结点日志||
|导入版本||
|公开应用|如果应用公开，任何拥有创作或终结点密钥的人员都可以查询应用。|
|修改模型|
|发布|
|查看用于[主动学习](luis-how-to-review-endpoint-utterances.md)的终结点陈述|
|训练|

## <a name="access-to-endpoint"></a>访问终结点

查询终结点的访问权限由“应用程序信息”页上的“管理”部分中的设置进行控制。 

![将应用设置为公共](./media/luis-concept-security/set-application-as-public.png)

|[专用终结点](#private-app-endpoint-security)|[公共终结点](#public-app-endpoint-access)|
|:--|:--|
|可供所有者和协作者使用|可供所有者、协作者以及知道应用 ID 的任何其他人使用|

### <a name="private-app-endpoint-security"></a>专有应用终结点安全

专有应用的终结点仅适用于下述情况：

|密钥和用户|说明|
|--|--|
|所有者的创作密钥| 最多 1000 个终结点命中数|
|协作者的创作密钥| 最多 1000 个终结点命中数|
|由创建者或协作者分配给 LUIS 的任何密钥|基于密钥用法层|

#### <a name="microsoft-user-accounts"></a>Microsoft 用户帐户

创建者和协作者可以将密钥分配给专用 LUIS 应用。 用以在 Azure 门户中创建 LUIS 密钥的 Microsoft 用户帐户需为应用所有者或应用协作者。 无法从其他 Azure 帐户将密钥分配给专用应用。

请参阅 [Azure Active Directory 租户用户](luis-how-to-collaborate.md#azure-active-directory-tenant-user)，了解有关 Active Directory 用户帐户的详细信息。 

### <a name="public-app-endpoint-access"></a>公共应用终结点访问权限

应用配置为公共后，任何有效的 LUIS 创作密钥或 LUIS 终结点密钥都可以查询应用，只要该密钥未使用整个终结点配额。

如果用户不是所有者或协作者，则只有在给定应用 ID 时才能访问公共应用。 LUIS 不提供公共市场或其他搜索公共应用的方式。  

公共应用在所有区域中发布，以便有基于区域的 LUIS 资源密钥的用户可以在与资源密钥关联的任何区域中访问该应用。

## <a name="microsoft-user-accounts"></a>Microsoft 用户帐户

创建者和协作者可以在“发布”页上向 LUIS 添加密钥。 用以在 Azure 门户中创建 LUIS 密钥的 Microsoft 用户帐户需为应用所有者或应用协作者。 

请参阅 [Azure Active Directory 租户用户](luis-how-to-collaborate.md#azure-active-directory-tenant-user)，了解有关 Active Directory 用户帐户的详细信息。 

## <a name="securing-the-endpoint"></a>保护终结点安全 

可以通过在服务器到服务器环境中调用终结点密钥来控制可以查看 LUIS 终结点的人员。 如果在机器人上使用 LUIS，则机器人和 LUIS 之间的连接已经安全。 如果直接调用 LUIS 终结点，则应创建具有受控访问权限（如 [AAD](https://azure.microsoft.com/services/active-directory/)）的服务器端 API（如 Azure [函数](https://azure.microsoft.com/services/functions/)）。 如果调用服务器端 API 并且身份验证和授权得到验证，则将调用传递到 LUIS。 尽管此策略不会防止中间人攻击，但它针对用户模糊化处理终结点，允许跟踪访问，并允许添加终结点响应日志记录（如 [Application Insights](https://azure.microsoft.com/services/application-insights/)）。  

## <a name="security-compliance"></a>安全性和符合性
 
[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-security-compliance.md)]

## <a name="next-steps"></a>后续步骤

请参阅[最佳做法](luis-concept-best-practices.md)，了解如何使用意向和实体进行最佳预测。
