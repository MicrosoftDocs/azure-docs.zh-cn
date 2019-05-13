---
title: 在 Azure Kubernetes 服务 (AKS) 中使用静态 IP 地址创建 HTTP 入口控制器
description: 了解如何在 Azure Kubernetes 服务 (AKS) 群集中使用静态 IP 地址安装和配置 NGINX 入口控制器。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/27/2019
ms.author: iainfou
ms.openlocfilehash: 57f71be436ac7632f111a7f88f9dc2d4bea608c4
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073872"
---
# <a name="create-an-ingress-controller-with-a-static-public-ip-address-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中使用静态公共 IP 地址创建入口控制器

入口控制器是一个软件片段，为 Kubernetes 服务提供反向代理、可配置的流量路由和 TLS 终止。 Kubernetes 入口资源用于配置各个 Kubernetes 服务的入口规则和路由。 借助入口控制器和入口规则，可以使用单个 IP 地址将流量路由到 Kubernetes 群集中的多个服务。

本文介绍如何在 Azure Kubernetes 服务 (AKS) 群集中部署 [NGINX 入口控制器][nginx-ingress]。 入口控制器配置了一个静态公共 IP 地址。 [cert-manager][cert-manager] 项目用于自动生成和配置 [Let's Encrypt][lets-encrypt] 证书。 最后，在 AKS 群集中运行两个应用程序（可通过单个 IP 地址访问其中的每个应用程序）。

还可以：

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [创建使用你自己的 TLS 证书的入口控制器][aks-ingress-own-tls]
- [创建一个使用 Let's Encrypt 的入口控制器，以自动生成具有动态公共 IP 地址的 TLS 证书][aks-ingress-tls]

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 门户][aks-quickstart-portal]。

本文使用 Helm 安装 NGINX 入口控制器、cert-manager 和示例 Web 应用。 你需要在 AKS 群集中初始化 Helm 并使用 Tiller 服务帐户。 请确保使用 Helm 的最新版本。 有关升级说明，请参阅 [Helm 安装文档][helm-install]。有关配置和使用 Helm 的详细信息，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用 Helm 安装应用程序][use-helm]。

这篇文章还要求运行 Azure CLI 版本 2.0.61 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="create-an-ingress-controller"></a>创建入口控制器

默认情况下，NGINX 入口控制器通过新的公共 IP 地址分配创建。 此公共 IP 地址仅对入口控制器的生命周期而言是静态的，如果删除并重新创建控制器，它将会丢失。 常见的配置要求是为 NGINX 入口控制器提供现有的静态公共 IP 地址。 如果删除入口控制器，静态公共 IP 地址仍存在。 此方法允许在应用程序的整个生命周期中以一致的方式使用现有的 DNS 记录和网络配置。

如果需要创建静态公用 IP 地址，请首先使用 [az aks show][az-aks-show] 命令获取 AKS 群集的资源组名称：

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

接下来，使用 [az network public-ip create][az-network-public-ip-create] 命令通过 *static* 分配方法创建一个公用 IP 地址。 以下示例在前面的步骤中获取的 AKS 群集资源组中创建名为 *myAKSPublicIP* 的公用 IP 地址：

```azurecli-interactive
az network public-ip create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP --allocation-method static
```

将显示的 IP 地址，如以下浓缩版输出中所示：

```json
{
  "publicIp": {
    [...]
    "ipAddress": "40.121.63.72",
    [...]
  }
}
```

现在，通过 Helm 部署 *nginx-ingress* 图表。 添加 `--set controller.service.loadBalancerIP` 参数，并指定在前面的步骤中创建的你自己的公共 IP 地址。 对于增加的冗余，NGINX 入口控制器的两个副本会在部署时具备 `--set controller.replicaCount` 参数。 若要充分利用正在运行的入口控制器副本，请确保 AKS 群集中有多个节点。

> [!TIP]
> 下面的示例创建名为的入口资源的 Kubernetes 命名空间*入口 basic*。 根据需要请指定你自己的环境的命名空间。 如果你的 AKS 群集不启用 RBAC，请添加`--set rbac.create=false`Helm 命令。

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Use Helm to deploy an NGINX ingress controller
helm install stable/nginx-ingress \
    --namespace ingress-basic \
    --set controller.service.loadBalancerIP="40.121.63.72"  \
    --set controller.replicaCount=2
```

为 NGINX 入口控制器创建 Kubernetes 负载均衡器服务时，会分配你的静态 IP 地址，如以下示例输出中所示：

```
$ kubectl get service -l app=nginx-ingress --namespace ingress-basic

