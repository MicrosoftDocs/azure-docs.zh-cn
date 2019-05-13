---
title: 使用 .NET 通过 Azure 媒体服务 v3 进行上传、编码和流式传输 | Microsoft Docs
description: 遵循本教程中的步骤，使用 .NET 通过媒体服务 v3 上传文件、编码视频和流式传输内容。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/22/2019
ms.author: juliako
ms.openlocfilehash: 66ee2c110edfdbd0e33c69d45dee8040654d421a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149148"
---
# <a name="tutorial-upload-encode-and-stream-videos-using-net"></a>教程：使用 .NET 上传、编码和流式传输视频

使用 Azure 媒体服务可以将媒体文件编码为可在各种浏览器和设备上播放的格式。 例如，可能需要以 Apple 的 HLS 或 MPEG DASH 格式流式传输内容。 在流式传输之前，应该对高质量的数字媒体文件进行编码。 有关编码指南，请参阅[编码概念](encoding-concept.md)。 本教程上传本地视频文件并对上传的文件进行编码。 还可以对可通过 HTTPS URL 访问的内容进行编码。 有关详细信息，请参阅[从 HTTP URL 创建作业输入](job-input-from-http-how-to.md)。

![播放视频](./media/stream-files-tutorial-with-api/final-video.png)

本教程演示如何：    

> [!div class="checklist"]
> * 下载本主题中所述的示例应用
> * 检查用于上传、编码和流式传输的代码
> * 运行应用
> * 测试流式 URL
> * 清理资源

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

- 如果没有安装 Visual Studio，可下载 [Visual Studio Community 2017](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15)。
- [创建媒体服务帐户](create-account-cli-how-to.md)。<br/>请务必记住用于资源组名称和媒体服务帐户名称的值。
- 遵循[使用 Azure CLI 访问 Azure 媒体服务 API](access-api-cli-how-to.md) 中的步骤并保存凭据。 需要使用这些凭据来访问 API。

## <a name="download-and-configure-the-sample"></a>下载并配置示例

使用以下命令将包含流式处理 .NET 示例的 GitHub 存储库克隆到计算机：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

该示例位于 [UploadEncodeAndStreamFiles](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/UploadEncodeAndStreamFiles) 文件夹。

打开下载的项目中的 [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/appsettings.json)。 将值替换为在[访问 API](access-api-cli-how-to.md) 中获取的凭据。

## <a name="examine-the-code-that-uploads-encodes-and-streams"></a>检查用于上传、编码和流式传输的代码

本节讨论 UploadEncodeAndStreamFiles 项目的 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs) 文件中定义的函数。

该示例执行以下操作：

1. 创建一个新**转换**（首先检查指定的转换是否存在）。 
2. 创建一个输出**资产**，用作编码**作业**的输出。
3. 创建一个输入**资产**并将指定的本地视频文件上传到其中。 该资产用作作业的输入。 
4. 使用创建的输入和输出提交编码作业。
5. 检查作业的状态。
6. 创建**流定位符**。
7. 生成流式处理 URL。

### <a name="a-idstartusingdotnet-start-using-media-services-apis-with-net-sdk"></a><a id="start_using_dotnet" />开始结合使用媒体服务 API 与 .NET SDK

若要开始将媒体服务 API 与 .NET 结合使用，需要创建 AzureMediaServicesClient 对象。 若要创建对象，需要提供客户端所需凭据以使用 Azure AD 连接到 Azure。 在本文开头克隆的代码中，**GetCredentialsAsync** 函数根据本地配置文件中提供的凭据创建 ServiceClientCredentials 对象。 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>创建输入资产并将本地文件上传到该资产 

