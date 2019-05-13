---
title: Azure Database for PostgreSQL-单个服务器配置 SSL 连接性
description: 若要配置用于 PostgreSQL 的 Azure 数据库，单个服务器和关联应用程序以正确使用 SSL 连接的说明和信息。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 56611267872ca79d7d2fe3a08c9b9f49a9b1840b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067408"
---
# <a name="configure-ssl-connectivity-in-azure-database-for-postgresql---single-server"></a>Azure Database for PostgreSQL-单个服务器配置 SSL 连接性
Azure Database for PostgreSQL 倾向于使用安全套接字层 (SSL) 将客户端应用程序连接到 PostgreSQL 服务。 通过在数据库服务器与客户端应用程序之间强制实施 SSL 连接，可以加密服务器与应用程序之间的数据流，有助于防止“中间人”攻击。

默认情况下，PostgreSQL 数据库服务配置为需要 SSL 连接。 （可选）如果客户端应用程序不支持 SSL 连接，则可以禁止要求 SSL 连接到数据库服务。 

## <a name="enforcing-ssl-connections"></a>强制实施 SSL 连接
对于通过 Azure 门户或 CLI 预配的所有 Azure Database for PostgreSQL 服务器，强制实施 SSL 连接是默认启用的。 

同样，在 Azure 门户中，用户服务器的“连接字符串”设置中预定义了连接字符串，该字符串中包含以通用语言使用 SSL 连接到数据库服务器所需的参数。 SSL 参数因连接器而异，例如“ssl=true”、“sslmode=require”或“sslmode=required”，以及其他变体。

## <a name="configure-enforcement-of-ssl"></a>配置强制实施 SSL
（可选）可以禁用强制实施 SSL 连接。 Microsoft Azure 建议始终启用“强制实施 SSL 连接”设置，以增强安全性。

### <a name="using-the-azure-portal"></a>使用 Azure 门户
访问 Azure Database for PostgreSQL 服务器，并单击“连接安全性”。 使用切换按钮来启用或禁用“强制实施 SSL 连接”设置。 然后，单击“保存”。 

![连接安全性 - 禁用强制实施 SSL](./media/concepts-ssl-connection-security/1-disable-ssl.png)

可以通过在“概述”页中查看“SSL 强制实施状态”指示器来确认设置。

### <a name="using-azure-cli"></a>使用 Azure CLI
可以通过在 Azure CLI 中分别使用 `Enabled` 或 `Disabled` 值来启用或禁用“ssl-enforcement”参数。

```azurecli
az postgres server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Enabled
```

## <a name="ensure-your-application-or-framework-supports-ssl-connections"></a>确保应用程序或框架支持 SSL 连接
许多将 PostgreSQL 用于其数据库服务的常见应用程序框架（例如 Drupal 和 Django）在安装过程中不会默认启用 SSL。 必须在安装完毕后启用 SSL 连接性，或者通过特定于应用程序的 CLI 命令启用。 如果 PostgreSQL 服务器强制实施了 SSL 连接，但是未正确配置关联的应用程序，那么应用程序可能无法连接到数据库服务器。 请查阅应用程序文档，了解如何启用 SSL 连接。


## <a name="applications-that-require-certificate-verification-for-ssl-connectivity"></a>需要证书验证才可启用 SSL 连接性的应用程序
在某些情况下，应用程序需要具备从受信任的证书颁发机构 (CA) 证书文件 (.cer) 生成的本地证书文件才能实现安全连接。 请参阅以下步骤获取 .cer 文件，解码证书并将其绑定到应用程序。

### <a name="download-the-certificate-file-from-the-certificate-authority-ca"></a>从证书颁发机构 (CA) 下载证书文件 
可在[此处](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt)找到通过 SSL 与 Azure Database for PostgreSQL 服务器通信所需的证书。 本地下载证书文件。

### <a name="install-a-cert-decoder-on-your-machine"></a>在计算机上安装证书解码器 
可以使用 [OpenSSL](https://github.com/openssl/openssl) 来解码应用程序安全连接到数据库服务器所需的证书文件。 若要了解如何安装 OpenSSL，请参阅 [OpenSSL 安装说明](https://github.com/openssl/openssl/blob/master/INSTALL)。 


### <a name="decode-your-certificate-file"></a>解码证书文件
下载的根 CA 文件采用加密格式。 使用 OpenSSL 解码证书文件。 要执行此操作，请运行此 OpenSSL 命令：

```
openssl x509 -inform DER -in BaltimoreCyberTrustRoot.crt -text -out root.crt
```

### <a name="connecting-to-azure-database-for-postgresql-with-ssl-certificate-authentication"></a>连接到具有 SSL 证书身份验证的 Azure Database for PostgreSQL
现已成功解码证书，可通过 SSL 安全连接到数据库服务器。 要允许服务器证书验证，必须将证书放在用户主目录中的 ~/.postgresql/root.crt 文件中。 （在 Microsoft Windows 上，该文件的名称是 %APPDATA%\postgresql\root.crt.）。 

#### <a name="connect-using-psql"></a>使用 psql 进行连接
以下示例演示如何使用 psql 命令行实用程序成功连接到 PostgreSQL 服务器。 使用创建的 `root.crt` 文件和 `sslmode=verify-ca` 或 `sslmode=verify-full` 选项。

使用 PostgreSQL 命令行接口执行以下命令：
```bash
psql "sslmode=verify-ca sslrootcert=root.crt host=mydemoserver.postgres.database.azure.com dbname=postgres user=mylogin@mydemoserver"
```
如果成功，则会收到以下输出：
```bash
Password for user mylogin@mydemoserver:
psql (9.6.2)
WARNING: Console code page (437) differs from Windows code page (1252)
     8-bit characters might not work correctly. See psql reference
     page "Notes for Windows users" for details.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=>
```

## <a name="next-steps"></a>后续步骤
在 [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)（Azure Database for PostgreSQL 的连接库）中查看各种应用程序连接选项。
