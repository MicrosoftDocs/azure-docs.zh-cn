---
title: Azure 区块链限制
description: 服务和 Azure 区块链服务中的功能限制概述
services: azure-blockchain
keywords: 区块链
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: janders
manager: femila
ms.openlocfilehash: 169ec7a8ef407af3f754046aa8e3b06793a7e962
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028165"
---
# <a name="limits-in-azure-blockchain-service"></a>在 Azure 区块链服务中的限制

Azure 区块链服务提供服务和功能限制，例如成员可以具有的节点、 联盟限制和存储量的数目。

## <a name="pricing-tier"></a>定价层

事务和验证程序节点的最大限制取决于是否预配 Azure 区块链服务基本或标准定价层。

| 定价层 | 节点的最大事务 | 验证程序节点的最大 |
|:---|:---:|:---:|
| 基本 | 10 | 1 |
| 标准 | 10 | 2 |

不支持成员的创建后更改定价层之间基本和标准。

## <a name="storage-capacity"></a>存储容量

最大的会计数据和日志可以每个节点中使用的存储量为 1tb。

不支持减小分类帐和日志存储大小。

## <a name="consortium-limits"></a>联盟限制

* **协会以及成员名称必须是唯一**从 Azure 区块链服务中的其他联合会和成员名称。

* **不能更改成员和联盟的名称**

* **联盟中的所有成员必须都在相同的定价层**

* **加入联盟的所有成员必须都位于同一区域**

    在联盟中创建的第一个成员决定了区域。 已邀请的成员到联盟必须驻留在第一个成员所在的同一区域中。 限制到同一区域的所有成员，可帮助确保网络达成一致意见不产生负面影响。

* **联盟必须具有至少一个管理员**

    如果在联盟中只有一个管理员，不能将自己从联盟移除或删除其成员，直到添加或升级在联盟中其他管理员。

* **删除从联盟的成员不能再次添加**

    相反，它们必须重新邀请加入联盟并创建新的成员。 不会删除其现有的成员资源，以保留历史交易记录。

* **联盟中的所有成员必须都使用相同的分类帐版本**

    修补、 更新和分类帐版本 Azure 区块链服务中提供的详细信息，请参阅[修补、 更新和版本](ledger-versions.md)。

## <a name="next-steps"></a>后续步骤

* [修补、 更新和版本](ledger-versions.md)
