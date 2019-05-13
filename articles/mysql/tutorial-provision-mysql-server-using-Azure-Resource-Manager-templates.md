---
title: 教程：使用 Azure 资源管理器模板预配 Azure Database for MySQL 服务器
description: 本教程介绍如何使用 Azure 资源管理器模板预配并自动执行 Azure Database for MySQL 服务器部署。
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: json
ms.topic: tutorial
ms.date: 12/21/2018
ms.custom: mvc
ms.openlocfilehash: 6e4bb7622fe51c0cab4fc45e945e5bb07b1d32f1
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64925840"
---
# <a name="tutorial-provision-an-azure-database-for-mysql-server-using-azure-resource-manager-template"></a>教程：使用 Azure 资源管理器模板预配 Azure Database for MySQL 服务器

借助 [Azure Database for MySQL REST API](https://docs.microsoft.com/rest/api/mysql/)，DevOps 工程师能够自动执行并集成 Azure 中托管的 MySQL 服务器和数据库的预配、配置和操作。  通过 API，可对 Azure Database for MySQL 服务上的 MySQL 服务器和数据库进行创建、枚举、管理和删除。

Azure 资源管理器利用基础 REST API 来声明并计划大规模部署所需的 Azure 资源，这与将基础结构作为代码的概念相一致。 模板可参数化 Azure 资源名称、SKU、网络、防火墙配置和设置，允许一次创建后多次使用。  可使用 [Azure 门户](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal)或 [Visual Studio Code](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-visual-studio-code?tabs=CLI) 轻松创建 Azure 资源管理器模板。 这些模板允许自动执行可集成到 DevOps CI/CD 管道的应用程序打包、标准化和部署。  例如，如果你希望使用 Azure Database for MySQL 后端快速部署 Web 应用，则可以使用来自 GitHub 库的此[快速入门模板](https://azure.microsoft.com/resources/templates/101-webapp-managed-mysql/)来执行端到端部署。

在本教程中，需使用 Azure 资源管理器模板以及其他实用工具了解如何完成以下操作：

> [!div class="checklist"]
> * 使用 Azure 资源管理器模板创建带有 VNet 服务终结点的 Azure Database for MySQL 服务器
> * 使用 [mysql 命令行工具](https://dev.mysql.com/doc/refman/5.6/en/mysql.html)创建数据库
> * 加载示例数据
> * 查询数据
> * 更新数据

如果还没有 Azure 订阅，可以在开始前创建一个[免费 Azure 帐户](https://azure.microsoft.com/free/)。

## <a name="create-an-azure-database-for-mysql-server-with-vnet-service-endpoint-using-azure-resource-manager-template"></a>使用 Azure 资源管理器模板创建带有 VNet 服务终结点的 Azure Database for MySQL 服务器

要获取 Azure Database for MySQL 服务器的 JSON 模板引用，请转到 [Microsoft.DBforMySQL 服务器](/azure/templates/microsoft.dbformysql/servers)模板引用。 下面是示例 JSON 模板，可用于创建运行带有 VNet 服务终结点的 Azure Database for MySQL 的新服务器。
```json
{
  "apiVersion": "2017-12-01",
  "type": "Microsoft.DBforMySQL/servers",
  "name": "string",
  "location": "string",
  "tags": "string",
  "properties": {
    "version": "string",
    "sslEnforcement": "string",
    "administratorLogin": "string",
    "administratorLoginPassword": "string",
    "storageProfile": {
      "storageMB": "string",
      "backupRetentionDays": "string",
      "geoRedundantBackup": "string"
    }
  },
  "sku": {
    "name": "string",
    "tier": "string",
    "capacity": "string",
    "family": "string"
  },
  "resources": [
    {
      "name": "AllowSubnet",
      "type": "virtualNetworkRules",
      "apiVersion": "2017-12-01",
      "properties": {
        "virtualNetworkSubnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnetName'))]",
        "ignoreMissingVnetServiceEndpoint": true
      },
      "dependsOn": [
        "[concat('Microsoft.DBforMySQL/servers/', parameters('serverName'))]"
      ]
    }
  ]
}
```
在此请求中，需要进行自定义的值有：
+   `name` - 指定 MySQL 服务器的名称（不带域名）。
+   `location` - 为 MySQL 服务器指定有效的 Azure 数据中心区域。 例如 westus2。
+   `properties/version` - 指定要部署的 MySQL 服务器版本。 例如 5.6 或 5.7。
+   `properties/administratorLogin` - 指定服务器的 MySQL 管理员登录名。 管理员登录名不能为 azure_superuser、admin、administrator、root、guest 或 public。
+   `properties/administratorLoginPassword` - 为上述指定的 MySQL 管理员用户指定密码。
+   `properties/sslEnforcement` - 指定“已启用/已禁用”以启用/禁用 sslEnforcement。
+   `storageProfile/storageMB` - 指定服务器所需的最大预配存储大小（以兆字节为单位）。 例如 5120。
+   `storageProfile/backupRetentionDays` - 指定所需的备份保持期（天）。 例如 7 天。 
+   `storageProfile/geoRedundantBackup` - 根据 Geo-DR 需求指定“已启用/已禁用”。
+   `sku/tier` - 指定部署的 Basic、GeneralPurpose 或 MemoryOptimized 层。
+   `sku/capacity` - 指定 vCore 容量。 可能值包括 2、4、8、16、32 或 64。
+   `sku/family` - 指定 Gen5 以选择用于服务器部署的硬件代系。
+   `sku/name` - 指定 TierPrefix_family_capacity。 例如 B_Gen5_1、GP_Gen5_16、MO_Gen5_32。 请参阅[定价层](./concepts-pricing-tiers.md)文档，了解每个区域和每个层的有效值。
+   `resources/properties/virtualNetworkSubnetId` - 指定 Azure MySQL 服务器应位于的 VNet 中的子网的 Azure 标识符。 
+   `tags(optional)` - 指定可选标记为用于对资源进行分类，以便计费等的键值对。

如果希望构建 Azure Resource Manager 模板以自动为组织执行 Azure Database for MySQL 部署，建议首先从 Azure 快速入门 GitHub 库中的示例 [Azure 资源管理器模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-managed-mysql-with-vnet)开始，然后在此基础上进行构建。 

如果不熟悉 Azure 资源管理器模板，但是想尝试一下，则可以按以下步骤进行操作：
+   从 Azure 快速入门库中克隆或下载示例 [Azure 资源管理器模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-managed-mysql-with-vnet)。  
+   修改 azuredeploy.parameters.json 以基于引用更新参数值，并保存文件。 
+   借助 Azure CLI，使用以下命令创建 Azure MySQL 服务器

可在浏览器中使用 Azure Cloud Shell，或在自己的计算机上安装 Azure CLI，运行本教程中的代码块。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

```azurecli-interactive
az login
az group create -n ExampleResourceGroup  -l “West US2”
az group deployment create -g $ ExampleResourceGroup   --template-file $ {templateloc} --parameters $ {parametersloc}
```

## <a name="get-the-connection-information"></a>获取连接信息
若要连接到服务器，需要提供主机信息和访问凭据。
```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

结果采用 JSON 格式。 记下 fullyQualifiedDomainName 和 administratorLogin。
```json
{
  "administratorLogin": "myadmin",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus2",
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
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-the-server-using-mysql"></a>使用 mysql 连接服务器
使用 [mysql 命令行工具](https://dev.mysql.com/doc/refman/5.6/en/mysql.html)建立与 Azure Database for MySQL 数据库的连接。 在此示例中，该命令是：
```cmd
mysql -h mydemoserver.database.windows.net -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>创建空数据库
连接到服务器后，创建一个空数据库。
```sql
mysql> CREATE DATABASE mysampledb;
```

出现提示时，请运行以下命令，切换为连接此新建的数据库：
```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>在数据库中创建表
现已介绍了如何连接 Azure Database for MySQL 数据库，接下来可以完成一些基本任务。

首先，创建表并加载一些数据。 创建一个存储清单信息的表。
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>将数据加载到表
表格创建好后，可向其插入一些数据。 在打开的命令提示窗口中，运行以下查询来插入几行数据。
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

现已将两行示例数据加载到了之前创建的表中。

## <a name="query-and-update-the-data-in-the-tables"></a>查询和更新表中的数据
执行以下查询，从数据库表中检索信息。
```sql
SELECT * FROM inventory;
```

还可以更新表中的数据。
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

检索数据时行也会相应进行更新。
```sql
SELECT * FROM inventory;
```

## <a name="next-steps"></a>后续步骤
本教程介绍：
> [!div class="checklist"]
> * 使用 Azure 资源管理器模板创建带有 VNet 服务终结点的 Azure Database for MySQL 服务器
> * 使用 [mysql 命令行工具](https://dev.mysql.com/doc/refman/5.6/en/mysql.html)创建数据库
> * 加载示例数据
> * 查询数据
> * 更新数据
> 
> [如何将应用程序连接到 Azure Database for MySQL](./howto-connection-string.md)
