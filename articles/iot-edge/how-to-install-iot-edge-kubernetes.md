---
title: 如何在 Kubernetes 上安装 IoT Edge |Microsoft Docs
description: 了解如何使用本地开发群集环境在 Kubernetes 上安装 IoT Edge
author: kgremban
manager: philmea
ms.author: veyalla
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: 66aca7be9a2df93d846d7e78bc64c93279afc2d1
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160690"
---
# <a name="how-to-install-iot-edge-on-kubernetes-preview"></a>如何在 Kubernetes （预览版） 上安装 IoT Edge

IoT Edge 可以与 Kubernetes 使用作为可复原且高度可用的基础结构层集成。 注册 IoT Edge*自定义资源定义*(CRD) 与 Kubernetes API 服务器。 此外，它提供了*运算符*（IoT Edge 代理） 的协调云托管与本地群集状态的所需的状态。 

模块生存期管理通过 Kubernetes 计划程序，它维护模块可用性，并选择它们的位置。 IoT Edge 管理 edge 应用程序平台上，运行连续协调与边缘群集上的状态在 IoT 中心中指定所需的状态。 Edge 应用程序模型仍是基于 IoT Edge 模块和路由的熟悉模型。 IoT Edge 代理操作员执行*自动*转换为 Kubernetes 本地人构造如 pod、 部署、 服务等。

下面是高级体系结构关系图：

![kubernetes 体系结构](./media/how-to-install-iot-edge-kubernetes/k8s-arch.png)

Edge 部署的每个组件的作用域是特定于设备，从而可以共享相同的群集资源在多个边缘设备和其部署的 Kubernetes 命名空间。

>[!NOTE]
>在 Kubernetes 上的 IoT Edge 处于[公共预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="install-locally-for-a-quick-test-environment"></a>本地安装用于快速测试环境

### <a name="prerequisites"></a>必备组件

* Kubernetes 1.10 或更高版本。 如果没有现有的群集设置，则可以使用[Minikube](https://kubernetes.io/docs/setup/minikube/)针对本地群集环境。 

* [Helm](https://helm.sh/docs/using_helm/#quickstart-guide)，Kubernetes 包管理器。

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)用于查看和与群集交互。

### <a name="setup-steps"></a>设置步骤

1. 启动**Minikube**

    ``` shell
    minikube start
    ```

1. 初始化**Helm**服务器组件 (*tiller*) 在群集中

    ``` shell
    helm init
    ```

1. 将 IoT Edge 存储库添加和更新 helm 安装

    ``` shell
    helm repo add edgek8s https://edgek8s.blob.core.windows.net/helm/
    helm repo update
    ```

1. [创建 IoT 中心](../iot-hub/iot-hub-create-through-portal.md)，[注册 IoT Edge 设备](how-to-register-device-portal.md)，并记下其连接字符串。

1. Iotedged 和 IoT Edge 代理安装到群集

    ```shell
    helm install \
    --name k8s-edge1 \
    --set "deviceConnectionString=replace-with-device-connection-string" \
    edgek8s/edge-kubernetes
    ```
1. 在浏览器中打开 Kubernetes 仪表板

    ```shell
    minikube dashboard
    ```

    在群集的命名空间，你会看到一个用于以下约定的 IoT Edge 设备*msiot-\<iothub 名称 >-\<edgedevice 名称 >*。 IoT Edge 代理和 iotedged pod 应启动并运行此命名空间中。

1. 添加模拟的温度传感器模块中的步骤[将模块部署](quickstart-linux.md#deploy-a-module)部分的快速入门。 IoT Edge 模块管理可从 IoT 中心门户，就像任何其他 IoT Edge 设备。 不建议通过 Kubernetes 的工具，模块配置进行本地更改，因为它们可能会被覆盖。

1. 几秒钟后刷新**Pod**边缘设备命名空间下的仪表板中的页将列出 IoT Edge 中心和 IoT Edge 中心运行的 as 的模拟的传感器 pod pod 数据引入到 IoT 中心。

## <a name="clean-up-resources"></a>清理资源

若要删除创建的边缘部署所有资源，请使用上一节的步骤 5 中使用的名称使用以下命令。

``` shell
helm delete --purge k8s-edge1
```

## <a name="next-steps"></a>后续步骤

### <a name="deploy-as-a-highly-available-edge-gateway"></a>将部署为高度可用的 edge 网关 

中的 Kubernetes 群集的边缘设备可以用作下游设备的 IoT 网关。 可以配置为能够弹性应对节点故障，从而提供到边缘部署的高可用性。 请参阅此[的详细的演练](https://github.com/Azure-Samples/iotedge-gateway-on-kubernetes)若要在此方案中使用 IoT Edge。