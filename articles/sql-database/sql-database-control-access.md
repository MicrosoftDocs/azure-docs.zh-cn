---
title: 授予对 Azure SQL 数据库和 SQL 数据仓库的访问权限 | Microsoft Docs
description: 了解如何授予对 Microsoft Azure SQL 数据库和 SQL 数据仓库的访问权限。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sql-data-warehouse
ms.devlang: ''
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
ms.reviewer: carlrab
manager: craigg
ms.date: 05/08/2019
ms.openlocfilehash: 783a8f0bc25717f1c2bf78a9c0d40b209a07939b
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473347"
---
# <a name="azure-sql-database-and-sql-data-warehouse-access-control"></a>Azure SQL 数据库和 SQL 数据仓库访问控制

为了提供安全性，Azure [SQL 数据库](sql-database-technical-overview.md)和 [SQL 数据仓库](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)会使用按 IP 地址限制连接的防火墙规则、要求用户证明其身份的身份验证机制，以及只允许用户执行特定操作和访问特定数据的授权机制，来控制访问。 

> [!IMPORTANT]
> 有关 SQL 数据库安全功能的概述，请参阅 [SQL 安全概述](sql-database-security-overview.md)。 相关教程，请参阅[保护 Azure SQL 数据库](sql-database-security-tutorial.md)。 有关 SQL 数据仓库安全功能的概述，请参阅 [SQL 数据仓库安全概述](../sql-data-warehouse/sql-data-warehouse-overview-manage-security.md)

## <a name="firewall-and-firewall-rules"></a>防火墙和防火墙规则

Microsoft Azure SQL 数据库为 Azure 和其他基于 Internet 的应用程序提供关系数据库服务。 为了保护数据，在指定哪些计算机具有访问权限之前，防火墙将禁止所有对数据库服务器的访问。 防火墙基于每个请求的起始 IP 地址授予数据库访问权限。 有关详细信息，请参阅 [Azure SQL 数据库防火墙规则概述](sql-database-firewall-configure.md)。

只能通过 TCP 端口 1433 使用 Azure SQL 数据库服务。 若要从计算机访问 SQL 数据库，请确保客户端计算机防火墙允许 TCP 端口 1433 上的传出 TCP 通信。 如果其他应用程序不需要，则阻止 TCP 端口 1433 上的入站连接。 

在连接过程中，来自 Azure 虚拟机的连接将重定向到每个辅助角色特有的不同 IP 地址和端口。 该端口号在 11000 到 11999 的范围内。 有关 TCP 端口的详细信息，请参阅[用于 ADO.NET 4.5 和 SQL Database2 的非 1433 端口](sql-database-develop-direct-route-ports-adonet-v12.md)。

## <a name="authentication"></a>Authentication

SQL 数据库支持两种类型的身份验证：

- **SQL 身份验证**：

  此身份验证方法使用用户名和密码。 在为数据库创建 SQL 数据库服务器时，已指定一个包含用户名和密码的“服务器管理员”登录名。 通过这些凭据，可以使用数据库所有者（即“dbo”）的身份通过服务器上任何数据库的身份验证。 
- **Azure Active Directory 身份验证**：

  此身份验证方法使用由 Azure Active Directory 托管的标识，并且受托管域和集成域支持。 如果想要使用 Azure Active Directory 身份验证，则必须创建名为“Azure AD 管理员”的另一个服务器管理员，用于管理 Azure AD 用户和组。 此管理员还能执行普通服务器管理员可以执行的所有操作。 有关如何创建 Azure AD 管理员以启用 Azure Active Directory 身份验证的演练，请参阅[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](sql-database-aad-authentication.md)。

