---
title: Node.js 入门指南
description: 了解如何创建简单的 Node.js Web 应用程序并将其部署到 Azure 云服务。
services: cloud-services
documentationcenter: nodejs
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 50951a87-fed4-48e0-bcfa-453b9e50452e
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/17/2017
ms.author: jeconnoc
ms.openlocfilehash: e235af8ae35a6ff8e310bac802484e6c3d0f5397
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506941"
---
# <a name="build-and-deploy-a-nodejs-application-to-an-azure-cloud-service"></a>生成 Node.js 应用程序并将其部署到 Azure 云服务

本教程演示如何创建在 Azure 云服务中运行的简单 Node.js 应用程序。 云服务是 Azure 中可缩放的云应用程序构建基块。 它们允许进行单独且独立的管理，并允许横向扩展应用程序的前端和后端组件。  云服务为可靠托管每个角色提供强大的专用虚拟机。

有关云服务以及如何将它们与 Azure 网站和虚拟机进行比较的更多信息，请参阅 [Azure 网站、云服务和虚拟机的比较]。

> [!TIP]
> 想要构建简单网站？ 如果方案只涉及一个简单的网站前端，则可以考虑[使用轻型 Web 应用]。 随着 Web 应用的不断扩大和需求的变化，可以轻松升级到云服务。

按照本教程，生成托管在 Web 角色中的简单 Web 应用程序。 将使用计算模拟器在本地测试你的应用程序，并使用 PowerShell 命令行工具来部署该应用程序。

该应用程序是一个简单的“hello world”应用程序：

![Web 浏览器中显示“Hello World”网页][A web browser displaying the Hello World web page]

## <a name="prerequisites"></a>必备组件
> [!NOTE]
> 本教程使用 Azure PowerShell，因此需要在 Windows 上运行。

* 安装和配置 [Azure PowerShell]。
* 下载并安装 [用于 .NET 2.7 的 Azure SDK]。 在安装设置中，选择：
  * MicrosoftAzureAuthoringTools
  * MicrosoftAzureComputeEmulator

## <a name="create-an-azure-cloud-service-project"></a>创建 Azure 云服务项目
执行以下任务可创建一个新的 Azure 云服务项目以及基本的 Node.js 基架：

1. 以管理员身份运行 **Windows PowerShell**；在“开始”菜单或“开始”屏幕中，搜索 **Windows PowerShell**。
2. [连接 PowerShell] 到订阅。
3. 输入以下 PowerShell cmdlet 来创建项目：

        New-AzureServiceProject helloworld

    ![New-AzureService helloworld 命令的结果][The result of the New-AzureService helloworld command]

    **New-AzureServiceProject** cmdlet 会生成一个基本结构用于将 Node.js 应用程序发布到云服务。 该结构包含向 Azure 发布应用程序所需的配置文件。 该 cmdlet 还会将工作目录更改为服务的目录。

    该 cmdlet 将创建以下文件：

   * **ServiceConfiguration.Cloud.cscfg**、**ServiceConfiguration.Local.cscfg** 和 **ServiceDefinition.csdef**：发布应用程序时必需的 Azure 特定文件。 有关详细信息，请参阅 [创建 Azure 托管服务概述]。
   * **deploymentSettings.json**：存储供 Azure PowerShell 部署 cmdlet 使用的本地设置。
4. 输入以下命令添加新的 Web 角色：

       Add-AzureNodeWebRole

   ![Add-AzureNodeWebRole 命令的输出][The output of the Add-AzureNodeWebRole command]

   **Add-azurenodewebrole** cmdlet 将创建一个基本 Node.js 应用程序。 它还会修改 **.csfg** 和 **.csdef** 文件，以添加新角色的配置条目。

   > [!NOTE]
   > 如果不指定角色名称，将使用默认名称。 可以提供一个名称作为第一个 cmdlet 参数： `Add-AzureNodeWebRole MyRole`

Node.js 应用在 **server.js** 文件中定义，该文件位于 Web 角色（默认为 **WebRole1**）的目录中。 代码如下：

    var http = require('http');
    var port = process.env.port || 1337;
    http.createServer(function (req, res) {
        res.writeHead(200, { 'Content-Type': 'text/plain' });
        res.end('Hello World\n');
    }).listen(port);

此代码基本上与 [nodejs.org] 网站上的“Hello World”示例相同，但它使用云环境分配的端口号。

## <a name="deploy-the-application-to-azure"></a>将应用程序部署到 Azure

> [!NOTE]
> 若要完成本教程，需要一个 Azure 帐户。 可以[激活 MSDN 订阅者权益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF)或[注册免费帐户](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF)。

### <a name="download-the-azure-publishing-settings"></a>下载 Azure 发布设置
要将应用程序部署到 Azure，必须先为 Azure 订阅下载发布设置。

