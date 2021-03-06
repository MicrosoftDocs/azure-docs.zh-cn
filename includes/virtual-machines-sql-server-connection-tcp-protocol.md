---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: f7af6962c3343cd9d3e35e96d88650aa6a42c6b3
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/24/2020
ms.locfileid: "95555601"
---
1. 使用远程桌面连接到虚拟机后，搜索“配置管理器”： 

    ![打开 SSCM](./media/virtual-machines-sql-server-connection-tcp-protocol/sql-server-configuration-manager.png)

1. 在 SQL Server 配置管理器中，在控制台窗格中，展开“SQL Server 网络配置”。 

1. 在控制台窗格中，单击“MSSQLSERVER 的协议”（默认实例名称）。  在详细信息窗格中，右键单击“TCP”，然后单击“启用”（如果尚未启用）。  

    ![启用 TCP](./media/virtual-machines-sql-server-connection-tcp-protocol/enable-tcp.png)

1. 在控制台窗格中，单击“SQL Server 服务”。  在详细信息窗格中，右键单击“SQL Server (实例名)”  （默认实例为 **SQL Server (MSSQLSERVER)** ），然后单击“重启”  以停止并重启该 SQL Server 实例。

    ![重新启动数据库引擎](./media/virtual-machines-sql-server-connection-tcp-protocol/restart-sql-server.png)

1. 关闭 SQL Server 配置管理器。

有关启用 SQL Server 数据库引擎的协议的详细信息，请参阅[启用或禁用服务器网络协议](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol)。