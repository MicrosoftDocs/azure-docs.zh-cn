---
title: Azure 区块链服务支持分类帐版本、 修补和升级
description: 在 Azure 区块链服务，包括有关修补和系统管理系统和用户管理升级策略中的受支持的分类帐版本的概述。
services: azure-blockchain
keywords: 区块链
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: janders
manager: femila
ms.openlocfilehash: 63c9a8b9e266dacbb0fb6faba50fb44ac9a4b46e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65027880"
---
# <a name="supported-azure-blockchain-service-ledger-versions"></a>支持的 Azure 区块链服务分类帐版本

Azure 区块链服务使用基于以太坊[仲裁](https://github.com/jpmorganchase/quorum/wiki)为已知参与方，标识为联盟 Azure 区块链服务中的组中的专用事务的处理而设计的分类帐。

目前，Azure 区块链服务支持[仲裁 2.2.1 版本](https://github.com/jpmorganchase/quorum/releases/tag/v2.2.1)并[Tessera 事务管理器](https://github.com/jpmorganchase/tessera)。

## <a name="managing-updates-and-upgrades"></a>管理更新和升级

仲裁中的版本控制是通过主版本、 主要点版本和次要点版本。 例如，如果仲裁版本 2.0.1 开始，2 是主要版本，0 是主要的点的发行版，并且 1 是一点小版本。 该服务会自动修补次要的单点发行版的分类帐。 目前，不支持升级的主版本号和主要的单点发行版。

## <a name="availability-of-new-ledger-versions"></a>新的分类帐版本

Azure 区块链服务从分类帐制造商的 60 天内在提供最新版本的分类帐。 为财团预配新的成员和联盟时可以选择提供了最多四个主要的单点发行版。

## <a name="next-steps"></a>后续步骤

[在 Azure 区块链服务中的限制](limits.md)
