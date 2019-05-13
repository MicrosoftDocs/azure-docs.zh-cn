---
title: PostgreSQL-使用 Azure 门户的单个服务器的重新启动 Azure 数据库
description: 本文介绍如何为 PostgreSQL-使用 Azure 门户的单个服务器重新启动 Azure 数据库。
author: ajlam
ms.author: andrela
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: fd92e27f53f52de3e9a7fd65d577c9dfea44991b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65066840"
---
# <a name="restart-azure-database-for-postgresql---single-server-using-the-azure-portal"></a>PostgreSQL-使用 Azure 门户的单个服务器的重新启动 Azure 数据库
本主题介绍如何重启 Azure Database for PostgreSQL 服务器。 出于维护原因，可能需要重启服务器，这会在服务器执行操作时导致短暂中断。

如果服务处于繁忙状态，则会阻止重启服务器。 例如，服务可以处理缩放 vCores 等先前请求的操作。
 
完成重启所需的时间取决于 PostgreSQL 恢复过程。 若要减少重启时间，建议在重启之前尽量减少服务器上发生的活动量。

## <a name="prerequisites"></a>必备组件
若要完成本操作指南，需要：
- [Azure Database for PostgreSQL 服务器](quickstart-create-server-database-portal.md)

## <a name="perform-server-restart"></a>执行服务器重启

可通过以下步骤重启 PostgreSQL 服务器：

1. 在 [Azure 门户](https://portal.azure.com/)中，选择 Azure Database for PostgreSQL 服务器。

2. 在服务器“概述”页的工具栏中，单击“重启”。

   ![Azure Database for PostgreSQL - 概述 - “重启”按钮](./media/howto-restart-server-portal/2-server.png)

3. 单击“是”以确认重启服务器。

   ![Azure Database for PostgreSQL - 重启确认](./media/howto-restart-server-portal/3-restart-confirm.png)

4. 观察到服务器状态更改为“正在重启”。

   ![Azure Database for PostgreSQL - 重启状态](./media/howto-restart-server-portal/4-restarting-status.png)

5. 确认服务器重启成功。

   ![Azure Database for PostgreSQL - 重启成功](./media/howto-restart-server-portal/5-restart-success.png)

## <a name="next-steps"></a>后续步骤

了解[如何在 Azure Database for PostgreSQL 中设置参数](howto-configure-server-parameters-using-portal.md)