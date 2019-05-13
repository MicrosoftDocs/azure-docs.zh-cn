---
title: Azure CLI 脚本示例 - 将自定义 SSL 证书绑定到 Function App | Microsoft Docs
description: Azure CLI 脚本示例 - 将自定义 SSL 证书绑定到 Azure 中的 Function App
services: functions
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.assetid: eb95d350-81ea-4145-a1e2-6eea3b7469b2
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/03/2013
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 84cc4e92419e683f8d2c29883b26148fdef23356
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60325714"
---
# <a name="bind-a-custom-ssl-certificate-to-a-function-app"></a>将自定义 SSL 证书绑定到 Function App

此示例脚本在应用服务中创建一个 Function App 及其相关资源，然后将自定义域名的 SSL 证书绑定到该应用。 在此示例中，需要以下项：

* 对域注册机构的 DNS 配置页的访问权限。
* 要上传和绑定的 SSL 证书的有效 .PFX 文件及其密码。
* 已在自定义域中配置了一个指向 Web 应用的默认域名的 A 记录。 有关详细信息，请参阅[适用于 Azure 应用服务的映射自定义域说明](https://aka.ms/appservicecustomdns)。

若要绑定 SSL 证书，必须在应用服务计划（而不是消耗计划）中创建 Function App。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果选择本地安装和使用 CLI，必须运行 Azure CLI 版本 2.0 或更高版本。 运行 `az --version` 即可查找版本。 如需进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>示例脚本

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom SSL certificate to a web app")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 创建 Function App 所需的存储帐户。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create) | 创建绑定 SSL 证书所需的应用服务计划。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在应用服务计划中创建函数应用。 |
| [az functionapp config hostname add](https://docs.microsoft.com/cli/azure/functionapp/config/hostname#az-functionapp-config-hostname-add) | 将自定义域映射到 Function App。 |
| [az functionapp config ssl upload](https://docs.microsoft.com/cli/azure/functionapp/config/ssl#az-functionapp-config-ssl-upload) | 将 SSL 证书上传到 Function App。 |
| [az functionapp config ssl bind](https://docs.microsoft.com/cli/azure/functionapp/config/ssl#az-functionapp-config-ssl-bind) | 将上传的 SSL 证书绑定到 Function App。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure)。

可以在 [Azure 应用服务文档](../functions-cli-samples.md)中找到其他应用服务 CLI 脚本示例。
