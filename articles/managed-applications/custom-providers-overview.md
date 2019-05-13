---
title: Azure 自定义提供程序预览版概述
description: 介绍用于创建自定义资源提供程序使用 Azure 资源管理器概念
author: MSEvanhi
ms.service: managed-applications
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: evanhi
ms.openlocfilehash: bbfb10f612690af0f4fd3683e0f58986a21048d8
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159850"
---
# <a name="azure-custom-providers-preview-overview"></a>Azure 自定义提供程序预览版概述

使用 Azure 自定义提供程序，可以扩展 Azure，以使用你的服务。 创建自己的资源提供程序，包括自定义的资源类型和操作。 使用 Azure 资源管理器中集成自定义提供程序。 资源管理器的功能，例如，模板部署和基于角色的访问控制，可用于部署和保护你的服务。

本文提供了自定义提供程序和其功能的概述。 下图显示了资源和操作自定义提供程序中定义的工作流。

![自定义提供程序概述](./media/custom-providers-overview/overview.png)

> [!IMPORTANT]
> 自定义提供程序目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="define-your-custom-provider"></a>定义自定义提供程序

首先让 Azure 资源管理器了解的有关自定义提供程序。 部署到 Azure 时使用的资源类型的自定义提供程序资源**Microsoft.CustomProviders/resourceProviders**。 在该资源中，为你的服务定义的资源和操作。

例如，如果服务需要使用一个名为的资源类型**用户**，在自定义提供程序定义中包含该资源类型。 对于每种资源类型，需要提供终结点，可提供该资源类型的 REST 操作 (PUT、 GET、 DELETE)。 可以在任何环境上承载终结点，其中包含你的服务如何处理对资源类型执行操作的逻辑。

此外可以定义资源提供程序的自定义操作。 操作表示 POST 操作。 使用操作来操作，如启动、 停止或重新启动。 提供处理请求的终结点。

下面的示例演示如何定义自定义提供程序使用的操作和资源类型。

```json
{
  "apiVersion": "2018-09-01-preview",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "name": "[parameters('funcName')]",
  "location": "[parameters('location')]",
  "properties": {
    "actions": [
      {
        "name": "ping",
        "routingType": "Proxy",
        "endpoint": "[concat('https://', parameters('funcName'), '.azurewebsites.net/api/{requestPath}')]"
      }
    ],
    "resourceTypes": [
      {
        "name": "users",
        "routingType": "Proxy,Cache",
        "endpoint": "[concat('https://', parameters('funcName'), '.azurewebsites.net/api/{requestPath}')]"
      }
    ]
  }
},
```

有关**routingType**，接受的值为`Proxy`和`Cache`。 代理资源类型意味着请求或操作处理的终结点。 资源类型，不操作仅支持缓存设置。 若要指定缓存，还必须指定代理服务器。 缓存意味着从终结点的响应进行存储，以便优化读取的操作。 使用的缓存设置，可以更轻松地实现与其他资源管理器服务是一致且合规的 API。

## <a name="deploy-your-resource-types"></a>部署资源类型

定义自定义提供程序之后, 你可以部署自定义的资源类型。 下面的示例显示了要为自定义提供程序部署的资源类型在模板中包含的 JSON。 可以与其他 Azure 资源在同一模板中部署此资源类型。

```json
{
    "apiVersion": "2018-09-01-preview",
    "type": "Microsoft.CustomProviders/resourceProviders/users",
    "name": "[concat(parameters('rpname'), '/santa')]",
    "location": "[parameters('location')]",
    "properties": {
        "FullName": "Santa Claus",
        "Location": "NorthPole"
    }
}
```

## <a name="manage-access"></a>管理访问权限

使用 Azure[基于角色的访问控制](../role-based-access-control/overview.md)管理到资源提供程序的访问权限。 可以将分配[内置角色](../role-based-access-control/built-in-roles.md)如所有者、 参与者或向用户的读取器。 或者，可以定义[自定义角色](../role-based-access-control/custom-roles.md)特定于资源提供程序中的操作。

## <a name="next-steps"></a>后续步骤

在本文中，您学习了有关自定义提供程序的信息。 请转到下一篇文章中创建自定义提供程序。

> [!div class="nextstepaction"]
> [教程：创建自定义提供程序和部署自定义资源](create-custom-provider.md)
