---
title: 使用自己的 TLS 证书在 Azure Kubernetes 服务 (AKS) 群集中进行入口路由
description: 了解如何在 Azure Kubernetes 服务 (AKS) 群集中安装和配置使用你自己的证书的 NGINX 入口控制器。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/27/2019
ms.author: iainfou
ms.openlocfilehash: e20f881d740c5d5b73c23c933ceb3d6f19e78ef9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073843"
---
# <a name="create-an-https-ingress-controller-and-use-your-own-tls-certificates-on-azure-kubernetes-service-aks"></a>创建 HTTPS 入口控制器并在 Azure Kubernetes 服务 (AKS) 中使用自己的 TLS 证书

入口控制器是一个软件片段，为 Kubernetes 服务提供反向代理、可配置的流量路由和 TLS 终止。 Kubernetes 入口资源用于配置各个 Kubernetes 服务的入口规则和路由。 借助入口控制器和入口规则，可以使用单个 IP 地址将流量路由到 Kubernetes 群集中的多个服务。

本文介绍如何在 Azure Kubernetes 服务 (AKS) 群集中部署 [NGINX 入口控制器][nginx-ingress]。 生成自己的证书，并创建用于入口路由的 Kubernetes 机密。 最后，在 AKS 群集中运行两个应用程序（可通过单个 IP 地址访问其中的每个应用程序）。

还可以：

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [创建使用内部、专用网络和 IP 地址的入口控制器][aks-ingress-internal]
- 创建一个使用 Let's Encrypt 的入口控制器，以自动生成[具有动态公共 IP 地址][aks-ingress-tls]或[具有静态公共 IP 地址][aks-ingress-static-tls]的 TLS 证书

## <a name="before-you-begin"></a>开始之前

本文使用 Helm 安装 NGINX 入口控制器和示例 Web 应用。 你需要在 AKS 群集中初始化 Helm 并使用 Tiller 服务帐户。 请确保使用 Helm 的最新版本。 有关升级说明，请参阅 [Helm 安装文档][helm-install]。有关配置和使用 Helm 的详细信息，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用 Helm 安装应用程序][use-helm]。

这篇文章还要求运行 Azure CLI 版本 2.0.61 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="create-an-ingress-controller"></a>创建入口控制器

若要创建入口控制器，请使用 `Helm` 来安装 *nginx-ingress*。 对于增加的冗余，NGINX 入口控制器的两个副本会在部署时具备 `--set controller.replicaCount` 参数。 若要充分利用正在运行的入口控制器副本，请确保 AKS 群集中有多个节点。

> [!TIP]
> 下面的示例创建名为的入口资源的 Kubernetes 命名空间*入口 basic*。 根据需要请指定你自己的环境的命名空间。 如果你的 AKS 群集不启用 RBAC，请添加`--set rbac.create=false`Helm 命令。

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Use Helm to deploy an NGINX ingress controller
helm install stable/nginx-ingress --namespace ingress-basic --set controller.replicaCount=2
```

在安装过程中，将为入口控制器创建一个 Azure 公共 IP 地址。 此公共 IP 地址在入口控制器的寿命期内是静态的。 如果你删除入口控制器，则公共 IP 地址分配会丢失。 如果你然后创建了另外的入口控制器，则会分配新的公共 IP 地址。 如果希望保持使用此公共 IP 地址，则可以改为[创建具有静态公共 IP 地址的入口控制器][aks-ingress-static-tls]。

若要获取公共 IP 地址，请使用 `kubectl get service` 命令。 将 IP 地址分配给服务需要几分钟时间。

```
$ kubectl get service -l app=nginx-ingress --namespace ingress-basic

NAME                                          TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
virulent-seal-nginx-ingress-controller        LoadBalancer   10.0.48.240   40.87.46.190   80:31159/TCP,443:30657/TCP   7m
virulent-seal-nginx-ingress-default-backend   ClusterIP      10.0.50.5     <none>         80/TCP                       7m
```

请记下此公共 IP 地址，因为在最后一个步骤中测试部署时需要用到。

尚未创建入口规则。 如果浏览到公共 IP 地址，将显示 NGINX 入口控制器的默认 404 页面。

## <a name="generate-tls-certificates"></a>生成 TLS 证书

在本文中，我们将生成使用 `openssl` 的自签名证书。 对于生产用途，应该通过提供商或你自己的证书颁发机构 (CA) 请求受信任的已签名证书。 下一步骤将使用 TLS 证书和 OpenSSL 生成的私钥来生成 Kubernetes 机密。

以下示例生成有效期为 365 天的、名为 *aks-ingress-tls.crt* 的 2048 位 RSA X509 证书。 私钥文件名为 *aks-ingress-tls.key*。 Kubernetes TLS 机密需要这两个文件。

本文使用 *demo.azure.com* 使用者公用名，无需进行更改。 对于生产用途，请为 `-subj` 参数指定自己的组织值：

```console
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -out aks-ingress-tls.crt \
    -keyout aks-ingress-tls.key \
    -subj "/CN=demo.azure.com/O=aks-ingress-tls"
