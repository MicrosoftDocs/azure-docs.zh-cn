---
title: OS 和运行时修补频率 - Azure 应用服务 | Microsoft Docs
description: 介绍 Azure 应用服务如何更新 OS 和运行时，以及如何获取更新公告。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: 086f5773a8ae4085a8e5bc0637bdebe5f2df4fb0
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203337"
---
# <a name="os-and-runtime-patching-in-azure-app-service"></a>Azure 应用服务中的 OS 和运行时修补

本文介绍如何有关[应用服务](overview.md)中的 OS 或软件的特定版本信息。 

应用服务是一种平台即服务，这意味着，Azure 会自行管理 OS 和应用程序堆栈，而你只需管理自己的应用程序及其数据。 在 [Azure 虚拟机](https://docs.microsoft.com/azure/virtual-machines/)中，能够以更高的力度控制 OS 和应用程序堆栈。 考虑到这一点，应用服务用户了解以下方面的详细信息总会有好处：

-   如何以及何时应用 OS 更新？
-   应用服务如何修补重大漏洞（例如零日漏洞）？
-   应用运行哪些 OS 和运行时版本？

出于安全原因，有些具体的安全信息不会公布。 但是，文章旨在缓解安全忧虑，它会最大程度地将该过程透明化，并介绍如何获取最新的安全相关公告或运行时更新。

## <a name="how-and-when-are-os-updates-applied"></a>如何以及何时应用 OS 更新？

Azure 管理两个级别的 OS 修补：运行应用服务资源的物理服务器和来宾虚拟机 (VM)。 这两种方案会根据每月的[周二修补](https://technet.microsoft.com/security/bulletins.aspx)计划更新。 这些更新会自动应用，保证达到 Azure 服务的高可用性 SLA。 

有关如何应用更新的详细信息，请参阅[揭示应用服务 OS 背后的秘密](https://azure.github.io/AppService/2018/01/18/Demystifying-the-magic-behind-App-Service-OS-updates.html)。

## <a name="how-does-azure-deal-with-significant-vulnerabilities"></a>Azure 如何处理重大漏洞？

严重的漏洞（例如[零日漏洞](https://wikipedia.org/wiki/Zero-day_(computing))）需要立即修补，将会根据案例以高优先级处理更新。

访问 [Azure 安全博客](https://azure.microsoft.com/blog/topics/security/)，及时了解 Azure 中的关键安全公告。 

## <a name="when-are-supported-language-runtimes-updated-added-or-deprecated"></a>何时更新、添加或弃用受支持的语言运行时？

受支持语言运行时的新稳定版本（主要、次要或修补版本）会定期添加到应用服务实例。 一些更新会覆盖现有的安装，还有一些更新会连同现有的版本一并安装。 覆盖安装意味着应用自动在更新的运行时上运行。 并列安装意味着必须手动迁移应用才能利用新的运行时版本。 有关详细信息，请参阅以下小节之一。

以下网页中公布了运行时更新和弃用情况：

- https://azure.microsoft.com/updates/?product=app-service 
- https://github.com/Azure/app-service-announcements/issues

> [!NOTE] 
> 此处的信息适用于应用服务应用中内置的语言运行时。 例如，上传到应用服务的自定义运行时将保持不变，除非手动升级。
>
>

### <a name="new-patch-updates"></a>新的修补更新

对 .NET、PHP、Java SDK 或 Tomcat/Jetty 版本所做的修补更新会通过将现有版本覆盖为新版本来自动应用。 Node.js 修补更新将与现有版本一并安装（类似于下一部分中的主要版本和次要版本）。 可以通过[站点扩展](https://www.siteextensions.net/packages?q=Tags%3A%22python%22)，与内置的 Python 安装一并手动安装新的 Python 修补版本。

### <a name="new-major-and-minor-versions"></a>新的主要版本和次要版本

添加新的主要版本或次要版本后，该版本与现有版本一并安装。 可将应用手动升级到新版本。 如果在配置文件（例如 `web.config` 和 `package.json`）中配置了运行时版本，则需要使用相同的方法升级。 如果使用应用服务设置配置了运行时版本，则可以在 [Azure 门户](https://portal.azure.com)中或者在 [Cloud Shell](../cloud-shell/overview.md) 中运行 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) 命令来更改此版本，如以下示例所示：

```azurecli-interactive
az webapp config set --net-framework-version v4.7 --resource-group <groupname> --name <appname>
az webapp config set --php-version 7.0 --resource-group <groupname> --name <appname>
az webapp config appsettings set --settings WEBSITE_NODE_DEFAULT_VERSION=8.9.3 --resource-group <groupname> --name <appname>
az webapp config set --python-version 3.4 --resource-group <groupname> --name <appname>
az webapp config set --java-version 1.8 --java-container Tomcat --java-container-version 9.0 --resource-group <groupname> --name <appname>
```

### <a name="deprecated-versions"></a>已弃用的版本  

弃用某个旧版本后，将会公布删除日期，以便你可以相应地规划运行时版本升级。 

## <a name="how-can-i-query-os-and-runtime-update-status-on-my-instances"></a>如何在实例中查询 OS 和运行时更新状态？  

尽管关键的 OS 信息已被限制访问（请参阅 [Azure 应用服务中的操作系统功能](operating-system-functionality.md)），但可以使用 [Kudu 控制台](https://github.com/projectkudu/kudu/wiki/Kudu-console)在应用服务实例中查询有关 OS 版本和运行时版本的信息。 

下表显示了应用中运行的 Windows 和语言运行时版本：

| 信息 | 查找位置 | 
|-|-|
| Windows 版本 | 查看 `https://<appname>.scm.azurewebsites.net/Env.cshtml`（在“系统信息”下） |
| .NET 版本 | 在 `https://<appname>.scm.azurewebsites.net/DebugConsole` 中的命令提示符下运行以下命令： <br>`powershell -command "gci 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Net Framework Setup\NDP\CDF'"` |
| .NET Core 版本 | 在 `https://<appname>.scm.azurewebsites.net/DebugConsole` 中的命令提示符下运行以下命令： <br> `dotnet --version` |
| PHP 版本 | 在 `https://<appname>.scm.azurewebsites.net/DebugConsole` 中的命令提示符下运行以下命令： <br> `php --version` |
| 默认的 Node.js 版本 | 在 [Cloud Shell](../cloud-shell/overview.md) 中运行以下命令： <br> `az webapp config appsettings list --resource-group <groupname> --name <appname> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION']"` |
| Python 版本 | 在 `https://<appname>.scm.azurewebsites.net/DebugConsole` 中的命令提示符下运行以下命令： <br> `python --version` |  

> [!NOTE]  
> 访问注册表位置 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages`，其中存储了有关[“KB”修补](https://docs.microsoft.com/security-updates/SecurityBulletins/securitybulletins)的信息。该位置已被锁定。
>
>

## <a name="more-resources"></a>更多资源

[信任中心：安全性](https://www.microsoft.com/en-us/trustcenter/security)  
[Azure 应用服务中的 64 位 ASP.NET Core](https://gist.github.com/glennc/e705cd85c9680d6a8f1bdb62099c7ac7)
