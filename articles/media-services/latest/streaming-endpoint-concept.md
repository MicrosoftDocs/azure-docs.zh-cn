---
title: 流式处理终结点 （来源） 在 Azure 媒体服务 |Microsoft Docs
description: 在 Azure 媒体服务流式处理终结点 （来源） 表示动态打包和流式处理服务，可以直接向客户端播放器应用程序，或以进一步分发给内容交付网络 (CDN) 传送内容。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 04/27/2019
ms.author: juliako
ms.openlocfilehash: 3f939154d2b34e6dc043e505ab89897221bcfe23
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149234"
---
# <a name="streaming-endpoints-origin"></a>流式处理终结点 （来源）

在 Microsoft Azure Media Services[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints)表示可直接向客户端播放器应用程序，使用其中一个传送实时和按需内容的动态 （实时） 打包和原始服务常用流式处理媒体协议 （HLS 或 DASH）。 此外，**流式处理终结点**提供到行业领先的 Drm 的动态 （实时） 加密。

用户创建媒体服务帐户时，将为用户创建一个处于“已停止”状态的默认流式处理终结点。 无法删除“默认”流式处理终结点。 可以在帐户下创建其他流式处理终结点 (请参阅[配额和限制](limits-quotas-constraints.md))。 

> [!NOTE]
> 若要开始流式处理视频，需启动要从中流式处理视频的**流式处理终结点**。 
>  
> 仅当流式处理终结点处于运行状态时才进行计费。

## <a name="naming-convention"></a>命名约定

对于默认终结点：`{AccountName}-{DatacenterAbbreviation}.streaming.media.azure.net`

对于任何其他终结点：`{EndpointName}-{AccountName}-{DatacenterAbbreviation}.streaming.media.azure.net`

## <a name="types"></a>类型  

有两种类型的**流式处理终结点**：**标准**（预览版） 和**高级**。 类型由用户为流式处理终结点分配的缩放单元（`scaleUnits`）数定义。 

下表描述了类型：  

|Type|缩放单元|描述|
|--------|--------|--------|  
|**标准**|0|默认的流式处理终结点是**标准**类型，可以通过调整更改为高级类型`scaleUnits`。|
|**高级**|>0|**高级**流式处理终结点是适用于高级工作负荷提供专用且可缩放带宽容量。 将移动到**Premium**类型通过调整`scaleUnits`（流式处理单位）。 `scaleUnits` 提供专用的出口容量，可以按照 200 Mbps 的增量购买。 使用高级类型时，每个启用的单元都为应用程序提供额外的带宽容量。 |

> [!NOTE]
> 对于想要将内容传送到大型的 internet 受众的客户，我们建议启用 CDN 的流式处理终结点上。

