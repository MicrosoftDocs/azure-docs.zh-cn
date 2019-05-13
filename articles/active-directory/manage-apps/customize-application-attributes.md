---
title: 自定义 Azure AD 属性映射 | Microsoft Docs
description: 了解 Azure Active Directory 中有哪些针对 SaaS 应用的属性映射，以及如何修改它们来满足业务需求。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: celested
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: c9db6babff3ae0c91f730e6ec2e6e2a41d55b0bf
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191508"
---
# <a name="customizing-user-provisioning-attribute-mappings-for-saas-applications-in-azure-active-directory"></a>为 Azure Active Directory 中的 SaaS 应用程序自定义用户预配属性映射
Microsoft Azure AD 用户预配到第三方 SaaS 应用，例如 Salesforce、 G Suite 和其他人提供支持。 如果启用了第三方 SaaS 应用程序的用户预配，Azure 门户将控制通过属性映射其属性值。

没有一组预配置的属性和 Azure AD 用户对象和每个 SaaS 应用的用户对象之间的属性映射。 某些应用程序管理其他类型的对象，以及用户、 组等。

可以根据业务需求自定义默认的属性映射。 因此，可以更改或删除现有属性的映射，或创建新的属性映射。
 
## <a name="editing-user-attribute-mappings"></a>编辑用户属性映射

请按照以下步骤访问**映射**用户预配的功能：

