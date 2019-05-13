---
title: 管理和连接到 Azure 中国世纪互联订阅 | Microsoft Docs
description: Azure 中国世纪互联提供独特的 URL 和终结点用于管理环境。 连接到 Azure 环境后，如果组件已正确部署，则可以正常运行管理服务的操作。 了解为何必须使用正确的连接来管理环境。
services: china
cloud: na
documentationcenter: na
author: v-wimarc
manager: edprice
ms.assetid: na
ms.service: china
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/29/2017
ms.author: v-wimarc
ms.openlocfilehash: 33c2c053fc822e0dd89b8d9aca9b4e55893c07b5
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60653727"
---
# <a name="manage-and-connect-to-your-subscription"></a>管理和连接到订阅
世纪互联运营的 Microsoft Azure（Azure 中国世纪互联）提供独特的 URL 和终结点用于管理环境。 必须使用正确的连接来管理环境。 连接到 Azure 环境后，如果已部署相应的组件，则可以运行管理服务的正常操作。

## <a name="connect-by-using-the-portal"></a>使用门户进行连接
可通过 [Azure 门户](https://portal.azure.cn/)（一个 [Azure 资源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)平台）访问 Microsoft Azure 中国世纪互联中的应用程序和服务。

![Azure 门户](media/china-get-started-service-manage-and-connect/azureportal.png)

## <a name="work-with-administrator-roles"></a>使用管理员角色
为每个 Azure 帐户创建一个帐户管理员角色，通常，该角色是注册或购买 Azure 订阅的人员。 该角色有权使用[帐户中心](https://account.windowsazure.cn/)执行管理任务。

若要登录，帐户管理员可使用购买订阅时创建的组织 ID (OrgID)。 

### <a name="create-a-service-administrator-to-manage-the-service-deployment"></a>创建用于管理服务部署的服务管理员 
为每个 Azure 帐户创建一个服务管理员角色，该角色有权在 Azure 门户中管理服务。 在新订阅中，帐户管理员也是服务管理员。

若要创建服务管理员，请执行以下操作：
1.  登录到 Azure 门户，在左侧导航菜单中单击“Active Directory”。
2.  在“名称”下，单击“添加用户”。 输入新用户的用户名、配置文件和临时密码。
3.  单击“查看我的帐单”，选择订阅，单击“编辑订阅详细信息”。
4.  将服务管理员更改为新建用户。

### <a name="create-a-co-administrator"></a>创建共同管理员
帐户管理员可为每个订阅最多创建 199 个共同管理员角色。 此角色的访问特权与服务管理员相同，但不能更改订阅与 Azure 目录之间的关联。
1.  登录到 Azure 门户，在左侧导航菜单中单击“Active Directory”。
2.  在“名称”下，单击“添加用户”。 输入新用户的用户名、配置文件和临时密码。
3.  在主菜单栏上，依次单击“设置”、“管理员”。
4.  单击“添加”，输入上一步骤中所提供的用户地址。 使用原始帐户域。 选择共同管理员有权访问的订阅。
5.  在“设置”页上检查更改。

## <a name="next-steps"></a>后续步骤
- [Azure 门户](https://portal.azure.cn/)
- [Azure 资源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)
- [帐户中心](https://account.windowsazure.cn/)

