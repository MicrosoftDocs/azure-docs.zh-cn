---
title: Azure 媒体服务打包和源错误 |Microsoft Docs
description: 本主题描述了可能会收到来自 Azure 媒体服务流式处理终结点 (Orgin) 服务的错误。
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: juliako
ms.openlocfilehash: ebcda6026f79bc88df91471d8be88316ba57bfc6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411375"
---
# <a name="streaming-endpoint-origin-errors"></a>流式处理终结点 （来源） 错误 

本主题介绍可能会收到 Azure 媒体服务中的错误[流式处理终结点服务](streaming-endpoint-concept.md)。

## <a name="400-bad-request"></a>400 错误请求

请求包含无效的信息，并使用这些错误代码和由于以下原因之一而被拒绝：

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_BAD_URL_SYNTAX |0x80890201|URL 语法或出现格式错误。 例如，请求类型无效，无效的片段或无效的跟踪。 |
|MPE_ENC_ENCRYPTION_NOT_SPECIFIED_IN_URL |0x8088024C|请求 URL 中有任何加密标记。 CMAF 请求需要在 URL 中的加密标记。 使用多个加密类型配置其他协议也需要消除二义性加密标记。 |
|MPE_STORAGE_BAD_URL_SYNTAX |0x808900E9|要完成请求失败，出现错误的请求错误存储的请求。 |

## <a name="403-forbidden"></a>403 禁止访问

出于以下原因之一，未允许该请求：

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_STORAGE_AUTHENTICATION_FAILED |0x808900EA|对存储的请求来完成该请求失败，出现身份验证失败。 可能的原因轮换存储密钥和服务无法同步存储密钥。 <br/><br/>请联系 Azure 支持通过转到[帮助 + 支持](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)在 Azure 门户中。|
|MPE_STORAGE_INSUFFICIENT_ACCOUNT_PERMISSIONS |0x808900EB |存储操作错误，访问失败，因为帐户权限不足。 |
|MPE_STORAGE_ACCOUNT_IS_DISABLED |0x808900EC |对存储的请求来完成该请求失败，因为的存储帐户已禁用。 |
|MPE_STORAGE_AUTHENTICATION_FAILURE |0x808900F3 |存储操作错误，访问失败，因为一般错误。 |
|MPE_OUTPUT_FORMAT_BLOCKED |0x80890207 |输出格式是由于 StreamingPolicy 中的配置而被阻止。 |
|MPE_ENC_ENCRYPTION_REQUIRED |0x8088021E |加密是必需的内容，传递策略所需的输出格式。 |
|MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY |0x8088024D |交付策略设置中未设置加密。 |

## <a name="404-not-found"></a>404 未找到

操作正在尝试对不再存在的资源。 例如，资源已被删除。

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_EGRESS_TRACK_NOT_FOUND |0x80890209 |找不到请求的跟踪。 |
|MPE_RESOURCE_NOT_FOUND |0x808901F9 |找不到请求的资源。 |
|MPE_UNAUTHORIZED |0x80890244 |未经授权访问。 |
|MPE_EGRESS_TIMESTAMP_NOT_FOUND |0x8089020A |找不到请求的时间戳。 |
|MPE_EGRESS_FILTER_NOT_FOUND |0x8089020C |找不到请求的动态清单筛选器。 |
|MPE_FRAGMENT_BY_INDEX_NOT_FOUND |0x80890252 |请求的片段索引不在有效范围。 |
|MPE_LIVE_MEDIA_ENTRIES_NOT_FOUND |0x80890254 |实时的媒体条目无法找到要获取 moov 缓冲区。 |
|MPE_FRAGMENT_TIMESTAMP_NOT_FOUND |0x80890255 |找不到请求时为特定的轨迹片段。<br/><br/>可能是，片段不是在存储中。 请重试可能会有一个片段的演示文稿的不同的层。 |
|MPE_MANIFEST_MEDIA_ENTRY_NOT_FOUND |0x80890256 |找不到清单中的请求比特率媒体项。 <br/><br/>可能是播放机要求的不是清单中的某些比特率的视频轨道。|
|MPE_METADATA_NOT_FOUND |0x80890257 |找不到清单中的某些元数据或找不到从存储变基。 |
|MPE_STORAGE_RESOURCE_NOT_FOUND |0x808900ED |存储操作错误，找不到资源。 |

