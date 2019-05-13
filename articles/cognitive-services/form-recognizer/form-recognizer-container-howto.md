---
title: 安装和运行容器 - 表单识别器
titleSuffix: Azure Cognitive Services
description: 了解如何使用表单识别器容器来分析表单和表数据。
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: form-recognizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: pafarley
ms.openlocfilehash: a7159fccc9c4ef232cfca08b173e712e268343ea
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507822"
---
# <a name="install-and-run-form-recognizer-containers"></a>安装和运行表单识别器容器
表单识别器应用机器学习技术从表与表单中识别和提取键值对。 它将值和表项关联到表单，然后输出包含原始文件中的关系的结构化数据。 可以使用简单的 REST API 调用自定义表单识别器模型，以降低复杂性，并轻松地将该模型集成到工作流自动化过程或其他应用程序中。 只需五个文档（或一个空表单），因此，可以快速准确地获取根据特定内容定制的结果，而无需进行繁琐的手动干预，也不需要具备丰富的数据科学专业知识。 表单识别器不要求标记或标注数据。

|函数|功能|
|-|-|
|表单识别器| <li>处理 PDF、PNG 和 JPG 类型的文件。<li>使用具有相同布局的 5 个表单（最少数目）即可训练自定义模型。 <li>提取键值对和表信息。 <li>使用认知服务计算机视觉 API RecognizeText 从表单内部的图像中检测和提取打印的文本。<li>无需进行标注或标记。|

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

使用表单识别器容器之前，必须满足以下先决条件：

