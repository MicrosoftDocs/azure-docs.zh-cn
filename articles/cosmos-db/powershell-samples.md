---
title: 适用于 Azure Cosmos DB 的 Azure PowerShell 示例
description: Azure PowerShell 示例 - 这些脚本可帮助你创建和管理 Azure Cosmos DB 帐户。
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 05/08/2019
ms.author: mjbrown
ms.openlocfilehash: 68e845a05f4ebe2d1f25b55c00042c8925c8109e
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65069305"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db"></a>适用于 Azure Cosmos DB 的 Azure PowerShell 示例

下表包含用于 Azure Cosmos DB for Core (SQL) API 的示例 Azure PowerShell 脚本的链接。

| |  |
|---|---|
|**Azure Cosmos 帐户**||
|[创建帐户](scripts/powershell/sql/ps-account-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 创建 Azure Cosmos SQL API 帐户。 |
|[获取一个帐户](scripts/powershell/sql/ps-account-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 获取 Azure Cosmos 帐户的属性。 |
|[添加区域](scripts/powershell/sql/ps-account-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 获取 Azure Cosmos 帐户并将区域添加到位置列表。 |
|[更改故障转移优先级](scripts/powershell/sql/ps-account-failover-priority-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 使用手动故障转移触发器更改 Azure Cosmos 帐户的优先级。 |
|[更新标记](scripts/powershell/sql/ps-account-tags-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 更新 Azure Cosmos 帐户标记。 |
|[获取帐户密钥](scripts/powershell/sql/ps-account-key-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 获取 Azure Cosmos 帐户的主密钥和辅助密钥。 |
|[重新生成帐户密钥](scripts/powershell/sql/ps-account-key-regenerate.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 重新生成 Azure Cosmos 帐户的主密钥和辅助密钥。 |
|[列出连接字符串](scripts/powershell/sql/ps-account-connection-string-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 获取 Azure Cosmos 帐户的主要和辅助连接字符串。 |
|[创建 IP 防火墙](scripts/powershell/sql/ps-account-firewall-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 创建 Azure Cosmos 帐户的 IP 防火墙。 |
|[删除 Azure Cosmos 帐户](scripts/powershell/sql/ps-account-delete.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 删除 Azure Cosmos 帐户。 |
|**Azure Cosmos 数据库**||
| [创建数据库](scripts/powershell/sql/ps-database-create.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 创建 Azure Cosmos 帐户中的数据库。|
| [使用共享/数据库级别的吞吐量创建数据库](scripts/powershell/sql/ps-database-create-shared.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 使用与容器共享的数据库级别的吞吐量创建 Azure Cosmos 数据库。|
| [列出所有数据库](scripts/powershell/sql/ps-database-list.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 列出 Azure Cosmos 帐户中的所有数据库。|
| [获取数据库](scripts/powershell/sql/ps-database-get.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 获取 Azure Cosmos 数据库的属性。|
|**Azure Cosmos 容器**||
| [创建容器](scripts/powershell/sql/ps-container-create.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 使用专用吞吐量创建 Azure Cosmos 容器。|
| [使用共享吞吐量创建容器](scripts/powershell/sql/ps-container-create-shared.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 使用与数据库中其他容器共享的吞吐量创建 Azure Cosmos 容器。|
| [使用索引策略创建容器](scripts/powershell/sql/ps-container-create-index-custom.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 使用自定义索引策略创建 Azure Cosmos 容器。|
| [使用无索引策略创建容器](scripts/powershell/sql/ps-container-create-index-none.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 在索引策略关闭的情况下创建 Azure Cosmos 容器。|
| [使用唯一键和 TTL 创建容器](scripts/powershell/sql/ps-container-create-unique-key-ttl.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 在已配置唯一键约束和生存时间的情况下创建 Azure Cosmos 容器。|
| [使用冲突解决创建容器](scripts/powershell/sql/ps-container-create-conflict-policy.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 使用“以最后写入者为准”冲突解决策略创建 Azure Cosmos 容器。|
| [列出所有容器](scripts/powershell/sql/ps-container-list.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 列出 Azure Cosmos 数据库中的所有容器。|
| [获取容器](scripts/powershell/sql/ps-container-get.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 获取 Azure Cosmos 数据库中容器的属性。|
| [删除容器](scripts/powershell/sql/ps-container-delete.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 删除 Azure Cosmos 数据库中的所有容器。|
|||