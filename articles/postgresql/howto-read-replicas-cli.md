---
title: 管理适用于 PostgreSQL 的 Azure CLI 从单个服务器的 Azure 数据库的只读的副本
description: 了解如何管理用于 PostgreSQL 的 Azure CLI 从单个服务器的 Azure 数据库中的只读的副本。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 9730faf3191ef2e2bd0b6c3caddefa0492b33fc5
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510234"
---
# <a name="create-and-manage-read-replicas-from-the-azure-cli"></a>创建和管理从 Azure CLI 的只读的副本

在本文中，您将学习如何创建和管理用于从 Azure CLI 的 PostgreSQL 的 Azure 数据库中的只读的副本。 若要详细了解只读副本，请参阅[概述](concepts-read-replicas.md)。

> [!NOTE]
> Azure CLI 尚不支持创建副本从主服务器不同的区域中。 若要创建跨区域副本，请使用[Azure 门户](howto-read-replicas-portal.md)。

## <a name="prerequisites"></a>必备组件
- 用作主服务器的 [Azure Database for PostgreSQL 服务器](quickstart-create-server-up-azure-cli.md)。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 2.0 版或更高版本。 若要查看安装的版本，请运行 `az --version` 命令。 如果需要进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="prepare-the-master-server"></a>准备主服务器
必须使用以下步骤在“常规用途”和“内存优化”层中准备主服务器。

主服务器上的 `azure.replication_support` 参数必须设置为 **REPLICA**。 更改此静态参数后，重新启动服务器是必需的更改才能生效。

1. 设置`azure.replication_support`到副本。

   ```azurecli-interactive
   az postgres server configuration set --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support --value REPLICA
   ```

2. 重新启动以将更改应用到服务器。

   ```azurecli-interactive
   az postgres server restart --name mydemoserver --resource-group myresourcegroup
   ```

## <a name="create-a-read-replica"></a>创建只读副本

[Az postgres server 副本创建](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-create)命令需要以下参数：

| 设置 | 示例值 | 描述  |
| --- | --- | --- |
| resource-group | myresourcegroup |  资源组将在其中创建副本服务器。  |
| 名称 | mydemoserver-replica | 所创建的新副本服务器的名称。 |
| source-server | mydemoserver | 名称或资源 ID 的现有的主服务器将从复制。 |

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup
```

如果尚未设置`azure.replication_support`参数**副本**在常规用途或内存优化的主服务器和重新启动服务器，收到错误。 完成这两个步骤后再创建一个副本。

副本是使用与主服务器相同的服务器配置创建的。 创建副本后，可以独立于主服务器更改多项设置：计算代系、vCore 数、存储和备份保留期。 定价层也可以独立更改，但“基本”层除外。

> [!IMPORTANT]
> 将主服务器的配置更新为新值之前，请将副本配置更新为与这些新值相等或更大的值。 此操作可确保副本与主服务器发生的任何更改保持同步。

## <a name="list-replicas"></a>列表副本
可以通过查看的主服务器的副本列表[az postgres server 副本列表](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-list)命令。

```azurecli-interactive
az postgres server replica list --server-name mydemoserver --resource-group myresourcegroup 
```

## <a name="stop-replication-to-a-replica-server"></a>停止复制到副本服务器
可以使用停止主服务器和读取的副本之间的复制[az postgres server 副本停止](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-stop)命令。

停止复制到主服务器和只读副本后，无法撤消该操作。 只读副本将成为支持读取和写入的独立服务器。 独立服务器不能再次成为副本。

```azurecli-interactive
az postgres server replica stop --name mydemoserver-replica --resource-group myresourcegroup 
```

## <a name="delete-a-master-or-replica-server"></a>删除主服务器或副本服务器
若要删除的 master 或副本服务器，请使用[az postgres server delete](/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-delete)命令。

删除主服务器后，将停止复制到所有只读副本。 只读副本将成为支持读取和写入的独立服务器。

```azurecli-interactive
az postgres server delete --name myserver --resource-group myresourcegroup
```

## <a name="next-steps"></a>后续步骤
详细了解 [Azure Database for PostgreSQL 中的只读副本](concepts-read-replicas.md)。