|必选|目的|
|--|--|
|Docker 引擎| 需要在[主计算机](#the-host-computer)上安装 Docker 引擎。 Docker 提供用于在 [macOS](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 和 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 上配置 Docker 环境的包。 有关 Docker 和容器的基础知识，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。<br><br> 必须将 Docker 配置为允许容器连接 Azure 并向其发送账单数据。 <br><br> 在 Windows 上，还必须将 Docker 配置为支持 Linux 容器。<br><br>|
|熟悉 Docker | 应对 Docker 概念有基本的了解，例如注册表、存储库、容器和容器映像，以及基本的 `docker` 命令的知识。|
|Azure CLI| 需要在主机上安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。|
|计算机视觉 API 资源| 若要处理扫描的文档和图像，需要配置**计算机视觉资源**。 可以访问作为 Azure 资源（REST API 或 SDK）或 `cognitive-services-recognize-text` [容器](../Computer-vision/computer-vision-how-to-install-containers.md##get-the-container-image-with-docker-pull)提供的“识别文本”功能。 使用此功能需要按普通费率付费。 <br><br>必须传入特定计算机视觉资源（Azure 云或认知服务容器）的密钥和计费终结点。 请将此密钥和计费终结点用作 {COMPUTER_VISION_API_KEY} 和 {COMPUTER_VISION_BILLING_ENDPOINT_URI}。<br><br> 如果使用 **`cognitive-services-recognize-text` 容器**，请确保：<br><br>*表单识别器容器的计算机视觉密钥是在 `cognitive-services-recognize-text` 容器的计算机视觉 `docker run` 命令中指定的密钥。<br>*计费终结点是容器的终结点，例如 `https://localhost:5000`。 如果在同一台主机上同时使用计算机视觉和表单识别器容器，不能使用默认端口 `5000` 将两者同时启动。  |  
|表单识别器资源 |若要使用这些容器，必须具有：<br><br>一个表单识别器 Azure 资源，用于获取关联的计费密钥和计费终结点 URI。 这两个值都可以从 Azure 门户中的表单识别器“概述”和“密钥”页获取；必须获取这两个值才能启动该容器。<br><br>**{BILLING_KEY}**：资源密钥<br><br>**{BILLING_ENDPOINT_URI}**：终结点 URI 示例如下：`https://westus.api.cognitive.microsoft.com/forms/v1.0`| 

## <a name="request-access-to-the-container-registry"></a>请求访问容器注册表

在请求访问该容器之前，必须先填写并提交[认知服务表单识别器容器请求表单](https://aka.ms/FormRecognizerRequestAccess)。 这也会注册计算机视觉。 不需要单独注册计算机视觉请求表单。 

[!INCLUDE [Request access to the container registry](../../../includes/cognitive-services-containers-request-access-only.md)]

[!INCLUDE [Authenticate to the container registry](../../../includes/cognitive-services-containers-access-registry.md)]

## <a name="the-host-computer"></a>主计算机

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>容器要求和建议

下表描述了为每个表单识别器容器分配的最小和建议 CPU 核心数和内存。

| 容器 | 最小值 | 建议 |
|-----------|---------|-------------|
|cognitive-services-form-recognizer | 2 核心，4 GB 内存 | 4 核心，8 GB 内存 |

* 每个核心必须至少为 2.6 千兆赫 (GHz) 或更快。
* TPS - 每秒事务数

核心和内存对应于 `--cpus` 和 `--memory` 设置，用作 `docker run` 命令的一部分。

> [!Note]
> 最小和建议值基于 Docker 限制，而不是基于主机资源。

## <a name="get-the-container-image-with-docker-pull-command"></a>使用 docker pull 命令获取容器映像

可以使用表单识别器的容器映像。

| 容器 | 存储库 |
|-----------|------------|
| cognitive-services-form-recognizer | `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest` |

若要使用 `cognitive-services-recognize-text` [容器](../Computer-vision/computer-vision-how-to-install-containers.md##get-the-container-image-with-docker-pull)而不是表单识别器服务，请确保在使用 `docker pull` 命令时使用正确的容器名称： 

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest
```

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]


### <a name="docker-pull-for-the-form-recognizer-container"></a>适用于表单识别器容器的 Docker 拉取

#### <a name="form-recognizer"></a>表单识别器

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest
```

## <a name="how-to-use-the-container"></a>如何使用容器

一旦容器位于[主计算机](#the-host-computer)上，请通过以下过程使用容器。

1. 使用所需的而不是所用的计费设置来[运行容器](#run-the-container-with-docker-run)。 提供 `docker run` 命令的多个[示例](form-recognizer-container-configuration.md#example-docker-run-commands)。
1. [查询容器的预测终结点](#query-the-containers-prediction-endpoint)。

## <a name="run-the-container-with-docker-run"></a>通过 `docker run` 运行容器

使用 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令运行三个容器中的任意一个。 该命令使用以下参数：

| 占位符 | 值 |
|-------------|-------|
|{BILLING_KEY} | 此密钥用于启动容器，可以从 Azure 门户的表单识别器“密钥”页获取。  |
|{BILLING_ENDPOINT_URI} | 可以从 Azure 门户的表单识别器“概述”页获取计费终结点 URI 值。|
|{COMPUTER_VISION_API_KEY}| 可以从 Azure 门户的计算机视觉“API 密钥”页获取。|
|{COMPUTER_VISION_ENDPOINT_URI}|计费终结点。 如果使用基于云的计算机视觉资源，则可以从 Azure 门户的计算机视觉“API 概述”页获取 URI 值。 如果使用 `cognitive-services-recognize-text` 容器，请使用在 `docker run` 命令中传递给容器的计费终结点 URL。|

在以下示例 `docker run` 命令中，请将这些参数替换为自己的值。

### <a name="form-recognizer"></a>表单识别器

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

此命令：

* 通过容器映像运行表单识别器容器
* 分配 2 个 CPU 核心和 8 GB 内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。
* 将 /input 和 /output 卷装载到容器

[!INCLUDE [Running multiple containers on the same host H2](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

### <a name="run-separate-containers-as-separate-docker-run-commands"></a>以独立的 docker run 命令形式运行单独的容器

对于托管在同一主机本地的表单识别器和文本识别器组合，可运行下面所示的两个示例 Docker CLI 命令。

在端口 5000 上运行第一个容器。 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

在端口 5001 上运行第二个容器。


```bash 
docker run --rm -it -p 5001:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text \
Eula=accept \
Billing={COMPUTER_VISION_ENDPOINT_URI} \
ApiKey={COMPUTER_VISION_API_KEY}
```
每个后续容器应位于不同的端口上。 

### <a name="run-separate-containers-with-docker-compose"></a>使用 Docker Compose 运行单独的容器

对于托管在同一主机本地的表单识别器和文本识别器组合，可运行下面所示的示例 Docker Compose YAML 文件。 对于 `formrecognizer` 和 `ocr` 容器，文本识别器 `{COMPUTER_VISION_API_KEY}` 必须相同。 `{COMPUTER_VISION_ENDPOINT_URI}` 只在 `ocr` 容器中使用，因为 `formrecognizer` 容器使用 `ocr` 名称和端口。 

```docker
version: '3.3'
services:   
  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{COMPUTER_VISION_ENDPOINT_URI}"
      apikey: {COMPUTER_VISION_API_KEY}  

  formrecognizer:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{BILLING_ENDPOINT_URI}"
      apikey: {BILLING_KEY}
      FormRecognizer__ComputerVisionApiKey: {COMPUTER_VISION_API_KEY}
      FormRecognizer__ComputerVisionEndpointUri: "http://ocr:5000"
      FormRecognizer__SyncProcessTaskCancelLimitInSecs: 75
    links:
      - ocr
    volumes:
      - type: bind
        source: c:\output
        target: /output
      - type: bind
        source: c:\input
        target: /input
    ports:
      - "5000:5000"  
```


> [!IMPORTANT]
> 必须指定 `Eula`、`Billing`、`ApiKey`、`FormRecognizer:ComputerVisionApiKey` 和 `FormRecognizer:ComputerVisionEndpointUri` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](#billing)。

## <a name="query-the-containers-prediction-endpoint"></a>查询容器的预测终结点

|容器|终结点|
|--|--|
|form-recognizer|http://localhost:5000


### <a name="form-recognizer"></a>表单识别器

该容器提供基于 WebSocket 的查询终结点 API，可通过[表单识别器服务 SDK 文档](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/)访问这些 API。

默认情况下，表单识别器 SDK 使用联机服务。 若要使用该容器，需要更改初始化方法。 请参阅下面的示例。

#### <a name="for-c"></a>对于 C#

请从使用此 Azure 云初始化调用：

```C#
var config = FormRecognizerConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

更改为使用容器终结点发出此调用：

```C#
var config = FormRecognizerConfig.FromEndpoint("ws://localhost:5000/formrecognizer/v1.0-preview/custom", "YourSubscriptionKey");
```

#### <a name="for-python"></a>对于 Python

请从使用此 Azure 云初始化调用

```python
formrecognizer_config = formrecognizersdk.FormRecognizerConfig(subscription=formrecognizer_key, region=service_region)
```

更改为使用容器终结点发出此调用：

```python
formrecognizer_config = formrecognizersdk.FormRecognizerConfig(subscription=formrecognizer_key, endpoint="ws://localhost:5000/formrecognizer/v1.0-preview/custom"
```

### <a name="form-recognizer"></a>表单识别器

该容器提供 REST 终结点 API，可在[此处](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api/operations/AnalyzeWithCustomModel)找到这些 API。


[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]


## <a name="stop-the-container"></a>停止容器

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>故障排除

运行该容器时，该容器将使用 **stdout** 和 **stderr** 来输出信息，这些信息有助于排查启动或运行容器时发生的问题。

## <a name="billing"></a>计费

表单识别器容器使用 Azure 帐户中的“表单识别器”资源向 Azure 发送计费信息。

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

有关这些选项的详细信息，请参阅[配置容器](form-recognizer-container-configuration.md)。

## <a name="summary"></a>摘要

本文介绍了与下载、安装和运行表单识别器容器相关的概念与工作流。 综上所述：

* 表单识别器提供一个适用于 Docker 的 Linux 容器。
* 可从 Azure 中的专用容器注册表下载容器映像。
* 容器映像在 Docker 中运行。
* 可以使用 REST API 或 SDK 通过指定容器的主机 URI 来调用表单识别器容器中的操作。
* 必须在实例化容器时指定账单信息。

> [!IMPORTANT]
>  如果未连接到 Azure 进行计量，则无法授权并运行认知服务容器。 客户需要始终让容器向计量服务传送账单信息。 认知服务容器不会将客户数据（例如，正在分析的图像或文本）发送给 Microsoft。

## <a name="next-steps"></a>后续步骤

* 查看[配置容器](form-recognizer-container-configuration.md)了解配置设置
* 使用更多[认知服务容器](../cognitive-services-container-support.md)
