---
title: 使用 Azure 媒体服务基于 URL 对远程文件进行编码并流式传输 - REST | Microsoft Docs
description: 按照本教程的步骤，使用 REST 通过 Azure 媒体服务基于 URL 对文件进行编码并流式传输内容。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/22/2019
ms.author: juliako
ms.openlocfilehash: 15b2408113d8bd19d2e988643442ac5e3b305237
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149198"
---
# <a name="tutorial-encode-a-remote-file-based-on-url-and-stream-the-video---rest"></a>教程：基于 URL 对远程文件进行编码并流式传输视频 - REST

使用 Azure 媒体服务可以将媒体文件编码为可在各种浏览器和设备上播放的格式。 例如，可能需要以 Apple 的 HLS 或 MPEG DASH 格式流式传输内容。 在流式传输之前，应该对高质量的数字媒体文件进行编码。 有关编码指南，请参阅[编码概念](encoding-concept.md)。

本教程介绍如何使用 REST 通过 Azure 媒体服务基于 URL 对文件进行编码并流式传输视频。 

![播放视频](./media/stream-files-tutorial-with-api/final-video.png)

本教程演示如何：    

> [!div class="checklist"]
> * 创建媒体服务帐户
> * 访问媒体服务 API
> * 下载 Postman 文件
> * 配置 Postman
> * 使用 Postman 发送请求
> * 测试流式 URL
> * 清理资源

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

- [创建媒体服务帐户](create-account-cli-how-to.md)。

    请务必记住用于资源组名称和媒体服务帐户名称的值

