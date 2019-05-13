---
title: 使用 Azure 媒体服务和 Azure CLI 流式传输视频文件 | Microsoft Docs
description: 按照此快速入门的步骤，创建新的 Azure 媒体服务帐户、编码文件并将文件流式传输到 Azure Media Player。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
keywords: Azure 媒体服务, 流式传输
ms.service: media-services
ms.workload: media
ms.topic: quickstart
ms.custom: ''
ms.date: 02/19/2019
ms.author: juliako
ms.openlocfilehash: bd818135f289c430416909d16c8568208936a9f8
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205406"
---
# <a name="quickstart-stream-video-files---cli"></a>快速入门：流式处理视频文件 - CLI

本快速入门介绍如何使用 Azure 媒体服务和 Azure CLI 在各种浏览器与设备上轻松编码和流式传输视频。 可以使用 HTTPS、SAS URL 或位于 Azure Blob 存储中的文件路径来指定输入内容。

本文中的示例对可通过 HTTPS URL 访问的内容进行编码。 媒体服务 v3 目前不支持基于 HTTPS URL 的块式传输编码。

完成本快速入门后，即可流式传输视频。  

![播放视频](./media/stream-files-dotnet-quickstart/final-video.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-media-services-account"></a>创建媒体服务帐户

在加密、编码、分析、管理和流式传输 Azure 中的媒体内容之前，需要创建一个媒体服务帐户。 该帐户必须与一个或多个存储帐户相关联。

媒体服务帐户和所有关联的存储帐户必须位于同一 Azure 订阅中。 我们建议在媒体服务帐户所在的同一位置使用存储帐户，以限制延迟和数据传出费用。

### <a name="create-a-resource-group"></a>创建资源组

```azurecli
az group create -n amsResourceGroup -l westus2
```

### <a name="create-an-azure-storage-account"></a>创建 Azure 存储帐户

本示例创建一个常规用途 v2 标准 LRS 帐户。

若要通过存储帐户进行试验，请使用 `--sku Standard_LRS`。 选取用于生产的 SKU 时，请考虑使用 `--sku Standard_RAGRS`，以便通过异地复制来确保业务连续性。 有关详细信息，请参阅[存储帐户](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest)。
 
```azurecli
az storage account create -n amsstorageaccount --kind StorageV2 --sku Standard_LRS -l westus2 -g amsResourceGroup
```

### <a name="create-an-azure-media-services-account"></a>创建 Azure 媒体服务帐户

```azurecli
az ams account create --n amsaccount -g amsResourceGroup --storage-account amsstorageaccount -l westus2
```

响应如下所示：

```
{
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount",
  "location": "West US 2",
  "mediaServiceId": "8b569c2e-d648-4fcb-9035-c7fcc3aa7ddf",
  "name": "amsaccount",
  "resourceGroup": "amsResourceGroupTest",
  "storageAccounts": [
    {
      "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Storage/storageAccounts/amsstorageaccount",
      "resourceGroup": "amsResourceGroupTest",
      "type": "Primary"
    }
  ],
  "tags": null,
  "type": "Microsoft.Media/mediaservices"
}
```

## <a name="start-the-streaming-endpoint"></a>启动流式处理终结点

以下 Azure CLI 命令将启动默认的**流式处理终结点**。

```azurecli
az ams streaming-endpoint start  -n default -a amsaccount -g amsResourceGroup
```

响应如下所示：

```
az ams streaming-endpoint start  -n default -a amsaccount -g amsResourceGroup
{
  "accessControl": null,
  "availabilitySetName": null,
  "cdnEnabled": true,
  "cdnProfile": "AzureMediaStreamingPlatformCdnProfile-StandardVerizon",
  "cdnProvider": "StandardVerizon",
  "created": "2019-02-06T21:58:03.604954+00:00",
  "crossSiteAccessPolicies": null,
  "customHostNames": [],
  "description": "",
  "freeTrialEndTime": "2019-02-21T22:05:31.277936+00:00",
  "hostName": "amsaccount-usw22.streaming.media.azure.net",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingendpoints/default",
  "lastModified": "2019-02-06T21:58:03.604954+00:00",
  "location": "West US 2",
  "maxCacheAge": null,
  "name": "default",
  "provisioningState": "Succeeded",
  "resourceGroup": "amsResourceGroup",
  "resourceState": "Running",
  "scaleUnits": 0,
  "tags": {},
  "type": "Microsoft.Media/mediaservices/streamingEndpoints"
}
```

如果流式处理终结点已运行，则会显示以下消息：

```
(InvalidOperation) The server cannot execute the operation in its current state.
```

## <a name="create-a-transform-for-adaptive-bitrate-encoding"></a>创建适用于自适应比特率编码的转换

创建一个**转换**，以便配置对视频进行编码或分析的常见任务。 本示例执行自适应比特率编码。 然后，在创建的转换下提交某个作业。 该作业是向媒体服务发出的请求，目的是将转换应用到给定的视频或音频内容输入。

```azurecli
az ams transform create --name testEncodingTransform --preset AdaptiveStreaming --description 'a simple Transform for Adaptive Bitrate Encoding' -g amsResourceGroup -a amsaccount
```

响应如下所示：

```
{
  "created": "2019-02-15T00:11:18.506019+00:00",
  "description": "a simple Transform for Adaptive Bitrate Encoding",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform",
  "lastModified": "2019-02-15T00:11:18.506019+00:00",
  "name": "testEncodingTransform",
  "outputs": [
    {
      "onError": "StopProcessingJob",
      "preset": {
        "odatatype": "#Microsoft.Media.BuiltInStandardEncoderPreset",
        "presetName": "AdaptiveStreaming"
      },
      "relativePriority": "Normal"
    }
  ],
  "resourceGroup": "amsResourceGroup",
  "type": "Microsoft.Media/mediaservices/transforms"
}
```

## <a name="create-an-output-asset"></a>创建输出资产

创建一个输出**资产**用作编码作业的输出。

```azurecli
az ams asset create -n testOutputAssetName -a amsaccount -g amsResourceGroup
```

响应如下所示：

```
{
  "alternateId": null,
  "assetId": "96427438-bbce-4a74-ba91-e38179b72f36",
  "container": null,
  "created": "2019-02-14T23:58:19.127000+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/assets/testOutputAssetName",
  "lastModified": "2019-02-14T23:58:19.127000+00:00",
  "name": "testOutputAssetName",
  "resourceGroup": "amsResourceGroup",
  "storageAccountName": "amsstorageaccount",
  "storageEncryptionFormat": "None",
  "type": "Microsoft.Media/mediaservices/assets"
}
```

## <a name="start-a-job-by-using-https-input"></a>使用 HTTPS 输入启动作业

提交用于处理视频的作业时，必须告知媒体服务查找输入视频的位置。 一个选项是指定 HTTPS URL 作为作业输入（如本示例所示）。

运行 `az ams job start` 时，可以在作业的输出上设置一个标签。 然后可以使用该标签来标识该输出资产的用途。

- 如果为标签赋值，请将“--output-assets”设置为“assetname=label”。
- 如果不为标签赋值，请将“--output-assets”设置为“assetname=”。

  请注意，我们将“=”添加到了 `output-assets`。

```azurecli
az ams job start --name testJob001 --transform-name testEncodingTransform --base-uri 'https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/' --files 'Ignite-short.mp4' --output-assets testOutputAssetName= -a amsaccount -g amsResourceGroup 
```

响应如下所示：

```
{
  "correlationData": {},
  "created": "2019-02-15T05:08:26.266104+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform/jobs/testJob001",
  "input": {
    "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
    "files": [
      "Ignite-short.mp4"
    ],
    "label": null,
    "odatatype": "#Microsoft.Media.JobInputHttp"
  },
  "lastModified": "2019-02-15T05:08:26.266104+00:00",
  "name": "testJob001",
  "outputs": [
    {
      "assetName": "testOutputAssetName",
      "error": null,
      "label": "",
      "odatatype": "#Microsoft.Media.JobOutputAsset",
      "progress": 0,
      "state": "Queued"
    }
  ],
  "priority": "Normal",
  "resourceGroup": "amsResourceGroup",
  "state": "Queued",
  "type": "Microsoft.Media/mediaservices/transforms/jobs"
}
```

### <a name="check-status"></a>查看状态

在 5 分钟内检查作业的状态。 该状态应该是“Finished”。 如果作业未完成，请在几分钟后重新检查。 如果作业已完成，请转到下一步骤并创建**流式处理定位符**。

```azurecli
az ams job show -a amsaccount -g amsResourceGroup -t testEncodingTransform -n testJob001
```

## <a name="create-a-streaming-locator-and-get-a-path"></a>创建流式处理定位符并获取路径

编码完成后，下一步是使输出资产中的视频可供客户端播放。 为此，请先创建一个流式处理定位符。 然后生成客户端可以使用的流 URL。

### <a name="create-a-streaming-locator"></a>创建流式处理定位符

```azurecli
az ams streaming-locator create -n testStreamingLocator --asset-name testOutputAssetName --streaming-policy-name Predefined_ClearStreamingOnly  -g amsResourceGroup -a amsaccount 
```

响应如下所示：

```
{
  "alternativeMediaId": null,
  "assetName": "output-3b6d7b1dffe9419fa104b952f7f6ab76",
  "contentKeys": [],
  "created": "2019-02-15T04:35:46.270750+00:00",
  "defaultContentKeyPolicyName": null,
  "endTime": "9999-12-31T23:59:59.999999+00:00",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingLocators/testStreamingLocator",
  "name": "testStreamingLocator",
  "resourceGroup": "amsResourceGroup",
  "startTime": null,
  "streamingLocatorId": "e01b2be1-5ea4-42ca-ae5d-7fe704a5962f",
  "streamingPolicyName": "Predefined_ClearStreamingOnly",
  "type": "Microsoft.Media/mediaservices/streamingLocators"
}
```

### <a name="get-streaming-locator-paths"></a>获取流式处理定位符路径

```azurecli
az ams streaming-locator get-paths -a amsaccount -g amsResourceGroup -n testStreamingLocator
```

响应如下所示：

```
{
  "downloadPaths": [],
  "streamingPaths": [
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)"
      ],
      "streamingProtocol": "Hls"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=mpd-time-csf)"
      ],
      "streamingProtocol": "Dash"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest"
      ],
      "streamingProtocol": "SmoothStreaming"
    }
  ]
}
```

复制 HTTP 实时传送视频流 (HLS) 路径。 在本例中，该值为 `/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)`。

## <a name="build-the-url"></a>生成 URL 

### <a name="get-the-streaming-endpoint-host-name"></a>获取流式处理终结点主机名

```azurecli
az ams streaming-endpoint list -a amsaccount -g amsResourceGroup -n default
```
复制 `hostName` 值。 在本例中，该值为 `amsaccount-usw22.streaming.media.azure.net`。

### <a name="assemble-the-url"></a>汇编 URL

“https:// ” + &lt;hostName 值&gt; + &lt;Hls 路径值&gt;

下面是一个示例：

`https://amsaccount-usw22.streaming.media.azure.net/7f19e783-927b-4e0a-a1c0-8a140c49856c/ignite.ism/manifest(format=m3u8-aapl)`

## <a name="test-playback-by-using-azure-media-player"></a>使用 Azure Media Player 测试播放

> [!NOTE]
> 如果播放器托管在 HTTPS 站点上，请确保在 URL 的开头添加“https”。

1. 打开 Web 浏览器并转到 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在“URL”框中，粘贴在上一部分生成的 URL。 可以粘贴 HLS、Dash 或 Smooth 格式的 URL。 Azure Media Player 将自动使用相应的流协议在设备上播放。
3. 选择“更新播放器”。

>[!NOTE]
>Azure Media Player 可用于测试，但不可在生产环境中使用。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组中的任一资源（包括本快速入门中创建的媒体服务和存储帐户），请删除该资源组。

运行以下 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="see-also"></a>另请参阅

请参阅[作业错误代码](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode)。

## <a name="next-steps"></a>后续步骤

[媒体服务概述](media-services-overview.md)
