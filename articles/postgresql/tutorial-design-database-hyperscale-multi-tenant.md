---
title: 使用 Azure Database for PostgreSQL 设计多租户数据库 – 超大规模 (Citus)（预览）教程
description: 本教程演示如何创建、填充和查询 Azure Database for PostgreSQL 超大规模 (Citus)（预览版）上的分布式表。
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 05/06/2019
ms.openlocfilehash: b135baf73e21cd524b6e8fad35452362f36cf0c0
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079542"
---
# <a name="tutorial-design-a-multi-tenant-database-by-using-azure-database-for-postgresql--hyperscale-citus-preview"></a>教程：通过使用 Azure Database for PostgreSQL - 超大规模 (Citus)（预览版）设计多租户数据库

在本教程中，你使用 Azure Database for PostgreSQL - 超大规模 (Citus)（预览版）了解如何：

> [!div class="checklist"]
> * 创建超大规模 (Citus) 服务器组
> * 使用 psql 实用程序来创建架构
> * 节点中的分片表
> * 引入示例数据
> * 查询租户数据
> * 租户之间共享数据
> * 自定义每租户架构

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-an-azure-database-for-postgresql"></a>创建用于 PostgreSQL 的 Azure 数据库

可以按照以下步骤创建用于 PostgreSQL 的 Azure 数据库：
1. 在 Azure 门户的左上角单击“创建资源”。
2. 从“新建”页中选择“数据库”，并从“数据库”页中选择“用于 PostgreSQL 的 Azure 数据库”。
3. 对于部署选项，请单击“超大规模 (Citus) 服务器组 - 预览”下的“创建”按钮。
4. 使用以下信息填写“新服务器详细信息”窗体：
   - 资源组：单击此字段的文本框下的“新建”链接。 输入一个名称，例如 myresourcegroup。
   - 服务器组名称：输入新服务器组的唯一名称，该名称也将用于服务器子域。
   - 管理员用户名：输入一个唯一用户名，它将稍后用于连接数据库。
   - 密码：长度必须至少为八个字符，且必须包含以下类别中三种类别的字符 - 英文大写字母、英文小写字母、数字 (0-9) 和非字母数字字符（!、$、#、% 等）
   - 位置：使用距离你的用户最近的位置，使用户可以最快速度访问数据。

   > [!IMPORTANT]
   > 此处指定的服务器管理员登录名和密码是以后在本教程中登录到服务器及其数据库所必需的。 请牢记或记录此信息，以后会使用到它。

5. 单击“配置服务器组”。 保留该部分的设置不变，单击“保存”。
6. 单击“查看 + 创建”，然后单击“创建”，预配服务器。 预配需要数分钟。
7. 页面会重定向，以监视部署。 当实时状态从“部署正在进行”变为“部署已完成”时，单击页面左侧的“输出”菜单项。
8. 输出页将包含协调器主机名，主机名旁边有一个按钮，用于将值复制到剪贴板。 记录此信息以供将来使用。

## <a name="configure-a-server-level-firewall-rule"></a>配置服务器级防火墙规则

Azure Database for PostgreSQL 服务在服务器级别使用防火墙。 默认情况下，防火墙会阻止所有外部应用程序和工具连接到服务器和服务器上的任何数据库。 我们必须添加一个规则，针对特定的 IP 地址范围打开防火墙。

1. 从你之前复制协调器节点主机名的“输出”部分，单击“概述”菜单项。

2. 在资源列表中找到部署的缩放组，并单击它。 （其名称的前缀是“sg-”。）

3. 在左侧菜单中的“安全性”下方单击“防火墙”。

4. 单击“+ 为当前客户端 IP 地址添加防火墙”链接。 最后，单击“保存”按钮。

5. 单击“ **保存**”。

   > [!NOTE]
   > Azure PostgreSQL 服务器通过端口 5432 进行通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 5432 的出站流量。 若是如此，则无法连接到 Azure SQL 数据库服务器，除非 IT 部门启用了端口 5432。
   >

## <a name="connect-to-the-database-using-psql-in-cloud-shell"></a>在 Cloud Shell 中使用 psql 连接到数据库

