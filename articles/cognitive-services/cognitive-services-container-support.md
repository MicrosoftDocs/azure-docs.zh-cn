---
title: 容器支持
titleSuffix: Azure Cognitive Services
description: 了解 Docker 容器如何使认知服务深入了解你的数据。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 241bda5c684197a43cc5564e950e924fed668b89
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147568"
---
# <a name="container-support-in-azure-cognitive-services"></a>Azure 认知服务中的容器支持

Azure 认知服务中的容器支持让开发人员能够使用与 Azure 中可用的 API 一样丰富的 API，并能够灵活地选择部署和托管随附 [Docker 容器](https://www.docker.com/what-container)的服务的位置。 容器支持目前以预览版的 Azure 认知服务，包括的部分子集：

* [异常情况检测程序](Anomaly-Detector/overview.md)
* [计算机视觉](Computer-vision/Home.md)
* [人脸](Face/Overview.md)
* [窗体识别器](https://go.microsoft.com/fwlink/?linkid=2083826&clcid=0x409)
* [语言理解](LUIS/luis-container-howto.md)(LUIS)
* [Personalizer](https://go.microsoft.com/fwlink/?linkid=2083923&clcid=0x409)
* [语音服务 API](https://go.microsoft.com/fwlink/?linkid=2083926&clcid=0x409)
* [文本分析](text-analytics/overview.md)

容器化是一种软件分发方法，其中应用程序或服务（包括其依赖关系和配置）被一起打包为容器映像。 如果几乎不进行修改，可将容器映像部署在容器主机上。 容器彼此隔离并与基础操作系统隔离，内存占用小于虚拟机。 容器可以从容器映像中实例化以用于短期任务，并在不再需要时将其删除。

以下视频演示如何使用认知服务容器。

[![认知服务的容器演示](./media/index/containers-video-image.png)](https://azure.microsoft.com/resources/videos/containers-support-of-cognitive-services)

认知服务资源位于[Microsoft Azure](https://azure.microsoft.com)。 登录到 [Azure 门户](https://portal.azure.com/)，创建和浏览适用于这些服务的 Azure 资源。

## <a name="features-and-benefits"></a>功能和优势

- **对数据的控制**：允许客户选择认知服务在何处处理其数据。 对于不能将数据发送到云，但需要访问认知服务技术的客户，此功能非常重要。 支持混合环境中的一致性 - 跨数据、管理、标识和安全性。
- **对模型更新的控制**：为客户提供其解决方案中部署的模型的版本控制和更新方面的灵活性。
- **可移植的体系结构**：支持创建可在 Azure、本地和边缘部署的可移植应用程序体系结构。 可直接将容器部署到 [Azure Kubernetes 服务](../aks/index.yml)、[Azure 容器实例](../container-instances/index.yml)，或部署到 [Azure Stack](/azure-stack/operator) 的 [Kubernetes](https://kubernetes.io/) 集群。 有关详细信息，请参阅[将 Kubernetes 部署到 Azure Stack](/azure-stack/user/azure-stack-solution-template-kubernetes-deploy)。
- **高吞吐量/低延迟**：通过使以物理方式运行的认知服务更深入了解其应用程序逻辑和数据，为客户提供缩放功能，以满足高吞吐量和低延迟扩展要求。 容器不限制每秒综合事务数 (TPS)，如果提供了必要的硬件资源，它还可进行纵向或横向扩展，来应对需求。 


## <a name="containers-in-azure-cognitive-services"></a>Azure 认知服务中的容器

Azure 认知服务容器提供以下一组 Docker 容器，其中每个容器都包含 Azure 认知服务中的服务的功能子集：

| 服务 | 受支持的定价层 | 容器 | 描述 |
|---------|----------|----------|-------------|
|[异常情况检测程序](https://go.microsoft.com/fwlink/?linkid=2083925&clcid=0x409) |F0, S0|**Anomaly-Detector** |异常情况检测器 API，可监视并使用机器学习时序数据中检测异常情况。<br>[请求访问权限](https://aka.ms/adcontainer)|
|[计算机视觉](Computer-vision/computer-vision-how-to-install-containers.md) |F0, S1|识别文本 |从具有不同表面和背景的各种对象的图像中提取打印文本，例如收据、海报和名片。<br/><br/>**重要提示：** 识别文本容器目前仅适用于英语。<br>[请求访问权限](Computer-vision/computer-vision-how-to-install-containers.md#request-access-to-the-private-container-registry)|
|[人脸](Face/face-how-to-install-containers.md) |F0, S0|**人脸** |检测图像中的人脸并标识属性，包括人脸特征（例如，鼻子和眼睛）、性别、年龄和其他计算机预测的面部特征。 除检测外，人脸还可以使用置信分数检查同一/不同图像中的两张人脸，或根据数据库比较人脸，以查看是否已存在类似或相同的人脸。 还可以使用共享视觉特征将类似人脸整理为许多组。<br>[请求访问权限](Face/face-how-to-install-containers.md#request-access-to-the-private-container-registry) |
|[窗体识别器](https://go.microsoft.com/fwlink/?linkid=2083826&clcid=0x409) |F0, S0|**窗体识别器** |窗体了解应用机器学习技术来识别和提取窗体中的键 / 值对和表。<br>[请求访问权限](https://aka.ms/FormRecognizerContainerRequestAccess)|
|[LUIS](LUIS/luis-container-howto.md) |F0, S0|**LUIS**（[映像](https://go.microsoft.com/fwlink/?linkid=2043204&clcid=0x409)）|可将已训练或已发布的语言理解模型（也称为 LUIS 应用）加载到 docker 容器中并提供对容器的 API 终结点中的查询预测的访问权限。 可以从容器中收集查询日志并将这些日志上传回 [LUIS 门户](https://www.luis.ai)以提高应用的预测准确性。|
|[Personalizer](https://go.microsoft.com/fwlink/?linkid=2083923&clcid=0x409) |F0, S0|**Personalizer** ([映像](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409))|Azure 个性化体验创建服务是基于云的 API 服务，可让你选择要显示给用户的最佳体验，并从其实时行为中学习信息。|
|[语音服务 API](https://go.microsoft.com/fwlink/?linkid=2083926&clcid=0x409) |F0, S0|**语音转文本** |将连续的实时语音转换为文本。<br>[请求访问权限](https://aka.ms/speechcontainerspreview/)|
|[语音服务 API](https://go.microsoft.com/fwlink/?linkid=2083926&clcid=0x409) |F0, S0|**文本转语音** |将文本转换为自然发音的语音。<br>[请求访问权限](https://aka.ms/speechcontainerspreview/)|
|[文本分析](text-analytics/how-tos/text-analytics-how-to-install-containers.md) |F0 S|关键短语提取（[映像](https://go.microsoft.com/fwlink/?linkid=2018757&clcid=0x409)） |提取关键短语，以标识要点。 例如，针对输入文本“The food was delicious and there were wonderful staff”，该 API 会返回谈话要点：“food”和“wonderful staff”。 |
|[文本分析](text-analytics/how-tos/text-analytics-how-to-install-containers.md)|F0 S|语言检测（[映像](https://go.microsoft.com/fwlink/?linkid=2018759&clcid=0x409)） |针对多达 120 种语言，检测输入文本是使用哪种语言编写的，并报告请求中提交的每个文档的单个语言代码。 语言代码与表示评分强度的评分相搭配。 |
|[文本分析](text-analytics/how-tos/text-analytics-how-to-install-containers.md)|F0 S|情绪分析（[映像](https://go.microsoft.com/fwlink/?linkid=2018654&clcid=0x409)） |分析原始文本，获取正面或负面情绪的线索。 此 API 针对每个文档返回介于 0 和 1 之间的情绪评分，1 是最积极的评分。 分析模型已使用 Microsoft 提供的大量文本正文和自然语言技术进行预先训练。 对于[选定的语言](./text-analytics/language-support.md)，该 API 可以分析和评分提供的任何原始文本，并直接将结果返回给调用方应用程序。 |

此外，认知服务中支持一些容器[**一体的产品/服务**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne)资源键。 可以创建一个单一的认知服务-一体化资源，并在以下服务支持的服务之间使用相同的计费密钥：

* 计算机视觉
* 人脸
* LUIS
* 文本分析

## <a name="container-availability-in-azure-cognitive-services"></a>Azure 认知服务中的容器可用性

Azure 认知服务容器通过 Azure 订阅公开发布，并可以从 Microsoft 容器注册表或 Docker 中心拉取 Docker 容器映像。 可以使用 [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) 命令从相应注册表下载容器映像。

> [!IMPORTANT]
> 目前，必须完成注册过程才能访问[人脸](Face/face-how-to-install-containers.md)和[识别文本](Computer-vision/computer-vision-how-to-install-containers.md)容器，注册过程需要填写关于你、你的公司和要为其实现容器的用例的问题的问卷调查，然后提交。 授予访问权限并提供凭据后，即可为人脸和识别文本容器从 Azure 容器注册表承载的私有容器注册表中拉取容器映像。

## <a name="prerequisites"></a>必备组件

使用 Azure 认知服务容器之前，必须先满足以下先决条件：

**Docker 引擎**：必须在本地安装 Docker 引擎。 Docker 提供用于在 [macOS](https://docs.docker.com/docker-for-mac/)、[Linux](https://docs.docker.com/engine/installation/#supported-platforms) 和 [Windows](https://docs.docker.com/docker-for-windows/) 上配置 Docker 环境的包。 在 Windows 上，必须将 Docker 配置为支持 Linux 容器。 还可直接将 Docker 容器直接部署到 [Azure Kubernetes 服务](../aks/index.yml)或 [Azure 容器实例](../container-instances/index.yml)。

必须将 Docker 配置为允许容器连接 Azure 并向其发送账单数据。

**熟悉 Microsoft 容器注册表和 Docker**：应对 Microsoft 容器注册表和 Docker 概念有基本的了解，例如注册表、存储库、容器和容器映像，以及基本的 `docker` 命令的知识。

有关 Docker 和容器的基础知识，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。

各容器还可以有其自己的要求，包括服务器和内存分配要求。

## <a name="developer-samples"></a>开发人员示例

可在 [GitHub 存储库](https://github.com/Azure-Samples/cognitive-services-containers-samples)中查看开发人员示例。

## <a name="next-steps"></a>后续步骤

安装和浏览 Azure 认知服务中的容器提供的功能：

* [异常情况检测程序容器](Anomaly-Detector/anomaly-detector-container-howto.md)
* [计算机视觉容器](Computer-vision/computer-vision-how-to-install-containers.md)
* [人脸容器](Face/face-how-to-install-containers.md)
* [窗体识别器容器](https://go.microsoft.com/fwlink/?linkid=2083826&clcid=0x409)
* [语言理解 (LUIS) 容器](LUIS/luis-container-howto.md)
* [Personalizer 容器](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409)
* [语音服务 API 容器](https://go.microsoft.com/fwlink/?linkid=2083926&clcid=0x409)
* [文本分析容器](text-analytics/how-tos/text-analytics-how-to-install-containers.md)