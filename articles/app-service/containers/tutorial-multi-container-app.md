---
title: 在 Web 应用中为容器创建多容器应用 - Azure 应用服务
description: 了解如何在 Azure 上将多个容器与 Docker Compose、WordPress 和 MySQL 配合使用。
keywords: azure 应用服务, web 应用, linux, docker compose, 多容器, 用于容器的 web 应用, 多个容器, 容器, wordpress, azure db for mysql, 包含容器的生产数据库
services: app-service
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/29/2019
ms.author: msangapu
ms.openlocfilehash: d4a93bbd823db59d688f77a626b1225519fb7b44
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407618"
---
# <a name="tutorial-create-a-multi-container-preview-app-in-web-app-for-containers"></a>教程：在用于容器的 Web 应用中创建多容器（预览版）应用

在[用于容器的 Web 应用](app-service-linux-intro.md)中可以灵活使用 Docker 映像。 本教程介绍如何使用 WordPress 和 MySQL 创建多容器应用。 你将在 Cloud Shell 中完成本教程，但是也可以使用 [Azure CLI](/cli/azure/install-azure-cli) 命令行工具（2.0.32 或更高版本）在本地运行这些命令。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 转换 Docker Compose 配置以使用用于容器的 Web 应用
> * 将多容器应用部署到 Azure
> * 添加应用程序设置
> * 对容器使用持久性存储
> * 连接到 Azure Database for MySQL
> * 排查错误

