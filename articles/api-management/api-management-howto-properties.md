---
title: 如何在 Azure API 管理策略中使用命名值
description: 了解如何在 Azure API 管理策略中使用命名值。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/25/2018
ms.author: apimpm
ms.openlocfilehash: 478b80b021b4df36e2eccc37ac9c74f75e43a5bb
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60657977"
---
# <a name="how-to-use-named-values-in-azure-api-management-policies"></a>如何在 Azure API 管理策略中使用命名值
API 管理策略是一项强大的系统功能，允许 Azure 门户通过配置更改 API 的行为。 策略是一组语句，在请求或 API 的响应时按顺序执行。 可以使用文字文本值、策略表达式和命名值构造策略语句。 

每个 API 管理服务实例都有一个属性集合，其中包含对服务实例来说属于全局性的键值对（称为“命名值”）。 这些命名值可以用来管理所有 API 配置和策略的常量字符串值。 每个属性都有以下特性：

| 属性 | Type | 描述 |
| --- | --- | --- |
| Display name |string |策略中用于引用属性的数字字母字符串。 |
| Value |string |属性的值。 不能为空或只由空格组成。 |
|Secret|boolean|确定值是否为密钥以及是否应加密。|
| Tags |字符串数组 |可选标记，提供用来筛选属性列表。 |

![命名值](./media/api-management-howto-properties/named-values.png)

属性值可以包含文本字符串和[策略表达式](/azure/api-management/api-management-policy-expressions)。 例如，`ExpressionProperty` 的值是一个策略表达式，其返回的字符串包含当前日期和时间。 属性 `ContosoHeaderValue` 被标记为密钥，因此不显示其值。

| 名称 | Value | Secret | Tags |
| --- | --- | --- | --- |
| ContosoHeader |TrackingId |False |Contoso |
| ContosoHeaderValue |•••••••••••••••••••••• |True |Contoso |
| ExpressionProperty |@(DateTime.Now.ToString()) |False | |

## <a name="to-add-and-edit-a-property"></a>添加和编辑属性

![添加属性](./media/api-management-howto-properties/add-property.png)

1. 在“API 管理”下面选择“API”。
2. 选择“命名值”。
3. 按“+添加”。

   “名称”和“值”是必需值。 如果此属性值为机密，请选中“这是机密”复选框。 输入一个或多个用于组织命名值的可选标记，并单击“保存”。
4. 单击**创建**。

创建属性后，可以通过单击该属性对其进行编辑。 如果更改属性名称，则会自动更新引用该属性的策略，让其使用新名称。

若要了解如何使用 REST API 编辑属性，请参阅 [Edit a property using the REST API](/rest/api/apimanagement/property?Patch)（使用 REST API 编辑属性）。

## <a name="to-delete-a-property"></a>删除属性

若要删除属性，请单击要删除的属性旁边的“删除”。

> [!IMPORTANT]
> 如果有策略引用了该属性，则无法成功地将该属性删除，除非将它从所有使用它的策略中移除。
> 
> 

若要了解如何使用 REST API 删除属性，请参阅 [Delete a property using the REST API](/rest/api/apimanagement/property?Delete)（使用 REST API 删除属性）。

## <a name="to-search-and-filter-named-values"></a>搜索和筛选命名值

“命名值”选项卡包含搜索和筛选功能，用于管理命名值。 若要按属性名称筛选属性列表，请在“搜索属性”文本框中输入搜索词。 若要显示所有命名值，请清除“搜索属性”文本框，并按 Enter。

要按标记值筛选属性列表，请将一个或多个标记输入到“按标记筛选”文本框中。 若要显示所有命名值，请清除“按标记筛选”文本框，并按 Enter。

## <a name="to-use-a-property"></a>使用属性

要在策略中使用属性，请将属性名称置于一对双大括号中（例如 `{{ContosoHeader}}`），如以下示例所示：

```xml
<set-header name="{{ContosoHeader}}" exists-action="override">
  <value>{{ContosoHeaderValue}}</value>
</set-header>
```

在此示例中，`ContosoHeader` 用作 `set-header` 策略中一个标头的名称，`ContosoHeaderValue` 用作该标头的值。 在请求或响应 API 管理网关的过程中，如果要对该策略进行评估，则会将 `{{ContosoHeader}}` 和 `{{ContosoHeaderValue}}` 替换为各自的属性值。

命名值可以用作完整的特性或元素值（如前面的示例所示），但也可插入到文字文本表达式中或与该表达式的一部分组合在一起，如以下示例所示：`<set-header name = "CustomHeader{{ContosoHeader}}" ...>`

命名值还可以包含策略表达式。 以下示例使用 `ExpressionProperty`。

```xml
<set-header name="CustomHeader" exists-action="override">
    <value>{{ExpressionProperty}}</value>
</set-header>
```

评估此策略时，会将 `{{ExpressionProperty}}` 替换为其值：`@(DateTime.Now.ToString())`。 由于值为策略表达式，因此会计算该表达式，然后策略会继续执行相关操作。

若要在开发人员门户中对此进行测试，可调用一个其策略命名值处于范围内的操作。 在以下示例中，调用了一个包含两个带命名值的前述示例性 `set-header` 策略的操作。 请注意，响应包含两个自定义标头，这两个标头是使用带命名值的策略配置的。

![开发人员门户][api-management-send-results]

如果在 [API 检查器跟踪](api-management-howto-api-inspector.md)中查看包含两个带命名值的前述示例性策略的调用，则可以看到两个 `set-header` 策略，这两个策略已插入了属性值，并已针对包含策略表达式的属性进行了策略表达式计算。

![API 检查器跟踪][api-management-api-inspector-trace]

虽然属性值可以包含策略表达式，但不能包含其他命名值。 如果使用包含属性引用的文本作为属性值（例如 `Property value text {{MyProperty}}`），则不会替换该属性引用，而会将其作为属性值的一部分包括进去。

## <a name="next-steps"></a>后续步骤
* 详细了解如何使用策略
  * [API 管理中的策略](api-management-howto-policies.md)
  * [策略参考](/azure/api-management/api-management-policies)
  * [策略表达式](/azure/api-management/api-management-policy-expressions)

[api-management-send-results]: ./media/api-management-howto-properties/api-management-send-results.png
[api-management-properties-filter]: ./media/api-management-howto-properties/api-management-properties-filter.png
[api-management-api-inspector-trace]: ./media/api-management-howto-properties/api-management-api-inspector-trace.png

