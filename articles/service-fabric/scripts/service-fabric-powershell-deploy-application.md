---
title: Azure PowerShell 脚本示例 - 将应用程序部署到群集 | Microsoft Docs
description: Azure PowerShell 脚本示例 - 将应用程序部署到 Service Fabric 群集。
services: service-fabric
documentationcenter: ''
author: aljo-microsoft
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 01/18/2018
ms.author: aljo
ms.custom: mvc
ms.openlocfilehash: 6de7a721a5042a20537d3f09ea9b8de1cc0e7169
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60622027"
---
# <a name="deploy-an-application-to-a-service-fabric-cluster"></a>将应用程序部署到 Service Fabric 群集

此示例脚本将应用程序包复制到群集映像存储，并在群集中注册了应用程序类型，同时还删除了不必要的应用程序包，并根据应用程序类型创建了应用程序实例。  如果目标应用程序类型的应用程序清单中定义了任何默认服务，则此时还会创建那些服务。 根据需要自定义参数。 

必要时，使用 [Service Fabric SDK](../service-fabric-get-started.md) 安装 Service Fabric PowerShell 模块。 

## <a name="sample-script"></a>示例脚本

[!code-powershell[main](../../../powershell_scripts/service-fabric/deploy-application/deploy-application.ps1 "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用[删除应用程序](service-fabric-powershell-remove-application.md)中的脚本删除应用程序实例，取消注册应用程序类型，并从映像存储中删除应用程序包。

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
|[Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps)| 与 Service Fabric 群集建立连接。 |
|[Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) | 将应用程序包复制到群集映像存储。  |
|[Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps)| 在群集上注册应用程序类型和版本。 |
|[New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication?view=azureservicefabricps)| 从注册的应用程序类型创建应用程序。 |
| [Remove-ServiceFabricApplicationPackage](/powershell/module/servicefabric/remove-servicefabricapplicationpackage?view=azureservicefabricps) | 从映像存储区中删除 Service Fabric 应用程序包。|

## <a name="next-steps"></a>后续步骤

有关 Service Fabric PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](/powershell/azure/service-fabric/?view=azureservicefabricps)。

可以在 [Azure PowerShell 示例](../service-fabric-powershell-samples.md)中找到 Azure Service Fabric 的其他 Powershell 示例。
