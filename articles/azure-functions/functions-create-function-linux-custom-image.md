---
title: 在 Linux 上使用自定义映像创建函数（预览）| Microsoft Docs
description: 了解如何创建在自定义 Linux 映像中运行的 Azure Functions。
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.date: 11/15/2017
ms.topic: tutorial
ms.service: functions
ms.custom: mvc
ms.devlang: azure-cli
manager: cfowler
ms.openlocfilehash: 758906126b42c103853e0047bb19d2e96a84fae6
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2018
---
# <a name="create-a-function-on-linux-using-a-custom-image-preview"></a>在 Linux 上使用自定义映像创建函数（预览）

使用 Azure Functions 可将函数托管在 Linux 上的自己的自定义容器中。 还可以[托管在默认的 Azure 应用服务容器中](functions-create-first-azure-function-azure-cli-linux.md)。 此功能目前为预览版，并且需要 [Functions 2.0 运行时](functions-versions.md)，后者也为预览版。

本教程介绍如何将函数应用部署为自定义 Docker 映像。 需要自定义内置的应用服务容器映像时，此模式很有用。 当函数需要特定的语言版本，或者需要内置映像中未预配的特定依赖项或配置时，可以使用自定义映像。

本教程逐步讲解如何使用 Azure Functions 创建自定义映像并将其推送到 Docker 中心。 然后，可将此映像用作 Linux 上运行的函数应用的部署源。 使用 Docker 生成和推送映像。 使用 Azure CLI 创建函数应用并从 Docker 中心部署映像。 

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 使用 Docker 生成自定义映像。
> * 将自定义映像发布到映象仓库。 
> * 创建 Azure 存储帐户。 
> * 创建 Linux 应用服务计划。 
> * 从 Docker 中心部署函数应用。
> * 将应用程序设置添加到函数应用。 

支持在 Mac、Windows 或 Linux 计算机上执行以下步骤。  

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

