---
title: Azure CLI 脚本示例 - 创建托管应用程序定义 | Microsoft Docs
description: Azure CLI 脚本示例 - 创建托管应用程序定义
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
ms.openlocfilehash: 79af2918c2342d1a0cc3b11763c30b35ab5e9f72
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61043880"
---
# <a name="create-a-managed-application-definition-with-azure-cli"></a>使用 Azure CLI 创建托管应用程序定义

此脚本会将托管应用程序定义发布到服务目录。 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-azurecli[main](../../../cli_scripts/managed-applications/create-definition/create-definition.sh "Create definition")]


## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建托管应用程序定义。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az managedapp definition create](https://docs.microsoft.com/cli/azure/managedapp/definition#az-managedapp-definition-create) | 创建托管应用程序定义。 提供包含所需文件的包。 |


## <a name="next-steps"></a>后续步骤

* 有关托管应用程序的简介，请参阅 [Azure 托管应用程序概述](../overview.md)。
* 有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure)。
