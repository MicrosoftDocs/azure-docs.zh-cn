---
title: 启用 Azure Active Directory B2C 中的年龄限制 | Microsoft Docs
description: 学习如何辨别使用应用程序的未成年人。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/13/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: b594dc6eadce5093c58d693492f8e86eb92ae4e3
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228000"
---
# <a name="enable-age-gating-in-azure-active-directory-b2c"></a>启用 Azure Active Directory B2C 中的年龄限制

>[!IMPORTANT]
>此功能目前以公共预览版提供。 请勿将此功能用于生产应用程序。 
>

通过 Azure Active Directory (Azure AD) B2C 中的年龄限制，可识别想要使用你的应用程序的未成年人。 可选择阻止未成年人登录应用程序。 用户还可返回应用程序并确定其年龄组和父母同意状态。 Azure AD B2C 可以阻止未获父母同意的未成年人使用应用程序。 也可以将 Azure AD B2C 设置为允许应用程序决定如何对待未成年人。

启用年龄限制中后你[用户流](active-directory-b2c-reference-policies.md)，会要求用户出生时并哪一个国家/地区他们居住在。 如果用户登录之前未输入信息，则他们需要在下次登录时输入该信息。 每次用户登录时都会应用这些规则。

Azure AD B2C 使用用户输入的信息来确定他们是否是未成年人。 然后，在其帐户中更新“ageGroup”字段。 值可为 `null`、`Undefined`、`Minor`、`Adult` 和 `NotAdult`。  然后，将 ageGroup 和 consentProvidedForMinor 字段用于计算 legalAgeGroupClassification 的值。

年龄限制涉及两个年龄值：不再视其为未成年人的年龄，以及未成年人必须征得父母同意的年龄。 下表列出的年龄规则用于定义未成年人和须征得父母同意的未成年人。

| 国家/地区 | 国家/地区名称 | 须征得同意的未成年人年龄 | 未成年人年龄 |
| -------------- | ------------------- | ----------------- | --------- |
| 默认 | 无 | 无 | 18 |
| AE | 阿拉伯联合酋长国 | 无 | 21 |
| AT | 奥地利 | 14 | 18 |
| BE | 比利时 | 14 | 18 |
| BG | 保加利亚 | 16 | 18 |
| BH | 巴林 | 无 | 21 |
| CM | 喀麦隆 | 无 | 21 |
| CY | 塞浦路斯 | 16 | 18 |
| CZ | 捷克共和国 | 16 | 18 |
| DE | 德国 | 16 | 18 |
| DK | 丹麦 | 16 | 18 |
| EE | 爱沙尼亚 | 16 | 18 |
| EG | 埃及 | 无 | 21 |
| ES | 西班牙 | 13 | 18 |
| FR | 法国 | 16 | 18 |
| GB | 英国 | 13 | 18 |
| GR | 希腊 | 16 | 18 |
| HR | 克罗地亚 | 16 | 18 |
| HU | 匈牙利 | 16 | 18 |
| IE | 爱尔兰 | 13 | 18 |
| IT | 意大利 | 16 | 18 |
| KR | 韩国 | 14 | 18 |
| LT | 立陶宛 | 16 | 18 |
| LU | 卢森堡 | 16 | 18 |
| LV | 拉脱维亚 | 16 | 18 |
| MT | 马耳他 | 16 | 18 |
| NA | 纳米比亚 | 无 | 21 |
| NL | 荷兰 | 16 | 18 |
| PL | 波兰 | 13 | 18 |
| PT | 葡萄牙 | 16 | 18 |
| RO | 罗马尼亚 | 16 | 18 |
| SE | 瑞典 | 13 | 18 |
| SG | 新加坡 | 无 | 21 |
| SI | 斯洛文尼亚 | 16 | 18 |
| SK | 斯洛伐克 | 16 | 18 |
| TD | 乍得 | 无 | 21 |
| TH | 泰国 | 无 | 20 |
| TW | 中国台湾 | 无 | 20 | 
| 美国 | 美国 | 13 | 18 |

## <a name="age-gating-options"></a>年龄限制选项
 
### <a name="allowing-minors-without-parental-consent"></a>允许未征得家长同意的未成年人

对于允许注册和/或登录的用户流，可以选择允许未经同意的未成年人使用应用程序。 对于未征得家长同意的未成年人，允许他们照常登录或注册，并且 Azure AD B2C 会颁发一个带有“legalAgeGroupClassification”声明的 ID 令牌。 此声明定义了用户体验，例如征得家长同意和更新“consentProvidedForMinor”字段。

### <a name="blocking-minors-without-parental-consent"></a>阻止未征得家长同意的未成年人

对于允许注册和/或登录的用户流，可以选择阻止未经同意的未成年人使用应用程序。 以下选项可用于处理 Azure AD B2C 中的被阻止用户：

- 将 JSON 发送回应用程序 - 此选项会将响应发送回阻止了未成年人的应用程序。
- 显示错误页 - 向用户显示一个页面，告知他们不能访问该应用程序。

## <a name="set-up-your-tenant-for-age-gating"></a>设置租户年龄限制

若要在用户流中使用年龄限制，需要将租户配置为具有附加属性。

1. 单击顶部菜单中的“目录和订阅筛选器”，确保使用包含 Azure AD B2C 租户的目录。 选择包含租户的目录。 
2. 选择 Azure 门户左上角的“所有服务”，搜索并选择 Azure AD B2C。
3. 在左侧菜单中选择租户的“属性”。
2. 在“年龄限制”部分，单击“配置”。
3. 等待操作完成，系统将设置租户的年龄限制。

## <a name="enable-age-gating-in-your-user-flow"></a>在用户流中启用年龄限制

将租户设置为使用年龄限制后，可在启用它的[用户流](user-flow-versions.md)中使用此功能。 通过执行以下步骤启用“年龄限制”：

1. 创建启用了年龄限制的用户流。
2. 创建用户流后，在菜单中选择“属性”。
3. 在“年龄限制”部分，选择“已启用”。
4. 然后确定如何管理标识为未成年人的用户。 对于“注册或登录”，可以选择 `Allow minors to access your application` 或 `Block minors from accessing your application`。 如果选择阻止未成年人，则选择 `Send a JSON back to the application` 或 `Show an error message`。 




