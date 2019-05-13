---
title: 如何修复已修改的默认规则 - Azure AD Connect | Microsoft Docs
description: 了解如何修复 Azure AD Connect 随附的经过修改的默认规则。
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.reviewer: darora10
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/21/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d2f0956b44d6df64fb73e5eee7844574237d8755
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067639"
---
# <a name="fix-modified-default-rules-in-azure-ad-connect"></a>在 Azure AD Connect 中修复已修改的默认规则

Azure Active Directory (Azure AD) Connect 使用默认规则的同步。  遗憾的是，这些规则不会应用通用于所有组织。 根据您的要求，可能需要对其进行修改。 本文讨论了两个示例的最常见的自定义，并介绍了实现这些自定义的正确方法。

>[!NOTE] 
> 不支持修改现有的默认规则，以实现所需的自定义。 如果您执行此操作，它会阻止在将来更新到最新版本的这些规则将释放。 不会获得所需的 bug 修补程序或新功能。 本文档介绍如何实现相同的结果，而无需修改现有的默认规则。 

## <a name="how-to-identify-modified-default-rules"></a>如何识别已修改的默认规则
从 Azure AD connect 版本 1.3.7.0 开始，很容易地标识已修改的默认规则。 转到**应用程序在桌面上**，然后选择**同步规则编辑器**。

![Azure AD Connect 同步规则编辑器突出显示](media/how-to-connect-fix-default-rules/default1.png)

在编辑器中，带有警告图标名称前面显示的任何已修改的默认规则。

![警告图标](media/how-to-connect-fix-default-rules/default2.png)

 已禁用规则具有相同名称旁边还会出现 （这是标准的默认规则）。

![同步规则编辑器，显示标准的默认规则和已修改的默认规则](media/how-to-connect-fix-default-rules/default2a.png)

## <a name="common-customizations"></a>常见自定义操作
下面是对默认规则执行的常见自定义操作：

- 更改属性流
- 更改范围筛选器
- 更改联接条件

之前更改任何规则：

- 禁用同步计划程序。 默认情况下，计划程序每隔 30 分钟运行一次。 请确保其未启动时要进行更改以及排查新规则。 若要临时禁用计划程序，请启动 PowerShell，并运行`Set-ADSyncScheduler -SyncCycleEnabled $false`。
 ![PowerShell 命令，以禁用同步计划程序](media/how-to-connect-fix-default-rules/default3.png)

- 范围筛选器中的更改可能导致删除目标目录中的对象。 在对象的作用域中进行任何更改之前要小心。 我们建议在活动服务器上进行更改之前，对临时服务器进行更改。
- 在单个对象上运行预览中所述[验证同步规则](#validate-sync-rule)部分中，添加任何新的规则之后。
- 添加新的规则或修改的任何自定义同步规则后运行完全同步。 此同步适用于所有对象的新规则。

## <a name="change-attribute-flow"></a>更改属性流
有三种不同方案用于更改属性流：
- 添加新属性。
- 重写现有属性的值。
- 选择不进行同步的现有属性。

您可以执行下列而无需更改标准默认规则。

### <a name="add-a-new-attribute"></a>添加新属性
如果您查找的属性不从流向源目录的目标目录，请使用[Azure AD Connect 同步：目录扩展](how-to-connect-sync-feature-directory-extensions.md)要解决此问题。

如果扩展不适合您，请尝试添加两个新的同步规则，以下各节中所述。


#### <a name="add-an-inbound-sync-rule"></a>添加入站的同步规则
入站的同步规则意味着该属性的源是连接器空间，且目标为 metaverse。 例如，若要具有新属性流从本地 Active Directory 到 Azure Active Directory 中，创建新的入站的同步规则。 启动**同步规则编辑器**，选择**入站**作为方向，然后选择**添加新规则**。 

 !同步规则 Editor](media/how-to-connect-fix-default-rules/default3a.png)

遵循您自己的命名约定命名规则。 在这里，我们使用**自定义 In from AD-用户**。 也就是说，该规则是自定义规则，并且是从 Active Directory 连接器空间到 metaverse 的入站的规则。   

 ![创建入站同步规则](media/how-to-connect-fix-default-rules/default3b.png)

