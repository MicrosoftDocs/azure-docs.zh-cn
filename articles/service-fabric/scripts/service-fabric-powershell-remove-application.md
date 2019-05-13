---
title: Azure PowerShell 脚本示例 - 从群集中删除应用程序 | Microsoft Docs
description: Azure PowerShell 脚本示例 - 从 Service Fabric 群集中删除应用程序。
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 01/18/2018
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: c8956b8bbbbe6d0928511da3cf75bfd23a76ed21
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60621768"
---
# <a name="remove-an-application-from-a-service-fabric-cluster"></a>从 Service Fabric 群集中删除应用程序

此示例脚本删除了正在运行的 Service Fabric 应用程序实例，并从群集中取消注册了应用程序类型和版本。  删除应用程序实例时也会删除与该应用程序关联的所有正在运行的服务实例。 根据需要自定义参数。 

必要时，使用 [Service Fabric SDK](../service-fabric-get-started.md) 安装 Service Fabric PowerShell 模块。 

## <a name="sample-script"></a>示例脚本

[!code-powershell[main](../../../powershell_scripts/service-fabric/remove-application/remove-application.ps1 "Remove an application from a cluster")]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication?view=azureservicefabricps) | 从群集中删除正在运行的 Service Fabric 应用程序实例。  |
| [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps) | 从群集中注销 Service Fabric 应用程序类型和版本。 |

## <a name="next-steps"></a>后续步骤

有关 Service Fabric PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](/powershell/azure/service-fabric/?view=azureservicefabricps)。

可以在 [Azure PowerShell 示例](../service-fabric-powershell-samples.md)中找到 Azure Service Fabric 的其他 Powershell 示例。
