---
title: 筛选器和 Azure 媒体服务动态清单 | Microsoft Docs
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
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 21fb2b84fd58fb7cca7551ee1cef0c79179cfa40
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467131"
---
# <a name="dynamic-manifests"></a>动态清单

媒体服务根据预定义的筛选器提供**动态清单**。 在定义筛选器（请参阅[定义筛选器](filters-concept.md)）后，客户端可以使用筛选器来流式传输视频的特定再现内容或子剪辑。 客户端会在流 URL 中指定筛选器。 筛选器可以应用于自适应比特率流式处理协议：Apple HTTP Live Streaming (HLS)、MPEG-DASH 和平滑流式处理。 

下表显示了一些包含筛选器的 URL 示例：

|Protocol|示例|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|平滑流式处理|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

> [!NOTE]
> 动态清单不会更改资产和该资产的默认清单。 客户端可以选择请求包含或不包含筛选器的流。 
> 

本主题介绍与**动态清单**相关的概念，并提供可能需要使用此功能的示例方案。

## <a name="manifests-overview"></a>清单概述

媒体服务支持 HLS、 MPEG DASH、 平滑流式处理协议。 作为的一部分[动态打包](dynamic-packaging-overview.md)，流式处理客户端清单 （HLS 主播放列表、 短划线媒体演示描述 (MPD)，和平滑流式处理） 动态生成基于在 URL 中的格式选择器。 请参阅中的交付协议[本节](dynamic-packaging-overview.md#delivery-protocols)。 

### <a name="get-and-examine-manifest-files"></a>获取并检查清单文件

指定筛选器轨迹属性条件的列表，应该根据该列表将流（直播或点播视频）的轨迹包含到动态创建的清单中。 若要获取并检查轨迹的属性，必须先加载平滑流式处理清单。

[使用 .NET 上传、编码和流式传输文件](stream-files-tutorial-with-api.md)教程介绍了如何使用 .NET 创建流式处理 URL（参阅[生成 URL](stream-files-tutorial-with-api.md#get-streaming-urls) 部分）。 如果运行应用，某个 URL 将指向平滑流式处理清单：`https://amsaccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-0000000000000/ignite.ism/manifest`。<br/>请复制该 URL 并将其粘贴到浏览器的地址栏中。 随后将下载文件。 在所选的文本编辑器中打开该文件。

有关 REST 示例，请参阅[使用 REST 上传、编码和流式传输文件](stream-files-tutorial-with-rest.md#list-paths-and-build-streaming-urls)。

### <a name="monitor-the-bitrate-of-a-video-stream"></a>监视视频流的比特率

可以使用 [Azure Media Player 演示页](https://aka.ms/amp)监视视频流的比特率。 演示页在“诊断”选项卡中显示诊断信息：

![Azure Media Player 诊断][amp_diagnostics]

## <a name="rendition-filtering"></a>再现内容筛选

可选择将资产编码成多个编码配置文件（H.264 Baseline、H.264 High、AACL、AACH、Dolby Digital Plus），以及多个优质比特率。 不过，并非所有的客户端设备都支持资产的所有配置文件和比特率。 例如，早期的 Android 设备只支持 H.264 Baseline+AACL。 将较高的比特率发送到不能利用这些优势的设备会浪费带宽及设备计算资源。 此类设备必须解码所有给定信息，而目的仅仅是为了缩小信号以便能够显示。

有了动态清单，可以创建设备配置文件（例如移动配置文件、控制台、HD/SD 等），并包含你想要纳入配置文件中的轨迹与质量。

![再现内容筛选示例][renditions2]

以下示例使用编码器将夹层资产编码成七个 ISO MP4 视频再现内容（从 180p 到 1080p）。 编码的资产可以是[动态打包](dynamic-packaging-overview.md)成以下任一流协议：HLS、MPEG DASH 和平滑流式处理。  图表顶部显示了不包含筛选器的资产的 HLS 清单（包含全部七个再现内容）。  左下角显示名为“ott”的筛选器已应用到 HLS 清单。 “ott”筛选器指定要删除所有不低于 1 Mbps 的比特率，因此将最差的两个质量级别从响应中剥除。 在右下角显示已应用名为“mobile”的筛选器的 HLS 清单。 “mobile”筛选器指定删除分辨率大于 720p 的再现内容，因此会剥除两个 1080p 再现内容。

![再现内容筛选][renditions1]

## <a name="removing-language-tracks"></a>删除语言轨迹
你的资产可能包含多种音频语言，例如英语、西班牙语、法语等。通常，播放器 SDK 管理器会按默认选择音频轨迹，并根据用户的选择来选择可用音频轨迹。 开发此类播放器 SDK 相当有挑战性，因为各个设备特定的播放器框架之间需要不同的实现。 此外，播放器 API 在某些平台上受到限制，且不包含音频选择功能，因此用户无法选择或更改默认的音频轨迹。使用资产筛选器，可以通过创建只包含所需音频语言的筛选器来控制行为。

![语言轨迹筛选][language_filter]

## <a name="trimming-start-of-an-asset"></a>修剪资产开头
在大多数实时流事件中，操作员必须在发生实际事件之前进行某些测试。 例如，他们可以在事件开始之前包含如下静态内容：“节目即将开始”。 如果节目正在存档，则测试和静态数据也会一并存档并包含在演播中。 但是，此信息不应向客户端显示。 借助动态清单，可以创建开始时间筛选器，并从清单中删除不需要的数据。

![剪裁开始][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>基于实时存档创建子剪辑（视图）
许多实时事件的运行时间较长，并且实时存档可能包含多个事件。 实时事件结束后，广播者可能想要将实时存档分解成有序的节目启动和停止序列。 接下来，分别发布这些虚拟节目，但不后续处理实时存档，也不创建独立的资产（这会无法利用 CDN 中现有缓存片段的优势）。 此类虚拟节目的示例包括足球或篮球比赛中的上下半场（第几节）、棒球比赛中的局，或者任何体育项目的单项赛事。

借助动态清单，可以使用开始/结束时间创建筛选器，并基于实时存档创建虚拟视图。 

![子剪辑筛选器][subclip_filter]

筛选的资产：

![滑雪][skiing]

## <a name="adjusting-presentation-window-dvr"></a>调整演播窗口（DVR）
目前，Azure 媒体服务提供持续时间可设为 5 分钟到 25 小时的循环存档。 清单筛选可以基于存档创建循环 DVR 窗口存档，而不会删除媒体。 在许多情况下，广播者想要提供受限制的 DVR 窗口，此窗口可随着实时边缘移动，并同时保留更大的存档窗口。 广播者可能想要使用超出 DVR 窗口的数据来突出显示剪辑，或者想要为不同的设备提供不同的 DVR 窗口。 例如，大多数移动设备不处理大的 DVR 窗口（可以让移动设备有 2 分钟的 DVR 窗口，桌面客户端有 1 小时的 DVR 窗口）。

![DVR 窗口][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>调整实时补偿（实时位置）
清单筛选可用于删除实时节目的实时边缘几秒钟的时间。 筛选操作使广播者可以在观众收到流之前（倒退 30 秒）先观赏预览发布点的演播，并创建广告插入点。 广播者接着可将这些广告及时推送到其客户端框架，以便能够接收与处理信息，然后借机播放广告。

除了广告支持外，实时补偿设置可用于调整观众位置，使客户端偏移并到达实时边缘时，仍然可以从服务器获取片段，而不会收到 HTTP 404 或 412 错误。

![livebackoff_filter][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>将多个规则合并成单个筛选器
可以将多个筛选规则合并成单个筛选器。 例如，可以定义一个“范围规则”，以便将静态内容从实时存档中删除，并筛选出可用的比特率。 应用多个筛选规则时，最终结果会是所有规则的交集。

![多规则][multiple-rules]

## <a name="combining-multiple-filters-filter-composition"></a>组合多个筛选器（筛选器组合）

也可以在单个 URL 中组合多个筛选器。 

以下方案演示了可能需要组合多个筛选器的原因：

1. 需要筛选视频质量（目的是限制视频质量）以供 Android 或 iPAD 之类的移动设备使用。 要删除其质量不符合需要的视频，可创建一个适合设备配置文件的帐户筛选器。 帐户筛选器可用于同一媒体服务帐户下的所有资产，这些资产并没有更多的其他联系。 
2. 还可以修改资产的开始时间和结束时间。 为此，可以创建一个资产筛选器并设置开始/结束时间。 
3. 希望能够将这些筛选器组合起来（如果不组合的话，则需要将质量筛选添加到进行修改的筛选器上，这会导致筛选器的使用更加困难）。

为了组合筛选器，需要在清单/播放列表 URL 中设置筛选器名称，用分号对名称进行分隔。 假设你有一个名为 MyMobileDevice 的筛选器，用于筛选质量，另外还有一个名为 MyStartTime 的筛选器，用于设置具体的开始时间。 可以将它们组合成下面这样：

最多可以组合 3 个筛选器。 

有关详细信息，请参阅[此](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)博客。

## <a name="associate-filters-with-streaming-locator"></a>将筛选器与流式处理定位符相关联

可以指定将适用于你的流式处理定位符的资产或帐户筛选器的列表。 [动态打包程序](dynamic-packaging-overview.md)适用此列表以及这些客户端在 URL 中指定的筛选器。 此组合生成[动态清单](filters-dynamic-manifest-overview.md)，后者基于在 URL 中的筛选器 + 流式处理定位符指定的筛选器。 我们建议你使用此功能，如果想要应用筛选器，但不是希望公开在 URL 中的筛选器名称。

## <a name="considerations-and-limitations"></a>注意事项和限制

- 不应设置 VoD 筛选器的 **forceEndTimestamp**、**presentationWindowDuration** 和 **liveBackoffDuration** 值。 这些值仅用于直播筛选器方案。 
- 动态清单在 GOP 边界（主键帧）内运行，因此修剪后具有精确的 GOP。 
- 可对帐户和资产筛选器使用相同的筛选器名称。 资产筛选器的优先级更高，会替代帐户筛选器。
- 如果更新筛选器，它可能需要 2 分钟的时间为流式处理终结点来刷新规则。 如果内容是通过使用某些筛选器提供的（并在代理和 CDN 缓存中缓存），则更新这些筛选器会导致播放器失败。 建议在更新筛选器之后清除缓存。 如果此选项不可用，请考虑使用其他筛选器。
- 客户需要手动下载清单，并分析确切的 startTimestamp 和时间刻度。
    
    - 若要确定资产中轨迹的属性，请[获取并检查清单文件](#get-and-examine-manifest-files)。
    - 设置资产筛选器时间戳属性的公式： <br/>startTimestamp = &lt;清单中的开始时间&gt; +  &lt;预期筛选器开始时间（秒）&gt;*时间刻度


## <a name="next-steps"></a>后续步骤

以下文章介绍了如何以编程方式创建筛选器。  

- [使用 REST API 创建筛选器](filters-dynamic-manifest-rest-howto.md)
- [使用 .NET 创建筛选器](filters-dynamic-manifest-dotnet-howto.md)
- [使用 CLI 创建筛选器](filters-dynamic-manifest-cli-howto.md)

[renditions1]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/filters-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/filters-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/filters-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/filters-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/filters-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/filters-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/filters-dynamic-manifest-overview/media-services-skiing.png
[amp_diagnostics]: ./media/filters-dynamic-manifest-overview/amp_diagnostics.png