## <a name="409-conflict"></a>409 冲突

提供的资源 ID 上`PUT`或`POST`操作已被现有资源。 使用另一个资源 ID 来解决此问题。

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_STORAGE_CONFLICT  |0x808900EE  |存储操作错误，冲突错误。  |

## <a name="410"></a>410

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_FILTER_FORCE_END_LEFT_EDGE_CROSSED_DVR_WINDOW|0x80890263|实时传送视频流，具有筛选器设置为 true，开始或结束时间戳的 forceEndTimestamp 时，当前的 DVR 窗口之外。|

## <a name="412-precondition-failure"></a>412 不满足前提条件

该操作指定不同于在服务器上，即，开放式并发错误可用的版本的 eTag。 读取资源的最新版本和更新请求的 eTag 后重试请求。

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_FRAGMENT_NOT_READY |0x80890200 |请求的片段尚未准备就绪。|
|MPE_STORAGE_PRECONDITION_FAILED| 0x808900EF|存储操作错误，不满足前提条件失败。|

## <a name="415-unsupported-media-type"></a>415 不支持的媒体类型

客户端发送的负载格式的格式不受支持。

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_ENC_ALREADY_ENCRYPTED| 0x8088021F| 应将应用到加密已加密的内容。|
|MPE_ENC_INVALID_INPUT_ENCRYPTION_FORMAT|0x8088021D |加密是无效的输入格式。|
|MPE_INVALID_ASSET_DELIVERY_POLICY_TYPE|0x8088021C| 传送策略类型无效。|
|MPE_ENC_MULTIPLE_SAME_DELIVERY_TYPE|0x8088024E |可能由多种输出格式共享原始设置。|
|MPE_FORMAT_NOT_SUPPORTED|0x80890205|媒体格式或类型不受支持。 例如，Media Services 不支持超过 64 的质量级别计数。 在 FLV 视频标记中，媒体服务不支持使用多个 SPS 和多个放映视频帧|
|MPE_INPUT_FORMAT_NOT_SUPPORTED|0x80890218| 不支持所请求资产的输入的格式。 媒体服务支持平滑流 （实时） MP4 (VoD) 和渐进式下载格式。|
|MPE_OUTPUT_FORMAT_NOT_SUPPORTED|0x8089020D|不支持请求的输出格式。 媒体服务支持平滑流、 DASH （CSF、 CMAF）、 HLS (v3 和 v4，CMAF) 和渐进式下载格式。|
|MPE_ENCRYPTION_NOT_SUPPORTED|0x80890208|遇到不受支持的加密类型。|
|MPE_MEDIA_TYPE_NOT_SUPPORTED|0x8089020E|输出格式不支持请求的媒体类型。 支持的类型为视频或音频"SUBT"副标题。|
|MPE_MEDIA_ENCODING_NOT_SUPPORTED|0x8089020F|源资产媒体已编码的媒体格式的输出格式与不兼容。|
|MPE_VIDEO_ENCODING_NOT_SUPPORTED|0x80890210|源资产已使用与输出格式不兼容的视频格式编码。 支持 H.264，AVC、 H.265 （HEVC、 hev1 或 hvc1）。|
|MPE_AUDIO_ENCODING_NOT_SUPPORTED|0x80890211|源资产已使用与输出格式不兼容的音频格式编码。 支持的音频格式为 AAC、 e-ac3 （DD +）、 Dolby DTS。|
|MPE_SOURCE_PROTECTION_CONVERSION_NOT_SUPPORTED|0x80890212|源受保护的资产不能转换为输出格式。|
|MPE_OUTPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890213|输出格式不支持保护格式。|
|MPE_INPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890219|输入格式不支持保护格式。|
|MPE_INVALID_VIDEO_NAL_UNIT|0x80890231|无效视频 NAL 单位，例如，仅第一个 NAL 中示例中可以为 AUD.|
|MPE_INVALID_NALU_SIZE|0x80890260|NAL 单元大小无效。|
|MPE_INVALID_NALU_LENGTH_FIELD|0x80890261|NAL 单元长度值无效。|
|MPE_FILTER_INVALID|0x80890236|无效动态清单筛选器。|
|MPE_FILTER_VERSION_INVALID|0x80890237|无效或不受支持的筛选器版本。|
|MPE_FILTER_TYPE_INVALID|0x80890238|无效的筛选器类型。|
|MPE_FILTER_RANGE_ATTRIBUTE_INVALID|0x80890239|无效的范围被指定的筛选器。|
|MPE_FILTER_TRACK_ATTRIBUTE_INVALID|0x8089023A|筛选器指定无效的跟踪属性。|
|MPE_FILTER_PRESENTATION_WINDOW_INVALID|0x8089023B|无效的演示文稿窗口长度指定筛选器。|
|MPE_FILTER_LIVE_BACKOFF_INVALID|0x8089023C|无效实时回退被指定的筛选器。|
|MPE_FILTER_MULTIPLE_SAME_TYPE_FILTERS|0x8089023D|只有一个 absTimeInHNS 元素支持旧的筛选器。|
|MPE_FILTER_REMOVED_ALL_STREAMS|0x8089023E|没有更多的流根本之后应用筛选器。|
|MPE_FILTER_LIVE_BACKOFF_OVER_DVRWINDOW|0x8089023F|实时退避已超出 DVR 窗口。|
|MPE_FILTER_LIVE_BACKOFF_OVER_PRESENTATION_WINDOW|0x80890262|实时退避大于演示窗口。|
|MPE_FILTER_COMPOSITION_FILTER_COUNT_OVER_LIMIT|0x80890246|已超过十 （10） 的最大允许的默认值筛选器。|
|MPE_FILTER_COMPOSITION_MULTIPLE_FIRST_QUALITY_OPERATOR_NOT_ALLOWED|0x80890248|合并的请求筛选器中不允许多个第一个视频质量运算符。|
|MPE_FILTER_FIRST_QUALITY_ATTRIBUTE_INVALID|0x80890249|第一个质量比特率属性数必须为一 (1)。|
|MPE_HLS_SEGMENT_TOO_LARGE|0x80890243|HLS 段持续时间必须小于三分之一的 DVR 窗口和 HLS 后退。|
|MPE_KEY_FRAME_INTERVAL_TOO_LARGE|0x808901FE|片段持续时间必须小于或等于大约 20 秒，或输入的质量级别并不对齐的时间。|
|MPE_DTS_RESERVEDBOX_EXPECTED|0x80890105|它应在分析 DTS 框期间呈现 DTSSpecficBox 中时，特定于 DTS 的错误，找不到 ReservedBox。|
|MPE_DTS_INVALID_CHANNEL_COUNT|0x80890106|特定于 DTS 的错误，在 DTS 框分析 DTSSpecficBox 中找到任何通道。|
|MPE_DTS_SAMPLETYPE_MISMATCH|0x80890107|特定于 DTS 的错误，DTSSpecficBox 中的示例类型不匹配。|
|MPE_DTS_MULTIASSET_DTSH_MISMATCH|0x80890108|特定于 DTS 的错误，但 DTSH 示例类型不匹配设置多资产。|
|MPE_DTS_INVALID_CORESTREAM_SIZE|0x80890109|特定于 DTS 的错误，核心流大小无效。|
|MPE_DTS_INVALID_SAMPLE_RESOLUTION|0x8089010A|特定于 DTS 的错误示例解决办法是无效的。|
|MPE_DTS_INVALID_SUBSTREAM_INDEX|0x8089010B|特定于 DTS 的错误子流扩展索引无效。|
|MPE_DTS_INVALID_BLOCK_NUM|0x8089010C|特定于 DTS 的错误子流块号无效。|
|MPE_DTS_INVALID_SAMPLING_FREQUENCE|0x8089010D|特定于 DTS 的错误，采样频率无效。|
|MPE_DTS_INVALID_REFCLOCKCODE|0x8089010E|特定于 DTS 的错误子流扩展中的引用时钟代码无效。|
|MPE_DTS_INVALID_SPEAKERS_REMAP|0x8089010F|DTS 特定于错误的演讲者重新映射集数量无效。|

