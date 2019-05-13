---
title: 介绍运行状况集成推出到 Azure 部署管理器
description: 介绍如何使用 Azure 部署管理器在多个区域部署服务。 其中介绍了安全部署实践，可在部署到所有区域前验证部署的稳定性。
services: azure-resource-manager
documentationcenter: na
author: mumian
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/19
ms.author: jgao
ms.openlocfilehash: 1006cc902cf4b6f763d86165a039a1fec5dc97a1
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467089"
---
# <a name="introduce-health-integration-rollout-to-azure-deployment-manager-public-preview"></a>介绍运行状况集成推出到 Azure 部署管理器 （公共预览版）

[Azure 的 Deployment Manager](./deployment-manager-overview.md)允许您执行分步的推出的 Azure 资源管理器资源。 资源的有序的方式部署区域的区域。 集成的运行状况检查的 Azure 部署管理器可以监视首次推出，并自动停止有问题的首次推出，以便可以进行故障排除并减少影响的小数位数。 此功能可以减少服务不可用而引起的更新中的回归。

## <a name="health-monitoring-providers"></a>运行状况监视提供程序

为了使运行状况集成以尽可能简化，Microsoft 一直致力于一些前几个服务运行状况监视公司为你提供一个简单的复制/粘贴解决方案，可以将运行状况检查与你的部署相集成。 如果尚不存在使用运行状况监视器，这些开始是很好的解决方案：

| ![azure 部署管理器运行状况监视提供程序 datadog](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-datadog.svg) | ![azure 部署管理器运行状况监视提供程序 site24x7](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-site24x7.svg) | ![azure 部署管理器运行状况监视提供程序 wavefront](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-wavefront.svg) |
|-----|------|------|
|Datadog、 前导监视和分析平台的现代云环境。 请参阅[Datadog 如何集成与 Azure 部署管理器](https://www.datadoghq.com/azure-deployment-manager/)。|Site24x7，-一体私有云和公有云服务监视解决方案。 请参阅[Site24x7 如何集成与 Azure 部署管理器](https://www.site24x7.com/azure/adm.html)。| Wavefront，多云应用程序环境的监视和分析平台。 请参阅[Wavefront 如何集成与 Azure 部署管理器](https://go.wavefront.com/wavefront-adm/)。|

## <a name="how-service-health-is-determined"></a>如何确定服务运行状况

[运行状况监视提供程序](#health-monitoring-providers)提供多种机制用于监视服务和警报的任何服务运行状况问题。 [Azure 监视器](/services/monitor/)是一个此类产品的一个示例。 可以使用 azure 监视器超出特定阈值时创建警报。 例如，内存和 CPU 使用率达到峰值超过预期的级别时将新的更新部署到你的服务。 当系统通知，您可以采取纠正措施。

这些运行状况提供商通常提供 REST Api，以便可以以编程方式检查服务的监视器的状态。 REST Api 可以既返回简单的正常运行/不正常信号 （由 HTTP 响应代码），和/或接收信号的详细信息。

新*healthCheck*步骤在 Azure 部署管理器允许您声明指示运行状况服务的 HTTP 代码，或为更复杂的其余部分结果，您甚至可以指定正则表达式，如果它们匹配，指示正常响应。

获取与 Azure 部署管理器运行状况检查安装程序流：

1. 创建运行状况监视器通过所选的运行状况服务提供商。
1. 在 Azure 部署管理器推出此功能的一部分创建一个或多个运行状况检查步骤。 填写以下信息的运行状况检查步骤：

    1. 运行状况监视器 （如定义的运行状况服务提供商） REST API 的 URI。
    1. 身份验证信息。 目前支持仅 API 密钥样式身份验证。
    1. [HTTP 状态代码](https://www.wikipedia.org/wiki/List_of_HTTP_status_codes)或定义正常响应的正则表达式。 请注意，可能会提供正则表达式，所有必须匹配项的响应被视为正常运行，或者你可能会都提供的表达式的中，任何必须匹配要被视为正常运行的响应。 支持这两种方法。

    以下 Json 是一个示例：

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

1. 在 Azure 部署管理器部署中的适当时间调用的运行状况检查步骤。 在以下示例中，运行状况检查步骤中调用**postDeploymentSteps**的**stepGroup2**。

    ```json
    "stepGroups": [
        {
            "name": "stepGroup1",
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup2",
            "dependsOnStepGroups": ["stepGroup1"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
            "postDeploymentSteps": [
                {
                    "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
                }
            ]
        },
        {
            "name": "stepGroup3",
            "dependsOnStepGroups": ["stepGroup2"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup4",
            "dependsOnStepGroups": ["stepGroup3"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
            "postDeploymentSteps": []
        }
    ]
    ```

若要演示一个示例，请参阅[教程：使用 Azure 部署管理器中的运行状况检查](./deployment-manager-health-check.md)。

## <a name="phases-of-a-health-check"></a>运行状况检查的阶段

此时 Azure 部署管理器知道如何为你的服务并在什么阶段在推出此功能来执行此操作时的运行状况查询。 但是，Azure 部署管理器还允许进行深入配置这些检查的时间。 在 3 个连续的阶段，所有这些具有可配置的持续时间中执行运行状况检查步骤： 

1. 等待

    1. 部署操作完成后，可能会重新启动 Vm，重新配置基于新数据，或甚至第一次启动。 它还花时间让服务开始发出运行状况信号由监视有用的功能，提供程序的运行状况聚合。 在此过程充满争议，它可能未利用有意义的服务运行状况检查，因为更新尚未达到稳定状态。 实际上，该服务可能剧烈波动正常和不正常状态之间按资源结清。 
    1. 在等待阶段中，不会监视服务运行状况。 这用于允许已部署的资源开始运行状况检查过程之前制作的时间。 
1. Elastic

    1. 由于它是不可能知道在所有情况下多长时间的资源所需制作趋于稳定，弹性阶段允许灵活的时间段之间可能不稳定资源时之前，并在需要维护正常稳定时状态。
    1. 弹性阶段开始时，Azure 部署管理器将开始定期轮询服务运行状况提供的 REST 终结点。 可配置轮询间隔。 
    1. 如果运行状况监视器返回的信号，该值指示服务不正常，将忽略这些信号，弹性阶段将继续，并将继续轮询。 
    1. 只要运行状况监视器的信号，该值指示服务处于正常状态返回后，弹性阶段结束和 HealthyState 阶段开始。 
    1. 因此，弹性阶段指定的持续时间是时间的最大可以轮询服务运行状况正常的响应被视为必需之前所用量。 
1. HealthyState

    1. 在 HealthyState 阶段中，服务运行状况持续轮询一次弹性阶段间隔。 
    1. 该服务应从运行状况监视提供程序的正常信号维护整个指定的持续时间。 
    1. 如果任何时候，检测到不正常的响应时，Azure 部署管理器将停止整个推出，并返回 REST 响应传输不正常的服务信号。
    1. HealthyState 持续时间结束后运行状况检查已完成，并且部署将继续到下一步。

## <a name="next-steps"></a>后续步骤

在本文中，您学习了如何将运行状况监视 Azure 部署管理器中进行集成。 请转到下一篇文章，了解如何使用部署管理器进行部署。

> [!div class="nextstepaction"]
> [教程： 将集成运行状况检查在 Azure 部署管理器](./deployment-manager-tutorial-health-check.md)