NAME                                        TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
dinky-panda-nginx-ingress-controller        LoadBalancer   10.0.232.56   40.121.63.72   80:31978/TCP,443:32037/TCP   3m
dinky-panda-nginx-ingress-default-backend   ClusterIP      10.0.95.248   <none>         80/TCP                       3m
```

由于尚未创建入口规则，如果浏览到该公共 IP 地址，则会显示 NGINX 入口控制器的默认 404 页面。 入口规则是通过以下步骤配置的。

## <a name="configure-a-dns-name"></a>配置 DNS 名称

若要使 HTTPS 证书正常工作，请为入口控制器 IP 地址配置 FQDN。 使用入口控制器的 IP 地址以及要用于 FQDN 的唯一名称更新以下脚本：

```azurecli-interactive
#!/bin/bash

# Public IP address of your ingress controller
IP="40.121.63.72"

# Name to associate with public IP address
DNSNAME="demo-aks-ingress"

# Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

# Update public ip address with DNS name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
```

现在，可以通过 FQDN 访问入口控制器。

## <a name="install-cert-manager"></a>安装证书管理器

NGINX 入口控制器支持 TLS 终止。 可通过多种方法为 HTTPS 检索和配置证书。 本文演示如何使用 [cert-manager][cert-manager]，该项目提供自动 [Lets Encrypt][lets-encrypt] 证书生成和管理功能。

> [!NOTE]
> 本文为 Let's Encrypt 使用 `staging` 环境。 在生产部署中，请在安装 Helm 图表时使用资源定义中的 `letsencrypt-prod` 和 `https://acme-v02.api.letsencrypt.org/directory`。

若要在启用了 RBAC 的群集中安装 cert-manager 控制器，请使用以下 `helm install` 命令：

```console
# Install the CustomResourceDefinition resources separately
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Label the cert-manager namespace to disable resource validation
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.7.0 \
  jetstack/cert-manager
```

若要详细了解证书管理器配置，请参阅[证书管理器项目][cert-manager]。

## <a name="create-a-ca-cluster-issuer"></a>创建 CA 群集证书颁发者

证书管理器必须有 [Issuer][cert-manager-issuer] 或 [ClusterIssuer][cert-manager-cluster-issuer] 资源，才能颁发证书。 这两种 Kubernetes 资源的功能完全相同，区别在于 `Issuer` 适用于单一命名空间，而 `ClusterIssuer` 适用于所有命名空间。 有关详细信息，请参阅[证书管理器证书颁发者][cert-manager-issuer]文档。

使用以下示例清单创建群集证书颁发者，例如 `cluster-issuer.yaml`。 将电子邮件地址更新为组织提供的有效地址：

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
  namespace: ingress-basic
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: user@contoso.com
    privateKeySecretRef:
      name: letsencrypt-staging
    http01: {}
```

若要创建证书颁发者，请使用 `kubectl apply -f cluster-issuer.yaml` 命令。

```
$ kubectl apply -f cluster-issuer.yaml

clusterissuer.certmanager.k8s.io/letsencrypt-staging created
```

## <a name="run-demo-applications"></a>运行演示应用程序

入口控制器和证书管理解决方案已配置完毕。 现在让我们在你的 AKS 群集中运行两个演示应用程序。 此示例使用 Helm 来部署一个简单“Hello world”应用程序的两个实例。

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

在以下示例中，传往地址 `https://demo-aks-ingress.eastus.cloudapp.azure.com/` 的流量将路由到名为 `aks-helloworld` 的服务。 传往地址 `https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two` 的流量将路由到 `ingress-demo` 服务。 将 *hosts* 和 *host* 更新为在前面步骤中创建的 DNS 名称。

创建名为 `hello-world-ingress.yaml` 的文件，并将其复制到以下示例 YAML 中。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - demo-aks-ingress.eastus.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: demo-aks-ingress.eastus.cloudapp.azure.com
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

## <a name="create-a-certificate-object"></a>创建证书对象

接下来，必须创建证书资源。 证书资源定义了必需的 X.509 证书。 有关详细信息，请参阅 [cert-manager 证书][cert-manager-certificates]。

证书管理器可能已使用 ingress-shim（自 v0.2.2 以来随证书管理器自动部署）为你自动创建了证书对象。 有关详细信息，请参阅 [ingress-shim 文档][ingress-shim]。

若要验证证书是否已成功创建，请使用 `kubectl describe certificate tls-secret --namespace ingress-basic` 命令。

如果颁发了证书，你将看到如下输出：
```
Type    Reason          Age   From          Message
----    ------          ----  ----          -------
  Normal  CreateOrder     11m   cert-manager  Created new ACME order, attempting validation...
  Normal  DomainVerified  10m   cert-manager  Domain "demo-aks-ingress.eastus.cloudapp.azure.com" verified with "http-01" validation
  Normal  IssueCert       10m   cert-manager  Issuing certificate...
  Normal  CertObtained    10m   cert-manager  Obtained certificate from ACME server
  Normal  CertIssued      10m   cert-manager  Certificate issued successfully
```