[!INCLUDE [Free trial note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

若要完成本教程，需要具有使用 [Docker Compose](https://docs.docker.com/compose/) 的经验。

## <a name="download-the-sample"></a>下载示例

本教程使用 [Docker](https://docs.docker.com/compose/wordpress/#define-the-project) 中的 compose 文件，但我们将对其进行修改，以包含 Azure Database for MySQL、持久性存储和 Redis。 可在 [Azure 示例](https://github.com/Azure-Samples/multicontainerwordpress)中找到该配置文件。

[!code-yml[Main](../../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

有关受支持的配置选项，请参阅 [Docker Compose 选项](configure-custom-container.md#docker-compose-options)。

在 Cloud Shell 中，创建一个 tutorial 目录，然后切换到该目录。

```bash
mkdir tutorial

cd tutorial
```

接下来，运行以下命令将示例应用存储库克隆到 tutorial 目录。 然后切换到 `multicontainerwordpress` 目录。

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>创建资源组

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

在 Cloud Shell 中，使用 [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) 命令创建资源组。 下面的示例命令在“美国中南部”位置创建名为 *myResourceGroup* 的资源组。 若要查看**标准**层中 Linux 上的应用服务支持的所有位置，请运行 [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice?view=azure-cli-latest#az-appservice-list-locations) 命令。

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

通常在附近的区域中创建资源组和资源。

此命令完成后，JSON 输出会显示资源组属性。

## <a name="create-an-azure-app-service-plan"></a>创建 Azure 应用服务计划

在 Cloud Shell 中，使用 [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) 命令在资源组中创建应用服务计划。

<!-- [!INCLUDE [app-service-plan](app-service-plan-linux.md)] -->

以下示例在**标准**定价层 (`--sku S1`) 和 Linux 容器 (`--is-linux`) 中创建名为 `myAppServicePlan` 的应用服务计划。

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

创建应用服务计划后，Cloud Shell 会显示类似于以下示例的信息。

```json
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  < JSON data removed for brevity. >
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
```

### <a name="docker-compose-with-wordpress-and-mysql-containers"></a>使用 WordPress 和 MySQL 容器的 Docker Compose

## <a name="create-a-docker-compose-app"></a>创建 Docker Compose 应用

在 Cloud Shell 中，使用 [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 命令在 `myAppServicePlan` 应用服务计划中创建一个多容器 [Web 应用](app-service-linux-intro.md)。 不要忘记将 \<app-name> 替换为唯一的应用名称。

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

创建 Web 应用后，Cloud Shell 会显示类似于以下示例的输出：

```json
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.azurewebsites.net",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

### <a name="browse-to-the-app"></a>浏览到应用

浏览到所部署的应用 (`http://<app-name>.azurewebsites.net`)。 该应用可能需要几分钟时间才能完成加载。 如果收到错误，请在几分钟后刷新浏览器。 如果遇到问题并想要进行故障排除，请查看[容器日志](#find-docker-container-logs)。

![用于容器的 Web 应用中的示例多容器应用][1]

**祝贺你**，现已在用于容器的 Web 应用中创建了多容器应用。 接下来，请将应用配置为使用 Azure Database for MySQL。 暂时不要安装 WordPress。

## <a name="connect-to-production-database"></a>连接到生产数据库

不建议在生产环境中使用数据库容器。 本地容器不可缩放。 应改用可缩放的 Azure Database for MySQL。

### <a name="create-an-azure-database-for-mysql-server"></a>创建 Azure Database for MySQL 服务器

使用 [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create) 命令创建 Azure Database for MySQL 服务器。

在以下命令中，请将 &lt;mysql-server-name> 占位符替换为你的 MySQL 服务器名称（有效字符是 `a-z`、`0-9` 和 `-`）。 此名称是 MySQL 服务器主机名 (`<mysql-server-name>.database.windows.net`) 的一部分，必须全局唯一。

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql-server-name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen4_1 --version 5.7
```

创建服务器可能需要几分钟才能完成。 创建 MySQL 服务器后，Cloud Shell 会显示类似于以下示例的信息：

```json
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "<mysql-server-name>.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql-server-name>",
  "location": "southcentralus",
  "name": "<mysql-server-name>",
  "resourceGroup": "myResourceGroup",
  ...
}
```

### <a name="configure-server-firewall"></a>配置服务器防火墙

使用 [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule?view=azure-cli-latest#az-mysql-server-firewall-rule-create) 命令创建 MySQL 服务器的防火墙规则，以便建立客户端连接。 若同时将起始 IP 和结束 IP 设置为 0.0.0.0，防火墙将仅对其他 Azure 资源开启。

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> 你甚至可以让防火墙规则更严格，即[只使用应用所使用的出站 IP 地址](../overview-inbound-outbound-ips.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips)。
>

### <a name="create-the-wordpress-database"></a>创建 WordPress 数据库

```azurecli-interactive
az mysql db create --resource-group myResourceGroup --server-name <mysql-server-name> --name wordpress
```

创建数据库后，Cloud Shell 会显示类似于以下示例的信息：

```json
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql-server-name>/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
```

### <a name="configure-database-variables-in-wordpress"></a>在 WordPress 中配置数据库变量

若要将 WordPress 应用连接到这个新的 MySQL 服务器，请配置几个特定于 WordPress 的环境变量，包括 `MYSQL_SSL_CA` 定义的 SSL CA 路径。 以下[自定义映像](https://docs.microsoft.com/azure/app-service/containers/tutorial-multi-container-app#use-a-custom-image-for-mysql-ssl-and-other-configurations)中提供了来自 [DigiCert](https://www.digicert.com/) 的 [Baltimore 网络信任根证书](https://www.digicert.com/digicert-root-certificates.htm)。

若要进行这些更改，请在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 应用设置区分大小写，用空格分开。

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WORDPRESS_DB_HOST="<mysql-server-name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql-server-name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

创建应用设置后，Cloud Shell 会显示类似于以下示例的信息。

```json
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "<mysql-server-name>.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql-server-name>"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  },
  {
    "name": "MYSQL_SSL_CA",
    "slotSetting": false,
    "value": "BaltimoreCyberTrustroot.crt.pem"
  }
]
```

有关环境变量的详细信息，请参阅[配置环境变量](configure-custom-container.md#configure-environment-variables)。

### <a name="use-a-custom-image-for-mysql-ssl-and-other-configurations"></a>对 MySQL SSL 和其他配置使用自定义映像

默认情况下，SSL 由 Azure Database for MySQL 使用。 WordPress 需要附加的配置才能将 SSL 和 MySQL 配合使用。 WordPress“官方映像”不提供附加的配置，但出于方便，已准备了一个[自定义映像](https://github.com/Azure-Samples/multicontainerwordpress)。 在实践中，请将所需的更改添加到自己的映像。

自定义映像基于 [Docker 中心的 WordPress](https://hub.docker.com/_/wordpress/) 的“官方映像”。 适用于 Azure Database for MySQL 的此自定义映像中已做出以下更改：

* [将适用于 SSL 的 Baltimore 网络信任根证书文件添加到 MySQL。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L61)
* [在 WordPress wp-config.php 中使用 MySQL SSL 证书颁发机构证书的应用设置。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L163)
* [为 MySQL SSL 所需的 MYSQL_CLIENT_FLAGS 添加 WordPress 定义。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L164)

为 Redis 做出了以下更改（在稍后的部分中使用）：

* [添加 Redis v4.0.2 的 PHP 扩展。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [添加解压缩功能以实现文件解压缩。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [添加 Redis 对象缓存 1.3.8 WordPress 插件。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [在 WordPress wp-config.php 中使用 Redis 主机名的应用设置。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

若要使用自定义映像，请更新 docker-compose-wordpress.yml 文件。 在 Cloud Shell 中，键入 `nano docker-compose-wordpress.yml` 打开 nano 文本编辑器。 更改 `image: wordpress` 以使用 `image: microsoft/multicontainerwordpress`。 不再需要数据库容器。 从配置文件中删除 `db`、`environment`、`depends_on` 和 `volumes` 节。 文件应如以下代码所示：

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always
```

保存更改并退出 nano。 使用命令 `^O` 来保存，使用 `^X` 来退出。

### <a name="update-app-with-new-configuration"></a>使用新配置更新应用

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新配置多容器 [Web 应用](app-service-linux-intro.md)。 不要忘记将 \<app-name> 替换为前面创建的 Web 应用的名称。

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

重新配置应用后，Cloud Shell 会显示类似于以下示例的信息：

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICB3b3JkcHJlc3M6CiAgICAgaW1hZ2U6IG1zYW5nYXB1L3dvcmRwcmVzcwogICAgIHBvcnRzOgogICAgICAgLSAiODAwMDo4MCIKICAgICByZXN0YXJ0OiBhbHdheXM="
  }
]
```

### <a name="browse-to-the-app"></a>浏览到应用

浏览到所部署的应用 (`http://<app-name>.azurewebsites.net`)。 应用正在使用 Azure Database for MySQL。

![用于容器的 Web 应用中的示例多容器应用][1]

## <a name="add-persistent-storage"></a>添加持久性存储

多容器应用正在用于容器的 Web 应用中运行。 但是，如果现在安装 WordPress 并稍后重启应用，则会发现 WordPress 安装已消失。 之所以发生这种情况，是因为 Docker Compose 配置当前指向容器中的存储位置。 重启应用后，在容器中安装的文件不会保留。 在本部分，我们将向 WordPress 容器[添加持久性存储](configure-custom-container.md#use-persistent-shared-storage)。

### <a name="configure-environment-variables"></a>配置环境变量

若要使用持久性存储，请在应用服务中启用此设置。 若要进行此项更改，请在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 应用设置区分大小写，用空格分开。

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

创建应用设置后，Cloud Shell 会显示类似于以下示例的信息。

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
```

### <a name="modify-configuration-file"></a>修改配置文件

在 Cloud Shell 中，键入 `nano docker-compose-wordpress.yml` 以打开 nano 文本编辑器。

`volumes` 选项将文件系统映射到容器中的某个目录。 `${WEBAPP_STORAGE_HOME}` 是应用服务中已映射到应用持续性存储的环境变量。 将在 volumes 选项中使用此环境变量，以便将 WordPress 文件安装到持久性存储，而不是安装到容器。 对文件做出以下修改：

如以下代码所示，在 `wordpress` 节中添加 `volumes` 选项：

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     volumes:
      - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
     ports:
       - "8000:80"
     restart: always
```

### <a name="update-app-with-new-configuration"></a>使用新配置更新应用

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新配置多容器 [Web 应用](app-service-linux-intro.md)。 不要忘记将 \<app-name> 替换为唯一的应用名称。

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

该命令运行后，会显示类似于以下示例的输出：

```json
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>浏览到应用

浏览到所部署的应用 (`http://<app-name>.azurewebsites.net`)。

WordPress 容器正在使用 Azure Database for MySQL 和持久性存储。

## <a name="add-redis-container"></a>添加 Redis 容器

 WordPress“官方映像”不包含 Redis 的依赖项。 此[自定义映像](https://github.com/Azure-Samples/multicontainerwordpress)中准备好这些依赖项和所需的附加配置，以便用户能够配合 WordPress 使用 Redis。 在实践中，请将所需的更改添加到自己的映像。

自定义映像基于 [Docker 中心的 WordPress](https://hub.docker.com/_/wordpress/) 的“官方映像”。 适用于 Redis 的此自定义映像中已做出以下更改：

* [添加 Redis v4.0.2 的 PHP 扩展。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [添加解压缩功能以实现文件解压缩。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [添加 Redis 对象缓存 1.3.8 WordPress 插件。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [在 WordPress wp-config.php 中使用 Redis 主机名的应用设置。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

如以下示例所示，将 Redis 容器添加到配置文件的底部：

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always

   redis:
     image: redis:3-alpine
     restart: always
```

### <a name="configure-environment-variables"></a>配置环境变量

若要使用 Redis，请在应用服务中启用 `WP_REDIS_HOST` 设置。 WordPress 在与 Redis 主机通信时必须使用此设置。 若要进行此项更改，请在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 应用设置区分大小写，用空格分开。

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WP_REDIS_HOST="redis"
```

创建应用设置后，Cloud Shell 会显示类似于以下示例的信息。

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql-server-name>"
  },
  {
    "name": "WP_REDIS_HOST",
    "slotSetting": false,
    "value": "redis"
  }
]
```

### <a name="update-app-with-new-configuration"></a>使用新配置更新应用

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新配置多容器 [Web 应用](app-service-linux-intro.md)。 不要忘记将 \<app-name> 替换为唯一的应用名称。

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

该命令运行后，会显示类似于以下示例的输出：

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>浏览到应用

浏览到所部署的应用 (`http://<app-name>.azurewebsites.net`)。

完成上述步骤并安装 WordPress。

### <a name="connect-wordpress-to-redis"></a>将 WordPress 连接到 Redis

以管理员身份登录到 WordPress。在左侧导航栏中选择“插件”，然后选择“安装插件”。

![选择 WordPress 插件][2]

此处显示了所有插件

在插件页中，找到“Redis 对象缓存”并单击“激活”。

![激活 Redis][3]

单击“设置” 。

![单击“设置”][4]

单击“启用对象缓存”按钮。

![单击“启用对象缓存”按钮][5]

WordPress 将连接到 Redis 服务器。 同一页面上会显示连接**状态**。

![WordPress 将连接到 Redis 服务器。 同一页面上会显示连接**状态**。][6]

**祝贺你**，现已将 WordPress 连接到 Redis。 生产就绪的应用正在使用 **Azure Database for MySQL、持久性存储和 Redis**。 现在可以扩展应用服务计划以包含多个实例。

## <a name="find-docker-container-logs"></a>查找 Docker 容器日志

如果使用多个容器时遇到问题，可以通过浏览到 `https://<app-name>.scm.azurewebsites.net/api/logs/docker` 来访问容器日志。

将会看到类似于以下示例的输出：

```json
[
   {
      "machineName":"RD00XYZYZE567A",
      "lastUpdated":"2018-05-10T04:11:45Z",
      "size":25125,
      "href":"https://<app-name>.scm.azurewebsites.net/api/vfs/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log",
      "path":"/home/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log"
   }
]
```

查看每个容器的日志，以及父进程的附加日志。 将相应的 `href` 值复制到浏览器以查看日志。

[!INCLUDE [Clean-up section](../../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：
> [!div class="checklist"]
> * 转换 Docker Compose 配置以使用用于容器的 Web 应用
> * 将多容器应用部署到 Azure
> * 添加应用程序设置
> * 对容器使用持久性存储
> * 连接到 Azure Database for MySQL
> * 排查错误

继续学习下一篇教程，了解如何将自定义 DNS 名称映射到应用。

> [!div class="nextstepaction"]
> [教程：将自定义 DNS 名称映射到应用](../app-service-web-tutorial-custom-domain.md)

或者，查看其他资源：

> [!div class="nextstepaction"]
> [配置自定义容器](configure-custom-container.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
[2]: ./media/tutorial-multi-container-app/wordpress-plugins.png
[3]: ./media/tutorial-multi-container-app/activate-redis.png
[4]: ./media/tutorial-multi-container-app/redis-settings.png
[5]: ./media/tutorial-multi-container-app/enable-object-cache.png
[6]: ./media/tutorial-multi-container-app/redis-connected.png
