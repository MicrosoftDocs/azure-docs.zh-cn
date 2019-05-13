---
title: Azure 应用程序配置常见问题 |Microsoft Docs
description: 有关 Azure 应用程序配置常见问题
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: 3181609bf34a04de4e31b73429f9bc5fa3fe3408
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411891"
---
# <a name="azure-app-configuration-faq"></a>Azure 应用程序配置常见问题

此文章地址频繁问题有关 Azure 应用程序配置。

## <a name="how-is-app-configuration-different-from-key-vault"></a>如何为应用配置不同于密钥保管库？

应用配置设计的一组不同的用例： 它可帮助开发人员管理应用程序设置和控制功能的可用性。 其目标是简化许多使用复杂的配置数据的任务。 它支持分层命名空间、 标记、 丰富查询、 批量检索和专用的管理操作和用户界面。 应用配置密钥保管库的补充，应在大多数应用程序部署中同时使用这两个。

## <a name="should-i-store-secrets-in-app-configuration"></a>应在应用程序配置中存储机密？

虽然应用配置提供了稳固的安全，密钥保管库仍是用于存储应用程序机密的最佳位置。 它提供硬件级别的加密、 精细的访问策略和管理操作，例如证书旋转。

## <a name="does-app-configuration-encrypt-my-data"></a>应用配置是否加密数据？

可以。 应用配置对所有密钥的值，它包含和网络通信进行加密。 密钥名称用作索引来检索配置数据，并且不加密。

## <a name="does-app-configuration-support-azure-virtual-network-vnet"></a>应用配置是否支持 Azure 虚拟网络 (VNET)？

还不可以。 VNET 支持为公开上市计划。

## <a name="how-should-i-store-configurations-for-multiple-environments-test-staging-production-and-so-on"></a>如何应存储多个环境 （测试、 过渡、 生产等） 的配置？

当前可以控制谁在每个存储级别有权访问应用程序配置。 为每个需要不同的权限的环境，应使用单独的存储库。 这种方法为您提供了最佳的安全隔离。

## <a name="what-are-the-recommended-ways-to-use-app-configuration"></a>将应用配置的建议的方式是什么？

请参阅[最佳做法](./howto-best-practices.md)。

## <a name="how-much-does-app-configuration-cost"></a>应用配置费用是多少？

该服务可以自由地在公共预览版中使用。

## <a name="how-can-i-report-an-issue-or-give-a-suggestion"></a>如何报告问题或提供建议？

您可以通过直接在我们[GitHub](https://github.com/Azure/AppConfiguration/issues)。

## <a name="next-steps"></a>后续步骤

* [有关 Azure 应用程序配置](./overview.md)
