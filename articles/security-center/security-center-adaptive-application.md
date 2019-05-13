---
title: Azure 安全中心的自适应应用程序控制 | Microsoft Docs
description: 本文档介绍如何在 Azure 安全中心使用自适应应用程序控制将在 Azure VM 中运行的应用程序加入允许列表。
services: security-center
documentationcenter: na
author: monhaber
manager: barbkess
editor: ''
ms.assetid: 9268b8dd-a327-4e36-918e-0c0b711e99d2
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/02/2019
ms.author: monhaber
ms.openlocfilehash: 581db3ebe3ce1cad5c8db3702af179e141f334b4
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143466"
---
# <a name="adaptive-application-controls-in-azure-security-center"></a>Azure 安全中心的自适应应用程序控制
了解如何通过本演练在 Azure 安全中心配置应用程序控制。

## <a name="what-are-adaptive-application-controls-in-security-center"></a>安全中心的自适应应用程序控制是什么？
自适应应用程序控制是 Azure 安全中心提供的智能的自动化端到端应用程序允许列表解决方案。 它可帮助你控制哪些应用程序可以在 Azure 运行和非 Azure Vm （Windows 和 Linux），其中还具有其他优点，可帮助强化 Vm 对抗恶意软件。 安全中心使用机器学习分析在 VM 上运行的应用程序，有助于运用此智能服务应用特定的允许列表规则。 此功能大大简化配置和维护应用程序允许列表策略的过程，让你可以：

- 阻止运行恶意应用程序的尝试（包括在其他情况下可能会被反恶意软件解决方案遗漏的尝试）或者向用户发出此方面的警报。
- 遵循组织要求只能使用许可软件的安全策略。
- 避免在环境中使用不需要的软件。
- 避免运行旧的不受支持的应用。
- 防止使用组织不允许的特定软件工具。
- 允许 IT 部门控制用户使用应用来访问敏感数据。

> [!NOTE]
> 对于非 Azure 和 Linux Vm，在审核模式下支持自适应应用程序控件。

## <a name="how-to-enable-adaptive-application-controls"></a>如何启用自适应应用程序控制？
可以使用自适应应用程序控制来定义一组应用程序，允许这些应用程序在所配置 VM 组上运行。 此功能仅适用于 Azure 和非 Azure Windows （所有版本、 经典或 Azure 资源管理器） 和 Linux 虚拟机和服务器。 以下步骤可以用来在安全中心配置应用程序允许列表功能：

1. 打开“安全中心”仪表板。
2. 在左窗格的“高级云防御”下选择“自适应应用程序控制”。

    ![防御](./media/security-center-adaptive-application/security-center-adaptive-application-fig1-new.png)

此时会显示“自适应应用程序控件”页。

![controls](./media/security-center-adaptive-application/security-center-adaptive-application-fig2.png)

“VM 组”部分包含三个选项卡：

* **已配置**：其中包含已配置了应用程序控制的 VM 的组列表。
* **已建议**：建议对其实施应用程序控制的组的列表。 安全中心使用机器学习，根据 VM 是否一致地运行相同应用程序来确定适用于应用程序控制的 VM。
* **无建议**：其中包含没有任何应用程序控制建议的 VM 的组列表。 例如，其上的应用程序始终变化，尚未达到稳定状态的 VM。

> [!NOTE]
> 安全中心使用专有聚类算法来创建 VM 组，确保相似的 VM 获得推荐的最佳应用程序控制策略。
>
>

### <a name="configure-a-new-application-control-policy"></a>配置新的应用程序控制策略
1. 单击“已建议”选项卡会出现一个列表，其中列出了具有应用程序控制建议的组：

   ![建议](./media/security-center-adaptive-application/security-center-adaptive-application-fig3.png)

   此列表包括：

   - **组名**:订阅和组的名称
   - **Vm 和计算机**:在组中的虚拟机数
   - **状态**： 建议的状态
   - **严重性**: 建议的严重性级别

2. 单击一个组，打开“创建应用程序控制规则”选项。

   ![应用程序控制规则](./media/security-center-adaptive-application/security-center-adaptive-application-fig4.png)

3. 在“选择 VM”中查看建议的 VM 的列表，取消选中不需向其应用应用程序允许列表策略的 VM。 接下来，你将看到两个列表：

   - **建议的应用程序**：此列表中的应用程序在此组中的 VM 上频繁出现，建议允许其运行。
   - **更多应用程序**：此列表中的应用程序在此组中的 VM 上不太频繁地出现（这类应用程序也称为“可利用项”，详见下文），因此建议对其进行复查。

4. 复查每个列表中的应用程序，并取消选中不希望应用规则的任何应用程序。 每个列表包括：

   - **名称**：应用程序的证书信息或其完整路径
   - **文件类型**：应用程序文件类型。 可以是 EXE、脚本、MSI 或这些类型的任意排列。
   - **可利用**：一个警告图标，表示攻击者可能会利用特定应用程序绕过应用程序允许列表解决方案。 建议在这些应用程序审批前查看它们。
   - **用户**：建议允许其运行应用程序的用户

5. 选择完以后，请选择“创建”。 <br>
   选择“创建”后，Azure 安全中心会自动基于 Windows 服务器 (AppLocker) 上可用的内置应用程序允许列表解决方案创建相应的规则。

> [!NOTE]
> - 安全中心需要至少两周的数据才能创建基线并根据 VM 组填充唯一建议。 安全中心标准层的新客户会遇到一种行为，即 VM 组首先显示在“无建议”选项卡下。
> - 安全中心内的自适应应用程序控制不支持已通过 GPO 或本地安全策略为其启用了 AppLocker 策略的 VM。
> -  安全中心始终会遵循安全方面的最佳做法，尝试为选择允许的应用程序创建发布者规则，只有在应用程序没有发布者信息（即未签名）的情况下，才会为特定应用程序的完整路径创建路径规则。
>   

