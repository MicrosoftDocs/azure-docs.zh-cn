---
title: 保护你的内容使用 Media Services 动态加密-Azure |Microsoft Docs
description: 本文概述了采用动态加密的内容保护。 它还将演示有关流式处理协议和加密类型。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/21/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: e13bcb7d4eeded691669277b64aba9048f3bbefa
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150416"
---
# <a name="content-protection-with-dynamic-encryption"></a>使用动态加密内容保护

可以使用 Azure 媒体服务在媒体从计算机离开到存储、处理和传送的整个过程中确保其安全。 借助媒体服务，可以传送使用高级加密标准 (AES-128) 或三个主要数字版权管理 (DRM) 系统（Microsoft PlayReady、Google Widevine 和 Apple FairPlay）中任意一个动态加密的实时和请求内容。 媒体服务还提供了用于向已授权客户端传送 AES 密钥和 DRM（PlayReady、Widevine 和 FairPlay）许可证的服务。 

下图阐释了媒体服务内容保护工作流： 

![保护内容](./media/content-protection/content-protection.svg)

&#42; *动态加密支持 AES-128“明文密钥”、CBCS 和 CENC。有关详细信息，请参阅[此处](#streaming-protocols-and-encryption-types)的支持矩阵。*

本文介绍与了解使用媒体服务进行内容保护相关的概念和术语。

## <a name="main-components-of-a-content-protection-system"></a>内容保护系统的主要组件

若要成功完成“内容保护”系统/应用程序设计中，需要全面了解工作范围。 以下列表概述了需要实施的三个部分。 

