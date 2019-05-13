---
title: 配置 Azure Function App 设置 | Microsoft Docs
description: 了解如何配置 Azure Function App 设置。
services: ''
documentationcenter: .net
author: ggailey777
manager: jeconnoc
ms.assetid: 81eb04f8-9a27-45bb-bf24-9ab6c30d205c
ms.service: azure-functions
ms.topic: conceptual
ms.date: 03/28/2018
ms.author: glenga
ms.custom: cc996988-fb4f-47
ms.openlocfilehash: 188c17b4e8ef84f3907b63fd62bf110ee94b4d7f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65511243"
---
# <a name="how-to-manage-a-function-app-in-the-azure-portal"></a>如何在 Azure 门户中管理 Function App 

在 Azure Functions 中，Function App 提供各个函数的执行上下文。 Function App 行为适用于由给定 Function App 托管的所有函数。 本主题介绍如何在 Azure 门户中配置和管理 Function App。

要开始，请转到 [Azure 门户](https://portal.azure.com)，并使用 Azure 帐户登录。 在门户顶端的搜索栏中，键入函数应用的名称，并从列表中将其选中。 选择 Function App 后，将看到以下页面：

![Azure 门户中 Function App 的概述](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-main.png)

您可以导航到特别从概述页中，管理 function app 所需的一切**[应用程序设置](#settings)** 并**[平台功能](#platform-features)**.

## <a name="settings"></a>应用程序设置

**应用程序设置**选项卡上维护函数应用使用的设置。

![在 Azure 门户中 function app 设置。](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-settings-tab.png)

这些设置存储加密，并且你必须选择**显示值**若要查看在门户中的值。

若要添加的设置，请选择**新的应用程序设置**并添加新的键 / 值对。

[!INCLUDE [functions-environment-variables](../../includes/functions-environment-variables.md)]

当开发函数应用本地时，这些值会维护 local.settings.json 项目文件中。

## <a name="platform-features"></a>平台功能

![Function App 平台功能选项卡。](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-features-tab.png)

Function App 运行于 Azure 应用服务平台，并由该平台维护。 在这种情况下，Function App 有权访问 Azure 核心 Web 托管平台的大多数功能。 可在“平台功能”选项卡中访问应用服务平台中许多可用于 Function App 的功能。 

> [!NOTE]
> Function App 运行于消耗托管计划中时，并非所有应用服务功能均可用。

本主题的其余部分侧重于 Azure 门户中以下可用于 Functions 的应用服务功能：

+ [应用服务编辑器](#editor)
+ [Console](#console)
+ [高级工具 (Kudu)](#kudu)
+ [部署选项](#deployment)
+ [CORS](#cors)
+ [身份验证](#auth)
+ [API 定义](#swagger)

若要深入了解如何使用应用服务设置，请参阅[配置 Azure 应用服务设置](../app-service/web-sites-configure.md)。

### <a name="editor"></a>应用服务编辑器

| | |
|-|-|
| ![Function App 应用服务编辑器。](./media/functions-how-to-use-azure-function-app-settings/function-app-appsvc-editor.png)  | 应用服务编辑器是一种高级的门户内编辑器，可用于修改诸如 JSON 配置文件和代码文件等内容。 选择此选项会启动单独的浏览器选项卡和基本编辑器。 借此，可与 Git 存储库集成、运行和调试代码，并可修改 Function App 设置。 同默认 Function App 边栏选项卡相比，此编辑器为 Functions 提供了增强的开发环境。    |

![应用服务编辑器](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-appservice-editor.png)

### <a name="console"></a>控制台

| | |
|-|-|
| ![Azure 门户中的 Function App 控制台](./media/functions-how-to-use-azure-function-app-settings/function-app-console.png) | 要从命令行与 Function App 交互时，门户内控制台就是非常合适的开发人员工具。 常见命令包括创建和导航目录与文件，以及执行批处理文件和脚本。 |

![Function App 控制台](./media/functions-how-to-use-azure-function-app-settings/configure-function-console.png)

### <a name="kudu"></a>高级工具 (Kudu)

| | |
|-|-|
| ![Azure 门户中的 Function App Kudu](./media/functions-how-to-use-azure-function-app-settings/function-app-advanced-tools.png) | 应用服务的高级工具（也称为 Kudu）提供对 Function App 高级管理功能的访问。 从 Kudu 中，可以管理系统信息、应用设置、环境变量、站点扩展、HTTP 头和服务器变量。 也可以通过浏览到 Function App 的 SCM 终结点（如 `https://<myfunctionapp>.scm.azurewebsites.net/`），启动 Kudu |

![配置 Kudu](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-kudu.png)


### <a name="a-namedeploymentdeployment-options"></a><a name="deployment">部署选项

| | |
|-|-|
| ![Azure 门户中的 Function App 部署选项](./media/functions-how-to-use-azure-function-app-settings/function-app-deployment-source.png) | Functions 允许在本地计算机上开发函数代码。 然后，可将本地 Function App 项目上传到 Azure。 除了传统的 FTP 上传外，Functions 还允许使用 GitHub、Azure DevOps、Dropbox、Bitbucket 等常用的持续集成解决方案部署函数应用。 有关详细信息，请参阅 [Azure Functions 的连续部署](functions-continuous-deployment.md)。 若要使用 FTP 或本地 Git 进行手动上传，还需[配置部署凭据](functions-continuous-deployment.md#credentials)。 |


### <a name="cors"></a>CORS

| | |
|-|-|
| ![Azure 门户中的 Function App CORS](./media/functions-how-to-use-azure-function-app-settings/function-app-cors.png) | 为了防止在服务中执行破坏性代码，应用服务将阻止从外部源调用 Function App。 Functions 支持跨源资源共享 (CORS)，以便定义允许源的“允许列表”，函数可接受来自该允许源的远程请求。  |

![配置函数应用的 CORS](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-cors.png)

### <a name="auth"></a>身份验证

| | |
|-|-|
| ![Azure 门户中的 Function App 身份验证](./media/functions-how-to-use-azure-function-app-settings/function-app-authentication.png) | 函数使用 HTTP 触发器时，可以要求首先对调用进行身份验证。 应用服务支持 Azure Active Directory 身份验证和使用社交提供程序登录，如 Facebook、Microsoft 和 Twitter。 有关配置特定身份验证提供程序的详细信息，请参阅 [Azure 应用服务身份验证概述](../app-service/overview-authentication-authorization.md)。 |

![配置 Function App 的身份验证](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-authentication.png)


### <a name="swagger"></a>API 定义

| | |
|-|-|
| ![Azure 门户中的 Function App API Swagger 定义](./media/functions-how-to-use-azure-function-app-settings/function-app-api-definition.png) | Functions 支持 Swagger，以便使客户更轻松地使用 HTTP 触发的函数。 有关使用 Swagger 创建 API 定义的详细信息，请访问[在 Azure 应用服务中使用 CORS 托管 RESTful API](../app-service/app-service-web-tutorial-rest-api.md)。 还可以使用函数代理来定义多个函数的单个 API 图面。 有关详细信息，请参阅[使用 Azure Functions 代理](functions-proxies.md)。 |

![配置函数应用的 API](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-apidef.png)



## <a name="next-steps"></a>后续步骤

+ [配置 Azure 应用服务设置](../app-service/web-sites-configure.md)
+ [Azure Functions 的连续部署](functions-continuous-deployment.md)



