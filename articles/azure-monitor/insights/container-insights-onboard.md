---
title: 如何为容器载入 Azure Monitor | Microsoft Docs
description: 本文介绍如何为容器安装和配置 Azure Monitor，以便了解容器的性能以及已识别的性能相关问题。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: magoedte
ms.openlocfilehash: 10b80a9749c5698195ac5d3493ac3b07fd6e24e1
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073306"
---
# <a name="how-to-onboard-azure-monitor-for-containers"></a>如何为容器载入 Azure Monitor  

本文提供的选项，可用于设置用于容器的 Azure Monitor 来监视部署到 Kubernetes 环境和上承载的工作负荷的性能概述[Azure Kubernetes 服务](https://docs.microsoft.com/azure/aks/)。

可使用以下支持的方法为 AKS 的新部署，或是一个或多个现有部署启用适用于容器的 Azure Monitor：

* 借助 Azure 门户、Azure PowerShell 或 Azure CLI
* 使用 [Terraform 和 AKS](../../terraform/terraform-create-k8s-cluster-with-tf-and-aks.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>必备组件 
在开始之前，请确保做好以下准备：

- **Log Analytics 工作区。** 可以在对新 AKS 群集启用监视时创建该工作区，或者让载入体验在 AKS 群集订阅的默认资源组中创建默认的工作区。 如果选择自行创建工作区，可以通过 [Azure 资源管理器](../platform/template-workspace-configuration.md)、[PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) 或在 [Azure 门户](../learn/quick-create-workspace.md)来创建。
- 需要成为 **Log Analytics 参与者角色**的成员才能启用容器监视。 有关如何控制对 Log Analytics 工作区的访问的详细信息，请参阅[管理工作区](../platform/manage-access.md)。
- 在 AKS 群集资源上，你是**[所有者](../../role-based-access-control/built-in-roles.md#owner)** 角色的成员。 

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

## <a name="components"></a>组件 

监视性能的能力依赖于 Linux 容器化 Log Analytics 代理，该代理专门针对用于容器的 Azure Monitor 开发而来。 此专用代理可从群集中的所有节点处收集性能和事件数据，并且在部署期间，会自动部署该代理，并注册指定 Log Analytics 工作区。 该代理的版本为 microsoft/oms:ciprod04202018 或更高版本，并由采用以下格式的日期表示：mmddyyyy。 

>[!NOTE]
>借助适用于 AKS 的 Windows Server 支持的预览版本，AKS 节点的群集，Windows Server 无需安装以收集数据然后转发到 Azure Monitor 的代理。 相反，会自动作为标准部署的一部分部署在群集中的 Linux 节点收集，并将数据转发到 Azure Monitor 代表群集中的所有 Windows 节点。  
>

当该代理的新版本发布时，会在承载于 Azure Kubernetes 服务 (AKS) 上的托管 Kubernetes 群集上自动升级该代理。 要关注发布的版本，请参阅[代理发布公告](https://github.com/microsoft/docker-provider/tree/ci_feature_prod)。 

>[!NOTE] 
>如果已部署 AKS 群集，可使用 Azure CLI 或提供的 Azure 资源管理器模板启用监视，如后文所示。 不能使用 `kubectl` 升级、删除、重新部署或部署代理。 模板需要部署在群集所在的资源组中。

使用下表中所述的以下方法之一用于容器启用 Azure Monitor。

| 部署状态 | 方法 | 描述 | 
|------------------|--------|-------------| 
| 新的 AKS 群集 | [使用 Azure CLI 创建群集](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| 您可以使用 Azure CLI 创建新的 AKS 群集的监视。 | 
| | [使用 Terraform 创建群集](container-insights-enable-new-cluster.md#enable-using-terraform)| 可以让您使用的开源工具 Terraform 创建新的 AKS 群集的监视。 | 
| 现有的 AKS 群集 | [启用使用 Azure CLI](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | 您可以启用监视的已使用 Azure CLI 部署 AKS 群集。 | 
| |[使用 Terraform 启用](container-insights-enable-existing-clusters.md#enable-using-terraform) | 您可以启用监视的已使用 Terraform 的开放源代码工具部署 AKS 群集。 | 
| | [启用从 Azure Monitor](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| 可以启用一个或多个已部署从 Azure Monitor 中的 AKS 群集多页的 AKS 群集的监视。 | 
| | [启用从 AKS 群集](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| 您可以直接从 Azure 门户中的 AKS 群集监视。 | 
| | [启用使用 Azure 资源管理器模板](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| 您可以启用预先配置的 Azure 资源管理器模板与 AKS 群集的监视。 | 

## <a name="next-steps"></a>后续步骤

* 启用监视来捕获 AKS 群集节点和 Pod 的运行状况指标后，可在 Azure 门户中找到这些运行状况指标。 要了解如何将 Azure Monitor 用于容器，请参阅[查看 Azure Kubernetes 服务运行状况](container-insights-analyze.md)。
