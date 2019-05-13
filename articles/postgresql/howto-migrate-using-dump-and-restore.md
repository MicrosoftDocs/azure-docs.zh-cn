---
title: 如何在转储和还原 Azure Database for PostgreSQL-单个服务器
description: 介绍如何将 PostgreSQL 数据库解压缩为转储文件以及从 pg_dump 创建的文件在 Azure Database for PostgreSQL-单个服务器还原。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: aa9485ec8fcabdc0276e0598bd3e19f04d70dfa1
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65066982"
---
# <a name="migrate-your-postgresql-database-using-dump-and-restore"></a>使用转储和还原迁移 PostgreSQL 数据库
可以使用 [pg_dump](https://www.postgresql.org/docs/9.3/static/app-pgdump.html) 将 PostgreSQL 数据库提取到转储文件，并使用 [pg_restore](https://www.postgresql.org/docs/9.3/static/app-pgrestore.html) 从 pg_dump 创建的存档文件中还原 PostgreSQL 数据库。

## <a name="prerequisites"></a>必备组件
若要逐步执行本操作方法指南，需要：
- 一个 [Azure Database for PostgreSQL 服务器](quickstart-create-server-database-portal.md)，其防火墙规则设置为允许访问，并且包含数据库。
- 已安装 [pg_dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) 和 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 命令行实用程序

按照以下步骤转储并还原 PostgreSQL 数据库：

## <a name="create-a-dump-file-using-pgdump-that-contains-the-data-to-be-loaded"></a>使用 pg_dump 创建转储文件，该文件包含要加载数据
若要在本地或某个 VM 中备份现有 PostgreSQL 数据库，请运行以下命令：
```bash
pg_dump -Fc -v --host=<host> --username=<name> --dbname=<database name> > <database>.dump
```
例如，如果有一个本地服务器，并且该服务器中包含一个名为 testdb 的数据库
```bash
pg_dump -Fc -v --host=localhost --username=masterlogin --dbname=testdb > testdb.dump
```


## <a name="restore-the-data-into-the-target-azure-database-for-postrgesql-using-pgrestore"></a>使用 pg_restore 将数据还原到目标 Azure Database for PostrgeSQL
创建目标数据库后，可以使用 pg_restore 命令和 -d、--dbname 参数将数据从转储文件还原到目标数据库。
```bash
pg_restore -v --no-owner –-host=<server name> --port=<port> --username=<user@servername> --dbname=<target database name> <database>.dump
```
包括 --no-owner 参数会导致还原过程中创建的所有对象由使用 --username 指定的用户拥有。 有关详细信息，请参阅有关 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 的正式 PostgreSQL 文档。

> [!NOTE]
> 如果 PostgreSQL 服务器需要 SSL 连接（默认情况下在 Azure Database for PostgreSQL 服务器中启用），请设置环境变量 `PGSSLMODE=require`，以便 pg_restore 工具使用 SSL 连接。 如果不使用 SSL，错误可能会显示为 `FATAL:  SSL connection is required. Please specify SSL options and retry.`
>
> 在 Windows 命令行中，在运行 pg_restore 命令之前运行命令 `SET PGSSLMODE=require`。 在 Linux 或 Bash 中，在运行 pg_restore 命令之前运行命令 `export PGSSLMODE=require`。
>

在此示例中，将数据从转储文件 **testdb.dump** 还原到目标服务器 **mydemoserver.postgres.database.azure.com** 上的数据库 **mypgsqldb**。 
```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb testdb.dump
```

## <a name="optimizing-the-migration-process"></a>优化迁移过程

将现有的 PostgreSQL 数据库迁移到 Azure Database for PostgreSQL 服务的一种方法是，在源上备份数据库并在 Azure 中进行还原。 若要最大限度地缩短完成迁移所需的时间，请考虑将以下参数与备份和还原命令结合使用。

> [!NOTE]
> 有关语法的详细信息，请参阅 [pg_dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) 和 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 文章。
>

### <a name="for-the-backup"></a>对于备份
- 使用 -Fc 交换机进行备份，以便能够并行执行还原以提高速度。 例如：

    ```
    pg_dump -h MySourceServerName -U MySourceUserName -Fc -d MySourceDatabaseName > Z:\Data\Backups\MyDatabaseBackup.dump
    ```

### <a name="for-the-restore"></a>对于还原
- 我们建议将备份文件移动到你要迁移到的 Azure Database for PostgreSQL 服务器所在区域中的 Azure VM，并从该 VM 执行 pg_restore 以减少网络延迟。 此外，我们还建议通过启用[加速网络](../virtual-network/create-vm-accelerated-networking-powershell.md)来创建 VM。

- 默认情况下应该已经完成，但需打开转储文件来验证 create index 语句是否在插入数据之后。 如果不是这种情况，请将 create index 语句移动到插入的数据之后。

- 使用 -Fc 和 -j *#* 交换机进行并行还原。 *#* 是目标服务器上的内核数。 你还可以尝试将 *#* 设置为目标服务器内核数的两倍，以查看产生的影响。 例如：

    ```
    pg_restore -h MyTargetServer.postgres.database.azure.com -U MyAzurePostgreSQLUserName -Fc -j 4 -d MyTargetDatabase Z:\Data\Backups\MyDatabaseBackup.dump
    ```

- 此外，还可以通过在开头添加 *set synchronous_commit = off;* 命令并在末尾添加 *set synchronous_commit = on;* 命令来编辑转储文件。 如果在应用更改数据之前未在末尾打开该功能，可能会导致随后的数据丢失。

- 在目标 Azure Database for PostgreSQL 服务器上，请考虑在还原之前执行以下操作：
    - 关闭查询性能跟踪，因为迁移期间不需要这些统计信息。 可以通过将 pg_stat_statements.track、pg_qs.query_capture_mode 和 pgms_wait_sampling.query_capture_mode 设置为 NONE 来完成此操作。

    - 使用高计算和高内存 sku（如 32 vCore 内存优化）来加速迁移。 完成还原操作后，可以轻松缩回到所需的 sku。 sku 越高，通过增加 pg_restore 命令中相应的 `-j` 参数就可以实现越多的并行性。 

    - 通过增加目标服务器上的 IOPS 可以提高还原性能。 你可以通过增加服务器的存储大小来预配更多 IOPS。 此设置不可逆，但要考虑的一点是，更高的 IOPS 是否在将来有益于你的实际工作负荷。

请记住先在测试环境中测试和验证这些命令，然后再将其用于生产。

## <a name="next-steps"></a>后续步骤
- 若要使用导出和导入迁移 PostgreSQL 数据库，请参阅[使用导入和导出迁移 PostgreSQL 数据库](howto-migrate-using-export-and-import.md)。
- 有关将数据库迁移到 Azure Database for PostgreSQL 的详细信息，请参阅[数据库迁移指南](https://aka.ms/datamigration)。
