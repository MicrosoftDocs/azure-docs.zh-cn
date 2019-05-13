---
title: Azure CLI 脚本示例 - 获取托管资源组和重设 VM 大小 | Microsoft Docs
description: Azure CLI 脚本示例 - 获取托管资源组和重设 VM 大小
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/25/2017
ms.author: tomfitz
ms.openlocfilehash: bbf03a0d53769c93a8aab304d3128ae0cc875a8f
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61364547"
---
# <a name="get-resources-in-a-managed-resource-group-and-resize-vms-with-azure-cli"></a>使用 Azure CLI 获取托管资源组中的资源并重设 VM 大小

此脚本从托管资源组中检索资源，并重设该资源组中 VM 的大小。


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-azurecli[main](../../../cli_scripts/managed-applications/get-application/get-application.sh "Get application")]


## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令部署托管应用程序。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az managedapp list](https://docs.microsoft.com/cli/azure/managedapp#az-managedapp-list) | 列出托管应用程序。 提供要重点关注结果的查询值。 |
| [az resource list](https://docs.microsoft.com/cli/azure/resource#az-resource-list) | 列出资源。 提供要重点关注结果的资源组和查询值。 |
| [az vm resize](https://docs.microsoft.com/cli/azure/vm#az-vm-resize) | 更新虚拟机的大小。 |


## <a name="next-steps"></a>后续步骤

* 有关托管应用程序的简介，请参阅 [Azure 托管应用程序概述](../overview.md)。
* 有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure)。
