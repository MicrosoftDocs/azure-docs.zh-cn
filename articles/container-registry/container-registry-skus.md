---
title: Azure 容器注册表 SKU
description: 比较 Azure 容器注册表中的不同服务层级。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: article
ms.date: 05/06/2019
ms.author: danlep
ms.openlocfilehash: f36b206ff015511dea7369617febe9220282bbe5
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65069047"
---
# <a name="azure-container-registry-skus"></a>Azure 容器注册表 SKU

Azure 容器注册表 (ACR) 分为多个服务层级（称为“SKU”）。 这些 SKU 提供可预测的定价和多个选项，用来适应你在 Azure 中的专用 Docker 注册表的容量和使用模式。

| SKU | 托管 | 描述 |
| --- | :-------: | ----------- |
| **基本** | 是 | 供开发者了解 Azure 容器注册表的入口点（已优化过成本）。 基本注册表的编程功能（例如 Azure Active Directory [身份验证集成](container-registry-authentication.md#individual-login-with-azure-ad)、[映像删除][container-registry-delete]和 [Webhook][container-registry-webhook]）与标准注册表和高级注册表相同。 但其附带的存储和映像吞吐量最适合使用较少的场景。 |
| **标准** | 是 | 标准注册表的功能与基本注册表相同。不同之处在于，前者附带更多的存储和映像吞吐量。 标准注册表应能够满足大部分生产方案的需求。 |
| **高级** | 是 | 高级注册表附带的存储和并发操作数最多，支持大容量方案。 除了更高版本的映像吞吐量，高级注册表还增添功能，包括[异地复制][ container-registry-geo-replication]用于跨多个区域管理单个注册表[内容信任](container-registry-content-trust.md)对于图像标记签名，并[防火墙和虚拟网络 （预览版）](container-registry-vnet.md)来限制对注册表的访问。 |
|  经典（*在 2019 年 4 月后不可用*） | 否 | 此 SKU 在 Azure 中启用了初始版 Azure 容器注册表服务。 经典注册表由 Azure 在订阅中创建的存储帐户提供支持，这会限制 ACR 提供更高级功能，如增加吞吐量和异地复制。 |

> [!IMPORTANT]
> 经典注册表 SKU 即将**弃用**，**2019 年 4 月**之后将不可用。 对于所有新的注册表，建议使用基本、标准或高级 SKU。 应在 2019 年 4 月之前升级所有现有的经典注册表。 有关升级信息，请参阅[升级经典注册表][container-registry-upgrade]。

基本、标准和高级 SKU（统称为“托管注册表”）全都提供相同的编程功能。 它们也全都可以从完全由 Azure 托管的[映像存储][container-registry-storage]受益。 选择的 SKU 级别越高，性能和可缩放性就越高。 使用多个服务层级，你可以从“基本”层级开始，然后随着注册表使用量增长转换到“标准”和“高级”层级。

## <a name="sku-feature-matrix"></a>SKU 功能矩阵

下表详细介绍了“基本”、“标准”和“高级”服务层级的功能和限制。

[!INCLUDE [container-instances-limits](../../includes/container-registry-limits.md)]

## <a name="changing-skus"></a>更改 SKU

可以通过 Azure CLI 或在 Azure 门户中更改注册表的 SKU。 你可以自由地在各种托管的 SKU 之间切换，只要你要切换到的 SKU 具有所需的最大存储容量即可。 如果从经典 SKU 切换到托管的 SKU 之一，则无法切换回经典 SKU，因为这是一个单向转换。

### <a name="azure-cli"></a>Azure CLI

若要使用 Azure CLI 在各种 SKU 之间切换，请使用 [az acr update][az-acr-update] 命令。 例如，若要切换到高级 SKU，请使用以下命令：

```azurecli
az acr update --name myregistry --sku Premium
```

### <a name="azure-portal"></a>Azure 门户

在 Azure 门户中的容器注册表“概述”中，选择“更新”，然后从“SKU”下拉列表中选择一个新 SKU。

![在 Azure 门户中更新容器注册表 SKU][update-registry-sku]

如果你有经典注册表，则无法在 Azure 门户中选择托管的 SKU。 而是必须先[升级][container-registry-upgrade]到托管的注册表。

## <a name="pricing"></a>定价

有关每个 Azure 容器注册表 SKU 的定价信息，请参阅[容器注册表定价][container-registry-pricing]。

有关数据传输定价的详细信息，请参阅[带宽定价详细信息](https://azure.microsoft.com/pricing/details/bandwidth/)。 

## <a name="next-steps"></a>后续步骤

**Azure 容器注册表路线图**

访问 GitHub 上的 [ACR 路线图][acr-roadmap]，了解服务即将推出的新功能。

**Azure 容器注册表 UserVoice**

在 [ACR UserVoice][container-registry-uservoice] 中提交新功能建议，以及对新功能建议进行投票。

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/forums/903958-azure-container-registry

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-upgrade]: container-registry-upgrade.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md