```

## <a name="create-kubernetes-secret-for-the-tls-certificate"></a>创建 TLS 证书的 Kubernetes 机密

若使 Kubernetes 能够对入口控制器使用 TLS 证书和私钥，需要创建并使用一个机密。 机密只需定义一次，它使用上一步骤中创建的证书和密钥文件。 然后，可以在定义入口路由时引用此机密。

以下示例创建名为 *aks-ingress-tls* 的机密：

```console
kubectl create secret tls aks-ingress-tls \
    --namespace ingress-basic \
    --key aks-ingress-tls.key \
    --cert aks-ingress-tls.crt
```

## <a name="run-demo-applications"></a>运行演示应用程序

已配置入口控制器和包含你的证书的机密。 现在让我们在你的 AKS 群集中运行两个演示应用程序。 此示例使用 Helm 来部署一个简单“Hello world”应用程序的两个实例。

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

两个应用程序现在都在 Kubernetes 群集中运行，但它们配置了服务类型 `ClusterIP`。 因此，无法通过 Internet 访问它们。 若要公开发布这两个应用程序，请创建 Kubernetes 入口资源。 该入口资源配置将流量路由到这两个应用程序之一的规则。

在以下示例中，传往地址 `https://demo.azure.com/` 的流量将路由到名为 `aks-helloworld` 的服务。 传往地址 `https://demo.azure.com/hello-world-two` 的流量将路由到 `ingress-demo` 服务。 在本文中，无需更改这些演示主机名。 对于生产用途，请提供在证书请求和生成过程中指定的名称。

> [!TIP]
> 如果在证书请求过程中指定的主机名（CN 名称）与在入口路由中定义的主机不匹配，则入口控制器将显示“Kubernetes 入口控制器虚构证书”。 请确保证书和入口路由主机名匹配。

*tls* 节告知入口路由要对主机 *demo.azure.com* 使用名为 *aks-ingress-tls* 的机密。 同样，对于生产用途，请指定自己的主机地址。

创建名为 `hello-world-ingress.yaml` 的文件，并将其复制到以下示例 YAML 中。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - demo.azure.com
    secretName: aks-ingress-tls
  rules:
  - host: demo.azure.com
    http:
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

## <a name="test-the-ingress-configuration"></a>测试入口配置

若要使用虚构的 *demo.azure.com* 主机测试证书，请使用 `curl` 并指定 *--resolve* 参数。 此参数告知要将 *demo.azure.com* 名称映射到入口控制器的公共 IP 地址。 请按以下示例中所示，指定自己的入口控制器的公共 IP 地址：

```
curl -v -k --resolve demo.azure.com:443:40.87.46.190 https://demo.azure.com
```

未为此地址提供其他路径，因此入口控制器默认为 */* 路由。 第一个演示应用程序已返回，如以下精简版示例输出中所示：

```
$ curl -v -k --resolve demo.azure.com:443:40.87.46.190 https://demo.azure.com

[...]
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>Welcome to Azure Kubernetes Service (AKS)</title>
[...]
```

`curl` 命令中的 *-v* 参数输出详细信息，包括收到的 TLS 证书。 在输出 curl 结果的中途，可以验证是否使用了你自己的 TLS 证书。 即使使用的是自签名证书，*-k* 参数也会继续加载页面。 以下示例显示了使用“颁发者:CN=demo.azure.com; O=aks-ingress-tls”证书：

```
[...]
* Server certificate:
*  subject: CN=demo.azure.com; O=aks-ingress-tls
*  start date: Oct 22 22:13:54 2018 GMT
*  expire date: Oct 22 22:13:54 2019 GMT
*  issuer: CN=demo.azure.com; O=aks-ingress-tls
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
[...]
```

现在向地址添加“/hello-world-two”路径，例如 `https://demo.azure.com/hello-world-two`。 第二个使用自定义标题的演示应用程序已返回，如以下精简版示例输出中所示：

```
$ curl -v -k --resolve demo.azure.com:443:137.117.36.18 https://demo.azure.com/hello-world-two

[...]
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

NAME            REVISION    UPDATED                     STATUS      CHART                   APP VERSION NAMESPACE
virulent-seal   1           Tue Oct 23 16:37:24 2018    DEPLOYED    nginx-ingress-0.22.1    0.15.0      kube-system
billowing-guppy 1           Tue Oct 23 16:41:38 2018    DEPLOYED    aks-helloworld-0.1.0                default
listless-quokka 1           Tue Oct 23 16:41:30 2018    DEPLOYED    aks-helloworld-0.1.0                default
```

通过 `helm delete` 命令删除发布。 以下示例删除 NGINX 入口部署和两个示例 AKS hello world 应用。

```
$ helm delete virulent-seal billowing-guppy listless-quokka

release "virulent-seal" deleted
release "billowing-guppy" deleted
release "listless-quokka" deleted
```

接下来，删除 AKS hello world 应用的 Helm 存储库：

```console
helm repo remove azure-samples
```

删除将流量定向到示例应用的入口路由：

```console
kubectl delete -f hello-world-ingress.yaml
```

删除证书密码：

```console
kubectl delete secret aks-ingress-tls
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
- [创建使用内部、专用网络和 IP 地址的入口控制器][aks-ingress-internal]
- 创建一个使用 Let's Encrypt 的入口控制器，以自动生成[具有动态公共 IP 地址][aks-ingress-tls]或[具有静态公共 IP 地址][aks-ingress-static-tls]的 TLS 证书

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-tls]: ingress-tls.md
