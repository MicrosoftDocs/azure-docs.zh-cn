---
title: 策略定义结构的详细信息
description: 介绍 Azure Policy 如何使用资源策略定义，通过描述何时强制实施策略和要实现的效果为组织中的资源建立约定。
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/13/2019
ms.topic: conceptual
ms.service: azure-policy
manager: carmonm
ms.custom: seodec18
ms.openlocfilehash: 0783251eaeef188c49c5b3aa61b5ecaec48127b7
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506694"
---
# <a name="azure-policy-definition-structure"></a>Azure Policy 定义结构

Azure Policy 使用资源策略定义来建立资源约定。 每个定义描述资源符合性，以及在资源不符合的情况下会产生什么影响。
通过定义约定，可以控制成本并更轻松地管理资源。 例如，可指定仅允许特定类型的虚拟机。 或者，可要求所有资源都拥有特定标记。 策略由所有子资源继承。 如果将策略应用到资源组，则会将其应用到该资源组中的所有资源。

可以在此处找到 Azure Policy 使用的架构：[https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json](https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json)

使用 JSON 创建策略定义。 策略定义包含以下项的元素：

- 模式
- parameters
- 显示名称
- description
- 策略规则
  - 逻辑评估
  - 效果

例如，以下 JSON 说明限制资源部署位置的策略：

