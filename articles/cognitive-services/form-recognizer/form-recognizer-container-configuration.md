---
title: 配置容器 - 表单识别器
titleSuffix: Azure Cognitive Services
description: 了解如何将表单识别器容器配置为分析表单和表数据。
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: form-recognizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: pafarley
ms.openlocfilehash: 3f6cc32f6fb2a9fb11220ac1a3134fb3ae4d2f84
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147467"
---
# <a name="configure-form-recognizer-containers"></a>配置表单识别器容器

表单识别器容器使客户能够生成经过优化的应用程序体系结构，以利用强大的云功能和边缘位置。

**表单识别器**容器运行时环境是使用 `docker run` 命令参数配置的。 此容器有多个必需设置，以及一些可选设置。 多个[示例](#example-docker-run-commands)命令均可用。 容器专用设置是帐单设置。

## <a name="configuration-settings"></a>配置设置

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> [`ApiKey`](#apikey-configuration-setting)、[`Billing`](#billing-configuration-setting) 和 [`Eula`](#eula-setting) 设置一起使用。必须为所有三个设置提供有效值，否则容器将无法启动。 有关使用这些配置设置实例化容器的详细信息，请参阅[计费](form-recognizer-container-howto.md#billing)。

## <a name="apikey-configuration-setting"></a>ApiKey 配置设置

`ApiKey` 设置指定用于跟踪容器账单信息的 Azure 资源键。 必须为 ApiKey 指定值，并且该值必须是为 [`Billing`](#billing-configuration-setting) 配置设置指定的表单识别器资源的有效密钥。

可以在以下位置找到此设置：

* Azure 门户：**表单识别器**“资源管理”部分的“密钥”下

## <a name="applicationinsights-setting"></a>ApplicationInsights 设置

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Billing 配置设置

`Billing` 设置指定 Azure 上用于计量容器的账单信息的表单识别器资源的终结点 URI。 必须为此配置设置指定值，并且该值必须是 Azure 上表单识别器资源的有效终结点 URI。 容器约每 10 到 15 分钟报告一次使用情况。

可以在以下位置找到此设置：

* Azure 门户：**表单识别器的**“概述”，带有 `Endpoint` 标签

|必选| 名称 | 数据类型 | 说明 |
|--|------|-----------|-------------|
|是| `Billing` | String | 账单终结点 URI<br><br>示例：<br>`Billing=https://westus2.api.cognitive.microsoft.com/` |

## <a name="eula-setting"></a>Eula 设置

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Fluentd 设置

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="logging-settings"></a>日志记录设置

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## <a name="mount-settings"></a>装载设置

使用绑定装载从容器读取数据并将数据写入容器。 可以通过在 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令中指定 `--mount` 选项来指定输入装载或输出装载。

表单识别器容器需要输入和输出装载。 输入装入点可以是只读的，需要使用它来访问用于训练和评分的数据。 输出装入点必须是可写的，用于存储模型和临时数据。

主机确切语法的安装位置因主机操作系统不同而异。 另外，由于 Docker 服务帐户使用的权限与主机装载位置权限之间有冲突，因此可能无法访问[主计算机](form-recognizer-container-howto.md#the-host-computer)的装载位置。

|可选| 名称 | 数据类型 | 说明 |
|-------|------|-----------|-------------|
|必选| `Input` | String | 输入装入点的目标。 默认值为 `/input`。    <br><br>示例：<br>`--mount type=bind,src=c:\input,target=/input`|
|必选| `Output` | String | 输出装入点的目标。 默认值为 `/output`。  <br><br>示例：<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Docker 运行命令示例

以下示例使用的配置设置说明如何编写和使用 `docker run` 命令。  运行后，容器将继续运行，直到[停止](form-recognizer-container-howto.md#stop-the-container)它。

* **行继续符**：以下各部分中的 Docker 命令使用反斜杠 `\` 作为行继续符。 根据主机操作系统的要求替换或删除字符。
* **参数顺序**：除非很熟悉 Docker 容器，否则不要更改参数顺序。

将 {_argument_name_} 替换为为你自己的值：

| 占位符 | 值 |
|-------------|-------|
|{BILLING_KEY} | 此密钥用于启动容器，可以从 Azure 门户的表单识别器“密钥”页获取。  |
|{BILLING_ENDPOINT_URI} | 可以从 Azure 门户的表单识别器“概述”页获取计费终结点 URI 值。|
|{COMPUTER_VISION_API_KEY}| 可以从 Azure 门户的计算机视觉“API 密钥”页获取。|
|{COMPUTER_VISION_ENDPOINT_URI}|计费终结点。 如果使用基于云的计算机视觉资源，则可以从 Azure 门户的计算机视觉“API 概述”页获取 URI 值。 如果使用 `cognitive-services-recognize-text` 容器，请使用在 `docker run` 命令中传递给容器的计费终结点 URL。|

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](#billing-configuration-setting)。
> ApiKey 值是 Azure 表单识别器“资源密钥”页中提供的“密钥”。

## <a name="form-recognizer-container-docker-examples"></a>表单识别器容器 Docker 示例

下面是表单识别器容器的 Docker 示例。

### <a name="basic-example-for-form-recognizer"></a>表单识别器基本示例

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

### <a name="logging-example-for-form-recognizer"></a>表单识别器日志记录示例

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
Logging:Console:LogLevel:Default=Information
```


## <a name="next-steps"></a>后续步骤

* 查看[如何安装和运行容器](form-recognizer-container-howto.md)
