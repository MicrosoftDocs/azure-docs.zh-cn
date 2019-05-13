---
title: 配置 PHP 应用程序-Azure 应用服务 |Microsoft Docs
description: 了解如何配置要在 Azure 应用服务中运行的 PHP 应用
services: app-service
documentationcenter: ''
author: cephalin
manager: jpconnock
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/28/2019
ms.author: cephalin
ms.openlocfilehash: ed6a50ee68d39e6e0d01b405eb02edd6d4c93613
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407591"
---
# <a name="configure-a-linux-php-app-for-azure-app-service"></a>为 Azure 应用服务中配置 Linux PHP 应用

本指南演示如何在 Azure 应用服务中配置 web 应用、 移动后端和 API 应用的内置 PHP 运行时。

本指南提供关键概念和 PHP 开发人员在应用服务中使用内置的 Linux 容器的说明。 如果你从未使用过 Azure 应用服务，请按照[PHP 快速入门](quickstart-php.md)并[PHP 与 MySQL 教程](tutorial-php-mysql-app.md)第一个。

## <a name="show-php-version"></a>显示 PHP 版本

若要显示当前的 PHP 版本，请运行以下命令[Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

若要显示所有受支持的 PHP 版本，请运行以下命令[Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep PHP
```

## <a name="set-php-version"></a>设置 PHP 版本

在中运行以下命令[Cloud Shell](https://shell.azure.com)将 PHP 版本设置为 7.2:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --linux-fx-version "PHP|7.2"
```

## <a name="run-composer"></a>运行编辑器

默认情况下，不会运行 Kudu[编辑器](https://getcomposer.org/)。 若要启用编辑器自动化在 Kudu 部署期间，你需要提供[自定义部署脚本](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script)。

从本地终端窗口中，将目录更改到存储库根路径。 请按照[命令行安装步骤](https://getcomposer.org/download/)若要下载*composer.phar*。

运行以下命令：

```bash
npm install kuduscript -g
kuduscript --php --scriptType bash --suppressPrompt
```

存储库根路径现在具有两个新文件，除了*composer.phar*: *.deployment*并*deploy.sh*。这些文件对 Windows 和 Linux 风格的应用服务都能正常工作。

打开*deploy.sh*并找到`Deployment`部分。 整个部分替换为以下代码：

```bash
##################################################################################################################################
# Deployment
# ----------

echo PHP deployment

# 1. KuduSync
if [[ "$IN_PLACE_DEPLOYMENT" -ne "1" ]]; then
  "$KUDU_SYNC_CMD" -v 50 -f "$DEPLOYMENT_SOURCE" -t "$DEPLOYMENT_TARGET" -n "$NEXT_MANIFEST_PATH" -p "$PREVIOUS_MANIFEST_PATH" -i ".git;.hg;.deployment;deploy.sh"
  exitWithMessageOnError "Kudu Sync failed"
fi

# 3. Initialize Composer Config
initializeDeploymentConfig

# 4. Use composer
echo "$DEPLOYMENT_TARGET"
if [ -e "$DEPLOYMENT_TARGET/composer.json" ]; then
  echo "Found composer.json"
  pushd "$DEPLOYMENT_TARGET"
  php composer.phar install $COMPOSER_ARGS
  exitWithMessageOnError "Composer install failed"
  popd
fi
##################################################################################################################################
```

提交所有更改并重新部署你的代码。 编辑器现在应作为自动化部署的一部分运行。

## <a name="customize-start-up"></a>自定义启动

默认情况下，内置 PHP 容器运行的 Apache 服务器。 在启动时，它在运行`apache2ctl -D FOREGROUND"`。 如果你愿意，您可以运行不同的命令在启动时，通过在运行以下命令[Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
```

## <a name="access-environment-variables"></a>访问环境变量

在应用服务中，可以在应用代码外部[设置应用设置](../web-sites-configure.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#app-settings)。 然后你可以访问它们使用标准[getenv （)](https://secure.php.net/manual/function.getenv.php)模式。 例如，若要访问名为 `DB_HOST` 的应用设置，请使用以下代码：

```php
getenv("DB_HOST")
```

## <a name="change-site-root"></a>更改站点根目录

所选的 web 框架可使用作为站点根目录的子目录。 例如， [Laravel](https://laravel.com/)，使用`public/`作为站点根目录的子目录。

为应用服务的默认 PHP 映像使用 Apache，并且它不允许您自定义您的应用程序的站点根目录。 若要解决此限制，请添加 *.htaccess*文件到存储库根目录包含以下内容：

```
<IfModule mod_rewrite.c>
    RewriteEngine on

    RewriteRule ^.*$ /public/$1 [NC,L,QSA]
</IfModule>
```

如果不希望使用 *.htaccess* 重写，可以改用[自定义 Docker 映像](quickstart-docker-go.md)来部署 Laravel 应用程序。

## <a name="detect-https-session"></a>检测 HTTPS 会话

在应用服务中，[SSL 终止](https://wikipedia.org/wiki/TLS_termination_proxy)在网络负载均衡器上发生，因此，所有 HTTPS 请求将以未加密的 HTTP 请求形式访问你的应用。 如果应用逻辑需要检查用户请求是否已加密，可以检查 `X-Forwarded-Proto` 标头。

```php
if (isset($_SERVER['X-Forwarded-Proto']) && $_SERVER['X-Forwarded-Proto'] === 'https') {
  // Do something when HTTPS is used
}
```

使用常用 Web 框架可以访问采用标准应用模式的 `X-Forwarded-*` 信息。 在 [CodeIgniter](https://codeigniter.com/) 中，[is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) 默认检查 `X_FORWARDED_PROTO` 的值。

## <a name="customize-phpini-settings"></a>自定义 php.ini 设置

如果需要对你的 PHP 安装进行更改，则可以更改的任何[php.ini 指令](https://www.php.net/manual/ini.list.php)通过执行以下步骤。

> [!NOTE]
> 请参阅 PHP 版本以及当前的最好办法*php.ini*配置为在调用[phpinfo （)](https://php.net/manual/function.phpinfo.php)应用程序中。
>

### <a name="customize-non-phpinisystem-directives"></a>自定义非 PHP_INI_SYSTEM 指令

若要自定义 PHP_INI_USER、 PHP_INI_PERDIR 和 PHP_INI_ALL 指令 (请参阅[php.ini 指令](https://www.php.net/manual/ini.list.php))，添加 *.htaccess*到您的应用程序的根目录的文件。

在中 *.htaccess*文件中，将使用的指令添加`php_value <directive-name> <value>`语法。 例如:

```
php_value upload_max_filesize 1000M
php_value post_max_size 2000M
php_value memory_limit 3000M
php_value max_execution_time 180
php_value max_input_time 180
php_value display_errors On
php_value upload_max_filesize 10M
```

重新部署应用程序与所做的更改并重新启动它。 如果使用 Kudu 部署 (例如，使用[Git](../deploy-local-git.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json))，将在部署后自动重启。

为使用的替代方法 *.htaccess*，可以使用[ini_set （)](https://www.php.net/manual/function.ini-set.php)应用自定义这些非 PHP_INI_SYSTEM 指令中。

### <a name="customize-phpinisystem-directives"></a>自定义 PHP_INI_SYSTEM 指令

若要自定义 PHP_INI_SYSTEM 指令 (请参阅[php.ini 指令](https://www.php.net/manual/ini.list.php))，不能使用 *.htaccess*方法。 应用服务提供单独的机制使用`PHP_INI_SCAN_DIR`应用设置。

首先，在中运行以下命令[Cloud Shell](https://shell.azure.com)若要添加的应用设置调用`PHP_INI_SCAN_DIR`:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="/usr/local/etc/php/conf.d:/home/site/ini"
```

`/usr/local/etc/php/conf.d` 是默认的目录位置*php.ini*存在。 `/home/site/ini` 是将在其中添加自定义的自定义目录 *.ini*文件。 分隔的值与`:`。

导航到包含在 Linux 容器的 web SSH 会话 (`https://cephalin-container.scm.azurewebsites.net/webssh/host`)。

创建的目录中`/home/site`称为`ini`，然后创建 *.ini*文件中`/home/site/ini`目录 (例如， *settings.ini)* 与想要自定义的指令。 使用你将在相同的语法*php.ini*文件。 

> [!TIP]
> 在应用服务中内置的 Linux 容器中 */home*用作永久性的共享存储空间。 
>

例如，若要更改的值[expose_php](https://php.net/manual/ini.core.php#ini.expose-php)运行以下命令：

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

要使更改生效，重新启动该应用程序。

## <a name="enable-php-extensions"></a>启用 PHP 扩展插件

内置的 PHP 安装包含最常用的扩展。 您可以启用其他扩展在同一个方法，你[自定义 php.ini 指令](#customize-php_ini_system-directives)。

> [!NOTE]
> 请参阅 PHP 版本以及当前的最好办法*php.ini*配置为在调用[phpinfo （)](https://php.net/manual/function.phpinfo.php)应用程序中。
>

若要启用其他扩展，请执行下列步骤：

添加`bin`目录的应用程序和 put 的根目录`.so`扩展文件 (例如， *mongodb.so*)。 请确保扩展与 Azure 并且是 vc9 版本且与非线程安全 (nts) 兼容的 PHP 版本兼容。

部署所做的更改。

按照中的步骤[自定义 PHP_INI_SYSTEM 指令](#customize-php_ini_system-directives)，将扩展添加到自定义 *.ini*文件[扩展](https://www.php.net/manual/ini.core.php#ini.extension)或[zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension)指令。

```ini
extension=/home/site/wwwroot/bin/mongodb.so
zend_extension=/home/site/wwwroot/bin/xdebug.so
```

要使更改生效，重新启动该应用程序。

## <a name="access-diagnostic-logs"></a>访问诊断日志

[!INCLUDE [Access diagnostic logs](../../../includes/app-service-web-logs-access-no-h.md)]

## <a name="open-ssh-session-in-browser"></a>在浏览器中打开 SSH 会话

[!INCLUDE [Open SSH session in browser](../../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

## <a name="troubleshooting"></a>故障排除

正在运行的 PHP 应用的行为是不同的应用服务中或出现错误时，请尝试以下方法：

- [访问日志流](#access-diagnostic-logs)。
- 在生产模式下，本地测试应用程序。 应用服务在生产模式下运行 Node.js 应用程序，因此您需要确保你的项目按预期方式在本地生产模式下能够正常工作。 例如:
    - 具体取决于你*composer.json*，可能会为生产模式下安装不同的包 (`require`与`require-dev`)。
    - 某些 web 框架可以部署在生产模式下以不同的方式的静态文件。
    - 在生产模式下运行时，某些 web 框架可能会使用自定义启动脚本。
- 在调试模式下，在应用服务中运行你的应用。 例如，在[Laravel](https://meanjs.org/)，可以配置你的应用器输出调试的生产环境中的消息[设置`APP_DEBUG`应用设置设为`true` ](../web-sites-configure.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)。

### <a name="robots933456"></a>robots933456

可能会看到以下消息中的容器日志：

```
2019-04-08T14:07:56.641002476Z "-" - - [08/Apr/2019:14:07:56 +0000] "GET /robots933456.txt HTTP/1.1" 404 415 "-" "-"
```

您可以放心地忽略此消息。 `/robots933456.txt` 是应用服务用于检查容器是否能够为请求提供服务的虚拟 URL 路径。 404 响应只是指示该路径不存在，但它可让应用服务知道容器正常，可以对请求作出响应。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [教程：将 PHP 应用与 MySQL 配合使用](tutorial-php-mysql-app.md)

> [!div class="nextstepaction"]
> [应用服务 Linux 常见问题解答](app-service-linux-faq.md)
