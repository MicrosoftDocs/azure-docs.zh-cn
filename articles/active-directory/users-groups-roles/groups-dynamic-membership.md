---
title: 动态自动组成员资格规则 - Azure Active Directory | Microsoft Docs
description: 如何创建成员资格规则以自动填充组和规则引用。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: article
ms.date: 01/31/2019
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: eebb68218fd6f9cbda229aae3d9e544e87441562
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192437"
---
# <a name="dynamic-membership-rules-for-groups-in-azure-active-directory"></a>Azure Active Directory 中的动态组成员资格规则

在 Azure Active Directory (Azure AD) 中，可以创建基于属性的复杂规则以启用组的动态成员身份。 动态组成员资格可减少添加和删除用户的管理开销。 本文详细介绍了用于为用户或设备创建动态成员资格规则的属性和语法。 可以为安全组或 Office 365 组中的动态成员身份设置规则。

当用户或设备的任何属性发生更改时，系统会评估目录中的所有动态组规则，以查看该更改是否会触发任何组添加或删除。 如果用户或设备满足组的规则，它们将添加为该组的成员。 如果用户或设备不再满足该规则，则会将其删除。 无法手动添加或删除动态组的成员。

* 可以创建设备或用户的动态组，但无法创建同时包含用户和设备的规则。
* 无法根据设备所有者的属性创建设备组。 设备成员资格规则只能引用设备属性。

> [!NOTE]
> 对于每一个作为一个或多个动态组成员的唯一用户，此功能需要 Azure AD Premium P1 许可证。 无需将许可证分配给用户使其成为动态组成员，但必须在租户中具有涵盖所有此类用户所需的最小许可证数。 例如：如果在租户的所有动态组中总共拥有 1,000 个唯一用户，则需要至少具有 1,000 个 Azure AD Premium P1 版的许可证，才能满足许可证要求。
>

## <a name="constructing-the-body-of-a-membership-rule"></a>构造成员资格规则的主体

使用用户或设备自动填充组的成员资格规则是一个二进制表达式，会生成 true 或 false 结果。 一个简单的规则包含三个部分：

* 属性
* 运算符
* 值

表达式中各部分的顺序对于避免语法错误至关重要。

### <a name="rules-with-a-single-expression"></a>带有单个表达式的规则

单个表达式是成员资格规则的最简单形式，只包括上述的三个部分。 具有单个表达式的规则与此类似：`Property Operator Value`，其中属性的语法是 object.property 的名称。

以下是使用单个表达式正确构造的成员资格规则的示例：

```
user.department -eq "Sales"
```

对于单个表达式，括号是可选的。 成员资格规则正文的总长度不能超过 2048 个字符。

## <a name="supported-properties"></a>支持的属性

有三种类型的属性可用于构建成员资格规则。

* Boolean
* String
* 字符串集合

以下是可用于创建单个表达式的用户属性。

### <a name="properties-of-type-boolean"></a>布尔值类型的属性

| 属性 | 允许的值 | 使用情况 |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true |
| dirSyncEnabled |true false |user.dirSyncEnabled -eq true |

### <a name="properties-of-type-string"></a>字符串类型的属性

| 属性 | 允许的值 | 使用情况 |
| --- | --- | --- |
| city |任意字符串值或 null |(user.city -eq "value") |
| country |任意字符串值或 null |(user.country -eq "value") |
| companyName | 任意字符串值或 null | (user.companyName -eq "value") |
| department |任意字符串值或 null |(user.department -eq "value") |
| displayName |任意字符串值 |(user.displayName -eq "value") |
| employeeId |任意字符串值 |(user.employeeId -eq "value")<br>(user.employeeId -ne *null*) |
| facsimileTelephoneNumber |任意字符串值或 null |(user.facsimileTelephoneNumber -eq "value") |
| givenName |任意字符串值或 null |(user.givenName -eq "value") |
| jobTitle |任意字符串值或 null |(user.jobTitle -eq "value") |
| mail |任意字符串值或 null（用户的 SMTP 地址） |(user.mail -eq "value") |
| mailNickName |任意字符串值（用户的邮件别名） |(user.mailNickName -eq "value") |
| mobile |任意字符串值或 null |(user.mobile -eq "value") |
| objectId |用户对象的 GUID。 |(user.objectId -eq "11111111-1111-1111-1111-111111111111") |
| onPremisesSecurityIdentifier | 从本地同步至云端的用户的本地安全标识符 (SID)。 |(user.onPremisesSecurityIdentifier -eq "S-1-1-11-1111111111-1111111111-1111111111-1111111") |
| passwordPolicies |None DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |任意字符串值或 null |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |任意字符串值或 null |(user.postalCode -eq "value") |
| preferredLanguage |ISO 639-1 代码 |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |任意字符串值或 null |(user.sipProxyAddress -eq "value") |
| state |任意字符串值或 null |(user.state -eq "value") |
| streetAddress |任意字符串值或 null |(user.streetAddress -eq "value") |
| surname |任意字符串值或 null |(user.surname -eq "value") |
| telephoneNumber |任意字符串值或 null |(user.telephoneNumber -eq "value") |
| usageLocation |双字母国家/地区代码 |(user.usageLocation -eq "US") |
| userPrincipalName |任意字符串值 |(user.userPrincipalName -eq "alias@domain") |
| userType |member guest null |(user.userType -eq "Member") |

