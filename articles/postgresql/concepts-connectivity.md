---
title: Azure database for PostgreSQL-单个服务器暂时性连接错误的处理
description: 了解如何针对 Azure Database for PostgreSQL-单个服务器处理暂时性连接错误。
keywords: postgresql 连接, 连接字符串, 连接问题, 暂时性错误, 连接错误
author: jan-eng
ms.author: janeng
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: ea90de612dcfb2559b29fbffce8306278beb45b9
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073511"
---
# <a name="handling-transient-connectivity-errors-for-azure-database-for-postgresql---single-server"></a>用于对 Azure Database for PostgreSQL-单个服务器处理暂时性连接错误

本文介绍如何处理连接到 Azure Database for PostgreSQL 的暂时性错误。

## <a name="transient-errors"></a>暂时性错误

暂时性错误也称为暂时性故障，是一种可以自行解决的错误。 这些错误往往表现为与数据库服务器的连接断开。 此外，无法与服务器建立的新连接。 例如，在发生硬件或网络故障时，可能会出现暂时性错误。 另一个可能的原因是正在推出 PaaS 服务的新版本。系统在 60 秒以内可自动解决其中的大部分事件。 设计和开发云中的应用程序时，预料到会出现暂时性错误是最佳做法。 假设这些错误随时可能在任意组件中发生，并部署相应的逻辑来应对这种情况。

## <a name="handling-transient-errors"></a>处理暂时性错误

应使用重试逻辑来处理暂时性错误。 必须考虑的情况包括：

* 尝试打开连接时出错
* 服务器端的空闲连接断开。 尝试发出的命令无法执行
* 当前正在用于执行命令的活动连接断开。

第一种和第二种情况的处理方式非常直接。 可以再试打开连接。 如果连接成功，则表示系统解决了暂时性错误。 现在又可以使用 Azure Database for PostgreSQL 了。 我们建议在重试连接之前等待一段时间。 如果初始重试失败，则回退。 这样，系统便可以使用所有可用资源来解决错误局面。 遵循的良好模式是：

* 在首次重试之前等待 5 秒。
* 对于每次后续重试，以指数级增大等待时间，最长为 60 秒。
* 设置最大重试次数，达到该次数时，应用程序认为操作失败。

活动事务的连接失败时，适当地处理恢复会更困难。 存在两种情况：如果事务在性质上是只读的，则可以安全地重新打开连接并重试事务。 但是，如果事务也在写入数据库，则必须确定事务在发生暂时性错误之前是已回滚还是已成功。 在这种情况下，你可能尚未从数据库服务器收到提交确认。

解决此问题的方法之一是，在客户端上生成一个用于所有重试的唯一 ID。 将此唯一 ID 作为事务的一部分传递给服务器，并将其存储在具有唯一约束的列中。 这样，便可以安全重试事务。 如果前一事务已回滚，并且客户端生成的唯一 ID 在系统中尚不存在，则重试将会成功。 如果之前已存储该唯一 ID（因为前一事务已成功完成），则重试将会失败，并指示重复键冲突。

如果程序通过第三方中间件来与 Azure Database for PostgreSQL 通信，请咨询供应商该中间件是否包含暂时性错误的重试逻辑。

请务必测试重试逻辑。 例如，尝试在纵向扩展或缩减 Azure Database for PostgreSQL 服务器的计算资源时执行代码。 应用程序应可处理此操作期间遇到的短暂停机，而不会出现任何问题。

## <a name="next-steps"></a>后续步骤

* [排查 Azure Databases for PostgreSQL 的连接问题](howto-troubleshoot-common-connection-issues.md)
