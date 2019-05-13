---
title: 个性化体验创建服务工作原理
titleSuffix: Azure Cognitive Services
description: 了解如何使用 docker run 命令配置设置运行个性化体验创建服务容器。
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: 85344d432c169ce8c4adffd059059660672df02e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65156812"
---
# <a name="configure-personalizer-containers"></a>配置个性化体验创建服务容器

**个性化体验创建服务**容器运行时环境是使用 `docker run` 命令参数配置的。 此容器有多个必需设置，以及一些可选设置。 多个[示例](#example-docker-run-commands)命令均可用。 容器专用设置是帐单设置。 

## <a name="configuration-settings"></a>配置设置

此容器具有以下配置设置：

|必选|设置|目的|
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
> [`ApiKey`](#apikey-configuration-setting)、[`Billing`](#billing-configuration-setting) 和 [`Eula`](#eula-setting) 设置一起使用。必须为所有三个设置提供有效值，否则容器将无法启动。 有关使用这些配置设置实例化容器的详细信息，请参阅[计费](personalizer-container-howto.md#billing)。

## <a name="apikey-configuration-setting"></a>ApiKey 配置设置

`ApiKey` 设置指定用于跟踪容器账单信息的 Azure 资源键。 必须为 ApiKey 指定值，并且该值必须是为 [`Billing`](#billing-configuration-setting) 配置设置指定的个性化体验创建服务资源的有效密钥。

可以在以下位置找到此设置：

* Azure 门户：**个性化体验创建服务**“资源管理”部分的“密钥”下

## <a name="applicationinsights-setting"></a>ApplicationInsights 设置

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Billing 配置设置

`Billing` 设置指定 Azure 上用于计量容器的账单信息的个性化体验创建服务资源的终结点 URI。 必须为此配置设置指定值，并且该值必须是 Azure 上个性化体验创建服务资源的有效终结点 URI。

可以在以下位置找到此设置：

* Azure 门户：**个性化体验创建服务的**“概览”，带有 `Endpoint` 标签

|必选| 名称 | 数据类型 | 说明 |
|--|------|-----------|-------------|
|是| `Billing` | String | 账单终结点 URI<br><br>示例：<br>`Billing=https://westus.api.cognitive.microsoft.com/sts/v1.0` |

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

容器不使用输入或输出装载来存储训练或服务数据。 

主机确切语法的安装位置因主机操作系统不同而异。 此外，由于 docker 服务帐户使用的权限与主机安装位置权限之间的冲突，可能无法访问[主计算机](personalizer-container-howto.md#the-host-computer)的装载位置。 

|可选| 名称 | 数据类型 | 说明 |
|-------|------|-----------|-------------|
|不允许| `Input` | String | 个性化体验创建服务容器不使用此项。|
|可选| `Output` | String | 输出装入点的目标。 默认值为 `/output`。 这是日志的位置。 这包括容器日志。 <br><br>示例：<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Docker 运行命令示例 

以下示例使用的配置设置说明如何编写和使用 `docker run` 命令。  运行后，容器将继续运行，直到[停止](personalizer-container-howto.md#stop-the-container)它。

* **行继续符**：以下各部分中的 Docker 命令使用反斜杠 `\` 作为 bash shell 的行继续符。 根据主机操作系统的要求替换或删除字符。 例如，Windows 的行继续符是插入符号 `^`。 将反斜杠替换为插入符号。 
* **参数顺序**：除非很熟悉 Docker 容器，否则不要更改参数顺序。

将括号 `{}` 中的值替换为你自己的值：

| 占位符 | 值 | 格式或示例 |
|-------------|-------|---|
|{BILLING_KEY} | 个性化体验创建服务资源的终结点密钥。 |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx|
|{BILLING_ENDPOINT_URI} | 包括区域的账单终结点值。|`https://westus2.api.cognitive.microsoft.com/`|

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](personalizer-container-howto.md#billing)。
> ApiKey 值是 Azure 个性化体验创建服务“资源密钥”页中提供的“密钥”。 

## <a name="personalizer-container-docker-examples"></a>个性化体验创建服务容器 Docker 示例

以下 Docker 示例适用于个性化体验创建服务容器。 

### <a name="basic-example"></a>基本示例 

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  mcr.microsoft.com/azure-cognitive-services/[container-name] \
  Eula=accept \
  Billing={BILLING_ENDPOINT_URI} \
  ApiKey={BILLING_KEY} 
  ```

### <a name="logging-example-with-command-line-arguments"></a>使用命令行参数的日志记录示例

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  mcr.microsoft.com/azure-cognitive-services/[container-name] \
  Eula=accept \
  Billing={BILLING_ENDPOINT_URI} ApiKey={BILLING_KEY} \
  Logging:Console:LogLevel:Default=Information
  ```

## <a name="next-steps"></a>后续步骤

[了解如何使用 Docker 容器](personalizer-container-howto.md)