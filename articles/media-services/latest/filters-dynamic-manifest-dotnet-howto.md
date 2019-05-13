---
title: 使用 Azure 媒体服务 .NET SDK 创建筛选器
description: 本主题介绍如何创建筛选器，以便客户端能够使用它们来流式传输流的特定部分。 媒体服务将创建动态清单来存档此选择性流。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 05/07/2019
ms.author: juliako
ms.openlocfilehash: 8c786f46308848c6b9182453510744942a8eb9e8
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472437"
---
# <a name="create-filters-with-media-services-net-sdk"></a>使用媒体服务 .NET SDK 创建筛选器

将内容传送到客户（直播流事件或点播视频）时，客户端所需的灵活性可能比默认资产的清单文件中描述的灵活性更高。 使用 Azure 媒体服务可为内容定义帐户筛选器和资产筛选器。 有关详细信息，请参阅[筛选器和动态清单](filters-dynamic-manifest-overview.md)。

本主题展示了如何使用媒体服务 .NET SDK 为点播视频资产定义筛选器，以及如何创建[帐户筛选器](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.accountfilter?view=azure-dotnet)和[资产筛选器](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.assetfilter?view=azure-dotnet)。 

## <a name="prerequisites"></a>必备组件 

- 查看[筛选器和动态清单](filters-dynamic-manifest-overview.md)。
- [创建媒体服务帐户](create-account-cli-how-to.md)。 请务必记住资源组名称和媒体服务帐户名称。 
- 获取[访问 API](access-api-cli-how-to.md) 所需的信息
- 查看[使用 Azure 媒体服务进行上传、编码和流式传输](stream-files-tutorial-with-api.md)来了解如何[开始使用 .NET SDK](stream-files-tutorial-with-api.md#start_using_dotnet)

## <a name="define-a-filter"></a>定义筛选器  

在 .NET 中，你使用 [FilterTrackSelection](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.filtertrackselection?view=azure-dotnet) 和 [FilterTrackPropertyCondition](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.filtertrackpropertycondition?view=azure-dotnet) 类配置曲目选择。 

以下代码定义了一个筛选器，它包括属于 EC-3 的任何音频曲目和比特率在 0-1000000 范围内的任何视频曲目。

```csharp
var audioConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Audio", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.FourCC, "EC-3", FilterTrackPropertyCompareOperation.Equal)
};

var videoConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Video", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Bitrate, "0-1000000", FilterTrackPropertyCompareOperation.Equal)
};

List<FilterTrackSelection> includedTracks = new List<FilterTrackSelection>()
{
    new FilterTrackSelection(audioConditions),
    new FilterTrackSelection(videoConditions)
};
```

## <a name="create-account-filters"></a>创建帐户筛选器

以下代码展示了如何使用 .NET 创建一个帐户筛选器，使其包括[上面定义的](#define-a-filter)所有曲目选择。 

```csharp
AccountFilter accountFilterParams = new AccountFilter(tracks: includedTracks);
client.AccountFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, "accountFilterName1", accountFilter);
```

## <a name="create-asset-filters"></a>创建资产筛选器

以下代码展示了如何使用 .NET 创建一个资产筛选器，使其包括[上面定义的](#define-a-filter)所有曲目选择。 

```csharp
AssetFilter assetFilterParams = new AssetFilter(tracks: includedTracks);
client.AssetFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, encodedOutputAsset.Name, "assetFilterName1", assetFilterParams);
```

## <a name="associate-filters-with-streaming-locator"></a>将筛选器与流式处理定位符相关联

可以指定将适用于你的流式处理定位符的资产或帐户筛选器的列表。 [动态打包程序 （流式处理终结点）](dynamic-packaging-overview.md)适用此列表以及这些客户端在 URL 中指定的筛选器。 此组合生成[动态清单](filters-dynamic-manifest-overview.md)，后者基于在 URL 中的筛选器 + 流式处理定位符指定的筛选器。 我们建议你使用此功能，如果想要应用筛选器，但不是希望公开在 URL 中的筛选器名称。

以下C#的代码演示如何创建流式处理定位符，并指定`StreamingLocator.Filters`。 此为可选属性采用`IList<string>`的筛选器名称。

```csharp
IList<string> filters = new List<string>();
filters.Add("filterName");

StreamingLocator locator = await client.StreamingLocators.CreateAsync(
    resourceGroup,
    accountName,
    locatorName,
    new StreamingLocator
    {
        AssetName = assetName,
        StreamingPolicyName = PredefinedStreamingPolicy.ClearStreamingOnly,
        Filters = filters
    });
```
      
## <a name="stream-using-filters"></a>Stream 使用筛选器

定义筛选器后，客户端可以在流式 URL 中使用它们。 筛选器可以应用于自适应比特率流式处理协议：Apple HTTP Live Streaming (HLS)、MPEG-DASH 和平滑流式处理。

下表显示了一些包含筛选器的 URL 示例：

|Protocol|示例|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|平滑流式处理|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>后续步骤

[流视频](stream-files-tutorial-with-api.md) 