- 安装 [Postman](https://www.getpostman.com/) REST 客户端，以便执行一些 AMS REST 教程中所示的 REST API。 

    我们使用的是 **Postman**，但任何 REST 工具都适用。 其他替代工具包括：带有 REST 插件的 **Visual Studio Code** 或 **Telerik Fiddler**。 

## <a name="download-postman-files"></a>下载 Postman 文件

克隆包含 Postman 集合和环境文件的 GitHub 存储库。

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

## <a name="configure-postman"></a>配置 Postman

此部分配置 Postman。

### <a name="configure-the-environment"></a>配置环境 

1. 打开 **Postman**。
2. 在屏幕的右侧，选择“管理环境”选项。

    ![管理环境](./media/develop-with-postman/postman-import-env.png)
4. 从“管理环境”对话框中，单击“导入”。
2. 浏览到克隆 `https://github.com/Azure-Samples/media-services-v3-rest-postman.git` 时下载的 `Azure Media Service v3 Environment.postman_environment.json` 文件。
6. **Azure Media Service v3 Environment** 环境已添加。

    > [!Note]
    > 使用从上面的“访问媒体服务 API”部分获得的值更新访问权限变量。

7. 双击所选的文件，并输入通过执行[访问 API](#access-the-media-services-api) 步骤获得的值。
8. 关闭对话框。
9. 从下拉列表中选择“Azure Media Service v3 Environment”环境。

    ![选择环境](./media/develop-with-postman/choose-env.png)
   
### <a name="configure-the-collection"></a>配置集合

1. 单击“导入”导入该集合文件。
1. 浏览到克隆 `https://github.com/Azure-Samples/media-services-v3-rest-postman.git` 时下载的 `Media Services v3.postman_collection.json` 文件
3. 选择 **Media Services v3.postman_collection.json** 文件。

    ![导入文件](./media/develop-with-postman/postman-import-collection.png)

## <a name="send-requests-using-postman"></a>使用 Postman 发送请求

在本部分中，请发送与编码和创建 URL 相关的请求，以便能够流式传输文件。 具体说来，将发送以下请求：

1. 获取适用于服务主体身份验证的 Azure AD 令牌
2. 创建输出资产
3. 创建**转换**
4. 创建**作业**
5. 创建**流式处理定位符**
6. 列出**流式处理定位符**的路径

> [!Note]
>  本教程假定你使用唯一名称创建所有资源。  

### <a name="get-azure-ad-token"></a>获取 Azure AD 令牌 

1. 在 Postman 的左窗口中，选择“步骤 1: 获取 AAD 身份验证令牌”。
2. 然后，选择“获取适用于服务主体身份验证的 Azure AD 令牌”。
3. 按“发送”。

    将会发送以下 **POST** 操作。

    ```
    https://login.microsoftonline.com/:tenantId/oauth2/token
    ```

4. 响应会返回此令牌并将“AccessToken”环境变量设置为令牌值。 若要查看设置“AccessToken”的代码，请单击“测试”选项卡。 

    ![获取 AAD 令牌](./media/develop-with-postman/postman-get-aad-auth-token.png)

### <a name="create-an-output-asset"></a>创建输出资产

输出[资产](https://docs.microsoft.com/rest/api/media/assets)会存储作业编码的结果。 

1. 在 Postman 的左窗口中，选择“资产”。
2. 然后选择“创建或更新资产”。
3. 按“发送”。

    * 将会发送以下 **PUT** 操作：

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/assets/:assetName?api-version={{api-version}}
        ```
    * 该操作具有以下正文：

        ```json
        {
        "properties": {
            "description": "My Asset",
            "alternateId" : "some GUID"
         }
        }
        ```

### <a name="create-a-transform"></a>创建转换

对媒体服务中的内容进行编码或处理时，一种常见的模式是将编码设置设为脚本。 然后，需提交**作业**，将该脚本应用于视频。 为每个新视频提交新作业后，可将该脚本应用到库中的所有视频。 媒体服务中的脚本称为**转换**。 有关详细信息，请参阅[转换和作业](transform-concept.md)。 本教程中的示例定义有关将视频进行编码以将其流式传输到各种 iOS 和 Android 设备的脚本。 

创建新[转换](https://docs.microsoft.com/rest/api/media/transforms)实例时，需要指定希望生成的输出内容。 所需参数是 TransformOutput 对象。 每个 TransformOutput 包含一个预设。 预设介绍了视频和/或音频处理操作的分步说明，这些操作将用于生成所需的 TransformOutput。 本文中的示例使用名为 AdaptiveStreaming 的内置预设。 此预设将输入的视频编码为基于输入的分辨率和比特率自动生成的比特率阶梯（比特率 - 分辨率对），并通过与每个比特率 - 分辨率对相对应的 H.264 视频和 AAC 音频生成 ISO MP4 文件。 有关此预设的信息，请参阅[自动生成比特率阶梯](autogen-bitrate-ladder.md)。

可以使用内置 EncoderNamedPreset 或使用自定义预设。 

> [!Note]
> 在创建[转换](https://docs.microsoft.com/rest/api/media/transforms)时，首先应检查是否已存在一个使用 **Get** 方法的转换。 本教程假定你使用唯一名称创建该转换。

1. 在 Postman 的左窗口中，选择“编码和分析”。
2. 然后选择“创建转换”。
3. 按“发送”。

    * 将会发送以下 **PUT** 操作。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
        ```
    * 该操作具有以下正文：

        ```json
        {
            "properties": {
                "description": "Standard Transform using an Adaptive Streaming encoding preset from the library of built-in Standard Encoder presets",
                "outputs": [
                    {
                    "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                    "preset": {
                        "@odata.type": "#Microsoft.Media.BuiltInStandardEncoderPreset",
                        "presetName": "AdaptiveStreaming"
                    }
                    }
                ]
            }
        }
        ```

### <a name="create-a-job"></a>创建作业

[作业](https://docs.microsoft.com/rest/api/media/jobs)是针对媒体服务的实际请求，目的是将创建的**转换**应用到给定的输入视频或音频内容。 作业指定输入视频位置和输出位置等信息。

在此示例中，作业的输入基于 HTTPS URL（“https:\//nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/”）。

1. 在 Postman 的左窗口中，选择“编码和分析”。
2. 然后选择“创建或更新作业”。
3. 按“发送”。

    * 将会发送以下 **PUT** 操作。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName/jobs/:jobName?api-version={{api-version}}
        ```
    * 该操作具有以下正文：

        ```json
        {
        "properties": {
            "input": {
            "@odata.type": "#Microsoft.Media.JobInputHttp",
            "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
            "files": [
                    "Ignite-short.mp4"
                ]
            },
            "outputs": [
            {
                "@odata.type": "#Microsoft.Media.JobOutputAsset",
                "assetName": "testAsset1"
            }
            ]
        }
        }
        ```

此作业需要一些时间才能完成，完成时可发出通知。 若要查看作业的进度，建议使用事件网格。 事件网格旨在实现高可用性、一致性能和动态缩放。 使用事件网格，应用可以侦听和响应来自几乎所有 Azure 服务和自定义源的事件。 处理基于 HTTP 的反应事件非常简单，这有助于通过对事件的智能筛选和路由生成高效的解决方案。  请参阅[将事件路由到自定义 Web 终结点](job-state-events-cli-how-to.md)。

**作业**通常会经历以下状态：**已计划**、**已排队**、**正在处理**、**已完成**（最终状态）。 如果作业出错，则显示“错误”状态。 如果作业正处于取消过程中，则显示“正在取消”，完成时则显示“已取消”。

#### <a name="job-error-codes"></a>作业错误代码

请参阅[错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

### <a name="create-a-streaming-locator"></a>创建流式处理定位符

编码作业完成后，下一步是使输出**资产**中的视频可供客户端播放。 可通过两个步骤完成此操作：首先创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)，然后生成客户端可以使用的流式处理 URL。 

创建**流定位符**的过程称为发布。 默认情况下，除非配置可选的开始和结束时间，否则调用 API 后，**流式处理定位符**立即生效，并持续到其被删除为止。 

创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)时，需要指定所需的 **StreamingPolicyName**。 在此示例中，将流式传输明文（或未加密的）内容，因此使用预定义的明文流式传输策略 (**PredefinedStreamingPolicy.ClearStreamingOnly**)。

> [!IMPORTANT]
> 使用自定义的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 时，应为媒体服务帐户设计有限的一组此类策略，并在需要同样的加密选项和协议时重新将这些策略用于 StreamingLocators。 

媒体服务帐户具有对应于**流式处理策略**条目数的配额。 不应为每个**流式处理定位符**创建新的**流式处理策略**。

1. 在 Postman 的左窗口中，选择“流式处理策略”。
2. 然后选择“创建流式处理定位符”。
3. 按“发送”。

    * 将会发送以下 **PUT** 操作。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingPolicies/:streamingPolicyName?api-version={{api-version}}
        ```
    * 该操作具有以下正文：

        ```json
        {
            "properties":{
            "assetName": "{{assetName}}",
            "streamingPolicyName": "{{streamingPolicyName}}"
            }
        }
        ```

### <a name="list-paths-and-build-streaming-urls"></a>列出路径和生成流式处理 URL

#### <a name="list-paths"></a>列出路径

创建[流式处理定位符](https://docs.microsoft.com/rest/api/media/streaminglocators)后，即可获取流式处理 URL

1. 在 Postman 的左窗口中，选择“流式处理策略”。
2. 然后，选择“列出路径”。
3. 按“发送”。

    * 将会发送以下 **POST** 操作。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName/listPaths?api-version={{api-version}}
        ```
        
    * 该操作没有正文：
        
4. 记下要用于流式处理的某个路径，以便在下一部分使用。 在这种情况下，返回以下路径：
    
    ```
    "streamingPaths": [
        {
            "streamingProtocol": "Hls",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)"
            ]
        },
        {
            "streamingProtocol": "Dash",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=mpd-time-csf)"
            ]
        },
        {
            "streamingProtocol": "SmoothStreaming",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest"
            ]
        }
    ]
    ```

#### <a name="build-the-streaming-urls"></a>生成流式处理 URL

在此部分，请生成 HLS 流式处理 URL。 URL 包含以下值：

1. 发送数据时采用的协议。 在此示例中为“https”。

    > [!NOTE]
    > 如果播放器在 Https 站点上进行托管，请确保将 URL 更新为“https”。

2. StreamingEndpoint 的主机名。 在本例中，该名称为“amsaccount-usw22.streaming.media.azure.net”。

    若要获取主机名，可以使用以下 GET 操作：
    
    ```
    https://management.azure.com/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingEndpoints/default?api-version={{api-version}}
    ```
    
3. 在前面的（列出路径）部分中获得的路径。  

因此，生成了以下 HLS URL

```
https://amsaccount-usw22.streaming.media.azure.net/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)
```

## <a name="test-the-streaming-url"></a>测试流式 URL


> [!NOTE]
> 确保要从中进行流式传输的**流式处理终结点**正在运行。

本文使用 Azure Media Player 测试流式传输。 

1. 打开 Web 浏览器并导航到 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在“URL:”框中，粘贴生成的 URL。 
3. 按“更新播放器”。

Azure Media Player 可用于测试，但不可在生产环境中使用。 

## <a name="clean-up-resources-in-your-media-services-account"></a>清理媒体服务帐户中的资源

通常情况下，除了打算重复使用的对象，应清理所有内容（通常会重复使用**转换**并保留**流式处理定位符**等）。 如果希望帐户在试验后保持干净状态，则应删除不打算重复使用的资源。  

若要删除资源，请在要删除的任意资源下选择“删除...”操作。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任何一个资源（包括为本教程创建的媒体服务和存储帐户），请删除之前创建的资源组。  

执行以下 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

现已介绍如何上传、编码和流式传输视频，请参阅以下文章： 

> [!div class="nextstepaction"]
> [分析视频](analyze-videos-tutorial-with-api.md)
