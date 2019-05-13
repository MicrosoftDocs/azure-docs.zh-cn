---
title: 自动执行在 Azure 开发测试实验室中添加实验室用户 |Microsoft Docs
description: 了解如何自动向 Azure 开发测试实验室中的实验室添加实验室用户。
services: devtest-lab,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/02/2019
ms.author: spelluru
ms.openlocfilehash: 0eed874d405fcf99241a702292f8ceadae6c5a07
ms.sourcegitcommit: 1d257ad14ab837dd13145a6908bc0ed7af7f50a2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65502030"
---
# <a name="automate-adding-a-lab-user-to-a-lab-in-azure-devtest-labs"></a>自动向 Azure 开发测试实验室中的实验室添加实验室用户
Azure 开发测试实验室，可通过使用 Azure 门户快速创建自助服务的开发测试环境。 但是，如果有多个团队和多个开发测试实验室实例，则自动执行创建过程可以节省时间。 [Azure 资源管理器模板](https://github.com/Azure/azure-devtestlab/tree/master/ARMTemplates)，可以创建实验室、 实验室 Vm、 自定义映像、 公式和以自动方式添加用户。 本文专门重点介绍将用户添加到开发测试实验室实例。

若要将用户添加到实验室，你将用户添加到**开发测试实验室用户**实验室的角色。 本文介绍如何自动将用户添加到实验室中使用以下方法之一：

- Azure 资源管理器模板
- Azure PowerShell cmdlet 
- Azure CLI。

## <a name="use-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板
下面的示例 Resource Manager 模板指定要添加到用户**开发测试实验室用户**实验室的角色。 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The objectId of the user, group, or service principal for the role."
      }
    },
    "labName": {
      "type": "string",
      "metadata": {
        "description": "The name of the lab instance to be created."
      }
    },
    "roleAssignmentGuid": {
      "type": "string",
      "metadata": {
        "description": "Guid to use as the name for the role assignment."
      }
    }
  },
  "variables": {
    "devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
    "fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]",
    "roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
  },
  "resources": [
    {
      "apiVersion": "2016-05-15",
      "type": "Microsoft.DevTestLab/labs",
      "name": "[parameters('labName')]",
      "location": "[resourceGroup().location]"
    },
    {
      "apiVersion": "2016-07-01",
      "type": "Microsoft.DevTestLab/labs/providers/roleAssignments",
      "name": "[variables('fullDevTestLabUserRoleName')]",
      "properties": {
        "roleDefinitionId": "[variables('devTestLabUserRoleId')]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('roleScope')]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
      ]
    }
  ]
}

```

如果您将创建实验室在同一模板中的角色分配，请记住添加角色分配资源和实验室之间的依赖项。 有关详细信息，请参阅[在 Azure 资源管理器模板中定义依赖项](/azure-resource-manager/resource-group-define-dependencies.md)一文。

### <a name="role-assignment-resource-information"></a>角色分配资源信息
角色分配资源需要指定的类型和名称。

第一件事要注意是资源的类型不是`Microsoft.Authorization/roleAssignments`因为它将资源组。  相反，资源类型遵循模式`{provider-namespace}/{resource-type}/providers/roleAssignments`。 在这种情况下，资源类型将`Microsoft.DevTestLab/labs/providers/roleAssignments`。

角色分配名称本身需要是全局唯一的。  分配的名称使用模式`{labName}/Microsoft.Authorization/{newGuid}`。 `newGuid`模板的参数值。 它可确保该角色分配名称唯一。 没有用于创建 Guid 模板函数，需要使用任何 GUID 生成器工具自己生成的 GUID。  

在模板中，通过定义角色分配的名称`fullDevTestLabUserRoleName`变量。 从模板的具体行是：

```json
"fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]"
```


### <a name="role-assignment-resource-properties"></a>角色分配资源属性
角色分配本身定义三个属性。 它需要`roleDefinitionId`， `principalId`，和`scope`。

### <a name="role-definition"></a>角色定义
角色定义 ID 是现有的角色定义的字符串标识符。 ID 是在窗体中的角色`/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}`。 

获取的订阅 ID 使用`subscription().subscriptionId`模板函数。  

您需要先获取的角色定义`DevTest Labs User`内置角色。 若要获取的 GUID[开发测试实验室用户](../role-based-access-control/built-in-roles.md#devtest-labs-user)角色，可以使用[角色分配 REST API](/rest/api/authorization/roleassignments)或[Get AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition?view=azps-1.8.0) cmdlet。

```powershell
$dtlUserRoleDefId = (Get-AzRoleDefinition -Name "DevTest Labs User").Id
```

在 variables 节中定义和名为的角色 ID `devTestLabUserRoleId`。 在模板中，角色 ID 设置为：111111111-0000-0000-11111111111111111. 

```json
"devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
```

### <a name="principal-id"></a>主体 ID
主体 ID 为 Active Directory 用户、 组或你想要将作为实验室用户添加到实验室的服务主体的对象 ID。 该模板使用`ObjectId`作为参数。

可以通过获取 ObjectId [Get-azurermaduser](/powershell/module/azurerm.resources/get-azurermaduser?view=azurermps-6.13.0)，[Get-azurermadgroup，或[Get-azurermadserviceprincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal?view=azurermps-6.13.0) PowerShell cmdlet。 这些 cmdlet 返回单个或具有 ID 属性，这是所需的对象 ID 的 Active Directory 对象的列表。 下面的示例演示如何在公司中获取单个用户的对象 ID。

```powershell
$userObjectId = (Get-AzureRmADUser -UserPrincipalName ‘email@company.com').Id
```

此外可以使用包含的 Azure Active Directory PowerShell cmdlet [Get-msoluser](/powershell/module/msonline/get-msoluser?view=azureadps-1.0)， [Get-msolgroup](/powershell/module/msonline/get-msolgroup?view=azureadps-1.0)，并[Get-msolserviceprincipal](/powershell/module/msonline/get-msolserviceprincipal?view=azureadps-1.0)。

### <a name="scope"></a>范围
作用域指定的资源或资源组应为其应用角色分配。 对于资源，范围是在窗体中： `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}`。 该模板使用`subscription().subscriptionId`函数来填充`subscription-id`一部分并`resourceGroup().name`模板函数，以填充`resource-group-name`一部分。 使用这些函数意味着要向其分配角色实验室必须存在于当前订阅和对其进行模板部署的同一资源组。 最后一个部分， `resource-name`，是在实验室的名称。 通过在此示例中的模板参数接收此值。 

在模板中的角色作用域： 

```json
"roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
```

### <a name="deploying-the-template"></a>部署模板
首先，创建参数文件 (例如： azuredeploy.parameters.json) 的资源管理器模板中传递参数的值。 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "value": "11111111-1111-1111-1111-111111111111"
    },
    "labName": {
      "value": "MyLab"
    },
    "roleAssignmentGuid": {
      "value": "22222222-2222-2222-2222-222222222222"
    }
  }
}
```