```json
{
    "properties": {
        "mode": "all",
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                },
                "defaultValue": [ "westus2" ]
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

所有 Azure 策略示例都位于[Azure 策略示例](../samples/index.md)。

[!INCLUDE [az-powershell-update](../../../../includes/updated-for-az.md)]

## <a name="mode"></a>模式

**模式**确定将对策略评估哪些资源类型。 支持的模式包括：

- `all`：评估资源组和所有资源类型
- `indexed`：仅评估支持标记和位置的资源类型

大多数情况下，建议将“mode”设置为 `all`。 通过门户创建的所有策略定义使用 `all` 模式。 如果使用 PowerShell 或 Azure CLI，则可以手动指定 **mode** 参数。 如果策略定义不包含 **mode** 值，为提供后向兼容性，在 Azure PowerShell 中默认为 `all`，在 Azure CLI 中默认为 `null`。 `null` 模式等同于使用 `indexed` 来支持后向兼容性。

在创建强制执行标记或位置的策略时，应该使用 `indexed`。 虽然并不是必需的，但是它会阻止不支持标记和位置的资源，使其不会在符合性结果中显示为不兼容。 资源组是一个例外。 在资源组上强制执行位置或标记的策略应将“mode”设为 `all`，并专门针对 `Microsoft.Resources/subscriptions/resourceGroups` 类型。 请在[强制执行资源组标记](../samples/enforce-tag-rg.md)查看相关示例。 如需支持标记的资源的列表，请参阅 [Azure 资源的标记支持](../../../azure-resource-manager/tag-support.md)。

## <a name="parameters"></a>parameters

参数可减少策略定义的数量，有助于简化策略管理。 使用类似窗体中字段的参数 - `name`、`address`、`city`、`state`。 这些参数始终不变，但其值会基于窗体中的各填写内容变化。
构建策略时，参数同样适用。 如果在策略定义中包括参数，就可以通过使用不同的值重新使用策略以执行不同方案。

> [!NOTE]
> 参数可以添加到现有和已分配的定义。 新参数必须包含 defaultValue 属性。 这可以防止策略或计划的现有分配间接被设为无效。

### <a name="parameter-properties"></a>参数属性

参数有下述可以在策略定义中使用的属性：

- **名称**：参数的名称。 由策略规则中的 `parameters` 部署函数使用。 有关详细信息，请参阅[使用参数值](#using-a-parameter-value)。
- `type`：确定参数是**字符串**还是**数组**。
- `metadata`：定义主要由 Azure 门户用来显示用户友好信息的子属性：
  - `description`：说明参数的用途。 可以用来提供可接受值的示例。
  - `displayName`：在门户中显示的用于参数的友好名称。
  - `strongType`：（可选）通过门户分配策略定义时使用。 提供上下文感知列表。 有关详细信息，请参阅 [strongType](#strongtype)。
  - `assignPermissions`：（可选）将设置为 _，则返回 true_能够在分配策略期间创建角色分配的 Azure 门户。 在您想要分配的分配作用域之外的权限的情况下，此属性很有用。 没有一个角色分配每个策略中的角色定义 （或每个角色定义中的所有计划的策略）。 参数值必须是有效的资源或作用域。
- `defaultValue`：（可选）设置分配的参数的值（如果值未给定）。 在更新已分配的现有策略定义时必须使用此项。
- `allowedValues`：（可选）提供了一个参数接受在分配过程的值数组。

例如，可以定义策略定义来限制资源的部署位置。 **allowedLocations** 可以是该策略定义的一个参数。 每次分配策略定义来限制接受的值时，会使用此参数。 使用 **strongType** 可以在通过门户完成分配时提供增强的体验：

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="using-a-parameter-value"></a>使用参数值

在策略规则中，使用以下 `parameters` 部署值函数语法引用参数：

```json
{
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

此示例引用 **allowedLocations** 参数，该参数已在[参数属性](#parameter-properties)中演示过。

### <a name="strongtype"></a>strongType

在 `metadata` 属性中，可以使用 **strongType** 提供 Azure 门户中的选项多选列表。 **strongType** 的允许值目前包括：

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`
- `omsWorkspace`
- `Microsoft.EventHub/Namespaces/EventHubs`
- `Microsoft.EventHub/Namespaces/EventHubs/AuthorizationRules`
- `Microsoft.EventHub/Namespaces/AuthorizationRules`
- `Microsoft.RecoveryServices/vaults`
- `Microsoft.RecoveryServices/vaults/backupPolicies`

## <a name="definition-location"></a>定义位置

创建计划或策略时，需要指定定义位置。 定义位置必须是管理组或订阅。 此位置决定了计划或策略的分配范围。 资源必须是用于分配的目标定义位置的层次结构中的直系成员或子代。

如果定义位置是：

- **订阅** - 只能将该订阅中的资源分配给策略。
- **管理组** - 只能将子管理组和子订阅中的资源分配给策略。 如果计划将策略定义应用于多个订阅，则位置必须是包含那些订阅的管理组。

## <a name="display-name-and-description"></a>显示名称和说明

请使用“displayName”和“description”来标识策略定义，并提供其使用上下文。 **displayName** 的最大长度为 128 个字符，**description** 的最大长度为 512 个字符。

## <a name="policy-rule"></a>策略规则

策略规则包括 **If** 和 **Then** 块。 在 **If** 块中，定义强制执行策略时指定的一个或多个条件。 可以对这些条件应用逻辑运算符，以精确定义策略的方案。

在 **Then** 块中，定义满足 **If** 条件时产生的效果。

```json
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | append | auditIfNotExists | deployIfNotExists | disabled"
    }
}
```

### <a name="logical-operators"></a>逻辑运算符

支持的逻辑运算符为：

- `"not": {condition  or operator}`
- `"allOf": [{condition or operator},{condition or operator}]`
- `"anyOf": [{condition or operator},{condition or operator}]`

**not** 语法反转条件的结果。 **allOf** 语法（与逻辑 **And** 操作相似）要求所有条件为 true。 **anyOf** 语法（与逻辑 **Or** 操作相似）要求一个或多个条件为 true。

可以嵌套逻辑运算符。 以下示例显示了嵌套在 allOf 操作中的 not 操作。

```json
"if": {
    "allOf": [{
            "not": {
                "field": "tags",
                "containsKey": "application"
            }
        },
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        }
    ]
},
```

### <a name="conditions"></a>条件

条件用于评估 **field** 或 **value** 访问器是否符合特定标准。 支持的条件有：

- `"equals": "value"`
- `"notEquals": "value"`
- `"like": "value"`
- `"notLike": "value"`
- `"match": "value"`
- `"matchInsensitively": "value"`
- `"notMatch": "value"`
- `"notMatchInsensitively": "value"`
- `"contains": "value"`
- `"notContains": "value"`
- `"in": ["value1","value2"]`
- `"notIn": ["value1","value2"]`
- `"containsKey": "keyName"`
- `"notContainsKey": "keyName"`
- `"less": "value"`
- `"lessOrEquals": "value"`
- `"greater": "value"`
- `"greaterOrEquals": "value"`
- `"exists": "bool"`

使用 like 和 notLike 条件时，请在值中指定通配符 `*`。
值不应包含多个通配符 `*`。

当使用 match 和 notMatch 条件时，请提供 `#` 来匹配数字，提供 `?` 来表示字母，提供 `.` 来匹配所有字符，并提供任何其他字符来匹配该实际字符。
“match”和“notMatch”区分大小写。 “matchInsensitively”和“notMatchInsensitively”中提供了不区分大小写的替代方案。 例如，请参阅[允许多个名称模式](../samples/allow-multiple-name-patterns.md)。

