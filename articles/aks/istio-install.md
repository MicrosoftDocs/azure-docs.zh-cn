---
title: 在 Azure Kubernetes 服务 (AKS) 中安装 Istio
description: 了解如何在 Azure Kubernetes 服务 (AKS) 群集中安装和使用 Istio 来创建服务网格
services: container-service
author: paulbouwer
ms.service: container-service
ms.topic: article
ms.date: 04/19/2019
ms.author: pabouwer
ms.openlocfilehash: fc95ce4aad4e8597b02b9c862be33bfcf6185541
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073802"
---
# <a name="install-and-use-istio-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中安装和使用 Istio

[Istio][istio-github] 是跨 Kubernetes 群集中的微服务提供关键功能集的开源服务网格。 这些功能包括流量管理、服务标识和安全性、策略执行以及可观察性。 有关 Istio 的详细信息，请参阅官方文档[什么是 Istio？][istio-docs-concepts]。

本文介绍如何安装 Istio。 Istio`istioctl`到客户端计算机上安装客户端二进制和 Istio 组件安装到 AKS 上的 Kubernetes 群集。

> [!NOTE]
> 这些指令引用 Istio 版本`1.1.3`。
>
> Istio `1.1.x` Istio 团队针对 Kubernetes 版本进行了测试版本`1.11`， `1.12`， `1.13`。 你可以找到其他 Istio 版本在[GitHub-Istio 释放][ istio-github-releases]以及有关每个在版本信息[Istio-发行说明][ istio-release-notes].

在本文中，学习如何：

> [!div class="checklist"]
> * 下载 Istio
> * 安装二进制 Istio istioctl 客户端
> * 在 AKS 上安装 Istio CRDs
> * Istio 组件安装在 AKS 上
> * 验证 Istio 安装
> * 访问加载项
> * 从 AKS 卸载 Istio

## <a name="before-you-begin"></a>开始之前

这篇文章中详述的步骤假定你已创建 AKS 群集 (Kubernetes`1.11`和更高版本，使用 RBAC 启用) 并已建立了`kubectl`与群集的连接。 如果需要帮助完成这些项目，请参阅 [AKS 快速入门][aks-quickstart]。

你将需要[Helm] [ helm]按照这些说明进行操作并安装 Istio。 建议具有版本`2.12.2`或更高版本正确安装和配置在群集中。 如果需要安装 Helm 的帮助，请参阅[AKS Helm 安装指南][helm-install]。 此外必须安排所有 Istio pod 在 Linux 节点上运行。

本文将 Istio 安装指南分为多个独立步骤。 最终结果的结构与官方 Istio 安装[指南][istio-install-helm]相同。

## <a name="download-istio"></a>下载 Istio

首先，下载并解压缩最新的 Istio 版本。 MacOS、Linux 或适用于 Linux 的 Windows 子系统上的 bash shell 的步骤和 PowerShell shell 的步骤稍有不同。 选择与您的首选的环境相匹配的以下安装步骤之一：

