---
title: 解决许可证分配问题的组-Azure Active Directory |Microsoft Docs
description: 使用基于 Azure Active Directory 组的许可时，如何识别和解决许可证分配问题
services: active-directory
keywords: Azure AD 许可
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.subservice: users-groups-roles
ms.topic: article
ms.workload: identity
ms.date: 03/18/2019
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3dbfbd76d235cedd297a5ad54b51bc4ebb550bb1
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466286"
---
# <a name="identify-and-resolve-license-assignment-problems-for-a-group-in-azure-active-directory"></a>识别和解决 Azure Active Directory 中组的许可证分配问题

Azure Active Directory (Azure AD) 中基于组的许可引入了处于许可错误状态的用户的概念。 本文说明用户可能会以此状态结束的原因。

如果直接将许可证分配给单个用户而不使用基于组的许可，分配操作可能会失败。 例如，对用户系统执行 PowerShell cmdlet `Set-MsolUserLicense` 时，由于与业务逻辑相关的多个原因，该 cmdlet 可能会失败。 例如，可能许可证数量不足，或者两个服务计划之间存在冲突，不能同时分配。 将立即向你返回该问题的报告。

使用基于组的许可时，可能会发生相同的错误，但是在 Azure AD 服务分配许可证时在后台发生这些错误。 由于此原因，这些错误无法立即传达给你。 但是，这些错误会记录在用户对象中，并通过管理门户进行报告。 为用户提供许可证的原始意图永远不会丢失，但以错误状态记录，供以后调查和解决。

## <a name="how-to-find-license-assignment-errors"></a>如何查找许可证分配错误
**查找许可证分配错误**

1. 若要查找特定组中处于错误状态的用户，请打开相应组的窗格。 如果有任何用户处于错误状态，会在“许可证”下显示通知。

   ![组和错误通知消息](./media/licensing-groups-resolve-problems/group-error-notification.png)

2. 选择通知以打开所有受影响的用户列表。 可以分别选择每个用户以查看更多详细信息。

   ![许可错误状态的组中的用户的列表](./media/licensing-groups-resolve-problems/list-of-users-with-errors.png)

3. 若要查找包含至少一个错误的所有组，请在“Azure Active Directory”边栏选项卡上，选择“许可证”，再选择“概述”。 如果有一些组需要关注，则会显示信息框。

   ![概述和有关处于错误状态的组的信息](./media/licensing-groups-resolve-problems/group-errors-widget.png)

4. 选中该框可查看具有错误的所有组的列表。 可以选择每个组以了解更多详细信息。

   ![概述和具有错误的组的列表](./media/licensing-groups-resolve-problems/list-of-groups-with-errors.png)


以下部分提供每个潜在问题的说明及其解决方法。

## <a name="not-enough-licenses"></a>没有足够的许可证

**问题：** 组中指定的某个产品没有足够的可用许可证。 需要为该产品购买更多的许可证，或者释放其他用户或组中未使用的许可证。

若要查看可用的许可证数量，请转到“Azure Active Directory” > “许可证” > “所有产品”。

若要查看哪些用户和组正在使用许可证，请选择某个产品。 在“许可的用户”下面，可以看到直接或者通过一个或多个组向其分配许可证的所有用户的列表。 在“许可的组”下面，可以看到该产品已分配到的所有组。

**PowerShell：** PowerShell cmdlet 将此错误报告为 _CountViolation_。

## <a name="conflicting-service-plans"></a>冲突的服务计划

**问题：** 组中指定的某个产品包含的服务计划，与已通过不同的产品分配给用户的另一个服务计划相冲突。 某些服务计划已配置为不能作为另一个相关服务计划分配给相同的用户。

请考虑以下示例。 为某个用户直接分配了 Office 365 企业版 *E1* 许可证并启用了所有计划。 该用户已添加到分配有 Office 365 企业版 *E3* 产品的组。 E3 产品包含的服务计划不能与 E1 中包含的计划重叠，因此，组许可证分配会失败并显示“冲突的服务计划”错误。 在此示例中，冲突的服务计划为：

-   SharePoint Online（计划 2）与 SharePoint Online（计划 1）冲突。
-   Exchange Online（计划 2）与 Exchange Online（计划 1）冲突。

若要解决此冲突，需要禁用两个计划。 可以禁用直接分配给用户的 E1 许可证。 或者，需要修改整个组许可证分配并在 E3 许可证中禁用计划。 或者，你可能会决定删除用户的 E1 许可证（如果 E1 许可证在 E3 许可证的上下文中是多余的）。

有冲突的产品许可证的解决方法始终由管理员决定。 Azure AD 不会自动解决许可证冲突。

