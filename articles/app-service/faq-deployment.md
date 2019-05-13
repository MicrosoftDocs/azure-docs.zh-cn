---
title: 部署常见问题解答 - Azure 应用服务 | Microsoft Docs
description: 获取有关 Azure 应用服务的 Web 应用功能的部署常见问题解答。
services: app-service\web
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 11/01/2018
ms.author: genli
ms.custom: seodec18
ms.openlocfilehash: 9d7fd41657394e006957392f37ec1a258b33c8c6
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413868"
---
# <a name="deployment-faqs-for-web-apps-in-azure"></a>Azure 中的 Web 应用的部署常见问题解答

本文包含有关针对 [Azure 应用服务的 Web 应用功能](https://azure.microsoft.com/services/app-service/web/)的部署问题的常见问题 (FAQ) 解答。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="i-am-just-getting-started-with-app-service-web-apps-how-do-i-publish-my-code"></a>我刚刚开始使用应用服务 Web 应用。 如何发布我的代码？

以下是一些用于发布 Web 应用代码的选项：

*   使用 Visual Studio 进行部署。 如果具有 Visual Studio 解决方案，则右键单击 Web 应用程序项目，然后选择“发布”。
*   使用 FTP 客户端进行部署。 在 Azure 门户中，下载要将代码部署到的 Web 应用的发布配置文件。 然后使用相同的发布配置文件 FTP 凭据将文件上传到 \site\wwwroot。

有关详细信息，请参阅[将应用部署到应用服务](deploy-local-git.md)。

## <a name="i-see-an-error-message-when-i-try-to-deploy-from-visual-studio-how-do-i-resolve-this"></a>我在尝试从 Visual Studio 进行部署时看到错误消息。 如何解决此问题？

如果看到以下消息，则可能使用的是旧版本的 SDK：“在资源组 'YourResourceGroup' 中部署资源 'YourResourceName' 时出错: MissingRegistrationForLocation: 订阅未在 '美国中部' 位置注册资源类型 'components'。 要访问此位置，请重新注册此提供程序。” 

若要解决此错误，请升级到[最新 SDK](https://azure.microsoft.com/downloads/)。 如果你看到此消息并且具有最新 SDK，请提交支持请求。

## <a name="how-do-i-deploy-an-aspnet-application-from-visual-studio-to-app-service"></a>如何从 Visual Studio 将 ASP.NET 应用程序部署到应用服务？
<a id="deployasp"></a>

本教程[5 分钟内创建在 Azure 中的第一个 ASP.NET web 应用](app-service-web-get-started-dotnet.md)演示如何使用 Visual Studio 部署到应用服务中的 web 应用的 ASP.NET web 应用程序。

## <a name="what-are-the-different-types-of-deployment-credentials"></a>有哪些不同类型的部署凭据？

应用服务支持两种类型的凭据，这些凭据适用于本地 GIT 部署和 FTP/S 部署。 有关如何配置部署凭据的详细信息，请参阅[为应用服务配置部署凭据](deploy-configure-credentials.md)。

## <a name="what-is-the-file-or-directory-structure-of-my-app-service-web-app"></a>应用服务 Web 应用的文件或目录结构是怎样的？

有关应用服务应用的文件结构的信息，请参阅 [Azure 中的文件结构](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)。

## <a name="how-do-i-resolve-ftp-error-550---there-is-not-enough-space-on-the-disk-when-i-try-to-ftp-my-files"></a>如何在尝试通过 FTP 传输文件时解决“FTP 错误 550 - 磁盘空间不足”？

如果看到此消息，则可能是遇到服务计划中针对 Web 应用的磁盘配额。 可能需要基于磁盘空间需求提升到较高服务层级。 有关定价计划和资源限制的详细信息，请参阅[应用服务定价](https://azure.microsoft.com/pricing/details/app-service/)。

## <a name="how-do-i-set-up-continuous-deployment-for-my-app-service-web-app"></a>如何为应用服务 Web 应用程序设置持续部署？

可以从多个资源（包括 Azure DevOps、OneDrive、GitHub、Bitbucket、Dropbox 和其他 Git 存储库）设置持续部署。 这些选项可在门户中使用。 [持续部署到应用服务](deploy-continuous-deployment.md)是很有帮助的教程，其中说明了如何设置持续部署。

## <a name="how-do-i-troubleshoot-issues-with-continuous-deployment-from-github-and-bitbucket"></a>如何对从 GitHub 和 Bitbucket 进行持续部署的问题进行故障排除？

若要帮助调查从 GitHub 或 Bitbucket 进行持续部署的问题，请参阅[调查持续部署](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)。

## <a name="i-cant-ftp-to-my-site-and-publish-my-code-how-do-i-resolve-this"></a>我无法通过 FTP 传输到我的站点并发布代码。 如何解决此问题？

解决 FTP 问题：

1. 检查是否输入了正确的主机名和凭据。 有关不同类型的凭据以及如何使用它们的详细信息，请参阅[部署凭据](https://github.com/projectkudu/kudu/wiki/Deployment-credentials)。
2. 验证防火墙是否阻止了 FTP 端口。 端口应具有以下这些设置：
    * FTP 控制连接端口：21
    * FTP 数据连接端口：989、10001-10300

## <a name="how-do-i-publish-my-code-to-app-service"></a>如何将我的代码发布到应用服务？

Azure 快速入门旨在帮助使用部署堆栈和所选方法部署应用。 若要使用快速入门，请在 Azure 门户中转到应用服务，在“部署”下，选择“快速入门”。

## <a name="why-does-my-app-sometimes-restart-after-deployment-to-app-service"></a>为何我的应用有时在部署到应用服务之后重新启动？

若要了解应用程序部署可能会导致重新启动的情况，请参阅[部署与运行时问题](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues#deployments-and-web-app-restarts")。 如文章中所述，应用服务会将文件部署到 wwwroot 文件夹。 它绝不会直接重新启动应用。

## <a name="how-do-i-integrate-azure-devops-code-with-app-service"></a>如何将 Azure DevOps 代码与应用服务集成？

使用 Azure DevOps 进行持续部署有两种方法：

*   使用 Git 项目。 使用部署中心连接通过应用服务。
*   使用 Team Foundation 版本控制 (TFVC) 项目。 通过对应用服务使用生成代理来进行部署。

这两个选项的持续代码部署都依赖于现有开发人员工作流和签入过程。 有关详细信息，请参阅以下文章： 

*   [实现应用到 Azure 网站的持续部署](https://www.visualstudio.com/docs/release/examples/azure/azure-web-apps-from-build-and-release-hubs)
*   [设置 Azure DevOps 组织，以便它可以部署到 Web 应用](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App)

## <a name="how-do-i-use-ftp-or-ftps-to-deploy-my-app-to-app-service"></a>如何使用 FTP 或 FTPS 将应用部署到应用服务？

有关使用 FTP 或 FTPS 将 Web 应用部署到应用服务的信息，请参阅[使用 FTP/S 将应用部署到应用服务](deploy-ftp.md)。
