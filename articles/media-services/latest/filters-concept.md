---
title: 在 Azure 媒体服务中定义筛选器
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
ms.openlocfilehash: 3a562f98635d581aa320fdbd59d05a0382f09606
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465531"
---
# <a name="define-account-filters-and-asset-filters"></a>定义帐户筛选器和资产筛选器  

你将内容传送到客户 （实时流式处理事件或视频点播） 时你的客户端可能需要更大的灵活性比默认资产的清单文件中描述的内容。 使用 Azure 媒体服务可为内容定义帐户筛选器和资产筛选器。 

筛选器是服务器端的规则，可让客户执行以下操作： 

- 仅播放视频的某个部分（而不是整个视频）。 例如:
  - 缩小清单以显示实时事件的子剪辑（“子剪辑筛选”），或
  - 修剪视频开头（“修剪视频”）。
- 只传送内容播放设备所支持的指定再现内容和/或指定的语言轨道（“再现内容筛选”）。 
- 调整演播窗口，以便在播放器中提供长度有限的 DVR 窗口（“调整演播窗口”）。

媒体服务根据预定义的筛选器提供[动态清单](filters-dynamic-manifest-overview.md)。 定义筛选器后，客户端可以在流式 URL 中使用它们。 筛选器可以应用于自适应比特率流式处理协议：Apple HTTP Live Streaming (HLS)、MPEG-DASH 和平滑流式处理。

下表显示了一些包含筛选器的 URL 示例：

