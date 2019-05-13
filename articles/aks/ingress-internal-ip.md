---
title: 在 Azure Kubernetes 服务 (AKS) 中创建适用于内部网络的入口控制器
description: 了解如何在 Azure Kubernetes 服务 (AKS) 群集中安装和配置适用于内部专用网络的 NGINX 入口控制器。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/27/2019
ms.author: iainfou
ms.openlocfilehash: 4a648bd2704e93abedeefae14aee66ae8bfeecef
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073900"
---
# <a name="create-an-ingress-controller-to-an-internal-virtual-network-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中创建内部虚拟网络的入口控制器

入口控制器是一个软件片段，为 Kubernetes 服务提供反向代理、可配置的流量路由和 TLS 终止。 Kubernetes 入口资源用于配置各个 Kubernetes 服务的入口规则和路由。 借助入口控制器和入口规则，可以使用单个 IP 地址将流量路由到 Kubernetes 群集中的多个服务。

本文介绍如何在 Azure Kubernetes 服务 (AKS) 群集中部署 [NGINX 入口控制器][nginx-ingress]。 入口控制器在内部专用虚拟网络和 IP 地址上配置。 不允许外部访问。 然后在 AKS 群集中运行两个应用程序（可通过单个 IP 地址访问其中的每个应用程序）。

还可以：

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [创建使用你自己的 TLS 证书的入口控制器][aks-ingress-own-tls]
- 创建一个使用 Let's Encrypt 的入口控制器，以自动生成[具有动态公共 IP 地址][aks-ingress-tls]或[具有静态公共 IP 地址][aks-ingress-static-tls]的 TLS 证书

## <a name="before-you-begin"></a>开始之前

本文使用 Helm 安装 NGINX 入口控制器、cert-manager 和示例 Web 应用。 你需要在 AKS 群集中初始化 Helm 并使用 Tiller 服务帐户。 有关配置和使用 Helm 的详细信息，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用 Helm 安装应用程序][use-helm]。

这篇文章还要求运行 Azure CLI 版本 2.0.61 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="create-an-ingress-controller"></a>创建入口控制器

默认情况下，NGINX 入口控制器通过动态公共 IP 地址分配创建。 常见的配置要求是使用内部专用网络和 IP 地址。 此方法仅限内部用户访问你的服务，不允许外部访问。

根据以下示例清单文件创建名为 *internal-ingress.yaml* 的文件。 此示例将 *10.240.0.42* 分配给 *loadBalancerIP* 资源。 提供自己的可以与入口控制器配合使用的内部 IP 地址。 请确保该 IP 地址尚未在虚拟网络中使用。

```yaml
controller:
  service:
    loadBalancerIP: 10.240.0.42
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

现在，通过 Helm 部署 *nginx-ingress* 图表。 若要使用在上一步创建的清单文件，请添加 `-f internal-ingress.yaml` 参数。 对于增加的冗余，NGINX 入口控制器的两个副本会在部署时具备 `--set controller.replicaCount` 参数。 若要充分利用正在运行的入口控制器副本，请确保 AKS 群集中有多个节点。

> [!TIP]
> 下面的示例创建名为的入口资源的 Kubernetes 命名空间*入口 basic*。 根据需要请指定你自己的环境的命名空间。 如果你的 AKS 群集不启用 RBAC，请添加`--set rbac.create=false`Helm 命令。

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Use Helm to deploy an NGINX ingress controller
helm install stable/nginx-ingress \
    --namespace ingress-basic \
    -f internal-ingress.yaml \
    --set controller.replicaCount=2
```

为 NGINX 入口控制器创建 Kubernetes 负载均衡器服务时，会分配你的内部 IP 地址，如以下示例输出中所示：

```
$ kubectl get service -l app=nginx-ingress --namespace ingress-basic

NAME                                              TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
alternating-coral-nginx-ingress-controller        LoadBalancer   10.0.97.109   10.240.0.42   80:31507/TCP,443:30707/TCP   1m
alternating-coral-nginx-ingress-default-backend   ClusterIP      10.0.134.66   <none>        80/TCP                       1m
```

由于尚未创建入口规则，如果浏览到该内部 IP 地址，则会显示 NGINX 入口控制器的默认 404 页面。 入口规则是通过以下步骤配置的。

## <a name="run-demo-applications"></a>运行演示应用程序

若要查看运行中的入口控制器，请在 AKS 群集中运行两个演示应用程序。 此示例使用 Helm 来部署一个简单“Hello world”应用程序的两个实例。

需按下面所示将 Azure 示例存储库添加到 Helm 环境中，才能安装示例 Helm 图表：

```console
helm repo add azure-samples https://azure-samples.github.io/helm-charts/
```

使用以下命令根据 Helm 图表创建第一个演示应用程序：

```console
helm install azure-samples/aks-helloworld --namespace ingress-basic
```

现在安装演示应用程序的第二个实例。 对于第二个实例，请指定新的标题，使两个应用程序在视觉上不同。 还需要指定唯一的服务名称：

```console
helm install azure-samples/aks-helloworld \
    --namespace ingress-basic \
    --set title="AKS Ingress Demo" \
    --set serviceName="ingress-demo"
```

