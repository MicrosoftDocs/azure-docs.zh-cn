---
title: 安装和运行容器的异常情况检测程序
titleSuffix: Azure Cognitive Services
description: 使用异常检测器 API 的高级的算法来标识时序数据中的异常。
services: cognitive-services
author: aahill
ms.service: cognitive-services
ms.subservice: anomaly-detection
ms.topic: article
ms.date: 05/07/2019
ms.author: aahi
ms.openlocfilehash: cc82e091ae4c033bda7f1d91c9aed36bb081de88
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233609"
---
# <a name="install-and-run-anomaly-detector-containers"></a>安装和运行异常情况检测程序容器

异常情况检测程序具有以下容器： 

|函数|功能|
|-|-|
|异常情况检测程序| <li> 在实时发生，检测到异常情况。 <li> 检测整个数据集作为一个批。 <li> 推断数据的预期正常范围。 <li> 支持异常情况检测敏感度调整以更好地适合你的数据。 |

有关这些 Api 的详细信息，请参阅：
* [了解有关异常情况检测器 API 服务详细信息](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>必备组件

使用异常检测器容器之前，必须满足以下先决条件：

|需要|目的|
|--|--|
|Docker 引擎| 需要在[主计算机](#the-host-computer)上安装 Docker 引擎。 Docker 提供用于在 [macOS](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 和 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 上配置 Docker 环境的包。 有关 Docker 和容器的基础知识，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。<br><br> 必须将 Docker 配置为允许容器连接 Azure 并向其发送账单数据。 <br><br> 在 Windows 上，还必须将 Docker 配置为支持 Linux 容器。<br><br>|
|熟悉 Docker | 应对 Docker 概念有基本的了解，例如注册表、存储库、容器和容器映像，以及基本的 `docker` 命令的知识。| 
|异常情况检测程序资源 |若要使用这些容器，必须具有：<br><br>_异常情况检测器_Azure 资源以获取对关联的帐单密钥和计费终结点 URI。 这两个值可在 Azure 门户的异常情况检测程序概述和密钥页上，并且要求来启动该容器。<br><br>**{BILLING_KEY}**：资源密钥<br><br>**{BILLING_ENDPOINT_URI}**：终结点 URI 示例如下：`https://westus2.api.cognitive.microsoft.com`|

## <a name="request-access-to-the-container-registry"></a>请求访问容器注册表

必须先完成并提交[异常情况检测器容器请求窗体](https://aka.ms/adcontainer)容器请求访问。

[!INCLUDE [Request access to the container registry](../../../includes/cognitive-services-containers-request-access-only.md)]

[!INCLUDE [Authenticate to the container registry](../../../includes/cognitive-services-containers-access-registry.md)]

## <a name="the-host-computer"></a>主计算机

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

<!--* [Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/). For instructions of deploying Anomaly Detector module in IoT Edge, see [How to deploy Anomaly Detector module in IoT Edge](how-to-deploy-anomaly-detector-module-in-iot-edge.md).-->

### <a name="container-requirements-and-recommendations"></a>容器要求和建议

下表描述的最低和推荐 CPU 内核和内存来分配为异常情况检测程序容器。

| QPS （每秒查询） | 最小值 | 建议 |
|-----------|---------|-------------|
| 10 QPS | 4 核，1 GB 内存 | 8 核 2 GB 内存 |
| 20 QPS | 8 核，2 GB 内存 | 16 核 4 GB 内存 |

每个核心必须至少为 2.6 千兆赫 (GHz) 或更快。

核心和内存对应于 `--cpus` 和 `--memory` 设置，用作 `docker run` 命令的一部分。

## <a name="get-the-container-image-with-docker-pull"></a>使用 `docker pull` 获取容器映像

使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令下载容器映像。

| 容器 | 存储库 |
|-----------|------------|
| cognitive-services-anomaly-detector | `containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector:latest` |

<!--
For a full description of available tags, such as `latest` used in the preceding command, see [anomaly-detector](https://go.microsoft.com/fwlink/?linkid=2083827&clcid=0x409) on Docker Hub.
-->
[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]


### <a name="docker-pull-for-the-anomaly-detector-container"></a>异常情况检测程序容器的 docker 拉取

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector:latest
```

## <a name="how-to-use-the-container"></a>如何使用容器

一旦容器位于[主计算机](#the-host-computer)上，请通过以下过程使用容器。

1. 使用所需的计费设置[运行容器](#run-the-container-with-docker-run)。 提供 `docker run` 命令的多个[示例](anomaly-detector-container-configuration.md#example-docker-run-commands)。 
1. [查询容器的预测终结点](#query-the-containers-prediction-endpoint)。 

## <a name="run-the-container-with-docker-run"></a>通过 `docker run` 运行容器

使用 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令运行三个容器中的任意一个。 该命令使用以下参数：

| 占位符 | 值 |
|-------------|-------|
|{BILLING_KEY} | 此密钥用于启动此容器，并可在 Azure 门户的异常情况检测程序密钥页上。  |
|{BILLING_ENDPOINT_URI} | 计费终结点 URI 值是可在 Azure 门户的异常情况检测程序概述页上。|

在以下示例 `docker run` 命令中，请将这些参数替换为自己的值。

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector:latest \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}
```

此命令：

* 运行容器映像从一个容器，异常探测器
* 分配一个 CPU 核心和 4 GB 内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。 

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](#billing)。

### <a name="running-multiple-containers-on-the-same-host"></a>在同一主机上运行多个容器

如果打算使用所公开的端口运行多个容器，请确保使用不同的端口运行每个容器。 例如，在端口 5000 上运行第一个容器，在端口 5001 上运行第二个容器。

请将 `<container-registry>` 和 `<container-name>` 替换为所用容器的值。 这些容器不需要是同一容器。 可以有异常情况检测程序容器和 LUIS 容器主机上一起运行，或者你可以运行的多个异常情况检测程序容器。 

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

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>停止容器

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>故障排除

如果运行启用了输出[装入点](anomaly-detector-container-configuration.md#mount-settings)和日志记录的容器，该容器会生成有助于排查启动或运行容器时发生的问题的日志文件。 

## <a name="billing"></a>计费

异常情况检测器容器发送帐单到 Azure 的信息，请使用_异常情况检测器_上你的 Azure 帐户的资源。 

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

有关这些选项的详细信息，请参阅[配置容器](anomaly-detector-container-configuration.md)。

## <a name="summary"></a>摘要

在本文中，你学习概念以及用于下载、 安装和运行异常情况检测程序的容器的工作流。 综上所述：

* 异常情况检测程序提供一个 Linux 容器的 Docker，批处理 vs 流式处理，预期的范围推理和敏感度优化封装异常情况检测。
* 从专用 Azure 容器注册表专用容器预览版下载容器映像。
* 容器映像在 Docker 中运行。
* 可以使用 REST API 或 SDK 调用操作时在异常情况检测程序容器中的通过指定主机的容器的 URI。
* 必须在实例化容器时指定账单信息。

> [!IMPORTANT]
> 如果未连接到 Azure 进行计量，则无法授权并运行认知服务容器。 客户需要始终让容器向计量服务传送账单信息。 认知服务容器不发送给 Microsoft 的客户数据 （例如，时间系列数据要分析的）。

## <a name="next-steps"></a>后续步骤

* 查看[配置容器](anomaly-detector-container-configuration.md)了解配置设置
* [了解有关异常情况检测器 API 服务详细信息](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)
