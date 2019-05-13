---
title: 安装和运行容器 - 个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: 如何为个性化体验创建服务下载、安装并运行容器。
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/09/2019
ms.author: edjez
ms.openlocfilehash: a197531a7c78823271c0a5fa5413b76746f63a9a
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507019"
---
# <a name="install-and-run-personalizer-containers"></a>安装并运行个性化体验创建服务容器

个性化体验创建服务有以下容器： 

|函数|功能|
|-|-|
|个性化体验创建服务|根据内容和用户的当前上下文确定最佳操作。|

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

使用个性化体验创建服务容器之前，必须满足以下先决条件：

|必选|目的|
|--|--|
|Docker 引擎| 需要在[主计算机](#the-host-computer)上安装 Docker 引擎。 Docker 提供用于在 [macOS](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 和 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 上配置 Docker 环境的包。 有关 Docker 和容器的基础知识，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。<br><br> 必须将 Docker 配置为允许容器连接 Azure 并向其发送账单数据。 <br><br> 在 Windows 上，还必须将 Docker 配置为支持 Linux 容器。<br><br>|
|熟悉 Docker | 应对 Docker 概念有基本的了解，例如注册表、存储库、容器和容器映像，以及基本的 `docker` 命令的知识。| 
|个性化体验创建服务资源 |若要使用这些容器，必须具有：<br><br>一项个性化体验创建服务 Azure 资源，用于获取关联的计费密钥和计费终结点 URI。 这两个值都可以从 Azure 门户中的个性化体验创建服务“概览”和“密钥”页获取；必须获取这两个值才能启动容器。<br><br>**{BILLING_KEY}**：资源密钥<br><br>**{BILLING_ENDPOINT_URI}**：终结点 URI 示例如下：`https://westus.api.cognitive.microsoft.com/`|

### <a name="the-host-computer"></a>主计算机

主机是运行 docker 容器的计算机。 它可以是本地计算机或 Azure 中的 docker 托管服务，包括：

* [Azure Kubernetes 服务](https://docs.microsoft.com/aks/index.yml)
* [Azure 容器实例](https://docs.microsoft.com/container-instances/index.yml)
* 部署到 [Azure Stack](https://docs.microsoft.com/azure-stack/index.yml) 的 [Kubernetes](https://kubernetes.io/) 群集。 有关详细信息，请参阅[将 Kubernetes 部署到 Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-solution-template-kubernetes-deploy.md)。

### <a name="container-requirements-and-recommendations"></a>容器要求和建议

下表描述了为每个个性化体验创建服务容器分配的最小的和建议的 CPU 核心数和内存。

| 容器 | 最小值 | 建议 |
|-----------|---------|-------------|
|个性化体验创建服务 | 单核，4 GB 内存 | 2 核心，8 GB 内存 |

每个核心必须至少为 2.6 千兆赫 (GHz) 或更快。

核心和内存对应于 `--cpus` 和 `--memory` 设置，用作 `docker run` 命令的一部分。

容器必须能够连接到 Azure EventHub 才能将排名和奖励调用中的信息中继到 Azure 中的个性化体验创建服务服务器，而且必须能够连接到个性化体验创建服务 API 才能加载所需的配置和最新的机器学习模型。

## <a name="get-the-container-image-with-docker-pull"></a>使用 `docker pull` 获取容器映像

可以使用个性化体验创建服务的容器映像。 

| 容器 | 存储库 |
|-----------|------------|
| 个性化体验创建服务 | `mcr.microsoft.com/azure-cognitive-services/personalizer:latest` |

> [!TIP]
> 可以使用 [docker images](https://docs.docker.com/engine/reference/commandline/images/) 命令列出下载的容器映像。 例如，以下命令以表格列出每个下载的容器映像的 ID、存储库和标记：
>
>  ```Docker
>  docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
>
>  IMAGE ID            REPOSITORY                                                                TAG
>  ebbee78a6baa        mcr.microsoft.com/azure-cognitive-services/personalizer                 latest
>  ``` 

### <a name="docker-pull-for-the-personalizer-service-container"></a>适用于个性化体验创建服务容器的 Docker 拉取

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/personalizer:latest
```

## <a name="how-the-container-works"></a>容器工作原理

个性化体验创建服务容器代表个性化体验创建服务的客户端部分。 容器在运行时，会从 Azure 云中的个性化体验创建服务获取模型和配置设置。 容器服务根据此信息来响应对**排名**和**奖励**的请求。 容器还将这些请求发送到 Azure 云中的个性化体验创建服务资源。 然后，可以使用此信息根据模型更新频率的当前设置来训练个性化体验创建服务循环模型。 更新的模型将发送回容器。 

![个性化体验创建服务容器的客户端的本地体系结构](./media/personalizer-container-howto/personalizer-container-architecture.png)

## <a name="how-to-use-the-container"></a>如何使用容器

一旦容器位于[主计算机](#the-host-computer)上，请通过以下过程使用容器。

1. 使用所需的计费设置[运行容器](#run-the-container-with-docker-run)。 提供 `docker run` 命令的多个[示例](personalizer-container-configuration.md#example-docker-run-commands)。 
1. [查询容器的预测终结点](#query-the-containers-prediction-endpoint)。 

## <a name="run-the-container-with-docker-run"></a>通过 `docker run` 运行容器

使用 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令运行三个容器中的任意一个。 该命令使用以下参数：

| 占位符 | 值 |
|-------------|-------|
|{BILLING_KEY} | 此密钥用于启动容器，可以从 Azure 门户的个性化体验创建服务“密钥”页获取。  |
|{BILLING_ENDPOINT_URI} | 可以从 Azure 门户的个性化体验创建服务“概览”页获取计费终结点 URI 值。|

在以下示例 `docker run` 命令中，请将这些参数替换为自己的值。

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/personalizer\
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}
```

此命令：

* 从容器映像运行个性化体验创建服务容器
* 分配一个 CPU 核心和 4 GB 内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。 

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](#billing)。

### <a name="run-multiple-containers-on-the-same-host"></a>在同一主机上运行多个容器

若要运行多个公开端口的容器，请确保在运行每个容器时使用不同的端口。 例如，在端口 5000 上运行第一个容器，在端口 5001 上运行第二个容器。

请将 `<container-registry>` 和 `<container-name>` 替换为所用容器的值。 这些容器不需要是同一容器。 可以在 HOST 上同时运行个性化体验创建服务容器和 LUIS 容器，也可以运行多个个性化体验创建服务容器。 

在端口 5000 上运行第一个容器。 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}
```

在端口 5001 上运行第二个容器。


```bash 
docker run --rm -it -p 5000:5001 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}
```

每个后续容器应位于不同的端口上。 

## <a name="query-the-containers-prediction-endpoint"></a>查询容器的预测终结点

容器提供了基于 REST 的查询预测终结点 API。 

使用主机 https://localhost:5000，以获得容器 API。

## <a name="stop-the-container"></a>停止容器

若要关闭容器，请在运行容器的命令行环境中按 Ctrl+C。

## <a name="troubleshoot"></a>故障排除

如果运行启用了输出[装入点](personalizer-container-configuration.md#mount-settings)和日志记录的容器，该容器会生成有助于排查启动或运行容器时发生的问题的日志文件。 

## <a name="container-api-documentation"></a>容器 API 文档

容器提供终结点以及 `Try it now` 功能的整套文档。 通过此功能可以将设置输入到基于 Web 的 HTML 窗体中并进行查询，而无需编写任何代码。 返回查询后，将提供示例 CURL 命令，用于演示所需的 HTTP 标头和正文格式。 

> [!TIP]
> 阅读 [OpenAPI 规范](https://swagger.io/docs/specification/about/)，该规范描述了 `/swagger` 相对 URI 中容器支持的 API 操作。 例如：
>
>  ```http
>  http://localhost:5000/swagger
>  ```

## <a name="billing"></a>计费

个性化体验创建服务容器使用 Azure 帐户中的“个性化体验创建服务”资源向 Azure 发送账单信息。 

如果未连接到 Azure 进行计量，则无法授权并运行认知服务容器。 客户需要始终让容器向计量服务传送账单信息。  

`docker run` 命令使用以下参数进行计费：

| 选项 | 说明 |
|--------|-------------|
| `ApiKey` | 用于跟踪账单信息的“个性化体验创建服务”资源的 API 密钥。 |
| `Billing` | 用于跟踪账单信息的“个性化体验创建服务”资源的终结点。|
| `Eula` | 表示已接受容器的许可条款。<br/>此选项的值必须设置为 `accept`。 |

> [!IMPORTANT]
> 必须使用有效值指定所有三个选项，否则容器将无法启动。

有关这些选项的详细信息，请参阅[配置容器](personalizer-container-configuration.md)。

## <a name="summary"></a>摘要

在本文中，我们已学习相关的概念，以及个性化体验创建服务容器的下载、安装和运行工作流。 综上所述：

* 个性化体验创建服务为 Docker 提供了Linux 容器，用于封装个性化设置。
* 从 Azure 中的 Microsoft 容器注册表 (MCR) 下载容器映像。
* 容器映像在 Docker 中运行。
* 可以使用 REST API 或 SDK 通过指定容器的主机 URI 来调用个性化体验创建服务容器中的操作。
* 必须在实例化容器时指定账单信息。

> [!IMPORTANT]
> 如果未连接到 Azure 进行计量，则无法授权并运行认知服务容器。 客户需要始终让容器向计量服务传送账单信息。 个性化体验创建服务容器还会将请求数据发送到 Azure 中的相应服务，以便进行联机培训，并且会从 Azure 定期获取更新的模型。

## <a name="next-steps"></a>后续步骤

[如何针对 `Docker Run` 命令配置 Docker 容器](personalizer-container-configuration.md)