---
title: 从本地 Git 存储库进行部署
description: 了解如何实现从本地 Git 部署到 Azure 应用服务。 从本地计算机部署代码的最简单方法之一。
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.topic: article
ms.date: 02/16/2021
ms.reviewer: dariac
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 5dd6183bf88c167adb2f084c319cd90b94351dfb
ms.sourcegitcommit: de98cb7b98eaab1b92aa6a378436d9d513494404
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/17/2021
ms.locfileid: "100560453"
---
# <a name="local-git-deployment-to-azure-app-service"></a>从本地 Git 部署到 Azure 应用服务

本操作方法指南介绍如何将应用从本地计算机上的 Git 存储库部署到 [Azure 应用服务](overview.md)。

## <a name="prerequisites"></a>先决条件

按照本操作方法指南中的步骤操作：

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- [安装 Git](https://www.git-scm.com/downloads)。

- 创建包含想要部署的代码的本地 Git 存储库。 若要下载示例存储库，请在本地终端窗口运行以下命令：
  
  ```bash
  git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
  ```

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="configure-a-deployment-user"></a>配置部署用户

请参阅 [配置 Azure App Service 的部署凭据](deploy-configure-credentials.md)。 您可以使用用户范围凭据或应用程序范围凭据。

## <a name="create-a-git-enabled-app"></a>创建启用 Git 的应用

如果你已有一个应用服务应用并想要为其配置本地 Git 部署，请改为 [配置现有应用](#configure-an-existing-app) 。

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

[`az webapp create`](/cli/azure/webapp#az_webapp_create)用选项运行 `--deployment-local-git` 。 例如：

```azurecli-interactive
az webapp create --resource-group <group-name> --plan <plan-name> --name <app-name> --runtime "<runtime-flag>" --deployment-local-git
```

输出包含如下所示的 URL： `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git` 。 在下一步骤中，将使用此 URL 部署应用。

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

从 Git 存储库的根目录中运行 [AzWebApp](/powershell/module/az.websites/new-azwebapp) 。 例如：

```azurepowershell-interactive
New-AzWebApp -Name <app-name>
```

当你从 Git 存储库的目录运行此 cmdlet 时，它会自动为你创建一个名为的应用服务应用的 Git 远程 `azure` 。

# <a name="azure-portal"></a>[Azure 门户](#tab/portal)

在门户中，需要首先创建应用，然后为其配置部署。 请参阅 [配置现有应用](#configure-an-existing-app)。

-----

## <a name="configure-an-existing-app"></a>配置现有应用

如果尚未创建应用，请参阅 [创建启用 Git 的应用](#create-a-git-enabled-app) 。

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

运行 `az webapp deployment source config-local-git`。 例如：

```azurecli-interactive
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```

输出包含如下所示的 URL： `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git` 。 在下一步骤中，将使用此 URL 部署应用。

> [!TIP]
> 此 URL 包含用户范围部署用户名。 如果需要，可以改 [为使用应用程序范围的凭据](deploy-configure-credentials.md#appscope) 。 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

`scmType`通过运行[AzResource](/powershell/module/az.resources/set-azresource) cmdlet 设置应用的。

```powershell-interactive
$PropertiesObject = @{
    scmType = "LocalGit";
}

Set-AzResource -PropertyObject $PropertiesObject -ResourceGroupName <group-name> `
-ResourceType Microsoft.Web/sites/config -ResourceName <app-name>/web `
-ApiVersion 2015-08-01 -Force
```

# <a name="azure-portal"></a>[Azure 门户](#tab/portal)

1. 在 [Azure 门户](https://portal.azure.com)中，导航到应用的管理页。

1. 在左侧菜单中，选择 "**部署中心**  >  **设置**"。 在 "**源**" 中选择 "**本地 Git** "，然后单击 "**保存**"。

    ![演示如何为 Azure 门户中的应用服务启用本地 Git 部署](./media/deploy-local-git/enable-portal.png)

1. 在 "本地 Git" 部分中，复制 **Git Clone Uri** 以供以后查看。 此 Uri 不包含任何凭据。

-----

## <a name="deploy-the-web-app"></a>部署 Web 应用

1. 在本地终端窗口中，将目录更改为你的 Git 存储库的根目录，并使用从应用获取的 URL 添加 Git 远程。 如果你选择的方法没有提供 URL，请 `https://<app-name>.scm.azurewebsites.net/<app-name>.git` 在中将与你的应用程序名称一起使用 `<app-name>` 。
   
   ```bash
   git remote add azure <url>
   ```

    > [!NOTE]
    > 如果在 [PowerShell 中使用 AzWebApp 创建了启用 Git 的应用](#create-a-git-enabled-app)，则已为你创建了远程。
   
1. 使用 `git push azure master` 推送到 Azure 远程实例。 
   
1. 在 " **Git 凭据管理器** " 窗口中，输入 [用户范围凭据或应用程序范围的凭据](#configure-a-deployment-user)，而不是 Azure 登录凭据。

    如果你的 Git 远程 URL 已经包含用户名和密码，则将不会出现提示。 
   
1. 查看输出。 你可能会看到特定于运行时的自动化，例如 MSBuild for ASP.NET、`npm install` for Node.js 和 `pip install` for Python。 
   
1. 在 Azure 门户中浏览到你的应用以检查内容是否已部署。

## <a name="troubleshoot-deployment"></a>排查部署问题

使用 Git 发布到 Azure 中的应用服务应用时，你可能会看到以下常见错误消息：

|Message|原因|解决方法
---|---|---|
|`Unable to access '[siteURL]': Failed to connect to [scmAddress]`|应用未正常运行。|在 Azure 门户中启动应用。 如果 Web 应用已停止，Git 部署将不可用。|
|`Couldn't resolve host 'hostname'`|“azure”远程实例的地址信息不正确。|使用 `git remote -v` 命令列出所有远程网站以及关联的 URL。 确认“azure”远程网站的 URL 正确。 如果需要，请删除此远程网站并使用正确的 URL 重新创建它。|
|`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'main'.`|在运行 `git push` 期间未指定分支，或者未在 `.gitconfig` 中设置 `push.default` 值。|再次运行 `git push`，并指定主分支：`git push azure main`。|
|`src refspec [branchname] does not match any.`|你已尝试推送到“azure”远程实例上除主节点以外的分支。|再次运行 `git push`，并指定主分支：`git push azure main`。|
|`RPC failed; result=22, HTTP code = 5xx.`|如果尝试通过 HTTPS 推送大型 Git 存储库，则可能出现此错误。|在本地计算机上更改 Git 配置，以增大 `postBuffer`。 例如：`git config --global http.postBuffer 524288000`。|
|`Error - Changes committed to remote repository but your web app not updated.`|你已使用一个指定了其他所需模块的 _package.json_ 文件部署了 Node.js 应用。|检查发生此错误之前出现的 `npm ERR!` 错误消息，以了解有关失败的更多上下文。 下面是此错误的已知原因，以及相应的 `npm ERR!` 消息：<br /><br />**package.json 文件格式不当**：`npm ERR! Couldn't read dependencies.`<br /><br />**本机模块没有适用于 Windows 的二进制分发版**：<br />`npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1` <br />或 <br />`npm ERR! [modulename@version] preinstall: \make || gmake\ `|

## <a name="additional-resources"></a>其他资源

- [应用服务生成服务器 (Project Kudu 文档) ](https://github.com/projectkudu/kudu/wiki)
- [持续部署到 Azure 应用服务](deploy-continuous-deployment.md)
- [示例：从本地 Git 存储库创建 Web 应用和部署代码 (Azure CLI)](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
- [示例：从本地 Git 存储库创建 Web 应用和部署代码 (PowerShell)](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
