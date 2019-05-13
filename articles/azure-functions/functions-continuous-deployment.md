---
title: Azure Functions 的连续部署 | Microsoft Docs
description: 使用 Azure 应用服务的连续部署功能来发布你的函数。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.assetid: 361daf37-598c-4703-8d78-c77dbef91643
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/25/2016
ms.author: glenga
ms.openlocfilehash: cb3f3ad3bb7b42429654ea4bf9b49f7e230db1da
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64943878"
---
# <a name="continuous-deployment-for-azure-functions"></a>Azure Functions 的连续部署
Azure Functions 可以轻松地使用持续集成部署 function app。 Functions 可与重大的代码存储库和部署源集成。 此集成，通过这些服务触发器部署之一进行到 Azure 函数代码更新所在的工作流。 如果您是初次接触 Azure Functions，请先使用[Azure Functions 概述](functions-overview.md)。

连续部署会很适合项目的选择将集成多个和频繁的发布内容。 它还允许你维护在函数代码上的源代码管理。 Azure Functions 支持以下部署源：

* [Bitbucket](https://bitbucket.org/)
* [Dropbox](https://www.dropbox.com/)
* 外部存储库（Git 或 Mercurial）
* [Git 本地存储库](../app-service/deploy-local-git.md)
* [GitHub](https://github.com)
* [OneDrive](https://onedrive.live.com/)
* [Azure DevOps](https://azure.microsoft.com/services/devops/)

在每个 Function App 基础上配置部署。 启用连续部署后，将门户中的函数代码访问设置为*只读*。

## <a name="requirements-for-continuous-deployment"></a>连续部署的要求

设置连续部署之前，你必须配置的部署源且在函数代码中的部署源。 在函数应用部署中，每个函数位于已命名的子目录，其中的目录名称是函数的名称。  

[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

为了能够从 Azure DevOps 进行部署，必须先将 Azure DevOps 组织与 Azure 订阅关联起来。 有关详细信息，请参阅[为 Azure DevOps 组织设置计费](https://docs.microsoft.com/azure/devops/organizations/billing/set-up-billing-for-your-organization-vs?view=vsts#set-up-billing-via-the-azure-portal)。

## <a name="set-up-continuous-deployment"></a>设置连续部署
使用此过程来配置现有 Function App 的连续部署。 这些步骤演示了与 GitHub 存储库的集成，但类似的步骤适用于 Azure DevOps 或其他部署服务。

1. 在函数应用中[Azure 门户](https://portal.azure.com)，选择**平台功能** > **部署选项**。 
   
    ![选择用于打开部署选项](./media/functions-continuous-deployment/setup-deployment.png)
 
1. 上**部署**边栏选项卡，选择**安装程序**。
 
    ![部署边栏选项卡](./media/functions-continuous-deployment/setup-deployment-1.png)
   
1. 上**部署源**边栏选项卡，选择**选择源**。 填写选择的部署源的信息，然后选择**确定**。
   
    ![选择部署源](./media/functions-continuous-deployment/choose-deployment-source.png)

设置连续部署后，部署源中的所有文件更改都复制到函数应用，并会触发完整的站点部署。 当源中的文件更新时重新部署该网站。

## <a name="deployment-scenarios"></a>部署方案

典型的部署方案包括创建过渡部署和将现有函数移至连续部署。

<a name="staging"></a>
### <a name="create-a-staging-deployment"></a>创建分阶段部署

Function app 尚不支持部署槽。 但您仍可以通过使用持续集成管理单独的过渡和生产部署。

配置和使用分阶段部署的过程通常是这样的：

1. 在订阅中创建两个函数应用： 一个用于生产代码，一个用于过渡。 

1. 如果还没有部署源，创建一个。 本示例使用 [GitHub]。

1. 对于生产 Function App，在[“设置连续部署”](#set-up-continuous-deployment)中完成上述步骤，并将部署分支设置为 GitHub 存储库的主分支。
   
    ![选择以选择部署分支](./media/functions-continuous-deployment/choose-deployment-branch.png)

1. 对于临时的 function app，重复步骤 3，但在 GitHub 存储库中改为选择过渡分支。 如果部署的源不支持分支，请使用不同的文件夹。
    
1. 对临时分支或文件夹中的代码进行更新，然后验证的过渡部署反映这些更改。

1. 测试完成后，将更改从临时分支合并到主分支。 此合并会触发生产 Function App 的部署。 如果部署的源不支持分支，请使用临时文件夹中的文件覆盖生产文件夹中的文件。

<a name="existing"></a>
### <a name="move-existing-functions-to-continuous-deployment"></a>将现有函数移至连续部署
当你具有的现有函数，你已创建并维护在门户中，您需要通过使用 FTP 下载函数代码文件或本地 Git 存储库，您可以设置连续部署，如前面所述。 您可以执行此操作的 Azure 应用服务设置为 function app。 下载文件后，可以将它们上载到您选择的连续部署的源。

> [!NOTE]
> 配置持续集成后，不再可以编辑在 Functions 门户中的源文件。

<a name="credentials"></a>
#### <a name="configure-deployment-credentials"></a>配置部署凭据
您可以通过使用 FTP 或本地 Git 存储库从 function app 下载文件之前，必须配置凭据以访问站点。 在函数应用级别设置凭据。 执行以下步骤，在 Azure 门户中设置部署凭据：

1. 在函数应用中[Azure 门户](https://portal.azure.com)，选择**平台功能** > **部署凭据**。
   
1. 输入用户名和密码，并选择**保存**。 

   ![选择设置本地的部署凭据](./media/functions-continuous-deployment/setup-deployment-credentials.png)

现在可以使用这些凭据来访问 FTP 或内置 Git 存储库内的函数应用。

<a name="downftp"></a>
#### <a name="download-files-by-using-ftp"></a>使用 FTP 下载文件

1. 在函数应用中[Azure 门户](https://portal.azure.com)，选择**平台功能** > **属性**。 然后，将复制的值**FTP/部署用户**， **FTP 主机名**，并**FTPS 主机名**。  

   根据门户显示，必须输入“FTP/部署用户”（包括应用名称），以便为 FTP 服务器提供适当的上下文。
   
   ![获取部署信息的选项](./media/functions-continuous-deployment/get-deployment-credentials.png)

1. 从 FTP 客户端，使用连接到您的应用程序并下载你的函数的源代码文件收集到的连接信息。

<a name="downgit"></a>
#### <a name="download-files-by-using-a-local-git-repository"></a>使用本地 Git 存储库下载文件

1. 在函数应用中[Azure 门户](https://portal.azure.com)，选择**平台功能** > **部署选项**。 
   
    ![选择用于打开部署选项](./media/functions-continuous-deployment/setup-deployment.png)
 
1. 然后在**部署**边栏选项卡，选择**安装程序**。
 
    ![部署边栏选项卡](./media/functions-continuous-deployment/setup-deployment-1.png)
   
1. 上**部署源**边栏选项卡，选择**本地 Git 存储库** > **确定**。

1. 在中**平台功能**，选择**属性**并记下 Git URL 的值。 
   
    ![选择用于获取 Git URL](./media/functions-continuous-deployment/get-local-git-deployment-url.png)

1. 使用 Git 感知的命令提示符或你喜欢的 Git 工具克隆本地计算机上的存储库。 Git 克隆命令如下所示：
   
        git clone https://username@my-function-app.scm.azurewebsites.net:443/my-function-app.git

1. 将文件从函数应用提取到本地计算机上的克隆，如以下示例所示：
   
        git pull origin master
   
    如果已请求，请提供[已配置的部署凭据](#credentials)。  

[GitHub]: https://github.com/

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Functions 最佳实践](functions-best-practices.md)