### <a name="properties-of-type-string-collection"></a>字符串集合类型的属性

| 属性 | 允许的值 | 使用情况 |
| --- | --- | --- |
| otherMails |任意字符串值 |(user.otherMails -contains "alias@domain") |
| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

有关用于设备规则的属性，请参阅[设备规则](#rules-for-devices)。

## <a name="supported-operators"></a>支持的运算符

下表列出了单个表达式支持的所有运算符及其语法。 运算符可以带或不带连字符 (-) 前缀。

| 运算符 | 语法 |
| --- | --- |
| 不等于 |-ne |
| 等于 |-eq |
| 开头不为 |-notStartsWith |
| 开头为 |-startsWith |
| 不包含 |-notContains |
| Contains |-contains |
| 不匹配 |-notMatch |
| 匹配 |-match |
| In | -in |
| 不位于 | -notIn |

### <a name="using-the--in-and--notin-operators"></a>使用 -in 和 -notIn 运算符

若要将用户属性的值与大量其他值进行比较，可使用 -in 或 -notIn 运算符。 使用括号符号“[”和“]”开始和结束值列表。

 在以下示例中，如果 user.department 的值等于列表中的任何值，则表达式的计算结果为 true：

```
   user.department -in ["50001","50002","50003","50005","50006","50007","50008","50016","50020","50024","50038","50039","51100"]
```


### <a name="using-the--match-operator"></a>使用 -match 运算符 
**-match** 运算符用于匹配任何正则表达式。 示例：

```
user.displayName -match "Da.*"   
```
Da、Dav、David 的计算结果为 true，aDa 的计算结果为 false。

```
user.displayName -match ".*vid"
```
David 的计算结果为 true，Da 的计算结果为 false。

## <a name="supported-values"></a>支持的值

表达式中使用的值可包含多种类型，包括：

* 字符串
* 布尔值 - true、false
* 数字
* 数组 - 数字数组、字符串数组

在表达式中指定值时，使用正确的语法来避免错误至关重要。 部分语法提示包括：

* 除非值是字符串，否则双引号是可选的。
* 字符串和正则表达式运算不区分大小写。
* 当字符串值包含双引号时，两个引号都应使用 \` 字符进行转义，例如，user.department -eq \`“Sales\`”是值为“Sales”时的正确语法。
* 还可执行 Null 检查，使用 null 作为值，例如 `user.department -eq null`。

### <a name="use-of-null-values"></a>Null 值的用法

要在规则中指定 null 值，可以使用 null 值。 

* 比较表达式中的 null 值时，请使用 -eq 或 -ne。
* 仅当希望将其解释为文本字符串值时，才在 null 两边加引号。
* 不能将 -not 运算符用作 null 的比较运算符。 如果使用该运算符，将会出现错误，不管使用 null 还是 $null。

引用 null 值的正确方法如下：

```
   user.mail –ne null
```

## <a name="rules-with-multiple-expressions"></a>具有多个表达式的规则

组成员资格规则可包含由 -and、-or 和 -not 逻辑运算符连接的多个单一表达式。 此外，还可结合使用逻辑运算符。 

具有多个表达式且构造正确的成员资格规则的示例如下：

```
(user.department -eq "Sales") -or (user.department -eq "Marketing")
(user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")
```

### <a name="operator-precedence"></a>运算符优先顺序

以下按从高到最的优先级顺序列出了所有运算符。 同一行上的运算符都采用相同的优先级：

```
-eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch -in -notIn
-not
-and
-or
-any -all
```

以下是运算符优先级的示例，其中为用户计算了两个表达式：

```
   user.department –eq "Marketing" –and user.country –eq "US"
```

仅当优先级不满足你的要求时，才需要括号。 例如，如果希望首先评估部门，请参看下面的内容，了解如何使用括号来确定顺序：

```
   user.country –eq "US" –and (user.department –eq "Marketing" –or user.department –eq "Sales")
```

## <a name="rules-with-complex-expressions"></a>具有复杂表达式的规则

成员资格规则可能包含复杂表达式，其中属性、运算符和值采用更复杂的形式。 只要满足以下任何条件，表达式就会被认为是复杂的：

* 属性由一组值组成；具体而言为多值属性
* 表达式使用 -any 和 -all 运算符
* 表达式的值可为一个或多个表达式

## <a name="multi-value-properties"></a>多值属性

多值属性是同一类型的对象的集合。 它们可用于使用 -any 和 -all 逻辑运算符创建成员资格规则。

| 属性 | 值 | 使用情况 |
| --- | --- | --- |
| assignedPlans | 集合中的每个对象均公开以下字符串属性：capabilityStatus、service、servicePlanId |user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled") |
| proxyAddresses| SMTP: alias@domain smtp: alias@domain | (user.proxyAddresses -any (\_ -contains "contoso")) |

### <a name="using-the--any-and--all-operators"></a>使用 -any 和 -all 运算符

可以使用 -any 和 -all 运算符将条件分别应用到集合中的一项或所有项。

* -any（当集合中至少有一项符合条件时满足条件）
* -all（当集合中的所有项都符合条件时满足条件）

#### <a name="example-1"></a>示例 1

assignedPlans 是多值属性，该项列出了分配给用户的所有服务计划。 以下表达式选择具有 Exchange Online（计划 2）服务计划（作为 GUID 值）且其处于“启用”状态的用户：

```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled")
```

可使用像这样的规则对启用了 Office 365（或其他 Microsoft 联机服务）功能的所有用户进行分组。 然后，可将一组策略应用于该组。

#### <a name="example-2"></a>示例 2

以下表达式选择加入任何与 Intune 服务（由服务名称“SCO”标识）关联的服务计划的所有用户：

```
user.assignedPlans -any (assignedPlan.service -eq "SCO" -and assignedPlan.capabilityStatus -eq "Enabled")
```

### <a name="using-the-underscore--syntax"></a>使用下划线 (\_) 语法

下划线 (\_) 语法匹配特定值在其中一个多值字符串集合属性中的出现，以便将用户或设备添加到动态组。 它与 -any 或 -all 运算符一起使用。

下面是在规则中使用下划线 (\_) 基于 user.proxyAddress 添加成员的示例（对于 user.otherMails，它的工作方式相同）。 此规则将任何使用包含“contoso”的代理地址的用户添加到该组。

```
(user.proxyAddresses -any (_ -contains "contoso"))
```

## <a name="other-properties-and-common-rules"></a>其他属性和通用规则

### <a name="create-a-direct-reports-rule"></a>创建“直接下属”规则

可以创建包含经理的所有直接下属的组。 当经理的直接下属将来发生更改时，组的成员资格将自动进行调整。

直接下属规则使用以下语法进行构造：

```
Direct Reports for "{objectID_of_manager}"
```

下面是"62e19b97-8b3d-4d4a-a106-4ce66896a863"所在的管理器的 objectID 的有效规则的示例：

```
Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863"
```

以下提示可帮助你正确使用该规则。

* “经理 ID”是经理的对象 ID。 可在经理的“配置文件”中找到它。
* 要使规则起作用，请确保租户中用户的 Manager 属性已正确设置。 可检查用户的“配置文件”中的当前值。
* 此规则仅支持经理的直接下属。 换言之，无法创建包含经理的直接下属及其下属的组。
* 此规则不能与任何其他成员资格规则结合使用。

### <a name="create-an-all-users-rule"></a>创建“所有用户”规则

可使用成员资格规则创建包含租户中所有用户的组。 以后向租户添加用户或从中删除用户时，将自动调整该组的成员资格。

"所有用户"规则是使用单个使用-ne 运算符和 null 值的表达式构造的。 此规则将 B2B 来宾用户以及成员用户添加到该组。

```
user.objectid -ne null
```

### <a name="create-an-all-devices-rule"></a>创建"所有设备"规则

可使用成员资格规则创建包含租户中所有设备的组。 以后向租户添加设备或从中删除设备时，将自动调整该组的成员资格。

"所有设备"规则是使用单个使用-ne 运算符和 null 值的表达式构造的：

```
device.objectid -ne null
```

## <a name="extension-properties-and-custom-extension-properties"></a>扩展属性和自定义扩展属性

为动态成员身份规则中的字符串属性支持的扩展属性和自定义扩展属性。 扩展属性从本地 Window Server AD 同步，并采用“ExtensionAttributeX”格式，其中 X 等于 1 - 15。 以下是使用扩展属性作为属性的规则示例：

```
(user.extensionAttribute15 -eq "Marketing")
```

自定义扩展属性与本地 Windows Server AD 或连接的 SaaS 应用程序同步，格式为 `user.extension_[GUID]__[Attribute]`，其中：

* [GUID] 是 Azure AD 中用于在 Azure AD 中创建属性的应用程序的唯一标识符
* [Attribute] 是属性创建时的名称

下面是使用自定义扩展属性的规则示例：

```
user.extension_c272a57b722d4eb29bfe327874ae79cb__OfficeNumber -eq "123"
```

通过使用 Graph Explorer 查询用户属性并搜索属性名，可在目录中找到自定义属性名称。 此外，现在可以在动态用户组规则生成器中选择“获取自定义扩展属性”链接，以输入唯一的应用程序 ID，并接收创建动态成员身份规则时要使用的自定义扩展属性的完整列表。 还可以刷新此列表，以获取该应用的任何新自定义扩展属性。

## <a name="rules-for-devices"></a>设备规则

还可以创建一个规则来为组中的成员身份选择设备对象。 无法将用户和设备都作为组成员。 不再列出 **organizationalUnit** 属性，不应使用该属性。 此字符串由 Intune 在特定情况下设置，但 Azure AD 无法识别，因此不会根据此属性向组添加任何设备。

可以使用以下设备属性。

 设备属性  | 值 | 示例
 ----- | ----- | ----------------
 accountEnabled | true false | (device.accountEnabled -eq true)
 displayName | 任意字符串值 |(device.displayName -eq "Rob iPhone")
 deviceOSType | 任意字符串值 | (device.deviceOSType -eq "iPad") -or (device.deviceOSType -eq "iPhone")<br>(device.deviceOSType -contains "AndroidEnterprise")<br>(device.deviceOSType -eq "AndroidForWork")
 deviceOSVersion | 任意字符串值 | (device.deviceOSVersion -eq "9.1")
 deviceCategory | 有效的设备类别名称 | (device.deviceCategory -eq "BYOD")
 deviceManufacturer | 任意字符串值 | (device.deviceManufacturer -eq "Samsung")
 deviceModel | 任意字符串值 | (device.deviceModel -eq "iPad Air")
 deviceOwnership | 个人、公司、未知 | (device.deviceOwnership -eq "Company")
 domainName | 任意字符串值 | (device.domainName -eq "contoso.com")
 enrollmentProfileName | Apple 设备注册配置文件或 Windows Autopilot 配置文件名称 | (device.enrollmentProfileName -eq "DEP iPhones")
 isRooted | true false | (device.isRooted -eq true)
 managementType | MDM（适用于移动设备）<br>电脑（适用于由 Intune 电脑代理管理的计算机） | (device.managementType -eq "MDM")
 deviceId | 有效的 Azure AD 设备 ID | (device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d")
 objectId | 有效的 Azure AD 对象 ID |  (device.objectId -eq 76ad43c9-32c5-45e8-a272-7b58b58f596d")
 systemLabels | 任何与 Intune 设备属性匹配的字符串，用于标记现代工作区设备 | (device.systemLabels -contains "M365Managed")

> [!Note]  
> 对于 deviceOwnership，在创建设备的动态组时，需要将该值设置为“Company”。 而在 Intune 上，设备所有权表示为 Corporate。 请参阅 [OwnerTypes](https://docs.microsoft.com/intune/reports-ref-devices#ownertypes)，了解更多详细信息。 

## <a name="next-steps"></a>后续步骤

以下文章提供了有关 Azure Active Directory 中的组的更多信息。

* [查看现有组](../fundamentals/active-directory-groups-view-azure-portal.md)
* [创建新组并添加成员](../fundamentals/active-directory-groups-create-azure-portal.md)
* [管理组的设置](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理组的成员身份](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理组中用户的动态规则](groups-dynamic-membership.md)
