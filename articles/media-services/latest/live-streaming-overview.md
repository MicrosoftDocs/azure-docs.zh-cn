---
title: 概述使用 Azure 媒体服务 v3 进行流式传输实时 |Microsoft Docs
description: 本文概述如何使用 Azure 媒体服务 v3 实时传送视频流。
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
ms.date: 04/03/2019
ms.author: juliako
ms.openlocfilehash: 3cd22b89852706ed129373fd24ca9a9d9a27d25b
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153379"
---
# <a name="live-streaming-with-azure-media-services-v3"></a>使用 Azure 媒体服务 v3 实时传送视频流

使用 Azure 媒体服务可将实时事件传送到 Azure 云中的客户。 若要使用媒体服务流式传输实时事件，需要以下组件：  

- 一个相机，用于捕获实时事件。<br/>有关设置建议，请查看[简单且可移植的事件视频设备设置]( https://link.medium.com/KNTtiN6IeT)。

    如果您没有对相机的访问，这样的工具[Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm)可以使用从视频文件生成的实时源。
- 一个实时视频编码器，用于将相机（或其他设备，例如便携式计算机）的信号转换为可发送到媒体服务的贡献源。 贡献源可包括与广告相关的信号，例如 SCTE-35 标记。<br/>有关推荐的实时传送视频流编码器的列表，请参阅[实时传送视频流编码器](recommended-on-premises-live-encoders.md)。 另外，请查看以下博客：[采用 OBS 的实时传送视频流生产](https://link.medium.com/ttuwHpaJeT)。
- 媒体服务中的组件，用于引入、预览、打包、记录、加密实时事件并将其广播给客户，或者广播给 CDN 进行进一步分发。

本文提供了概述和指南的实时传送视频流媒体服务和其他相关文章的链接。

> [!NOTE]
> 目前，无法使用 Azure 门户来管理 v3 资源。 请使用 [REST API](https://aka.ms/ams-v3-rest-ref)、[CLI](https://aka.ms/ams-v3-cli-ref) 或受支持的 [SDK](developers-guide.md) 之一。

## <a name="dynamic-packaging"></a>动态打包

使用媒体服务时，可以充分利用[动态打包](dynamic-packaging-overview.md)，可用于预览和广播实时流中的[MPEG DASH、 HLS 和平滑流式处理格式](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)从贡献源正在向服务发送的。 观看者可以使用任何与 HLS、DASH 或平滑流式处理兼容的播放器播放实时流。 可以使用 Web 应用程序或移动应用程序中的 [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) 传送采用上述任何协议的流。

## <a name="dynamic-encryption"></a>动态加密

动态加密，可动态加密使用 AES-128 或三个主要数字版权管理 (DRM) 系统的任何实时或按需内容：内容。 媒体服务还提供了用于向已授权客户端传送 AES 密钥和 DRM（PlayReady、Widevine 和 FairPlay）许可证的服务。 有关详细信息，请参阅[动态加密](content-protection-overview.md)。

## <a name="dynamic-manifest"></a>动态清单

使用动态筛选来控制跟踪、 格式、 比特率和演示文稿时间窗口，发送到参与方的数目。 有关详细信息，请参阅[筛选器和动态清单](filters-dynamic-manifest-overview.md)。

## <a name="live-event-types"></a>实时事件类型

实时事件可以是两种类型之一： 直通和实时编码。 有关实时传送视频流媒体服务 v3 中的详细信息，请参阅[实时事件和实时输出](live-events-outputs-concept.md)。

### <a name="pass-through"></a>直通

![直通](./media/live-streaming/pass-through.svg)

使用直通**实时事件**，可以依赖本地实时编码器生成多比特率视频流，并将其作为贡献源发送到实时事件（使用 RTMP 或分段 MP4 协议）。 然后，实时事件会接受无需进一步处理的传入视频流。 此类传递实时事件适用于长时间运行的实时事件或 24x365 线性实时传送视频流。 

### <a name="live-encoding"></a>实时编码  

![实时编码](./media/live-streaming/live-encoding.svg)

将实时编码与媒体服务配合使用时，需配置本地实时编码器，以便将单比特率视频作为贡献源发送到实时事件（使用 RTMP 或分段 MP4 协议）。 实时事件会将该传入的单比特率流编码为[多比特率视频流](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)，使其可通过 MPEG-DASH、HLS 和平滑流式处理等协议传送到播放设备。 

## <a name="live-streaming-workflow"></a>实时传送视频流工作流

若要了解媒体服务 v3 中的实时流式处理工作流，你必须首先回顾一下，并了解以下概念： 

- [流式处理终结点](streaming-endpoint-concept.md)
- [实时事件和实时输出](live-events-outputs-concept.md)
- [流式处理定位符](streaming-locators-concept.md)

### <a name="general-steps"></a>常规步骤

1. 在媒体服务帐户，请确保**流式处理终结点**(Origin) 正在运行。 
2. 创建[实时事件](live-events-outputs-concept.md)。 <br/>创建事件时，可以将其启动方式指定为自动启动。 或者，可以在准备好开始流式传输后，启动事件。<br/> 如果将 autostart 设置为 true，则实时事件会在创建后立即启动。 只要实时事件开始运行，就会开始计费。 必须显式对实时事件资源调用停止操作才能停止进一步计费。 有关详细信息，请参阅[实时事件状态和计费](live-event-states-billing.md)。
3. 获取的引入 URL，并配置本地编码器使用的 URL 发送贡献源。<br/>请参阅[推荐的实时编码器](recommended-on-premises-live-encoders.md)。
4. 获取预览 URL 并使用它验证来自编码器的输入是否实际接收。
5. 创建新的**资产**对象。
6. 创建**实时输出**并使用创建的资产名称。<br/>**实时输出**会将流存档到**资产**中。
7. 使用内置的**流式处理策略**类型创建**流式处理定位符**。<br/>如果想要加密内容，请查看[内容保护概述](content-protection-overview.md)。
8. 列出流式处理定位器的路径，以取回要使用的 URL（这些是确定性的）。
9. 获取的主机名**流式处理终结点**想要从 （源）。
10. 将步骤 8 中的 URL 与步骤 9 中的主机名合并，获取完整的 URL。
11. 如果希望停止查看**实时事件**，则需要停止流式处理事件并删除**流式处理定位符**。

## <a name="other-important-articles"></a>其他重要文章

- [推荐的实时编码器](recommended-on-premises-live-encoders.md)
- [使用云 DVR](live-event-cloud-dvr.md)
- [实时事件类型功能比较](live-event-types-comparison.md)
- [状态和计费](live-event-states-billing.md)
- [延迟](live-event-latency.md)

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

* [实时传送视频流教程](stream-live-tutorial-with-api.md)
* [有关从媒体服务 v2 迁移到 v3 的指导](migrate-from-v2-to-v3.md)