|Protocol|示例|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`<br/>对于 HLS v3，请使用： `format=m3u8-aapl-v3`。|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|平滑流式处理|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="define-filters"></a>定义筛选器

有两种类型的资产筛选器： 

* [帐户筛选器](https://docs.microsoft.com/rest/api/media/accountfilters)（全局）- 可应用到 Azure 媒体服务帐户中所有的资产，生存期与帐户相同。
* [资产筛选器](https://docs.microsoft.com/rest/api/media/assetfilters)（本地）- 在创建后只能应用到与筛选器关联的资产，生存期与资产相同。 

[帐户筛选器](https://docs.microsoft.com/rest/api/media/accountfilters)和[资产筛选器](https://docs.microsoft.com/rest/api/media/assetfilters)类型的用于定义/描述筛选器的属性完全相同。 需要指定要与筛选器关联的资产名称，但创建**资产筛选器**时除外。

根据具体的方案确定哪种类型的筛选器更合适（资产筛选器或帐户筛选器）。 帐户筛选器适用于设备配置文件（再现内容筛选），而资产筛选器可用于修剪特定的资产。

使用以下属性来描述筛选器。 

|名称|描述|
|---|---|
|firstQuality|筛选器的第一个质量比特率。|
|presentationTimeRange|呈现时间范围。 此属性用于筛选清单起点/终点、呈现窗口长度和直播起始位置。 <br/>有关详细信息，请参阅 [PresentationTimeRange](#presentationtimerange)。|
|tracks|轨迹选择条件。 有关详细信息，请参阅[轨迹](#tracks)|

### <a name="presentationtimerange"></a>presentationTimeRange

请将此属性用于**资产筛选器**。 不建议对**帐户筛选器**设置该属性。

|名称|描述|
|---|---|
|**endTimestamp**|适用于点播视频 (VoD)。<br/>对于实时流式处理演示中，它是以无提示方式忽略，并且应用时演示结束并且流变为 VoD。<br/>这是一个长值，表示演示文稿，舍入为最接近的下一个 GOP 起始的绝对结束点。 单位为时间刻度，因此 1800000000 endTimestamp 将是 3 分钟内。<br/>使用 startTimestamp 和 endTimestamp 来修剪播放列表 （清单） 中的片段。<br/>例如，startTimestamp = 40000000 和 endTimestamp = 100000000 使用默认时间刻度将生成包含介于 4 秒 （） 和 VoD 演示文稿的 10 秒的片段播放列表。 如果某个段跨越边界，则整个段将包含在清单中。|
|**forceEndTimestamp**|适用于实时传送视频流仅。<br/>指示是否必须存在 endTimestamp 属性。 如果为 true，则必须指定 endTimestamp 或返回了错误的请求代码。<br/>允许的值：false、true。|
|**liveBackoffDuration**|适用于实时传送视频流仅。<br/> 此值定义客户端可以查找到的最新实时位置。<br/>使用此属性，可以延迟实时播放位置并为播放器创建服务器端缓冲区。<br/>此属性的单位是时间刻度 （见下文）。<br/>实时回退持续时间的最大值为 300 秒 (3000000000)。<br/>例如，值为最新可用的内容为 20 秒的 2000000000 表示延迟从真正实时边缘。|
|**presentationWindowDuration**|适用于实时传送视频流仅。<br/>使用 presentationWindowDuration 应用的片段播放列表中包括的滑动窗口。<br/>此属性的单位是时间刻度 （见下文）。<br/>例如，设置 presentationWindowDuration=1200000000 会应用 2 分钟的滑动窗口。 直播边缘 2 分钟内的媒体将包含在播放列表中。 如果某个段跨越边界，则整个段将包含在播放列表中。 最小呈现窗口持续时间为 60 秒。|
|**startTimestamp**|适用于按需 (VoD) 或实时传送视频流视频。<br/>这是一个长值，表示流的绝对起始点。 该值将舍入为最接近的下一个 GOP 起点。 单位为时间刻度，因此 150000000 startTimestamp 应为 15 秒。<br/>使用 startTimestamp 和 endTimestampp 来修剪播放列表 （清单） 中的片段。<br/>例如，startTimestamp = 40000000 和 endTimestamp = 100000000 使用默认时间刻度将生成包含介于 4 秒 （） 和 VoD 演示文稿的 10 秒的片段播放列表。 如果某个段跨越边界，则整个段将包含在清单中。|
|**timescale**|在演示文稿时间范围，在一秒中指定的增量数为适用于所有时间戳和持续时间。<br/>默认值为每个增量将 100 纳秒长的一秒内一千万 10000000 的增量。<br/>例如，如果你想要设置为 30 秒 startTimestamp，您将使用 300000000 值时使用的默认时间刻度。|

### <a name="tracks"></a>轨迹

您指定的筛选器跟踪属性条件 (FilterTrackPropertyConditions) 列表基于在其上轨道的流 （实时流式处理或点播视频） 应包含到动态创建的清单。 使用逻辑 **AND** 和 **OR** 运算来组合筛选器。

筛选器轨迹属性条件描述轨迹类型、值（如下表所述）和运算（Equal、NotEqual）。 

|名称|描述|
|---|---|
|**Bitrate**|使用轨迹的比特率进行筛选。<br/><br/>建议的值为一系列比特率，以比特/秒为单位。 例如“0-2427000”。<br/><br/>注意：尽管可以使用特定的比特率值（例如 250000 比特/秒），但不建议使用此方法，因为确切的比特率可能根据资产的不同而波动。|
|**FourCC**|使用轨迹的 FourCC 值进行筛选。<br/><br/>该值是 [RFC 6381](https://tools.ietf.org/html/rfc6381) 中指定的编解码器格式的第一个元素。 目前支持以下编解码器： <br/>视频：“avc1”、“hev1”、“hvc1”<br/>音频：“mp4a”、“ec-3”<br/><br/>若要确定资产中的轨道 FourCC 值，获取并检查清单文件。|
|**语言**|使用轨迹的语言进行筛选。<br/><br/>该值是 RFC 5646 中指定的、要包含的语言的标记。 例如，“en”。|
|**名称**|使用轨迹的名称进行筛选。|
|类型|使用轨迹的类型进行筛选。<br/><br/>允许以下值：“video”、“audio”或“text”。|

## <a name="associate-filters-with-streaming-locator"></a>将筛选器与流式处理定位符相关联

可以指定将适用于你的流式处理定位符的资产或帐户筛选器的列表。 [动态打包程序](dynamic-packaging-overview.md)适用此列表以及这些客户端在 URL 中指定的筛选器。 此组合生成[动态清单](filters-dynamic-manifest-overview.md)，后者基于在 URL 中的筛选器 + 流式处理定位符指定的筛选器。 我们建议你使用此功能，如果想要应用筛选器，但不是希望公开在 URL 中的筛选器名称。

## <a name="definition-example"></a>定义示例

```json
{
  "properties": {
    "presentationTimeRange": {
      "startTimestamp": 0,
      "endTimestamp": 170000000,
      "presentationWindowDuration": 9223372036854776000,
      "liveBackoffDuration": 0,
      "timescale": 10000000,
      "forceEndTimestamp": false
    },
    "firstQuality": {
      "bitrate": 128000
    },
    "tracks": [
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Audio"
          },
          {
            "property": "Language",
            "operation": "NotEqual",
            "value": "en"
          },
          {
            "property": "FourCC",
            "operation": "NotEqual",
            "value": "EC-3"
          }
        ]
      },
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Video"
          },
          {
            "property": "Bitrate",
            "operation": "Equal",
            "value": "3000000-5000000"
          }
        ]
      }
    ]
  }
}
```

## <a name="next-steps"></a>后续步骤

以下文章介绍了如何以编程方式创建筛选器。  

- [使用 REST API 创建筛选器](filters-dynamic-manifest-rest-howto.md)
- [使用 .NET 创建筛选器](filters-dynamic-manifest-dotnet-howto.md)
- [使用 CLI 创建筛选器](filters-dynamic-manifest-cli-howto.md)

