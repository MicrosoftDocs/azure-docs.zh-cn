---
title: Azure 监视器-监视 kubernetes 托管的应用零检测应用程序 |Microsoft Docs
description: 监视 Kubernetes 托管应用的零个检测应用程序是一个监视解决方案，允许您收集有关传入和传出请求到和从由在 Kubernetes 群集中运行的 pod 的 Application Insights 遥测数据使用服务网格技术称为 Istio。
services: application-insights
author: tokaplan
manager: carmonm
ms.service: application-insights
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: alkaplan
ms.openlocfilehash: 42b81ec0fa01841791a5b2651d1c1189db5e27ff
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408213"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes-hosted-apps"></a>Kubernetes 零检测应用程序监视托管应用程序

> [!IMPORTANT]
> 此功能当前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Azure 监视器现在利用服务网格技术上的 Kubernetes 群集提供了现成的框应用程序监视的任何托管的 Kubernetes 应用程序。 Application Insight 功能，例如，默认值[应用程序映射](../../azure-monitor/app/app-map.md)建模将依赖项[实时指标 Stream](../../azure-monitor/app/live-stream.md)进行实时监视，使用功能强大的可视化效果[默认仪表板](../../azure-monitor/app/overview-dashboard.md)，[指标资源管理器](../../azure-monitor/platform/metrics-getting-started.md)，和[工作簿](../../azure-monitor/app/usage-workbooks.md)。 此功能将帮助用户找出性能瓶颈和热点失败跨所有所选的 Kubernetes 命名空间内其 Kubernetes 工作负荷。 通过利用现有服务网格投资使用 Istio 等技术，Azure Monitor 启用自动检测的应用程序监视而无需对应用程序的代码进行任何修改。

> [!NOTE]
> 这是一个很多方法来执行应用程序监视在 Kubernetes 上。 您还可以开发使用托管在 Kubernetes 中的任何应用[Application Insights SDK](../../azure-monitor/azure-monitor-app-hub.md)而无需服务网格。 若要监视 Kubernetes，而不检测使用 SDK 可以使用应用程序方法的下方。

## <a name="prerequisites"></a>必备组件

