---
title: 启用 Azure Sentinel 预览版中的合成 |Microsoft Docs
description: 了解如何启用 Azure Sentinel 中的合成。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 82becf50-6628-47e4-b3d7-18d7d72d505f
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2019
ms.author: rkarlin
ms.openlocfilehash: bf9a78006d13614a73a3fccfc57f28ce850456d9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204474"
---
# <a name="enable-fusion"></a>启用融合

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

在 Azure Sentinel 机器学习是从一开始就内置。 与针对高效地进行安全分析师、 安全数据科学家和工程师的机器学习创新，我们已经经过深思熟虑设计系统。 此类的一个创新是 Azure Sentinel 合成尤其是旨在减少警报疲劳。

合成使用 graph 提供支持的机器学习算法数以百万计的低保真度异常活动，从不同的产品，例如 Azure AD Identity Protection 和 Microsoft Cloud App Security，以将它们组合成便于管理的数之间关联起来有趣的安全情况。

## <a name="enable-fusion"></a>启用融合

1. 在 Azure 门户中，选择该图标可以打开 Cloud Shell。
  ![Cloud Shell](./media/connect-fusion/cloud-shell.png)

2.  在中**欢迎使用 Cloud Shell**打开下面，选择 PowerShell。

3.  选择要在其部署 Azure Sentinel 的订阅并**创建存储**。

4. 后进行身份验证和生成 Azure 驱动器后，在命令提示符处，运行以下命令：

            az resource update --ids /subscriptions/{Subscription Guid}/resourceGroups/{Log analytics resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Log analytics workspace Name}/providers/Microsoft.SecurityInsights/settings/Fusion --api-version 2019-01-01-preview --set properties.IsEnabled=true --subscription "{Subscription Guid}"

## <a name="disable-fusion"></a>禁用合成

按照相同的步骤与上面一样，并运行以下命令：

            az resource update --ids /subscriptions/{Subscription Guid}/resourceGroups/{Log analytics resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Log analytics workspace Name}/providers/Microsoft.SecurityInsights/settings/Fusion --api-version 2019-01-01-preview --set properties.IsEnabled=false --subscription "{Subscription Guid}"

## <a name="view-the-status-of-fusion"></a>查看合成的状态

            az resource show --ids /subscriptions/{Subscription Guid}/resourceGroups/{Log analytics resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Log analytics workspace Name}/providers/Microsoft.SecurityInsights/settings/Fusion --api-version 2019-01-01-preview --subscription "{Subscription Guid}"


## <a name="next-steps"></a>后续步骤

在本文档中，您学习了如何能够在 Azure Sentinel 合成。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