有关加密的文章和示例，请参阅：

- [概念： 内容保护](content-protection-overview.md)
- [概念：内容密钥的策略](content-key-policy-concept.md)
- [概念：流式处理策略](streaming-policy-concept.md)
- [示例： 使用 AES 加密进行保护](protect-with-aes128.md)
- [示例： 使用 DRM 进行保护](protect-with-drm.md)

有关筛选器的指南，请参阅：

- [动态清单是概念：](filters-dynamic-manifest-overview.md)
- [概念： 筛选器](filters-concept.md)
- [示例： 使用 REST Api 创建筛选器](filters-dynamic-manifest-rest-howto.md)
- [示例： 使用.NET 创建筛选器](filters-dynamic-manifest-dotnet-howto.md)
- [示例： 使用 CLI 创建筛选器](filters-dynamic-manifest-cli-howto.md)

有关实时文章和示例，请参阅：

- [概念： 实时流式处理概述](live-streaming-overview.md)
- [概念：实时事件和实时输出](live-events-outputs-concept.md)
- [示例： 实时流式处理教程](stream-live-tutorial-with-api.md)

## <a name="416-range-not-satisfiable"></a>416 无法满足的范围

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_STORAGE_INVALID_RANGE|0x808900F1|存储操作错误，返回 http 416 错误，无效的范围。|

