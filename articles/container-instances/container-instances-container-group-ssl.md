---
title: 在 Azure 容器实例中启用 SSL
description: 创建在 Azure 容器实例中运行的容器组的 SSL 或 TLS 终结点
services: container-instances
author: dlepow
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 04/03/2019
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: 12de4ef31084d8ac8586c79ffe3d0a8e891727bf
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411387"
---
# <a name="enable-an-ssl-endpoint-in-a-container-group"></a>启用容器组中的 SSL 终结点

本文介绍如何创建[容器组](container-instances-container-groups.md)与应用程序容器和运行 SSL 提供程序的挎斗容器。 通过设置单独的 SSL 终结点的容器组，可以启用 SSL 连接用于你的应用程序而无需更改应用程序代码。

您设置了由两个容器组成的一个容器组：
* 运行一个简单的 web 应用使用公共 Microsoft 应用程序容器[aci helloworld](https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld)映像。 
* 挎斗容器运行公众[Nginx](https://hub.docker.com/_/nginx)图中，配置为使用 SSL。 

在此示例中，容器组仅端口 443 针对公开 Nginx 使用其公共 IP 地址。 Nginx 将 HTTPS 请求路由到辅助 web 应用程序，在内部侦听端口 80。 你可以调整侦听其他端口的容器应用的示例。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

可以使用 Azure Cloud Shell 或 Azure CLI 的本地安装完成本文的内容。 如果想要在本地使用它，建议使用 2.0.55 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-self-signed-certificate"></a>创建自签名证书

若要设置 Nginx 用作 SSL 提供程序，您需要一个 SSL 证书。 本文介绍如何创建和设置自签名 SSL 证书。 对于生产方案，应从证书颁发机构获取证书。

若要创建自签名的 SSL 证书，请使用[OpenSSL](https://www.openssl.org/)工具在 Azure Cloud Shell 和许多 Linux 分发版中可用或在你的操作系统中使用的可比较客户端工具。

首先在本地工作目录中创建证书请求 （.csr 文件）：

```console
openssl req -new -newkey rsa:2048 -nodes -keyout ssl.key -out ssl.csr
```

按照提示后，若要添加的标识信息。 对于公用名，请输入与证书关联的主机名。 当系统提示输入密码，而无需键入要跳过添加密码按 Enter。

运行以下命令以创建自签名的证书 （.crt 文件） 的证书请求中。 例如:

```console
openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
```

现在应看到在目录中的三个文件： 证书请求 (`ssl.csr`)，私钥 (`ssl.key`)，和自签名的证书 (`ssl.crt`)。 您使用`ssl.key`和`ssl.crt`在后续步骤中。

## <a name="configure-nginx-to-use-ssl"></a>配置 Nginx 以使用 SSL

### <a name="create-nginx-configuration-file"></a>创建 Nginx 配置文件

在本部分中，您将创建 Nginx 以使用 SSL 的配置文件。 通过将以下文本复制到名为的新文件启动`nginx.conf`。 在 Azure Cloud Shell 中，可以使用 Visual Studio Code 在工作目录中创建文件：

```console
code nginx.conf
```

在中`location`，请务必设置`proxy_pass`替换为应用程序的正确端口。 在此示例中，我们将设置为端口 80`aci-helloworld`容器。

```console
# nginx Configuration File
# https://wiki.nginx.org/Configuration

# Run as a less privileged user for security reasons.
user nginx;

worker_processes auto;

events {
  worker_connections 1024;
}

pid        /var/run/nginx.pid;

http {

    #Redirect to https, using 307 instead of 301 to preserve post data

    server {
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name localhost;

        # Protect against the BEAST attack by not using SSLv3 at all. If you need to support older browsers (IE6) you may need to add
        # SSLv3 to the list of protocols below.
        ssl_protocols              TLSv1 TLSv1.1 TLSv1.2;

        # Ciphers set to best allow protection from Beast, while providing forwarding secrecy, as defined by Mozilla - https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx
        ssl_ciphers                ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:ECDHE-RSA-RC4-SHA:ECDHE-ECDSA-RC4-SHA:AES128:AES256:RC4-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK;
        ssl_prefer_server_ciphers  on;

        # Optimize SSL by caching session parameters for 10 minutes. This cuts down on the number of expensive SSL handshakes.
        # The handshake is the most CPU-intensive operation, and by default it is re-negotiated on every new/parallel connection.
        # By enabling a cache (of type "shared between all Nginx workers"), we tell the client to re-use the already negotiated state.
        # Further optimization can be achieved by raising keepalive_timeout, but that shouldn't be done unless you serve primarily HTTPS.
        ssl_session_cache    shared:SSL:10m; # a 1mb cache can hold about 4000 sessions, so we can hold 40000 sessions
        ssl_session_timeout  24h;


        # Use a higher keepalive timeout to reduce the need for repeated handshakes
        keepalive_timeout 300; # up from 75 secs default

        # remember the certificate for a year and automatically connect to HTTPS
        add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains';

        ssl_certificate      /etc/nginx/ssl.crt;
        ssl_certificate_key  /etc/nginx/ssl.key;

        location / {
            proxy_pass http://localhost:80; # TODO: replace port if app listens on port other than 80
            
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
}
```

### <a name="base64-encode-secrets-and-configuration-file"></a>Base64 编码机密和配置文件

进行 Base64 编码 Nginx 配置文件、 SSL 证书和 SSL 密钥。 在下一部分中，输入用来部署容器组的 YAML 文件中的编码的内容。

```console
cat nginx.conf | base64 -w 0 > base64-nginx.conf
cat ssl.crt | base64 -w 0 > base64-ssl.crt
cat ssl.key | base64 -w 0 > base64-ssl.key
```

## <a name="deploy-container-group"></a>部署容器组

现在，通过指定的容器配置中部署容器组[YAML 文件](container-instances-multi-container-yaml.md)。

### <a name="create-yaml-file"></a>创建 YAML 文件

将以下 YAML 复制到名为的新文件`deploy-aci.yaml`。 在 Azure Cloud Shell 中，可以使用 Visual Studio Code 在工作目录中创建文件：

```console
code deploy-aci.yaml
```

输入的 base64 编码内容文件下的指示位置`secret`。 例如，`cat`每个 base64 编码文件以查看其内容。 在部署期间，这些文件添加到[密卷](container-instances-volume-secret.md)容器组中。 在此示例中，机密卷装载到 Nginx 容器。

```YAML
api-version: 2018-10-01
location: westus
name: app-with-ssl
properties:
  containers:
  - name: nginx-with-ssl
    properties:
      image: nginx
      ports:
      - port: 443
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - name: nginx-config
        mountPath: /etc/nginx
  - name: my-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  volumes:
  - secret:
      ssl.crt: <Enter contents of base64-ssl.crt here>
      ssl.key: <Enter contents of base64-ssl.key here>
      nginx.conf: <Enter contents of base64-nginx.conf here>
    name: nginx-config
  ipAddress:
    ports:
    - port: 443
      protocol: TCP
    type: Public
  osType: Linux
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

### <a name="deploy-the-container-group"></a>部署容器组

创建一个包含资源组[az 组创建](/cli/azure/group#az-group-create)命令：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

部署容器组[az 容器创建](/cli/azure/container#az-container-create)命令，将作为参数传递的 YAML 文件。

```azurecli
az container create --resource-group <myResourceGroup> --file deploy-aci.yaml
```

### <a name="view-deployment-state"></a>查看部署状态

若要查看部署的状态，请使用以下[az 容器显示](/cli/azure/container#az-container-show)命令：

```azurecli
az container show --resource-group <myResourceGroup> --name app-with-ssl --output table
```

对于成功的部署，输出是类似于下面：

```console
Name          ResourceGroup    Status    Image                                                    IP:ports             Network    CPU/Memory       OsType    Location
------------  ---------------  --------  -------------------------------------------------------  -------------------  ---------  ---------------  --------  ----------
app-with-ssl  myresourcegroup  Running   mcr.microsoft.com/azuredocs/nginx, aci-helloworld        52.157.22.76:443     Public     1.0 core/1.5 gb  Linux     westus
```

## <a name="verify-ssl-connection"></a>验证 SSL 连接

若要查看正在运行的应用程序，请导航到其在浏览器中的 IP 地址。 例如，在此示例中所示的 IP 地址是`52.157.22.76`。 必须使用`https://<IP-ADDRESS>`查看正在运行的应用程序，由于 Nginx 服务器配置。 尝试连接与`http://<IP-ADDRESS>`失败。

![浏览器屏幕截图，显示应用程序在 Azure 容器实例中运行](./media/container-instances-container-group-ssl/aci-app-ssl-browser.png)

> [!NOTE]
> 由于此示例中使用自签名的证书，而不是从证书颁发机构，浏览器将显示安全警告，当通过 HTTPS 连接到的站点。 此行为是预期的行为。
>

## <a name="next-steps"></a>后续步骤

本文档演示如何设置 Nginx 容器启用容器组中运行的 web 应用的 SSL 连接。 您可以改写此示例中为在端口 80 以外的端口侦听的应用。 此外可以更新 Nginx 配置文件将自动重定向端口 80 (HTTP) 以使用 HTTPS 上的服务器连接。

虽然这篇文章中挎斗使用 Nginx，你可以使用如另一个 SSL 提供程序[托盘](https://caddyserver.com/)。

在容器组中启用 SSL 的另一种方法是部署中的组[Azure 虚拟网络](container-instances-vnet.md)与[Azure 应用程序网关](../application-gateway/overview.md)。 为 SSL 终结点，可以设置网关。 查看示例[部署模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress-vnet)会进行调整以启用在网关上的 SSL 终止。
