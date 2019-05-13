---
title: SQL 数据仓库 (T-SQL) 中的透明数据加密 | Microsoft 文档
description: SQL 数据仓库 (T-SQL) 中的透明数据加密 (TDE)
services: sql-data-warehouse
author: KavithaJonnakuti
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: security
ms.date: 04/30/2019
ms.author: kavithaj
ms.reviewer: rortloff
ms.openlocfilehash: 1d62bc5cb080f2167f13bd58e9a8a334b8ae5163
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146070"
---
# <a name="get-started-with-transparent-data-encryption-tde"></a>透明数据加密 (TDE) 入门
> [!div class="op_single_selector"]
> * [安全性概述](sql-data-warehouse-overview-manage-security.md)
> * [身份验证](sql-data-warehouse-authentication.md)
> * [加密（门户）](sql-data-warehouse-encryption-tde.md)
> * [加密 (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)
> 
> 

## <a name="required-permissions"></a>所需权限
若要启用透明数据加密 (TDE)，用户必须是管理员或 dbmanager 角色的成员。

## <a name="enabling-encryption"></a>启用加密
执行以下步骤，对 SQL 数据仓库启用 TDE：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到托管数据库的服务器上的 *master* 数据库
2. 执行以下语句来加密数据库。

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;
```

## <a name="disabling-encryption"></a>禁用加密
执行以下步骤，对 SQL 数据仓库禁用 TDE：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库
2. 执行以下语句来加密数据库。

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION OFF;
```

> [!NOTE]
> 在更改 TDE 设置之前，必须恢复暂停的 SQL 数据仓库。
> 
> 

## <a name="verifying-encryption"></a>验证加密
若要验证 SQL 数据仓库的加密状态，请遵循以下步骤：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库或实例数据库
2. 执行以下语句来加密数据库。

```sql
SELECT
    [name],
    [is_encrypted]
FROM
    sys.databases;
```

结果 ```1``` 表示数据库已加密，```0``` 表示数据库未加密。

## <a name="encryption-dmvs"></a>加密 DMV
* [sys.databases][sys.databases] 
* [sys.dm_pdw_nodes_database_encryption_keys][sys.dm_pdw_nodes_database_encryption_keys]

<!--Anchors-->
[Transparent Data Encryption (TDE)]: https://msdn.microsoft.com/library/bb934049.aspx
[sys.databases]: https://msdn.microsoft.com/library/ms178534.aspx  
[sys.dm_pdw_nodes_database_encryption_keys]: https://msdn.microsoft.com/library/mt203922.aspx  

<!--Image references-->

<!--Link references-->