CreateInputAsset 函数创建新的输入[资产](https://docs.microsoft.com/rest/api/media/assets)并将指定的本地视频文件上传到该资产。 此**资产**用作编码作业的输入。 在媒体服务 v3 中，**作业**输入可以是**资产**，也可以是可通过 HTTPS URL 使用媒体服务帐户访问的内容。 如果想要了解如何从 HTTPS URL 进行编码，请参阅[此](job-input-from-http-how-to.md)文章。  

在媒体服务 v3 中，使用 Azure 存储 API 上传文件。 以下 .NET 片段显示如何上传。

以下函数执行以下操作：

* 创建**资产** 
* 获取资产的[存储中容器](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-dotnet?tabs=windows#upload-blobs-to-the-container)的可写 [SAS URL](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
* 使用 SAS URL 将文件上传到存储中的容器中

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-a-job"></a>创建输出资产用于存储作业结果 

输出[资产](https://docs.microsoft.com/rest/api/media/assets)会存储作业编码的结果。 项目定义 DownloadResults 函数，该函数将结果从此输出资产中下载到**输出**文件夹中，便于用户查看获取的内容。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>创建转换和一个对上传的文件进行编码的作业

对媒体服务中的内容进行编码或处理时，一种常见的模式是将编码设置设为脚本。 然后，需提交**作业**，将该脚本应用于视频。 为每个新视频提交新作业后，可将该脚本应用到库中的所有视频。 媒体服务中的脚本称为**转换**。 有关详细信息，请参阅[转换和作业](transform-concept.md)。 本教程中的示例定义有关将视频进行编码以将其流式传输到各种 iOS 和 Android 设备的脚本。 

#### <a name="transform"></a>转换

创建新[转换](https://docs.microsoft.com/rest/api/media/transforms)实例时，需要指定希望生成的输出内容。 所需参数是 TransformOutput 对象，如以下代码所示。 每个 TransformOutput 包含一个预设。 预设介绍了视频和/或音频处理操作的分步说明，这些操作将用于生成所需的 TransformOutput。 本文中的示例使用名为 AdaptiveStreaming 的内置预设。 此预设将输入的视频编码为基于输入的分辨率和比特率自动生成的比特率阶梯（比特率 - 分辨率对），并通过与每个比特率 - 分辨率对相对应的 H.264 视频和 AAC 音频生成 ISO MP4 文件。 有关此预设的信息，请参阅[自动生成比特率阶梯](autogen-bitrate-ladder.md)。

可以使用内置 EncoderNamedPreset 或使用自定义预设。 有关详细信息，请参阅[如何自定义编码器预设](customize-encoder-presets-how-to.md)。

在创建时[转换](https://docs.microsoft.com/rest/api/media/transforms)，首先应检查是否其中一个已存在使用**获取**方法，如下面的代码中所示。  在 Media Services v3**获取**实体上的方法返回**null**如果实体不存在 （不区分大小写的名称检查）。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>作业

如上所述，[转换](https://docs.microsoft.com/rest/api/media/transforms)对象为脚本，[作业则是](https://docs.microsoft.com/rest/api/media/jobs)对媒体服务的实际请求，请求将转换应用到给定输入视频或音频内容。 作业指定输入视频位置和输出位置等信息。

在此示例中，已从本地计算机上传输入视频。 如果想要了解如何从 HTTPS URL 进行编码，请参阅[此](job-input-from-http-how-to.md)文章。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>等待作业完成

此作业需要一些时间才能完成，完成时可发出通知。 以下代码示例显示如何轮询服务以获取[作业](https://docs.microsoft.com/rest/api/media/jobs)状态。 对于生产应用程序，由于可能出现延迟，并不建议将轮询作为最佳做法。 如果在帐户上过度使用轮询，轮询会受到限制。 开发者应改用事件网格。

事件网格旨在实现高可用性、一致性能和动态缩放。 使用事件网格，应用可以侦听和响应来自几乎所有 Azure 服务和自定义源的事件。 处理基于 HTTP 的反应事件非常简单，这有助于通过对事件的智能筛选和路由生成高效的解决方案。  请参阅[将事件路由到自定义 Web 终结点](job-state-events-cli-how-to.md)。

**作业**通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消”。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#WaitForJobToFinish)]

### <a name="job-error-codes"></a>作业错误代码

请参阅[错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

### <a name="get-a-streaming-locator"></a>获取流定位符

编码完成后，下一步是使输出资产中的视频可供客户端播放。 可通过两个步骤完成此操作：首先创建[流定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)，然后生成客户端可以使用的流 URL。 

创建**流定位符**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，**流定位符**立即生效，并持续到被删除为止。 

创建 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 时，需要指定所需的 StreamingPolicyName。 在此示例中将流式传输明文（或未加密的内容），因此使用预定义的明文流式传输策略 (**PredefinedStreamingPolicy.ClearStreamingOnly**)。

> [!IMPORTANT]
> 使用自定义的[流策略](https://docs.microsoft.com/rest/api/media/streamingpolicies)时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的加密选项和协议时重新将这些策略用于 StreamingLocators。 媒体服务帐户具有对应于流式处理策略条目数的配额。 不应为每个流式处理定位符创建新的流式处理策略。

以下代码假定使用唯一的 locatorName 调用该函数。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateStreamingLocator)]

虽然本主题的示例讨论的是流式传输，但可以使用相同的调用创建一个流定位符，通过渐进式下载的方式来提供视频。

### <a name="get-streaming-urls"></a>获取流式 URL

创建[流定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)后，可以获取流 URL，如 **GetStreamingURLs** 中所示。 若要生成 URL，需要连接[流式处理终结点](https://docs.microsoft.com/rest/api/media/streamingendpoints)的主机名和**流定位符**路径。 此示例使用默认的**流式处理终结点**。 首次创建媒体服务帐户时，默认的**流式处理终结点**处于停止状态，因此需要调用 **Start**。

> [!NOTE]
> 在此方法中，需要指定在创建输出资产的**流定位符**时所用的 locatorName。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#GetStreamingURLs)]

### <a name="clean-up-resources-in-your-media-services-account"></a>清理媒体服务帐户中的资源

通常情况下，除了打算重复使用的对象，用户应清理所有内容（通常将重复使用转换并保留 StreamingLocators 等）。 如果希望帐户在试验后保持干净状态，则应删除不打算重复使用的资源。  例如，以下代码可删除作业。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>运行示例应用

1. 按 Ctrl+F5 运行 EncodeAndStreamFiles 应用程序。
2. 从控制台复制其中一个流式 URL。

此示例显示的 URL 可用于使用不同协议来播放视频：

![输出](./media/stream-files-tutorial-with-api/output.png)

## <a name="test-the-streaming-url"></a>测试流式 URL

本文使用 Azure Media Player 测试流式传输。 

> [!NOTE]
> 如果播放器在 Https 站点上进行托管，请确保将 URL 更新为“https”。

1. 打开 Web 浏览器并导航到 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在“URL:”框中，粘贴运行应用程序时获取的某个流式处理 URL 值。 
3. 按“更新播放器”。

Azure Media Player 可用于测试，但不可在生产环境中使用。 

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任何一个资源（包括为本教程创建的媒体服务和存储帐户），请删除之前创建的资源组。

执行以下 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>多线程处理

Azure 媒体服务 v3 SDK 不是线程安全的。 开发多线程应用程序时，应在每个线程上生成并使用一个新的 AzureMediaServicesClient 对象。

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

现已介绍如何上传、编码和流式传输视频，请参阅以下文章： 

> [!div class="nextstepaction"]
> [分析视频](analyze-videos-tutorial-with-api.md)
