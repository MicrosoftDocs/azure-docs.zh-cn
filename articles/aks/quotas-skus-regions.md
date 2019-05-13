---
title: 配额、 SKU 和区域可用性在 Azure Kubernetes 服务 (AKS)
description: 了解有关默认配额、 限制的节点 VM SKU 大小和区域可用性的 Azure Kubernetes 服务 (AKS)。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: iainfou
ms.openlocfilehash: abeb9ef6e467b62cf7332e01e1b77c710b9ba4f4
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072878"
---
# <a name="quotas-virtual-machine-size-restrictions-and-region-availability-in-azure-kubernetes-service-aks"></a>配额、 虚拟机大小限制和区域可用性在 Azure Kubernetes 服务 (AKS)

所有 Azure 服务都包括某些针对资源和功能的默认限制和配额。 对于节点大小，某些虚拟机 (VM) 的 Sku 为然后受限制的使用。

本文详细介绍有关 Azure Kubernetes 服务 (AKS) 资源，以及在 Azure 区域中 AKS 服务的可用性的默认资源限制。

## <a name="service-quotas-and-limits"></a>服务配额和限制

[!INCLUDE [container-service-limits](../../includes/container-service-limits.md)]

## <a name="provisioned-infrastructure"></a>预配的基础结构

所有其他网络、计算和存储限制均适用于预配的基础结构。 有关相关限制，请参阅 [Azure 订阅和服务限制](../azure-subscription-service-limits.md)。

## <a name="restricted-vm-sizes"></a>受限制的 VM 大小

在 AKS 群集中的每个节点包含一个固定的大小的 vCPU 和内存等计算资源。 如果 AKS 节点包含没有足够的计算资源，pod 可能无法正常运行。 若要确保所需*kube 系统*pod 和您的应用程序可以可靠地计划、 不能在 AKS 中使用以下 VM Sku:

- Standard_A0
- Standard_A1
- Standard_A1_v2
- Standard_B1s
- Standard_B1ms
- Standard_F1
- Standard_F1s

VM 类型和其计算资源的详细信息，请参阅[在 Azure 中的虚拟机的大小][vm-skus]。

## <a name="region-availability"></a>上市区域

有关最新的位置的列表可以部署和运行群集，请参阅[AKS 区域可用性][region-availability]。

## <a name="next-steps"></a>后续步骤

某些默认限制和配额可以提高。 若要请求增加一个或多个资源（如果支持此功能），请提交 [Azure 支持请求][azure-support]（选择“配额”作为“问题类型”）。

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/linux/sizes.md
