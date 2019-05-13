---
title: 使用 Azure 自定义提供程序预览版创建资源提供程序
description: 介绍如何创建资源提供程序并部署其自定义资源类型。
services: managed-applications
author: MSEvanhi
ms.service: managed-applications
ms.topic: tutorial
ms.date: 05/01/2019
ms.author: evanhi
ms.openlocfilehash: e4196cf59537be5194ceb510a1b7b066c97de19a
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410224"
---
# <a name="tutorial-create-custom-provider-and-deploy-custom-resources"></a>教程：创建自定义提供程序并部署自定义资源

在本教程中，我们将创建自己的资源提供程序，并为该资源提供程序部署自定义资源类型。 有关自定义提供程序的详细信息，请参阅 [Azure 自定义提供程序预览版概述](custom-providers-overview.md)。

## <a name="prerequisites"></a>先决条件

完成本教程中的步骤需要调用 REST 操作。 可以通过[不同的方法来发送 REST 请求](/rest/api/azure/)。 如果还没有用于 REST 操作的工具，请安装 [ARMClient](https://github.com/projectkudu/ARMClient)。 它是一种可简化 Azure 资源管理器 API 调用的开源命令行工具。

## <a name="deploy-custom-provider"></a>部署自定义提供程序

若要设置自定义提供程序，请将[示例模板](https://github.com/Azure/azure-docs-json-samples/blob/master/custom-providers/customprovider.json)部署到 Azure 订阅。

部署模板之后，订阅将具有以下资源：

* 可以针对资源和操作运行的 Function App。
* 存储帐户，用于存储通过自定义提供程序创建的用户。
* 自定义提供程序，用于定义自定义资源类型和操作。 它使用函数应用终结点来发送请求。
* 自定义提供程序提供的自定义资源。

若要通过 PowerShell 部署自定义提供程序，请使用：

```azurepowershell-interactive
$rgName = "<resource-group-name>"
$funcName = "<function-app-name>"

New-AzResourceGroup -Name $rgName -Location eastus
New-AzResourceGroupDeployment -ResourceGroupName $rgName `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json `
  -funcname $funcName
```

也可通过以下按钮来部署解决方案：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-docs-json-samples%2Fmaster%2Fcustom-providers%2Fcustomprovider.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

## <a name="view-custom-provider-and-resource"></a>查看自定义提供程序和资源

在门户中，自定义提供程序是隐藏的资源类型。 若要确认资源提供程序是否已部署，请导航到资源组。 选择“显示隐藏类型”选项。

![显示隐藏的资源类型](./media/create-custom-providers/show-hidden.png)

若要查看已部署的自定义资源类型，请在资源类型上使用 GET 操作。

```
GET https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users?api-version=2018-09-01-preview
```

有了 ARMClient，可以使用：

```powershell
$subID = (Get-AzContext).Subscription.Id
$requestURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"

armclient GET $requestURI
```

你会收到响应：

```json
{
  "value": [
    {
      "properties": {
        "provisioningState": "Succeeded",
        "FullName": "Santa Claus",
        "Location": "NorthPole"
      },
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

## <a name="call-action"></a>调用操作

自定义提供程序还有名为 **ping** 的操作。 处理请求的代码在函数应用中实现。 ping 操作使用一条问候信息进行回复。

若要发送 ping 请求，请在自定义提供程序上使用 POST 操作。

```
POST https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/ping?api-version=2018-09-01-preview
```

有了 ARMClient，可以使用：

```powershell
$pingURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"

armclient POST $pingURI
```

你会收到响应：

```json
{
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  },
  "message": "hello <function-name>.azurewebsites.net"
}
```

## <a name="create-resource-type"></a>创建资源类型

若要创建自定义资源类型，可以在模板中部署资源。 此方法显示在本教程部署的模板中。 也可针对此资源类型发送 PUT 请求。

```
PUT https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/<resource-name>?api-version=2018-09-01-preview

{"properties":{"FullName": "Test User", "Location": "Earth"}}
```

有了 ARMClient，可以使用：

```powershell
$addURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
$requestBody = "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"

armclient PUT $addURI $requestBody
```

你会收到响应：

```json
{
  "properties": {
    "provisioningState": "Succeeded",
    "FullName": "Test User",
    "Location": "Earth"
  },
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

## <a name="next-steps"></a>后续步骤

有关自定义提供程序的简介，请参阅 [Azure 自定义提供程序预览版概述](custom-providers-overview.md)。