现在，使用 [psql](https://www.postgresql.org/docs/current/app-psql.html) 命令行实用工具连接到 Azure Database for PostgreSQL 服务器。
1. 通过顶部导航窗格中的终端图标启动 Azure Cloud Shell。

   ![用于 PostgreSQL 的 Azure 数据库 - Azure Cloud Shell 终端图标](./media/tutorial-design-database-hyperscale-multi-tenant/psql-cloud-shell.png)

2. Azure Cloud Shell 会在浏览器中打开，并允许键入 bash 命令。

   ![用于 PostgreSQL 的 Azure 数据库 - Azure Shell Bash 提示符](./media/tutorial-design-database-hyperscale-multi-tenant/psql-bash.png)

3. 在 Cloud Shell 提示符下，使用 psql 命令连接到“用于 PostgreSQL 的 Azure 数据库”服务器。 借助 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 实用工具可以使用以下格式连接到用于 PostgreSQL 的 Azure 数据库：
   ```bash
   psql --host=<myserver> --username=myadmin --dbname=citus
   ```

   例如，以下命令使用访问凭据连接到 PostgreSQL 服务器 mydemoserver.postgres.database.azure.com 上名为“citus”的默认数据库。 在出现提示时输入服务器管理员密码。

   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --username=myadmin --dbname=citus
   ```

## <a name="use-psql-utility-to-create-a-schema"></a>使用 psql 实用程序来创建架构

使用 psql 连接到 Azure Database for PostgreSQL - 超大规模 (Citus)（预览版）后，可以完成一些基本任务。 本教程将指导你创建 Web 应用，该应用允许广告厂商跟踪他们的广告系列。

多家公司可以使用该应用，让我们创建一个表来记录这些公司，创建另一个表来记录他们的广告系列。 在 psql 控制台中，运行这些命令：

```sql
CREATE TABLE companies (
  id bigserial PRIMARY KEY,
  name text NOT NULL,
  image_url text,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL
);

CREATE TABLE campaigns (
  id bigserial,
  company_id bigint REFERENCES companies (id),
  name text NOT NULL,
  cost_model text NOT NULL,
  state text NOT NULL,
  monthly_budget bigint,
  blacklisted_site_urls text[],
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id)
);
```

每个广告系列会支付运行广告的费用。 此外，添加用于广告的一个表，方法是在以上代码后，运行以下代码：

```sql
CREATE TABLE ads (
  id bigserial,
  company_id bigint,
  campaign_id bigint,
  name text NOT NULL,
  image_url text,
  target_url text,
  impressions_count bigint DEFAULT 0,
  clicks_count bigint DEFAULT 0,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, campaign_id)
    REFERENCES campaigns (company_id, id)
);
```

最后，我们将跟踪有关每个广告的点击次数和展示次数的统计信息：

```sql
CREATE TABLE clicks (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  clicked_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_click_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);

CREATE TABLE impressions (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  seen_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_impression_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);
```

现在，你可以在 psql 中看到表列表中新建的表，方法是运行：

```postgres
\dt
```

多租户应用程序仅可以对每个租户强制实施唯一性，因此所有主键和外键包含公司 ID。

## <a name="shard-tables-across-nodes"></a>节点中的分片表

超大规模部署基于用户指定的列的值存储不同节点上的表行。 此“分布列”标记哪个租户拥有哪些行。

让我们将分布列设置为 company\_id，即租户标识符。 在 Psql 中，运行这些函数：

```sql
SELECT create_distributed_table('companies',   'id');
SELECT create_distributed_table('campaigns',   'company_id');
SELECT create_distributed_table('ads',         'company_id');
SELECT create_distributed_table('clicks',      'company_id');
SELECT create_distributed_table('impressions', 'company_id');
```

## <a name="ingest-sample-data"></a>引入示例数据

现在，在 psql 外部，在常规命令行中，下载示例数据集：

```bash
for dataset in companies campaigns ads clicks impressions geo_ips; do
  curl -O https://examples.citusdata.com/mt_ref_arch/${dataset}.csv
done
```

返回到 psql 内部，大容量加载数据。 请务必在下载数据文件的目录中运行 psql。

```sql
\copy companies from 'companies.csv' with csv
\copy campaigns from 'campaigns.csv' with csv
\copy ads from 'ads.csv' with csv
\copy clicks from 'clicks.csv' with csv
\copy impressions from 'impressions.csv' with csv
```

现在，此数据将跨工作器节点分布。

## <a name="query-tenant-data"></a>查询租户数据

当应用程序请求单个租户的数据时，数据库可以在单个工作器节点上执行查询。 单租户查询根据单个租户 ID 进行筛选。 例如，以下查询针对 `company_id = 5` 筛选广告和展示次数。 尝试在 psql 中运行它并查看结果。

```sql
SELECT a.campaign_id,
       RANK() OVER (
         PARTITION BY a.campaign_id
         ORDER BY a.campaign_id, count(*) desc
       ), count(*) as n_impressions, a.id
  FROM ads as a
  JOIN impressions as i
    ON i.company_id = a.company_id
   AND i.ad_id      = a.id
 WHERE a.company_id = 5