**PowerShell：** PowerShell cmdlet 将此错误报告为 _MutuallyExclusiveViolation_。

## <a name="other-products-depend-on-this-license"></a>依赖于此许可证的其他产品

**问题：** 组中指定的某个产品包含的服务计划必须为另一个产品中的另一个服务计划启用才能正常工作。 当 Azure AD 尝试删除基础服务计划时，将出现此错误。 例如，从组中删除用户时，可能会发生此错误。

若要解决此问题，需确保所需的计划仍通过其他某种方法分配给用户，或者为这些用户禁用了依赖服务。 执行这些操作后，可以正确地删除这些用户的组许可证。

**PowerShell：** PowerShell cmdlet 将此错误报告为 _DependencyViolation_。

## <a name="usage-location-isnt-allowed"></a>不允许的使用位置

**问题：** 由于当地法律和法规方面的原因，某些 Microsoft 服务不能在所有位置使用。 必须先为用户指定“使用位置”属性，才能将许可证分配给用户。 可以在 Azure 门户中的“用户” > “配置文件” > “设置”部分下指定位置。

当 Azure AD 尝试向使用位置不受支持的用户分配组许可证时，该操作会失败，并且会记录用户发生的此项错误。

若要解决此问题，请从许可组中删除其位置不受支持的用户。 或者，如果当前使用位置值不代表实际用户位置，可以修改这些值，以便下次可以正常分配许可证（如果新位置受支持）。

**PowerShell：** PowerShell cmdlet 将此错误报告为 _ProhibitedInUsageLocationViolation_。

> [!NOTE]
> 当 Azure AD 分配组许可证时，任何未指定使用位置的用户将继承目录的位置。 建议管理员在使用基于组的许可之前，先为用户设置正确的使用位置值，以符合当地法律和法规。

## <a name="duplicate-proxy-addresses"></a>重复的代理地址

如果使用的是 Exchange Online，可能会使用相同的代理地址值错误地配置租户中的某些用户。 当基于组的许可尝试为此类用户分配许可证时，此操作会失败并显示“代理地址已被使用”。

