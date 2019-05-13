---
title: 升级到 Azure 搜索 .NET 管理 SDK 版本 2 - Azure 搜索
description: 从以前版本升级到 Azure 搜索 .NET 管理 SDK 版本 2。 了解新增功能和所需的代码更改。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/02/2019
ms.openlocfilehash: 62c2ed555fcac56677f4950c10d38ded8fb0649d
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025177"
---
# <a name="upgrading-to-the-azure-search-net-management-sdk-version"></a>升级到 Azure 搜索.NET 管理 SDK 版本 

> [!Important]
> 此内容是仍在构建。 NuGet 上提供的 Azure 搜索管理.NET SDK 版本 3.0。 我们正在努力更新本迁移指南说明如何升级到最新版本。 
>

如果使用的是版本 1.0.2 或更早版本的 [Azure 搜索 .NET 管理 SDK](https://aka.ms/search-mgmt-sdk)，本文有助于升级应用程序，以便使用版本 2。

版本 2 的 Azure 搜索 .NET 管理 SDK 包含了某些针对早期版本进行的更改。 这些更改主要涉及次要版本，因此更改代码的工作量并不是太大。 有关如何更改代码以使用新版 SDK 的说明，请参阅[升级步骤](#UpgradeSteps)。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-2"></a>版本 2 中的新增功能
版本 2 的 Azure 搜索 .NET 管理 SDK 与以前的 SDK 版本面向相同的 Azure 搜索管理 REST API 公开发行版本，具体而言是 2015-08-19 版。 针对 SDK 的更改是严格的客户端更改，以便提高 SDK 本身的可用性。 这些更改包括以下内容：

* `Services.CreateOrUpdate` 及其异步版本现在会自动轮询预配 `SearchService`，且在服务预配完成后才返回。 因此，无需自己编写此类轮询代码。
* 如果仍想手动轮询服务预配，可使用新的 `Services.BeginCreateOrUpdate` 方法或其异步版本之一。
* 新方法 `Services.Update` 及其异步版本已添加到 SDK。 这些方法使用 HTTP PATCH 来支持服务的增量更新。 例如，现在可以通过将 `SearchService` 实例传递到仅包含所需 `partitionCount` 和 `replicaCount` 属性的方法来缩放服务。 仍支持调用 `Services.Get`、修改返回的 `SearchService` 并将其传递给 `Services.CreateOrUpdate` 这一旧方法，但该方法不再是必要的。 

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升级步骤
首先，按以下方法操作更新 `Microsoft.Azure.Management.Search` 的 NuGet 引用：使用 NuGet 包管理器控制台，或者在 Visual Studio 中右键单击项目引用，然后选择“管理 NuGet 程序包...”。

在 NuGet 已下载新的程序包及其依赖项后，请重新生成项目。 项目重新生成可能会成功，具体取决于代码的结构。 如果成功，一切准备就绪！

如果生成失败，则可能是因为已实现某些已更改的 SDK 接口（如出于单元测试目）。 若要解决此问题，需要实现 `BeginCreateOrUpdateWithHttpMessagesAsync` 等新方法。

在修复了任何生成错误后，可以对应用程序进行更改，以利用新功能（如果愿意）。 有关 SDK 中的新功能的详细信息，请参阅[版本 2 中的新增功能](#WhatsNew)。

## <a name="conclusion"></a>结束语
我们欢迎你对 SDK 提供反馈。 如果遇到问题，请随时通过 [Azure 搜索 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/home?forum=azuresearch)向我们寻求帮助。 如果找到 Bug，可以在 [Azure .NET SDK GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/issues)中提出问题。 务必在问题标题上加前缀“[Azure 搜索]”。

感谢使用 Azure 搜索！
