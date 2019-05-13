---
title: Azure Database for PostgreSQL-单个服务器中的服务器概念
description: 本文提供的注意事项和准则，以便配置和管理 Azure Database for PostgreSQL-单个服务器。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: bc135e58d0fbabc809f3718915e9f4e35b8ed875
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067159"
---
# <a name="azure-database-for-postgresql---single-server"></a>Azure Database for PostgreSQL-单个服务器
本文提供如何使用 Azure Database for PostgreSQL-单个服务器的注意事项和准则。

## <a name="what-is-an-azure-database-for-postgresql-server"></a>什么是 Azure Database for PostgreSQL 服务器？
Azure Database for PostgreSQL-单个服务器部署选项中的服务器是多个数据库的中心管理点。 它的 PostgreSQL 服务器构造与本地环境中用户可能比较熟悉的构造相同。 具体而言，PostgreSQL 服务是托管的服务，它提供性能保证、公开服务器级访问权限和功能。

Azure Database for PostgreSQL 服务器：

- 在 Azure 订阅中创建。
- 是数据库的父资源。
- 为数据库提供了一个命名空间。
- 是具有强生存期语义的容器 - 删除服务器时会删除所包含的数据库。
- 并置区域中的资源。
- 为服务器和数据库访问提供连接终结点 
- 提供应用于其数据库的管理策略的作用域：登录名、防火墙、用户、角色、配置等。
- 在多个版本内可用。 有关详细信息，请参阅[支持的 PostgreSQL 数据库版本](concepts-supported-versions.md)。
- 用户可将其进行扩展。 有关详细信息，请参阅 [PostgreSQL 扩展](concepts-extensions.md)。

在 Azure Database for PostgreSQL 数据库中，可创建一个或多个数据库。 可以选择为每个服务器创建单一数据库来使用所有资源，还可以创建多个数据库来共享资源。 按服务器根据定价层、vCore 和存储 (GB) 的配置采用结构化定价。 有关详细信息，请参阅[定价层](./concepts-pricing-tiers.md)。

## <a name="how-do-i-connect-and-authenticate-to-an-azure-database-for-postgresql-server"></a>如何连接到 Azure Database for PostgreSQL 服务器并向其进行身份验证？
以下元素有助于确保安全地访问数据库：

|||
|:--|:--|
| **身份验证和授权** | Azure Database for PostgreSQL 服务器支持本机 PostgreSQL 身份验证。 可使用服务器的管理员登录名连接到服务器并进行身份验证。 |
| 协议 | 该服务支持 PostgreSQL 使用的基于消息的协议。 |
| TCP/IP | 通过 TCP/IP 和 Unix 域套接字支持该协议。 |
| **防火墙** | 为帮助保护数据，在用户指定具有访问权限的计算机之前，防火墙规则将禁止所有对服务器及其数据库的访问。 请参阅  [Azure Database for PostgreSQL 服务器防火墙规则](concepts-firewall-rules.md)。 |

## <a name="managing-your-server"></a>管理服务器
可使用 [Azure 门户](https://portal.azure.com)或 [Azure CLI](/cli/azure/postgres) 管理 Azure Database for PostgreSQL 服务器。

在创建服务器时，设置管理员用户的凭据。 管理员用户是在服务器上具有最高特权的用户。 它属于角色 azure_pg_admin。 此角色没有完整的超级用户权限。 

PostgreSQL 超级用户属性分配给属于托管服务的 azure_superuser。 你无权访问此角色。

Azure Database for PostgreSQL 服务器具有默认数据库： 
- postgres - 可以在创建服务器后连接到的默认数据库。
- azure_maintenance - 此数据库用于将提供托管服务的进程与用户操作分开。 你无权访问此数据库。
- azure_sys - 查询存储的一个数据库。 当查询存储处于关闭状态时，此数据库不会累积数据；这是默认设置。 有关详细信息，请参阅[查询存储概述](concepts-query-store.md)。


## <a name="server-parameters"></a>服务器参数
PostgreSQL 服务器参数可确定服务器的配置。 在 Azure Database for PostgreSQL 中，可使用 Azure 门户或 Azure CLI 查看和编辑参数列表。 

由于是 Postgres 的托管服务，Azure Database for PostgreSQL 中的可配置参数是本地 Postgres 实例中的参数的子集（有关 Postgres 参数的详细信息，请参阅 [PostgreSQL 文档](https://www.postgresql.org/docs/9.6/static/runtime-config.html)）。 在创建时为每个参数使用了默认值，启用了 Azure Database for PostgreSQL 服务器。 用户不能配置某些需要服务器重启或超级用户访问权限才能使更改生效的参数。


## <a name="next-steps"></a>后续步骤
- 有关该服务的概述，请参阅  [Azure Database for PostgreSQL 概述](overview.md)。
- 有关基于服务层级 **** 的具体资源配额和限制的信息，请参阅 [服务层级](concepts-pricing-tiers.md)。
- 有关连接到服务的信息，请参阅  [Azure Database for PostgreSQL 的连接库](concepts-connection-libraries.md)。
- 通过 [Azure 门户](howto-configure-server-parameters-using-portal.md)或 [Azure CLI](howto-configure-server-parameters-using-cli.md) 查看和编辑服务器参数。
