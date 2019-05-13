---
title: 使用 Azure CLI 创建媒体服务帐户 - Azure | Microsoft Docs
description: 按照本快速入门的步骤，创建 Azure 媒体服务帐户。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: seodec18
ms.date: 02/15/2019
ms.author: juliako
ms.openlocfilehash: 62d26ce5a2e1a98b6ed024fb98c9c887fb985ebf
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150981"
---
# <a name="create-an-azure-media-services-account"></a>创建 Azure 媒体服务帐户

若要开始加密、编码、分析、管理和流式处理 Azure 中的媒体内容，需要创建媒体服务帐户。 媒体服务帐户需与一个或多个存储帐户相关联。

> [!NOTE]
> 媒体服务帐户和所有关联的存储帐户必须位于同一 Azure 订阅中。 强烈建议在媒体服务帐户所在的位置使用存储帐户，避免额外的延迟和数据出口成本。

本文介绍使用 Azure CLI 创建新 Azure 媒体服务帐户的步骤。  

## <a name="prerequisites"></a>必备组件

一个有效的 Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="set-the-azure-subscription"></a>设置 Azure 订阅

在以下命令中，为媒体服务帐户提供想要使用的 Azure 订阅 ID。 导航到[订阅](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)即可查看有权访问的订阅列表。

```azurecli
az account set --subscription mySubscriptionId
```
 
[!INCLUDE [media-services-cli-create-v3-account-include](../../../includes/media-services-cli-create-v3-account-include.md)]
 
## <a name="next-steps"></a>后续步骤

* [访问 v3 Api](access-api-cli-how-to.md)
* [流式传输文件](stream-files-dotnet-quickstart.md)

## <a name="see-also"></a>另请参阅

[Azure CLI](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)