> [!TIP]
> 若要查看是否有重复的代理地址，请针对 Exchange Online 执行以下 PowerShell cmdlet：
> ```
> Run Get-Recipient | where {$_.EmailAddresses -match "user@contoso.onmicrosoft.com"} | fL Name, RecipientType,emailaddresses
> ```
> 有关此问题的详细信息，请参阅 [Exchange Online 中的“代理地址已被使用”错误消息](https://support.microsoft.com/help/3042584/-proxy-address-address-is-already-being-used-error-message-in-exchange-online)。 此文还包含有关[如何使用远程 PowerShell 连接到 Exchange Online](https://technet.microsoft.com/library/jj984289.aspx) 的信息。

为受影响的用户解决代理地址问题之后，请确保强制对组进行许可证处理，确保现在可以应用许可证。

## <a name="azure-ad-mail-and-proxyaddresses-attribute-change"></a>Azure AD 邮件和 ProxyAddresses 属性更改

**问题：** 更新用户或组的许可证分配，时可能会看到更改某些用户的 Azure AD 邮件和 ProxyAddresses 属性。

更新代理地址计算触发用户原因的许可证分配，这可以更改用户属性。 若要了解此更改的确切原因和解决问题，请参阅这篇文章[如何在 Azure AD 中填充 proxyAddresses 属性](https://support.microsoft.com/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad)。

## <a name="what-happens-when-theres-more-than-one-product-license-on-a-group"></a>如果组中有多个产品许可证，会发生什么情况？

可将多个产品许可证分配到一个组。 例如，可将 Office 365 企业版 E3 和企业移动性 + 安全性分配到某个组，轻松为用户启用所有包含的服务。

Azure AD 会尝试将该组中指定的所有许可证分配给每个用户。 如果由于业务逻辑问题，Azure AD 无法分配某个产品，则也不会在该组中分配其他许可证。 例如，所有产品的许可证数量不足，或者与针对用户启用的其他服务冲突。

可以查看无法被分配许可证的用户，并且可以检查哪些产品受此问题的影响。

## <a name="what-happens-when-a-group-with-licenses-assigned-is-deleted"></a>删除分配了许可证的组会发生什么情况？

在删除组之前，必须先删除分配到该组的所有许可证。 但是，删除组中所有用户的许可证可能比较耗时。 删除组的许可证分配时，如果为用户分配了依赖的许可证，或者某个代理地址冲突问题阻止删除许可证，则删除操作可能失败。 如果用户的某个许可证依赖于删除组时一同删除的许可证，则用户的许可证分配将从继承转换为直接。

例如，假设分配了 Office 365 E3/E5 的某个组启用了 Skype for Business 服务计划。 此外，假设直接为组中的几个成员分配了 Audio Conferencing 许可证。 删除该组时，基于组的许可会尝试删除所有用户的 Office 365 E3/E5。 由于 Audio Conferencing 依赖于 Skype for Business，对于分配了 Audio Conferencing 的所有用户，基于组的许可会将 Office 365 E3/E5 许可证转换为直接许可证分配。

## <a name="how-do-you-manage-licenses-for-products-with-prerequisites"></a>如何为有先决条件的产品管理许可证？

你拥有的某些 Microsoft Online 产品可能是“附加产品”。 附加产品要求先为用户或组启用先决服务计划，才能向其分配许可证。 要使用基于组的许可，系统要求先决条件和附加产品服务计划存在于同一组中。 这是为了确保添加到组的任何用户都能收到功能齐全的产品。 请考虑以下示例：

Microsoft Workplace Analytics 是一个附加产品。 它包含同名单一服务计划。 仅当同时分配了以下必备产品之一时，才能将此服务计划分配到用户或组：
- Exchange Online（计划 1） 
- Exchange Online（计划 2）

如果尝试将此产品本身分配到组，门户会返回错误。 选择错误通知会显示以下详细信息：

![组，缺少先决条件](./media/licensing-groups-resolve-problems/group-prerequisite-required.png)

选择详细信息会显示以下错误消息：

>许可证操作失败。 添加或删除从属服务前，请确保组具有必要的服务。 **Microsoft Workplace Analytics 服务还要求启用 Exchange Online（计划 2）。**

若要将此附加产品许可证分配到组，必须确保该组也包含先决服务计划。 例如，可更新已经包含完整 Office 365 E3 产品的现有组，并向其添加附加产品。

还可以创建一个独立的组，其中只包含正常运行附加产品所需的最少量产品。 使用该组可以仅向选定的用户授予附加产品的许可证。 本示例向同一个组分配了以下产品：
- Office 365 企业版 E3，仅启用了 Exchange Online（计划 2）服务计划
- Microsoft Workplace Analytics

![组，包含先决条件](./media/licensing-groups-resolve-problems/group-addon-with-prerequisite.png)

从现在起，添加到此组的任何用户都将使用一个 E3 产品许可正和一个 Workplace Analytics 产品许可证。 同时，这些用户可以是另一组的成员，为其提供完整的 E3 产品，他们仍只使用该产品的一个许可证。

> [!TIP]
> 可为每个先决服务计划创建多个组。 例如，如果有用户使用 Office 365 Enterprise E1，也有用户 Office 365 Enterprise E3，则可创建两个组来授权 Microsoft Workplace Analytics：一个使用 E1 作为先决条件，另一个使用 E3。 这样，不需要额外的许可证，即可将附加产品分发到 E1 和 E3 用户。

## <a name="how-do-you-force-license-processing-in-a-group-to-resolve-errors"></a>如何强制处理组中的许可证以解决错误？

根据解决错误时采取的措施，可能需要手动触发组的处理来更新用户状态。

例如，如果通过删除用户的直接许可证分配来释放某些许可证，则需要触发以前无法完全为所有用户成员提供许可证的组的处理。 若要重新处理某个组，请转到组窗格，打开“许可证”，并在工具栏中选择“重新处理”按钮。

## <a name="how-do-you-force-license-processing-on-a-user-to-resolve-errors"></a>如何强制用户处理许可证以解决错误？

根据解决错误时采取的措施，可能需要手动触发用户的处理来更新用户状态。

例如，解决受影响用户的重复代理地址问题后，需要触发用户的处理。 若要重新处理某个用户，请转到用户窗格，打开“许可证”，并在工具栏中选择“重新处理”按钮。

## <a name="next-steps"></a>后续步骤

若要详细了解通过组进行许可证管理的其他方案，请参阅以下部分：

* [Azure Active Directory 中基于组的许可是什么？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [将许可证分配到 Azure Active Directory 中的组](licensing-groups-assign.md)
* [如何将单个许可用户迁移到 Azure Active Directory 中基于组的许可](licensing-groups-migrate-users.md)
* [如何在 Azure Active Directory 中使用基于组的许可在产品许可证之间迁移用户](licensing-groups-change-licenses.md)
* [Azure Active Directory 基于组的许可的其他方案](licensing-group-advanced.md)
* [Azure Active Directory 中基于组的许可的 PowerShell 示例](licensing-ps-examples.md)
