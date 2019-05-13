---
title: 教程：使用 Azure CLI 设计 Azure Database for PostgreSQL - 单个服务器
description: 本教程演示如何使用 Azure CLI 创建、配置和查询你的第一个 Azure Database for PostgreSQL - 单个服务器。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 5/6/2019
ms.openlocfilehash: ed272afcfedaf6c781d2a96e5732fe2368914a67
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073071"
---
# <a name="tutorial-design-an-azure-database-for-postgresql---single-server-using-azure-cli"></a>教程：使用 Azure CLI 设计 Azure Database for PostgreSQL - 单个服务器 
在本教程中，需使用 Azure CLI（命令行接口）以及其他实用工具了解如何完成以下操作：
> [!div class="checklist"]
> * 创建 Azure Database for PostgreSQL 服务器
> * 配置服务器防火墙
> * 使用 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 实用工具创建数据库
> * 加载示例数据
> * 查询数据
> * 更新数据
> * 还原数据

可在浏览器中使用 Azure Cloud Shell，或在自己的计算机上[安装 Azure CLI]( /cli/azure/install-azure-cli)，以运行本教程中的命令。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 2.0 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。 

如果有多个订阅，请选择资源所在的相应订阅或对资源进行计费的订阅。 使用 [az account set](/cli/azure/account) 命令选择帐户下的特定订阅 ID。
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>创建资源组
使用 [az group create](/cli/azure/group) 命令创建 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)。 资源组是在其中以组的形式部署和管理 Azure 资源的逻辑容器。 以下示例在 `westus` 位置创建名为 `myresourcegroup` 的资源组。
```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-postgresql-server"></a>创建 Azure Database for PostgreSQL 服务器
使用 [az postgres server create](/cli/azure/postgres/server) 命令创建 [Azure Database for PostgreSQL 服务器](overview.md)。 服务器包含作为组进行管理的一组数据库。 

下面的示例使用服务器管理员登录名 `myadmin` 在资源组 `myresourcegroup` 中创建名为 `mydemoserver` 的服务器。 服务器的名称映射到 DNS 名称，因此需要在 Azure 中全局唯一。 用自己的值替换 `<server_admin_password>`。 它是一台常规用途第 5 代服务器，具有 2 个 vCore。
```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 9.6
```
sku-name 参数值遵循 {定价层}\_{计算层代}\_{vCore 数} 约定，如以下示例中所示：
+ `--sku-name B_Gen5_2` 映射到基本、第 5 代和 2 个 vCore。
+ `--sku-name GP_Gen5_32` 映射到常规用途、第 5 层和 32 个 vCore。
+ `--sku-name MO_Gen5_2` 映射到内存优化、第 5 层和 2 个 vCore。

请参阅[定价层](./concepts-pricing-tiers.md)文档来了解适用于每个区域和每个层的有效值。

> [!IMPORTANT]
> 此处指定的服务器管理员登录名和密码是以后在本快速入门中登录到服务器及其数据库所必需的。 请牢记或记录此信息，以后会使用到它。

默认情况下，在服务器下创建 postgres 数据库。 [postgres](https://www.postgresql.org/docs/9.6/static/app-initdb.html) 是供用户、实用工具和第三方应用程序使用的默认数据库。 


## <a name="configure-a-server-level-firewall-rule"></a>配置服务器级防火墙规则

使用 [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) 命令创建 Azure PostgreSQL 服务器级防火墙规则。 服务器级防火墙规则允许外部应用程序（如 [psql](https://www.postgresql.org/docs/9.2/static/app-psql.html) 或 [PgAdmin](https://www.pgadmin.org/)）通过 Azure PostgreSQL 服务防火墙连接到服务器。 

可以设置涵盖某个 IP 范围的防火墙规则，以便通过网络进行连接。 下面的示例使用 [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) 创建允许从单个 IP 地址进行连接的防火墙规则 `AllowMyIP`。

```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

若要仅允许从你的网络访问 Azure PostgreSQL 服务器，可以将防火墙规则设置为仅涵盖你公司网络 IP 地址范围。

> [!NOTE]
> Azure PostgreSQL 服务器通过端口 5432 进行通信。 从企业网络内部进行连接时，该网络的防火墙可能不允许经端口 5432 的出站流量。 让 IT 部门打开端口 5432，以便连接到 Azure SQL 数据库服务器。
>

## <a name="get-the-connection-information"></a>获取连接信息

