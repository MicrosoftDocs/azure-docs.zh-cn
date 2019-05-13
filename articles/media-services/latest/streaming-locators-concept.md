---
title: Azure 媒体服务中的流式处理定位符 | Microsoft Docs
description: 本文介绍何为流式处理定位符以及 Azure 媒体服务如何使用这些定位符。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/08/2019
ms.author: juliako
ms.openlocfilehash: 24ee700e326ef61aa6a93aae725e85e7b4780edf
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465042"
---
# <a name="streaming-locators"></a>流式处理定位符

若要使输出资产中的视频可供客户端进行播放，必须创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)，然后生成流式处理 URL。 对于 .NET 示例，请参阅[获取流式处理定位符](stream-files-tutorial-with-api.md#get-a-streaming-locator)。

创建**流定位符**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，**流定位符**立即生效，并持续到被删除为止。 

创建流式处理定位符时，需要指定[资产](https://docs.microsoft.com/rest/api/media/assets)名称和[流式处理策略](https://docs.microsoft.com/rest/api/media/streamingpolicies)名称。 可以使用一个预定义流式处理策略或创建的自定义策略。 目前可用的预定义策略包括：“Predefined_DownloadOnly”、“Predefined_ClearStreamingOnly”、“Predefined_DownloadAndClearStreaming”、“Predefined_ClearKey”、“Predefined_MultiDrmCencStreaming”和“Predefined_MultiDrmStreaming”。 使用自定义的流式处理策略时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的选项和协议时重新将这些策略用于流式处理定位符。 

如果要对流指定加密选项，则创建[内容密钥策略](https://docs.microsoft.com/rest/api/media/contentkeypolicies)配置内容密钥如何通过媒体服务密钥传送组件传送到最终客户端。 将流式处理定位符与内容密钥策略和内容密钥相关联。 你可以让媒体服务自动生成密钥。 下面的 .NET 示例演示如何在媒体服务 v3 中使用令牌限制配置 AES 加密：[EncodeHTTPAndPublishAESEncrypted](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/tree/master/NETCore/EncodeHTTPAndPublishAESEncrypted)。 内容密钥策略可更新，如果需要进行密钥轮换，则可能需要更新该策略。 密钥传递缓存可能需要长达 15 分钟来进行更新并获取更新后的策略。 建议不要为每个流式处理定位符都创建新内容密钥策略。 应尝试在每次需要相同选项时重用现有策略。

> [!IMPORTANT]
> * 属于日期/时间类型的流式处理定位符的属性始终采用 UTC 格式。
> * 应为媒体服务帐户设计一组有限的策略，并在需要相同的选项时重新将这些策略用于流式处理定位符。 

## <a name="associate-filters-with-streaming-locators"></a>将筛选器与流式处理定位符相关联

可以指定一系列[资产或帐户筛选器](filters-concept.md)，这将会应用于你[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators/create#request-body)。 [动态打包程序](dynamic-packaging-overview.md)适用此列表以及这些客户端在 URL 中指定的筛选器。 此组合生成[动态清单](filters-dynamic-manifest-overview.md)，后者基于在 URL 中的筛选器 + 流式处理定位符指定的筛选器。 我们建议你使用此功能，如果想要应用筛选器，但不是希望公开在 URL 中的筛选器名称。

## <a name="filter-order-page-streaming-locator-entities"></a>筛选器、 顺序、 页流式处理定位符实体

请参阅[媒体服务实体的筛选、排序、分页](entities-overview.md)。

## <a name="next-steps"></a>后续步骤

* [教程：使用 .NET 上传、编码和流式传输视频](stream-files-tutorial-with-api.md)
* [使用 DRM 动态加密和许可证传送服务](protect-with-drm.md)
