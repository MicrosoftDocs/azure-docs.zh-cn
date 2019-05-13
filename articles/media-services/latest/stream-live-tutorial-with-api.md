---
title: 使用 .NET 通过 Azure 媒体服务 v3 进行实时流式传输 | Microsoft Docs
description: 本教程详述了使用 .NET Core 通过媒体服务 v3 进行实时流式传输的步骤。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/21/2019
ms.author: juliako
ms.openlocfilehash: e4f32e14e8c1035055bd8a37bb453764984fbe4d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149137"
---
# <a name="tutorial-stream-live-with-media-services-v3-using-net"></a>教程：使用 .NET 通过媒体服务 v3 进行实时流式传输

在 Azure 媒体服务中，[实时事件](https://docs.microsoft.com/rest/api/media/liveevents)负责处理实时传送视频流内容。 实时事件提供输入终结点（引入 URL），然后由你将该终结点提供给实时编码器。 实时事件从实时编码器接收实时输入流，并通过一个或多个[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints)使其可用于流式处理。 实时事件还提供可用于预览的预览终结点（预览 URL），并在进一步处理和传递流之前对流进行验证。 本教程演示如何使用 .NET Core 创建**直通**类型的直播事件。 

> [!NOTE]
> 确保在继续操作之前查看[使用媒体服务 v3 的实时传送视频流](live-streaming-overview.md)。 

本教程介绍如何：    

> [!div class="checklist"]
> * 下载本主题中所述的示例应用
> * 检查执行实时传送视频流的代码
> * 使用 [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) 在 https://ampdemo.azureedge.net 观看事件
> * 清理资源

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

以下是完成本教程所需具备的条件。

- 安装 Visual Studio Code 或 Visual Studio。
- [创建媒体服务帐户](create-account-cli-how-to.md)。<br/>请务必记住用于资源组名称和媒体服务帐户名称的值。
- 遵循[使用 Azure CLI 访问 Azure 媒体服务 API](access-api-cli-how-to.md) 中的步骤并保存凭据。 需要使用这些凭据来访问 API。
- 一个用于广播事件的相机或设备（例如便携式计算机）。
- 一个本地实时编码器，用于将信号从相机转换为发送至媒体服务实时传送视频流服务的流。 流必须为 **RTMP** 或“平滑流式处理”格式。

## <a name="download-and-configure-the-sample"></a>下载并配置示例

使用以下命令将包含流式处理 .NET 示例的 GitHub 存储库克隆到计算机：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```

实时传送视频流示例位于 [Live](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/tree/master/NETCore/Live/MediaV3LiveApp) 文件夹中。

打开下载的项目中的 [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/appsettings.json)。 将值替换为在[访问 API](access-api-cli-how-to.md) 中获取的凭据。

> [!IMPORTANT]
> 此示例为每个资源使用唯一的后缀。 如果取消调试操作或者中途终止应用，则帐户中会有多个实时事件。 <br/>请务必停止正在运行的实时事件， 否则需**付费**！

## <a name="examine-the-code-that-performs-live-streaming"></a>检查执行实时传送视频流的代码

此部分讨论 MediaV3LiveApp 项目的 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs) 文件中定义的函数。

此示例为每个资源创建唯一的后缀，因此即使在没有清理的情况下运行示例多次，也不会有名称冲突。

> [!IMPORTANT]
> 此示例为每个资源使用唯一的后缀。 如果取消调试操作或者中途终止应用，则帐户中会有多个实时事件。 <br/>
> 请务必停止正在运行的实时事件， 否则需**付费**！
 
### <a name="start-using-media-services-apis-with-net-sdk"></a>开始结合使用媒体服务 API 与 .NET SDK

若要开始将媒体服务 API 与 .NET 结合使用，需要创建 AzureMediaServicesClient 对象。 若要创建对象，需要提供客户端所需凭据以使用 Azure AD 连接到 Azure。 在本文开头克隆的代码中，**GetCredentialsAsync** 函数根据本地配置文件中提供的凭据创建 ServiceClientCredentials 对象。 

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateMediaServicesClient)]

### <a name="create-a-live-event"></a>创建直播事件

本部分介绍如何创建**直通**类型的实时事件（LiveEventEncodingType 设置为 None）。 有关实时事件的可用类型的详细信息，请参阅[实时事件类型](live-events-outputs-concept.md#live-event-types)。 
 
可能需要在创建实时事件时指定的一些事项包括：

* 媒体服务位置 
* 实时事件的流式处理协议（目前支持 RTMP 和平滑流式处理协议）。<br/>运行实时事件或其关联的实时输出时，无法更改协议选项。 如果需要其他协议，应当为每个流式处理协议创建单独的实时事件。  
* 对引入和预览的 IP 限制。 可定义允许向该实时事件引入视频的 IP 地址。 允许的 IP 地址可以指定为单个 IP 地址（例如“10.0.0.1”）、使用一个 IP 地址和 CIDR 子网掩码的 IP 范围（例如“10.0.0.1/22”）或使用一个 IP 地址和点分十进制子网掩码的 IP 范围（例如“10.0.0.1(255.255.252.0)”）。<br/>如果未指定 IP 地址并且没有规则定义，则不会允许任何 IP 地址。 若要允许任何 IP 地址，请创建一个规则并设置 0.0.0.0/0。<br/>IP 地址必须采用以下格式之一：具有 4 个数字、CIDR 地址范围的 IpV4 地址。
* 创建事件时，可以将其启动方式指定为自动启动。 <br/>如果将 autostart 设置为 true，则实时事件会在创建后启动。 这意味着，只要实时事件开始运行，就会开始计费。 必须显式对实时事件资源调用 Stop 才能停止进一步计费。 有关详细信息，请参阅[实时事件状态和计费](live-event-states-billing.md)。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateLiveEvent)]

### <a name="get-ingest-urls"></a>获取引入 URL

创建实时事件后，可以获得要提供给实时编码器的引入 URL。 编码器将使用这些 URL 来输入实时流。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#GetIngestURL)]

### <a name="get-the-preview-url"></a>获取预览 URL

使用 previewEndpoint 预览来自编码器的输入并验证其是否已确实收到。

> [!IMPORTANT]
> 确保视频流向预览 URL，然后再继续操作！

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#GetPreviewURLs)]

### <a name="create-and-manage-live-events-and-live-outputs"></a>创建和管理实时事件与实时输出

将流传输到实时事件后，可以通过创建资产、实时输出和流定位符来启动流式传输事件。 这会存档流，并使观看者可通过流式处理终结点使用该流。 

#### <a name="create-an-asset"></a>创建资产

创建供实时输出使用的资产。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateAsset)]

#### <a name="create-a-live-output"></a>创建实时输出

实时输出在创建时启动，在删除后停止。 删除实时输出不会删除基础资产和该资产中的内容。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateLiveOutput)]

#### <a name="create-a-streaming-locator"></a>创建流定位符

> [!NOTE]
> 创建媒体服务帐户后，会将一个处于“已停止”状态的**默认**流式处理终结点添加到帐户。 若要开始流式传输内容并利用[动态打包](dynamic-packaging-overview.md)和动态加密，要从中流式传输内容的流式处理终结点必须处于“正在运行”状态。 

如果已使用流定位符发布了实时输出资产，则实时事件（长达 DVR 窗口长度）将继续可见，直到流定位符过期或被删除（以先发生为准）。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateStreamingLocator)]

```csharp

// Get the url to stream the output
ListPathsResponse paths = await client.StreamingLocators.ListPathsAsync(resourceGroupName, accountName, locatorName);

foreach (StreamingPath path in paths.StreamingPaths)
{
    UriBuilder uriBuilder = new UriBuilder();
    uriBuilder.Scheme = "https";
    uriBuilder.Host = streamingEndpoint.HostName;

    uriBuilder.Path = path.Paths[0];
    // Get the URL from the uriBuilder: uriBuilder.ToString()
}
```

### <a name="cleaning-up-resources-in-your-media-services-account"></a>清理媒体服务帐户中的资源

如果已完成流式处理事件，并想要清理先前设置的资源，请遵循以下过程。

* 停止从编码器推送流。
* 停止实时事件。 实时事件在停止后，不会产生任何费用。 当需要重新启动它时，它会采用相同的引入 URL，因此，无需重新配置编码器。
* 除非你想要继续以点播流形式提供实时事件的存档，否则可以停止流式处理终结点。 如果实时事件处于停止状态，则不会产生任何费用。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CleanupLiveEventAndOutput)]

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CleanupLocatorAssetAndStreamingEndpoint)]

以下代码演示如何清理所有实时事件的帐户：

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CleanupAccount)]   

## <a name="watch-the-event"></a>观看事件

若要观看事件，请复制流式传输 URL（在运行“创建流定位符”中所述的代码时获得），然后使用所选的播放器。 可以使用 [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) 来测试 https://ampdemo.azureedge.net 中的流。 

实时事件在停止后会自动转换为点播内容。 即使你停止并删除了事件，只要没有删除资产，用户也能够按需将已存档内容作为视频进行流式传输。 如果资产被某个事件使用，则无法将其删除，必须先删除该事件。 

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任何一个资源（包括为本教程创建的媒体服务和存储帐户），请删除之前创建的资源组。

执行以下 CLI 命令：

```azurecli-interactive
az group delete --name amsResourceGroup
```

> [!IMPORTANT]
> 让实时事件保持运行会产生费用。 请注意，如果项目/节目崩溃或因某种原因而关闭，可能会导致实时事件保持运行状态，从而产生费用。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

[对文件进行流式处理](stream-files-tutorial-with-api.md)
 