提供您自己的规则的说明，以便以后维护的规则非常简单。 例如，说明可以基于规则的目的是什么，以及为什么需要它。

进行选择**连接的系统**，**连接的系统对象类型**，并**Metaverse 对象类型**字段。

指定从 0 到 99 之间的优先级值 (较低数字，优先级越高)。 有关**标记**，**启用密码同步**，并**禁用**字段中，使用默认的选择。

保持**范围筛选器**为空。 这意味着，将规则应用于所有对象加入 Active Directory 连接系统与 metaverse 之间。

保持**联接规则**为空。 这意味着该规则使用标准的默认规则中定义的联接条件。 这是另一个原因，不能禁用或删除标准默认规则。 如果没有联接条件，该属性不会流动。 

添加属性的适当转换。 可以将分配一个常量，以使一个常量值流到目标属性。 可以使用的源或目标属性之间的直接映射。 或者，您可以使用表达式的属性。 下面是可以使用的各种[表达式函数](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-sync-functions-reference)。

#### <a name="add-an-outbound-sync-rule"></a>添加出站同步规则
若要链接到目标目录的属性，您需要创建出站规则。 这意味着源是 metaverse，目标是连接的系统。 若要创建出站规则，请启动**同步规则编辑器**，更改**方向**到**出站**，然后选择**添加新规则**. 

![同步规则编辑器](media/how-to-connect-fix-default-rules/default3c.png)

因为使用入站规则时，可以使用命名约定来命名规则。 选择**连接的系统**作为 Azure AD 租户，然后选择已连接系统要向其对象设置属性值。 设置从 0 到 99 之间的优先顺序。 

![创建出站同步规则](media/how-to-connect-fix-default-rules/default3d.png)

保持**范围筛选器**并**联接规则**为空。 填写常量、 直接、 或表达式的转换。 

