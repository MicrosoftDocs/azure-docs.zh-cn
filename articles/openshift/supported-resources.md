---
title: 支持 Azure Red Hat OpenShift 的资源 |Microsoft Docs
description: 了解 Microsoft Azure Red Hat OpenShift 支持哪些 Azure 区域和虚拟机大小。
services: container-service
author: tylermsft
ms.author: twhitney
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 22cc195b7808ad4a9111aafcf883a68c51f3709f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076100"
---
# <a name="azure-red-hat-openshift-resources"></a>Red Hat OpenShift 的 azure 资源

本主题列出了 Azure 区域和 Microsoft Azure Red Hat OpenShift 服务支持的虚拟机大小。

## <a name="azure-regions"></a>Azure 区域

可以将 Azure Red Hat OpenShift 群集部署到在全球范围内的以下 Azure 区域：

|区域|CLI 代码|
|-|-|
|🇦🇺 澳大利亚东部|`australiaeast`|
|🇨🇦 加拿大中部|`canadacentral`|
|🇨🇦 加拿大东部|`canadaeast`|
|🇺🇸 美国东部|`eastus`|
|🇺🇸 美国西部|`westus`|
|🇪🇺 欧洲西部|`westeurope`|
|🇪🇺 欧洲北部|`northeurope`|

## <a name="virtual-machine-sizes"></a>虚拟机大小

下面是可以指定 Azure Red Hat OpenShift 群集中的计算节点的支持的虚拟机大小。

> [!Important]
> 每个 VM 都有不同数量的可附加的驱动器。 这可能不是立即清除为内存或 CPU 大小。
> 并非所有 VM 大小都可在所有区域中使用。 即使该 API 支持你指定的大小，可能会遇到错误，如果大小在你指定的区域中不可用。
> 请参阅[每个区域大小的受支持 VM 的当前列表](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)有关详细信息。

## <a name="compute-node-sizes"></a>计算节点大小

Azure Red Hat OpenShift REST API 支持以下的计算节点大小：

|大小|vCPU|RAM|
|-|-|-|
|标准 D4s v3|4|16 GB|
|标准 D8s v3|8|32 GB|
|标准 D16s v3|16|64 GB|
|标准 D32s v3|32|128 GB|
|-|-|-|
|标准 E4s v3|4|32 GB|
|标准 E8s v3|8|64 GB|
|标准 E16s v3|16|128 GB|
|标准 E32s v3|32|256 GB|
|-|-|-|
|标准 F8s v2|8|16 GB|
|标准 F16s v2|16|32 GB|
|标准 F32s v2|32|64 GB|

## <a name="master-node-sizes"></a>主节点大小

以下母版 / Azure Red Hat OpenShift REST API 支持基础结构节点大小：

|大小|vCPU|RAM|
|-|-|-|
|标准 D4s v3|4|16 GB|
|标准 D8s v3|8|32 GB|
|标准 D16s v3|16|64 GB|
|标准 D32s v3|32|128 GB|

## <a name="next-steps"></a>后续步骤

请尝试[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。