数据库引擎将关闭空闲超过 30 分钟的连接。 该连接必须重新登录才可供使用。 连续与 SQL 数据库建立活动连接需要至少每隔 10 小时重新授权一次（由数据库引擎执行授权）。 数据库引擎将尝试使用最初提交的密码重新授权，且不需要用户输入。 出于性能原因，在 SQL 数据库中重置密码时，不会对连接重新进行身份验证，即使该连接由于连接池而重置。 这一点与本地 SQL Server 的行为不同。 如果在最初授权连接后密码发生更改，则必须终止该连接，并使用新密码建立新连接。 具有 `KILL DATABASE CONNECTION` 权限的用户可以使用 [KILL](https://docs.microsoft.com/sql/t-sql/language-elements/kill-transact-sql) 命令显式终止与 SQL 数据库的连接。

可在 master 数据库中创建用户帐户并向其授予服务器上所有数据库中的权限，或者在数据库本身内部创建用户帐户（称为包含的用户）。 有关创建和管理登录名的信息，请参阅[管理登录名](sql-database-manage-logins.md)。 若要增强可移植性和可伸缩性，请使用包含的数据库。 有关包含的用户的详细信息，请参阅[包含的数据库用户 - 使数据库可移植](https://docs.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable)、[CREATE USER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql) 和[包含的数据库](https://docs.microsoft.com/sql/relational-databases/databases/contained-databases)。

根据最佳实践，应用程序应使用专用帐户进行身份验证 – 这样，就可以限制授予应用程序的权限，并在应用程序代码容易受到 SQL 注入攻击的情况下降低恶意活动的风险。 建议的方法是创建 [包含数据库用户](https://docs.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable)，使应用程序能够直接向数据库进行身份验证。 

## <a name="authorization"></a>授权

授权是指用户可以在 Azure SQL 数据库中执行哪些操作，这由用户帐户的数据库[角色成员身份](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles)和[对象级权限](https://docs.microsoft.com/sql/relational-databases/security/permissions-database-engine)来控制。 作为最佳实践，应向用户授予所需的最低权限。 用于连接的服务器管理员帐户是 db_owner 所有者的成员，该帐户有权在数据库中执行任何操作。 请保存此帐户，以便部署架构升级并执行其他管理操作。 权限受到更多限制的“ApplicationUser”帐户可让用户使用应用程序所需的最低权限从应用程序连接到数据库。 有关详细信息，请参阅[管理登录名](sql-database-manage-logins.md)。

通常，只有管理员需要 `master` 数据库的访问权限。 对每个用户数据库的例程访问应通过数据库用户进行，这些用户创建于每个数据库且不包含管理员。 使用包含的数据库用户时，不需要在 `master` 数据库中创建登录名。 有关详细信息，请参阅[包含数据库用户 - 使数据库可移植](https://docs.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable)。

用户应熟悉以下功能，以便限制或提升权限：

- [模拟](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/customizing-permissions-with-impersonation-in-sql-server)和[模块签名](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/signing-stored-procedures-in-sql-server)可用于安全地暂时提升权限。
- [行级安全性](https://docs.microsoft.com/sql/relational-databases/security/row-level-security)可用于限制用户可访问的行。
- [数据屏蔽](sql-database-dynamic-data-masking-get-started.md)可用于限制敏感数据的公开。
- [存储过程](https://docs.microsoft.com/sql/relational-databases/stored-procedures/stored-procedures-database-engine)可用于限制可对数据库执行的操作。

## <a name="next-steps"></a>后续步骤

- 有关 SQL 数据库安全功能的概述，请参阅 [SQL 安全概述](sql-database-security-overview.md)。
- 有关防火墙规则的详细信息，请参阅[防火墙规则](sql-database-firewall-configure.md)。
- 若要了解用户和登录名，请参阅[管理登录名](sql-database-manage-logins.md)。 
- 有关主动监视的介绍，请参阅[数据库审核](sql-database-auditing.md)和 [SQL 数据库威胁检测](sql-database-threat-detection.md)。
- 相关教程，请参阅[保护 Azure SQL 数据库](sql-database-security-tutorial.md)。
