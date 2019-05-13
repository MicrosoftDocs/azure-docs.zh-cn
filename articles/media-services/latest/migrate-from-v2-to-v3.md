---
title: 从 Azure 媒体服务 v2 迁移到 v3 | Microsoft Docs
description: 本文介绍了 Azure 媒体服务 v3 中引入的更改，并说明了两个版本之间的差异。 本文还提供了从媒体服务 v2 迁移到 v3 的迁移指南。
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: azure 媒体服务, 流, 广播, 实时, 脱机
ms.service: media-services
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 05/01/2019
ms.author: juliako
ms.openlocfilehash: e64b18cef44a5fa6d63137f57fcada27adcd0299
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205487"
---
# <a name="migration-guidance-for-moving-from-media-services-v2-to-v3"></a>有关从媒体服务 v2 迁移到 v3 的指导

本文介绍 Azure 媒体服务 v3 中引入的更改，说明两个版本之间的差异，并提供迁移指导。

如果你目前基于[旧版媒体服务 v2 API](../previous/media-services-overview.md) 开发了一个视频服务，则在迁移到 v3 API 之前，应查看以下指导原则和注意事项。 v3 API 中的许多优势和新功能可以改进开发体验和媒体服务的功能。 但是，如本文的[已知问题](#known-issues)部分中所述，API 版本的变化也带来了一些限制。 在媒体服务团队不断改进 v3 API 并解决版本差距的过程中，本页面会得到维护。 

> [!NOTE]
> 目前，无法使用 Azure 门户来管理 v3 资源。 请使用 [REST API](https://aka.ms/ams-v3-rest-ref)、[CLI](https://aka.ms/ams-v3-cli-ref) 或受支持的 [SDK](developers-guide.md) 之一。

## <a name="benefits-of-media-services-v3"></a>媒体服务 v3 的优势
  
### <a name="api-is-more-approachable"></a>API 更便于访问

*  v3 基于一个统一的 API 接口，该接口公开了基于 Azure 资源管理器构建的管理和操作功能。 Azure 资源管理器模板可用于创建和部署转换、流式处理终结点、实时事件等等。
* [开放的 API（又称为 Swagger）规范](https://aka.ms/ams-v3-rest-sdk)文档。
    公开所有服务组件的架构，包括基于文件的编码。
* 适用于 [.NET](https://aka.ms/ams-v3-dotnet-ref)、.NET Core、[Node.js](https://aka.ms/ams-v3-nodejs-ref)、[Python](https://aka.ms/ams-v3-python-ref)、[Java](https://aka.ms/ams-v3-java-ref)、[Go](https://aka.ms/ams-v3-go-ref) 和 Ruby 的 SDK。
* 简化脚本支持的 [Azure CLI](https://aka.ms/ams-v3-cli-ref) 集成。

### <a name="new-features"></a>新增功能

* 对于基于文件的作业处理，可以使用 HTTP(S) URL 作为输入。<br/>无需事先在 Azure 中存储内容，无需创建资产。
* 为基于文件的作业处理引入了[转换](transforms-jobs-concept.md)的概念。 使用转换可以生成可重用的配置、创建 Azure 资源管理器模板，并隔离多个客户或租户之间的处理设置。
* 一个资产可以有多个[流式处理定位符](streaming-locators-concept.md)，其中每个流式处理定位符有不同的[动态打包](dynamic-packaging-overview.md)和动态加密设置。
* [内容保护](content-key-policy-concept.md)支持多密钥功能。
* 使用媒体服务将单比特率贡献源转码为具有多比特率的输出流时，可以流式传输最长 24 小时的实时事件。
* 实时事件支持新的低延迟实时传送视频流。 有关详细信息，请参阅[延迟](live-event-latency.md)。
* 实时事件预览版支持[动态打包](dynamic-packaging-overview.md)和动态加密。 这样，用户便可以使用预览版中的内容保护以及 DASH 和 HLS 打包。
* 与 v2 API 中的节目实体相比，实时输出更易于使用。 
* 改进的 RTMP 支持（提高了稳定性并提供了更多的源编码器支持）。
* RTMPS 安全引入。<br/>创建实时事件时，将获得 4 个引入 URL。 这 4 个引入 URL 几乎是相同的，具有相同的流式处理令牌 (AppId)，仅端口号部分不同。 其中两个 URL 是 RTMPS 的主要和备份 URL。   
* 可对实体使用基于角色的访问控制 (RBAC)。 

## <a name="changes-from-v2"></a>自 v2 以来的更改

* 对于通过 v3 创建的资产，媒体服务仅支持 [Azure 存储服务器端存储加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)。
    * 对于通过 v2 API 创建的，并采用媒体服务提供的[存储加密](../previous/media-services-rest-storage-encryption.md) (AES 256) 的资产，可以使用 v3 API。
    * 无法使用 v3 API 创建采用旧版 AES 256 [存储加密](../previous/media-services-rest-storage-encryption.md)的新资产。
* v3 SDK 现在已与存储 SDK 分离，可让你更精细地控制所要使用的存储 SDK 版本，并避免版本控制问题。 
* 在 v3 API 中，所有编码比特率以“比特/秒”为单位。 这与 v2 Media Encoder Standard 预设不同。 例如，v2 中的比特率指定为 128 (kbps)，而在 v3 中，则指定 128000（比特/秒）。 
* v3 中不存在实体 AssetFile、AccessPolicy 和 Ingestmanifest。
* v3 中不存在 IAsset.ParentAssets 属性。
* 现在，Contentkey 不再是实体，而是流式处理定位符的一个属性。
* 事件网格支持替换 NotificationEndpoint。
* 以下实体已重命名
    * 作业输出取代了任务，现在是作业的一部分。
    * 流式处理定位符取代了定位符。
    * 实时事件取代了频道。<br/>实时事件计费基于实时频道计量器。 有关详细信息，请参阅[计费](live-event-states-billing.md)和[定价](https://azure.microsoft.com/pricing/details/media-services/)。
    * 实时输出取代了节目。
* 实时输出在创建时启动，在删除后停止。 v2 API 中的节目以不同的方式工作，它们必须在创建后启动。
*  若要获取有关作业的信息，您需要知道在其下创建作业的转换名称。 

## <a name="feature-gaps-with-respect-to-v2-apis"></a>与 v2 API 之间的功能差距

与 v2 API 相比，v3 API 存在以下功能差距。 我们正在弥补这些差距。

* [Premium Encoder](../previous/media-services-premium-workflow-encoder-formats.md) 和旧版[媒体分析处理器](../previous/media-services-analytics-overview.md)（Azure 媒体服务索引器 2 预览版、Face Redactor 等）不可通过 v3 访问。<br/>想要从媒体索引器 1 或 2 预览版迁移的客户可以立即使用 v3 API 中的 AudioAnalyzer 预设。  此新预设包含的功能比旧版媒体索引器 1 或 2 更多。 
* 许多[Media Encoder Standard v2 中的高级功能](../previous/media-services-advanced-encoding-with-mes.md)Api 目前不可在 v3，如：
    * 剪辑（适用于按需和实时方案）
    * 资产拼接
    * 叠加
    * 裁剪
    * 缩略图子画面
    * 插入静音曲目时输入不包含音频
    * 插入时输入的视频轨道不包含视频
* 包含转码的实时事件目前不支持静态图像插入中间流，以及通过 API 调用执行的广告标记插入。 

> [!NOTE]
> 请将本文加入书签，并不时地查看最新信息。
 
## <a name="code-differences"></a>代码差异

下表显示了常见方案中 v2 和 v3 的代码差异。

|场景|V2 API|V3 API|
|---|---|---|
|创建资产并上传文件 |[v2 .NET 示例](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L113)|[v3 .NET 示例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L169)|
|提交作业|[v2 .NET 示例](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L146)|[v3 .NET 示例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L298)<br/><br/>演示如何先创建转换，再提交作业。|
|发布使用 AES 加密的资产 |1.创建 ContentKeyAuthorizationPolicyOption<br/>2.创建 ContentKeyAuthorizationPolicy<br/>3.创建 AssetDeliveryPolicy<br/>4.创建资产并上传内容或提交作业并使用输出资产<br/>5.将 AssetDeliveryPolicy 与 Asset 关联<br/>6.创建 ContentKey<br/>7.将 ContentKey 附加到 Asset<br/>8.创建 AccessPolicy<br/>9.创建 Locator<br/><br/>[v2 .NET 示例](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L64)|1.创建内容密钥策略<br/>2.创建 Asset<br/>3.上传内容或将 Asset 用作 JobOutput<br/>4.创建流式处理定位符<br/><br/>[v3 .NET 示例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs#L105)|
|获取作业详细信息和管理作业 |[使用 v2 管理作业](../previous/media-services-dotnet-manage-entities.md#get-a-job-reference) |[使用 v3 管理作业](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L546)|

## <a name="known-issues"></a>已知问题

* 目前，无法使用 Azure 门户来管理 v3 资源。 请使用 [REST API](https://aka.ms/ams-v3-rest-sdk)、CLI 或支持的 SDK 之一。
* 需要在帐户中预配媒体保留单位 (MRU) 才能控制作业的并发和性能，特别是涉及到视频或音频分析的那些作业。 有关详细信息，请参阅[缩放媒体处理](../previous/media-services-scale-media-processing-overview.md)。 可以使用[适用于媒体服务 v3 的 CLI 2.0](media-reserved-units-cli-how-to.md)、使用 [Azure 门户](../previous/media-services-portal-scale-media-processing.md)或 [v2 API](../previous/media-services-dotnet-encoding-units.md) 来管理 MRU。 无论使用的是媒体服务 v2 还是 v3 API，都需要预配 MRU。
* 无法使用 v2 API 来管理通过 v3 API 创建的媒体服务实体。  
* 不建议使用 v3 API 来管理通过 v2 API 创建的实体。 下面是会导致两个版本中的实体不兼容的差异示例：   
    * 在 v2 中创建作业和任务不会在 v3 中显示，因为它们不与转换相关联。 建议切换到 v3 转换和作业。 在切换过程中，需要花费一段相对较短的时间来监视即时 v2 作业。
    * 无法持续使用 v3 来管理通过 v2 创建的频道和节目（映射到 v3 中的实时事件和实时输出）。 建议在方便停止频道时切换到 v3 实时事件和实时输出。<br/>目前，无法持续迁移正在运行的频道。  

> [!NOTE]
> 在媒体服务团队不断改进 v3 API 并解决版本差距的过程中，本页面会得到维护。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

若要了解如何轻松地开始编码和流式传输视频文件，请查看[流文件](stream-files-dotnet-quickstart.md)。 

