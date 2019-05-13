---
title: Azure 存储指标迁移 | Microsoft Docs
description: 了解如何将旧指标迁移到 Azure Monitor 托管的新指标。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 03/30/2018
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: 58ac15c1aba715c9a5b67e723401b531e76608b2
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153600"
---
# <a name="azure-storage-metrics-migration"></a>Azure 存储指标迁移

按照 Azure 中统一监视体验的策略，Azure 存储会将指标集成到 Azure Monitor 平台。 将来，旧指标服务将会终止，并基于 Azure Policy 进行提前通知。 如果使用旧存储指标，则需要在服务终止日期前进行迁移，以便保留指标信息。

本文演示如何从旧指标迁移到新指标。

## <a name="understand-old-metrics-that-are-managed-by-azure-storage"></a>了解 Azure 存储托管的旧指标

Azure 存储收集旧指标值，将其聚合并存储在同一存储帐户内的 $Metric 表中。 可以使用 Azure 门户设置监视图表。 还可以使用 Azure 存储 SDK 从基于架构的 $Metric 表格读取数据。 有关详细信息，请参阅[存储分析](./storage-analytics.md)。

旧指标仅对 Azure Blob 存储提供容量指标。 旧指标对 Blob 存储、表存储、Azure 文件和队列存储提供事务指标。

旧指标采用平面架构设计。 当没有可触发指标的流量模式时，此设计会导致生成指标值 0。 例如，即使未在流向存储帐户的实时流量中收到任何服务器超时错误，$Metric 表中的 ServerTimeoutError 值也将设置为 0。

## <a name="understand-new-metrics-managed-by-azure-monitor"></a>了解 Azure Monitor 托管的新指标

对于新存储指标，Azure 存储会向 Azure Monitor 后端发出指标数据。 Azure Monitor 对通过门户和数据引入获取的数据提供统一的监视体验。 有关详细信息，请参阅此[文章](../../monitoring-and-diagnostics/monitoring-overview-metrics.md)。

新指标提供 Blob、表、文件、队列和高级存储上的容量指标和事务指标。

多维度是 Azure Monitor 提供的一种功能。 Azure 存储采用这一设计来定义新指标架构。 有关指标支持的维度的详细信息，请参阅 [Azure Monitor 中的 Azure 存储指标](./storage-metrics-in-azure-monitor.md)。 多维设计可节约存储指标的带宽（包括引入和容量两方面）成本。 因此，如果流量未触发相关指标，则不会生成相关指标数据。 例如，如果流量未触发任何服务器超时错误，当查询事务指标（维度 ResponseType 等于 ServerTimeoutError）的值时，Azure Monitor 不会返回任何数据。

## <a name="metrics-mapping-between-old-metrics-and-new-metrics"></a>旧指标和新指标之间的指标映射

如果以编程方式读取指标数据，则需要在程序中采用新指标架构。 为了更好地了解相关更改，可以参考下表中列出的映射：

**容量指标**

| 旧指标 | 新指标 |
| ------------------- | ----------------- |
| **容量**            | BlobCapacity（维度 BlobType 等于 BlockBlob 或 PageBlob） |
| **ObjectCount**        | BlobCount（维度 BlobType 等于 BlockBlob 或 PageBlob） |
| **ContainerCount**      | **ContainerCount** |

以下是旧指标不支持的新指标：
* **TableCapacity**
* **TableCount**
* **TableEntityCount**
* **QueueCapacity**
* **QueueCount**
* **QueueMessageCount**
* **FileCapacity**
* **FileCount**
* **FileShareCount**
* **UsedCapacity**

**事务指标**

