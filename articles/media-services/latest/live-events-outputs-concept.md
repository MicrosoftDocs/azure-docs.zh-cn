---
title: Azure 媒体服务中的实时传送视频流概念 - 实时事件和实时输出 | Microsoft Docs
description: 本文概述了 Azure 媒体服务 v3 中的实时传送视频流概念。
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
ms.date: 04/21/2019
ms.author: juliako
ms.openlocfilehash: 87620cb0c7aafb78b2c8fe610f42ad092313d305
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153482"
---
# <a name="live-events-and-live-outputs"></a>实时事件和实时输出

使用 Azure 媒体服务可将实时事件传送到 Azure 云中的客户。 若要配置媒体服务 v3 中的实时流式处理事件，您需要了解本文中讨论的概念。 <br/>在页面的右侧列出的部分列表。

## <a name="live-events"></a>实时事件

[实时事件](https://docs.microsoft.com/rest/api/media/liveevents)负责引入和处理实时视频源。 创建实时事件时，会创建一个输入终结点，可以使用它来从远程编码器发送实时信号。 远程实时编码器使用 [RTMP](https://www.adobe.com/devnet/rtmp.html) 或[平滑流式处理](https://msdn.microsoft.com/library/ff469518.aspx)（分段 MP4）协议将贡献源发送到该输入终结点。 对于平滑流式处理引入协议，支持的 URL 方案为 `http://` 或 `https://`。 对于 RTMP 引入协议，支持的 URL 方案为 `rtmp://` 或 `rtmps://`。 

## <a name="live-event-types"></a>实时事件类型

[实时事件](https://docs.microsoft.com/rest/api/media/liveevents)可以是下述两种类型之一：直通和实时编码。 

### <a name="pass-through"></a>直通

![直通](./media/live-streaming/pass-through.svg)

使用直通**实时事件**，可以依赖本地实时编码器生成多比特率视频流，并将其作为贡献源发送到实时事件（使用 RTMP 或分段 MP4 协议）。 然后，实时事件会接受无需进一步处理的传入视频流。 此类直通 LiveEvent 已针对长时间运行的实时事件或 24x365 线性实时传送视频流进行优化。 创建此类实时事件时，请指定 None (LiveEventEncodingType.None)。

发送的贡献源的最高分辨率可为 4K，帧速率可为 60 帧/秒，采用 H.264/AVC 或 H.265/HEVC 视频编解码器，以及 AAC（AAC-LC、HE-AACv1 或 HE-AACv2）音频编解码器。  有关详细信息，请参阅[实时事件类型比较](live-event-types-comparison.md)一文。

> [!NOTE]
> 当需要长时间处理多个事件，并且已经在本地编码器上进行了投入时，则可使用直通这种最经济的方法来实时传送视频流。 请参阅 [定价](https://azure.microsoft.com/pricing/details/media-services/) 详细信息。
> 

请参阅 [MediaV3LiveApp](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs#L126) 中的 .NET 代码示例。

### <a name="live-encoding"></a>实时编码  

![实时编码](./media/live-streaming/live-encoding.svg)

将实时编码与媒体服务配合使用时，需配置本地实时编码器，以便将单比特率视频作为贡献源发送到实时事件（使用 RTMP 或分段 MP4 协议）。 实时事件会将该传入的单比特率流编码为[多比特率视频流](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)，使其可通过 MPEG-DASH、HLS 和平滑流式处理等协议传送到播放设备。 创建此类实时事件时，请将编码类型指定为“标准”(LiveEventEncodingType.Standard)。

发送的贡献源的最高分辨率可为 1080p，帧速率可为 30 帧/秒，采用 H.264/AVC 视频编解码器，以及 AAC（AAC-LC、HE-AACv1 或 HE-AACv2）音频编解码器。 有关详细信息，请参阅[实时事件类型比较](live-event-types-comparison.md)一文。

使用实时编码（实时事件设置为**标准**）时，编码预设定义了如何将传入的流编码为多个比特率或图层。 有关信息，请参阅[系统预设](live-event-types-comparison.md#system-presets)。

> [!NOTE]
> 当前，实时事件的标准类型唯一允许的预设值为 *Default720p*。 如果你需要使用自定义的实时编码预设，请联系 amshelp@microsoft.com。 你应当指定所需的分辨率和比特率的表。 请确认只有一个 720p 的图层，最多有 6 个图层。

## <a name="live-event-creation-options"></a>实时事件创建选项

创建实时事件时，可以指定以下选项：

* 实时事件的流式处理协议（目前支持 RTMP 和平滑流式处理协议）。<br/>运行实时事件或其关联的实时输出时，无法更改协议选项。 如果需要其他协议，应当为每个流式处理协议创建单独的实时事件。  
* 对引入和预览的 IP 限制。 可定义允许向该实时事件引入视频的 IP 地址。 允许的 IP 地址可以指定为单个 IP 地址（例如“10.0.0.1”）、使用一个 IP 地址和 CIDR 子网掩码的 IP 范围（例如“10.0.0.1/22”）或使用一个 IP 地址和点分十进制子网掩码的 IP 范围（例如“10.0.0.1(255.255.252.0)”）。<br/>如果未指定 IP 地址并且没有规则定义，则不会允许任何 IP 地址。 若要允许任何 IP 地址，请创建一个规则并设置 0.0.0.0/0。<br/>IP 地址必须采用以下格式之一：具有 4 个数字、CIDR 地址范围的 IpV4 地址。
* 创建事件时，可以将其启动方式指定为自动启动。 <br/>如果将 autostart 设置为 true，则实时事件会在创建后启动。 只要实时事件开始运行，就会开始计费。 必须显式对实时事件资源调用停止操作才能停止进一步计费。 或者，可以在准备好开始流式传输后，启动事件。 

    有关详细信息，请参阅[实时事件状态和计费](live-event-states-billing.md)。

## <a name="live-event-ingest-urls"></a>实时事件引入 URL

创建实时事件后，可以获得要提供给实时本地编码器的引入 URL。 实时编码器使用这些 URL 来输入实时流。 有关详细信息，请参阅[建议的本地实时编码器](recommended-on-premises-live-encoders.md)。 

可以使用非虚 URL 或虚 URL。 

* 非虚 URL

    在 AMS v3 中，非虚 URL 是默认模式。 可以快速获取实时事件，但只有在实时事件启动后，才会知道引入 URL。 如果停止/启动实时事件，此 URL 会更改。 <br/>非虚 URL 适用于这样的情况：最终用户希望使用应用进行流式处理，而应用希望尽快获取实时事件，并且可以使用动态引入 URL。
* 虚 URL

    对于使用硬件广播编码器且不希望在启动实时事件时重新配置其编码器的大型媒体广播者来说，虚模式是首选。 他们需要不随时间而改变的预测性引入 URL。

> [!NOTE] 
> 若要让引入 URL 成为预测性 URL，需使用“虚”模式并传递你自己的访问令牌（避免在 URL 中使用随机令牌）。

### <a name="live-ingest-url-naming-rules"></a>实时引入 URL 命名规则

下面的随机字符串是一个 128 位的十六进制数字（由 32 个 0-9 a-f 字符组成）。<br/>
下面的访问令牌是需为固定 URL 指定的内容。 它也是 128 位的十六进制数字。

#### <a name="non-vanity-url"></a>非虚 URL

##### <a name="rtmp"></a>RTMP

`rtmp://<random 128bit hex string>.channel.media.azure.net:1935/<access token>`
`rtmp://<random 128bit hex string>.channel.media.azure.net:1936/<access token>`
`rtmps://<random 128bit hex string>.channel.media.azure.net:2935/<access token>`
`rtmps://<random 128bit hex string>.channel.media.azure.net:2936/<access token>`

##### <a name="smooth-streaming"></a>平滑流

`http://<random 128bit hex string>.channel.media.azure.net/<access token>/ingest.isml`
`https://<random 128bit hex string>.channel.media.azure.net/<access token>/ingest.isml`

#### <a name="vanity-url"></a>虚 URL

##### <a name="rtmp"></a>RTMP

`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1935/<access token>`
`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1936/<access token>`
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2935/<access token>`
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2936/<access token>`

##### <a name="smooth-streaming"></a>平滑流

`http://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<access token>/ingest.isml`
`https://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<access token>/ingest.isml`

## <a name="live-event-preview-url"></a>实时事件预览 URL

一旦**实时事件**开始接收贡献源，你就可以使用其预览终结点进行预览，并在进一步发布之前验证是否可以收到实时流。 确认预览流正常后，可以使用 LiveEvent 来确保可以通过一个或多个（预先创建的）**流式处理终结点**传送实时流。 为此，需针对**实时事件**创建新的[实时输出](https://docs.microsoft.com/rest/api/media/liveoutputs)。 

> [!IMPORTANT]
> 确保视频流向预览 URL，然后再继续操作！

## <a name="live-event-long-running-operations"></a>实时事件长时间运行的操作

有关详细信息，请参阅[长时间运行的操作](media-services-apis-overview.md#long-running-operations)

## <a name="live-outputs"></a>实时输出

将流传输到实时事件后，可以通过创建[资产](https://docs.microsoft.com/rest/api/media/assets)、[实时输出](https://docs.microsoft.com/rest/api/media/liveoutputs)和[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)来启动流式传输事件。 实时输出会存档流，并使观看者可通过[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints)使用该流。  

> [!NOTE]
> 实时输出在创建时启动，在删除后停止。 删除实时输出不会删除基础资产和该资产中的内容。 

**实时事件**与其**实时输出**之间的关系类似于传统的电视广播，其中的频道（**实时事件**）表示恒定的视频流，录制（**实时输出**）限定为特定的时间段（例如，下午 6:30 到 7:00 的晚间新闻）。 可以使用数字视频录像机 (DVR) 录制电视节目 - 实时事件中的等效功能是通过 **ArchiveWindowLength** 属性管理的。 它是一个 ISO-8601 时间跨度持续时间（例如 PTHH:MM:SS），指定 DVR 的容量，最小可设置为 3 分钟，最大为 25 小时。

**实时输出**对象类似于磁带录制机，它可以捕获实时流并将其记录到媒体服务帐户中的资产。 记录的内容将保存到你的帐户上附加的 Azure 存储帐户，并保存到资产资源定义的容器。 使用**实时输出**还可以控制传出实时流的某些属性，例如，存档记录中保存的流的量（如云 DVR 的容量），以及观看者是否可以开始观看实时流。 磁盘上的存档是一个环形存档“窗口”，仅保存**实时输出**的 **archiveWindowLength** 属性中指定的内容量。 超出此窗口的内容将自动从存储容器中丢弃，且不可恢复。 可以使用不同的存档长度和设置针对**实时事件**创建多个**实时输出**（最多三个）。  

如果已使用**流式处理定位符**发布了**实时输出**的**资产**，则**实时事件**（最长为 DVR 窗口长度）将继续可见，直到流式处理定位符过期或被删除（以先发生者为准）。

有关详细信息，请参阅[使用云 DVR](live-event-cloud-dvr.md)。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

[实时传送视频流教程](stream-live-tutorial-with-api.md)
