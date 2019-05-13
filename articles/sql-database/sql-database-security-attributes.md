---
title: Azure SQL 数据库的安全属性
description: 评估 Azure SQL 数据库的安全属性进行的检查表
services: sql-database
author: msmbaldwin
manager: barbkess
ms.service: load-balancer
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mbaldwin
ms.openlocfilehash: df1ffa07c9b813ee3da4952bbcc394f43c69b7ac
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204239"
---
# <a name="security-attributes-for-azure-sql-database"></a>Azure SQL 数据库的安全属性

本文介绍内置于 Azure SQL 数据库的常见安全属性。

[!INCLUDE [Security Attributes Header](../../includes/security-attributes-header.md)]

Azure SQL 数据库包括[单一数据库](sql-database-single-index.yml)并[托管的实例](sql-database-managed-instance.md)。 下面的条目适用于这两个产品/服务，除非特别指出。

## <a name="preventative"></a>预防

| 安全属性 | 是/否 | 说明 |
|---|---|--|
| 静态加密：<ul><li>服务器端加密</li><li>使用客户托管密钥的服务器端加密</li><li>其他加密功能（例如客户端、始终加密等）</ul>| “是” | 称为"加密--使用中"，因为本文所述[Always Encrypted](sql-database-always-encrypted.md)。 使用服务端加密[透明数据加密](transparent-data-encryption-azure-sql.md)(TDE)。|
| 传输中加密：<ul><li>ExpressRoute 加密</li><li>Vnet 中加密</li><li>VNet-VNet 加密</ul>| “是” | 使用 HTTPS。 |
| 加密密钥处理（CMK、BYOK 等）| “是” | 提供服务托管和客户托管密钥的处理 (通过后者[Azure 密钥保管库](../key-vault/index.yml)。 |
| 列级加密（Azure 数据服务）| “是” | 通过[始终加密](sql-database-always-encrypted.md)。 |
| 加密的 API 调用| “是” | 使用 HTTPS/SSL。 |

## <a name="network-segmentation"></a>网络分段

| 安全属性 | 是/否 | 说明 |
|---|---|--|
| 服务终结点支持| “是” | 适用于[单个数据库](sql-database-single-index.yml)仅。 |
| vNET 注入支持| “是” | 适用于[托管的实例](sql-database-managed-instance.md)仅。 |
| 网络隔离/防火墙支持| “是” | 防火墙在这两个数据库和服务器级别;网络隔离，适用[托管的实例](sql-database-managed-instance.md)仅 |
| 支持强制隧道 | “是” | [托管的实例](sql-database-managed-instance.md)通过[Azure ExpressRoute](../expressroute/index.yml) VPN |

## <a name="detection"></a>检测

| 安全属性 | 是/否 | 说明|
|---|---|--|
| Azure 监视支持 （日志分析、 应用程序见解等）| “是” | 来自 Imperva (SecureSphere) 的第三方 SIEM 解决方案还支持，则通过[Azure 事件中心](../event-hubs/index.yml)通过集成[SQL 审核](sql-database-auditing.md)。 |

## <a name="identity-and-access-management"></a>标识和访问管理

| 安全属性 | 是/否 | 说明|
|---|---|--|
| 访问管理 - 身份验证| “是” | Azure Active Directory。 |
| 访问管理 - 授权| “是” |  |


## <a name="audit-trail"></a>审核线索

| 安全属性 | 是/否 | 说明|
|---|---|--|
| 控制/管理计划日志记录和审核| “是” | 对于某些事件仅是。 |
| 数据平面日志记录和审核 | “是” | 通过[SQL 审核](sql-database-auditing.md)。 |

## <a name="configuration-management"></a>配置管理

| 安全属性 | 是/否 | 说明|
|---|---|--|
| 配置管理支持 （版本控制的配置，等等）。| “否”  | | 

## <a name="additional-security-attributes-for-sql-database"></a>SQL 数据库的其他安全属性

| 安全属性 | 是/否 | 说明|
|---|---|--|
| 防护： 漏洞评估 | “是” | 请参阅[SQL 漏洞评估服务可帮助您确定数据库的漏洞](sql-vulnerability-assessment.md)。 |
| 预防： 数据发现和分类  | “是” | 请参阅[Azure SQL 数据库和 SQL 数据仓库数据发现和分类](sql-database-data-discovery-and-classification.md)。 |
| 检测： 威胁检测 | “是” | 请参阅[高级威胁防护，Azure SQL Database 的](sql-database-threat-detection-overview.md)。 |