| 旧指标 | 新指标 |
| ------------------- | ----------------- |
| **AnonymousAuthorizationError** | 维度 **ResponseType** 等于 **AuthorizationError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousClientOtherError** | 维度 **ResponseType** 等于 **ClientOtherError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousClientTimeoutError** | 维度 **ResponseType** 等于 **ClientTimeoutError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousNetworkError** | 维度 **ResponseType** 等于 **NetworkError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousServerOtherError** | 维度 **ResponseType** 等于 **ServerOtherError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousServerTimeoutError** | 维度 **ResponseType** 等于 **ServerTimeoutError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousSuccess** | 维度 **ResponseType** 等于 **Success** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AnonymousThrottlingError** | 维度 **ResponseType** 等于 **ClientThrottlingError** 或 **ServerBusyError** 且维度 **Authentication** 等于 **Anonymous** 的事务 |
| **AuthorizationError** | 事务（维度 ResponseType 等于 AuthorizationError） |
| **可用性** | **可用性** |
| **AverageE2ELatency** | **SuccessE2ELatency** |
| **AverageServerLatency** | **SuccessServerLatency** |
| **ClientOtherError** | 事务（维度 ResponseType 等于 ClientOtherError） |
| **ClientTimeoutError** | 事务（维度 ResponseType 等于 ClientTimeoutError） |
| **NetworkError** | 事务（维度 ResponseType 等于 NetworkError） |
| **PercentAuthorizationError** | 事务（维度 ResponseType 等于 AuthorizationError） |
| **PercentClientOtherError** | 事务（维度 ResponseType 等于 ClientOtherError） |
| **PercentNetworkError** | 事务（维度 ResponseType 等于 NetworkError） |
| **PercentServerOtherError** | 事务（维度 ResponseType 等于 ServerOtherError） |
| **PercentSuccess** | 事务（维度 ResponseType 等于 Success） |
| **PercentThrottlingError** | 事务（维度 ResponseType 等于 ClientThrottlingError 或 ServerBusyError） |
| **PercentTimeoutError** | 事务（维度 ResponseType 等于 ServerTimeoutError 或 ResponseType 等于 ClientTimeoutError） |
| **SASAuthorizationError** | 维度 **ResponseType** 等于 **AuthorizationError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASClientOtherError** | 维度 **ResponseType** 等于 **ClientOtherError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASClientTimeoutError** | 维度 **ResponseType** 等于 **ClientTimeoutError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASNetworkError** | 维度 **ResponseType** 等于 **NetworkError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASServerOtherError** | 维度 **ResponseType** 等于 **ServerOtherError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASServerTimeoutError** | 维度 **ResponseType** 等于 **ServerTimeoutError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASSuccess** | 维度 **ResponseType** 等于 **Success** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **SASThrottlingError** | 维度 **ResponseType** 等于 **ClientThrottlingError** 或 **ServerBusyError** 且维度 **Authentication** 等于 **SAS** 的事务 |
| **ServerOtherError** | 事务（维度 ResponseType 等于 ServerOtherError） |
| **ServerTimeoutError** | 事务（维度 ResponseType 等于 ServerTimeoutError） |
| **Success** | 事务（维度 ResponseType 等于 Success） |
| **ThrottlingError** | 事务（维度 ResponseType 等于 ClientThrottlingError 或 ServerBusyError）|
| **TotalBillableRequests** | **Transactions** |
| **TotalEgress** | **流出量** |
| **TotalIngress** | **流入量** |
| **TotalRequests** | **Transactions** |

## <a name="faq"></a>常见问题解答

### <a name="how-should-i-migrate-existing-alert-rules"></a>应如何迁移现有警报规则？

如果基于旧存储指标创建了经典警报规则，则需要基于新指标架构创建新警报规则。

### <a name="is-new-metric-data-stored-in-the-same-storage-account-by-default"></a>新指标数据是否会默认存储在同一存储帐户中？

不。 若要将指标数据存档到存储帐户，请使用 [Azure Monitor 诊断设置 API](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings/createorupdate)。

## <a name="next-steps"></a>后续步骤

* [Azure Monitor](../../monitoring-and-diagnostics/monitoring-overview.md)
* [Azure Monitor 中的存储指标](./storage-metrics-in-azure-monitor.md)
