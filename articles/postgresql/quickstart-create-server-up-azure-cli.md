---
title: 快速入门：使用 CLI 命令 az postgres up 创建 Azure Database for PostgreSQL - 单一服务器
description: 使用 Azure CLI（命令行界面）up 命令创建 Azure Database for PostgreSQL - 单一服务器的快速入门指南。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/06/2019
ms.openlocfilehash: 49f71c199a2832d763bb3c19d878fade47dfb8e4
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65069075"
---
# <a name="quickstart-use-an-azure-cli-command-az-postgres-up-preview-to-create-an-azure-database-for-postgresql---single-server"></a>快速入门：使用 Azure CLI 命令、az postgres up（预览版）创建 Azure Database for PostgreSQL - 单一服务器

> [!IMPORTANT]
> [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) Azure CLI 命令现为预览版。

用于 PostgreSQL 的 Azure 数据库是一种托管服务，可用于在云中运行、管理和缩放具有高可用性的 PostgreSQL 数据库。 Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 此快速入门介绍了如何使用 [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) 命令和 Azure CLI 来创建 Azure Database for PostgreSQL 服务器。 除了创建服务器，`az postgres up` 命令还会创建示例数据库、在数据库中创建根用户、为 Azure 服务打开防火墙，并为客户端计算机创建默认防火墙规则。 这些默认值有助于加快开发过程。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

本文要求在本地运行 Azure CLI 2.0 或更高版本。 若要查看安装的版本，请运行 `az --version` 命令。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

需要使用 [az login](/cli/azure/authenticate-azure-cli?view=interactive-log-in) 命令登录账户。 记下命令行输出中相应订阅名称的 ID 属性。

```azurecli
az login
```

如果有多个订阅，请选择应计费的资源所在的相应订阅。 使用 [az account set](/cli/azure/account) 命令选择帐户下的特定订阅 ID。 将 az login 输出中的你的订阅的订阅 ID 属性替换到订阅 ID 占位符中。

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-postgresql-server"></a>创建 Azure Database for PostgreSQL 服务器

要使用命令，请安装 [db-up](/cli/azure/ext/db-up) 扩展。 如果返回错误，则请确保已安装最新版本的 Azure CLI。 请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

```azurecli
az extension add --name db-up
```

若要使用以下命令创建 Azure Database for PostgreSQL 服务器：

```azurecli
az postgres up
```

服务器是使用以下默认值创建的（除非手动替代它们）：

**设置** | **默认值** | **说明**
---|---|---
server-name | 系统生成的 | 用于标识用于 PostgreSQL 的 Azure 数据库服务器的唯一名称。
resource-group | 系统生成的 | 新的 Azure 资源组。
sku-name | GP_Gen5_2 | SKU 的名称。 请遵循简写约定 {pricing tier}\_{compute generation}\_{vCores}。 默认为带 2 个 vCore 的常规用途 Gen5 服务器。 有关各层的详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/postgresql/)。
backup-retention | 7 | 备份保留的期限。 单位为天。
geo-redundant-backup | 已禁用 | 是否应该为此服务启用异地冗余备份。
location | westus2 | 服务器的 Azure 位置。
ssl-enforcement | 已禁用 | 是否应该为此服务启用 ssl。
storage-size | 5120 | 服务器的存储容量（单位是兆字节）。
版本 | 10 | PostgreSQL 主版本。
admin-user | 系统生成的 | 管理员用户名。
admin-password | 系统生成的 | 管理员用户的密码。

> [!NOTE]
> 有关 `az postgres up` 命令及其附加参数的详细信息，请参阅 [Azure CLI 文档](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up)。

服务器创建后，随附有以下设置：

- 创建了一条名为“devbox”的防火墙规则。 Azure CLI 尝试检测运行 `az postgres up` 命令的计算机的 IP 地址以及该 IP 面向的允许列表。
- “允许访问 Azure 服务”设置为“开”。 此设置会将服务器的防火墙配置为接受来自所有 Azure 资源（包括非订阅中的资源）的连接。
- 创建了一个名为“sampledb”的空数据库
- 创建了一个特权为“sampledb”且名为“root”的新用户

> [!NOTE]
> Azure Database for PostgreSQL 通过端口 5432 进行通信。 从企业网络内部进行连接时，该网络的防火墙可能不允许经端口 5432 的出站流量。 让 IT 部门打开端口 5432 以连接到你的服务器。

## <a name="get-the-connection-information"></a>获取连接信息

在完成 `az postgres up` 命令后，将向你返回一个常用编程语言连接字符串列表。 这些连接字符串使用你新创建的 Azure Database for PostgreSQL 服务器的特定属性进行了预配置。

可使用 [az postgres show-connection-string](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-show-connection-string) 命令再次列出这些连接字符串。

## <a name="clean-up-resources"></a>清理资源

请使用以下命令清除在此快速入门中创建的所有资源。 此命令会删除 Azure Database for PostgreSQL 服务器和资源组。

```azurecli
az postgres down --delete-group
```

若要删除新创建的服务器，可运行 [az postgres down](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-down) 命令。

```azurecli
az postgres down
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用导出和导入功能迁移数据库](./howto-migrate-using-export-and-import.md)