## <a name="create-an-ingress-route"></a>创建入口路由

两个应用程序现在都在 Kubernetes 群集中运行。 若要将流量路由到每个应用程序，请创建 Kubernetes 入口资源。 该入口资源配置将流量路由到这两个应用程序之一的规则。

在以下示例中，传往地址 `http://10.240.0.42/` 的流量将路由到名为 `aks-helloworld` 的服务。 传往地址 `http://10.240.0.42/hello-world-two` 的流量将路由到 `ingress-demo` 服务。

创建名为 `hello-world-ingress.yaml` 的文件，并将其复制到以下示例 YAML 中。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: aks-helloworld
          servicePort: 80
      - path: /hello-world-two
        backend:
          serviceName: ingress-demo
          servicePort: 80
```

使用 `kubectl apply -f hello-world-ingress.yaml` 命令创建入口资源。

```
$ kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
```

## <a name="test-the-ingress-controller"></a>测试入口控制器

若要测试入口控制器的路由，请通过一个 Web 客户端浏览到两个应用程序。 可以根据需要通过 AKS 群集上的 Pod 快速测试这项仅限内部使用的功能。 创建一个测试 Pod，并在其上附加一个终端会话：

```console
kubectl run -it --rm aks-ingress-test --image=debian --namespace ingress-basic
```

使用 `apt-get` 在 Pod 中安装 `curl`：

```console
apt-get update && apt-get install -y curl
```

现在，请使用 `curl` 访问 Kubernetes 入口控制器的地址，例如 *http://10.240.0.42*。 提供自己的内部 IP 地址，该地址是在本文第一步中部署入口控制器时指定的。

```console
curl -L http://10.240.0.42
```

未为此地址提供其他路径，因此入口控制器默认为 */* 路由。 第一个演示应用程序已返回，如以下精简版示例输出中所示：

```
$ curl -L 10.240.0.42

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>Welcome to Azure Kubernetes Service (AKS)</title>
[...]
```

现在向地址添加 */hello-world-two* 路径，例如 *http://10.240.0.42/hello-world-two*。 第二个使用自定义标题的演示应用程序已返回，如以下精简版示例输出中所示：

```
$ curl -L -k http://10.240.0.42/hello-world-two

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>AKS Ingress Demo</title>
[...]
```

## <a name="clean-up-resources"></a>清理资源

本文使用了 Helm 来安装入口组件和示例应用。 在部署 Helm 图表时，会创建若干 Kubernetes 资源。 这些资源包括 pod、部署和服务。 若要清理这些资源，你可以删除整个示例命名空间或单个资源。

### <a name="delete-the-sample-namespace-and-all-resources"></a>删除示例命名空间和所有资源

若要删除整个示例命名空间，请使用`kubectl delete`命令并指定命名空间名称。 会删除命名空间中的所有资源。

```console
kubectl delete namespace ingress-basic
```

然后，删除 AKS 你好 world 应用的 Helm 存储库：

```console
helm repo remove azure-samples
```

### <a name="delete-resources-individually"></a>逐个删除资源

或者，更精细的方法是删除创建的单个资源。 Helm 中发布的列表`helm list`命令。 查找名为“nginx-ingress”和“aks-helloworld”的图表，如以下示例输出中所示：

```
$ helm list

NAME                REVISION    UPDATED                     STATUS      CHART                   APP VERSION NAMESPACE
kissing-ferret      1           Tue Oct 16 17:13:39 2018    DEPLOYED    nginx-ingress-0.22.1    0.15.0      kube-system
intended-lemur      1           Tue Oct 16 17:20:59 2018    DEPLOYED    aks-helloworld-0.1.0                default
pioneering-wombat   1           Tue Oct 16 17:21:05 2018    DEPLOYED    aks-helloworld-0.1.0                default
```

通过 `helm delete` 命令删除发布。 以下示例删除 NGINX 入口部署和两个示例 AKS hello world 应用。

```
$ helm delete kissing-ferret intended-lemur pioneering-wombat

release "kissing-ferret" deleted
release "intended-lemur" deleted
release "pioneering-wombat" deleted
```

接下来，删除 AKS hello world 应用的 Helm 存储库：

```console
helm repo remove azure-samples
```

删除将流量定向到示例应用的入口路由：

```console
kubectl delete -f hello-world-ingress.yaml
```

最后，您可以删除其自身提供命名空间。 使用`kubectl delete`命令并指定命名空间名称：

```console
kubectl delete namespace ingress-basic
```

## <a name="next-steps"></a>后续步骤

本文包含 AKS 的一些外部组件。 若要详细了解这些组件，请参阅以下项目页面：

- [Helm CLI][helm-cli]
- [NGINX 入口控制器][nginx-ingress]

还可以：

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [使用动态公共 IP 创建入口控制器并配置 Let 's Encrypt 以自动生成 TLS 证书][aks-ingress-tls]
- [使用静态公共 IP 地址创建入口控制器并配置 Let 's Encrypt 以自动生成 TLS 证书][aks-ingress-static-tls]

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
