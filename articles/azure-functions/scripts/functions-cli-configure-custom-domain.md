---
title: Azure CLI 脚本示例 - 将自定义域映射到 Function App | Microsoft Docs
description: Azure CLI 脚本示例 - 将自定义域映射到 Azure 中的 Function App。
services: functions
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.assetid: d127e347-7581-47d7-b289-e0f51f2fbfbc
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/04/2018
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 8874039d148cc89d3f47979370e64dd78777f268
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60325742"
---
# <a name="map-a-custom-domain-to-a-function-app"></a>将自定义域映射到 Function App

此示例脚本在应用服务计划中创建函数应用，然后将其映射到你提供的自定义域。 当函数应用承载在[应用服务计划](../functions-scale.md#app-service-plan)中时，可以使用 CNAME 或 A 记录来映射自定义域。 对于[消耗计划](../functions-scale.md#consumption-plan)中的函数应用，仅支持 CNAME 选项。 此示例创建一个应用服务计划，并需要一个 A 记录来映射域。 

若要运行此示例脚本，必须已在自定义域中配置了一个指向 Web 应用的默认域名的 A 记录。 有关详细信息，请参阅[适用于 Azure 应用服务的映射自定义域说明](https://aka.ms/appservicecustomdns)。 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果选择本地安装和使用 CLI，必须使用 Azure CLI 版本 2.0 或更高版本。 要查找版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="sample-script"></a>示例脚本

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/configure-custom-domain/configure-custom-domain.sh?highlight=3 "Map a custom domain to a function app")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令：表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 创建 Function App 所需的存储帐户。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create) | 创建映射自定义域所需的应用服务计划。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在应用服务计划中创建函数应用。 |
| [az functionapp config hostname add](https://docs.microsoft.com/cli/azure/functionapp/config/hostname#az-functionapp-config-hostname-add) | 将自定义域映射到 Function App。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure)。

可以在 [Azure Functions 文档](../functions-cli-samples.md)中找到其他 Functions CLI 脚本示例。