1. 运行以下 Azure Powershell cmdlet：

       Get-AzurePublishSettingsFile

   此操作将使用浏览器导航到发布设置下载页。 系统可能会提示使用 Microsoft 帐户登录。 如果出现提示，请使用与 Azure 订阅关联的帐户。

   将已下载的配置文件保存到能够轻松访问的文件位置。
2. 运行以下 cmdlet 以导入下载的发布配置文件：

       Import-AzurePublishSettingsFile [path to file]

    > [!NOTE]
    > 导入发布设置之后，请考虑删除下载的 .publishSettings 文件，因为它包含了可供他人访问你帐户的信息。

### <a name="publish-the-application"></a>发布应用程序
若要发布，请运行以下命令：

      $ServiceName = "NodeHelloWorld" + $(Get-Date -Format ('ddhhmm'))
    Publish-AzureServiceProject -ServiceName $ServiceName  -Location "East US" -Launch

* **-ServiceName** 指定部署的名称。 此名称必须唯一，否则发布过程会失败。 **Get-Date** 命令附加应使名称唯一的日期/时间字符串。
* **-Location** 指定托管应用程序的数据中心。 若要查看可用数据中心的列表，请使用 **Get-AzureLocation** cmdlet。
* **-Launch** 用于在部署完成后打开浏览器窗口并导航到托管服务。

发布成功之后，可看到如下响应：

![Publish-AzureService 命令的输出][The output of the Publish-AzureService command]

> [!NOTE]
> 部署应用程序并在首次发布该程序后使其可供使用可能需要花费几分钟时间。

在部署完成后，系统会打开一个浏览器窗口并导航到云服务。

![显示“hello world”页面的浏览器窗口；URL 指示该页面托管在 Azure 上。][A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]

应用程序现在在 Azure 上运行。

**Publish-AzureServiceProject** cmdlet 执行以下步骤：

1. 创建要部署的包。 该包将包含应用程序文件夹中的所有文件。
2. 如果存储帐户不存在，则创建一个新的**存储帐户** 。 Azure 存储帐户用于存储部署期间的应用程序包。 部署完成后，可以安全删除该存储帐户。
3. 如果云服务尚不存在，会创建一个新的**云服务** 。 **云服务** 是一个容器，用于在将应用程序部署到 Azure 后托管该应用程序。 有关详细信息，请参阅 [创建 Azure 托管服务概述]。
4. 将部署包发布到 Azure。

## <a name="stopping-and-deleting-your-application"></a>停止并删除应用程序
部署应用程序后，可能需要禁用它以免产生额外费用。 Azure 将按使用的服务器小时数对 Web 角色实例计费。 应用程序部署后即开始使用服务器时间，即使相关实例未运行并处于停止状态。

1. 在 Windows PowerShell 窗口中，使用以下 cmdlet 以停止上一节中创建的服务部署：

       Stop-AzureService

   停止服务可能需要花费几分钟时间。 在服务停止时，会收到一条指示服务已停止的消息。

   ![Stop-AzureService 命令的状态][The status of the Stop-AzureService command]
2. 若要删除服务，请调用以下 cmdlet：

       Remove-AzureService

   在出现提示时，输入 **Y** 以删除服务。

   删除服务可能需要花费几分钟时间。 删除服务后，将收到一条指示服务已被删除的消息。

   ![Remove-AzureService 命令的状态][The status of the Remove-AzureService command]

   > [!NOTE]
   > 删除服务不会删除最初发布服务时创建的存储帐户，用户仍需为使用的存储付费。 如果没有任何其他对象使用存储，请删除它。

## <a name="next-steps"></a>后续步骤
有关详细信息，请参阅 [Node.js 开发人员中心]。

<!-- URL List -->

[Azure 网站、云服务和虚拟机的比较]: /azure/architecture/guide/technology-choices/compute-decision-tree
[使用轻型 Web 应用]: ../app-service/app-service-web-get-started-nodejs.md
[Azure PowerShell]: /powershell/azureps-cmdlets-docs
[用于 .NET 2.7 的 Azure SDK]: https://www.microsoft.com/en-us/download/details.aspx?id=48178
[连接 PowerShell]: /powershell/azureps-cmdlets-docs
[nodejs.org]: https://nodejs.org/
[创建 Azure 托管服务概述]: https://azure.microsoft.com/documentation/services/cloud-services/
[Node.js 开发人员中心]: https://azure.microsoft.com/develop/nodejs/

<!-- IMG List -->

[The result of the New-AzureService helloworld command]: ./media/cloud-services-nodejs-develop-deploy-app/node9.png
[The output of the Add-AzureNodeWebRole command]: ./media/cloud-services-nodejs-develop-deploy-app/node11.png
[A web browser displaying the Hello World web page]: ./media/cloud-services-nodejs-develop-deploy-app/node14.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node19.png
[A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]: ./media/cloud-services-nodejs-develop-deploy-app/node21.png
[The status of the Stop-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node48.png
[The status of the Remove-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node49.png
