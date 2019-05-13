---
title: 有关使用 Azure 应用程序配置管理功能标志的教程 | Microsoft Docs
description: 本教程介绍如何使用 Azure 应用程序配置单独从应用程序中管理功能标志
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/19/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: d995a2e9f0d05dc3b0853036e8fb3c04ccdfab96
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412286"
---
# <a name="tutorial-manage-feature-flags-in-azure-app-configuration"></a>教程：在 Azure 应用程序配置中管理功能标志

可将所有功能标志存储在 Azure 应用程序配置中，并从一个位置对其进行管理。 Azure 应用程序配置提供一个名为“功能管理器”的门户 UI 专门用于管理功能标志。 此外，应用程序配置原生支持 .NET Core 功能标志数据架构。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 在应用程序配置中定义和管理功能标志。
> * 从应用程序访问功能标志。

## <a name="create-feature-flags"></a>创建功能标志

应用程序配置的 Azure 门户中的“功能管理器”提供一个 UI 用于创建和管理可在应用程序中使用的功能标志。 遵循以下步骤添加新的功能标志。

1. 选择“功能管理器” > “+ 创建”以添加功能标志。

    ![功能标志列表](./media/azure-app-configuration-feature-flags.png)

2. 输入功能标志的唯一键名称。 需要使用名称在代码中引用标志。

3. （可选）为功能标志提供更友好的说明。

4. 设置功能标志的初始状态。 该状态通常是“开”或“关”。

    ![创建功能标志](./media/azure-app-configuration-feature-flag-create.png)

5. 当状态为“开”时，可以选择性地使用“添加筛选器”来指定任何附加条件，以限定该标志。 输入内置或自定义的筛选器键并关联参数。 内置筛选器包括：

    | 密钥 | JSON 参数 |
    |---|---|
    | Microsoft.Percentage | {"Value":0-100 percent} |
    | Microsoft.TimeWindow | {"Start":UTC time, "End":UTC time} |

    ![功能标志筛选器](./media/azure-app-configuration-feature-flag-filter.png)

## <a name="update-feature-flag-states"></a>更新功能标志状态

遵循以下步骤更改功能标志的状态值。

1. 选择“功能管理器”。

2. 单击要修改的功能标志右侧的“...” > “编辑”。

3. 设置功能标志的新状态。

## <a name="access-feature-flags"></a>访问功能标志

“功能管理器”创建的功能标志作为常规的键-值进行存储和检索。 它们保留在特殊命名空间前缀 *.appconfig.featureflag* 下。 可以使用“配置资源管理器”来查看基础键-值。 应用程序可以使用应用配置的配置提供程序、SDK、命令行扩展和 REST API 来检索这些键-值。

## <a name="next-steps"></a>后续步骤

本教程已介绍如何使用应用程序配置管理功能标志及其状态。 有关应用程序配置和 ASP.NET Core 中的功能管理支持的详细信息，请参阅以下资源。

* [在 ASP.NET Core 应用中使用功能标志](./use-feature-flags-dotnet-core.md)