* [MacOS、Linux 或适用于 Linux 的 Windows 子系统上的 Bash](#bash)
* [PowerShell](#powershell)

### <a name="bash"></a>Bash

在 MacOS 上，使用 `curl` 下载最新的 Istio 版本，然后使用 `tar` 进行解压缩，如下所示：

```bash
# Specify the Istio version that will be leveraged throughout these instructions
ISTIO_VERSION=1.1.3

# MacOS
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-osx.tar.gz" | tar xz
```

在 Linux 或适用于 Linux 的 Windows 子系统上，使用 `curl` 下载最新的 Istio 版本，然后使用 `tar` 进行解压缩，如下所示：

```bash
# Specify the Istio version that will be leveraged throughout these instructions
ISTIO_VERSION=1.1.3

curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-linux.tar.gz" | tar xz
```

现在，转到的部分[安装 Istio istioctl 客户端二进制](#install-the-istio-istioctl-client-binary)。

### <a name="powershell"></a>PowerShell

在 PowerShell 中，使用`Invoke-WebRequest`下载最新的 Istio 版本和然后提取与`Expand-Archive`，如下所示：

```powershell
# Specify the Istio version that will be leveraged throughout these instructions
$ISTIO_VERSION="1.1.3"

# Windows
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -URI "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-win.zip" -OutFile "istio-$ISTIO_VERSION.zip"
Expand-Archive -Path "istio-$ISTIO_VERSION.zip" -DestinationPath .
```

现在，转到的部分[安装 Istio istioctl 客户端二进制](#install-the-istio-istioctl-client-binary)。

## <a name="install-the-istio-istioctl-client-binary"></a>安装二进制 Istio istioctl 客户端

> [!IMPORTANT]
> 确保从已经下载并提取的 Istio 版本的顶层文件夹运行此部分的步骤。

`istioctl`二进制客户端在客户端计算机上运行并允许您与 Istio 服务网格进行交互。 客户端操作系统之间稍有不同的安装步骤。 选择与您的首选的环境相匹配的以下安装步骤之一：

* [MacOS](#macos)
* [Linux 或适用于 Linux 的 Windows 子系统](#linux-or-windows-subsystem-for-linux)
* [Windows](#windows)

### <a name="macos"></a>MacOS

若要在 MacOS 上的基于 bash 的 shell 中安装 Istio `istioctl` 客户端二进制文件，请使用以下命令。 这些命令可将 `istioctl` 客户端二进制文件复制到 `PATH` 中的标准用户程序位置。

```bash
cd istio-$ISTIO_VERSION
sudo cp ./bin/istioctl /usr/local/bin/istioctl
sudo chmod +x /usr/local/bin/istioctl
```

如果想要通过命令行完成 `istioctl` 客户端二进制文件，则按如下所示进行设置：

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && istioctl collateral --bash -o ~/completions
source ~/completions/istioctl.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/istioctl.bash" >> ~/.bashrc
```

现在，转到下一节[安装在 AKS 上 Istio CRDs](#install-the-istio-crds-on-aks)。

### <a name="linux-or-windows-subsystem-for-linux"></a>Linux 或适用于 Linux 的 Windows 子系统

使用以下命令在 Linux 或[适用于 Linux 的 Windows 子系统][install-wsl]上的基于 bash 的 shell 中安装 Istio `istioctl` 客户端二进制文件。 这些命令可将 `istioctl` 客户端二进制文件复制到 `PATH` 中的标准用户程序位置。

```bash
cd istio-$ISTIO_VERSION
sudo cp ./bin/istioctl /usr/local/bin/istioctl
sudo chmod +x /usr/local/bin/istioctl
```

如果想要通过命令行完成 `istioctl` 客户端二进制文件，则按如下所示进行设置：

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && istioctl collateral --bash -o ~/completions
source ~/completions/istioctl.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/istioctl.bash" >> ~/.bashrc
```

现在，转到下一节[安装在 AKS 上 Istio CRDs](#install-the-istio-crds-on-aks)。

### <a name="windows"></a>Windows

若要安装 Istio`istioctl`二进制文件中的客户端**Powershell**-基于的 shell 上 Windows，使用以下命令。 这些命令将复制`istioctl`二进制 Istio 文件夹到客户端并使其永久可通过在`PATH`。 不需要提升的 （管理员） 权限才能运行这些命令。

```powershell
cd istio-$ISTIO_VERSION
New-Item -ItemType Directory -Force -Path "C:\Istio"
Copy-Item -Path .\bin\istioctl.exe -Destination "C:\Istio\"
$PATH = [environment]::GetEnvironmentVariable("PATH", "User")
[environment]::SetEnvironmentVariable("PATH", $PATH + "; C:\Istio\", "User")
```

现在，转到下一节[安装在 AKS 上 Istio CRDs](#install-the-istio-crds-on-aks)。

## <a name="install-the-istio-crds-on-aks"></a>在 AKS 上安装 Istio CRDs

> [!IMPORTANT]
> 确保从已经下载并提取的 Istio 版本的顶层文件夹运行此部分的步骤。

使用 Istio[自定义资源定义 (CRDs)] [ kubernetes-crd]来管理其运行时配置。 我们需要首先，安装 Istio CRDs，因为 Istio 组件对其具有依赖关系。 使用 Helm 并`istio-init`图表以安装到 Istio CRDs `istio-system` AKS 群集中的命名空间：

```azurecli
helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
```

[作业][ kubernetes-jobs]作为的一部分部署`istio-init`Helm 图表安装 CRDs。 这些作业应该花费 1 到 2 分钟才能完成，具体取决于你的群集环境。 你可以验证该作业已成功完成，如下所示：

```azurecli
kubectl get jobs -n istio-system
```

以下示例输出显示已成功完成的作业。

```console
NAME                COMPLETIONS   DURATION   AGE
istio-init-crd-10   1/1           16s        18s
istio-init-crd-11   1/1           15s        18s
```

现在，我们已证实作业成功完成后，让我们验证有了正确的 Istio CRDs 安装数。 你可以验证已通过运行相应命令为您的环境安装了所有 53 Istio CRDs。 此命令应返回数`53`。

Bash

```bash
kubectl get crds | grep 'istio.io' | wc -l
```

Powershell

```powershell
(kubectl get crds | Select-String -Pattern 'istio.io').Count
```

如果您有连接到此点，即表示你已成功安装 Istio CRDs。 现在，转到下一节[Istio 组件安装在 AKS 上](#install-the-istio-components-on-aks)。

## <a name="install-the-istio-components-on-aks"></a>Istio 组件安装在 AKS 上

> [!IMPORTANT]
> 确保从已经下载并提取的 Istio 版本的顶层文件夹运行此部分的步骤。

我们将安装[Grafana] [ grafana]并[Kiali] [ kiali]作为我们 Istio 安装的一部分。 Grafana 提供分析和监视仪表板，并 Kiali 提供服务网格可观察性仪表板。 在我们的设置，这些组件的每个要求必须为提供的凭据[机密][kubernetes-secrets]。

我们可以安装 Istio 组件之前，我们必须为 Grafana 和 Kiali 创建机密。 通过运行适当的命令为您的环境中创建这些机密。

### <a name="add-grafana-secret"></a>添加 Grafana 机密

替换为`REPLACE_WITH_YOUR_SECURE_PASSWORD`令牌与您的密码，并运行以下命令：

#### <a name="macos-linux"></a>MacOS, Linux

```bash
GRAFANA_USERNAME=$(echo -n "grafana" | base64)
GRAFANA_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE
EOF
```

#### <a name="windows"></a>Windows

```powershell
$GRAFANA_USERNAME=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("grafana"))
$GRAFANA_PASSPHRASE=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("REPLACE_WITH_YOUR_SECURE_PASSWORD"))

"apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE" | kubectl apply -f -
```

### <a name="add-kiali-secret"></a>添加 Kiali 机密

替换为`REPLACE_WITH_YOUR_SECURE_PASSWORD`令牌与您的密码，并运行以下命令：

#### <a name="macos-linux"></a>MacOS, Linux

```bash
KIALI_USERNAME=$(echo -n "kiali" | base64)
KIALI_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

#### <a name="windows"></a>Windows

```powershell
$KIALI_USERNAME=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("kiali"))
$KIALI_PASSPHRASE=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("REPLACE_WITH_YOUR_SECURE_PASSWORD"))

"apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE" | kubectl apply -f -
```

### <a name="install-istio-components"></a>安装 Istio 组件

现在，我们已在 AKS 群集中成功创建的 Grafana 和 Kiali 机密，就可以安装 Istio 组件。 使用 Helm 并`istio`图表以安装到 Istio 组件`istio-system`AKS 群集中的命名空间。 为您的环境使用适当的命令。

> [!NOTE]
> 我们正在将用作我们安装的一部分的以下选项：
> - `global.controlPlaneSecurityEnabled=true` -启用了控制平面的相互 TLS
> - `mixer.adapters.useAdapterCRDs=false` -删除 mixer 适配器 CRDs 上的监视，因为它们是过时，这将提高性能
> - `grafana.enabled=true` -启用分析的 Grafana 部署和监视仪表板
> - `grafana.security.enabled=true` -为 Grafana 启用身份验证
> - `tracing.enabled=true` -启用跟踪 Jaeger 部署
> - `kiali.enabled=true` -启用 Kiali 部署服务网格可观察性仪表板

Bash

```bash
helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
  --set global.controlPlaneSecurityEnabled=true \
  --set mixer.adapters.useAdapterCRDs=false \
  --set grafana.enabled=true --set grafana.security.enabled=true \
  --set tracing.enabled=true \
  --set kiali.enabled=true
```

Powershell

```powershell
helm install install/kubernetes/helm/istio --name istio --namespace istio-system `
  --set global.controlPlaneSecurityEnabled=true `
  --set mixer.adapters.useAdapterCRDs=false `
  --set grafana.enabled=true --set grafana.security.enabled=true `
  --set tracing.enabled=true `
  --set kiali.enabled=true
```

`istio` Helm 图表部署大量对象。 您可以看到输出中的列表在`helm install`上述命令。 Istio 组件的部署可能需要 4 到 5 分钟才能完成，具体取决于你的群集环境。

> [!NOTE]
> 若要在 Linux 节点上运行，必须安排所有 Istio pod。 如果您有 Windows Server 节点池除了 Linux 节点池上的群集，验证所有 Istio pod 有已都计划在 Linux 节点上运行。

此时，已部署到 AKS 群集的 Istio。 若要确保我们已成功部署 Istio，让我们转到下一节[验证 Istio 安装](#validate-the-istio-installation)。

## <a name="validate-the-istio-installation"></a>验证 Istio 安装

首先，确认已创建所需的服务。 使用 [kubectl get svc][kubectl-get] 命令查看正在运行的服务。 查询`istio-system`命名空间，其中的 Istio 和外接程序组件安装的`istio`Helm 图表：

```console
kubectl get svc --namespace istio-system --output wide
```

以下示例输出显示了现在应正在运行的服务：

- `istio-*` 服务
- `jaeger-*``tracing`，和`zipkin`外接程序跟踪服务
- `prometheus` 外接程序指标服务
- `grafana` 外接程序分析和监视仪表板服务
- `kiali` 外接程序服务网格仪表板服务

如果 `istio-ingressgateway` 显示 `<pending>` 的外部 IP，请在 Azure 网络分配 IP 地址之前等待几分钟。

```console
NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                                                                                                                      AGE       SELECTOR
grafana                  ClusterIP      10.0.81.182    <none>          3000/TCP                                                                                                                                     119s      app=grafana
istio-citadel            ClusterIP      10.0.96.33     <none>          8060/TCP,15014/TCP                                                                                                                           119s      istio=citadel
istio-galley             ClusterIP      10.0.237.158   <none>          443/TCP,15014/TCP,9901/TCP                                                                                                                   119s      istio=galley
istio-ingressgateway     LoadBalancer   10.0.154.12    20.188.211.19   15020:30603/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31198/TCP,15030:30610/TCP,15031:30937/TCP,15032:31344/TCP,15443:31499/TCP   119s      app=istio-ingressgateway,istio=ingressgateway,release=istio
istio-pilot              ClusterIP      10.0.178.56    <none>          15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       119s      istio=pilot
istio-policy             ClusterIP      10.0.116.118   <none>          9091/TCP,15004/TCP,15014/TCP                                                                                                                 119s      istio-mixer-type=policy,istio=mixer
istio-sidecar-injector   ClusterIP      10.0.31.160    <none>          443/TCP                                                                                                                                      119s      istio=sidecar-injector
istio-telemetry          ClusterIP      10.0.187.246   <none>          9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       119s      istio-mixer-type=telemetry,istio=mixer
jaeger-agent             ClusterIP      None           <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                                   119s      app=jaeger
jaeger-collector         ClusterIP      10.0.116.63    <none>          14267/TCP,14268/TCP                                                                                                                          119s      app=jaeger
jaeger-query             ClusterIP      10.0.22.108    <none>          16686/TCP                                                                                                                                    119s      app=jaeger
kiali                    ClusterIP      10.0.142.50    <none>          20001/TCP                                                                                                                                    119s      app=kiali
prometheus               ClusterIP      10.0.138.134   <none>          9090/TCP                                                                                                                                     119s      app=prometheus
tracing                  ClusterIP      10.0.165.210   <none>          80/TCP                                                                                                                                       118s      app=jaeger
zipkin                   ClusterIP      10.0.126.211   <none>          9411/TCP                                                                                                                                     118s      app=jaeger
```

接下来，确认已创建所需的 Pod。 使用[kubectl get pod] [ kubectl-get]命令，并再次查询`istio-system`命名空间：

```console
kubectl get pods --namespace istio-system
```

以下示例输出显示了正在运行的 Pod：

- `istio-*` pod
- `prometheus-*`外接程序指标 pod
- `grafana-*`外接程序分析和监视仪表板 pod
- `kiali`外接程序服务网格仪表板 pod

```console
NAME                                     READY     STATUS      RESTARTS   AGE
grafana-88779954d-nzpm7                  1/1       Running     0          6m26s
istio-citadel-7f699dc8c8-n7q8g           1/1       Running     0          6m26s
istio-galley-649bc8cd97-wfjzm            1/1       Running     0          6m26s
istio-ingressgateway-65dfbd566-42wkn     1/1       Running     0          6m26s
istio-init-crd-10-tmtw5                  0/1       Completed   0          20m38s
istio-init-crd-11-ql25l                  0/1       Completed   0          20m38s
istio-pilot-958dd8cc4-4ckf9              2/2       Running     0          6m26s
istio-policy-86b4b7cf9-zf7v7             2/2       Running     4          6m26s
istio-sidecar-injector-d48786c5c-pmrj9   1/1       Running     0          6m26s
istio-telemetry-7f6996fdcc-84w94         2/2       Running     3          6m26s
istio-tracing-79db5954f-h7hmz            1/1       Running     0          6m26s
kiali-5c4cdbb869-s28dv                   1/1       Running     0          6m26s
prometheus-67599bf55b-pgxd8              1/1       Running     0          6m26s
```

应该有两个`istio-init-crd-*`pod 数与`Completed`状态。 这些 pod 都负责运行在前面步骤中创建 CRDs 的作业。 所有其他 pod 应显示的状态`Running`。 如果 Pod 没有这些状态，请在运行之前等待一两分钟。 如果任何 Pod 报告问题，请使用 [kubectl describe pod][kubectl-describe] 命令查看其输出和状态。

## <a name="accessing-the-add-ons"></a>访问加载项

上述安装程序中安装了大量提供附加功能的 Istio 加载项。 加载项用户界面不会通过外部 IP 地址公开。 若要访问加载项用户界面，请使用 [kubectl port-forward][kubectl-port-forward] 命令。 此命令创建 AKS 群集中的客户端计算机和相关的 pod 之间建立安全连接。

我们通过在本文前面为其指定凭据为 Grafana 和 Kiali 添加额外的安全层。

### <a name="grafana"></a>Grafana

通过提供分析和监视仪表板 Istio [Grafana][grafana]。 本地端口转接`3000`端口在客户端计算机上`3000`AKS 群集中运行 Grafana pod 上：

```console
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000
```

以下示例输出显示了正在为 Grafana 配置的端口转发：

```console
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

现在可以在客户端计算机上通过以下 URL 访问 Grafana：[http://localhost:3000](http://localhost:3000)。 请记住使用 Grafana 通过机密之前创建系统提示时的凭据。

### <a name="prometheus"></a>Prometheus

通过提供 Istio 的指标[Prometheus][prometheus]。 本地端口转接`9090`端口在客户端计算机上`9090`AKS 群集中运行 Prometheus pod 上：

```console
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090
```

以下示例输出显示了正在为 Prometheus 配置的端口转发：

```console
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

现在可以在客户端计算机上通过以下 URL 访问 Prometheus 表达式浏览器：[http://localhost:9090](http://localhost:9090)。

### <a name="jaeger"></a>Jaeger

提供跟踪内 Istio [Jaeger][jaeger]。 本地端口转接`16686`端口在客户端计算机上`16686`AKS 群集中运行 Jaeger pod 上：

```console
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686
```

以下示例输出显示了正在为 Jaeger 配置的端口转发：

```console
Forwarding from 127.0.0.1:16686 -> 16686
Forwarding from [::1]:16686 -> 16686
```

现在可以在客户端计算机上通过以下 URL 访问 Jaeger 跟踪用户界面：[http://localhost:16686](http://localhost:16686)。

### <a name="kiali"></a>Kiali

服务网格可观察性仪表板由 [Kiali][kiali] 提供。 本地端口转接`20001`端口在客户端计算机上`20001`AKS 群集中运行 Kiali pod 上：

```console
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```

以下示例输出显示了正在为 Kiali 配置的端口转发：

```console
Forwarding from 127.0.0.1:20001 -> 20001
Forwarding from [::1]:20001 -> 20001
```

现在可以在客户端计算机上通过以下 URL 访问 Kiali 服务网格可观察性仪表板：[http://localhost:20001/kiali/console/](http://localhost:20001/kiali/console/)。 请记住使用你创建的凭据通过 Kiali 机密前面出现提示时。

## <a name="uninstall-istio-from-aks"></a>从 AKS 卸载 Istio

> [!WARNING]
> 从正在运行的系统中删除 Istio 可能会导致流量与相关的服务之间的问题。 请确保您已经为您的系统仍正常运行而无需 Istio 在继续之前预配。

### <a name="remove-istio-components-and-namespace"></a>删除 Istio 组件和命名空间

若要从 AKS 群集中删除 Istio，使用以下命令。 `helm delete`命令将删除`istio`并`istio-init`图表，并且`kubectl delete ns`命令将删除`istio-system`命名空间。

```azurecli
helm delete --purge istio
helm delete --purge istio-init
kubectl delete ns istio-system
```

### <a name="remove-istio-crds"></a>删除 Istio CRDs

上述命令将删除所有 Istio 组件和命名空间，但我们仍保留使用 Istio CRDs。 若要删除 CRDs，您可以使用一个以下方法。

方法 #1-此命令假定 Istio 用来安装使用 Istio 的已下载并提取版本的顶级文件夹中运行此步骤。

```azure-cli
kubectl delete -f install/kubernetes/helm/istio-init/files
```

如果不再有权访问已下载并解压的 Istio 用来安装使用 Istio 发布这些命令 #2-使用以下任意方法。 此命令将需要稍长的时间-它预计需要几分钟才能完成。

Bash
```bash
kubectl get crds -o name | grep 'istio.io' | xargs -n1 kubectl delete
```

Powershell
```powershell
kubectl get crds -o name | Select-String -Pattern 'istio.io' |% { kubectl delete $_ }
```

## <a name="next-steps"></a>后续步骤

以下文档介绍如何使用 Istio 来提供智能路由推出 canary 发布：

> [!div class="nextstepaction"]
> [AKS Istio 智能路由方案][istio-scenario-routing]

若要了解 Istio 的更多安装和配置选项，请参阅以下官方 Istio 文章：

- [Istio - Helm 安装指南][istio-install-helm]
- [Istio - Helm 安装选项][istio-install-helm-options]

也可以使用 [Istio Bookinfo 应用程序示例][istio-bookinfo-example]关注其他方案。

<!-- LINKS - external -->
[istio]: https://istio.io
[helm]: https://helm.sh

[istio-docs-concepts]: https://istio.io/docs/concepts/what-is-istio/
[istio-github]: https://github.com/istio/istio
[istio-github-releases]: https://github.com/istio/istio/releases
[istio-release-notes]: https://istio.io/about/notes/
[istio-install-download]: https://istio.io/docs/setup/kubernetes/download-release/
[istio-install-helm]: https://istio.io/docs/setup/kubernetes/install/helm/
[istio-install-helm-options]: https://istio.io/docs/reference/config/installation-options/
[istio-bookinfo-example]: https://istio.io/docs/examples/bookinfo/
[install-wsl]: https://docs.microsoft.com/windows/wsl/install-win10

[kubernetes-crd]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[kubernetes-jobs]: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
[kubernetes-secrets]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/
[jaeger]: https://www.jaegertracing.io/
[kiali]: https://www.kiali.io/

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[istio-scenario-routing]: ./istio-scenario-routing.md
[helm-install]: ./kubernetes-helm.md