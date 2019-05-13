---
title: Azure Database for PostgreSQL - Hyperscale (Citus)（预览版）快速入门
description: 有关在 Azure Database for PostgreSQL Hyperscale (Citus)（预览版）中创建和查询分布式表的快速入门。
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: quickstart
ms.date: 05/06/2019
ms.openlocfilehash: 4271d94f07125a870cc4aa859b01db819d583f40
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406451"
---
# <a name="quickstart-create-an-azure-database-for-postgresql---hyperscale-citus-preview-in-the-azure-portal"></a>快速入门：在 Azure 门户中创建 Azure Database for PostgreSQL - Hyperscale (Citus)（预览版）

用于 PostgreSQL 的 Azure 数据库是一种托管服务，可用于在云中运行、管理和缩放具有高可用性的 PostgreSQL 数据库。 本快速入门介绍如何使用 Azure 门户创建 Azure Database for PostgreSQL - Hyperscale (Citus)（预览版）服务器组。 其中将会介绍如何浏览分布式数据：在节点之间将表分片、引入示例数据，以及运行在多个节点上执行的查询。

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-an-azure-database-for-postgresql"></a>创建用于 PostgreSQL 的 Azure 数据库

可以按照以下步骤创建用于 PostgreSQL 的 Azure 数据库：
1. 在 Azure 门户的左上角单击“创建资源”。
2. 从“新建”页中选择“数据库”，并从“数据库”页中选择“用于 PostgreSQL 的 Azure 数据库”。
3. 对于部署选项，请单击“Hyperscale (Citus)服务器组 - 预览版”下的“创建”按钮。
4. 使用以下信息填写“新服务器详细信息”窗体：
   - 资源组：单击此字段的文本框下的“新建”链接。 输入一个名称，例如 **myresourcegroup**。
   - 服务器组名称：输入新服务器组的唯一名称，该名称也将用于服务器子域。
   - 管理员用户名：输入唯一的用户名，稍后将使用它来连接数据库。
   - 密码：长度必须至少为八个字符，且必须包含以下类别中三种类别的字符 - 英文大写字母、英文小写字母、数字 (0-9) 和非字母数字字符（!、$、#、% 等）
   - 位置：使用距离你的用户最近的位置，使用户可以最快速度访问数据。

   > [!IMPORTANT]
   > 在此处指定的服务器管理员登录名和密码是你以后在本快速入门中登录到服务器及其数据库所必需的。 请牢记或记录此信息，以后会使用到它。

5. 单击“配置服务器组”。 保留该部分的设置不变，单击“保存”。
6. 单击“查看 + 创建”，然后单击“创建”，预配服务器。 预配需要数分钟。
7. 页面会重定向，以监视部署。 当实时状态从“部署正在进行”变为“部署已完成”时，单击页面左侧的“输出”菜单项。
8. 输出页将包含协调器主机名，主机名旁边有一个按钮，用于将值复制到剪贴板。 记录此信息以供将来使用。

## <a name="configure-a-server-level-firewall-rule"></a>配置服务器级防火墙规则

Azure Database for PostgreSQL – Hyperscale (Citus)（预览版）服务在服务器级别使用防火墙。 默认情况下，防火墙会阻止所有外部应用程序和工具连接到内部的协调器节点和任何数据库。 我们必须添加一个规则，针对特定的 IP 地址范围打开防火墙。

1. 从你之前复制协调器节点主机名的“输出”部分，单击“概述”菜单项。

2. 部署的缩放组的名称带有“sg-”前缀。 在资源列表中找到该组并单击它。

3. 在左侧菜单中的“安全性”下方单击“防火墙”。

4. 单击“+ 针对当前客户端 IP 地址添加防火墙规则”链接。 最后，单击“保存”按钮。

5. 单击“ **保存**”。

   > [!NOTE]
   > Azure PostgreSQL 服务器通过端口 5432 进行通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 5432 的出站流量。 若是如此，则无法连接到 Azure SQL 数据库服务器，除非 IT 部门启用了端口 5432。
   >

## <a name="connect-to-the-database-using-psql-in-cloud-shell"></a>在 Cloud Shell 中使用 psql 连接到数据库

