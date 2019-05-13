---
title: 创建、管理和保护管理员和查询 API 密钥 - Azure 搜索
description: API 密钥控制服务终结点的访问权限。 管理员密钥授予写入权限。 可为只读访问权限创建查询密钥。
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.openlocfilehash: f12d7f1a7dfcaf80df9a71a0bfc598d72db63a2e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024417"
---
# <a name="create-and-manage-api-keys-for-an-azure-search-service"></a>创建和管理 Azure 搜索服务的 API 密钥

对搜索服务的所有请求都需要专为服务生成的只读 API 密钥。 API 密钥是用于验证搜索服务终结点的访问的唯一机制，必须包含在每个请求中。 在 [REST 解决方案](search-get-started-nodejs.md#update-the-configjs-with-your-search-service-url-and-api-key)中，API 密钥通常在请求标头中指定。 在 [.NET 解决方案](search-howto-dotnet-sdk.md#core-scenarios)中，密钥通常以配置设置的形式指定，然后在 [SearchServiceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient) 上作为[凭据](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient.credentials)（管理密钥）或 [SearchCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient.searchcredentials)（查询密钥）传递。

在服务预配期间，将使用搜索服务创建密钥。 可以在 [Azure 门户](https://portal.azure.com)中查看和获取密钥值。

![门户页上的“设置”>“密钥”部分](media/search-manage/azure-search-view-keys.png)

## <a name="what-is-an-api-key"></a>什么是 API 密钥？

API 密钥是随机生成的数字和字母所组成的字符串。 通过[基于角色的权限](search-security-rbac.md)，可以删除或读取密钥，但无法将密钥替换为用户定义的密码或使用 Active Directory 作为访问搜索操作的主要身份验证方法。 

使用了两种类型的密钥来访问搜索服务：管理员（只写）和查询（只读）。

|密钥|描述|限制|  
|---------|-----------------|------------|  
|管理员|授予所有操作的完全控制权限，包括管理服务以及创建和删除索引、索引器与数据源的能力。<br /><br /> 创建服务时，会在门户中生成两个管理密钥（称为主密钥和辅助密钥），且可按需单独重新生成。 由于有两个密钥，可以在滚动其中一个密钥时，使用另一个密钥来持续访问服务。<br /><br /> 只能在 HTTP 请求标头中指定管理密钥。 不能在 URL 中放置管理 API 密钥。|每个服务最多有 2 个密钥|  
|Query|授予对索引和文档的只读访问权限，通常分发给发出搜索请求的客户端应用程序。<br /><br /> 按需创建查询密钥。 可以在门户中手动创建查询密钥，或者通过[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 以编程方式创建查询密钥。<br /><br /> 可以在 HTTP 请求标头中为搜索、建议或查找操作指定查询密钥。 或者，可以在 URL 中以参数形式传递查询密钥。 根据客户端应用程序编写请求的方式，以查询参数的形式传递密钥可能更方便：<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2019-05-06&api-key=[query key]`|每个服务 50 个密钥|  

 在表面上，管理密钥与查询密钥之间没有区别。 这两个密钥都是由 32 个随机生成的字母数字字符组成的字符串。 如果无法跟踪应用程序中指定了哪种类型的密钥，可以[在门户中检查密钥值](https://portal.azure.com)，或使用 [REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 返回值和密钥类型。  

> [!NOTE]  
>  在请求 URI 中传递敏感数据（例如 `api-key`）被认为是不良的安全做法。 为此，Azure 搜索只接受查询字符串中 `api-key` 形式的查询密钥。除非必须公开索引的内容，否则应避免这样做。 作为经验法则，我们建议以请求标头的形式传递 `api-key`。  

## <a name="find-existing-keys"></a>查找现有密钥

可以在门户中或通过[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 获取访问密钥。 有关详细信息，请参阅[管理管理员和查询 API 密钥](search-security-api-keys.md)。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 列出订阅的[搜索服务](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。
3. 选择服务，然后在概述页上单击**设置** >**密钥**若要查看管理密钥和查询密钥。

   ![门户页上的“设置”>“密钥”部分](media/search-security-overview/settings-keys.png)

## <a name="create-query-keys"></a>创建查询密钥

查询密钥用于只读访问到索引中的文档的文档集合为目标的操作。 搜索、 筛选和建议的查询是进行查询密钥的所有操作。 返回系统数据或对象定义，如索引定义或索引器状态，任何只读操作需要管理密钥。

限制访问和客户端应用中的操作是必要的保护你的服务上的搜索资产。 始终使用从客户端应用程序发起的任何查询的查询密钥而不是管理密钥。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 列出订阅的[搜索服务](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。
3. 选择服务，然后在概述页上单击**设置** >**密钥**。
4. 单击**管理查询密钥**。
5. 使用已为你的服务，生成的查询密钥或创建最多 50 个新的查询密钥。 未命名的默认查询密钥，但其他查询密钥可以命名的可管理性。

   ![创建或使用查询密钥](media/search-security-overview/create-query-key.png) 

> [!Note]
> 显示查询密钥使用情况的代码示例可在[查询中的 Azure 搜索索引C# ](search-query-dotnet.md)。

<a name="regenerate-admin-keys"></a>

## <a name="regenerate-admin-keys"></a>重新生成管理员密钥

针对每个服务创建两个管理密钥，以便可以旋转主键，实现业务连续性使用辅助密钥。

1. 在“设置” >“密钥”页中，复制辅助密钥。
2. 对于所有应用程序，更新 API 密钥设置以使用辅助密钥。
3. 重新生成主密钥。
4. 更新所有应用程序以使用新的主密钥。

如果你无意中重新生成在同一时间的这两个密钥，使用这些密钥的所有客户端请求将失败并 HTTP 403 禁止访问。 但是，不删除内容，并且您不应局限永久。 

您仍可以通过门户或管理层访问服务 ([REST API](https://docs.microsoft.com/rest/api/searchmanagement/)， [PowerShell](https://docs.microsoft.com/azure/search/search-manage-powershell)，或 Azure 资源管理器)。 管理功能是订阅 ID 不是服务 api 的密钥，通过操作性，因此仍然可用，即使你的 api 密钥并不是。 

创建新的密钥通过门户或管理层后，你的内容 （索引、 索引器、 数据源、 同义词映射） 恢复访问权限后具有新的密钥和在请求中提供这些键。

## <a name="secure-api-keys"></a>保护 API 密钥
通过门户或 Resource Manager 界面（PowerShell 或命令行接口）以限制访问，从而保护密钥安全。 如前所述，订阅管理员可以查看和重新生成所有 API 密钥。 作为预防措施，查看角色分配以了解谁有权访问管理密钥。

+ 在服务仪表板中，依次单击“访问控制(IAM)”和“角色分配”选项卡可查看服务的角色分配。

以下角色的成员可以查看和重新生成密钥：所有者、参与者、[搜索服务参与者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#search-service-contributor)

> [!Note]
> 如果要实现针对搜索结果的、基于标识的访问，可创建安全筛选器按标识来细化结果，由此去除请求者不应具有访问权限的那些文档。 有关详细信息，请参阅[安全筛选器](search-security-trimming-for-azure-search.md)和[使用 Active Directory 进行保护](search-security-trimming-for-azure-search-with-aad.md)。

## <a name="see-also"></a>另请参阅

+ [Azure 搜索中基于角色的访问控制](search-security-rbac.md)
+ [使用 PowerShell 进行管理](search-manage-powershell.md) 
+ [性能和优化文章](search-performance-optimization.md)