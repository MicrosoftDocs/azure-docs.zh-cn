---
title: 使用 Azure 自动化 (经典) 管理 Azure 云服务 |Microsoft Docs
description: 了解如何使用 Azure 自动化服务管理大规模 Azure 云服务。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 530efd09f3632637c6a12648495dcff0e7bf0e6d
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "98743485"
---
# <a name="managing-azure-cloud-services-classic-using-azure-automation"></a>使用 Azure 自动化管理 Azure 云服务 (经典) 

> [!IMPORTANT]
> [Azure 云服务 (扩展支持) ](../cloud-services-extended-support/overview.md) 是适用于 Azure 云服务产品的新的基于 azure 资源管理器的部署模型。进行此更改后，基于 Azure Service Manager 的部署模型运行的 Azure 云服务已重命名为云服务 (经典) ，所有新部署应使用 [云服务 (扩展支持) ](../cloud-services-extended-support/overview.md)。
本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure 云服务的管理。

## <a name="what-is-azure-automation"></a>什么是 Azure 自动化？
[Azure 自动化](https://azure.microsoft.com/services/automation/)是用于通过流程自动化简化云管理的一项 Azure 服务。 使用 Azure 自动化可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化能够提供高度可靠且长期可用的工作流执行引擎，它可以在组织的发展过程中根据需求进行扩展。 利用 Azure 自动化，流程可以手动、通过第三方系统或按计划时间间隔启动，从而使任务完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT/DevOps 人员，让他们将精力集中在增加企业价值的工作上。

## <a name="how-can-azure-automation-help-manage-azure-cloud-services"></a>Azure 自动化如何帮助管理 Azure 云服务？
可以使用 [Azure PowerShell 工具](/powershell/)中提供的 PowerShell cmdlet 在 Azure 自动化中管理 Azure 云服务。 Azure 自动化现成地提供了这些云服务 PowerShell cmdlet，因此，可以在该服务中执行所有云服务管理任务。 还可以将 Azure 自动化中的 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

使用 Azure 自动化管理 Azure 云服务的部分示例包括：

* [每当在 Azure Blob 存储中更新了 cscfg 或 cspkg 时持续部署云服务](https://gallery.technet.microsoft.com/scriptcenter/Continuous-Deployment-of-A-eeebf3a6)
* [并行重启云服务实例，每次重启一个升级域](https://gallery.technet.microsoft.com/scriptcenter/Reboot-Cloud-Service-PaaS-b337a06d)

## <a name="next-steps"></a>后续步骤
了解 Azure 自动化以及如何使用它来管理 Azure 云服务的基础知识后，请点击以下链接了解有关 Azure 自动化的更多信息。

* [Azure 自动化概述](../automation/automation-intro.md)
* [我的第一个 Runbook](../automation/learn/automation-tutorial-runbook-graphical.md)