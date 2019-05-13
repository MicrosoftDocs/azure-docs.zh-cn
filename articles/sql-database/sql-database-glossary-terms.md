---
title: Azure SQL 数据库术语表 | Microsoft Docs
description: Azure SQL 数据库术语表
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 04/26/2019
ms.openlocfilehash: fb628358c91d80a3117b065cadb284fd1665daa3
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073689"
---
# <a name="azure-sql-database-glossary-of-terms"></a>Azure SQL 数据库术语表

|上下文|术语|详细信息|
|:---|:---|:---|
|Azure 服务名称|Azure SQL 数据库或 SQL 数据库|[Azure SQL 数据库服务](sql-database-technical-overview.md)|
|计算层|无服务器 （预览版）|[无服务器计算层](sql-database-serverless.md)
||已预配|[无服务器计算层](sql-database-serverless.md)
|部署选项 |单个数据库|[单一数据库](sql-database-single-database.md)|
||弹性池|[弹性池](sql-database-elastic-pool.md)|
||托管实例|[托管实例](sql-database-managed-instance.md)|
|服务器对象|SQL 数据库服务器或数据库服务器|[数据库服务器](sql-database-servers.md)|
||SQL 数据库托管实例服务器、托管实例服务器或实例服务器|[托管实例](sql-database-managed-instance.md)|
数据库对象|Azure SQL 数据库|Azure SQL 数据库中的任何数据库|
||单个数据库|使用单一数据库部署选项创建的数据库|
||共用数据库|在弹性池内创建或移入其中的数据库|
||实例数据库|在托管实例中创建的数据库|
||基本数据库|在基于 DTU 的购买模型的基本服务层级内创建或移入其中的数据库|
||标准数据库|在基于 DTU 的购买模型的标准服务层级内创建或移入其中的数据库|
||高级数据库|在基于 DTU 的购买模型的高级服务层级内创建或移入其中的数据库|
||常规用途数据库|在基于 vCore 的购买模型的一般用途服务层级内创建或移入其中的数据库|
||“超大规模”数据库|在基于 vCore 的购买模型的超大规模服务层级内创建或移入其中的数据库|
||业务关键数据库|在基于 vCore 的购买模型的业务关键服务层级内创建或移入其中的数据库|
||已设置的数据库|配置预配的计算层中的数据库|
|[购买模型和资源](sql-database-purchase-models.md)|基于 DTU 的购买模型|[基于 DTU 的购买模型](sql-database-service-tiers-dtu.md)|
||基于 vCore 的购买模型|[基于 vCore 的购买模型](sql-database-service-tiers-vcore.md)|
||vCore|虚拟机监控程序向来宾 OS 提供的核心。|
||服务层|购买模型中的服务级别|
||计算大小|服务层级内单一数据库、弹性池或托管实例的计算资源的数量|
||存储量|单个数据库、弹性池或托管实例的可用存储量|
||计算的代|服务层级内的处理器代系|
|数据库服务器 IP 防火墙规则|IP 防火墙规则|[IP 防火墙规则](sql-database-firewall-configure.md)|
||服务器级别 IP 防火墙规则|[服务器级 IP 防火墙规则](sql-database-firewall-configure.md#overview)|
|| 数据库级 IP 防火墙规则|[数据库级 IP 防火墙规则](sql-database-firewall-configure.md#overview)|
||虚拟网络终结点和规则|[虚拟网络终结点和规则](sql-database-vnet-service-endpoint-rule-overview.md)|
