---
title: Azure Monitor 中的 Azure 存储指标 | Microsoft Docs
description: 了解 Azure Monitor 提供的新指标。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 09/05/2017
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: 8b091ecce98a626f18fe6547445d898b6710e1a5
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510550"
---
# <a name="azure-storage-metrics-in-azure-monitor"></a>Azure Monitor 中的 Azure 存储指标

使用 Azure 存储的指标可以分析使用趋势、跟踪请求，以及诊断存储帐户的问题。

Azure Monitor 提供统一的用户界面用于监视不同的 Azure 服务。 有关详细信息，请参阅 [Azure Monitor](../../monitoring-and-diagnostics/monitoring-overview.md)。 Azure 存储通过向 Azure Monitor 平台发送指标数据来与 Azure Monitor 集成。

## <a name="access-metrics"></a>访问指标

Azure Monitor 提供多种访问指标的方法。 你可以访问它们从[Azure 门户](https://portal.azure.com)，Azure Monitor Api （REST 和.NET） 与分析解决方案，例如事件中心。 有关详细信息，请参阅 [Azure Monitor 指标](../../monitoring-and-diagnostics/monitoring-overview-metrics.md)。

默认情况下，已启用指标，并且可访问过去 93 天的数据。 如需将数据保留更长一段时间，可将指标数据存档到 Azure 存储帐户。 可在 Azure Monitor 的 [诊断设置](../../azure-monitor/platform/diagnostic-logs-overview.md) 中完成这种配置。

### <a name="access-metrics-in-the-azure-portal"></a>在 Azure 门户中访问指标

可在 Azure 门户中访问不同时间的指标。 以下示例演示如何查看帐户级别的**事务**。

![在 Azure 门户中访问指标的屏幕截图](./media/storage-metrics-in-azure-monitor/access-metrics-in-portal.png)

对于支持维度的指标，可使用所需的维度值筛选指标。 下面的示例演示了如何通过选择“API 名称”维度的值，在特定操作上查看帐户级别的“事务”。

![在 Azure 门户中访问包含维度的指标的屏幕截图](./media/storage-metrics-in-azure-monitor/access-metrics-in-portal-with-dimension.png)

### <a name="access-metrics-with-the-rest-api"></a>使用 REST API 访问指标

Azure Monitor 提供 [REST API](/rest/api/monitor/) 用于读取指标定义和值。 本部分介绍如何读取存储指标。 所有 REST API 中使用了资源 ID。 有关详细信息，请阅读“了解存储中服务的资源 ID”。

以下示例演示如何在命令行中使用 [ArmClient](https://github.com/projectkudu/ARMClient)，借助 REST API 来简化测试。

#### <a name="list-account-level-metric-definition-with-the-rest-api"></a>使用 REST API 列出帐户级指标定义

以下示例演示如何列出帐户级别的指标定义：

```
# Login to Azure and enter your credentials when prompted.
> armclient login

> armclient GET /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metricdefinitions?api-version=2018-01-01

```

如果想要列出 Blob、表、文件或队列的指标定义，必须使用 API 指定每个服务的不同资源 ID。

响应包含 JSON 格式的指标定义：

```Json
{
  "value": [
    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metricdefinitions/UsedCapacity",
      "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}",
      "category": "Capacity",
      "name": {
        "value": "UsedCapacity",
        "localizedValue": "Used capacity"
      },
      "isDimensionRequired": false,
      "unit": "Bytes",
      "primaryAggregationType": "Average",
      "metricAvailabilities": [
        {
          "timeGrain": "PT1M",
          "retention": "P30D"
        },
        {
          "timeGrain": "PT1H",
          "retention": "P30D"
        }
      ]
    },
    ... next metric definition
  ]
}

```

#### <a name="read-account-level-metric-values-with-the-rest-api"></a>使用 REST API 读取帐户级指标值

以下示例演示如何读取帐户级别的指标数据：

```
> armclient GET "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metrics?metricnames=Availability&api-version=2018-01-01&aggregation=Average&interval=PT1H"

```

在上述示例中，如果想要读取 Blob、表、文件或队列的指标值，必须使用 API 指定每个服务的不同资源 ID。

以下响应包含 JSON 格式的指标值：

```Json
{
  "cost": 0,
  "timespan": "2017-09-07T17:27:41Z/2017-09-07T18:27:41Z",
  "interval": "PT1H",
  "value": [
    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/Microsoft.Insights/metrics/Availability",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Availability",
        "localizedValue": "Availability"
      },
      "unit": "Percent",
      "timeseries": [
        {
          "metadatavalues": [],
          "data": [
            {
              "timeStamp": "2017-09-07T17:27:00Z",
              "average": 100.0
            }
          ]
        }
      ]
    }
  ]
}

```

### <a name="access-metrics-with-the-net-sdk"></a>使用.NET SDK 访问指标

Azure Monitor 提供[.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/)用于读取指标定义和值。 [示例代码](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/)演示如何通过不同的参数来使用 SDK。 对于存储指标，需使用 `0.18.0-preview` 或更高版本。 在.NET SDK 中使用资源 ID。 有关详细信息，请阅读“了解存储中服务的资源 ID”。

下面的示例演示如何使用 Azure Monitor.NET SDK 来读取存储指标。

#### <a name="list-account-level-metric-definition-with-the-net-sdk"></a>列出帐户级指标定义使用.NET SDK

以下示例演示如何列出帐户级别的指标定义：

```csharp
    public static async Task ListStorageMetricDefinition()
    {
        // Resource ID for storage account
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}";
        var subscriptionId = "{SubscriptionID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "{TenantID}";
        var applicationId = "{ApplicationID}";
        var accessKey = "{AccessKey}";

        // Using metrics in Azure Monitor is currently free. However, if you use additional solutions ingesting metrics data, you may be billed by these solutions. For example, you are billed by Azure Storage if you archive metrics data to an Azure Storage account. Or you are billed by Operation Management Suite (OMS) if you stream metrics data to OMS for advanced analysis.
        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;
        IEnumerable<MetricDefinition> metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceId, cancellationToken: new CancellationToken());

        foreach (var metricDefinition in metricDefinitions)
        {
            //Enumrate metric definition:
            //    Id
            //    ResourceId
            //    Name
            //    Unit
            //    MetricAvailabilities
            //    PrimaryAggregationType
            //    Dimensions
            //    IsDimensionRequired
        }
    }

```

如果想要列出 Blob、表、文件或队列的指标定义，必须使用 API 指定每个服务的不同资源 ID。

#### <a name="read-metric-values-with-the-net-sdk"></a>使用.NET SDK 读取指标值

以下示例演示如何读取帐户级别的 `UsedCapacity` 数据：

```csharp
    public static async Task ReadStorageMetricValue()
    {
        // Resource ID for storage account
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}";
        var subscriptionId = "{SubscriptionID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "{TenantID}";
        var applicationId = "{ApplicationID}";
        var accessKey = "{AccessKey}";

        MonitorClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;

        Response = await readOnlyClient.Metrics.ListAsync(
            resourceUri: resourceId,
            timespan: timeSpan,
            interval: System.TimeSpan.FromHours(1),
            metricnames: "UsedCapacity",

            aggregation: "Average",
            resultType: ResultType.Data,
            cancellationToken: CancellationToken.None);

        foreach (var metric in Response.Value)
        {
            //Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

在上述示例中，如果想要读取 Blob、表、文件或队列的指标值，必须使用 API 指定每个服务的不同资源 ID。

#### <a name="read-multi-dimensional-metric-values-with-the-net-sdk"></a>使用.NET SDK 读取多维指标值

对于多维指标，如果需要读取基于特定维度值的指标数据，则需定义元数据筛选器。

以下示例演示如何根据支持多维的指标读取指标数据：

```csharp
    public static async Task ReadStorageMetricValueTest()
    {
        // Resource ID for blob storage
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/blobServices/default";
        var subscriptionId = "{SubscriptionID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "{TenantID}";
        var applicationId = "{ApplicationID}";
        var accessKey = "{AccessKey}";

        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;
        // It's applicable to define meta data filter when a metric support dimension
        // More conditions can be added with the 'or' and 'and' operators, example: BlobType eq 'BlockBlob' or BlobType eq 'PageBlob'
        ODataQuery<MetadataValue> odataFilterMetrics = new ODataQuery<MetadataValue>(
            string.Format("BlobType eq '{0}'", "BlockBlob"));

        Response = readOnlyClient.Metrics.List(
                        resourceUri: resourceId,
                        timespan: timeSpan,
                        interval: System.TimeSpan.FromHours(1),
                        metricnames: "BlobCapacity",
                        odataQuery: odataFilterMetrics,
                        aggregation: "Average",
                        resultType: ResultType.Data);

        foreach (var metric in Response.Value)
        {
            //Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

## <a name="understanding-resource-id-for-services-in-azure-storage"></a>了解 Azure 存储中服务的资源 ID

资源 ID 是资源在 Azure 中的唯一标识符。 使用 Azure Monitor REST API 读取指标定义或值时，必须使用要对其执行操作的资源的资源 ID。 资源 ID 模板遵循以下格式：

```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

存储通过 Azure Monitor 提供存储帐户级别和服务级别的指标。 例如，可以仅检索 Blob 存储的指标。 每个级别有其自身的资源 ID，用于仅检索该级别的指标。

### <a name="resource-id-for-a-storage-account"></a>存储帐户的资源 ID

下面显示了指定存储帐户的资源 ID 时所用的格式。

```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}
```

### <a name="resource-id-for-the-storage-services"></a>存储服务的资源 ID

下面显示了指定每个存储服务的资源 ID 时所用的格式。

* Blob 服务资源 ID
```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/blobServices/default
```
* 表服务资源 ID
```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/tableServices/default
```
* 队列服务资源 ID
```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/queueServices/default
```
* 文件服务资源 ID
```
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/fileServices/default
```

### <a name="resource-id-in-azure-monitor-rest-api"></a>Azure Monitor REST API 中的资源 ID

下面显示了调用 Azure Monitor REST API 时所用的模式。

```
GET {resourceId}/providers/microsoft.insights/metrics?{parameters}
```

## <a name="capacity-metrics"></a>容量度量值
容量指标每隔一小时发送到 Azure Monitor。 值每日刷新。 时间粒度定义呈现指标值的时间间隔。 所有容量指标的受支持时间粒度为一小时 (PT1H)。

Azure 存储在 Azure Monitor 中提供以下容量指标。

### <a name="account-level"></a>帐户级别

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| UsedCapacity | 存储帐户使用的存储量。 对于标准存储帐户，该指标是 Blob、表、文件和队列使用的容量总和。 对于高级存储帐户和 Blob 存储帐户，它与 BlobCapacity 相同。 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 |

### <a name="blob-storage"></a>Blob 存储

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| BlobCapacity | 存储帐户中使用的 Blob 存储总计。 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 <br/> 尺寸：**BlobType**，并**BlobTier** ([定义](#metrics-dimensions)) |
| BlobCount    | 在存储帐户中存储的 Blob 对象数。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 <br/> 尺寸：**BlobType**，并**BlobTier** ([定义](#metrics-dimensions)) |
| ContainerCount    | 存储帐户中的容器数。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| IndexCapacity     | ADLS Gen2 分层索引所使用的存储量 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 |

### <a name="table-storage"></a>表存储

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| TableCapacity | 存储帐户使用的表存储量。 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| TableCount   | 存储帐户中的表数目。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| TableEntityCount | 存储帐户中的表实体数目。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |

### <a name="queue-storage"></a>队列存储

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| QueueCapacity | 存储帐户使用的队列存储量。 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| QueueCount   | 存储帐户中的队列数目。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| QueueMessageCount | 存储帐户中未失效的队列消息数目。 <br/><br/>单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |

### <a name="file-storage"></a>文件存储

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| FileCapacity | 存储帐户使用的文件存储量。 <br/><br/> 单位：字节 <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| FileCount   | 存储帐户中的文件数目。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |
| FileShareCount | 存储帐户中的文件共享数目。 <br/><br/> 单位：Count <br/> 聚合类型：平均值 <br/> 值示例：1024 |

## <a name="transaction-metrics"></a>事务度量值

从 Azure 存储到 Azure Monitor 的每个存储帐户请求都会发出事务指标。 如果存储帐户中没有任何活动，则在此期间不会有关于事务指标的数据。 所有事务指标在帐户和服务级别（Blob 存储、表存储、Azure 文件和队列存储）提供。 时间粒度定义呈现指标值的时间间隔。 所有事务指标的受支持时间粒度为 PT1H 和 PT1M。

Azure 存储在 Azure Monitor 中提供以下事务指标。

| 指标名称 | 描述 |
| ------------------- | ----------------- |
| 事务 | 向存储服务或指定的 API 操作发出的请求数。 此数字包括成功和失败的请求数，以及引发错误的请求数。 <br/><br/> 单位：Count <br/> 聚合类型：总计 <br/> 适用维度：ResponseType、GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)）<br/> 值示例：1024 |
| 流入量 | 流入数据量。 此数值包括从外部客户端到 Azure 存储流入的数据量，以及流入 Azure 中的数据量。 <br/><br/> 单位：字节 <br/> 聚合类型：总计 <br/> 适用维度：GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| 流出量 | 流出数据量。 此数字包括从外部客户端到 Azure 存储流出的数据量，以及流出 Azure 中的数据量。 因此，此数值不反映计费的流出量。 <br/><br/> 单位：字节 <br/> 聚合类型：总计 <br/> 适用维度：GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| SuccessServerLatency | Azure 存储处理成功请求所用的平均时间。 此值不包括 SuccessE2ELatency 中指定的网络延迟。 <br/><br/> 单位：毫秒 <br/> 聚合类型：平均值 <br/> 适用维度：GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| SuccessE2ELatency | 向存储服务或指定的 API 操作发出的成功请求的平均端到端延迟。 此值包括在 Azure 存储中读取请求、发送响应和接收响应确认所需的处理时间。 <br/><br/> 单位：毫秒 <br/> 聚合类型：平均值 <br/> 适用维度：GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| 可用性 | 存储服务或指定的 API 操作的可用性百分比。 可用性通过由“计费请求总数”值除以适用的请求数（包括引发意外错误的请求）计算得出。 所有意外错误都会导致存储服务或指定的 API 操作的可用性下降。 <br/><br/> 单位：百分比 <br/> 聚合类型：平均值 <br/> 适用维度：GeoType、ApiName 和 Authentication（[定义](#metrics-dimensions)） <br/> 值示例：99.99 |

## <a name="metrics-dimensions"></a>指标维度

Azure 存储支持对 Azure Monitor 中的指标使用以下维度。

| 维度名称 | 描述 |
| ------------------- | ----------------- |
| **BlobType** | 仅限 Blob 指标的 Blob 类型。 支持的值为**BlockBlob**， **PageBlob**，并**Azure Data Lake 存储**。 BlockBlob 中包含追加 Blob。 |
| **BlobTier** | Azure 存储提供了不同的访问层，允许以最具成本效益的方式存储 Blob 对象数据。 请参阅中的详细信息[Azure 存储 blob 层](../blobs/storage-blob-storage-tiers.md)。 支持的值包括： <br/> <li>**热**:热层</li> <li>**冷**:冷层</li> <li>**存档**：存档层</li> <li>**高级**:块 blob 的高级层</li> <li>**P4/P6/P10/P15/P20/P30/P40/P50/P60**:高级页 blob 的层类型</li> <li>**标准**：标准页 Blob 的层类型</li> <li>**Untiered**:常规用途 v1 存储帐户的层类型</li> |
| **GeoType** | 来自主要或辅助群集的事务。 可用值包括**主**并**辅助**。 从辅助租户读取对象时，该维度会应用到读取访问异地冗余存储 (RA-GRS)。 |
| **ResponseType** | 事务响应类型。 可用的值包括： <br/><br/> <li>**ServerOtherError**：除描述的错误以外的其他所有服务器端错误 </li> <li>**ServerBusyError**：返回了 HTTP 503 状态代码的已经过身份验证的请求。 </li> <li>**ServerTimeoutError**：返回了 HTTP 500 状态代码的超时已经过身份验证的请求。 由于服务器错误而发生超时。 </li> <li>**AuthorizationError**：由于未经授权访问数据或者授权失败，经过身份验证的请求失败。 </li> <li>**NetworkError**：由于网络错误，经过身份验证的请求失败。 往往发生于客户端在超时失效之前提前关闭了连接时。 </li> <li>**ClientThrottlingError**：客户端限制错误。 </li> <li>**ClientTimeoutError**：返回了 HTTP 500 状态代码的超时已经过身份验证的请求。 如果将客户端的网络超时或请求超时设置为比存储服务预期值更小的值，则预期会发生此超时。 否则，会报告为 ServerTimeoutError。 </li> <li>**ClientOtherError**：除描述的错误以外的其他所有客户端错误。 </li> <li>**成功**：请求成功</li> <li> **SuccessWithThrottling**:成功的请求时 SMB 客户端在第一个尝试获取受限，但重试后成功。</li> |
| **ApiName** | 操作的名称。 例如: <br/> <li>**CreateContainer**</li> <li>**DeleteBlob**</li> <li>**GetBlob**</li> 有关所有操作名称，请参阅[文档](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)。 |
| **身份验证** | 事务中所用的身份验证类型。 可用的值包括： <br/> <li>**AccountKey**:事务通过存储帐户密钥进行身份验证。</li> <li>**SAS**:事务通过共享访问签名进行身份验证。</li> <li>**OAuth**:事务通过 OAuth 访问令牌进行身份验证。</li> <li>**匿名**:事务以匿名方式请求。 不包括预检请求。</li> <li>**AnonymousPreflight**:事务为预检请求。</li> |

对于支持维度的指标，需要指定维度值才能查看相应的指标值。 例如，如果查看成功响应的 **Transactions** 值，需要使用 **Success** 筛选 **ResponseType** 维度。 或者，如果查看块 Blob 的 **BlobCount** 值，需要使用 **BlockBlob** 筛选 **BlobType** 维度。

## <a name="service-continuity-of-legacy-metrics"></a>旧指标的服务连续性

旧指标可与 Azure Monitor 托管的指标一同使用。 在 Azure 存储终止旧指标的服务之前，支持范围保持不变。

## <a name="faq"></a>常见问题解答

**新指标是否支持经典存储帐户？**

否，Azure Monitor 中的新指标仅支持 Azure 资源管理器存储帐户。 如果要使用存储帐户上的指标，则需要迁移到 Azure 资源管理器存储帐户。 请参阅[迁移到 Azure 资源管理器](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-overview)。

**Azure 存储是否支持托管磁盘或非托管磁盘的指标？**

否，Azure 计算支持磁盘上的指标。 有关详细信息，请参阅此[文章](https://azure.microsoft.com/blog/per-disk-metrics-managed-disks/)。

**如何使用新指标映射和迁移经典指标？**

可以在 [Azure 存储指标迁移](./storage-metrics-migration.md)中找到经典指标与新指标之间的详细映射。

## <a name="next-steps"></a>后续步骤

* [Azure Monitor](../../monitoring-and-diagnostics/monitoring-overview.md)