### <a name="editing-and-monitoring-a-group-configured-with-application-control"></a>编辑和监视配置了应用程序控制的组

1. 若要编辑和监视使用应用程序允许列表策略配置的组，请返回到“自适应应用程序控制”页，在“VM 组”下选择“已配置”：

   ![组](./media/security-center-adaptive-application/security-center-adaptive-application-fig5.png)

   此列表包括：

   - **组名**： 订阅和组的名称
   - **Vm 和计算机**： 组中的虚拟机数
   - **模式**：“审核”模式将记录运行未加入允许列表的应用程序的尝试；“强制”模式将阻止未加入允许列表的应用程序运行
   - **警报**：任何当前的冲突

2. 单击一个组，可在“编辑应用程序控制策略”页中进行更改。

   ![保护](./media/security-center-adaptive-application/security-center-adaptive-application-fig6.png)

3. 在“保护模式”下，可以在以下选项之间进行选择：

   - **审核**：在此模式下，应用程序控制解决方案不强制实施规则，只审核受保护 VM 上的活动。 如果在阻止应用在目标 VM 中运行之前需先观察总体行为，建议使用此模式。
   - **强制**：在此模式下，应用程序控制解决方案强制实施规则，确保阻止不允许运行的应用程序。

   > [!NOTE]
   > -  在进一步通知之前，禁用“强制”保护模式。
   > - 如前所述，默认情况下，新的应用程序控制策略始终在“审核”模式下配置。 
   >

4. 可以在“策略扩展”下添加需要允许的任何应用程序路径。 添加这些路径后，除了已有的规则之外，安全中心还会更新所选 VMS 组中 VM 上的应用程序允许列表策略，并为这些应用程序创建适当的规则。

5. 查看“最近警报”部分中列出的当前冲突。 单击要重定向到 Azure 安全中心内“警报”页面的每一行，并查看 Azure 安全中心在关联的 VM 上检测到的所有警报。
   - **警报**：已记录的任何冲突。
   - **VM 数量**：带有此类警报的虚拟机数量。

6. 在“发布者允许列表规则”、“路径允许列表规则”和“哈希允许列表规则”下，可以根据规则集合类型，查看当前在组内的 VM 上配置了哪些应用程序允许列表规则。 对于每条规则，可以查看：

   - **规则**：AppLocker 根据此参数检查应用程序是否能运行。
   - **文件类型**：特定规则涵盖的文件类型。 可以是以下任意一种类型：EXE、脚本、MSI 或这些文件类型的任意排列。
   - **用户**：允许运行应用程序允许列表规则所涵盖的应用程序的用户的名称或数量。

   ![允许列表规则](./media/security-center-adaptive-application/security-center-adaptive-application-fig9.png)

7. 若要删除特定规则或编辑允许的用户，请单击每行末尾的三点符号。

8. 更改“自适应应用程序控制”策略后，请点击“保存”。

### <a name="not-recommended-list"></a>“不建议”列表

安全中心只为运行稳定的一组应用程序的虚拟机建议应用程序允许列表规则。 如果已关联 VM 上的应用程序始终变化不定，则不会创建建议。

![建议](./media/security-center-adaptive-application/security-center-adaptive-application-fig11.png)

此列表包含：
- **组名**： 订阅和组的名称
- **Vm 和计算机**： 组中的虚拟机数

Azure 安全中心还可以在非推荐的 VM 组上定义应用程序允许列表策略。 遵循与前述相同的原则，在这些组上配置应用程序允许列表策略。

## <a name="move-a-vm-from-one-group-to-another"></a>将 VM 从一个组移到另一个

 当 VM 从一个组移到另一个中时，应用于它的应用程序控制策略更改为组设置，您将其移动到。 此外可以从配置的组移动 VM，到非配置组，这会导致删除任何之前应用到 VM 的应用程序控制策略。

 1. 从**自适应应用程序控件**页上，从**配置**选项卡上，单击要移动当前 VM 所属的组。
1. 单击**配置的 Vm 和计算机**。
1. 单击 VM，然后单击的行中的三点式**移动**。 **移动到不同的组的计算机**窗口随即打开。

    ![保护](./media/security-center-adaptive-application/adaptive-application-move-group.png)

 1. 选择要移动到，VM，然后单击的组**移动计算机**，然后单击**保存**。

    ![保护](./media/security-center-adaptive-application/adaptive-application-move-group2.png)

 > [!NOTE]
> 请务必单击**保存**单击后**移动计算机**。 如果不单击**保存**，则不会移动计算机。

## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何在 Azure 和非 Azure Vm 中运行的白名单应用程序在 Azure 安全中心中使用自适应应用程序控制。 若要了解更多有关 Azure 安全中心的详细信息，请参阅以下内容：

* [Managing and responding to security alerts in Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)（管理和响应 Azure 安全中心的安全警报）。 了解如何管理警报并响应安全中心的安全事件。
* [Security health monitoring in Azure Security Center](security-center-monitoring.md)（在 Azure 安全中心进行安全运行状况监视）。 了解如何监视 Azure 资源的运行状况。
* [了解 Azure 安全中心中的安全警报](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同类型的安全警报。
* [Azure 安全中心故障排除指南](https://docs.microsoft.com/azure/security-center/security-center-troubleshooting-guide)。 了解如何排查安全中心的常见问题。
* [Azure Security Center FAQ](security-center-faq.md)（Azure 安全中心常见问题）。 查找有关如何使用服务的常见问题。
* [Azure 安全性博客](https://blogs.msdn.com/b/azuresecurity/)。 查找关于 Azure 安全性及合规性的博客文章。
