---
title: MLOps:管理、 部署和监视机器学习模型
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务的 MLOps： 部署、 管理和监视才能持续改进这些模型。 可以在本地计算机上或从其他源部署使用 Azure 机器学习服务训练的模型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: chris-lauren
ms.author: clauren
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 18a80af8422e30ce3e87395449fca7b5f6a73762
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025019"
---
# <a name="mlops-manage-deploy-and-monitor-models-with-azure-machine-learning-service"></a>MLOps:使用 Azure 机器学习服务管理、部署和监视模型

本文介绍如何使用 Azure 机器学习服务来部署、管理和监视模型，以便不断改进它们。 可以在本地计算机上部署通过 Azure 机器学习训练的模型，也可从其他源进行部署。 

下图演示了完整的部署工作流：[![为 Azure 机器学习部署工作流](media/concept-model-management-and-deployment/deployment-pipeline.png)](media/concept-model-management-and-deployment/deployment-pipeline.png#lightbox)

MLOps / 部署工作流包括以下步骤：
1. 在 Azure 机器学习服务工作区中托管的注册表中**注册模型**
1. **使用**云中、 在 IoT 设备，或用于分析 Power BI 中的 web 服务中的模型。
1. **监视和收集数据**
1. 更新部署以使用新映像。

单独或作为单个命令的一部分，可以执行每个步骤。 此外，还可以创建**CI/CD 工作流**中此图形所示。

[![Azure 机器学习持续集成/持续部署 (CI/CD) 周期](media/concept-model-management-and-deployment/model-ci-cd.png)](media/concept-model-management-and-deployment/model-ci-cd.png#lightbox)

> [!VIDEO https://www.youtube.com/embed/0MaHb070H_8]

## <a name="step-1-register-model"></a>步骤 1：注册模型

通过模型注册，可在 Azure 云的工作区中存储模型并确定模型版本。 使用模型注册表，可轻松组织和跟踪定型的模型。
 
按名称和版本标识已注册的模型。 每次使用与现有名称相同的名称来注册模型时，注册表都会将版本递增。 也可在注册过程中提供其他的元数据标记，这些标记可以在搜索模型时使用。 Azure 机器学习服务支持使用 Python 3.5.2 加载或更高版本可以是任何模型。

不能删除正在使用的活动部署中的模型。

有关详细信息，请参阅[部署模型](how-to-deploy-and-where.md#registermodel)的注册模型部分。

有关如何注册采用 pickle 格式存储的模型示例，请参阅[教程：训练映像分类模型](tutorial-deploy-models-with-aml.md)。

## <a name="step-2-use-the-model"></a>步骤 2：使用模型

可以为 web 服务，IoT Edge 设备上或进行从 Power BI 等服务分析使用机器学习模型。

### <a name="web-service"></a>Web 服务

可以使用您的模型中**web 服务**以下的计算目标：

* Azure 容器实例
* Azure Kubernetes 服务

若要部署为 web 服务模型，您必须提供：

* 模型或模型的系综。
* 若要使用该模型所需的依赖项。 例如，一个脚本来接受请求并调用模型、 conda 依赖项等。
* 部署配置，以描述如何以及在何处部署模型。

有关详细信息，请参阅[部署模型](how-to-deploy-and-where.md)。

### <a name="iot-edge-devices"></a>IoT Edge 设备

可以将模型与 IoT 设备，通过使用**Azure IoT Edge 模块**。 IoT Edge 模块部署到硬件设备，这样在设备上的推断。

有关详细信息，请参阅[部署模型](how-to-deploy-and-where.md)。

### <a name="analytics"></a>分析

Microsoft Power BI 支持使用机器学习模型进行数据分析。 有关详细信息，请参阅[Power BI （预览版） 中的 Azure 机器学习集成](https://docs.microsoft.com/power-bi/service-machine-learning-integration)。

## <a name="step-3-monitor-models-and-collect-data"></a>步骤 3：监视模型和收集数据

监视可以了解哪些数据发送到您的模型，并且它将返回的预测。

此信息可帮助您了解您的模型的使用方式。 收集的输入的数据也可能会很有用训练的模型的未来版本中。

有关详细信息，请参阅[如何启用模型数据收集](how-to-enable-data-collection.md)。

## <a name="step-4-update-the-deployment"></a>步骤 4：更新部署

必须显式更新部署。 有关详细信息，请参阅[部署模型](how-to-deploy-and-where.md#update)的更新部分。

## <a name="next-steps"></a>后续步骤

详细了解在使用 Azure 机器学习服务时，[部署模型的方式和位置](how-to-deploy-and-where.md)。 有关部署的示例，请参阅[教程：部署 Azure 容器实例中的图像分类模型](tutorial-deploy-models-with-aml.md)。

了解如何创建[使用部署为 Web 服务的模型](how-to-consume-web-service.md)的客户端应用程序和服务。