有关 SLA 的信息，请参阅[定价和 SLA](https://azure.microsoft.com/pricing/details/media-services/)。

## <a name="comparing-streaming-types"></a>比较流式处理类型

Feature|标准|高级
---|---|---
前 15 天免费<sup>1</sup>| 是 |否
Throughput |达 600 Mbps 并使用 CDN 时，可提供多更高的有效吞吐量。|每个流单元 (SU) 200 Mbps。 使用 CDN 时，可以提供多更高的有效吞吐量。
CDN|Azure CDN、第三方 CDN 或没有 CDN。|Azure CDN、第三方 CDN 或没有 CDN。
按比例计费| 每日|每日
动态加密|是|是
动态打包|是|是
缩放|自动增加到目标吞吐量。|额外的 Su
IP 筛选/G20/自定义主机<sup>2</sup>|是|是
渐进式下载|是|是
建议的用法 |建议用于绝大多数的流式处理方案。|专业用法。

<sup>1</sup>免费试用版仅适用于新创建的媒体服务帐户和默认的流式处理终结点。<br/>
<sup>2</sup> CDN 终结点上未启用时才使用直接在流式处理终结点上。<br/>

## <a name="properties"></a>属性 

本部分提供有关的某些流式处理终结点的属性的详细信息。 有关如何创建新流式处理终结点和所有属性描述的示例，请参阅[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints/create)。 

- `accessControl` -用于配置此流式处理终结点的以下安全设置：Akamai 签名标头身份验证密钥和允许连接到此终结点的 IP 地址。<br />时，才可以设置此属性`cdnEnabled`设置为 false。
- `cdnEnabled` -指示此流式处理终结点的 Azure CDN 集成已启用 （默认情况下禁用）。 如果将 `cdnEnabled` 设置为 true，则会禁用以下配置：`customHostNames` 和 `accessControl`。
  
    并非所有数据中心都支持 Azure CDN 集成。 若要检查你的数据中心有可用的 Azure CDN 集成，请执行以下操作：
 
  - 尝试设置`cdnEnabled`为 true。
  - 检查返回的结果为`HTTP Error Code 412`(PreconditionFailed) 的消息的"流式处理终结点 CdnEnabled 属性不能设置为 true，因为 CDN 功能在当前区域中不可用。" 

    如果出现此错误，则数据中心不支持 Azure CDN 集成。 你应该尝试其他数据中心。
- `cdnProfile` -当`cdnEnabled`设置为 true，你还可以传递`cdnProfile`值。 `cdnProfile` 是将在其中创建 CDN 终结点的 CDN 配置文件的名称。 可以提供现有的 cdnProfile 或使用新的 cdnProfile。 如果值为 NULL 且 `cdnEnabled` 为 true，则使用默认值“AzureMediaStreamingPlatformCdnProfile”。 如果提供的 `cdnProfile` 已经存在，则在其下创建一个终结点。 如果该配置文件不存在，新的配置文件会自动创建。
- `cdnProvider` -启用 CDN 后，你还可以传递`cdnProvider`值。 `cdnProvider` 控制将使用哪个提供程序。 目前，支持三个值：“StandardVerizon”、“PremiumVerizon”和“StandardAkamai”。 如果未不提供任何值和`cdnEnabled`为 true，"StandardVerizon"使用 （这是默认值）。
- `crossSiteAccessPolicies` -用于为各种客户端指定跨站点访问策略。 有关详细信息，请参阅[跨域策略文件规范](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)和[提供跨域边界的服务](https://msdn.microsoft.com/library/cc197955\(v=vs.95\).aspx)。<br/>设置仅适用于平滑流式处理。
- `customHostNames` -用于配置流式处理终结点以接受定向到自定义主机名的流量。  此属性适用于标准和高级流式处理终结点和时可以设置`cdnEnabled`: false。
    
    通过媒体服务，必须确认域名的所有权。 媒体服务通过要求验证域名所有权`CName`作为组件添加到使用中的域中包含的媒体服务帐户 ID 的记录。 例如，要将“sports.contoso.com”用作流式处理终结点的自定义主机名，则必须将 `<accountId>.contoso.com` 的记录配置为指向其中一个媒体服务验证主机名。 验证主机名由 verifydns.\<mediaservices-dns-zone> 组成。 

    以下是预期的 DNS 区域，要用于的验证记录中不同 Azure 区域。
  
  - North America、 Europe、 新加坡、 香港特別行政区，日本：
      
    - `media.azure.net`
    - `verifydns.media.azure.net`
      
  - 中国：
        
    - `mediaservices.chinacloudapi.cn`
    - `verifydns.mediaservices.chinacloudapi.cn`
        
    例如，`CName`映射"945a4c4e-28ea-45 cd-8ccb-a519f6b700ad 拥有"到"verifydns.media.azure.net"的记录可证明 Media Services ID 945a4c4e-28ea-45cd-8ccb-a519f6b700ad 因此具有 contoso.com 域的所有权启用 contoso.com 用作该帐户下的流式处理终结点的自定义主机名称下的任何名称。 若要查找媒体服务 ID 值，请转至 [Azure 门户](https://portal.azure.com/)，然后选择你的媒体服务帐户。 **帐户 ID**显示于顶部页面右上角。
        
    如果尝试在没有正确验证 `CName` 记录的情况下设置自定义主机名，则 DNS 响应将失败，然后缓存一段时间。 拥有适当的记录后，可能需要一段时间才能重新验证缓存的响应。 根据自定义域的 DNS 提供程序，重新验证记录可能需要几分钟到一个小时的时间。
        
    除了`CName`，它映射`<accountId>.<parent domain>`到`verifydns.<mediaservices-dns-zone>`，你必须创建另一个`CName`，它映射自定义主机名 (例如， `sports.contoso.com`) 到你的媒体服务流式处理终结点的主机名 (例如， `amstest-usea.streaming.media.azure.net`).
 
    > [!NOTE]
    > 流式处理终结点位于同一数据中心中，不能共享相同的自定义主机名。

    目前，媒体服务不支持的自定义域的 SSL。 
    
- `maxCacheAge` -重写默认的最大期限 HTTP 缓存控制标头已设置的媒体片段和点播清单上的流式处理终结点。 该值以秒为单位进行设置。
- `resourceState` -

    - 已停止的流式处理终结点创建后的初始状态
    - 正在启动-转换到正在运行状态
    - 正在运行的是能够将向客户端的流内容
    - 缩放-的规模单元正在增加或减少
    - 停止-正在过渡到停止状态
    - Deleting-正在删除
    
- `scaleUnits` -为您提供可按照 200 Mbps 的增量购买的专用的出口容量。 如果需要转到高级类型，请调整 `scaleUnits`。

## <a name="working-with-cdn"></a>配合 CDN 使用

在大多数情况下，应该启用 CDN。 但是，如果预计最大并发低于 500 个查看者，则建议禁用 CDN，因为 CDN 最适用于随并发缩放。

### <a name="considerations"></a>注意事项

* 无论你是否启用 CDN，流式处理终结点 `hostname` 和流式处理 URL 都保持不变。
* 如果需要使用或不使用 CDN 测试内容的功能，可以创建另一个流式处理终结点，不启用 CDN。

### <a name="detailed-explanation-of-how-caching-works"></a>有关缓存工作原理的详细说明

添加 CDN 时没有特定的带宽值，因为启用 CDN 的流式处理终结点所需的带宽量会有所不同。 很大程度上取决于内容的类型，它的受欢迎程度、比特率和协议。 CDN 仅缓存所请求的内容。 这意味着，只要缓存了视频片段，就会直接从 CDN 提供受欢迎的内容。 可能会缓存实时内容，因为通常会有很多人观看完全相同的内容。 按需内容可能有点棘手，因为你可能会有一些受欢迎的内容，也有一些不受欢迎的内容。 如果你有数百万的视频资产，其中没有一个是受欢迎的（每周只有 1 或 2 个查看者），但有成千上万的人观看所有不同的视频，CDN 的效果会大打折扣。 如果此缓存失误，则会增加流式处理终结点的负载。
 
还需要考虑自适应流式处理的工作原理。 每个单独的视频片段都作为自己的实体进行缓存。 例如，如果第一次观看某个视频，此人在此处仅仅观看几秒钟就跳过，那么只有与此人观看内容相关联的视频片段会被缓存在 CDN 中。 使用自适应流式处理，你通常会有 5 到 7 种不同的视频比特率。 如果一个人正在观看一种比特率并且另一个人在观看不同的比特率，则它们会在 CDN 中单独缓存。 即使两个人在观看相同的比特率，他们也可以通过不同的协议进行流式处理。 每个协议（HLS、MPEG-DASH、平滑流式处理）都是单独缓存的。 因此，会单独缓存每个比特率和协议，并且仅缓存已请求的视频片段。

### <a name="enable-azure-cdn-integration"></a>启用 Azure CDN 集成

流式处理终结点都预配有启用 CDN 后使用媒体服务上定义的等待时间进行 DNS 更新流式处理终结点映射到 CDN 终结点之前。

如果以后想要禁用/启用 CDN，流式处理终结点必须处于“已停止”状态。 可能需要长达两小时才能启用 Azure CDN 集成并使更改在所有 CDN POP 中生效。 但是，可以启动流式处理终结点和流，而不会被流式处理终结点中断，集成完成后，将从 CDN 传送流。 在预配期间，流式处理终结点的状态为“正在启动”，可能会观测到性能降低。

创建标准流式处理终结点时，它是默认情况下，与标准 Verizon 配置。 你可以配置使用 REST Api 的高级 Verizon 或 Akamai 标准提供程序。 

将在所有 Azure 数据中心启用 CDN 集成，但中国区域和联邦政府区域除外。

> [!IMPORTANT]
> 标准流式处理终结点可在 **Verizon 提供的 Azure CDN** 上实现 Azure 媒体服务与 Azure CDN 的集成。 可以使用所有 **Azure CDN 定价层和提供程序**配置高级流式处理终结点。 有关 Azure CDN 功能的详细信息，请参阅 [CDN 概述](../../cdn/cdn-overview.md)。

### <a name="determine-if-dns-change-has-been-made"></a>确定是否已进行 DNS 更改

可以通过确定是否已流式处理终结点 （流量将被定向到 Azure CDN） 上进行 DNS 更改 https://www.digwebinterface.com。 如果结果包含在结果中 azureedge.net 域名称，流量现在指向 CDN。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、 提供反馈，获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

此[存储库](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs)中的示例介绍了如何使用 .NET 启动默认流式处理终结点。

