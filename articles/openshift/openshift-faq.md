---
title: Azure Red Hat OpenShift 的常见问题 |Microsoft Docs
description: 以下是有关 Microsoft Azure Red Hat OpenShift 的常见问题的解答
services: container-service
author: tylermsft
ms.author: twhitney
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 05/08/2019
ms.openlocfilehash: 881734caf855ccfc4f001693fe261b8448b49bc4
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466203"
---
# <a name="azure-red-hat-openshift-faq"></a>Azure Red Hat OpenShift 常见问题

本文介绍了常见问题 (Faq) 有关 Microsoft Azure Red Hat OpenShift。

## <a name="which-azure-regions-are-supported"></a>支持哪些 Azure 区域？

请参阅[支持的资源](supported-resources.md#azure-regions)有关支持 Azure Red Hat OpenShift 的全球区域列表。

## <a name="can-i-deploy-a-cluster-into-an-existing-virtual-network"></a>可以部署到现有虚拟网络的群集？

不。 但你可以连接到现有的 VNET 通过对等互连的 Azure Red Hat OpenShift 群集。 请参阅[群集的虚拟网络连接到现有的虚拟网络](tutorial-create-cluster.md#optional-connect-the-clusters-virtual-network-to-an-existing-virtual-network)有关详细信息。

## <a name="what-cluster-operations-are-available"></a>群集操作是否可用？

仅可以扩展或减少计算节点的数目。 任何其他修改，允许对`Microsoft.ContainerService/openShiftManagedClusters`后创建的资源。 计算节点的最大数目被限制为 20。

## <a name="what-virtual-machine-sizes-can-i-use"></a>可以使用哪些虚拟机大小？

请参阅[Azure Red Hat OpenShift 的虚拟机大小](supported-resources.md#virtual-machine-sizes)有关可用于 Azure Red Hat OpenShift 群集的虚拟机大小的列表。

## <a name="is-data-on-my-cluster-encrypted"></a>是在我的群集加密数据？

默认情况下是静态加密。 Azure 存储平台保留它，并解密之前检索数据之前自动加密数据。 请参阅[静态数据的 Azure 存储服务加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)有关详细信息。

## <a name="can-i-use-prometheusgrafana-to-monitor-containers-and-manage-capacity"></a>可以使用 Prometheus/Grafana 监视容器和管理容量？

否，不是在当前时间。

## <a name="is-the-docker-registry-available-externally-so-i-can-use-tools-such-as-jenkins"></a>因此，我可以使用 Jenkins 等工具从外部位于 Docker 注册表？

Docker 注册表可用于从`https://docker-registry.apps.<clustername>.<region>.azmosa.io/`但是，不提供强存储持续性保证。 此外可以使用[Azure 容器注册表](https://azure.microsoft.com/services/container-registry/)。

## <a name="is-cross-namespace-networking-supported"></a>跨命名空间网络是否支持？

客户单个项目管理员可以自定义跨命名空间 （包括拒绝它） 的网络上每个项目使用`NetworkPolicy`对象。

## <a name="can-an-admin-manage-users-and-quotas"></a>管理员可以管理用户和配额？

可以。 Azure Red Hat OpenShift 管理员可以管理用户和除了访问创建项目的所有用户的配额。

## <a name="can-i-restrict-a-cluster-to-only-certain-azure-ad-users"></a>可以限制只有特定 Azure AD 用户对群集？

可以。 你可以限制哪些 Azure AD 用户可以登录到群集配置 Azure AD 应用程序。 有关详细信息，请参阅[如何：将您的应用程序限制为一组用户](https://docs.microsoft.com/azure/active-directory/develop/howto-restrict-your-app-to-a-set-of-users)

## <a name="can-a-cluster-have-compute-nodes-across-multiple-azure-regions"></a>可以在群集计算节点具有多个 Azure 区域？

不。 Azure Red Hat OpenShift 群集中的所有节点必须都来自同一个 Azure 区域。

## <a name="are-master-and-infrastructure-nodes-abstracted-away-as-they-are-with-azure-kubernetes-service-aks"></a>主节点和基础结构节点抽取出来一样因为它们是使用 Azure Kubernetes 服务 (AKS)？

不。 在客户订阅中运行所有资源，包括群集主节点中。 这些类型的资源放在只读资源组中。
