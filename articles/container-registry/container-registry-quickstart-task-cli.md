---
title: 快速入门 - 在 Azure 容器注册表中生成和运行容器映像
description: 使用 Azure 容器注册表快速运行任务，以便在云中按需生成和运行容器映像。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: quickstart
ms.date: 04/02/2019
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: be120ea8ae588da486c9a5acd4eb7bfdb4e45dee
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64701572"
---
# <a name="quickstart-build-and-run-a-container-image-using-azure-container-registry-tasks"></a>快速入门：使用 Azure 容器注册表任务生成和运行容器映像

本快速入门介绍如何使用 Azure 容器注册表任务命令在 Azure 本地快速生成、推送和运行 Docker 容器映像，并借此演示如何将“内部循环”开发周期工作卸载到云中。 [ACR 任务][container-registry-tasks-overview]是 Azure 容器注册表中的一套功能，可在整个容器生命周期内帮助管理和修改容器映像。 

完成本快速入门后，请探索 ACR 任务的更高级功能。 ACR 任务可以基于代码提交或基础映像更新自动生成映像，或者同时测试多个容器，此外还支持其他一些方案。 

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户][azure-account]。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

可以使用 Azure Cloud Shell 或 Azure CLI 的本地安装完成本快速入门。 如果想要在本地使用 Azure CLI，建议使用 2.0.58 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="create-a-resource-group"></a>创建资源组

如果没有容器注册表，请先使用 [az group create][az-group-create] 命令创建一个资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

以下示例在“eastus”位置创建名为“myResourceGroup”的资源组。

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>创建容器注册表

使用 [az acr create][az-acr-create] 命令创建容器注册表。 注册表名称在 Azure 中必须唯一，并且包含 5-50 个字母数字字符。 以下示例使用 *myContainerRegistry008*。 将其更新为唯一值。

```azurecli-interactive
az acr create --resource-group myResourceGroup --name myContainerRegistry008 --sku Basic
```

此示例创建一个基本注册表，该注册表已针对成本进行优化，可让开发人员了解 Azure 容器注册表。 有关可用服务层级的详细信息，请参阅[容器注册表 SKU][container-registry-skus]。

## <a name="build-an-image-from-a-dockerfile"></a>基于 Dockerfile 生成映像

现在，请使用 Azure 容器注册表来生成映像。 首先创建一个工作目录，然后创建包含以下内容的名为 *Dockerfile* 的 Dockerfile。 这是一个用于演示如何生成 Linux 容器映像的简单示例，你可以创建自己的标准 Dockerfile，并生成适用于其他平台的映像。

```bash
echo FROM hello-world > Dockerfile
```

运行 [az acr build][az-acr-build] 命令生成映像。 成功生成后，映像将推送到注册表。 以下示例推送 `sample/hello-world:v1` 映像。 命令末尾处的 `.` 设置 Dockerfile 的位置（在本例中为当前目录）。

```azurecli-interactive
az acr build --image sample/hello-world:v1 --registry myContainerRegistry008 --file Dockerfile . 
```

成功生成并推送后，输出将如下所示：

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/build_archive_b0bc1e5d361b44f0833xxxx41b78c24e.tar.gz'...
Sending context (1.856 KiB) to registry: mycontainerregistry008...
Queued a build with ID: ca8
Waiting for agent...
2019/03/18 21:56:57 Using acb_vol_4c7ffa31-c862-4be3-xxxx-ab8e615c55c4 as the home volume
2019/03/18 21:56:57 Setting up Docker configuration...
2019/03/18 21:56:58 Successfully set up Docker configuration
2019/03/18 21:56:58 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Executing step ID: build. Working directory: '', Network: ''
2019/03/18 21:56:59 Obtaining source code and scanning for dependencies...
2019/03/18 21:57:00 Successfully obtained source code and scanned for dependencies
2019/03/18 21:57:00 Launching container with name: build
Sending build context to Docker daemon  13.82kB
Step 1/1 : FROM hello-world
latest: Pulling from library/hello-world
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586fxxxx21577a99efb77324b0fe535
Successfully built fce289e99eb9
Successfully tagged mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:01 Successfully executed container: build
2019/03/18 21:57:01 Executing step ID: push. Working directory: '', Network: ''
2019/03/18 21:57:01 Pushing image: mycontainerregistry008.azurecr.io/sample/hello-world:v1, attempt 1
The push refers to repository [mycontainerregistry008.azurecr.io/sample/hello-world]
af0b15c8625b: Preparing
af0b15c8625b: Layer already exists
v1: digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a size: 524
2019/03/18 21:57:03 Successfully pushed image: mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:03 Step ID: build marked as successful (elapsed time in seconds: 2.543040)
2019/03/18 21:57:03 Populating digests for step ID: build...
2019/03/18 21:57:05 Successfully populated digests for step ID: build
2019/03/18 21:57:05 Step ID: push marked as successful (elapsed time in seconds: 1.473581)
2019/03/18 21:57:05 The following dependencies were found:
2019/03/18 21:57:05
- image:
    registry: mycontainerregistry008.azurecr.io
    repository: sample/hello-world
    tag: v1
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: v1
    digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
  git: {}

Run ID: ca8 was successful after 10s
```

## <a name="run-the-image"></a>运行映像

现在，请快速运行已生成并推送到注册表的映像。 在容器开发工作流中，可以先运行此步骤来验证映像，然后部署映像。

在本地工作目录中，通过一个步骤创建包含以下内容的 *quickrun.yaml* 文件。 请将 *\<acrLoginServer\>* 替换为注册表的登录服务器名称。 登录服务器名称采用 *\<registry-name\>.azurecr.io*（全小写）格式，例如 *mycontainerregistry008.azurecr.io*。 此示例假设在上一部分生成并推送了 `sample/hello-world:v1` 映像：

```yml
steps:
  - cmd: <acrLoginServer>/sample/hello-world:v1
```

此示例中的 `cmd` 步骤在默认配置中运行容器，但 `cmd` 支持其他 `docker run` 参数，甚至其他 `docker` 命令。

使用以下命令运行容器：

```azurecli-interactive
az acr run --registry myContainerRegistry008 --file quickrun.yaml .
```

输出与下面类似：

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
Sending context (1.855 KiB) to registry: mycontainerre...
Queued a run with ID: cab
Waiting for an agent...
2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
2019/03/19 19:01:53 Setting up Docker configuration...
2019/03/19 19:01:54 Successfully set up Docker configuration
2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2019/03/19 19:01:55 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

2019/03/19 19:01:56 Successfully executed container: acb_step_0
2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)

Run ID: cab was successful after 6s
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要存储在该处的资源组、容器注册表和容器映像，可以使用 [az group delete][az-group-delete] 命令将其删除。

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，我们使用 ACR 任务功能在 Azure 本地快速生成、推送并运行了一个 Docker 容器映像。 请继续学习 Azure 容器注册表教程，了解如何使用 ACR 任务来自动生成和更新映像。

> [!div class="nextstepaction"]
> [Azure 容器注册表教程][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[azure-account]: https://azure.microsoft.com/free/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tasks-overview]: container-registry-tasks-overview.md
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