如果需要创建其他证书资源，则使用以下示例清单来实现。 将 *dnsNames* 和 *domains* 更新为在前面步骤中创建的 DNS 名称。 如果使用仅限内部使用的入口控制器，请指定服务的内部 DNS 名称。

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: tls-secret
  namespace: ingress-basic
spec:
  secretName: tls-secret
  dnsNames:
  - demo-aks-ingress.eastus.cloudapp.azure.com
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - demo-aks-ingress.eastus.cloudapp.azure.com
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

若要创建证书资源，请使用 `kubectl apply -f certificates.yaml` 命令。

```
$ kubectl apply -f certificates.yaml

certificate.certmanager.k8s.io/tls-secret created
```

## <a name="test-the-ingress-configuration"></a>测试入口配置

打开 Web 浏览器，访问 Kubernetes 入口控制器的 FQDN，例如 *https://demo-aks-ingress.eastus.cloudapp.azure.com*。

由于这些示例使用 `letsencrypt-staging`，因此浏览器不信任颁发的 SSL 证书。 接受警告提示以继续运行应用程序。 证书信息显示这个*伪 LE 中间 X1* 证书是由 Let's Encrypt 颁发的。 此伪证书指出 `cert-manager` 正确处理了请求并接收了提供程序提供的证书：

![Let's Encrypt 临时证书](media/ingress/staging-certificate.png)

如果将 Let's Encrypt 更改为使用 `prod` 而不是 `staging`，则会使用由 Let's Encrypt 颁发的可信证书，如以下示例所示：

![Let's Encrypt 证书](media/ingress/certificate.png)

演示应用程序显示在 Web 浏览器中：

![应用程序示例 1](media/ingress/app-one.png)

现在向 FQDN 添加 */hello-world-two* 路径，例如 *https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two*。 下面显示了带自定义标题的第二个演示应用程序：

![应用程序示例 2](media/ingress/app-two.png)

## <a name="clean-up-resources"></a>清理资源

本文使用 Helm 来安装入口组件、证书和示例应用。 在部署 Helm 图表时，会创建若干 Kubernetes 资源。 这些资源包括 pod、部署和服务。 若要清理这些资源，你可以删除整个示例命名空间或单个资源。

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

或者，更精细的方法是删除创建的单个资源。 首先，删除证书资源：

```console
kubectl delete -f certificates.yaml
kubectl delete -f cluster-issuer.yaml
```

现在使用 `helm list` 命令列出 Helm 版本。 查找名为“nginx-ingress”、“cert-manager”和“aks-helloworld”的图表，如以下示例输出中所示：

```
$ helm list

NAME                    REVISION    UPDATED                     STATUS      CHART                   APP VERSION NAMESPACE
waxen-hamster           1           Wed Mar  6 23:16:00 2019    DEPLOYED    nginx-ingress-1.3.1   0.22.0        kube-system
alliterating-peacock    1           Wed Mar  6 23:17:37 2019    DEPLOYED    cert-manager-v0.6.6     v0.6.2      kube-system
mollified-armadillo     1           Wed Mar  6 23:26:04 2019    DEPLOYED    aks-helloworld-0.1.0                default
wondering-clam          1           Wed Mar  6 23:26:07 2019    DEPLOYED    aks-helloworld-0.1.0                default
```

通过 `helm delete` 命令删除发布。 以下示例删除 NGINX 入口部署、证书管理器和两个示例 AKS hello world 应用。

```
$ helm delete waxen-hamster alliterating-peacock mollified-armadillo wondering-clam

release "billowing-kitten" deleted
release "loitering-waterbuffalo" deleted
release "flabby-deer" deleted
release "linting-echidna" deleted
```

接下来，删除 AKS hello world 应用的 Helm 存储库：

```console
helm repo remove azure-samples
```

删除将流量定向到示例应用的入口路由：

```console
kubectl delete -f hello-world-ingress.yaml
```

删除其自身命名空间。 使用`kubectl delete`命令并指定命名空间名称：

```console
kubectl delete namespace ingress-basic
```

最后，删除为入口控制器创建的静态公共 IP 地址。 提供在本文第一步中获取的 MC_ 群集资源组名，例如 MC_myResourceGroup_myAKSCluster_eastus：

```azurecli-interactive
az network public-ip delete --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP
```

## <a name="next-steps"></a>后续步骤

本文包含 AKS 的一些外部组件。 若要详细了解这些组件，请参阅以下项目页面：

- [Helm CLI][helm-cli]
- [NGINX 入口控制器][nginx-ingress]
- [证书管理器][cert-manager]

还可以：

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [创建使用内部、专用网络和 IP 地址的入口控制器][aks-ingress-internal]
- [创建使用你自己的 TLS 证书的入口控制器][aks-ingress-own-tls]
- [使用动态公共 IP 创建入口控制器并配置 Let 's Encrypt 以自动生成 TLS 证书][aks-ingress-tls]

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm
[ingress-shim]: https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