GROUP BY a.campaign_id, a.id
ORDER BY a.campaign_id, n_impressions desc;
```

## <a name="share-data-between-tenants"></a>租户之间共享数据

到目前为止，已按照 `company_id` 分发所有表，但一些数据不会“自然地”属于任何租户，并且这些数据可以共享。 例如，示例广告平台中的所有公司可能需要获取基于 IP 地址的受众地理位置信息。

创建一个表来记录共享地理位置信息。 在 psql 中运行它：

```sql
CREATE TABLE geo_ips (
  addrs cidr NOT NULL PRIMARY KEY,
  latlon point NOT NULL
    CHECK (-90  <= latlon[0] AND latlon[0] <= 90 AND
           -180 <= latlon[1] AND latlon[1] <= 180)
);
CREATE INDEX ON geo_ips USING gist (addrs inet_ops);
```

接下来，将 `geo_ips` 设置为“引用表”，存储每个工作器节点上的表副本。

```sql
SELECT create_reference_table('geo_ips');
```

将它和示例数据一同加载。 请记住要从下载数据集的目录内部运行它。

```sql
\copy geo_ips from 'geo_ips.csv' with csv
```

将点击次数表和 geo\_ips 联接，这在所有节点上都是高效的。
下面是查找点击广告的所有人的位置的联接
290. 请尝试在 psql 中运行查询。

```sql
SELECT c.id, clicked_at, latlon
  FROM geo_ips, clicks c
 WHERE addrs >> c.user_ip
   AND c.company_id = 5
   AND c.ad_id = 290;
```

## <a name="customize-the-schema-per-tenant"></a>自定义每租户架构

每个租户可能需要存储他人不需要的特殊信息。 但是，所有租户都共享具有相同数据库架构的同一基础结构。 额外的数据可以到哪儿去？

一个技巧是使用开放式列类型，例如 PostgreSQL 的 JSONB。  我们的架构在 `clicks` 中有一个 JSONB 字段，称为 `user_data`。
一家公司（例如 company five）可以使用列来跟踪用户是否使用移动设备。

以下查询用于发现是移动设备访客还是传统访客的点击次数更高。

```sql
SELECT
  user_data->>'is_mobile' AS is_mobile,
  count(*) AS count
FROM clicks
WHERE company_id = 5
GROUP BY user_data->>'is_mobile'
ORDER BY count DESC;
```

可以为单个公司优化此查询，方法是创建[部分索引](https://www.postgresql.org/docs/current/static/indexes-partial.html)。

```sql
CREATE INDEX click_user_data_is_mobile
ON clicks ((user_data->>'is_mobile'))
WHERE company_id = 5;
```

更概括地说，我们可以在列中的每个键和值上创建 [GIN 索引](https://www.postgresql.org/docs/current/static/gin-intro.html)。

```sql
CREATE INDEX click_user_data
ON clicks USING gin (user_data);

-- this speeds up queries like, "which clicks have
-- the is_mobile key present in user_data?"

SELECT id
  FROM clicks
 WHERE user_data ? 'is_mobile'
   AND company_id = 5;
```

## <a name="clean-up-resources"></a>清理资源

在前面的步骤中，已在服务器组中创建 Azure 资源。 如果你认为以后不需要这些资源，请删除服务器组。 在你的服务器组的“概述”页面，按“删除”按钮。 弹出页面上出现提示时，确认服务器组的名称，然后单击最后一个“删除”按钮。

## <a name="next-steps"></a>后续步骤

在本教程中，你学习了如何预配超大规模 (Citus) 服务器组。 使用 psql 连接它，创建架构和分布式数据。 你已了解如何在租户中和租户之间查询数据，以及如何自定义每租户架构。

接下来，了解有关超大规模的概念。
> [!div class="nextstepaction"]
> [超大规模节点类型](https://aka.ms/hyperscale-concepts)
