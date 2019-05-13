---
title: Azure Database for PostgreSQL-单个服务器中的防火墙规则
description: 本文介绍 Azure Database for PostgreSQL-单个服务器防火墙规则。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 40a675fbefe9743f5de1f9766cf33ae7dba9e5a7
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073575"
---
# <a name="firewall-rules-in-azure-database-for-postgresql---single-server"></a>Azure Database for PostgreSQL-单个服务器中的防火墙规则
Azure Database for PostgreSQL 服务器防火墙阻止对你的数据库服务器所有访问，直到你指定哪些计算机具有的权限。 防火墙基于每个请求的起始 IP 地址授予对服务器的访问权限。
要配置防火墙，请创建防火墙规则，以指定可接受的 IP 地址的范围。 可以在服务器级别创建防火墙规则。

**防火墙规则：** 这些规则允许客户端访问整个 Azure Database for PostgreSQL 服务器，即同一逻辑服务器内的所有数据库。 可以通过使用 Azure 门户或 Azure CLI 命令配置服务器级防火墙规则。 若要创建服务器级防火墙规则，用户必须是订阅所有者或订阅参与者。

## <a name="firewall-overview"></a>防火墙概述
默认情况下，防火墙将阻止对 Azure Database for PostgreSQL 服务器的所有数据库访问。 若要从另一台计算机开始使用服务器，需要指定一个或多个服务器级防火墙规则以允许访问服务器。 使用防火墙规则指定要允许的来自 Internet 的 IP 地址范围。 对 Azure 门户网站本身的访问不受防火墙规则影响。
来自 Internet 和 Azure 的连接尝试必须首先通过防火墙，才能访问 PostgreSQL 数据库，如下图中所示：

![防火墙工作流示例](media/concepts-firewall-rules/1-firewall-concept.png)

## <a name="connecting-from-the-internet"></a>从 Internet 连接
服务器级防火墙规则适用于 Azure Database for PostgreSQL 服务器上的所有数据库。 如果该请求的 IP 地址位于服务器级防火墙规则中指定的某个范围内，则授予连接权限。
如果该请求的 IP 地址不位于任何服务器级防火墙规则中指定的范围内，则连接请求失败。
例如，如果应用程序与 PostgreSQL 的 JDBC 驱动程序连接，则在防火墙阻止连接时尝试进行连接可能会遇到此错误。
> java.util.concurrent.ExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException:致命错误: 主机 "123.45.67.890"、用户 "adminuser"、数据库 "postgresql"、SSL 没有 pg\_hba.conf 条目

## <a name="connecting-from-azure"></a>从 Azure 连接
若要允许来自 Azure 的应用程序连接到 Azure Database for PostgreSQL 服务器，必须启用 Azure 连接。 例如，为了托管“Azure Web 应用”应用程序或 Azure VM 中运行的应用程序，或者为了从 Azure 数据工厂数据管理网关进行连接。 资源无需在同一虚拟网络 (VNet) 或资源组中，即可使用防火墙规则启用这些连接。 在应用程序尝试从 Azure 连接到数据库服务器时，防火墙会验证是否允许 Azure 连接。 有几种方法可启用这些类型的连接。 如果防火墙设置的开始地址和结束地址都等于 0.0.0.0，则表示允许这些连接。 或者，可以在门户中从“连接安全性”窗格将“允许访问 Azure 服务”选项设为“启用”并点击“保存”。 如果不允许该连接尝试，则该请求将不会访问 Azure Database for PostgreSQL 服务器。

> [!IMPORTANT]
> 该选项将防火墙配置为允许来自 Azure 的所有连接，包括来自其他客户的订阅的连接。 选择该选项时，请确保登录名和用户权限将访问权限限制为仅已授权用户使用。
> 

![在门户中配置“允许访问 Azure 服务”](media/concepts-firewall-rules/allow-azure-services.png)

## <a name="programmatically-managing-firewall-rules"></a>以编程方式管理防火墙规则
除了 Azure 门户外，还可使用 Azure CLI 通过编程方式管理防火墙规则。
另请参阅[使用 Azure CLI 创建和管理 Azure Database for PostgreSQL 防火墙规则](howto-manage-firewall-using-cli.md)

## <a name="troubleshooting-the-database-server-firewall"></a>数据库服务器防火墙故障排除
在对 Microsoft Azure Database for PostgreSQL 服务器服务的访问与期望不符时，请考虑以下几点：

* **对允许列表的更改尚未生效：** 对 Azure Database for PostgreSQL 防火墙配置所做的更改可能最多需要 5 分钟的延迟才可生效。

* **登录名未授权或使用了错误的密码：** 如果某个登录名对 Azure Database for PostgreSQL 服务器没有权限或者使用的密码不正确，则与 Azure Database for PostgreSQL 服务器连接会被拒绝。 创建防火墙设置，仅向客户端提供尝试连接到服务器的机会；每个客户端必须提供必需的安全凭据。

例如，使用 JDBC 客户端可能会出现以下错误。
> java.util.concurrent.ExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException:致命错误: 用户 "yourusername" 的密码验证失败

* **动态 IP 地址：** 如果 Internet 连接使用动态 IP 寻址，并且在通过防火墙时遇到问题，则可以尝试以下解决方法之一：

* 向 Internet 服务提供商 (ISP) 询问分配给客户端计算机、用于访问 Azure Database for PostgreSQL 服务器的 IP 地址范围，然后将该 IP 地址范围作为防火墙规则添加。

* 改为获取客户端计算机的静态 IP 地址，然后将该静态 IP 地址作为防火墙规则添加。

## <a name="next-steps"></a>后续步骤
有关如何创建服务器级和数据库级防火墙规则的文章，请参阅：
* [使用 Azure 门户创建和管理 Azure Database for PostgreSQL 防火墙规则](howto-manage-firewall-using-portal.md)
* [使用 Azure CLI 创建和管理 Azure Database for PostgreSQL 防火墙规则](howto-manage-firewall-using-cli.md)