若要连接到服务器，需要提供主机信息和访问凭据。
```azurecli-interactive
az postgres server show --resource-group myresourcegroup --name mydemoserver
```

结果采用 JSON 格式。 记下 administratorLogin 和 fullyQualifiedDomainName。
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"

}
```

## <a name="connect-to-azure-database-for-postgresql-database-using-psql"></a>使用 psql 连接到 Azure Database for PostgreSQL 数据库
如果客户端计算机已安装 PostgreSQL，则可使用 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 的本地实例，或 Azure 云控制台连接到 Azure PostgreSQL 服务器。 现在，使用 psql 命令行实用工具连接到“用于 PostgreSQL 的 Azure 数据库”服务器。

1. 运行以下 psql 命令连接到 Azure Database for PostgreSQL 数据库：
   ```azurecli-interactive
   psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
   ```

   例如，以下命令使用访问凭据连接到 PostgreSQL 服务器 mydemoserver.postgres.database.azure.com 上名为“postgres”的默认数据库。 提示输入密码时，输入之前选择的 `<server_admin_password>`。
  
   ```azurecli-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

2. 连接到服务器后，在出现提示时创建空数据库：
   ```sql
   CREATE DATABASE mypgsqldb;
   ```

3. 出现提示时，请执行以下命令，将连接切换到新建的数据库 mypgsqldb：
   ```sql
   \c mypgsqldb
   ```

## <a name="create-tables-in-the-database"></a>在数据库中创建表
现已介绍了如何连接 Azure Database for PostgreSQL，接下来你可以完成一些基本任务：

首先，创建表并加载一些数据。 例如，创建一个跟踪库存信息的表：
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

现可通过键入以下内容在表列表中查看新创建的表：
```sql
\dt
```

## <a name="load-data-into-the-table"></a>将数据加载到表中
创建表后，请在其中插入一些数据。 在打开的命令提示符窗口中，运行以下查询插入几行数据：
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

现已将两行示例数据添加到了之前创建的表中。

## <a name="query-and-update-the-data-in-the-tables"></a>查询和更新表中的数据
执行以下查询，从库存表中检索信息： 
```sql
SELECT * FROM inventory;
```

还可以更新库存表中的数据：
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

检索数据时，可以看到更新后的值：
```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>将数据库还原到以前的时间点
假设你意外删除了某个表。 这是你不能轻易还原的内容。 使用 Azure Database for PostgreSQL 可以返回到服务器有其备份的任何时间点（由所配置的备份保留期确定），并可将此时间点还原到新服务器。 可以使用此新服务器恢复已删除的数据。 

以下命令将示例服务器还原到添加此表之前的时间点：
```azurecli-interactive
az postgres server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time 2017-04-13T13:59:00Z --source-server mydemoserver
```

`az postgres server restore` 命令需以下参数：

| 设置 | 建议的值 | 说明  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  源服务器所在的资源组。  |
| 名称 | mydemoserver-restored | 通过还原命令创建的新服务器的名称。 |
| restore-point-in-time | 2017-04-13T13:59:00Z | 选择要还原到的时间点。 此日期和时间必须在源服务器的备份保留期限内。 使用 ISO8601 日期和时间格式。 例如，可使用自己的本地时区（如 `2017-04-13T05:59:00-08:00`），或使用 UTC Zulu 格式 `2017-04-13T13:59:00Z`。 |
| source-server | mydemoserver | 要从其还原的源服务器的名称或 ID。 |

将服务器还原到某个时间点会创建一个新服务器，该服务器是通过复制所指定的时间点之前那段时间的原始服务器而生成的。 还原的服务器的位置值和定价层值与源服务器相同。

该命令是同步的，且会在服务器还原后返回。 还原完成后，找到创建的新服务器。 验证数据是否按预期还原。


## <a name="next-steps"></a>后续步骤
本教程介绍如何使用 Azure CLI（命令行接口）以及其他实用工具完成以下任务：
> [!div class="checklist"]
> * 创建 Azure Database for PostgreSQL 服务器
> * 配置服务器防火墙
> * 使用 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 实用工具创建数据库
> * 加载示例数据
> * 查询数据
> * 更新数据
> * 还原数据

接下来，了解如何使用 Azure 门户执行类似任务，请查看此教程：[使用 Azure 门户设计第一个 Azure Database for PostgreSQL](tutorial-design-database-using-azure-portal.md)
