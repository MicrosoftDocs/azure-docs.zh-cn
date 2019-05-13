---
title: Azure database for PostgreSQL-单个服务器的连接库
description: 本指南介绍了几个库和驱动程序开发人员可以使用时编写应用程序以连接并查询 Azure Database for PostgreSQL-单个服务器。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 85110126f9bdec225b1644860814cd89832132a1
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073593"
---
# <a name="connection-libraries-for-azure-database-for-postgresql---single-server"></a>Azure database for PostgreSQL-单个服务器的连接库
本文列出了开发人员在开发要连接到的应用程序和查询 Azure Database for PostgreSQL 时可使用的库和驱动程序。

## <a name="client-interfaces"></a>客户端接口
用于连接到 PostgreSQL 服务器的大多数语言客户端库为外部项目，并且独立分发。 Windows、Linux 和 Mac 平台支持所列出的库连接到 Azure Database for PostgreSQL。 “后续步骤”部分中列出了几个快速入门示例。

| **语言** | 客户端接口 | 其他信息 | **下载** |
|--------------|----------------------------------------------------------------|-------------------------------------|--------------------------------------------------------------------|
| Python | [psycopg](http://initd.org/psycopg/) | 符合 DB API 2.0 规范 | [下载](http://initd.org/psycopg/download/) |
| PHP | [php-pgsql](https://secure.php.net/manual/en/book.pgsql.php) | 数据库扩展 | [安装](https://secure.php.net/manual/en/pgsql.installation.php) |
| Node.js | [Pg npm 包](https://www.npmjs.com/package/pg) | 纯 JavaScript 非阻止客户端 | [安装](https://www.npmjs.com/package/pg) |
| Java | [JDBC](https://jdbc.postgresql.org/) | 类型 4 JDBC 驱动程序 | [下载](https://jdbc.postgresql.org/download.html)  |
| Ruby | [Pg gem](https://deveiate.org/code/pg/) | Ruby 接口 | [下载](https://rubygems.org/downloads/pg-0.20.0.gem) |
| Go | [Package pq](https://godoc.org/github.com/lib/pq) | 纯 Go 语言 postgres 驱动程序 | [安装](https://github.com/lib/pq/blob/master/README.md) |
| C\#/ .NET | [Npgsql](https://www.npgsql.org/) | ADO.NET 数据提供程序 | [下载](https://www.microsoft.com/net/) |
| ODBC | [psqlODBC](https://odbc.postgresql.org/) | ODBC 驱动程序 | [下载](https://www.postgresql.org/ftp/odbc/versions/) |
| C | [libpq](https://www.postgresql.org/docs/9.6/static/libpq.html) | 主要 C 语言接口 | 附送 |
| C++ | [libpqxx](http://pqxx.org/) | 新样式 C++ 接口 | [下载](http://pqxx.org/download/software/) |

## <a name="next-steps"></a>后续步骤
阅读这些快速入门，了解如何使用所选语言连接和查询 Azure Database for PostgreSQL：

[Python](./connect-python.md) | [Node.JS](./connect-nodejs.md) | [Java](./connect-java.md) | [Ruby](./connect-ruby.md) | [PHP](./connect-php.md) | [.NET (C#)](./connect-csharp.md) | [Go](./connect-go.md)
