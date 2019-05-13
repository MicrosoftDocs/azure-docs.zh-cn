---
title: Azure Cosmos DB etcd API 简介
description: 本文提供了 Azure Cosmos DB 中的 etcd API 的概述和关键优势
author: deborahc
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: dech
ms.reviewer: sngun
ms.openlocfilehash: 6c7fcb1429438ee024cb226b63cfcdcab05ed9f8
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205809"
---
# <a name="introduction-to-the-azure-cosmos-db-etcd-api-preview"></a>Azure Cosmos DB etcd API （预览版） 简介

Azure Cosmos DB 是 Microsoft 的全球分布式多模型数据库服务的关键任务应用程序。 它提供统包全局分发、 弹性缩放吞吐量和存储，达 99%的个位数的毫秒级延迟以及得到保证的高可用性，所有这些均由行业领先的 SLA。

[Etcd](https://github.com/etcd-io/etcd)是分布式的键/值存储。 在中[Kubernetes](https://kubernetes.io/)，etcd 用于存储状态和 Kubernetes 群集的配置。 确保可用性、 可靠性和性能的 etcd 对群集整体运行状况、 可伸缩性、 弹性可用性和 Kubernetes 群集的性能至关重要。 

Azure Cosmos DB 中的 etcd API，可使用 Azure Cosmos DB 作为后端存储[Azure Kubernetes](../aks/index.yml)。 Azure Cosmos DB 中的 etcd API 目前处于预览状态。 Azure Cosmos DB 实现 etcd 线路协议。 Azure Cosmos DB 中 etcd API，开发人员将自动获得高度可靠[可用](high-availability.md)，[全球分布式](distribute-data-globally.md)Kubernetes。 此 API 允许开发人员在完全托管的云本机 PaaS 服务，可扩展 Kubernetes 状态管理。 

> [!NOTE]
> 与不同的 Azure Cosmos DB 中的其他 Api，则无法预配 etcd API 帐户通过 Azure 门户、 CLI 或 Sdk。 可以通过部署; 仅资源管理器模板来预配 etcd API 帐户有关详细步骤，请参阅[如何预配使用 Azure Cosmos DB 的 Azure Kubernetes](bootstrap-kubernetes-cluster.md)一文。 Azure Cosmos DB etcd API 当前处于受限预览状态。 你可以[注册预览版](https://aka.ms/cosmosetcdapi-signup)，通过填写注册表单。

## <a name="wire-level-compatibility"></a>网络级别的兼容性

Azure Cosmos DB 实施 etcd 版本 3，wire protocol，并且允许[主节点](https://kubernetes.io/docs/concepts/overview/components/)API 服务器，就像它像在本地安装的 etcd 环境中使用 Azure Cosmos DB。 Etcd API 支持 TLS 相互身份验证。 

下图显示了 Kubernetes 群集的组件。 在群集主机中，为 API 服务器而不是本地安装 etcd 使用 Azure Cosmos DB etcd API。 

![Azure Cosmos DB 实现 etcd 线路协议](./media/etcd-api-introduction/etcd-api-wire-protocol.png)

## <a name="key-benefits"></a>主要优点

### <a name="no-etcd-operations-management"></a>无需 etcd 操作管理

作为一种完全托管的本机云服务，Azure Cosmos DB 中删除 Kubernetes 开发人员设置和管理 etcd 需要。 Azure Cosmos DB 中的 API etcd 具有可缩放、 高度可用、 容错能力，并提供高性能。 开销的复制设置跨多个节点，执行滚动更新，安全修补程序，以及监视 etcd 运行状况由 Azure Cosmos DB 进行处理。

### <a name="global-distribution--high-availability"></a>全局分发和高可用性 

通过使用 etcd API，Azure Cosmos DB 来读取数据保证 99.99%的可用性、 跨多个区域写入中单个区域中，并提供 99.999%的可用性。 

### <a name="elastic-scalability"></a>弹性的可伸缩性

Azure Cosmos DB 提供灵活的可伸缩性进行读取和写入请求跨不同的区域。
随着 Kubernetes 群集时，Azure Cosmos DB 中的 etcd API 帐户灵活缩放而无需任何停机。 存储 etcd 数据在 Azure Cosmos DB 中，而不 Kubernetes 主节点还可以更灵活的主节点缩放。 

### <a name="security--enterprise-readiness"></a>安全和企业就绪情况

当 etcd 数据存储在 Azure Cosmos DB 时，Kubernetes 开发人员可以自动获得[静态的内置加密](database-encryption-at-rest.md)，[认证和法规遵从性](compliance.md)，和[备份和还原功能](online-backup-and-restore.md)Azure Cosmos DB 支持。 

## <a name="next-steps"></a>后续步骤

* [如何使用 Azure Cosmos DB 使用 Azure Kubernetes](bootstrap-kubernetes-cluster.md)
* [Azure Cosmos DB 的主要优点](introduction.md)
* [AKS 引擎快速入门指南](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/quickstart.md)