---
title: 编码使用媒体服务 v3.NET 的 Azure 自定义转换 |Microsoft Docs
description: 本主题演示如何使用 Azure 媒体服务 v3 进行编码使用.NET 进行自定义转换。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: seodec18
ms.date: 05/03/2019
ms.author: juliako
ms.openlocfilehash: 2167a74dc81bdbb2562211cf5c0195a755941d9d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148338"
---
# <a name="how-to-encode-with-a-custom-transform---net"></a>如何使用自定义转换的.NET 进行编码

使用 Azure 媒体服务编码时，你可以快速开始使用基于行业最佳实践，如中所示的建议内置预设之一[文件进行流式处理](stream-files-tutorial-with-api.md)教程。 此外可以构建自定义预设，以满足特定的方案或设备要求。

## <a name="considerations"></a>注意事项

时创建自定义预设，应该注意以下事项：

* 高度和宽度 AVC 内容上的所有值必须都是 4 的倍数。
* 在 Azure 媒体服务 v3 所有编码比特率是比特 / 秒。 这是不同于与我们使用千比特/秒为单位的 v2 Api 的预设。 例如，如果在 v2 中的比特率 （千比特/秒） 已指定为 128，v3 中它将设置为 128000 （比特/秒）。

## <a name="prerequisites"></a>必备组件 

[创建媒体服务帐户](create-account-cli-how-to.md)

## <a name="download-the-sample"></a>下载示例

使用以下命令将包含完整 .NET Core 示例的 GitHub 存储库克隆到计算机：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```
 
自定义预设示例位于 [EncodeCustomTransform](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/EncodeCustomTransform/) 文件夹中。

## <a name="create-a-transform-with-a-custom-preset"></a>使用自定义预设创建转换 

创建新的[转换](https://docs.microsoft.com/rest/api/media/transforms)时，需要指定希望生成的输出内容。 所需参数是 [TransformOutput](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#transformoutput) 对象，如以下代码所示。 每个 TransformOutput 包含一个预设。 **预设**描述了要用来生成所需的视频和/或音频处理操作的分步说明**TransformOutput**。 以下 **TransformOutput** 将创建自定义编解码器和层输出设置。

在创建时[转换](https://docs.microsoft.com/rest/api/media/transforms)，首先应检查是否其中一个已存在使用**获取**方法，如下面的代码中所示。 在媒体服务 v3 中，如果实体不存在（不区分大小写的名称检查），实体上的 **Get** 方法会返回 **null**。

### <a name="example"></a>示例

下面的示例定义我们想要使用此转换时生成的输出一组。 我们首先添加一个 AacAudio 层的音频编码和视频编码的两个 H264Video 层。 在视频的层，我们将分配标签，以便可以在输出文件名称中使用它们。 接下来，我们想要输出还包括缩略图。 在下面的示例中，我们将指定 PNG 格式，生成在 50%的输入视频的分辨率和三个时间戳的 {25%、 50%、 75} 的输入视频的长度的图像。 最后，我们指定的输出文件-一个用于视频 + 音频格式，另一个用于缩略图。 由于我们有多个 H264Layers，我们必须使用生成每个层的唯一名称的宏。 我们可以使用`{Label}`或`{Bitrate}`宏，该示例显示了前者。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/EncodeCustomTransform/MediaV3ConsoleApp/Program.cs#EnsureTransformExists)]

## <a name="next-steps"></a>后续步骤

[流式传输文件](stream-files-tutorial-with-api.md) 
