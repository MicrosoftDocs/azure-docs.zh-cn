---
title: 配置容器-异常情况检测程序
titleSuffix: Azure Cognitive Services
description: 使用配置的异常情况检测程序容器运行时环境`docker run`命令参数。 此容器有多个必需设置，以及一些可选设置。
services: cognitive-services
author: aahill
ms.service: cognitive-services
ms.subservice: anomaly-detection
ms.topic: article
ms.date: 05/07/2019
ms.author: aahi
ms.openlocfilehash: 0d09ce29aa5431de3eb82e5d9fe7440d4e3352e1
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65026386"
---
# <a name="configure-anomaly-detector-containers"></a>配置异常情况检测程序容器

**异常情况检测器**容器运行时环境配置为使用`docker run`命令参数。 此容器有多个必需设置，以及一些可选设置。 多个[示例](#example-docker-run-commands)命令均可用。 容器专用设置是帐单设置。 

# <a name="configuration-settings"></a>配置设置

此容器具有以下配置设置：

|需要|设置|目的|
|--|--|--|
|是|[ApiKey](#apikey-configuration-setting)|用于跟踪账单信息。|
|否|[ApplicationInsights](#applicationinsights-setting)|允许向容器添加 [Azure Application Insights](https://docs.microsoft.com/azure/application-insights) 遥测支持。|
|是|[计费](#billing-configuration-setting)|指定 Azure 上服务资源的终结点 URI。|
|是|[Eula](#eula-setting)| 表示已接受容器的许可条款。|
|否|[Fluentd](#fluentd-settings)|将日志和（可选）指标数据写入 Fluentd 服务器。|
|否|[Http 代理](#http-proxy-credentials-settings)|配置 HTTP 代理以发出出站请求。|
|否|[日志记录](#logging-settings)|为容器提供 ASP.NET Core 日志记录支持。 |
|否|[Mounts](#mount-settings)|从主计算机读取数据并将其写入到容器，以及从容器读回数据并将其写回到主计算机。|

> [!IMPORTANT]
> [`ApiKey`](#apikey-configuration-setting)、[`Billing`](#billing-configuration-setting) 和 [`Eula`](#eula-setting) 设置一起使用。必须为所有三个设置提供有效值，否则容器将无法启动。 有关使用这些配置设置实例化容器的详细信息，请参阅[计费](anomaly-detector-container-howto.md#billing)。

## <a name="apikey-configuration-setting"></a>ApiKey 配置设置

`ApiKey` 设置指定用于跟踪容器账单信息的 Azure 资源键。 必须为 ApiKey 指定一个值，该值必须为有效的密钥_异常情况检测器_为指定的资源[ `Billing` ](#billing-configuration-setting)配置设置。

可以在以下位置找到此设置：

* Azure 门户：**异常情况检测器**资源管理下**密钥**

## <a name="applicationinsights-setting"></a>ApplicationInsights 设置

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Billing 配置设置

`Billing`设置指定的终结点 URI 的_异常情况检测器_使用在 Azure 上的资源要计数的容器的计费信息。 必须指定此配置设置，一个值，该值必须是有效的终结点 URI 对于_异常情况检测器_在 Azure 上的资源。

可以在以下位置找到此设置：

* Azure 门户：**异常情况检测器**概述，标记为 `Endpoint`

|需要| 名称 | 数据类型 | 描述 |
|--|------|-----------|-------------|
|是| `Billing` | String | 账单终结点 URI<br><br>示例：<br>`Billing=https://westus2.api.cognitive.microsoft.com` |

## <a name="eula-setting"></a>Eula 设置

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Fluentd 设置

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Http 代理凭据设置

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>日志记录设置
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## <a name="mount-settings"></a>装载设置

使用绑定装载从容器读取数据并将数据写入容器。 可以通过在 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令中指定 `--mount` 选项来指定输入装载或输出装载。

异常情况检测器容器不使用输入或输出装载可存储培训或服务的数据。 

主机确切语法的安装位置因主机操作系统不同而异。 另外，由于 Docker 服务帐户使用的权限与主机装载位置权限之间有冲突，因此可能无法访问[主计算机](anomaly-detector-container-howto.md#the-host-computer)的装载位置。 

|可选| 名称 | 数据类型 | 描述 |
|-------|------|-----------|-------------|
|不允许| `Input` | String | 异常情况检测器容器请勿使用此功能。|
|可选| `Output` | String | 输出装入点的目标。 默认值为 `/output`。 这是日志的位置。 这包括容器日志。 <br><br>示例：<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Docker 运行命令示例 

以下示例使用的配置设置说明如何编写和使用 `docker run` 命令。  运行后，容器将继续运行，直到[停止](anomaly-detector-container-howto.md#stop-the-container)它。

* **行继续符**：以下各节中的 Docker 命令使用反斜杠， `\`，作为行延续字符，bash shell。 根据主机操作系统的要求替换或删除字符。 例如，适用于 windows 的行延续字符是插入符号， `^`。 将插入符号替换为反斜杠。 
* **参数顺序**：除非很熟悉 Docker 容器，否则不要更改参数顺序。

将在方括号中，值为`{}`，使用你自己的值：

| 占位符 | 值 | 格式或示例 |
|-------------|-------|---|
|{BILLING_KEY} | 异常情况检测程序资源的终结点密钥。 |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx|
|{BILLING_ENDPOINT_URI} | 包括区域的账单终结点值。|`https://westus2.api.cognitive.microsoft.com`|

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](anomaly-detector-container-howto.md#billing)。
> ApiKey 当值**密钥**从 Azure 异常情况检测程序资源的密钥页。 

## <a name="anomaly-detector-container-docker-examples"></a>异常情况检测器容器 Docker 示例

以下 Docker 示例是为异常情况检测程序容器。 

### <a name="basic-example"></a>基本示例 

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector \
  Eula=accept \
  Billing={BILLING_ENDPOINT_URI} \
  ApiKey={BILLING_KEY} 
  ```

### <a name="logging-example-with-command-line-arguments"></a>使用命令行参数的日志记录示例

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector \
  Eula=accept \
  Billing={BILLING_ENDPOINT_URI} ApiKey={BILLING_KEY} \
  Logging:Console:LogLevel:Default=Information
  ```
