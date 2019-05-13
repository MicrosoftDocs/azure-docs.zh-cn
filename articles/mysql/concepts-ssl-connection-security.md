---
title: Azure Database for MySQL 的 SSL 连接
description: 有关配置 Azure Database for MySQL 和关联应用程序以正确使用 SSL 连接的信息
author: JasonMAnderson
ms.author: janders
ms.service: mysql
ms.topic: conceptual
ms.date: 02/28/2018
ms.openlocfilehash: 129f90d495627edb25dfafdeb1b274aa2c4c71cb
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "60525811"
---
# <a name="ssl-connectivity-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的 SSL 连接
Azure Database for MySQL 支持使用安全套接字层 (SSL) 将数据库服务器连接到客户端应用程序。 通过在数据库服务器与客户端应用程序之间强制实施 SSL 连接，可以加密服务器与应用程序之间的数据流，有助于防止“中间人”攻击。

## <a name="default-settings"></a>默认设置
默认情况下，应将数据库服务配置为需要 SSL 连接才可连接到 MySQL。  建议尽量不要禁用 SSL 选项。 

通过 Azure 门户和 CLI 预配新的 Azure Database for MySQL 服务器时，默认情况下会强制实施 SSL 连接。 

Azure 门户中显示了各种编程语言的连接字符串。 这些连接字符串包含连接到数据库所需的 SSL 参数。 在 Azure 门户中，选择服务器。 在“设置”标题下，选择“连接字符串”。 SSL 参数因连接器而异，例如“ssl=true”、“sslmode=require”或“sslmode=required”，以及其他变体。

若要了解如何在开发应用程序期间启用或禁用 SSL 连接，请参阅[如何配置 SSL](howto-configure-ssl.md)。 

## <a name="next-steps"></a>后续步骤
[Azure Database for MySQL 的连接库](concepts-connection-libraries.md)
