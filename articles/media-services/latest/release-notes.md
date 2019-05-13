---
title: Azure 媒体服务 v3 发行说明 | Microsoft Docs
description: 为了让大家随时了解最新的开发成果，本文提供了 Azure 媒体服务 v3 的最新更新。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: na
ms.topic: article
ms.date: 05/01/2019
ms.author: juliako
ms.openlocfilehash: 8f50188480b963d13460c79175ea6972d3e68f6a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153062"
---
# <a name="azure-media-services-v3-release-notes"></a>Azure 媒体服务 v3 发行说明

为了让大家随时了解最新的开发成果，本文将提供以下方面的信息：

* 最新版本
* 已知问题
* Bug 修复
* 已弃用的功能

## <a name="known-issues"></a>已知问题

> [!NOTE]
> 目前，无法使用 Azure 门户来管理 v3 资源。 请使用 [REST API](https://aka.ms/ams-v3-rest-sdk)、CLI 或支持的 SDK 之一。

有关详细信息，请参阅[有关从媒体服务 v2 迁移到 v3 的指导](migrate-from-v2-to-v3.md#known-issues)。

## <a name="may-2019"></a>2019 年 5

### <a name="performance-improvements"></a>性能提升

* 已更新，支持以进行处理的最大文件大小。 查看，请[配额和限制](limits-quotas-constraints.md)。
* [编码的速度改进](media-reserved-units-cli-how-to.md#choosing-between-different-reserved-unit-types)。

## <a name="april-2019"></a>2019 年 4 月

### <a name="new-presets"></a>新预设

* [FaceDetectorPreset](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#facedetectorpreset)已添加到内置分析器预设。
* [ContentAwareEncodingExperimental](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#encodernamedpreset)已添加到内置编码器预设。 有关详细信息，请参阅[识别的内容的编码](cae-experimental.md)。 

## <a name="march-2019"></a>2019 年 3 月

动态打包现在支持 Dolby Atmos. 有关详细信息，请参阅[音频编解码器支持的动态打包](dynamic-packaging-overview.md#audio-codecs-supported-by-dynamic-packaging)。

现在可以指定将适用于你的流式处理定位符的资产或帐户筛选器的列表。 有关详细信息，请参阅[流式处理定位符相关联的筛选器](filters-concept.md#associate-filters-with-streaming-locator)。

## <a name="february-2019"></a>2019 年 2 月

媒体服务 v3 现在支持在 Azure 的国家/地区云。 目前，并非所有功能在所有云中都可用。 有关详细信息，请参阅[存在 Azure 媒体服务 v3 的云和区域](azure-clouds-regions.md)。

[Microsoft.Media.JobOutputProgress](media-services-event-schemas.md#monitoring-job-output-progress) 事件已添加到媒体服务的 Azure 事件网格架构中。

## <a name="january-2019"></a>2019 年 1 月

### <a name="media-encoder-standard-and-mpi-files"></a>Media Encoder Standard 和 MPI 文件 

使用 Media Encoder Standard 编码生成 MP4 文件时，新的 .mpi 文件会生成并添加到输出资产中。 此 MPI 文件旨在提高[动态打包](dynamic-packaging-overview.md)和流式处理方案的性能。

不应修改或删除该 MPI 文件，也不应在存在（或不存在）此类文件的情况下采用服务中的任何依赖项。

## <a name="december-2018"></a>2018 年 12 月

V3 API 的正式版中的更新包括：
       
* **PresentationTimeRange** 属性不再是 **AssetFilters** 和 **AccountFilters** 所必需的。 
* 已删除 **Jobs** 和 **Transforms** 的 $top 和 $skip 查询选项，并添加了 $orderby。 在添加新排序功能的过程中，我们发现之前意外地公开了 $top 和 $skip 选项，尽管它们并未实现。
* 重新启用了枚举可扩展性。 此功能在 SDK 的预览版中已启用，但在正式版中被意外禁用。
* 已重命名两个预定义的流式处理策略。 **SecureStreaming** 现在名为 **MultiDrmCencStreaming**。 **SecureStreamingWithFairPlay** 现在名为 **Predefined_MultiDrmStreaming**。

## <a name="november-2018"></a>2018 年 11 月

CLI 2.0 模块现在可用于 [Azure 媒体服务 v3 正式版](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest) - v 2.0.50。

### <a name="new-commands"></a>新命令

- [az ams account](https://docs.microsoft.com/cli/azure/ams/account?view=azure-cli-latest)
- [az ams account-filter](https://docs.microsoft.com/cli/azure/ams/account-filter?view=azure-cli-latest)
- [az ams asset](https://docs.microsoft.com/cli/azure/ams/asset?view=azure-cli-latest)
- [az ams asset-filter](https://docs.microsoft.com/cli/azure/ams/asset-filter?view=azure-cli-latest)
- [az ams content-key-policy](https://docs.microsoft.com/cli/azure/ams/content-key-policy?view=azure-cli-latest)
- [az ams job](https://docs.microsoft.com/cli/azure/ams/job?view=azure-cli-latest)
- [az ams live-event](https://docs.microsoft.com/cli/azure/ams/live-event?view=azure-cli-latest)
- [az ams live-output](https://docs.microsoft.com/cli/azure/ams/live-output?view=azure-cli-latest)
- [az ams streaming-endpoint](https://docs.microsoft.com/cli/azure/ams/streaming-endpoint?view=azure-cli-latest)
- [az ams streaming-locator](https://docs.microsoft.com/cli/azure/ams/streaming-locator?view=azure-cli-latest)
- [az ams account mru](https://docs.microsoft.com/cli/azure/ams/account/mru?view=azure-cli-latest) - 用于管理媒体保留单位。 有关详细信息，请参阅[缩放媒体保留单位](media-reserved-units-cli-how-to.md)。

### <a name="new-features-and-breaking-changes"></a>新功能和重大更改

#### <a name="asset-commands"></a>资产命令

- 添加了 ```--storage-account``` 和 ```--container``` 参数。
- 在 ```az ams asset get-sas-url``` 命令中添加了到期时间默认值（现在时间 + 23 小时）和权限默认值（读取）。

#### <a name="job-commands"></a>作业命令

- 添加了 ```--correlation-data``` 和 ```--label``` 参数
- ```--output-asset-names``` 已重命名为 ```--output-assets```。 现在，它接受 'assetName=label' 格式的资产列表（以空格分隔）。 没有标签的资产可以采用以下格式发送：'assetName='。

#### <a name="streaming-locator-commands"></a>创建流式处理定位符命令

- ```az ams streaming locator``` 基本命令已替换为 ```az ams streaming-locator```。
- 添加了 ```--streaming-locator-id``` 和 ```--alternative-media-id support``` 参数。
- 更新了 ```--content-keys argument``` 参数。
- ```--content-policy-name``` 已重命名为 ```--content-key-policy-name```。

#### <a name="streaming-policy-commands"></a>流式处理策略命令

- ```az ams streaming policy``` 基本命令已替换为 ```az ams streaming-policy```。
- 在 ```az ams streaming-policy create``` 中添加了加密参数支持。

#### <a name="transform-commands"></a>转换命令

- ```--preset-names``` 参数已替换为 ```--preset```。 现在只能一次设置 1 个输出/预设（若要添加更多，必须运行 ```az ams transform output add```）。 此外，还可以通过将路径传递到自定义 JSON 来设置自定义 StandardEncoderPreset。
- 可以通过传递要删除的输出索引来执行 ```az ams transform output remove```。
- 在 ```az ams transform create``` 和 ```az ams transform output add``` 命令中添加了 ```--relative-priority, --on-error, --audio-language and --insights-to-extract``` 参数。

## <a name="october-2018---ga"></a>October 2018 - GA

本部分介绍 Azure 媒体服务 (AMS) 的 10 月更新。

### <a name="rest-v3-ga-release"></a>REST v3 GA 发布

[REST v3 GA 发布](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01)包含更多用于实时、帐户/资产级别清单筛选器和 DRM 支持的 API。

#### <a name="azure-resource-management"></a>Azure 资源管理 

对 Azure 资源管理的支持实现了统一的管理和操作 API（现在所有功能已整合在一起）。

从此版本开始，可以使用资源管理器模板创建实时事件。

#### <a name="improvement-of-asset-operations"></a>改善资产操作 

引入了以下改进：

- 从 HTTP(s) URL 或 Azure Blob 存储 SAS URL 中引入。
- 为资产指定自己的容器名称。 
- 使用 Azure Functions 创建自定义工作流支持更轻松的输出。

#### <a name="new-transform-object"></a>新转换对象

新转换对象简化编码模型。 通过新对象，可以轻松创建和共享编码资源管理器模板和预设。 

#### <a name="azure-active-directory-authentication-and-rbac"></a>Azure Active Directory 身份验证和 RBAC

Azure AD 身份验证和基于角色的访问控制 (RBAC) 通过 Azure AD 中的角色或用户启用安全转换、实时事件、内容密钥策略或资产。

#### <a name="client-sdks"></a>客户端 SDK  

媒体服务 v3 支持的语言：NET Core、Java、Node.js、Ruby、Typescript、Python、Go。

#### <a name="live-encoding-updates"></a>实时编码更新

引入了以下实时编码更新：

- 针对实时传送视频的新的低延迟模式（10 秒端到端）。
- 改进的 RTMP 支持（提高了稳定性并提供了更多的源编码器支持）。
- RTMPS 安全引入。

    创建实时事件时，现在会得到 4 个引入 URL。 这 4 个引入 URL 几乎是相同的，具有相同的流式处理令牌 (AppId)，仅端口号部分不同。 其中两个 URL 是 RTMPS 的主要和备份 URL。 
- 24 小时转码支持。 
- 通过 SCTE35 改进了 RTMP 中的广告信号支持。

#### <a name="improved-event-grid-support"></a>改进了事件网格支持

可以看到以下事件网格支持改进：

- 可通过逻辑应用和 Azure Functions 更轻松地开发 Azure 事件网格集成。 
- 订阅事件的编码、直播频道等。

### <a name="cmaf-support"></a>CMAF 支持

CMAF 和“cbcs”加密支持 Apple HLS (iOS 11+) 以及支持 CMAF 的 MPEG-DASH 播放器。

### <a name="video-indexer"></a>视频索引器

视频索引器 GA 版本于 8 月份发布。 有关当前支持的功能的新信息，请参阅[什么是视频索引器](../../cognitive-services/video-indexer/video-indexer-overview.md?toc=/azure/media-services/video-indexer/toc.json&bc=/azure/media-services/video-indexer/breadcrumb/toc.json)。 

### <a name="plans-for-changes"></a>更改计划

#### <a name="azure-cli-20"></a>Azure CLI 2.0
 
包含所有功能（包括 Live、内容密钥策略、帐户/资产筛选器、流式处理策略）的操作的 Azure CLI 2.0 模块即将推出。 

### <a name="known-issues"></a>已知问题

只有使用资产或 AccountFilter 预览版 API 的客户才会受到以下问题的影响。

如果你使用媒体服务 v3 CLI 或 API 在 9 月 28 日到 10 月 12 日之间创建了资产或帐户筛选器，因版本冲突，需要删除所有资产和 AccountFilter，并重新创建它们。 

## <a name="may-2018---preview"></a>2018 年 5 月 - 预览版

### <a name="net-sdk"></a>.NET SDK

以下功能是.NET SDK 中：

* 转换和作业，用于对媒体内容来进行编码或分析。 有关示例，请参阅[流式传输文件](stream-files-tutorial-with-api.md)和[分析](analyze-videos-tutorial-with-api.md)。
* **流式处理定位符**，用于发布内容并将其流式传输到最终用户设备
* **流式处理策略**和**内容密钥策略**，用于在传送内容时配置密钥传递和内容保护 (DRM)。
* **实时事件**和**实时输出**，用于配置实时传送视频流内容的引入和归档。
* 资产，用于在 Azure 存储中存储和发布媒体内容。 
* **流式处理终结点**，用于配置和缩放实时和点播媒体内容的动态打包、加密和流式处理。

### <a name="known-issues"></a>已知问题

* 提交作业时，可以指定使用 HTTPS URL、SAS URL 或位于 Azure Blob 存储中的文件路径引入源视频。 目前，AMS v3 不支持基于 HTTPS URL 的块传输编码。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

[概述](media-services-overview.md)