您现在知道如何对 Azure Active Directory 从 Active Directory 进行用户对象流的新属性。 可以使用这些步骤将任意对象中的任意属性映射到源和目标。 有关详细信息，请参阅[创建自定义同步规则](how-to-connect-create-custom-sync-rule.md)并[来设置用户准备](https://docs.microsoft.com/office365/enterprise/prepare-for-directory-synchronization)。

### <a name="override-the-value-of-an-existing-attribute"></a>重写现有属性的值
你可能想要覆盖已映射的属性的值。 例如，如果始终想要在 Azure AD 中为属性设置 null 值，只需创建入站的规则。 设为常量值， `AuthoritativeNull`，流向目标属性。 

>[!NOTE] 
> 使用`AuthoritativeNull`而不是`Null`这种情况下。 这是因为非 null 值替换 null 值，即使它具有较低优先级 （在规则中的较高数字值）。 `AuthoritativeNull`但是，不替换为非 null 值的其他规则。 

### <a name="dont-sync-existing-attribute"></a>不同步现有属性
如果你想要从同步中排除某个属性，使用筛选功能在 Azure AD Connect 中提供的属性。 启动**Azure AD Connect**从桌面图标，，然后选择**自定义同步选项**。

![Azure AD Connect 的其他任务选项](media/how-to-connect-fix-default-rules/default4.png)

 请确保**Azure AD 应用程序和属性筛选**是所选，然后选择**下一步**。

![Azure AD Connect 的可选功能](media/how-to-connect-fix-default-rules/default5.png)

清除你想要从同步中排除的属性。

![Azure AD Connect 属性](media/how-to-connect-fix-default-rules/default6a.png)

## <a name="change-scoping-filter"></a>更改范围筛选器
Azure AD 同步负责的大多数对象。 可以缩小的作用域的对象，并减少要导出，而无需更改的标准默认同步规则的对象数。 

使用以下方法之一来缩小要同步的对象的作用域：

- cloudFiltered 属性
- 组织单位筛选

如果减少要同步的用户的作用域，也会筛选出用户的密码哈希同步停止。 如果已同步对象后缩小作用域,，筛选出对象是从目标目录中删除。 出于此原因，请确保你能非常仔细地作用域。

>[!IMPORTANT] 
> 不建议增加配置的 Azure AD Connect 的对象的作用域。 执行此操作使难 Microsoft 支持团队，若要了解自定义项。 如果您必须增加的对象的范围，编辑现有规则、 克隆它，并禁用原始规则。 

### <a name="cloudfiltered-attribute"></a>cloudFiltered 属性
无法在 Active Directory 中设置此属性。 添加新的入站的规则来设置此属性的值。 然后，可以使用**转换**并**表达式**在 metaverse 中设置此属性。 下面的示例演示你不想要同步其院系名称开头的所有用户**HRD** （不区分大小写）：

`cloudFiltered <= IIF(Left(LCase([department]), 3) = "hrd", True, NULL)`

我们先转换该部门从源 (Active Directory) 为小写。 然后，使用`Left`函数，我们花费了只有前三个字符并比较其与`hrd`。 如果匹配，将值设置为`True`; 否则为`NULL`。 在将值设置为 null，具有较低优先级 （较高数字值） 的某些其他规则可以写入其不同条件。 要验证同步规则中所述的一个对象上运行的预览[验证同步规则](#validate-sync-rule)部分。

![创建入站的同步规则选项](media/how-to-connect-fix-default-rules/default7a.png)

### <a name="organizational-unit-filtering"></a>筛选的组织单位
您可以创建一个或多个组织单位 (Ou)，并移动不想要同步到这些 Ou 的对象。 然后，配置在 Azure AD Connect 中进行筛选的 OU。 启动**Azure AD Connect**从桌面图标，并选择以下选项。 也可以在安装 Azure AD Connect 时配置 OU 筛选。 

![Azure AD Connect 的其他任务](media/how-to-connect-fix-default-rules/default8.png)

按照向导中，并清除不想要同步的 Ou。

![Azure AD 连接的域和 OU 筛选选项](media/how-to-connect-fix-default-rules/default9.png)

## <a name="change-join-condition"></a>更改联接条件
使用 Azure AD connect 配置的默认联接条件。 更改默认的联接条件使难 Microsoft 支持部门以了解自定义项和支持该产品。

## <a name="validate-sync-rule"></a>验证同步规则
可以通过使用预览功能，而无需运行完全同步周期验证新添加的同步规则。 在 Azure AD Connect，选择**同步服务**。

![Azure AD Connect 同步服务突出显示](media/how-to-connect-fix-default-rules/default10.png)

选择**Metaverse 搜索**。 选择作用域对象作为**person**，选择**添加子句**，并指明您的搜索条件。 接下来，选择**搜索**，然后双击搜索结果中的对象。 请确保你在 Azure AD Connect 中的数据是通过在运行此步骤之前将林的运行导入和同步最新的对象，该对象。

![Synchronization Service Manager](media/how-to-connect-fix-default-rules/default11.png)

上**Metaverse 对象属性**，选择**连接器**，选择相应的连接器 （林） 中的对象，并选择**属性...**.

![Metaverse 对象属性](media/how-to-connect-fix-default-rules/default12.png)

选择**预览...**

![连接器空间对象属性](media/how-to-connect-fix-default-rules/default13a.png)

在预览窗口中，选择**生成预览**并**导入属性流**的左窗格中。

![预览](media/how-to-connect-fix-default-rules/default14.png)
 
在这里，请注意，新添加的规则对象上运行，并且已设置`cloudFiltered`属性为 true。

![预览](media/how-to-connect-fix-default-rules/default15a.png)
 
若要比较的默认规则与已修改的规则，将导出这两个规则分别为文本文件。 这些规则将导出为 PowerShell 脚本文件。 您可以比较它们通过使用任何文件比较工具 (例如，windiff) 以查看所做的更改。 
 
请注意，在已修改的规则`msExchMailboxGuid`属性更改为**表达式**类型，而不是**直接**。 此外，将的值更改为**NULL**并**ExecuteOnce**选项。 可以忽略 Identified（已识别）和 Precedence（优先顺序）的差异。 

![windiff 工具输出](media/how-to-connect-fix-default-rules/default17.png)
 
若要解决你的规则以将其更改回默认设置，删除已修改的规则和启用默认规则。 请确保不会丢失正在努力实现目标的自定义项。 当准备好后时，运行**完全同步**。

## <a name="next-steps"></a>后续步骤
- [硬件和先决条件](how-to-connect-install-prerequisites.md) 
- [快速设置](how-to-connect-install-express.md)
- [自定义设置](how-to-connect-install-custom.md)



