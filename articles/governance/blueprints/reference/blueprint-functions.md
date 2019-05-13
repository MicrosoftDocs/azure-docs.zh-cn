---
title: Azure 蓝图函数
description: 介绍用于 Azure 蓝图定义和分配的函数。
author: DCtheGeek
ms.author: dacoulte
ms.date: 04/15/2019
ms.topic: reference
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: dc72113a8f5ed978d64d35c43e94dc9e19e4cdb1
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209413"
---
# <a name="functions-for-use-with-azure-blueprints"></a>用于 Azure 蓝图函数

Azure 蓝图提供了生成蓝图定义更动态的函数。 这些函数用于蓝图定义，蓝图项目。 资源管理器模板项目支持完整使用资源管理器除了通过蓝图参数获取动态值的函数。

支持以下函数：

- [artifacts](#artifacts)
- [concat](#concat)
- [parameters](#parameters)
- [resourceGroup](#resourcegroup)
- [resourceGroups](#resourcegroups)
- [subscription](#subscription)

## <a name="artifacts"></a>项目

`artifacts(artifactName)`

返回属性的对象填充该蓝图项目输出。

### <a name="parameters"></a>parameters

| 参数 | 需要 | Type | 描述 |
|:--- |:--- |:--- |:--- |
| artifactName |“是” |string |蓝图项目的名称。 |

### <a name="return-value"></a>返回值

一个输出属性的对象。 **输出**属性是依赖于蓝图项目所引用的类型。 所有类型都遵循格式：

```json
{
  "outputs": {collectionOfOutputProperties}
}
```

#### <a name="policy-assignment-artifact"></a>策略分配项目

```json
{
    "outputs": {
        "policyAssignmentId": "{resourceId-of-policy-assignment}",
        "policyAssignmentName": "{name-of-policy-assignment}",
        "policyDefinitionId": "{resourceId-of-policy-definition}",
    }
}
```

#### <a name="resource-manager-template-artifact"></a>资源管理器模板项目

**输出**返回对象的属性是资源管理器模板中定义，返回的部署。

#### <a name="role-assignment-artifact"></a>角色分配项目

```json
{
    "outputs": {
        "roleAssignmentId": "{resourceId-of-role-assignment}",
        "roleDefinitionId": "{resourceId-of-role-definition}",
        "principalId": "{principalId-role-is-being-assigned-to}",
    }
}
```

### <a name="example"></a>示例

具有 ID 的资源管理器模板项目_myTemplateArtifact_包含下面的示例输出属性：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    ...
    "outputs": {
        "myArray": {
            "type": "array",
            "value": ["first", "second"]
        },
        "myString": {
            "type": "string",
            "value": "my string value"
        },
        "myObject": {
            "type": "object",
            "value": {
                "myProperty": "my value",
                "anotherProperty": true
            }
        }
    }
}
```

从中检索数据的一些示例_myTemplateArtifact_示例是：

| 表达式 | Type | 值 |
|:---|:---|:---|
|`[artifacts("myTemplateArtifact").outputs.myArray]` | 阵列 | \["first", "second"\] |
|`[artifacts("myTemplateArtifact").outputs.myArray[0]]` | String | "第一个" |
|`[artifacts("myTemplateArtifact").outputs.myString]` | String | "我的字符串值" |
|`[artifacts("myTemplateArtifact").outputs.myObject]` | Object | { "myproperty": "my value", "anotherProperty": true } |
|`[artifacts("myTemplateArtifact").outputs.myObject.myProperty]` | String | "我的值" |
|`[artifacts("myTemplateArtifact").outputs.myObject.anotherProperty]` | Bool | True |

## <a name="concat"></a>concat

`concat(string1, string2, string3, ...)`

组合多个字符串值并返回串联的字符串。

### <a name="parameters"></a>parameters

| 参数 | 需要 | Type | 描述 |
|:--- |:--- |:--- |:--- |
| string1 |“是” |string |串联的第一个值。 |
| 其他参数 |“否” |string |按顺序排列的串联的其他值 |

### <a name="return-value"></a>返回值

串联值的字符串。

### <a name="remarks"></a>备注

Azure Blueprint 函数与不同 Azure 资源管理器模板函数，因为它仅适用于字符串。

### <a name="example"></a>示例

`concat(parameters('organizationName'), '-vm')`

## <a name="parameters"></a>parameters

`parameters(parameterName)`

返回一个蓝图参数值。 蓝图定义中或蓝图项目，必须定义指定的参数名称。

### <a name="parameters"></a>parameters

| 参数 | 需要 | Type | 描述 |
|:--- |:--- |:--- |:--- |
| parameterName |“是” |string |要返回的参数名称。 |

### <a name="return-value"></a>返回值

指定的蓝图或蓝图项目参数的值。

### <a name="remarks"></a>备注

Azure Blueprint 函数与不同 Azure 资源管理器模板函数，因为它仅适用于蓝图参数。

### <a name="example"></a>示例

定义参数_principalIds_蓝图定义中：

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "parameters": {
            "principalIds": {
                "type": "array",
                "metadata": {
                    "displayName": "Principal IDs",
                    "description": "This is a blueprint parameter that any artifact can reference. We'll display these descriptions for you in the info bubble. Supply principal IDs for the users,groups, or service principals for the RBAC assignment.",
                    "strongType": "PrincipalId"
                }
            }
        },
        ...
    }
}
```

然后，使用_principalIds_为参数`parameters()`蓝图项目中：

```json
{
    "type": "Microsoft.Blueprint/blueprints/artifacts",
    "kind": "roleAssignment",
    ...
    "properties": {
        "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
        "principalIds": "[parameters('principalIds')]",
        ...
    }
}
```

## <a name="resourcegroup"></a>resourceGroup

`resourceGroup()`

返回表示当前资源组的对象。

### <a name="return-value"></a>返回值

返回的对象采用以下格式：

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="remarks"></a>备注

Azure Blueprint 函数不同于 Azure 资源管理器模板函数。 `resourceGroup()`函数不能使用在订阅级别的项目或蓝图定义中。 它仅可在资源组项目一部分的蓝图项目。

一个常见用途`resourceGroup()`函数是在资源组项目所在的同一位置中创建的资源。

### <a name="example"></a>示例

若要使用资源组的位置，在蓝图定义或设置期间分配，作为另一个项目的位置声明蓝图定义中的资源组占位符对象。 在此示例中， _NetworkingPlaceholder_是资源组占位符的名称。

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

然后，使用`resourceGroup()`蓝图项目定目标到的资源组占位符对象的上下文中的函数。 在此示例中，模板项目部署到_NetworkingPlaceholder_资源组，并提供参数_resourceLocation_动态填充_NetworkingPlaceholder_到模板的资源组位置。 位置_NetworkingPlaceholder_资源组可能已蓝图定义中静态定义或分配期间动态定义。 在任一情况下，模板项目提供该信息作为参数，并使用它来将资源部署到正确的位置。

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "template",
  "properties": {
      "template": {
        ...
      },
      "resourceGroup": "NetworkingPlaceholder",
      ...
      "parameters": {
        "resourceLocation": {
          "value": "[resourceGroup().location]"
        }
      }
  }
}
```

## <a name="resourcegroups"></a>resourceGroups

`resourceGroups(placeholderName)`

返回一个对象，表示指定的资源组项目。 与不同`resourceGroup()`，这需要的项目的上下文，此函数用于获取特定资源时不在上下文中该资源组的组占位符的属性。

### <a name="parameters"></a>parameters

| 参数 | 需要 | Type | 描述 |
|:--- |:--- |:--- |:--- |
| placeholderName |“是” |string |要返回的资源组项目的占位符名称。 |

### <a name="return-value"></a>返回值

返回的对象采用以下格式：

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="example"></a>示例

若要使用资源组的位置，在蓝图定义或设置期间分配，作为另一个项目的位置声明蓝图定义中的资源组占位符对象。 在此示例中， _NetworkingPlaceholder_是资源组占位符的名称。

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

然后，使用`resourceGroups()`函数要获取对资源组占位符对象的引用任何蓝图项目的上下文中。 在此示例中，模板项目部署之外_NetworkingPlaceholder_资源组，并提供参数_artifactLocation_动态填充_NetworkingPlaceholder_到模板的资源组位置。 位置_NetworkingPlaceholder_资源组可能已蓝图定义中静态定义或分配期间动态定义。 在任一情况下，模板项目提供该信息作为参数，并使用它来将资源部署到正确的位置。

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "artifactLocation": {
          "value": "[resourceGroups('NetworkingPlaceholder').location]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="subscription"></a>订阅

`subscription()`

返回有关当前蓝图分配的订阅的详细信息。

### <a name="return-value"></a>返回值

返回的对象采用以下格式：

```json
{
    "id": "/subscriptions/{subscriptionId}",
    "subscriptionId": "{subscriptionId}",
    "tenantId": "{tenantId}",
    "displayName": "{name-of-subscription}"
}
```

### <a name="example"></a>示例

使用订阅的显示名称和`concat()`函数来创建作为参数传递的命名约定_resourceName_对模板项目。

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "resourceName": {
          "value": "[concat(subscription().displayName, '-vm')]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="next-steps"></a>后续步骤

- 了解[蓝图生命周期](../concepts/lifecycle.md)。
- 了解如何使用[静态和动态参数](../concepts/parameters.md)。
- 了解如何自定义[蓝图排序顺序](../concepts/sequencing-order.md)。
- 了解如何利用[蓝图资源锁定](../concepts/resource-locking.md)。
- 了解如何[更新现有分配](../how-to/update-existing-assignments.md)。
- 使用[一般故障排除](../troubleshoot/general.md)在蓝图的分配期间解决问题。