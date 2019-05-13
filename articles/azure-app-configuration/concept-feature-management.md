---
title: Azure 应用程序配置功能管理 | Microsoft Docs
description: 概述如何使用 Azure 应用程序配置来按需启用和禁用应用程序功能
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.service: azure-app-configuration
ms.devlang: na
ms.topic: overview
ms.workload: tbd
ms.date: 04/19/2019
ms.author: yegu
ms.openlocfilehash: 304ca08275ccf4bf32d38e3fc89dd5cf07460fa4
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413772"
---
# <a name="feature-management"></a>特色管理

一直以来，交付新的应用程序功能都需要完全重新部署该应用程序本身。 若要测试某项功能，可能需要多次部署应用程序，以控制何时显示该功能，或谁可以查看该功能。 功能管理是一种新式软件开发实践，它将功能发布与代码部署分离开来，可以按需快速更改功能的可用性。 它使用一种称作“功能标志”（也称为“功能切换”、“功能开关”等）的技术来动态管理功能的生命周期。 它可以帮助开发人员处理以下问题：

* **代码分支管理**：功能标志经常用于包装正在开发中的新应用程序功能。 此类功能默认是“隐藏的”。 不过，可以在此功能尚未完成的情况下交付它，它可以在生产环境中保持主导地位。 使用这种称作“暗处部署”的方法，可以在每个开发周期结束时发布所有代码。 不再需要在多个周期中维护任何代码分支，因为完成某项功能需要经过多个期段。
* **在生产环境中测试**：在生产环境中，可以使用功能标志提前授予对新功能的访问权限。 例如，可将此访问权限限制为团队成员或内部 Beta 测试人员。 这些用户将获得完全保真的生产体验，而无需在测试环境中使用模拟或部分体验。
* **外部测试**：还可以使用功能标志以增量方式将新功能推出给最终用户。 一开始可以面向少量的用户，在实施方面有了更大的自信心之后，可以逐渐提高目标用户百分比。
* **即时终止开关**：功能标志提供一个固有的安全网络用于发布新功能。 使用功能标志不仅可以启用应用程序功能，而且还能即时将其关闭。 可以根据需要快速禁用某项功能，而无需重新生成再重新部署应用程序。
* **选择性激活**：尽管大多数功能标志只会保留到其相关功能成功发布为止，但有些功能标志可以持续很长时间。 使用此类功能标志可以划分用户，并将特定的一组功能交付给每个组用户。 例如，你可能在特定的 Web 浏览器中使用了某项功能。 可以定义一个功能标志，以便只有运行该浏览器的用户能够查看和使用该功能。 此方法的优点是，以后可以轻松扩展支持的浏览器列表，而无需进行任何代码更改。

## <a name="basic-concepts"></a>基本概念

下面是与功能管理相关的几个新术语。

* **功能标志**：功能标志是具有 *on* 或 *off* 二进制状态且包含关联代码块的变量。 其状态决定了代码块是否运行。
* **功能管理器**：功能管理器是一个应用程序包，可以处理应用程序中所有功能标志的生命周期。 它通常提供附加的功能，例如标志缓存和标志状态更新。
* **筛选器**：筛选器是用于评估功能标志状态的规则。 用户组、设备或浏览器类型、地理位置和时间窗口都是可用作筛选依据的示例。

功能管理的有效实施方案至少包括两个协同工作的组件：利用功能标志的应用程序，以及用于存储功能标志及其当前状态的独立存储库。 下面演示了这两个组件的交互。

## <a name="feature-flag-usage-in-code"></a>在代码中使用功能标志

在应用程序中实施功能标志的基本模式很简单。 可将其概念化为与代码中的 `if` 条件语句配合使用的布尔值状态变量。

```csharp
if (featureFlag) {
    // Run the following code
}
```

在本例中，如果 `featureFlag` 设置为 `True`，则会执行包含的代码块；否则跳过该代码块。 可以静态设置 `featureFlag` 值：

```csharp
bool featureFlag = true;
```

也可以根据特定的规则评估该值：

```csharp
bool featureFlag = isBetaUser();
```

稍微复杂一些的功能标志模式还包含 `else` 语句。

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
} else {
    // This following code will run if the featureFlag value is false
}
```

但是，可在基本模式中重写此功能标志。 [在 ASP.NET Core 应用中使用功能标志](./use-feature-flags-dotnet-core.md)介绍了基于简单代码模式进行标准化的优点。

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
}

if (!featureFlag) {
    // This following code will run if the featureFlag value is false
}
```

## <a name="feature-flag-repository"></a>功能标志存储库

若要有效使用功能标志，需要外部化应用程序中使用的所有功能标志。 这样便可以更改功能标志状态，而无需修改再重新部署应用程序本身。 Azure 应用程序配置旨在充当功能标志的集中式存储库。 可以利用它来定义不同类型的功能标志，并快速且有把握地操作其状态。 然后，可以使用适用于各种编程语言框架的应用程序配置库，轻松从应用程序访问这些功能标志。 [在 ASP.NET Core 应用中使用功能标志](./use-feature-flags-dotnet-core.md)介绍了如何结合使用 .NET Core 应用程序配置提供程序和功能管理库作为一个完整的解决方案，来针对 ASP.NET Web 应用程序实施功能标志。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [将功能标志添加到 ASP.NET Core Web 应用](./quickstart-feature-flag-aspnet-core.md)  