1. Azure 媒体服务代码
  
   [DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs)示例演示如何实现使用媒体服务 v3 多重 DRM 系统和也使用媒体服务许可证/密钥传送服务。 可以使用多个加密类型（AES-128、PlayReady、Widevine、FairPlay）来加密每个资产。 请参阅[流式处理协议和加密类型](#streaming-protocols-and-encryption-types)，以了解有效的组合方式。
  
   该示例演示如何：

   1. 创建和配置[内容策略](https://docs.microsoft.com/rest/api/media/contentkeypolicies)。

      * 定义许可证交付授权，指定基于 JWT 中的声明执行的授权检查逻辑。
      * 通过指定内容密钥配置 DRM 加密。
      * 配置[PlayReady](playready-license-template-overview.md)， [Widevine](widevine-license-template-overview.md)，和/或[FairPlay](fairplay-license-overview.md)许可证。 使用这些模板可为使用的每个 DRM 配置权利和权限。

        ```
        ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
        ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
        ContentKeyPolicyFairPlayConfiguration fairPlayConfig = ConfigureFairPlayPolicyOptions();
        ```
   2. 创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)，它配置为流式传输加密的资产。 
  
      **流式处理定位符**必须与关联[流式处理策略](https://docs.microsoft.com/rest/api/media/streamingpolicies)。 在示例中，我们将 StreamingLocator.StreamingPolicyName 设置为"Predefined_MultiDrmCencStreaming"策略。 此策略表示我们要为两个内容密钥 （信封和 CENC），以获取生成并设置该定位符。 因此，应用了信封、PlayReady 和 Widevine 加密（根据配置的 DRM 许可证将密钥传送到播放客户端）。 如果还要使用 CBCS (FairPlay) 加密流，请使用“Predefined_MultiDrmStreaming”。
    
      由于我们想要加密视频中，因此**内容密钥策略**我们前面配置也必须与关联**流式处理定位符**。 
    
   3. 创建测试令牌。

      **GetTokenAsync** 方法演示如何创建测试令牌。
   4. 生成流 URL。

      **GetDASHStreamingUrlAsync** 方法演示如何生成流 URL。 在本例中，URL 流式传输 **DASH** 内容。

2. 使用 AES 或 DRM 客户端的播放器。 基于播放器 SDK 的视频播放器应用（本机或基于浏览器）需要满足以下要求：
   * 播放器 SDK 支持所需的 DRM 客户端
   * 播放器 SDK 支持所需的流式处理协议：平滑流式处理、DASH 和/或 HLS
   * 播放器 SDK 需要能够处理许可证获取请求中 JWT 令牌的传递
  
     可以使用 [Azure Media Player API](https://amp.azure.net/libs/amp/latest/docs/) 创建播放器。 通过 [Azure 媒体播放器的 ProtectionInfo API](https://amp.azure.net/libs/amp/latest/docs/) 指定要在不同的 DRM 平台上使用哪种 DRM 技术。

     若要测试 AES 或 CENC (Widevine 和/或 PlayReady) 加密内容，可以使用 [Azure Media Player](https://ampdemo.azureedge.net/azuremediaplayer.html)。 请务必单击“高级选项”，并选择加密选项。

     若要测试 FairPlay 加密内容，请使用[此测试播放器](https://aka.ms/amtest)。 该播放器支持 Widevine、PlayReady 和 FairPlay DRM，以及 AES-128 明文密钥加密。 
    
     需要选择适当的浏览器来测试不同的 DRM：使用 Chrome/Opera/Firefox 测试 Widevine，使用 Microsoft Edge/IE11 测试 PlayReady，使用 macOS 版 Safari 测试 FairPlay。

3. 安全令牌服务 (STS)，颁发 JSON Web 令牌 (JWT) 作为用于访问后端资源的访问令牌。 可以使用 AMS 许可证传送服务作为后端资源。 STS 必须定义以下信息：

   * 颁发者和受众（或范围）
   * 与内容保护中的业务要求相关的声明
   * 签名的对称或非对称验证
   * 密钥滚动更新支持（如有必要）

     可以使用[此 STS 工具](https://openidconnectweb.azurewebsites.net/DRMTool/Jwt)测试 STS，此工具支持所有 3 种类型的验证密钥：对称、非对称，或者带密钥滚动更新的 Azure AD。 

> [!NOTE]
> 强烈建议在移到下一部分之前，将重点放在每个部分的完整测试上。 若要测试“内容保护”系统，请使用上述列表中指定的工具。  

## <a name="streaming-protocols-and-encryption-types"></a>流式处理协议和加密类型

可以通过 Azure 媒体服务传送使用 AES 明文密钥或 DRM 加密（利用 PlayReady、Widevine 或 FairPlay）动态加密的内容。 当前可以加密 HTTP Live Streaming (HLS)、MPEG DASH 和平滑流式处理格式。 每个协议支持以下加密方法：

### <a name="hls"></a>HLS

HLS 协议支持以下容器格式和加密方案。

|容器格式|加密方案|URL 示例|
|---|---|---|
|全部|AES|`https://amsv3account-usw22.streaming.media.azure.net/<id>/ignite.ism/manifest(format=m3u8-aapl,encryption=cbc)`|
|MPG2-TS |CBC (FairPlay) ||
|CMAF(fmp4) |CBC (FairPlay) |`https://amsv3account-usw22.streaming.media.azure.net/<id>/ignite.ism/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`|
|MPG2-TS |CENC (PlayReady) ||
|CMAF(fmp4) |CENC (PlayReady) ||

/ CMAF HLS + FairPlay (包括 HEVC / H.265) 支持的以下设备：

* iOS v11 或更高版本 
* iPhone 8 或更高版本
* 使用 Intel MacOS high Sierra 第 7 个常规 CPU

### <a name="mpeg-dash"></a>MPEG-DASH

MPEG DASH 协议支持以下容器格式和加密方案。

|容器格式|加密方案|URL 示例
|---|---|---|
|全部|AES|`https://amsv3account-usw22.streaming.media.azure.net/<id>/ignite.ism/manifest(format=mpd-time-csf,encryption=cbc)`|
|CSF(fmp4) |CENC (Widevine + PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/<id>/ignite.ism/manifest(format=mpd-time-csf,encryption=cenc)`|
|CMAF(fmp4)|CENC (Widevine + PlayReady)||

### <a name="smooth-streaming"></a>平滑流

平滑流式处理协议支持以下容器格式和加密方案。

|Protocol|容器格式|加密方案|
|---|---|---|
|fMP4|AES||
|fMP4 | CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/<id>/ignite.ism/manifest(encryption=cenc)`|

### <a name="browsers"></a>浏览器

常见的浏览器支持以下 DRM 客户端：

|浏览器|加密|
|---|---|
|Chrome|Widevine|
|Edge, IE 11|PlayReady|
|Firefox|Widevine|
|Opera|Widevine|
|Safari|FairPlay|

## <a name="aes-128-clear-key-vs-drm"></a>AES-128 明文密钥与DRM

客户通常希望知道他们应该使用 AES 加密还是 DRM 系统。 两个系统之间的主要区别是，使用 AES 加密内容密钥传输到客户端通过 TLS，以便在传输过程中，但无需任何额外的加密 （"明文"），该密钥进行加密。 因此，用来解密内容的密钥到客户端播放器可访问，可以查看网络跟踪在纯文本中的客户端上。 AES-128 明文密钥加密适合查看者是受信任方的用例（例如，员工查看公司内分发的加密公司视频）。

DRM 系统等 PlayReady、 Widevine 和 FairPlay 全都提供一层额外的加密密钥用于解密 AES-128 明文密钥与相比的内容。 内容密钥加密中的 DRM 运行时所保护的密钥为附加到提供的 TLS 任何传输级加密。 此外，解密是操作系统级别在安全的环境中处理的，在这样的环境中，恶意用户更难进行攻击。 在查看者可能不是受信任方且需要更高等级的安全性的用例中，建议使用 DRM。

## <a name="dynamic-encryption-and-key-delivery-service"></a>动态加密和密钥传送服务

在媒体服务 v3，内容密钥是与流式处理定位符相关联 (请参阅[本例](protect-with-aes128.md))。 如果使用媒体服务密钥传送服务，您可以让 Azure 媒体服务为你生成内容密钥。 如果使用自己的密钥传送服务，或者需要处理高可用性方案（需要在两个数据中心使用相同的内容密钥），则应自行生成内容密钥。

播放器请求流时，媒体服务将通过 AES 明文密钥或 DRM 加密使用指定的密钥来动态加密内容。 为了解密流，播放器将从媒体服务密钥传送服务或者指定的密钥传送服务请求密钥。 为了确定是否已授权用户获取密钥，服务将评估你为密钥指定的内容密钥策略。

媒体服务提供用于向已授权客户端传送 DRM（PlayReady、Widevine 和 FairPlay）许可证和 AES 密钥的密钥传送服务。 可以使用 REST API 或媒体服务客户端库来配置许可证和密钥的授权与身份验证策略。

### <a name="custom-key-and-license-acquisition-url"></a>自定义密钥和许可证获取 URL

如果你想要指定不同密钥和许可证传送服务 （而不是媒体服务），请使用以下模板。 在模板中的两个可替换字段有，以便可以跨多个资产而不是创建每个资产流式处理策略共享你的流式处理策略。 

* EnvelopeEncryption.CustomKeyAcquisitionUrlTemplate-模板自定义服务密钥传送到最终用户播放器的 URL。 使用用于颁发密钥的 Azure 媒体服务时，不需要。 模板支持替换本服务将在运行时使用特定于请求的值更新的令牌。  当前支持的令牌值为 {AlternativeMediaId}，其中替换为 StreamingLocatorId.AlternativeMediaId 和 {ContentKeyId} 的值，这将替换所请求的密钥标识符的值。
* StreamingPolicyPlayReadyConfiguration.CustomLicenseAcquisitionUrlTemplate-模板自定义服务传送到最终用户播放机的许可证的 URL。 不需要时使用 Azure 媒体服务进行颁发许可证。 模板支持替换本服务将在运行时使用特定于请求的值更新的令牌。 当前支持的令牌值为 {AlternativeMediaId}，其中替换为 StreamingLocatorId.AlternativeMediaId 和 {ContentKeyId} 的值，这将替换所请求的密钥标识符的值。 
* StreamingPolicyWidevineConfiguration.CustomLicenseAcquisitionUrlTemplate-与上述相同，仅为 Widevine。 
* StreamingPolicyFairPlayConfiguration.CustomLicenseAcquisitionUrlTemplate-与上述相同，仅对 FairPlay。  

例如：

```csharp
streamingPolicy.EnvelopEncryption.customKeyAcquisitionUrlTemplate = "https://mykeyserver.hostname.com/envelopekey/{AlternativeMediaId}/{ContentKeyId}";
```

`ContentKeyId`的值为所请求的密钥和`AlternativeMediaId`如果你想要将请求映射到一端的实体可以使用。 例如，`AlternativeMediaId`可用于帮助您查找的权限。

使用自定义的 REST 示例密钥和许可证获取 Url，请参阅[流式处理策略-创建](https://docs.microsoft.com/rest/api/media/streamingpolicies/create)

## <a name="control-content-access"></a>控制内容访问

可以通过配置内容密钥策略来控制谁有权访问内容。 媒体服务支持通过多种方式对发出密钥请求的用户进行授权。 必须配置内容密钥策略。 客户端（播放器）必须符合该策略才能将密钥传送到客户端。 内容密钥策略可以采用**开放**或**令牌**限制。 

使用令牌限制的内容密钥策略时，内容密钥仅发送到在密钥/许可证请求中提供了有效 JSON Web 令牌 (JWT) 或简单 Web 令牌 (SWT) 的客户端。 此令牌必须是由安全令牌服务 (STS) 颁发的。 可以使用 Azure Active Directory 作为 STS，也可以部署自定义 STS。 必须将 STS 配置为创建令牌，该令牌使用指定密钥以及在令牌限制配置中指定的颁发声明进行签名。 如果令牌有效，而且令牌中的声明与为密钥/许可证配置的声明相匹配，则媒体服务密钥传送服务会将所请求的密钥/许可证返回到客户端。

配置令牌限制策略时，必须指定主验证密钥、颁发者和受众参数。 主验证密钥包含为令牌签名时使用的密钥。 颁发者是颁发令牌的安全令牌服务。 受众（有时称为范围）描述该令牌的意图，或者令牌授权访问的资源。 媒体服务密钥交付服务会验证令牌中的这些值是否与模板中的值匹配。

客户通常使用自定义 STS 使用不同的 DRM 许可证参数 （而不是租赁许可证有订阅许可证） 的不同 ContentKeyPolicyOptions 之间进行选择，或包含表示内容密钥的声明的令牌中包含自定义声明令牌授予访问权限的密钥标识符。
 
## <a name="storage-side-encryption"></a>存储端加密

若要保护静态资产，应通过存储端加密对资产进行加密。 下表显示了存储端加密在媒体服务 v3 中的工作方式：

|加密选项|描述|媒体服务 v3|
|---|---|---|
|媒体服务存储加密| AES-256 加密，媒体服务管理的密钥|不支持<sup>(1)</sup>|
|[静态数据的存储服务加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)|由 Azure 存储提供的服务器端加密，由 Azure 或客户管理的密钥|支持|
|[存储客户端加密](https://docs.microsoft.com/azure/storage/common/storage-client-side-encryption)|由 Azure 存储提供的客户端加密，由 Key Vault 中的客户管理的密钥|不支持|

<sup>1</sup> 在媒体服务 v3 中，仅当资产是使用媒体服务 v2 创建的时才支持存储加密（AES-256 加密）以实现向后兼容性。 这意味着 v3 会处理现有的存储加密资产，但不会允许创建新资产。

## <a name="troubleshoot"></a>故障排除

如果收到`MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY`错误，请确保指定适当的流式处理策略。

如果出现错误结尾`_NOT_SPECIFIED_IN_URL`，请确保在 URL 中指定的加密格式。 例如，`…/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`。 请参阅[流式处理协议和加密类型](#streaming-protocols-and-encryption-types)。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

* [使用 AES 加密进行保护](protect-with-aes128.md)
* [使用 DRM 提供保护](protect-with-drm.md)
* [设计多 DRM 内容保护系统，使用访问控制](design-multi-drm-system-with-access-control.md)
* [常见问题](frequently-asked-questions.md)

