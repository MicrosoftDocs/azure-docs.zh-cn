---
title: 是试验性预设识别内容的编码-Azure |Microsoft Docs
description: 本文介绍 Azure 媒体服务中的内容识别的编码
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 04/05/2019
ms.author: juliako
ms.custom: ''
ms.openlocfilehash: ddb7bfd2437af806c8db75068c50545e69867ea0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65151010"
---
# <a name="experimental-preset-for-content-aware-encoding"></a>实验性识别内容的编码预设

若要准备传送的内容[自适应比特率流式处理](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)，视频需要经过编码的多个比特率 （从高到低）。 为了确保妥善降级的质量，如降低比特率，因此是视频的分辨率。 这会导致所谓的编码阶梯 – 的分辨率和比特率; 表请参阅 Media Services[内置的编码预设](https://docs.microsoft.com/rest/api/media/transforms/createorupdate#encodernamedpreset)。

## <a name="overview"></a>概述

关注移动出一个预设的适合-all-视频方法增加 Netflix 发布后其[博客](https://medium.com/netflix-techblog/per-title-encode-optimization-7e99442b62a2)在 2015 年 12 月。 从那时起，识别内容的编码的多个解决方案已在发布 marketplace;请参阅[这篇文章](https://www.streamingmedia.com/Articles/Editorial/Featured-Articles/Buyers-Guide-to-Per-Title-Encoding-130676.aspx)概述。 其理念是内容的需要注意 — — 自定义或优化的各个视频复杂性的编码之。 每个分辨率、 没有超出该任何增加的质量不角度 – 在此获得最佳的比特率值进行操作编码器的比特率。 下一级别的优化是选择基于内容的解决方法 – 例如，PowerPoint 演示文稿的视频不会获得如下 720p 的流。 更进一步，可以对编码器需要负责以优化视频中每个快照的设置。 所述的 Netflix[这种方法](https://medium.com/netflix-techblog/optimized-shot-based-encodes-now-streaming-4b9464204830)2018年中。

2017 年初，Microsoft 发布了[自适应流式处理](autogen-bitrate-ladder.md)预设以解决问题的变化程度的质量和源视频的分辨率。 我们的客户都具有不同各种各样的内容、 一些在 1080p、 720p，其他人和多个在 SD 和较低的分辨率。 此外，并非所有源内容都已从电影或电视演播室高质量 mezzanines。 通过确保永远不会比特率阶梯超过分辨率或输入夹层的平均比特率流式处理预设的地址这些问题的自适应。

实验性识别内容的编码预设扩展该机制，通过将合并自定义逻辑，可让编码器查找最佳的比特率值对于给定的解析，而无需大量计算的分析。 最终结果是此新预设会生成具有低的比特率比自适应流式处理预设中，但在更高质量的输出。 请参阅以下示例关系图显示使用类似的质量度量值的比较[PSNR](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio)并[VMAF](https://en.wikipedia.org/wiki/Video_Multimethod_Assessment_Fusion)。 源已通过串联的电影的高的复杂性截图短小的剪辑和电视节目，旨在强调编码器。 根据定义，改变内容到内容 – 此预设会生成结果它也意味着某些内容可能不会出现显著降低比特率或质量的改进。

![使用 PSNR 速率扭曲 (RD) 曲线](media/cae-experimental/msrv1.png)

图 1：**为高的复杂性源使用 PSNR 指标的速率扭曲 (RD) 曲线**

![使用 VMAF 速率扭曲 (RD) 曲线](media/cae-experimental/msrv2.png)

图 2：**为高的复杂性源使用 VMAF 指标的速率扭曲 (RD) 曲线**

当前预设已优化了高的复杂性，高质量源视频 （电影、 电视节目）。 工作正在进行中以适应较低的复杂性内容 （例如，PowerPoint 演示文稿），以及不佳质量视频。 此预设还使用一组相同的解决方法，如自适应流式处理预设。 Microsoft 正致力于方法来选择最小的基于内容的解决方法集。 按如下所示是结果另一种类别的源内容编码器已能够确定输入是低质量 （许多由于低比特率压缩项目）。 请注意，进行实验性预设，编码器决定生成一个输出层 – 足够低的比特率，因此大多数客户端能够而无需停止播放流。

![使用 PSNR RD 曲线](media/cae-experimental/msrv3.png)

图 3：**（在 1080p) 的低质量输入使用 PSNR RD 曲线**

![使用 VMAF RD 曲线](media/cae-experimental/msrv4.png)

图 4：**（在 1080p) 的低质量输入使用 VMAF RD 曲线**

## <a name="use-the-experimental-preset"></a>使用实验性预设

您可以创建使用此预设，如下所示的转换。 如果使用的教程，说明如何[如下](stream-files-tutorial-with-api.md)，你可以更新代码，如下所示：

```csharp
TransformOutput[] output = new TransformOutput[]
{
   new TransformOutput
   {
      // The preset for the Transform is set to one of Media Services built-in sample presets.
      // You can customize the encoding settings by changing this to use "StandardEncoderPreset" class.
      Preset = new BuiltInStandardEncoderPreset()
      {
         // This sample uses the new experimental preset for content-aware encoding
         PresetName = EncoderNamedPreset.ContentAwareEncodingExperimental
      }
   }
};
```

> [!NOTE]
> 此处使用"实验"前缀以指示基础算法仍在发展。 那里可以并且将随时间用于生成比特率阶梯，目的是为了聚合算法的功能强大，并能够适应各种输入条件的逻辑会更改。 编码作业使用此预设的将仍是计费基于的输出分钟，并且可以从在协议，如 DASH 和 HLS 流式处理终结点传递输出资产。

## <a name="next-steps"></a>后续步骤

现在，你已了解有关优化视频的此新选项，我们邀请您试试看。可以向我们发送反馈链接使用本文中，结束时，也可以与我们更直接地在合作<amsved@microsoft.com>。
