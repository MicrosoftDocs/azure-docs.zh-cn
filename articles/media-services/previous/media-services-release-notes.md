---
title: 媒体服务发行说明 | Microsoft Docs
description: 媒体服务发行说明
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: media
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 427ba2b386810749810397afed8ef3f62dcf9217
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506624"
---
# <a name="azure-media-services-release-notes"></a>Azure 媒体服务发行说明

这些 Azure 媒体服务发行说明汇总了与以前版本相比的变更之处和已知的问题。

> [!NOTE]
> 不会向媒体服务 v2 添加任何新特性或新功能。 <br/>查看最新版本：[媒体服务 v3](https://docs.microsoft.com/azure/media-services/latest/)。 此外，请参阅[从 v2 到 v3 迁移指南](../latest/migrate-from-v2-to-v3.md)

我们希望能够倾听客户的心声，以便努力解决对客户造成影响的问题。 要报告问题或提出问题，请将在 [Azure 媒体服务 MSDN 论坛]提交问题。 

## <a name="a-idissuescurrently-known-issues"></a><a id="issues"/>当前已知的问题
### <a name="a-idgeneralissuesmedia-services-general-issues"></a><a id="general_issues"/>媒体服务一般问题

| 问题 | 描述 |
| --- | --- |
| REST API 中未提供几种常见的 HTTP 标头。 |如果使用 REST API 来开发媒体服务应用程序，将发现一些常见的 HTTP 标头字段（包括 CLIENT-REQUEST-ID、REQUEST-ID 和 RETURN-CLIENT-REQUEST-ID）不受支持。 未来的更新将增加这些标头。 |
| 不允许使用百分号编码。 |为流内容构建 URL 时，媒体服务使用 IAssetFile.Name 属性的值（例如，`http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters`）。 出于这个原因，不允许使用百分号编码。 Name 属性的值不能含有任何以下[百分号编码保留字符](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!* '();:@&=+$,/?%#[]"。 此外，文件扩展名中只能含有一个“.”。 |
| Azure 存储 SDK 版本 3.x 中的 ListBlobs 方法会失败。 |媒体服务基于 [2012-02-12](https://docs.microsoft.com/rest/api/storageservices/Version-2012-02-12) 版本生成 SAS URL。 如果希望使用存储 SDK 来列出 BLOB 容器中的 BLOB，请使用存储 SDK 版本 2.x 中的 [CloudBlobContainer.ListBlobs](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs) 方法。 |
| 媒体服务限制机制会限制那些发出过多服务请求的应用程序的资源使用情况。 该服务可能返回“服务不可用” 503 HTTP 状态代码。 |有关详细信息，请参阅[媒体服务错误代码](media-services-encoding-error-codes.md)中的 503 HTTP 状态代码说明。 |
| 查询实体时，一次返回的实体数限制为 1,000 个，因为公共 REST 第 2 版将查询结果数限制为 1,000 个。 |使用[此 .NET 示例](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities)和[此 REST API 示例](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities)中所述的 Skip 和 Take (.NET)/ top (REST)。 |
| 某些客户端可能会在平滑流式处理清单中碰到重复标记问题。 |有关详细信息，请参阅[此部分](media-services-deliver-content-overview.md#known-issues)。 |
| 媒体服务 .NET SDK 对象无法序列化，因此无法与 Azure Redis 缓存配合使用。 |如果尝试对 SDK AssetCollection 对象进行序列化以将其添加到 Azure Redis 缓存，则会引发异常。 |

## <a name="a-idrestversionhistoryrest-api-version-history"></a><a id="rest_version_history"/>REST API 版本历史记录
有关媒体服务 REST API 版本历史记录的信息，请参阅 [Azure 媒体服务 REST API 参考]。

## <a name="december-2018"></a>2018 年 12 月

很快将停用 Azure 媒体服务的媒体 Hyperlapse 预览功能。 从 2018 年 12 月 19 日起，媒体服务不再对 Media Hyperlapse 进行更改或改进。 在 2019 年 3 月 29 日，它将停用并不再可用。

## <a name="october-2018"></a>2018 年 10 月

### <a name="cmaf-support"></a>CMAF 支持

CMAF 和“cbcs”加密支持 Apple HLS (iOS 11+) 以及支持 CMAF 的 MPEG-DASH 播放器。

### <a name="web-vtt-thumbnail-sprites"></a>Web VTT 缩略图子画面

可以使用媒体服务生成使用 v2 API 的 Web VTT 缩略图子画面。 有关详细信息，请参阅[生成缩略图子画面](generate-thumbnail-sprite.md)。

## <a name="july-2018"></a>2018 年 7 月

使用最新的服务版本，当作业失败时，服务返回的错误消息会有一些细微的格式更改，这些更改与如何分解为两行或更多行相关。

## <a name="may-2018"></a>2018 年 5 月 

自 2018 年 5 月 12 日起，实时频道将不再支持 RTP/MPEG-2 传输流引入协议。 请从 RTP/MPEG-2 迁移到 RTMP 或分段 MP4（平滑流式处理）引入协议。

## <a name="october-2017-release"></a>2017 年 10 月版本
> [!IMPORTANT] 
> 媒体服务支持 Azure 访问控制服务身份验证密钥。 2018 年 6 月 22 日起，不再可使用访问控制服务密钥通过代码向媒体服务后端进行身份验证。 必须根据[基于 Azure AD 的身份验证](media-services-use-aad-auth-to-access-ams-api.md)一文中所述，将代码更新为使用 Azure Active Directory (AAD)。 请注意 Azure 门户中有关此更改的警告。

### <a name="updates-for-october-2017"></a>2017 年 10 月更新
#### <a name="sdks"></a>SDK
* .NET SDK 已更新为支持 Azure AD 身份验证。 为鼓励用户尽快迁移到 Azure AD，Nuget.org 上的 .NET SDK 已不再支持访问控制服务身份验证。 
* JAVA SDK 已更新为支持 Azure AD 身份验证。 已向 Java SDK 添加 Azure AD 身份验证支持。 有关如何将 Java SDK 用于 Media Services 的信息，请参阅[适用于 Azure 媒体服务的 Java 客户端 SDK 入门](media-services-java-how-to-use.md)

#### <a name="file-based-encoding"></a>基于文件的编码
* 现可使用高级编码器，针对 H.265 高效率视频编码 (HEVC) 视频编解码器进行内容编码。 选择 H.265 而不是其他编解码器（例如 H.264）不会影响价格。 有关 HEVC 专利许可证的信息，请参阅[联机服务条款](https://azure.microsoft.com/support/legal/)。
* 如果源视频是使用 H.265(HEVC) 视频编解码器编码的（例如，使用 iOS11 或 GoPro Hero 6 捕获的视频），则现在可以使用高级编码器或标准编码器来编码这些视频。 有关专利许可证的信息，请参阅[联机服务条款](https://azure.microsoft.com/support/legal/)。
* 对于包含多语言音轨的内容，必须根据相应文件格式规范（例如 ISO MP4）正确标记语言值。 然后即可使用标准编码器编码内容以便进行流式处理。 生成的流式处理定位符将列出可用的音频语言。
* 标准编码器现在支持两个仅限音频的新系统预设：“AAC 音频”和“AAC 优质音频”。 这两个预设分别生成 128 Kbps 和 192 Kbps 比特率的立体声高级音频编码 (AAC) 输出。
* 高级编码器现支持以 QuickTime/MOV 文件格式作为输入。 视频编解码器必须是[这篇 GitHub 文章中列出的 Apple ProRes 类型](https://docs.microsoft.com/azure/media-services/media-services-media-encoder-standard-formats)之一。 音频必须是 AAC 或脉冲编码调制 (PCM) 格式。 高级编码器不支持使用 QuickTime/MOV 文件中包装的 DVC/DVCPro 视频等内容作为输入。 标准版编码器支持这些视频编解码器。
* 编码器中进行了以下 bug 修复：

    * 现可使用输入资产提交作业。 这些作业完成后，可修改资产（例如添加、删除或重命名资产中的文件），并提交其他作业。
    * 改进了标准编码器生成的 JPEG 缩略图的质量。
    * 在持续时间极短的视频中，标准编码器现可更好地处理输入元数据和缩略图生成。
    * 对标准编码器中使用的 H.264 解码器做了改进，消除了某些罕见颗粒。 

#### <a name="media-analytics"></a>媒体分析
Azure 媒体编修器正式发布：此媒体处理器通过模糊选定个体的脸部进行匿名化处理，特别适用于公共安全和新闻媒体。 

有关此新处理器的概述，请参阅[这篇博客文章](https://azure.microsoft.com/blog/azure-media-redactor/)。 有关文档和设置的信息，请参阅[使用 Azure 媒体分析来编修面部](media-services-face-redaction.md)。



## <a name="june-2017-release"></a>2017 年 6 月版本

媒体服务现在支持 [基于 Azure AD 的身份验证](media-services-use-aad-auth-to-access-ams-api.md)。

> [!IMPORTANT]
> 目前，媒体服务支持访问控制服务身份验证模型。 访问控制服务授权将于 2018 年 6 月 1 日弃用。 建议尽快迁移到 Azure AD 身份验证模型。

## <a name="march-2017-release"></a>2017 年 3 月版本

在创建编码任务时，现在可通过指定“自适应流式处理”预设字符串使用标准编码器[自动生成比特率阶梯](media-services-autogen-bitrate-ladder-with-mes.md)。 若要使用媒体服务对视频进行流式处理，请使用“自适应流式处理”预设。 若要为特定方案自定义编码预设，可从[这些](media-services-mes-presets-overview.md)预设开始。

现在可使用标准编码器或媒体编码器高级工作流来[创建生成 fMP4 区块的编码任务](media-services-generate-fmp4-chunks.md)。 

## <a name="february-2017-release"></a>2017 年 2 月版本

自 2017 年 4 月 1 日起，将自动删除帐户中超过 90 天的所有作业记录及其相关任务记录。 即使记录总数低于最大配额，也会进行删除。 若要存档作业/任务信息，可使用[使用媒体服务 .NET SDK 管理资产和相关实体](media-services-dotnet-manage-entities.md)中所述的代码。

## <a name="january-2017-release"></a>2017 年 1 月版本

在媒体服务中，流式处理终结点表示一个流服务，该服务可以直接将内容传递给客户端播放器应用程序，也可以传递给内容支付网络 (CDN) 以进一步分发。 媒体服务还提供无缝 Azure 内容分发网络集成。 StreamingEndpoint 服务的出站流可以是媒体服务帐户中的实时流、点播视频或渐进式下载的资产。 每个媒体服务帐户均包括一个默认的流式处理终结点。 可以在帐户下创建其他流式处理终结点。 

有两个版本的流式处理终结点：1.0 和 2.0。 2017 年 1 月 10 日起，任何新创建的媒体服务帐户都会包括 2.0 版默认流式处理终结点。 可添加到此帐户的其他流式处理终结点也是 2.0 版。 此更改不会影响现有帐户。 现有流式处理终结点为 1.0，可升级到 2.0 版。 此次更改带来了行为、计费和功能更改。 有关详细信息，请参阅[流式处理终结点概述](media-services-streaming-endpoints-overview.md)。

从 2.15 版本开始，向流式处理终结点实体的以下属性添加了媒体服务：

* CdnProvider 
* CdnProfile
* FreeTrialEndTime 
* StreamingEndpointVersion 

有关这些属性的信息，请参阅 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。 

## <a name="december-2016-release"></a>2016 年 12 月版本

 现可使用媒体服务访问其服务的遥测/指标数据。 可使用当前版本的媒体服务收集实时通道、流式处理终结点和存档实体的遥测数据。 有关详细信息，请参阅[媒体服务遥测](media-services-telemetry-overview.md)。

## <a name="a-idjulychanges16july-2016-release"></a><a id="july_changes16"/>2016 年 7 月版本
### <a name="updates-to-the-manifest-file-ism-generated-by-encoding-tasks"></a>编码任务所生成清单文件 (*.ISM) 的更新
将某个编码任务提交到 Media Encoder Standard 或 Media Encoder Premium 后，该编码任务会在输出资产中生成[流式处理清单文件](media-services-deliver-content-overview.md) (*.ism)。 最新的服务版本已更新此流式处理清单文件的语法。

> [!NOTE]
> 流式处理清单 (.ism) 文件的语法保留供内部使用。 这在将来的版本中可能会有所改变。 请勿修改或操作此文件的内容。
> 
> 

### <a name="a-new-client-manifest-ismc-file-is-generated-in-the-output-asset-when-an-encoding-task-outputs-one-or-more-mp4-files"></a>当某个编码任务输出一个或多个 MP4 文件时，会在输出资产中生成新的客户端清单 (*.ISMC) 文件
从最新服务版本开始，在完成编码任务以生成一个或多个 MP4 文件以后，输出资产中也会包含流式处理客户端清单 (*.ismc) 文件。 该 .ismc 文件有助于改进动态流式处理的性能。 

> [!NOTE]
> 客户端清单 (.ismc) 文件的语法保留供内部使用。 这在将来的版本中可能会有所改变。 请勿修改或操作此文件的内容。
> 
> 

有关详细信息，请参阅[此博客](https://blogs.msdn.microsoft.com/randomnumber/2016/07/08/encoder-changes-within-azure-media-services-now-create-ismc-file/)。

### <a name="known-issues"></a>已知问题
某些客户端可能会在平滑流式处理清单中碰到重复标记问题。 有关详细信息，请参阅[此部分](media-services-deliver-content-overview.md#known-issues)。

## <a id="apr_changes16"></a>2016 年 4 月版本
### <a name="media-analytics"></a>媒体分析
 媒体服务引入了媒体分析，以实现功能强大的视频智能。 有关详细信息，请参阅[媒体服务分析概述](media-services-analytics-overview.md)。

### <a name="apple-fairplay-preview"></a>Apple FairPlay（预览版）
现可使用 Azure 媒体服务，通过 Apple FairPlay 动态加密 HTTP Live Streaming (HLS) 内容。 还可使用媒体服务许可证传送服务，将 FairPlay 许可证传送到客户端。 有关详细信息，请参阅“使用 Azure 媒体服务流式传输受 Apple FairPlay 保护的 HLS 内容”。

## <a id="feb_changes16"></a>2016 年 2 月版本
适用于 .NET 的媒体服务 SDK 最新版本 (3.5.3) 包含 Google Widevine 相关的 bug 修复程序。 之前无法对 Widevine 加密的多个资产重复使用 AssetDeliveryPolicy。 为了修复此 bug，SDK 中添加了以下属性：WidevineBaseLicenseAcquisitionUrl。

    Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
        new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
    {
        {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl,"http://testurl"},

    };

## <a id="jan_changes_16"></a>2016 年 1 月版本
编码预留单位已重命名，以减少与编码器名称的混淆。

基本、标准和高级编码预留单位已分别重命名为 S1、S2 和 S3 预留单位。 使用基本编码预留单位的客户现可在 Azure 门户（和帐单）中看到 S1 标签。 使用标准和高级版的客户分别会看到 S2 和 S3 标签。 

## <a id="dec_changes_15"></a>2015 年 12 月版本

### <a name="media-encoder-deprecation-announcement"></a>媒体编码器弃用公告

 将在 Media Encoder Standard 发布后大约 12 个月内开始弃用媒体编码器。

### <a name="azure-sdk-for-php"></a>内置 Web 服务器
Azure SDK 团队已发布新版 [Azure SDK for PHP](https://github.com/Azure/azure-sdk-for-php) 包，其中包含媒体服务的更新与新功能。 具体而言，适用于 PHP 的媒体服务 SDK 现支持最新[内容保护](media-services-content-protection-overview.md)功能。 使用 AES 和 DRM（PlayReady 和 Widevine）对这些功能进行动态加密（可以使用也可不使用标记限制）。 还支持缩放[编码单位](media-services-dotnet-encoding-units.md)。

有关详细信息，请参阅：

* 以下[代码示例](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)可帮助你快速入门：
  * vodworkflow_aes.php：此 PHP 文件演示如何使用 AES-128 动态加密和密钥传送服务。 它基于[使用 AES-128 动态加密和密钥传送服务](media-services-protect-with-aes128.md)中所述的 .NET 示例。
  * vodworkflow_aes.php：此 PHP 文件演示如何使用 PlayReady 动态加密和许可证传送服务。 它基于[使用 PlayReady 和/或 Widevine 动态通用加密](media-services-protect-with-playready-widevine.md)中所述的 .NET 示例。
  * scale_encoding_units.php：此 PHP 文件演示如何缩放编码预留单位。

## <a id="nov_changes_15"></a>2015 年 11 月版本
 媒体服务现已在云中提供 Widevine 许可证传送服务。 有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/)。 另请参阅[此教程](media-services-protect-with-playready-widevine.md)和 [GitHub 存储库](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm)。 

媒体服务提供的 Widevine 许可证传送服务为预览版。 有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/)。

## <a id="oct_changes_15"></a>2015 年 10 月版本
媒体服务现在位于以下数据中心：巴西南部、印度西部、印度南部和印度中部。 现在可以使用 Azure 门户[创建媒体服务帐户](media-services-portal-create-account.md)，以及执行[媒体服务文档网页](https://azure.microsoft.com/documentation/services/media-services/)中所述的各项任务。 这些数据中心未启用实时编码。 此外，并非所有类型的编码预留单位都可用于这些数据中心。

* 巴西南部：                                        只可以使用标准和基本编码保留单位。
* 印度西部、印度南部和印度中部：           只可以使用基本编码保留单位。

## <a id="september_changes_15"></a>2015 年 9 月版本
媒体服务现在提供通过 Widevine 模块化 DRM 技术保护点播视频和实时流的功能。 可以通过以下交付服务合作伙伴来交付 Widevine 许可证：
* [Axinom](https://www.axinom.com/press/ibc-axinom-drm-6/) 
* [EZDRM](https://ezdrm.com/) 
* [castLabs](https://castlabs.com/company/partners/azure/) 

有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/)。
  
可以通过[媒体服务 .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/)（从版本 3.5.1 开始）或 REST API 来配置 AssetDeliveryConfiguration 以使用 Widevine。 
* 媒体服务增加了对 Apple ProRes 视频的支持。 你现在可以上传使用 Apple ProRes 或其他编解码器的 QuickTime 源视频文件。 有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/announcing-support-for-apple-prores-videos-in-azure-media-services/)。
* 现可使用 Media Encoder Standard 来执行子剪辑和实时存档提取操作。 有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/)。
* 在筛选方面做了以下更新： 
  
  * 现可使用通过“仅音频”筛选器使用 Apple HLS 格式。 通过此更新，可在 URL 中指定 (audio-only=false) 来删除仅音频曲目。
  * 为资产定义筛选器时，现在可以将多个（最多 3 个）筛选器组合到一个 URL 中。
    
    有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。
* 媒体服务现在支持 HLS 第 4 版格式的 I-frame。 I-frame 支持优化快进和倒带操作。 默认情况下，所有 HLS 第 4 版输出包括 I-frame 播放列表 (EXT-X-I-FRAME-STREAM-INF)。
有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。

## <a id="august_changes_15"></a>2015 年 8 月版本
* 现已推出用于 Java 0.8.0 版的媒体服务 SDK 和新示例。 有关详细信息，请参阅：
    
* 已更新 Azure Media Player，现支持多音频流。 有关详细信息，请参阅[此博客文章](https://azure.microsoft.com/blog/2015/08/13/azure-media-player-update-with-multi-audio-stream-support/)。

## <a id="july_changes_15"></a>2015 年 7 月版本
* 宣布了 Media Encoder Standard 公开上市。 有关详细信息，请参阅[此博客文章](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)。
  
    Media Encoder Standard 使用[本节](https://go.microsoft.com/fwlink/?LinkId=618336)所述的预设值。 使用 4k 编码预设时，请获取高级预留单位类型。 有关详细信息，请参阅[缩放编码](media-services-scale-media-processing-overview.md)。
* 媒体服务和 Media Player 中采用了直播实时字幕。 有关详细信息，请参阅[此博客文章](https://azure.microsoft.com/blog/2015/07/08/live-real-time-captions-with-azure-media-services-and-player/)。

### <a name="media-services-net-sdk-updates"></a>媒体服务 .NET SDK 更新
媒体服务 .NET SDK 当前版本为 3.4.0.0。 进行了以下更新： 

* 实现了对实时存档的支持。 不能下载包含实时存档的资产。
* 实现了对动态筛选器的支持。
* 实现了这样的功能，让用户可在删除资产时保留存储容器。
* 进行了与频道中的重试策略相关的 Bug 修复。
* 启用了 Media Encoder Premium Workflow。

## <a id="june_changes_15"></a>2015 年 6 月版本
### <a name="media-services-net-sdk-updates"></a>媒体服务 .NET SDK 更新
媒体服务 .NET SDK 当前版本为 3.3.0.0。 进行了以下更新： 

* 添加了对 OpenId Connect 发现规范的支持。
* 添加了对标识提供者端处理密钥变换的支持。

如果使用的标识提供程序会公开 OpenID Connect 发现文档（就像 Azure AD、Google、Salesforce 那样），可指示 Azure 媒体服务获取签名密钥，以根据 OpenID Connect 发现规范验证 JSON Web 令牌 (JWT)。 

有关详细信息，请参阅[使用 OpenID Connect 发现规范中的 Json Web 密钥在媒体服务中进行 JWT 身份验证](http://gtrifonov.com/2015/06/07/using-json-web-keys-from-openid-connect-discovery-spec-to-work-with-jwt-token-authentication-in-azure-media-services/)。

## <a id="may_changes_15"></a>2015 年 5 月版本
宣布推出了以下新功能：

* [使用媒体服务进行实时编码的预览](media-services-manage-live-encoder-enabled-channels.md)
* [动态清单](media-services-dynamic-manifest-overview.md)

## <a id="april_changes_15"></a>2015 年 4 月版本
### <a name="general-media-services-updates"></a>媒体服务一般更新
* 公布了 [Media Player](https://azure.microsoft.com/blog/2015/04/15/announcing-azure-media-player/)。
* 从媒体服务 REST 2.10 开始，使用主要和辅助引入 URL 创建了配置为引入实时消息传输协议 (RTMP) 协议的通道。 有关详细信息，请参阅[频道引入配置](media-services-live-streaming-with-onprem-encoders.md#channel_input)。
* 更新了 Azure Media Indexer。
* 添加了西班牙语支持。
* 添加了 XML 格式的新配置。

有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/)。

### <a name="media-services-net-sdk-updates"></a>媒体服务 .NET SDK 更新
媒体服务 .NET SDK 当前版本为 3.2.0.0。 进行了以下更新：

* 重大更改：TokenRestrictionTemplate.Issuer 和 TokenRestrictionTemplate.Audience 更改为了字符串类型。
* 进行了与创建自定义重试策略相关的更新。
* 进行了与上传和下载文件相关的 Bug 修复。
* MediaServicesCredentials 类现在接受向主要和辅助访问控制终结点进行身份验证。

## <a id="march_changes_15"></a>2015 年 3 月版本
### <a name="general-media-services-updates"></a>媒体服务一般更新
* 媒体服务现提供内容分发网络集成。 为了支持集成，将 CdnEnabled 属性添加到了 StreamingEndpoint。 CdnEnabled 可用于 2.9 及以上版本的 REST API。 有关详细信息，请参阅 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。 CdnEnabled 可用于 3.1.0.2 及以上版本的 .NET SDK。 有关详细信息，请参阅 [StreamingEndpoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mediaservices.client.istreamingendpoint\(v=azure.10\).aspx)。
* 公布了 Media Encoder Premium Workflow。 有关详细信息，请参阅[在 Azure 媒体服务中引入高级编码](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/)。

## <a id="february_changes_15"></a>2015 年 2 月版本
### <a name="general-media-services-updates"></a>媒体服务一般更新
媒体服务 REST API 当前版本为 2.9。 自此版本起，可以通过流式处理终结点启用 Azure 内容分发网络集成。 有关详细信息，请参阅 [StreamingEndpoint](https://msdn.microsoft.com/library/dn783468.aspx)。

## <a id="january_changes_15"></a>2015 年 1 月版本
### <a name="general-media-services-updates"></a>媒体服务一般更新
公布了采用动态加密的内容保护公开上市。 有关详细信息，请参阅[媒体服务采用公开上市的 DRM 技术增强流式处理的安全性](https://azure.microsoft.com/blog/2015/01/29/azure-media-services-enhances-streaming-security-with-general-availability-of-drm-technology/)。

### <a name="media-services-net-sdk-updates"></a>媒体服务 .NET SDK 更新
媒体服务 .NET SDK 当前版本为 3.1.0.1。

此版本将默认的 Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization.TokenRestrictionTemplate 构造函数标记为已过时。 新的构造函数将 TokenType 作为参数。

    TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);


## <a id="december_changes_14"></a>2014 年 12 月版本
### <a name="general-media-services-updates"></a>媒体服务一般更新
* 向 Media Indexer 添加了一些更新和新功能。 有关详细信息，请参阅 [Azure Media Indexer 1.1.6.7 版发行说明](https://azure.microsoft.com/blog/2014/12/03/azure-media-indexer-version-1-1-6-7-release-notes/)。
* 添加了新的 REST API，可用于更新编码预留单位。 有关详细信息，请参阅 [REST 中的 EncodingReservedUnitType](https://docs.microsoft.com/rest/api/media/operations/encodingreservedunittype)。
* 添加了对密钥传送服务的 CORS 支持。
* 改进了查询授权策略选项的性能。
* 在中国数据中心，[密钥传递 URL](https://docs.microsoft.com/rest/api/media/operations/contentkey#get_delivery_service_url) 现在是每个客户一份（同其他数据中心一样）。
* 增加了 HLS 自动目标持续时间。 当执行实时流式传输时，HLS 始终是动态打包的。 默认情况下，媒体服务将根据从关键帧间隔 (KeyFrameInterval) 自动计算 HLS 段打包率 (FragmentsPerSegment)。 此方法也称为从实时编码器接收的帧组 (GOP)。 有关详细信息，请参阅[使用媒体服务实时传送视频流](https://msdn.microsoft.com/library/azure/dn783466.aspx)。

### <a name="media-services-net-sdk-updates"></a>媒体服务 .NET SDK 更新
[媒体服务 .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/) 当前版本为 3.1.0.0。 进行了以下更新：

* 将 .Net SDK 依赖项升级到了 .NET 4.5 Framework。
* 添加了新的 API，可用于更新编码预留单位。 有关详细信息，请参阅[使用 .NET 更新预留单位类型和增加编码预留单位](media-services-dotnet-encoding-units.md)。
* 添加了对令牌身份验证的 JWT 支持。 有关详细信息，请参阅[媒体服务和动态加密中的 JWT 令牌身份验证](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。
* 增加了 PlayReady 许可证模板中 BeginDate 和 ExpirationDate 的相对偏移量。

## <a id="november_changes_14"></a>2014 年 11 月版本
* 现可使用媒体服务，通过 SSL 连接插入实时平滑流式处理 (FMP4) 内容。 要通过 SSL 进行摄取，请确保将摄取 URL 更新为 HTTPS。 媒体服务当前对自定义域不支持 SSL。 有关实时传送视频流的详细信息，请参阅[使用 Azure 媒体服务实时传送视频流](https://msdn.microsoft.com/library/azure/dn783466.aspx)。
* 当前，无法通过 SSL 连接摄取 RTMP 实时流。
* 仅当要从中传送内容的流式处理终结点是在 2014 年 9 月 10 日之后创建的情况下，才可通过 SSL 流式传输内容。 如果流式处理 URL 是基于 2014 年 9 月 10 日之后创建的流式处理终结点，则 URL 会包含“streaming.mediaservices.windows.net”（新格式）。 包含“origin.mediaservices.windows.net”（旧格式）的流式处理 URL 不支持 SSL。 如果 URL 采用旧格式，并且希望通过 SSL 流式传输内容，请[创建新的流式处理终结点](media-services-portal-manage-streaming-endpoints.md)。 若要通过 SSL 流式传输内容，请使用基于新流式处理终结点的 URL。

### <a id="oct_sdk"></a>媒体服务 .NET SDK
适用于 .NET 的媒体服务 SDK 扩展当前版本为 2.0.0.3。

适用于 .NET 的媒体服务 SDK 当前版本为 3.0.0.8。 进行了以下更新：

* 在重试策略类中实现了重构。
* 向 HTTP 请求标头添加了用户代理字符串。
* 添加了 NuGet 还原生成步骤。
* 修复了方案测试以使用存储库中的 x509 证书。
* 通道和流式处理结束更新时，添加了验证设置。

### <a name="new-github-repository-to-host-media-services-samples"></a>新增了承载媒体服务示例的 GitHub 存储库
[Azure 媒体服务示例 GitHub 存储库](https://github.com/Azure/Azure-Media-Services-Samples)中提供了示例。

## <a id="september_changes_14"></a>2014 年 9 月版本
媒体服务 REST 元数据当前版本为 2.7。 有关最新 REST 更新的详细信息，请参阅[媒体服务 REST API 参考](https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference)。

适用于 .NET 的媒体服务 SDK 当前版本为 3.0.0.7。

### <a id="sept_14_breaking_changes"></a>重大更改
* 已将 Origin 重命名为 [StreamingEndpoint]。
* 更改了使用 Azure 门户编码和发布 MP4 文件时的默认行为。

### <a id="sept_14_GA_changes"></a>公开上市版中的新增功能/方案
* 引入了 Media Indexer 媒体处理器。 有关详细信息，请参阅[使用 Media Indexer 索引媒体文件](https://msdn.microsoft.com/library/azure/dn783455.aspx)。
* 可使用 [StreamingEndpoint] 实体添加自定义域（主机）名。
  
    若要使用自定义域名作为媒体服务流式处理终结点名称，请向流式处理终结点添加自定义主机名。 使用媒体服务 REST API 或 .NET SDK 添加自定义主机名。
  
    请注意以下事项：
  
  * 必须具有该自定义域名的所有权。
  * 域名的所有权必须通过媒体服务验证。 若要验证域，请创建映射 MediaServicesAccountId 父域的 CName 来验证 DNS mediaservices-dns-zone。
  * 必须创建另一个 CName，以将自定义主机名（例如 sports.contoso.com）映射到媒体服务 StreamingEndpont 的主机名（例如 amstest.streaming.mediaservices.windows.net）。

    有关详细信息，请参阅 [StreamingEndpoint](https://msdn.microsoft.com/library/azure/dn783468.aspx) 一文中的 CustomHostNames 属性。

### <a id="sept_14_preview_changes"></a>公共预览版的新增功能/方案
* 实时流式处理预览版。 有关详细信息，请参阅[使用媒体服务实时传送视频流](https://msdn.microsoft.com/library/azure/dn783466.aspx)。
* 密钥传递服务。 有关详细信息，请参阅[使用 AES-128 动态加密和密钥传递服务](https://msdn.microsoft.com/library/azure/dn783457.aspx)。
* AES 动态加密。 有关详细信息，请参阅[使用 AES-128 动态加密和密钥传递服务](https://msdn.microsoft.com/library/azure/dn783457.aspx)。
* PlayReady 许可证传递服务。 
* PlayReady 动态加密。 
* 媒体服务 PlayReady 许可证模板。 有关详细信息，请参阅[媒体服务 PlayReady 许可证模板概述]。
* 流式处理存储加密资产。 有关详细信息，请参阅[流式处理存储加密内容](https://msdn.microsoft.com/library/azure/dn783451.aspx)。

## <a id="august_changes_14"></a>2014 年 8 月版本
对资产进行编码时，完成编码作业后会生成输出资产。 在此版本之前，媒体服务编码器会生成有关输出资产的元数据。 自此版本起，编码器还将生成有关输入资产的元数据。 有关详细信息，请参阅[输入元数据]和[输出元数据]。

## <a id="july_changes_14"></a>2014 年 7 月版本
修复了 Azure 媒体服务包装程序和加密程序中的以下 Bug：

* 将实时存档资产传输到 HLS 时，仅播放音频：此问题已修复，现在可以播放音频和视频。
* 当资产打包为 HLS 和 AES 128 位信封加密时，打包的流无法在 Android 设备上播放：此 bug 已修复，打包的流可在支持 HLS 的 Android 设备上播放。

## <a id="may_changes_14"></a>2014 年 5 月版本
### <a id="may_14_changes"></a>媒体服务一般更新
现可使用[动态打包]对 HLS 第 3 版文件进行流式处理。 要对 HLS 第 3 版文件进行流式处理，请将以下格式添加到原点定位符路径：*.ism/manifest(format=m3u8-aapl-v3)。 有关详细信息，请访问[此论坛](https://social.msdn.microsoft.com/Forums/en-US/13b8a776-9519-4145-b9ed-d2b632861fde/dynamic-packaging-to-hls-v3)。

动态打包现在还支持基于使用 PlayReady 静态加密的平滑流式处理内容传递使用 PlayReady 加密的 HLS（第 3 版和第 4 版）。 有关如何使用 PlayReady 加密平滑流式处理内容的信息，请参阅[使用 PlayReady 保护平滑流](https://msdn.microsoft.com/library/azure/dn189154.aspx)。

### <a name="may_14_donnet_changes"></a>媒体服务 .NET SDK 更新
媒体服务 .NET SDK 当前版本为 3.1.0.5。 进行了以下更新：

* 上传和下载媒体资产时，速度和复原能力得到提升。
* 进行了重试逻辑和暂时性异常处理方面的改进： 
  
  * 改进了暂时性错误检测和重试逻辑，以处理由查询、保存更改、上传或下载文件引起的异常。 
  * 收到 Web 异常时（例如在访问控制服务令牌请求过程中），灾难性错误现将更快失败。

有关详细信息，请参阅[适用于 .NET 的媒体服务 SDK 中的重试逻辑]。

## <a id="jan_feb_changes_14"></a>2014 年 1/2 月版本
### <a name="jan_fab_14_donnet_changes"></a>媒体服务 .NET SDK 3.0.0.1、3.0.0.2 和 3.0.0.3
3.0.0.1 和 3.0.0.2 中的更改包括：

* 修复了与具有 OrderBy 语句的 LINQ 查询的使用相关的问题。
* 将 [GitHub] 中的测试解决方案拆分为了基于单位的测试和基于方案的测试。

有关这些更改的详细信息，请参阅[媒体服务 .NET SDK 3.0.0.1 和 3.0.0.2 版本](http://gtrifonov.com/2014/02/07/windows-azure-media-services-net-sdk-3-0-0-2-release/index.html)。

版本 3.0.0.3 中进行了以下更改：

* Azure 存储依赖项已升级为使用版本 3.0.3.0。
* 修复了 3.0.*.* 的后向兼容性问题 版本的向后兼容性问题。

## <a id="december_changes_13"></a>2013 年 12 月版本
### <a name="dec_13_donnet_changes"></a>媒体服务 .NET SDK 3.0.0.0
> [!NOTE]
> 3.0.x.x 版本不后向兼容 2.4.x.x 版本。
> 
> 

媒体服务 SDK 当前的最新版本为 3.0.0.0。 可从 NuGet 下载最新程序包或从 [GitHub] 获取资料。

自媒体服务 SDK 3.0.0.0 版本起，可以重复使用 [Azure AD 访问控制服务](https://msdn.microsoft.com/library/hh147631.aspx)令牌。 有关详细信息，请参阅[使用适用于 .NET 的媒体服务 SDK 连接到媒体服务](https://msdn.microsoft.com/library/azure/jj129571.aspx)中的“重复使用访问控制服务令牌”一节。

### <a name="dec_13_donnet_ext_changes"></a>媒体服务 .NET SDK 扩展 2.0.0.0
 媒体服务.NET SDK 扩展是一组扩展方法和帮助器函数，可简化代码，并令使用媒体服务进行开发变得更加容易。 [媒体服务 .NET SDK 扩展](https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev)中提供了最新信息。

## <a id="november_changes_13"></a>2013 年 11 月版本
### <a name="nov_13_donnet_changes"></a>媒体服务 .NET SDK 更改
自此版本起，适用于 .NET 的媒体服务 SDK 将处理在调用媒体服务 REST API 层时可能发生的暂时性故障错误。

## <a id="august_changes_13"></a>2013 年 8 月版本
### <a name="aug_13_powershell_changes"></a>Azure SDK 工具中包含的媒体服务 PowerShell cmdlet
[Azure SDK 工具](https://github.com/Azure/azure-sdk-tools)中现在包含以下媒体服务 PowerShell cmdlet。

* Get-AzureMediaServices 

    例如： `Get-AzureMediaServicesAccount`
* New-AzureMediaServicesAccount 
  
    例如： `New-AzureMediaServicesAccount -Name “MediaAccountName” -Location “Region” -StorageAccountName “StorageAccountName”`
* New-AzureMediaServicesKey 
  
    例如： `New-AzureMediaServicesKey -Name “MediaAccountName” -KeyType Secondary -Force`
* Remove-AzureMediaServicesAccount 
  
    例如： `Remove-AzureMediaServicesAccount -Name “MediaAccountName” -Force`

## <a id="june_changes_13"></a>2013 年 6 月版本
### <a name="june_13_general_changes"></a>媒体服务更改
本节所述的以下变化是 2013 年 6 月媒体服务版本中包含的更新。

* 将多个存储帐户链接到一个媒体服务帐户的功能。 
    * StorageAccount
    * Asset.StorageAccountName 和 Asset.StorageAccount
* 更新 Job.Priority 的功能。 
* 与通知相关的实体和属性： 
    * JobNotificationSubscription
    * NotificationEndPoint
    * 作业
* Asset.Uri 
* Locator.Name 

### <a name="june_13_dotnet_changes"></a>媒体服务 .NET SDK 更改
2013 年 6 月媒体服务 SDK 版本中包含以下更改。 GitHub 上提供有最新的媒体服务 SDK。

* 自 2.3.0.0 版起，媒体服务 SDK 支持将多个存储帐户链接到一个媒体服务帐户。 以下 API 支持此功能：
  
    * IStorageAccount 类型
    * Microsoft.WindowsAzure.MediaServices.Client.CloudMediaContext.StorageAccounts 属性
    * StorageAccount 属性
    * StorageAccountName 属性
  
      有关详细信息，请参阅[跨多个存储帐户管理媒体服务资产](https://msdn.microsoft.com/library/azure/dn271889.aspx)。
* 与通知相关的 API。 自 2.2.0.0 版起，可侦听 Azure 队列存储通知。 有关详细信息，请参阅[处理媒体服务作业通知](https://msdn.microsoft.com/library/azure/dn261241.aspx)。
  
    * Microsoft.WindowsAzure.MediaServices.Client.IJob.JobNotificationSubscriptions 属性
    * Microsoft.WindowsAzure.MediaServices.Client.INotificationEndPoint 类型
    * Microsoft.WindowsAzure.MediaServices.Client.IJobNotificationSubscription 类型
    * Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointCollection 类型
    * Microsoft.WindowsAzure.MediaServices.Client.IJobNotificationSubscription 类型
* 存储客户端 SDK 2.0 中的依赖项 (Microsoft.WindowsAzure.StorageClient.dll)
* OData 5.5 中的依赖项 (Microsoft.Data.OData.dll)

## <a id="december_changes_12"></a>2012 年 12 月版本
### <a name="dec_12_dotnet_changes"></a>媒体服务 .NET SDK 更改
* IntelliSense：为许多类型添加了缺少的 IntelliSense 文档。
* Microsoft.Practices.TransientFaultHandling.Core：修复了 SDK 仍依赖于此程序集的旧版本的问题。 SDK 现在引用此程序集的 5.1.1209.1 版本。

修复了 2012 年 11 月版 SDK 中发现的问题：

* IAsset.Locators.Count：现在会在删除所有定位符后在新的 IAsset 接口上正确报告此计数。
* IAssetFile.ContentFileSize：现在会在通过 IAssetFile.Upload(filepath) 上传后正确设置此值。
* IAssetFile.ContentFileSize：现可在创建资产文件时设置此属性。 此属性以前是只读的。
* IAssetFile.Upload(filepath)：修复了将多个文件上传到资产时，此同步上传方法引发以下错误的问题。 错误为“服务器未能对请求进行身份验证。 请确保授权标头的值构成正确，且包括签名。”
* IAssetFile.UploadAsync：修复了同时上传文件限于五个的问题。
* IAssetFile.UploadProgressChanged：此事件现在由 SDK 提供。
* IAssetFile.DownloadAsync(string, BlobTransferClient, ILocator, CancellationToken)：现在提供了此方法重载。
* IAssetFile.DownloadAsync：修复了同时下载文件限于五个的问题。
* IAssetFile.Delete()：修复了如果没有为 IAssetFile 上传文件，调用 delete 时可能会引发异常的问题。
* Jobs：修复了使用作业模板将“MP4 平滑流任务”与“PlayReady 保护任务”链接在一起时未创建任何任务的问题。
* EncryptionUtils.GetCertificateFromStore()：由于未找到基于证书配置问题的证书，此方法不再引发 null 异常。

## <a id="november_changes_12"></a>2012 年 11 月版本
本部分所述的变化是 2012 年 11 月（2.0.0.0 版）SDK 中包含的更新。 这些更改可能要求对 2012 年 6 月预览版 SDK 的代码进行修改或重写。

* 资产
  
    * IAsset.Create(assetName) 是唯一的资产创建函数。 IAsset.Create 不再在方法调用中上传文件。 使用 IAssetFile 进行上传。
    * IAsset.Publish 方法和 AssetState.Publish 枚举值已从媒体服务 SDK 中删除。 必须重写依赖于此值的任何代码。
* FileInfo
  
    * 此类已由 IAssetFile 删除并取代。
  
* IAssetFile
  
    * IAssetFile 取代了 FileInfo 并具有不同的行为。 若要使用它，请先实例化 IAssetFile 对象，然后使用媒体服务 SDK 或存储 SDK 上传文件。 可以使用以下 IAssetFile.Upload 重载：
  
        * IAssetFile.Upload(filePath)：此同步方法将阻止线程，建议仅在上传单个文件时使用。
        * IAssetFile.UploadAsync(filePath, blobTransferClient, locator, cancellationToken)：此异步方法是首选的上传机制。 
    
            已知的 bug：如果使用取消令牌，将取消上传。 任务可以有多个取消状态。 必须正确捕获并处理异常。
* 定位符
  
    * 删除了原点特定的版本。 SAS 特定的 context.Locators.CreateSasLocator(asset, accessPolicy) 将标记为已弃用或在公开上市时删除。 请参阅“新增功能”下的“定位符”部分以了解更新行为。

## <a id="june_changes_12"></a>2012 年 6 月预览版
以下是 11 月版 SDK 中的新增功能：

* 删除实体
  
    * IAsset、IAssetFile、ILocator、IAccessPolicy、IContentKey 对象现在已从对象级别（即 IObject.Delete()）删除，而不要求在集合（即 cloudMediaContext.ObjCollection.Delete(objInstance)）中删除。
* 定位符
  
    * 现在只能使用 CreateLocator 方法创建定位符。 它们只能使用 LocatorType.SAS 或 LocatorType.OnDemandOrigin 枚举值作为希望创建的特定类型定位符的参数。
    * 定位符增加了新的属性，以便更轻松地为内容获取可用的 URL。 这种重新设计的定位符旨在为将来的第三方扩展提供更大的灵活性，并提高媒体客户端应用程序的易用性。
* 异步方法支持
  
    * 对所有方法增加了异步支持。

## <a name="provide-feedback"></a>提供反馈
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

<!-- Anchors. -->

<!-- Images. -->

<!--- URLs. --->
[Azure 媒体服务 MSDN 论坛]: https://social.msdn.microsoft.com/forums/azure/home?forum=MediaServices
[Azure 媒体服务 REST API 参考]: https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference
[Media Services pricing details]: https://azure.microsoft.com/pricing/details/media-services/
[输入元数据]: https://msdn.microsoft.com/library/azure/dn783120.aspx
[输出元数据]: https://msdn.microsoft.com/library/azure/dn783217.aspx
[Deliver content]: https://msdn.microsoft.com/library/azure/hh973618.aspx
[Index media files with the Azure Media Indexer]: https://msdn.microsoft.com/library/azure/dn783455.aspx
[StreamingEndpoint]: https://msdn.microsoft.com/library/azure/dn783468.aspx
[Work with Media Services live streaming]: https://msdn.microsoft.com/library/azure/dn783466.aspx
[Use AES-128 dynamic encryption and the key delivery service]: https://msdn.microsoft.com/library/azure/dn783457.aspx
[Use PlayReady dynamic encryption and the license delivery service]: https://msdn.microsoft.com/library/azure/dn783467.aspx
[Preview features]: https://azure.microsoft.com/services/preview/
[媒体服务 PlayReady 许可证模板概述]: https://msdn.microsoft.com/library/azure/dn783459.aspx
[Stream storage-encrypted content]: https://msdn.microsoft.com/library/azure/dn783451.aspx
[Azure portal]: https://portal.azure.com
[动态打包]: https://msdn.microsoft.com/library/azure/jj889436.aspx
[Nick Drouin's blog]: http://blog-ndrouin.azurewebsites.net/hls-v3-new-old-thing/
[Protect Smooth Streaming with PlayReady]: https://msdn.microsoft.com/library/azure/dn189154.aspx
[适用于 .NET 的媒体服务 SDK 中的重试逻辑]: https://msdn.microsoft.com/library/azure/dn745650.aspx
[Grass Valley announces EDIUS 7 streaming through the cloud]: https://www.streamingmedia.com/Producer/Articles/ReadArticle.aspx?ArticleID=96351&utm_source=dlvr.it&utm_medium=twitter
[Control Media Services Encoder output file names]: https://msdn.microsoft.com/library/azure/dn303341.aspx
[Create overlays]: https://msdn.microsoft.com/library/azure/dn640496.aspx
[Stitch video segments]: https://msdn.microsoft.com/library/azure/dn640504.aspx
[Azure Media Services .NET SDK 3.0.0.1 and 3.0.0.2 releases]: http://www.gtrifonov.com/2014/02/07/windows-azure-media-services-.net-sdk-3.0.0.2-release/
[Azure AD Access Control Service]: https://msdn.microsoft.com/library/hh147631.aspx
[Connect to Media Services with the Media Services SDK for .NET]: https://msdn.microsoft.com/library/azure/jj129571.aspx
[Media Services .NET SDK extensions]: https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev
[Azure SDK tools]: https://github.com/Azure/azure-sdk-tools
[GitHub]: https://github.com/Azure/azure-sdk-for-media-services
[Manage Media Services assets across multiple Storage accounts]: https://msdn.microsoft.com/library/azure/dn271889.aspx
[Handle Media Services job notifications]: https://msdn.microsoft.com/library/azure/dn261241.aspx