1. 登录到[Azure Active Directory 门户](https://aad.portal.azure.com)。

1. 选择**企业应用程序**在左窗格中。 显示所有已配置应用的列表，包括那些从库添加的应用。

1. 选择要上传其应用的管理窗格，其中可以查看报表和管理应用程序设置任何应用。

1. 选择**预配**来管理用户帐户预配所选应用的设置。

1. 展开**映射**查看和编辑 Azure AD 之间流动的用户属性和目标应用程序。 如果目标应用程序支持，可以使用本部分 （可选） 配置的组和用户帐户预配。

   ![Salesforce](./media/customize-application-attributes/21.png) 

1. 选择**映射**配置，以打开相关**属性映射**屏幕。 某些属性映射所需的 SaaS 应用程序才能正常工作。 对于必需属性，“删除”功能将不可用。

   ![Salesforce](./media/customize-application-attributes/22.png)

   在此屏幕截图中，可以看到**用户名**Salesforce 中的托管对象的属性使用填充**userPrincipalName**链接的 Azure Active Directory 对象的值。

1. 选择一个现有**属性映射**以打开**编辑属性**屏幕。 在这里，您可以编辑 Azure AD 之间流动的用户属性和目标应用程序。

   ![Salesforce](./media/customize-application-attributes/23.png)


### <a name="understanding-attribute-mapping-types"></a>了解属性映射类型
使用属性映射可以控制属性在第三方 SaaS 应用程序中的填充方式。 支持四种不同的映射类型：

* **直接** – 目标属性由 Azure AD 中的链接对象的属性值填充。
* **常量**– 目标属性使用你指定的特定字符串填充。
* **表达式** - 目标属性是基于脚本式表达式的结果填充的。 
  有关详细信息，请参阅[在 Azure Active Directory 中编写属性映射的表达式](functions-for-customizing-application-data.md)。
* **无** - 目标属性保持不变。 不过，如果目标属性曾经为空，它是与你指定的默认值填充。

这四种基本类型，以及自定义属性映射还支持可选的概念**默认**值赋值。 默认值赋值确保如果没有一个值，Azure AD 中或目标对象上使用某个值填充目标属性。 最常见的配置是将此项留空。


### <a name="understanding-attribute-mapping-properties"></a>了解属性映射的属性

在上一节中已介绍了属性映射类型属性。
此属性，以及属性映射还支持以下属性：

- **源属性** - 来自源系统的用户属性（示例：Azure Active Directory）。
- **目标属性** - 目标系统中的用户属性（示例：ServiceNow）。
- **使用此属性匹配对象**– 是否应使用此映射唯一标识源和目标系统之间的用户。 它通常在 Azure AD 中，此值通常映射到目标应用程序中的用户名字段设置 userPrincipalName 或邮件属性上。
- 匹配优先级 – 可设置多个匹配属性。 如果有多个，它们是按此字段定义的顺序进行计算。 一旦找到匹配，就不会进一步评估其他匹配属性。
- **应用此映射**
    - **始终**– 在这两个用户创建应用此映射和更新操作。
    - **仅创建期间**-仅在用户的创建操作上应用此映射。


## <a name="editing-group-attribute-mappings"></a>编辑组属性映射

一些特定的应用程序，例如 ServiceNow、 Box 和 G Suite 支持预配组对象和用户对象的能力。 组对象可以包含组的属性，例如显示名称和电子邮件别名，以及组成员。

![ServiceNow](./media/customize-application-attributes/24.png)

组预配可以根据需要启用或禁用通过选择下的组映射**映射**，并设置**已启用**中所需的选项为**的属性映射**屏幕。

可以自定义预配为“组”对象的一部分的属性，就像前面所述的预配“用户”对象一样。 

>[!TIP]
>预配组对象（属性和成员）的过程完全不同于[将组分配](assign-user-or-group-access-portal.md)到应用程序。 可以将组分配到应用程序，但只能预配组中包含的用户对象。 预配整个组对象不需要使用分配中的组。


## <a name="editing-the-list-of-supported-attributes"></a>编辑受支持属性的列表

给定应用程序支持的用户属性是预先配置的。 大多数应用程序的用户管理 Api 不支持架构发现。 因此，Azure AD 预配服务不能动态调用应用程序，从而生成受支持的属性列表。 

但是，某些应用程序支持自定义特性，并且 Azure AD 预配服务可以读取和写入到自定义属性。 若要在 Azure 门户中输入其定义，请选择**显示高级选项**底部的复选框**属性映射**屏幕上，并选择**编辑属性列表**您的应用程序。

支持属性列表自定义的应用程序和系统包括：

* Salesforce
* ServiceNow
* Workday
* Azure Active Directory（支持 [Azure AD 图形 API 默认属性](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity)和自定义目录扩展）
* 支持 [SCIM 2.0](https://tools.ietf.org/html/rfc7643) 的应用（需要添加[核心架构](https://tools.ietf.org/html/rfc7643)中定义的属性）

>[!NOTE]
>建议只让自定义了其应用程序和系统的架构，并且原本就知道自定义属性定义方式的管理员来编辑受支持属性的列表。 有时，需要熟悉应用程序或系统提供的 API 和开发人员工具才能执行此操作。 

编辑受支持属性的列表时，将提供以下属性：

* **名称** - 属性的系统名称，在目标对象的架构中定义。 
* **类型**-的数据类型属性存储，定义在目标对象的架构中，可以是以下类型之一：
   * *二进制* - 属性包含二进制数据。
   * *布尔值* - 属性包含 True 或 False 值。
   * *日期时间* - 属性包含日期字符串。
   * *整数* - 属性包含整数。
   * *引用* - 属性包含 ID，该 ID 引用目标应用程序中的另一个表存储的值。
   * *字符串* - 属性包含文本字符串。 
* **主键?** -是否该属性定义为目标对象的架构中的主键字段。
* **必需？** -是否该属性为所需的目标应用程序或系统进行填充。
* **多值?** -是否该属性支持多个值。
* **区分大小写?** -是否区分大小写的方式计算属性值。
* **API 表达式**-不使用，除非要求这样做 （例如 Workday) 特定的预配连接器的文档。
* **引用的对象属性**-如果它为引用类型的属性，则此菜单允许您选择的表和属性中包含的值与属性关联的目标应用程序。 例如，如果名为“Department”的属性的存储值引用了独立“Departments”表中的对象，则需要选择“Departments.Name”。 引用表和主要 ID 字段给定的应用程序支持的预配置目前不能使用 Azure 门户中，编辑但可使用编辑[图形 API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-configure-with-custom-target-attributes)。

若要添加新属性，请滚动到受支持属性列表的末尾，使用提供的输入填充上述字段，然后选择“添加属性”。 添加完属性后选择“保存”。 然后，你需要重新加载**预配**将新的属性，属性映射编辑器中可用的选项卡。

## <a name="restoring-the-default-attributes-and-attribute-mappings"></a>还原默认属性和属性映射

如果需要通过启动和重置现有映射回其默认状态，你可以选择**还原默认映射**复选框并保存配置。 执行此操作集的所有映射，当应用程序刚添加到你的 Azure AD 租户应用程序库中。 

预配服务正在运行时，选择此选项可以有效地强制重新同步所有用户。 

>[!IMPORTANT]
>我们强烈建议**预配状态**设置为**关闭**之前调用此选项。


## <a name="what-you-should-know"></a>要点

* Microsoft Azure AD 提供了一个有效的同步进程实现。 在初始化环境中，在同步周期中只会处理需要更新的对象。 

* 更新属性映射会影响同步周期的性能。 对属性映射配置进行更新需要重新评估所有托管对象。 

* 推荐的最佳做法是连续更改的数量保持在最少属性的映射。

* 添加照片属性预配到应用程序目前不支持作为不能指定要同步的照片的格式。 可以在请求功能[用户之声](https://feedback.azure.com/forums/169401-azure-active-directory)


## <a name="next-steps"></a>后续步骤

* [在 SaaS 应用中自动预配和取消预配用户](user-provisioning.md)
* [为属性映射编写表达式](functions-for-customizing-application-data.md)
* [用于用户预配的作用域筛选器](define-conditional-rules-for-provisioning-user-accounts.md)
* [使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配](use-scim-to-provision-users-and-groups.md)
* [有关如何集成 SaaS 应用的教程列表](../saas-apps/tutorial-list.md)