- 一个[Kubernetes 群集](https://docs.microsoft.com/azure/aks/concepts-clusters-workloads)。
- 控制台到要运行的群集的访问权限*kubectl*。
- [Application Insight 资源](create-new-resource.md)
- 使用服务网格。 如果你的群集不具有 Istio 部署，你可以了解如何[安装并在 Azure Kubernetes 服务中使用 Istio](https://docs.microsoft.com/azure/aks/istio-install)。

## <a name="capabilities"></a>功能

通过使用适用于 Kubernetes 托管的应用程序监视零检测应用程序，你将能够使用：

- [应用程序映射](../../azure-monitor/app/app-map.md)
- [Live Stream 指标](../../azure-monitor/app/live-stream.md)
- [仪表板](../../azure-monitor/app/overview-dashboard.md)
- [Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md)
- [分布式跟踪](../../azure-monitor/app/distributed-tracing.md)
- [端到端事务监视](../../azure-monitor/learn/tutorial-performance.md#identify-slow-server-operations)

## <a name="installation-steps"></a>安装步骤

若要启用该解决方案，我们将执行以下步骤：
- 部署应用程序 （如果尚未部署）。
- 确保应用程序是服务网格的一部分。
- 查看收集的遥测。

### <a name="configure-your-app-to-work-with-a-service-mesh"></a>将应用配置为使用服务网格

Istio 支持两种方法的[检测 pod](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/)。
在大多数情况下，它是最简单的方法将标记包含与应用程序的 Kubernetes 命名空间*istio 注入*标签：

```console
kubectl label namespace <my-app-namespace> istio-injection=enabled
```

> [!NOTE]
> 服务网格作为数据关闭网络，因为我们无法截获加密的流量。 对于不会将留在群集的流量，使用未加密的协议 (例如，HTTP)。 必须加密的外部流量，请考虑[设置 SSL 终止](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)在入口控制器。

服务网格外部运行的应用程序不受影响。

### <a name="deploy-your-application"></a>部署应用程序

- 应用程序部署到*我的应用程序的命名空间*命名空间。 如果已部署应用程序，并且您已按照上文所述的自动挎斗注入方法，则需要重新创建 pod 以确保 Istio 注入其挎斗;要么启动滚动更新或删除单个 pod 并等待他们被重新创建。
- 确保你的应用程序符合[Istio 要求](https://istio.io/docs/setup/kubernetes/prepare/requirements/)。

### <a name="deploy-zero-instrumentation-application-monitoring-for-kubernetes-hosted-apps"></a>将监视托管的应用，适用于 Kubernetes 零检测应用程序部署

1. 下载并提取[ *Application Insights 适配器*发行](https://github.com/Microsoft/Application-Insights-Istio-Adapter/releases/)。
2. 导航到 */src/kubernetes/* release 文件夹内。
3. 编辑*application-insights-istio-mixer-adapter-deployment.yaml*
    - 值编辑*ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY*环境变量包含以包含将遥测数据在 Azure 门户中的 Application Insights 资源的检测密钥。
    - 如果需要，编辑的值*ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES*环境变量包含要为其启用监视的命名空间的以逗号分隔列表。 将其留空以监视所有命名空间。
4. 将应用*每个*YAML 文件下找到*src/kubernetes/* 运行以下命令 (仍必须位于 */src/kubernetes/*):

   ```console
   kubectl apply -f .
   ```

### <a name="verify-deployment"></a>验证部署

- 请确保已部署 Application Insights 适配器：

  ```console
  kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
  ```
> [!NOTE]
> 在某些情况下，微调优化是必需的。 若要包括或排除单个 pod 的遥测数据收集，请使用*appinsights/monitoring.enabled*该 pod 上的标签。 这会优先于所有基于命名空间的配置。 设置*appinsights/monitoring.enabled*到*true*包括 pod，并对其*false*排除它。

### <a name="view-application-insights-telemetry"></a>查看 Application Insights 遥测数据

- 生成针对应用程序以确认的监视正常运行正确的示例请求。
- 在 3 到 5 分钟内，应开始看到显示在 Azure 门户中的遥测数据。 请务必查看*应用程序映射*在门户中 Application Insights 资源部分。

## <a name="troubleshooting"></a>故障排除

下面是故障排除的流不会作为在 Azure 门户中显示遥测数据时要使用预期。

1. 请确保应用程序在负载下，并且是发送/接收在普通的 HTTP 请求。 由于遥测从线路提升，不支持加密的流量。 如果没有任何传入或传出请求，将没有遥测数据或者。
2. 确保在中提供正确的检测密钥*ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY*中的环境变量*application-insights-istio-mixer-adapter-deployment.yaml*。 在上找到的检测密钥*概述*选项卡上的 Azure 门户中的 Application Insights 资源。
3. 确保在中提供正确的 Kubernetes 命名空间*ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES*中的环境变量*application-insights-istio-mixer-adapter-deployment.yaml*。 将其留空以监视所有命名空间。
4. 请确保已按 Istio 挎斗注入应用程序的 pod。 验证 Istio 的挎斗存在上每个 pod。

   ```console
   kubectl describe pod -n <my-app-namespace> <my-app-pod-name>
   ```
   验证是否有一个名为容器*istio 代理*在 pod 上运行。

5. 查看 Application Insights 适配器的跟踪。

   ```console
   kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
   kubectl logs -n istio-system application-insights-istio-mixer-adapter-<fill in from previous command output>
   ```

   收到的遥测项的计数会更新一次一分钟的时间。 如果它不会增长分钟 1 分钟的任何遥测数据不发送到适配器的 Istio。
   查找日志中的任何错误。
6. 如果尚未建立，*适用于 Kubernetes 的 Application Insight*适配器不在馈送遥测数据，请检查 Istio 的 Mixer 日志，找出为什么它不将数据发送到适配器：

   ```console
   kubectl get pods -n istio-system -l "istio=mixer,app=telemetry"
   kubectl logs -n istio-system istio-telemetry-<fill in from previous command output> -c mixer
   ```
   查找任何错误，尤其指与之间的通信*applicationinsightsadapter*适配器。

## <a name="faq"></a>常见问题解答

此项目上的进度的最新信息，请访问[Istio Mixer 项目的 GitHub 的 Application Insights 适配器](https://github.com/Microsoft/Application-Insights-Istio-Adapter/blob/master/SETUP.md#faq)。

## <a name="uninstall"></a>卸载

若要卸载此产品，为*每个*YAML 文件下找到*src/kubernetes/* 运行：

```console
kubectl delete -f <filename.yaml>
```


## <a name="next-steps"></a>后续步骤

若要详细了解 Azure Monitor 和容器如何协作，请访问[Azure 监视的容器概述](../../azure-monitor/insights/container-insights-overview.md)