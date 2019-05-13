---
title: 将 Cloud App Security 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Cloud App Security 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: cd9e5e27-fdd4-4717-8924-be4c1c430f23
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: a418bb318654752eaf48ffbdd05b80cabb487ece
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65207553"
---
# <a name="connect-data-from-microsoft-cloud-app-security"></a>从 Microsoft Cloud App Security 连接数据 

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

可以从日志流式传输[Cloud App Security](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security)到 Azure Sentinel 单击一次。 此连接，可从 Cloud App Security 警报流到 Azure Sentinel。 

## <a name="prerequisites"></a>必备组件

- 具有全局管理员或安全管理员权限的用户

## <a name="connect-to-cloud-app-security"></a>连接到 Cloud App Security

如果已有 Cloud App Security，请确保它是[在网络上启用](https://docs.microsoft.com/cloud-app-security/getting-started-with-cloud-app-security)。
如果部署 Cloud App Security，引入你的数据，警报数据可以轻松地进行流式传输到 Azure Sentinel。


1. 在 Azure Sentinel，选择**数据连接器**，然后单击**Cloud App Security**磁贴。

2. 单击“连接”。

3. 若要使用 Log Analytics 中的 Cloud App Security 警报相关的架构，搜索**SecurityAlert**。


## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Microsoft Cloud App Security 连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
