---
title: 基于使用 KEDA Kubernetes 的 azure 函数
description: 了解如何在云中的 Kubernetes 中运行 Azure Functions 或本地使用 KEDA，基于 Kubernetes 的事件驱动的自动缩放。
services: functions
documentationcenter: na
author: jeffhollan
manager: jeconnoc
keywords: azure functions、 函数、 事件处理、 动态计算，无服务器体系结构、 kubernetes
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: jehollan
ms.openlocfilehash: c82ed7aa841f53f5c81f3281ed1b09926e565e75
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077615"
---
# <a name="azure-functions-on-kubernetes-with-keda"></a>基于使用 KEDA Kubernetes 的 azure 函数

Azure Functions 运行时提供的灵活地托管所需位置和方式。  [KEDA](https://github.com/kedacore/kore) （基于 Kubernetes 的事件驱动的自动缩放） 对无缝使用 Azure Functions 运行时和工具提供在 Kubernetes 中的事件驱动的缩放。

## <a name="how-kubernetes-based-functions-work"></a>如何基于 Kubernetes 的工作函数

Azure Functions 服务由两个关键组件组成： 一个运行时和缩放控制器。  Functions 运行时运行，并执行你的代码。  运行时包括有关如何触发、 登录，以及管理函数执行逻辑。  另一个组件是缩放控制器。  缩放控制器监视以函数中，为目标的事件的速率和主动运行您的应用程序的实例的数量进行缩放。  若要了解详细信息，请参阅[Azure Functions 的缩放和托管](functions-scale.md)。

基于 Kubernetes 的 Functions 提供的函数运行时[Docker 容器](functions-create-function-linux-custom-image.md)KEDA 通过事件驱动缩放。  对 0 实例 （如果不发生任何事件） 和最多可以降低 KEDA *n*实例。 通过公开 Kubernetes 自动缩放程序 （水平 Pod 自动缩放程序） 用于自定义指标来执行此操作。  函数容器使用 KEDA 使可能复制任何 Kubernetes 群集中的无服务器函数功能。  此外可以使用部署这些函数[Azure Kubernetes 服务 (AKS) 的虚拟节点](../aks/virtual-nodes-cli.md)无服务器基础结构的功能。

## <a name="managing-keda-and-functions-in-kubernetes"></a>管理 KEDA 和在 Kubernetes 中的函数

若要在 Kubernetes 群集上运行函数，则必须安装 KEDA 组件。 您可以将安装此组件使用[Azure Functions Core Tools](functions-run-local.md)。

### <a name="installing-with-the-azure-functions-core-tools"></a>安装与 Azure Functions Core Tools

默认情况下，Core Tools 将安装 KEDA 和 Osiris 组件，支持事件驱动和 HTTP 缩放，分别。  安装使用`kubectl`当前上下文中运行。

通过运行以下安装命令，在群集中安装 KEDA:

```cli
func kubernetes install --namespace keda
```

## <a name="deploying-a-function-app-to-kubernetes"></a>将函数应用部署到 Kubernetes

可以将任何函数应用部署到运行 KEDA 的 Kubernetes 群集。  在 Docker 容器中运行你的函数，因为你的项目需要`Dockerfile`。  如果它尚不会无订阅，可以添加 Dockerfile，通过 Functions 项目的根目录运行以下命令：

```cli
func init --docker-only
```

若要生成的映像并将函数部署到 Kubernetes，运行以下命令：

```cli
func kubernetes deploy --name <name-of-function-deployment> --registry <container-registry-username>
```

> 将 `<name-of-function-deployment>` 替换为你的函数应用的名称。

这将创建 Kubernetes`Deployment`资源，`ScaledObject`资源，并`Secrets`，其中包括环境变量从导入你`local.settings.json`文件。

## <a name="removing-a-function-app-from-kubernetes"></a>从 Kubernetes 中删除的函数应用

部署您可以通过删除关联删除函数后`Deployment`， `ScaledObject`、`Secrets`创建。

```cli
kubectl delete deploy <name-of-function-deployment>
kubectl delete ScaledObject <name-of-function-deployment>
kubectl delete secret <name-of-function-deployment>
```

## <a name="uninstalling-keda-from-kubernetes"></a>在 Kubernetes 中卸载 KEDA

可以运行以下的核心工具命令，以从 Kubernetes 群集中删除 KEDA:

```cli
func kubernetes remove --namespace keda
```

## <a name="supported-triggers-in-keda"></a>KEDA 中受支持的触发器

KEDA 当前为 beta 版本，支持以下 Azure 函数触发器：

* [Azure 存储队列](functions-bindings-storage-queue.md)
* [Azure Service Bus 队列](functions-bindings-service-bus.md)
* [HTTP](functions-bindings-http-webhook.md)
* [Apache Kafka](https://github.com/azure/azure-functions-kafka-extension)

## <a name="next-steps"></a>后续步骤
有关详细信息，请参阅以下资源：

* [创建使用自定义映像的函数](functions-create-function-linux-custom-image.md)
* [在本地对 Azure Functions 进行编码和测试](functions-develop-local.md)
* [Azure 函数消耗计划的工作原理](functions-scale.md)