* [Git](https://git-scm.com/downloads)
* 一个有效的 [Azure 订阅](https://azure.microsoft.com/pricing/free-trial/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
* [Docker](https://docs.docker.com/get-started/#setup)
* 一个 [Docker 中心帐户](https://docs.docker.com/docker-id/)

[!INCLUDE [Free trial note](../../includes/quickstarts-free-trial-note.md)]

## <a name="download-the-sample"></a>下载示例

在终端窗口中，运行以下命令，将示例应用存储库克隆到本地计算机，然后切换至包含示例代码的目录。

```bash
git clone https://github.com/Azure-Samples/functions-linux-custom-image.git --config core.autocrlf=input
cd functions-linux-custom-image
```

## <a name="build-the-image-from-the-docker-file"></a>从 Docker 文件生成映像

在此 Git 存储库中查看 _Dockerfile_。 此文件描述在 Linux 上运行函数应用所需的环境。 

```docker
# Base the image on the built-in Azure Functions Linux image.
FROM microsoft/azure-functions-runtime:2.0.0-jessie
ENV AzureWebJobsScriptRoot=/home/site/wwwroot

# Add files from this repo to the root site folder.
COPY . /home/site/wwwroot 
```
>[!NOTE]
> 在专用映象仓库中托管映像时，应使用 Dockerfile 中的 **ENV** 变量将连接设置添加到函数应用。 由于本教程无法保证使用的是私有仓库，因此连接设置是[在部署后使用 Azure CLI 添加的](#configure-the-function-app)，这是一种最佳安全做法。   

### <a name="run-the-build-command"></a>运行 Build 命令
要生成 Docker 映像，请运行 `docker build` 命令，并提供名称、`mydockerimage` 和标记 `v1.0.0`。 将 `<docker-id>` 替换为 Docker 中心帐户 ID。

```bash
docker build --tag <docker-id>/mydockerimage:v1.0.0 .
```

该命令生成的输出如下所示：

```bash
Sending build context to Docker daemon  169.5kB
Step 1/3 : FROM microsoft/azure-functions-runtime:v2.0.0-jessie
v2.0.0-jessie: Pulling from microsoft/azure-functions-runtime
b178b12f7913: Pull complete
2d9ce077a781: Pull complete
4775d4ba55c8: Pull complete
Digest: sha256:073f45fc167b3b5c6642ef4b3c99064430d6b17507095...
Status: Downloaded newer image for microsoft/azure-functions-runtime:v2.0.0-jessie
 ---> 217799efa500
Step 2/3 : ENV AzureWebJobsScriptRoot /home/site/wwwroot
 ---> Running in 528fa2077d17
 ---> 7cc6323b8ae0
Removing intermediate container 528fa2077d17
Step 3/3 : COPY . /home/site/wwwroot
 ---> 5bdac9878423
Successfully built 5bdac9878423
Successfully tagged ggailey777/mydockerimage:v1.0.0
```

### <a name="test-the-image-locally"></a>在本地测试映像
通过在本地容器中运行生成的 Docker 映像，验证该映像是否正常工作。 发布 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令并将映像的名称和标记传递给它。 确保使用 `-p` 参数指定端口。

```bash
docker run -p 8080:80 -it <docker-ID>/mydockerimage:v1.0.0
```

使用本地 Docker 容器中运行的自定义映像，通过浏览到 <http://localhost:8080> 验证函数应用和容器是否正常运行。

![在本地测试函数应用。](./media/functions-create-function-linux-custom-image/run-image-local-success.png)

验证容器中的函数应用后，请停止执行。 现在，可将自定义映像推送到 Docker 中心帐户。

## <a name="push-the-custom-image-to-docker-hub"></a>将自定义映像推送到 Docker 中心

仓库是一种托管映像、提供服务映像和容器服务的应用程序。 要共享映像，必须先将其推送至仓库。 Docker 中心是一种用于 Docker 映像的仓库，使用该仓库，用户能够托管自己的存储库，无论是公共的还是专用的均可。 

推送映像之前，必须先使用 [docker login](https://docs.docker.com/engine/reference/commandline/login/) 命令登录到 Docker 中心。 请将 `<docker-id>` 替换为自己的帐户名，并在控制台中的提示符下键入密码。 有关其他 Docker 中心密码选项，请参阅 [docker login 命令的文档](https://docs.docker.com/engine/reference/commandline/login/)。

```bash
docker login --username <docker-id> 
```

如果看到“登录成功”消息，说明登录成功。 登录后，使用 [docker push](https://docs.docker.com/engine/reference/commandline/push/) 命令将映像推送到 Docker 中心。

```bash
docker push <docker-id>/mydockerimage:v1.0.0
```

可通过检查命令的输出来验证推送是否成功。

```bash
The push refers to a repository [docker.io/<docker-id>/mydockerimage:v1.0.0]
24d81eb139bf: Pushed
fd9e998161c9: Mounted from microsoft/azure-functions-runtime
e7796c35add2: Mounted from microsoft/azure-functions-runtime
ae9a05b85848: Mounted from microsoft/azure-functions-runtime
45c86e20670d: Mounted from microsoft/azure-functions-runtime
v1.0.0: digest: sha256:be080d80770df71234eb893fbe4d... size: 2422
```
现在，可在 Azure 中将此映像用作新函数应用的部署源。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本主题要求使用 Azure CLI 2.0.21 版或更高版本。 运行 `az --version` 即可确定你拥有的版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0]( /cli/azure/install-azure-cli)。 

[!INCLUDE [functions-create-resource-group](../../includes/functions-create-resource-group.md)]

[!INCLUDE [functions-create-storage-account](../../includes/functions-create-storage-account.md)]

## <a name="create-a-linux-app-service-plan"></a>创建 Linux 应用服务计划

消耗计划目前不支持在 Linux 上托管 Functions。 必须在 Linux 应用服务计划中运行 Functions。 若要了解有关托管的详细信息，请参阅 [Azure Functions 托管计划比较](functions-scale.md)。 

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]


## <a name="create-and-deploy-the-custom-image"></a>创建并部署自定义映像

函数应用托管函数的执行环境。 使用 [az functionapp create](/cli/azure/functionapp#az_functionapp_create) 命令基于 Docker 中心映像创建一个函数应用。 

在以下命令中，请将 `<app_name>` 占位符替换成唯一函数应用名称，将 `<storage_name>` 替换为存储帐户名。 `<app_name>` 将用作 Function App 的默认 DNS 域，因此，该名称需要在 Azure 中的所有应用之间保持唯一。 如前所述，`<docker-id>` 是你的 Docker 帐户名称。

```azurecli-interactive
az functionapp create --name <app_name> --storage-account  <storage_name>  --resource-group myResourceGroup \
--plan myAppServicePlan --deployment-container-image-name <docker-id>/mydockerimage:v1.0.0 
```
创建 Function App 后，Azure CLI 会显示类似于以下示例的信息：

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "containerSize": 1536,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "quickstart.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "quickstart.azurewebsites.net",
    "quickstart.scm.azurewebsites.net"
  ],
   ....
    // Remaining output has been truncated for readability.
}
```

_deployment-container-image-name_ 参数表示 Docker 中心托管的、用于创建函数应用的映像。 


## <a name="configure-the-function-app"></a>配置函数应用

该函数需要使用连接字符串连接到默认的存储帐户。 将自定义映像发布到专用容器帐户时，应该使用 [ENV 指令](https://docs.docker.com/engine/reference/builder/#env)或等效的命令将这些应用程序设置指定为 Dockerfile 中的环境变量。 

在本例中，`<storage_account>` 是所创建的存储帐户的名称。 使用 [az storage account show-connection-string](/cli/azure/storage/account#show-connection-string) 命令获得连接字符串。 使用 [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_set) 命令将这些应用程序设置添加到函数应用。

```azurecli-interactive
storageConnectionString=$(az storage account show-connection-string \
--resource-group myResourceGroup --name <storage_account> \
--query connectionString --output tsv)

az functionapp config appsettings set --name <function_app> \
--resource-group myResourceGroup \
--settings AzureWebJobsDashboard=$storageConnectionString \
AzureWebJobsStorage=$storageConnectionString
```

现在，可以测试 Azure 中的 Linux 上运行的函数。

[!INCLUDE [functions-test-function-code](../../includes/functions-test-function-code.md)]

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 使用 Docker 生成自定义映像。
> * 将自定义映像发布到映象仓库。 
> * 创建 Azure 存储帐户。 
> * 创建 Linux 应用服务计划。 
> * 从 Docker 中心部署函数应用。
> * 将应用程序设置添加到函数应用。

详细了解如何使用 Azure Functions Core Tools 在本地开发 Azure Functions。

> [!div class="nextstepaction"] 
> [在本地对 Azure Functions 进行编码和测试](functions-run-local.md)
