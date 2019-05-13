---
title: 升级 Azure Service Fabric 群集 | Microsoft 文档
description: 了解有关升级 Azure Service Fabric 群集的版本或配置的信息。  本文介绍如何设置群集更新模式、升级证书、添加应用程序端口、执行 OS 修补程序以及执行升级时可预期的内容
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: chackdan
editor: ''
ms.assetid: 15190ace-31ed-491f-a54b-b5ff61e718db
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/12/2018
ms.author: aljo
ms.openlocfilehash: 8fa461d8c3a70d4b0d2d9973a840ffc7d1ff6470
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472770"
---
# <a name="upgrading-and-updating-an-azure-service-fabric-cluster"></a>升级和更新 Azure Service Fabric 群集

对于任何现代系统，设计可升级性都是实现产品长期成功的关键。 Azure Service Fabric 群集是你拥有的，但部分由 Microsoft 管理的资源。 本文说明自动管理的项目以及可以自行配置的项目。

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>控制在群集上运行的结构版本

确保始终让群集运行[受支持的结构版本](service-fabric-versions.md)。 当我们公布新版本的 Service Fabric 发布后，则标志着自该日期起至少 60 天以后结束对旧版本的支持。 新版本在 Service Fabric 团队博客上公布。 之后新版本则可供选择。

群集运行的版本过期前 14 天，系统会生成运行状况事件，使群集进入警告运行状况状态。 群集将继续处于警告状态，直至升级至支持的结构版本。

可以将群集设置为 Microsoft 发布结构升级时自动接收该升级，或可以选择想要群集安装的受支持结构版本。  若要了解详细信息，请参阅[升级群集的 Service Fabric 版本](service-fabric-cluster-upgrade-version-azure.md)。

## <a name="fabric-upgrade-behavior-during-automatic-upgrades"></a>自动升级期间的结构升级行为
Microsoft 将维护 Azure 群集中运行的结构代码和配置。 我们会根据需要，对软件执行受监视的自动升级。 升级的部分可能是代码和/或配置。 为了确保应用程序不受这些升级的影响或者将影响降到最低，我们将分以下阶段执行升级：

### <a name="phase-1-an-upgrade-is-performed-by-using-all-cluster-health-policies"></a>阶段 1：使用所有群集运行状况策略执行升级
在此阶段，升级过程将每次升级一个升级域，已在群集中运行的应用程序继续运行，而不会造成任何停机时间。 在升级期间，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 然后，系统会向订阅所有者发送一封电子邮件。 电子邮件中包含以下信息：

* 有关必须回滚群集升级的通知。
* 建议的补救措施（如果有）。
* 距离执行阶段 2 的天数 (n)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 自电子邮件发送日期的 n 天之后，我们将继续执行阶段 2。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。 这是为了避免发送过多的电子邮件。收到电子邮件则表示出现异常。 大多数群集升级预期都会成功，且不影响应用程序可用性。

### <a name="phase-2-an-upgrade-is-performed-by-using-default-health-policies-only"></a>阶段 2：仅使用默认运行状况策略执行升级
在此阶段设置好运行状况策略，以便在升级开始时运行正常的应用程序数目在升级程序期间保持不变。 与阶段 1 一样，阶段 2 升级过程将每次升级一个升级域，已在群集中运行的应用程序将继续运行，而不会造成任何停机时间。 在升级期间，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 然后，系统会向订阅所有者发送一封电子邮件。 电子邮件中包含以下信息：

* 有关必须回滚群集升级的通知。
* 建议的补救措施（如果有）。
* 距离执行阶段 3 的天数 (n)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 将在 n 天结束前的几天发送提醒电子邮件。 自电子邮件发送日期的 n 天之后，我们将继续执行阶段 3。 必须认真看待阶段 2 发送的电子邮件并采取补救措施。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。

### <a name="phase-3-an-upgrade-is-performed-by-using-aggressive-health-policies"></a>阶段 3：使用积极的群集运行状况策略执行升级
此阶段中的这些运行状况策略旨在升级完成，而不反映应用程序的运行状况。 很少有群集升级在此阶段结束。 如果群集进入此阶段，则表示应用程序有可能变得不正常和/或失去可用性。