### <a name="fields"></a>字段

使用字段构成条件。 字段匹配资源请求有效负载中的属性，并说明资源的状态。

支持以下字段：

- `name`
- `fullName`
  - 返回资源全名。 资源全名是最前面为任意父资源名称的资源名称（例如“myServer/myDatabase”）。
- `kind`
- `type`
- `location`
  - 对于不限位置的资源，请使用 **global**。 如需示例，请参阅[示例 - 允许的位置](../samples/allowed-locations.md)。
- `identity.type`
  - 返回在资源上启用的[托管标识](../../../active-directory/managed-identities-azure-resources/overview.md)类型。
- `tags`
- `tags['<tagName>']`
  - 此括号语法支持具有标点符号的标记名称，例如连字符、句点或空格。
  - 其中 **\<tagName\>** 是要验证其条件的标记的名称。
  - 示例：`tags['Acct.CostCenter']`，其中 Acct.CostCenter 是标记的名称。
- `tags['''<tagName>''']`
  - 此括号语法通过双撇号进行转义，可支持在其中包含撇号的标记名称。
  - 其中“\<tagName\>”是要验证其条件的标记的名称。
  - 示例：`tags['''My.Apostrophe.Tag''']`，其中“\<tagName\>”是标记的名称。
- 属性别名 - 有关列表，请参阅[别名](#aliases)。

> [!NOTE]
> `tags.<tagName>``tags[tagName]` 和 `tags[tag.with.dots]` 仍然是可接受的用于声明标记字段的方式。
> 但是，首选表达式是上面列出的那些。

#### <a name="use-tags-with-parameters"></a>使用带参数的标记

参数值可以传递给标记字段。 将参数传递给标记字段可在策略分配期间提高策略定义的灵活性。

在以下示例中，`concat` 用于为名为 tagName 参数值的标记创建标记字段查找。 如果该标记不存在，则使用“追加”效果来添加该标记，该效果使用 `resourcegroup()` 查找函数在审计资源父资源组上使用同一命名标记集的值来添加该标记。

```json
{
    "if": {
        "field": "[concat('tags[', parameters('tagName'), ']')]",
        "exists": "false"
    },
    "then": {
        "effect": "append",
        "details": [{
            "field": "[concat('tags[', parameters('tagName'), ']')]",
            "value": "[resourcegroup().tags[parameters('tagName')]]"
        }]
    }
}
```

### <a name="value"></a>值

也可使用 **value** 来形成条件。 **value** 会针对[参数](#parameters)、[支持的模板函数](#policy-functions)或文本来检查条件。
**value** 可与任何支持的[条件](#conditions)配对。

> [!WARNING]
> 如果模板函数的结果是错误，策略评估将会失败。 失败的评估是一种隐式**拒绝**。 有关详细信息，请参阅[避免模板错误](#avoiding-template-failures)。

#### <a name="value-examples"></a>Value 示例

此策略规则示例使用 **value** 将 `resourceGroup()` 函数和返回的 **name** 属性的结果与 **like** 条件 `*netrg` 进行对比。 此规则拒绝名称以 `*netrg` 结尾的资源组中 **type** 不为 `Microsoft.Network/*` 的资源。

```json
{
    "if": {
        "allOf": [{
                "value": "[resourceGroup().name]",
                "like": "*netrg"
            },
            {
                "field": "type",
                "notLike": "Microsoft.Network/*"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

此策略规则示例使用 **value** 来检查多个嵌套函数的结果是否 **equals** `true`。 此规则拒绝并没有至少三个标记的资源。

```json
{
    "mode": "indexed",
    "policyRule": {
        "if": {
            "value": "[less(length(field('tags')), 3)]",
            "equals": true
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

#### <a name="avoiding-template-failures"></a>避免模板错误

在 **value** 中使用模板函数可以指定许多复杂的嵌套函数。 如果模板函数的结果是错误，策略评估将会失败。 失败的评估是一种隐式**拒绝**。 在特定情况下失败的 **value** 示例：

```json
{
    "policyRule": {
        "if": {
            "value": "[substring(field('name'), 0, 3)]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

上面的示例策略规则使用 [substring()](../../../azure-resource-manager/resource-group-template-functions-string.md#substring) 将 **name** 的前三个字符与 **abc** 进行比较。 如果 **name** 短于三个字符，`substring()` 函数会导致出错。 此错误导致策略成为一种 **deny**（拒绝）效果。

改用 [if()](../../../azure-resource-manager/resource-group-template-functions-logical.md#if) 函数来检查 **name** 的前三个字符是否等于 **abc**，同时避免短于三个字符的 **name** 导致出错：

```json
{
    "policyRule": {
        "if": {
            "value": "[if(greaterOrEquals(length(field('name')), 3), substring(field('name'), 0, 3), 'not starting with abc')]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

`if()` 使用修改后的策略规则检查 **name** 的长度，然后尝试在短于三个字符的值中获取 `substring()`。 如果 **name** 过短，则会返回“not starting with abc”，而不是与 **abc** 进行比较。 短名称不是以 **abc** 开头的资源仍不符合策略规则，但不再在评估期间导致出错。

### <a name="effect"></a>效果

Azure 策略支持以下类型的效果：

- **Deny**：会在活动日志中生成一个事件，并使请求失败
- **Audit**：会在活动日志中生成一个警告事件，但不会使请求失败
- **Append**：会将定义的字段集添加到请求
- **AuditIfNotExists**：如果资源不存在，则启用审核
- **DeployIfNotExists**：如果资源不存在，则部署一个资源
- **Disabled**：不评估资源是否符合策略规则

对于 **append**，必须提供以下详细信息：

```json
"effect": "append",
"details": [{
    "field": "field name",
    "value": "value of the field"
}]
```

值可以是字符串或 JSON 格式对象。

AuditIfNotExists 和 DeployIfNotExists 评估相关的资源是否存在，并应用规则。 如果资源与规则不匹配，则会实现效果。 例如，可以要求为所有虚拟网络部署网络观察程序。 有关更多信息，请参阅[在扩展不存在的情况下审核](../samples/audit-ext-not-exist.md)示例。

**DeployIfNotExists** 效果需要策略规则的 **details** 部分中的 **roleDefinitionId** 属性。 有关详细信息，请参阅[修正 - 配置策略定义](../how-to/remediate-resources.md#configure-policy-definition)。

```json
"details": {
    ...
    "roleDefinitionIds": [
        "/subscription/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
        "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
    ]
}
```

有关完整详细信息每个效果，顺序评估、 属性和示例，请参阅[了解 Azure 策略效果](effects.md)。

### <a name="policy-functions"></a>策略函数

所有[资源管理器模板函数](../../../azure-resource-manager/resource-group-template-functions.md)均可用于在策略规则，除以下函数和用户定义的函数中：

- copyIndex()
- deployment()
- list*
- newGuid()
- pickZones()
- providers()
- reference()
- resourceId()
- variables()

以下函数是可用于在策略规则中，使用但不同于在 Azure 资源管理器模板中使用：

- addDays(dateTime, numberOfDaysToAdd)
  - **dateTime**: [必需] 字符串-通用 ISO 8601 日期时间格式字符串 yyyy-MM-ddTHH:mm:ss.fffffffZ
  - **numberOfDaysToAdd**: [必需] 整数-要添加的天数
- utcnow （)-与 Resource Manager 模板，这可以使用外部 defaultValue。
  - 返回设置为当前日期和时间以通用的 ISO 8601 日期时间格式的字符串 yyyy-MM-ddTHH:mm:ss.fffffffZ

此外，`field` 函数可用于策略规则。 `field` 主要用于 **AuditIfNotExists** 和 **DeployIfNotExists**，以引用所评估资源上的字段。 可以在 [DeployIfNotExists 示例](effects.md#deployifnotexists-example)中看到这种用法的示例。

#### <a name="policy-function-example"></a>策略函数示例

此策略规则示例使用 `resourceGroup` 资源函数获取 **name** 属性，并将该属性与 `concat` 数组和对象函数结合使用以构建 `like` 条件，该条件强制资源名称以资源组名称开头。

```json
{
    "if": {
        "not": {
            "field": "name",
            "like": "[concat(resourceGroup().name,'*')]"
        }
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="aliases"></a>别名

可以使用属性别名来访问资源类型的特定属性。 通过别名，可限制允许用于资源属性的值和条件。 每个别名会映射到给定资源类型不同 API 版本的路径。 在策略评估期间，策略引擎会获取该 API 版本的属性路径。

别名列表始终不断增长。 若要找出 Azure Policy 当前支持哪些别名，请使用以下方法之一：

- Azure PowerShell

  ```azurepowershell-interactive
  # Login first with Connect-AzAccount if not using Cloud Shell

  # Use Get-AzPolicyAlias to list available providers
  Get-AzPolicyAlias -ListAvailable

  # Use Get-AzPolicyAlias to list aliases for a Namespace (such as Azure Automation -- Microsoft.Automation)
  Get-AzPolicyAlias -NamespaceMatch 'automation'
  ```

- Azure CLI

  ```azurecli-interactive
  # Login first with az login if not using Cloud Shell

  # List namespaces
  az provider list --query [*].namespace

  # Get Azure Policy aliases for a specific Namespace (such as Azure Automation -- Microsoft.Automation)
  az provider show --namespace Microsoft.Automation --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
  ```

- REST API/ARMClient

  ```http
  GET https://management.azure.com/providers/?api-version=2017-08-01&$expand=resourceTypes/aliases
  ```

### <a name="understanding-the--alias"></a>了解 [*] 别名

可用的几个别名的版本显示为“普通”名称，另一个版本的名称则附加了 **[\*]**。 例如:

- `Microsoft.Storage/storageAccounts/networkAcls.ipRules`
- `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]`

“normal”别名表示单一值字段。 如果整个值集必须确切定义（不能多定义，也不能少定义），则此字段将用于完全匹配比较方案。

使用 **[\*]** 别名可以比较数组中的每个元素值以及每个元素的特定属性。 使用这种方法可以比较“if none of”、“if any of”或“if all of”方案的元素属性。 使用 **ipRules [\*]** 时，某个示例将会验证每个 _action_ 是否为 _Deny_，但不考虑存在多少个规则，或 IP 的 _value_ 是什么。 此示例规则检查 **10.0.4.1** 的所有 **ipRules [\*].value** 匹配项，仅当至少未找到一个匹配项时，才应用 **effectType**：

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value",
                "notEquals": "10.0.4.1"
            }
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```

有关详细信息，请参阅[评估 [\*] 别名](../how-to/author-policies-for-arrays.md#evaluating-the--alias)。

## <a name="initiatives"></a>计划

使用计划可组合多个相关策略定义，以简化分配和管理，因为可将组作为单个项使用。 例如，可以将相关标记策略组合为单个计划。 将应用计划，而非单独分配每个策略。

下面的示例演示如何创建用于处理 `costCenter` 和 `productName` 这两个标记的计划。 它使用两个内置策略来应用默认标记值。

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                }
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                }
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    },
    "id": "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policySetDefinitions/billingTagsPolicy",
    "type": "Microsoft.Authorization/policySetDefinitions",
    "name": "billingTagsPolicy"
}
```

## <a name="next-steps"></a>后续步骤

- 查看示例[Azure 策略示例](../samples/index.md)。
- 查看[了解策略效果](effects.md)。
- 了解如何[以编程方式创建策略](../how-to/programmatically-create.md)。
- 了解如何[获取符合性数据](../how-to/getting-compliance-data.md)。
- 了解如何[修正的不合规资源](../how-to/remediate-resources.md)。
- 查看管理组与[使用 Azure 管理组组织资源](../../management-groups/overview.md)。