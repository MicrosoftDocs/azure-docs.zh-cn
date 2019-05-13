---
title: 手动停止或启动 Azure 容器实例中的容器
description: 了解如何手动停止或启动 Azure 容器实例中的容器组。
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: article
ms.date: 04/15/2019
ms.author: danlep
ms.openlocfilehash: 8e62d106a42dfbec897e5e14cf68fd3d7fd823c4
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65070820"
---
# <a name="manually-stop-or-start-containers-in-azure-container-instances"></a>手动停止或启动 Azure 容器实例中的容器

[重启策略](container-instances-restart-policy.md)容器组的设置确定容器实例如何启动或停止默认情况下。 您可以覆盖默认设置通过手动停止或启动容器组。

## <a name="stop"></a>停止

手动停止正在运行的容器组-例如，通过使用[az 容器停止][ az-container-stop]命令或 Azure 门户。 对于某些容器工作负载，可能想要停止长时间运行的容器组后定义的时间段，以节省成本。 

*当容器组进入停止状态时，它会终止并回收组中的所有容器。它不会保留容器状态。*

当容器会被回收，[资源](container-instances-container-groups.md#resource-allocation)已解除分配和计费的容器组将停止。

停止操作不起如果容器组已终止 （处于 Succeeded 或 Failed 状态）。 例如，包含已成功运行的一次性运行容器任务的容器组终止处于成功状态。 尝试停止组中的状态不会更改状态。 

## <a name="start"></a>开始

时已停止的容器组的或者因为靠自己终止容器或手动停止组-你可以启动容器。 例如，使用[az 容器开始][ az-container-start]命令或 Azure 门户在组中，手动启动容器。 如果更新了任何容器的容器映像，则会拉取一个新映像。 

启动容器组会使用相同的容器配置开始一个新部署。 此操作可帮助你快速重复使用按预期方式工作的已知容器组配置。 你无需创建新的容器组便可运行相同的工作负荷。

容器组中的所有容器会都启动此操作。 无法启动特定容器组中。

手动启动或重启容器组后，容器组将根据所配置的重启策略运行。
  
## <a name="restart"></a>重新启动

在运行时-例如，通过使用，你可以重新启动容器组[az 容器重启][ az-container-restart]命令。 此操作会重启容器组中的所有容器。 如果更新了任何容器的容器映像，则会拉取一个新映像。 

如果你想要排查部署问题，则重启容器组会有所帮助。 例如，如果临时资源限制阻止了你的容器成功运行，则重启组可能会解决此问题。

此操作重启容器组中的所有容器。 在组中，无法重新启动特定的容器。

手动重启容器组后，根据已配置的容器组运行重启策略。

## <a name="next-steps"></a>后续步骤

详细了解如何[重新启动策略设置](container-instances-restart-policy.md)Azure 容器实例中。

除了手动停止和启动使用现有配置的容器组，你可以[更新设置](container-instances-update.md)的正在运行的容器组。

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[az-container-restart]: /cli/azure/container?view=azure-cli-latest#az-container-restart
[az-container-start]: /cli/azure/container?view=azure-cli-latest#az-container-start
[az-container-stop]: /cli/azure/container?view=azure-cli-latest#az-container-stop