现在，使用 [psql](https://www.postgresql.org/docs/current/app-psql.html) 命令行实用工具连接到 Azure Database for PostgreSQL 服务器。
1. 通过顶部导航窗格中的终端图标启动 Azure Cloud Shell。

   ![用于 PostgreSQL 的 Azure 数据库 - Azure Cloud Shell 终端图标](./media/quickstart-create-hyperscale-portal/psql-cloud-shell.png)

2. Azure Cloud Shell 会在浏览器中打开，并允许键入 bash 命令。

   ![用于 PostgreSQL 的 Azure 数据库 - Azure Shell Bash 提示符](./media/quickstart-create-hyperscale-portal/psql-bash.png)

3. 在 Cloud Shell 提示符下，使用 psql 命令连接到“用于 PostgreSQL 的 Azure 数据库”服务器。 借助 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 实用工具可以使用以下格式连接到用于 PostgreSQL 的 Azure 数据库：
   ```bash
   psql --host=<myserver> --username=myadmin --dbname=citus
   ```

   例如，以下命令使用访问凭据连接到 PostgreSQL 服务器 **mydemoserver.postgres.database.azure.com** 上名为 **citus** 的默认数据库。 在出现提示时输入服务器管理员密码。

   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --username=myadmin --dbname=citus
   ```

## <a name="create-and-distribute-tables"></a>创建和分布表

使用 psql 连接到 Hyperscale 协调器节点后，可以完成一些基本任务。

Hyperscale 服务器中有三种类型的表：

- 分布式表或分片表（分散在不同的位置以帮助进行缩放，从而提高性能并便于并行化）
- 引用表（保留多个副本）
- 本地表（通常用于内部管理表）

本快速入门重点介绍分布式表，以帮助读者熟悉此类表。

我们要使用的数据模型非常简单：来自 GitHub 的用户和事件数据。 事件包括分叉创建、组织相关的 git 提交，等等。

通过 psql 建立连接后，让我们创建表。 在 psql 控制台中运行：

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

`github_events` 的 `payload` 字段包含 JSONB 数据类型。 JSONB 是 Postgres 中采用二进制格式的 JSON 数据类型。 这可以方便在单个列中存储更灵活的架构。

Postgres 可以基于此类型创建 `GIN` 索引，以便为表中的每个键和值编制索引。 通过索引可以使用各种条件快速轻松地查询有效负载。 在加载数据之前，让我们继续创建几个索引。 在 psql 中：

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

接下来，我们将这些 Postgres 表放到协调器节点上，并告知 Hyperscale 在工作器节点之间将这些表分片。 为此，我们将针对每个表运行一个查询，指定将该表分片所依据的键。 在当前示例中，我们要将 `user_id` 上的事件和用户表分片：

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

现已准备好加载数据。 下载两个示例文件：[users.csv](https://examples.citusdata.com/users.csv) 和 [events.csv](https://examples.citusdata.com/events.csv)。 下载文件后，使用 psql 连接到数据库，请注意要从包含所下载文件的目录运行 psql。 使用 `\copy` 命令加载数据：

```sql
\copy github_events from 'events.csv' WITH CSV
\copy github_users from 'users.csv' WITH CSV
```

## <a name="run-queries"></a>运行查询

接下来的操作比较有趣，但实际上只是运行几个查询。 让我们从一个简单的 `count (*)` 着手，以查看加载的数据量：

```sql
SELECT count(*) from github_events;
```

查询非常顺利。 稍后我们会花点时间来了解聚合，但我们先看看其他几个查询。 JSONB `payload` 列中包含许多数据，但具体的数据根据事件类型而异。 `PushEvent` 事件包含一个大小，它反映了推送操作的非重复提交次数。 我们可以使用该大小找到每小时的提交总次数：

```sql
SELECT date_trunc('hour', created_at) AS hour,
       sum((payload->>'distinct_size')::int) AS num_commits
FROM github_events
WHERE event_type = 'PushEvent'
GROUP BY hour
ORDER BY hour;
```

到目前为止，查询只是涉及到 github\_events，但我们可将这些信息与 github\_users 合并。 由于我们基于同一个标识符 (`user_id`) 分片了用户和事件，因此，这两个表中具有匹配用户 ID 的行将[共置](https://docs.citusdata.com/en/stable/sharding/data_modeling.html#colocation)到同一个数据库节点，并可以轻松联接。

如果在 `user_id` 上联接，则 Hyperscale 可将联接执行下推到分片中，以便在工作器节点上并行执行。 例如，让我们查找创建了最多存储库的用户：

```sql
SELECT login, count(*)
FROM github_events ge
JOIN github_users gu
ON ge.user_id = gu.user_id
WHERE event_type = 'CreateEvent' AND
      payload @> '{"ref_type": "repository"}'
GROUP BY login
ORDER BY count(*) DESC;
```

## <a name="clean-up-resources"></a>清理资源

在前面的步骤中，你已在服务器组中创建了 Azure 资源。 如果你认为以后不需要这些资源，请删除该服务器组。 在服务器组的“概述”页中，按“删除”按钮。 弹出页面上出现提示时，请确认服务器组的名称，然后单击最后一个“删除”按钮。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解如何预配 Hyperscale (Citus) 服务器组。 你已使用 psql 连接到该组，创建了架构并分布了数据。

接下来，请遵循以下教程生成可缩放的多租户应用程序。
> [!div class="nextstepaction"]
> [设计多租户数据库](https://aka.ms/hyperscale-tutorial-multi-tenant)