然后，使用[New-azurermresourcegroupdeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment?view=azurermps-6.13.0) PowerShell cmdlet 部署资源管理器模板。 以下示例命令将个人、 组或服务主体分配到实验室的开发测试实验室用户角色。

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateParameterFile .\azuredeploy.parameters.json -TemplateFile .\azuredeploy.json
```

请务必注意，需要是唯一的组部署名称和角色分配 GUID。 如果你尝试部署的资源分配与非唯一的 GUID，则你将获得`RoleAssignmentUpdateNotPermitted`错误。

如果你打算多次使用模板将多个 Active Directory 对象添加到实验室的开发测试实验室用户角色，请考虑在 PowerShell 命令中使用动态对象。 下面的示例使用[New-guid](/powershell/module/Microsoft.PowerShell.Utility/New-Guid?view=powershell-5.0) cmdlet 来动态指定资源组部署名称和角色分配 GUID。

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateFile .\azuredeploy.json -roleAssignmentGuid "$(New-Guid)" -labName "MyLab" -principalId "11111111-1111-1111-1111-111111111111"
```

## <a name="use-azure-powershell"></a>使用 Azure PowerShell
如简介中所述，创建一个新的 Azure 角色分配，以将用户添加到**开发测试实验室用户**实验室的角色。 在 PowerShell 中，则执行操作来使用[New-azurermroleassignment](/powershell/module/azurerm.resources/new-azurermroleassignment?view=azurermps-6.13.0) cmdlet。 此 cmdlet 具有许多可选参数，使灵活。 `ObjectId`， `SigninName`，或`ServicePrincipalName`可以指定为其授予权限的对象。  

下面是一个示例 Azure PowerShell 命令，将用户添加到指定的实验室中的开发测试实验室用户角色。

```powershell
New-AzureRmRoleAssignment -UserPrincipalName <email@company.com> -RoleDefinitionName 'DevTest Labs User' -ResourceName '<Lab Name>' -ResourceGroupName '<Resource Group Name>' -ResourceType 'Microsoft.DevTestLab/labs'
```

若要指定授予可的组合来指定正在向其权限的资源`ResourceName`， `ResourceType`，`ResourceGroup`或通过`scope`参数。 使用参数的任何组合，提供足够的信息到 cmdlet，以唯一标识 Active Directory 对象 （用户、 组或服务主体）、 作用域 （资源组或资源） 和角色定义。

## <a name="use-azure-command-line-interface-cli"></a>使用 Azure 命令行接口 (CLI)
在 Azure CLI 中，添加到实验室的实验室用户可通过使用`az role assignment create`命令。 Azure CLI cmdlet 的详细信息，请参阅[访问管理对 Azure 资源使用 RBAC 和 Azure CLI](../role-based-access-control/role-assignments-cli.md)。

可以通过指定其授予访问权限的对象`objectId`， `signInName`，`spn`参数。 可以通过标识该对象被授予访问权限的实验室`scope`url 或多种`resource-name`， `resource-type`，和`resource-group`参数。

以下 Azure CLI 示例演示如何为指定的实验室添加到开发测试实验室用户角色的人员。  

```azurecli
az role assignment create --roleName "DevTest Labs User" --signInName <email@company.com> -–resource-name "<Lab Name>" --resource-type “Microsoft.DevTestLab/labs" --resource-group "<Resource Group Name>"
```

## <a name="next-steps"></a>后续步骤
请参阅以下文章：

- [创建和使用开发测试实验室使用 Azure CLI 管理虚拟机](devtest-lab-vmcli.md)
- [使用开发测试实验室使用 Azure PowerShell 创建虚拟机](devtest-lab-vm-powershell.md)
- [使用命令行工具来启动和停止 Azure 开发测试实验室虚拟机](use-command-line-start-stop-virtual-machines.md)