类似于另外两个阶段，阶段 3 每次升级一个升级域。

如果不符合现行的群集运行状况策略，则回滚升级。 如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 之后，便会锁定群集，使它不再接收支持和/或升级。

系统会将包含此信息以及补救措施的电子邮件发送给订阅所有者。 预期不会有任何群集遇到阶段 3 失败的状况。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。

## <a name="manage-certificates"></a>管理证书
Service Fabric 使用创建群集时指定的 [X.509 服务器证书](service-fabric-cluster-security.md)，以保护群集节点之间的通信并对客户端进行身份验证。 可以在 [Azure 门户](https://portal.azure.com) 中添加、更新或删除群集和客户端的证书，也可以使用 PowerShell/Azure CLI 完成这些操作。  若要了解详细信息，请参阅[添加或删除证书](service-fabric-cluster-security-update-certs-azure.md)

## <a name="open-application-ports"></a>打开应用程序端口
可以通过更改与节点类型关联的负载均衡器资源属性来更改应用程序端口。 可以使用 Azure 门户，也可以使用 PowerShell/Azure CLI。 有关详细信息，请参阅[打开群集的应用程序端口](create-load-balancer-rule.md)。

## <a name="define-node-properties"></a>定义节点属性
有时，可能需要确保仅在群集中特定类型的节点上运行某些工作负荷。 例如，某些工作负荷可能需要 GPU 或 SSD，而有些则不用。 对于群集中的每个节点类型，可以向群集节点添加自定义节点属性。 放置约束是附加到单个服务的语句，这些服务专供 1 个或多个节点属性选择。 放置约束定义服务运行的位置。

有关使用放置约束、节点属性以及如何定义它们的详细信息，请参阅[节点属性和放置约束](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints)。

## <a name="add-capacity-metrics"></a>添加容量指标
对于每个节点类型，可以添加要在应用程序中用于报告负载的自定义容量度量值。 有关使用容量指标来报告负载的详细信息，请参阅 Service Fabric 群集 Resource Manager 文档[描述群集](service-fabric-cluster-resource-manager-cluster-description.md)，以及[指标和负载](service-fabric-cluster-resource-manager-metrics.md)。

## <a name="set-health-policies-for-automatic-upgrades"></a>设置自动升级的运行状况策略
可以为结构升级指定自定义运行状况策略。 如果已将群集设置为自动结构升级，则这些策略会应用到自动结构升级的阶段 1 中。
如果已将群集设置为手动结构升级，则在每次选择新版本时应用这些策略，来触发系统启动群集中的结构升级。 如果未重写这些策略，则会使用默认设置。

可以在“结构升级”边栏选项卡中选择高级升级设置来指定自定义运行状况策略或查看当前设置。 查看以下图片了解操作方法。 

![管理自定义运行状况策略][HealthPolices]

## <a name="customize-fabric-settings-for-your-cluster"></a>自定义群集的结构设置
可以在群集上自定义许多不同的配置设置，例如群集的可靠性级别和节点属性。 有关详细信息，请参阅 [Service Fabric 群集结构设置](service-fabric-cluster-fabric-settings.md)。

## <a name="patch-the-os-in-the-cluster-nodes"></a>修补群集节点的操作系统
修补业务流程应用程序 (POA) 是一个 Service Fabric 应用程序，可在 Service Fabric 群集中自动修补操作系统，而无需停机。 [适用于 Windows 的修补业务流程应用程序](service-fabric-patch-orchestration-application.md)可部署在群集上，以便以协调一致的方式安装修补程序，同时使服务始终可用。


## <a name="next-steps"></a>后续步骤
* 了解如何自定义 [Service Fabric 群集结构设置](service-fabric-cluster-fabric-settings.md)的部分内容
* 了解如何[扩展和缩减群集](service-fabric-cluster-scale-up-down.md)
* 了解[应用程序升级](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
