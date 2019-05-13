---
title: Azure 状态监视器 v2 API 参考：禁用检测引擎 |Microsoft Docs
description: 状态监视器 v2 API 引用禁用-InstrumentationEngine。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
services: application-insights
documentationcenter: .net
author: MS-TimothyMothra
manager: alexklim
ms.assetid: 769a5ea4-a8c6-4c18-b46c-657e864e24de
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: tilee
ms.openlocfilehash: 7d1b658da59c0e778c82301077ef27bdd9c9e614
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65144906"
---
# <a name="status-monitor-v2-api-disable-instrumentationengine-v021-alpha"></a>状态监视器 v2 API:禁用 InstrumentationEngine (v0.2.1-alpha)

本文档介绍了作为的成员提供的 cmdlet [Az.ApplicationMonitor PowerShell 模块](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/)。

> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="description"></a>描述

此 cmdlet 将禁用检测引擎通过删除一些注册表项。
重新启动 IIS 以实现这些更改才会生效。

> [!IMPORTANT] 
> 此 cmdlet 需要具有管理员权限的 PowerShell 会话。

## <a name="examples"></a>示例

```powershell
PS C:\> Disable-InstrumentationEngine
```

## <a name="parameters"></a>parameters 

### <a name="-verbose"></a>-Verbose
**通用参数。** 使用此开关以输出详细的日志。

## <a name="output"></a>输出


#### <a name="example-output-from-successfully-disabling-the-instrumentation-engine"></a>已成功禁用检测引擎的示例输出

```
Configuring IIS Environment for instrumentation engine...
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]'
Configuring registry for instrumentation engine...
```


## <a name="next-steps"></a>后续步骤

 用做更多状态监视器 v2:
 - 使用指南，了解如何[疑难解答](status-monitor-v2-troubleshoot.md)状态监视器 v2。