## <a name="500-internal-server-error"></a>500 内部服务器错误

在处理请求期间，媒体服务遇到了某种错误，从而阻止继续处理请求。  

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_STORAGE_SOCKET_TIMEOUT|0x808900F4|接收和从已翻译 Winhttp 错误代码的 ERROR_WINHTTP_TIMEOUT (0x00002ee2)。|
|MPE_STORAGE_SOCKET_CONNECTION_ERROR|0x808900F5|接收和从已翻译 Winhttp 错误代码的 ERROR_WINHTTP_CONNECTION_ERROR (0x00002efe)。|
|MPE_STORAGE_SOCKET_NAME_NOT_RESOLVED|0x808900F6|接收和从已翻译 Winhttp 错误代码的 ERROR_WINHTTP_NAME_NOT_RESOLVED (0x00002ee7)。|
|MPE_STORAGE_INTERNAL_ERROR|0x808900E6|存储操作错误，常规 InternalError 的其中一个 HTTP 500 错误。|
|MPE_STORAGE_OPERATION_TIMED_OUT|0x808900E7|存储操作错误，其中一个 HTTP 500 错误的常规 OperationTimedOut。|
|MPE_STORAGE_FAILURE|0x808900F2|存储操作错误，比 InternalError 或 OperationTimedOut 其他 HTTP 500 错误。|

## <a name="503-service-unavailable"></a>503 服务不可用

服务器当前无法接收请求。 服务请求过多可能引发此错误。 媒体服务限制机制会限制那些发出过多服务请求的应用程序的资源使用情况。

> [!NOTE]
> 检查错误消息和错误代码字符串以获取有关收到 503 错误的原因的更多详细信息。 此错误并不始终意味着限制。
> 

|错误代码|十六进制值 |错误说明|
|---|---|---|
|MPE_STORAGE_SERVER_BUSY|0x808900E8|存储操作错误，接收到 HTTP 服务器繁忙错误 503。|

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="see-also"></a>另请参阅

- [编码错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)
- [Azure 媒体服务概念](concepts-overview.md)
- [配额和限制](limits-quotas-constraints.md)

## <a name="next-steps"></a>后续步骤

[示例： 从 ApiException 使用.NET 访问错误代码和消息](configure-connect-dotnet-howto.md#connect-to-the-net-client)
