---
title: Azure 监视器应用程序更改分析-发现分析，更改可能会影响实时站点问题/中断与 Azure Monitor 应用程序的更改 |Microsoft Docs
description: 对 Azure 应用服务上使用 Azure Monitor 应用程序更改分析的应用程序实时站点问题进行故障排除
services: application-insights
author: cawams
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: cawa
ms.openlocfilehash: b04bd57c66b52e9a2c14d43c9e89d7c54fc48ae2
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415848"
---
# <a name="azure-monitor-application-change-analysis-public-preview"></a>Azure 监视器应用程序更改分析 （公共预览版）

当实时站点问题/中断发生时，快速确定根本原因是关键。 标准监视解决方案可帮助您快速确定问题，并且通常，即使哪个组件发生故障。 但这始终不会导致立即发生失败的原因的理由。 你的站点过五分钟之前，现在它已损坏。 在过去 5 分钟内发生什么变化？ 这是新功能 Azure Monitor 应用程序更改分析旨在回答的问题。 通过基于的强大功能构建[Azure 资源 Graph](https://docs.microsoft.com/azure/governance/resource-graph/overview)应用程序更改分析提供了深入了解 Azure 应用服务应用程序更改来提高可观察性并减少 MTTR （平均时间修复）。

> [!IMPORTANT]
> Azure 监视器应用程序更改分析当前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="how-do-i-use-application-change-analysis"></a>如何使用应用程序更改分析？

Azure 监视器应用程序更改分析当前内置到自助服务**诊断并解决问题**体验，可以从访问哪一**概述**部分中的 Azure 应用服务应用程序：

![Azure 应用服务的屏幕截图概述页上概述按钮周围的红框和诊断并解决问题按钮](./media/change-analysis/change-analysis.png)

### <a name="enable-change-analysis"></a>启用更改分析

1. 选择**可用性和性能**

    ![可用性和性能故障排除选项的屏幕截图](./media/change-analysis/availability-and-performance.png)

2. 单击**应用程序崩溃**磁贴。

   ![与应用程序崩溃磁贴的屏幕截图](./media/change-analysis/application-crashes-tile.png)

3. 若要启用**更改 Analysis**选择**立即启用**。

   ![可用性和性能故障排除选项的屏幕截图](./media/change-analysis/application-crashes.png)

4. 若要充分利用完整更改分析功能集**更改 Analysis**，**扫描检测代码改动**，并**始终打开**到**上**然后选择**保存**。

    ![Azure 应用服务启用更改分析用户界面的屏幕截图](./media/change-analysis/change-analysis-on.png)

    如果**更改分析**是启用，你将能够检测资源级别的更改。 如果**扫描代码更改**是启用，你将还请参阅部署文件和站点配置更改。 启用**始终打开**将优化更改扫描的性能，但可能会导致附加成本从计费角度来看。

5.  启用的所有内容后，选择**诊断并解决问题** > **可用性和性能** > **应用程序崩溃**将允许你访问更改分析体验。 在关系图将 summerize 的这些更改随时间的详细信息以及发生的更改的类型：

     ![更改差异视图的屏幕截图](./media/change-analysis/change-view.png)

## <a name="troubleshooting"></a>故障排除

### <a name="unable-to-fetch-change-analysis-information"></a>无法提取更改分析信息。

这是与当前的预览版载入体验是个暂时性问题。 解决方法由在 web 应用上设置隐藏的标记，然后刷新页面组成：

   ![更改隐藏标记的屏幕截图](./media/change-analysis/hidden-tag.png)

若要设置使用 PowerShell 的隐藏的标记：

1. 打开 Azure Cloud Shell:

    ![更改 Azure Cloud Shell 的屏幕截图](./media/change-analysis/cloud-shell.png)

2. Shell 类型更改为 PowerShell:

   ![更改 Azure Cloud Shell 的屏幕截图](./media/change-analysis/choose-powershell.png)

3. 运行以下命令：

```powershell
$webapp=Get-AzWebApp -Name <name_of_your_webapp>
$tags = $webapp.Tags
$tags[“hidden-related:diagnostics/changeAnalysisScanEnabled”]=$true
Set-AzResource -ResourceId <your_webapp_resourceid> -Tag $tag
```

> [!NOTE]
> 隐藏的标记添加后，你可能仍需要最初等待最多 4 小时，以便能够在第一次查看更改。 这是因为 4 小时 freqeuncy 更改分析服务用来扫描 web 应用，又限制了扫描的性能影响。

## <a name="next-steps"></a>后续步骤

- 提高 Azure 应用服务的监视[通过启用 Application Insights 功能](azure-web-apps.md)的 Azure Monitor。
- 增强的对理解[Azure 资源 Graph](https://docs.microsoft.com/azure/governance/resource-graph/overview)从而可帮助 power Azure Monitor 应用程序更改分析。 
