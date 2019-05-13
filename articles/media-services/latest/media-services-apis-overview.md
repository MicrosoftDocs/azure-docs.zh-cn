---
title: 使用 v3 Api-Azure 进行开发 |Microsoft Docs
description: 本文介绍了使用媒体服务 v3 进行开发时对实体和 Api 应用的规则。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/02/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 4c5b30ab075bbca22b6a58ccf65e55d332820937
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406551"
---
# <a name="developing-with-media-services-v3-apis"></a>使用媒体服务 v3 Api 进行开发

本文介绍了使用媒体服务 v3 进行开发时对实体和 Api 应用的规则。

## <a name="accessing-the-azure-media-services-api"></a>访问 Azure 媒体服务 API

若要有权访问媒体服务资源和媒体服务 API，必须先进行身份验证。 媒体服务支持[Azure Active Directory (Azure AD) 的基于](../../active-directory/fundamentals/active-directory-whatis.md)身份验证。 两个常见的身份验证选项包括：
 
* **服务主体身份验证**-用于进行身份验证服务 (例如： web 应用、 函数应用、 逻辑应用、 API 和微服务)。 常常使用这种身份验证方法的应用程序是运行守护程序服务、中间层服务或计划作业的应用程序。 例如，对于 Web 应用程序存在应始终为连接到媒体服务与服务主体的中间层。
* **用户身份验证**-用于进行身份验证使用应用程序与媒体服务资源进行交互的人员。 交互式应用程序应先提示用户输入用户凭据。 例如，授权用户用来监视编码作业或实时传送视频流的管理控制台应用程序。

媒体服务 API 需要用户或应用程序进行 REST API 请求有权访问媒体服务帐户资源，并使用**参与者**或**所有者**角色。 可以使用访问 API**读取器**角色但只**获取**或**列表** 操作将可用。 有关详细信息，请参阅[媒体服务帐户的基于角色的访问控制](rbac-overview.md)。

而不是创建服务主体，请考虑使用 Azure 资源的管理的标识访问通过 Azure 资源管理器中的媒体服务 API。 若要了解有关 Azure 资源的管理的标识的详细信息，请参阅[什么是 Azure 资源的管理的标识](../../active-directory/managed-identities-azure-resources/overview.md)。

### <a name="azure-ad-service-principal"></a>Azure AD 服务主体 

如果要创建 Azure AD 应用程序和服务主体，该应用程序必须处于它自己的租户。 创建应用程序后，让应用程序**参与者**或**所有者**角色到媒体服务帐户的访问。 

如果您不确定是否有权创建一个 Azure AD 应用程序，请参阅[所需的权限](../../active-directory/develop/howto-create-service-principal-portal.md#required-permissions)。

在下图中，数字表示按时间顺序的请求流：

![中间层应用](./media/use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

1. 中间层应用程序请求 Azure AD 访问令牌具有以下参数：  

   * Azure AD 租户终结点。
   * 媒体服务资源 URI。
   * REST 媒体服务的资源 URI。
   * Azure AD 应用程序值：客户端 ID和客户端密码。
   
   若要获取所有所需的值，请参阅[使用 Azure CLI 访问 Azure 媒体服务 API](access-api-cli-how-to.md)

2. Azure AD 访问令牌发送到中间层。
4. 中间层使用 Azure AD 令牌向 Azure 媒体 REST API 发送请求。
5. 中间层获取媒体服务返回的数据。

### <a name="samples"></a>示例

请参阅下面的示例演示如何使用 Azure AD 服务主体进行连接：

* [使用 REST 连接](media-rest-apis-with-postman.md)  
* [通过 Java 进行连接](configure-connect-java-howto.md)
* [通过 .NET 进行连接](configure-connect-dotnet-howto.md)
* [通过 Node.js 进行连接](configure-connect-nodejs-howto.md)
* [通过 Python 进行连接](configure-connect-python-howto.md)

## <a name="naming-conventions"></a>命名约定

Azure 媒体服务 v3 资源名称（例如，资产、作业、转换）需遵循 Azure 资源管理器命名约束。 根据 Azure 资源管理器的要求，资源名称始终必须唯一。 因此，可以为资源名称使用任何唯一的标识符字符串（例如，GUID）。 

媒体服务资源名称不能包含“<”、“>”、“%”、“&”、“:”、“&#92;”、“?”、“/”、“*”、“+”、“.”、单引号或任何控制字符。 允许其他所有字符。 资源名称的最大长度为 260 个字符。 

有关 Azure 资源管理器命名的详细信息，请参阅：[命名需求](https://github.com/Azure/azure-resource-manager-rpc/blob/master/v1.0/resource-api-reference.md#arguments-for-crud-on-resource)和[命名约定](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。

## <a name="long-running-operations"></a>长时间运行的操作

操作标记为与`x-ms-long-running-operation`在 Azure 媒体服务[swagger 文件](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/streamingservice.json)是长时间运行的操作。 

有关如何跟踪异步 Azure 操作的详细信息，请参阅[异步操作](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-async-operations#monitor-status-of-operation)。

媒体服务具有以下长时间运行的操作：

* 创建 LiveEvent
* 更新 LiveEvent
* 删除 LiveEvent
* Start LiveEvent
* 停止 LiveEvent
* 重置 LiveEvent
* 创建 LiveOutput
* Delete LiveOutput
* 创建 StreamingEndpoint
* 更新 StreamingEndpoint
* 删除 StreamingEndpoint
* 启动 StreamingEndpoint
* 停止 StreamingEndpoint
* 缩放 StreamingEndpoint

## <a name="filtering-ordering-paging-of-media-services-entities"></a>媒体服务实体的筛选、排序和分页

请参阅[筛选、 排序、 分页的 Azure 媒体服务实体](entities-overview.md)

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

[开始使用媒体服务 v3 API 使用 Sdk/工具进行开发](developers-guide.md)
