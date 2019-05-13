---
title: 示例：使用大规模功能 - 人脸 API
titleSuffix: Azure Cognitive Services
description: 使用人脸 API 中的大规模功能。
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 05/01/2019
ms.author: sbowles
ms.openlocfilehash: 35ab2d36a5d6c9977398fdbc16ba22eb1d9656a4
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65229839"
---
# <a name="example-how-to-use-the-large-scale-feature"></a>示例：如何使用大规模功能

本指南是有关如何从现有 **PersonGroup** 和 **FaceList** 分别纵向扩展到 **LargePersonGroup** 和 **LargeFaceList** 的高级文章。 本指南演示迁移过程，假设读者基本上熟悉 **PersonGroup**、**FaceList**、[训练](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4)操作和人脸识别函数。 请参阅[人脸识别](../concepts/face-recognition.md)概念指南以了解更多相关信息。

LargePersonGroup 和 LargeFaceList 统称为大规模操作。 LargePersonGroup 最多可以包含 1,000,000 个人员，每个人员最多有 248 张人脸；LargeFaceList 最多可以包含 1,000,000 张人脸。大规模操作类似于传统的 PersonGroup 和 FaceList，但由于采用新的体系结构，同时也存在一些明显的差异。 

示例在 C# 中使用人脸 API 客户端库编写。

> [!NOTE]
> 若要面向大规模启用人脸搜索性能以进行 Identification 和 FindSimilar ，则需要引入“定型”操作，以预处理 LargeFaceList 和 LargePersonGroup。 定型时间从几秒到约半小时不等，具体取决于实际容量。 如果之前进行过成功定型，则仍可能在定型期间执行 Identification 和 FindSimilar 操作。 但缺点是，在完成迁移到大规模定型的最新后处理前，新添加的人员/人脸不会出现在结果中。

## <a name="step-1-initialize-the-client-object"></a>步骤 1：初始化客户端对象

使用人脸 API 客户端库时，订阅密钥和订阅终结点将通过 FaceServiceClient 类的构造函数传入。 例如：

```CSharp
string SubscriptionKey = "<Subscription Key>";
// Use your own subscription endpoint corresponding to the subscription key.
string SubscriptionRegion = "https://westcentralus.api.cognitive.microsoft.com/face/v1.0/";
FaceServiceClient FaceServiceClient = new FaceServiceClient(SubscriptionKey, SubscriptionRegion);
```

可从 Azure 门户的“商城”页获取具有对应终结点的订阅密钥。
请参阅[订阅](https://azure.microsoft.com/services/cognitive-services/directory/vision/)。

## <a name="step-2-code-migration"></a>步骤 2：代码迁移

本部分仅介绍如何将 PersonGroup/FaceList 实现迁移到 LargePersonGroup/LargeFaceList。 虽然 LargePersonGroup/LargeFaceList 与 PersonGroup/FaceList 在设计和内部实现方面存在差异，但用于后向兼容的 API 接口是类似的。

不支持数据迁移，必须改为重新创建 LargePersonGroup/LargeFaceList。

### <a name="migrate-persongroup-to-largepersongroup"></a>将 PersonGroup 迁移到 LargePersonGroup

从 PersonGroup 迁移到 LargePersonGroup 较为容易，因为它们采用完全相同的组级别操作。

对于 PersonGroup/Person 相关的实现，仅需要将 API 路径或 SDK 类/模块更改为 LargePersonGroup 和 LargePersonGroup Person。

需要将 PersonGroup 中的所有人脸和人员添加到新的 LargePersonGroup。 请参考[如何添加人脸](how-to-add-faces.md)。

### <a name="migrate-facelist-to-largefacelist"></a>将 FaceList 迁移到 LargeFaceList

| FaceList API | LargeFaceList API |
|:---:|:---:|
| 创建 | 创建 |
| 删除 | 删除 |
| 获取 | 获取 |
| 列出 | 列出 |
| 更新 | 更新 |
| - | 定型 |
| - | 获取定型状态 |

上表对 FaceList 和 LargeFaceList 的列级操作进行了对比。 如表所示，与 FaceList 相比，LargeFaceList 具有新操作（定型和获取定型状态）。 对 LargeFaceList 进行定型是 [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) 操作的前提条件，而 FaceList 则不需要定型。 以下代码片段是一个帮助器函数，用于等待 LargeFaceList 定型。

```CSharp
/// <summary>
/// Helper function to train LargeFaceList and wait for finish.
/// </summary>
/// <remarks>
/// The time interval can be adjusted considering the following factors:
/// - The training time which depends on the capacity of the LargeFaceList.
/// - The acceptable latency for getting the training status.
/// - The call frequency and cost.
///
/// Estimated training time for LargeFaceList in different scale:
/// -     1,000 faces cost about  1 to  2 seconds.
/// -    10,000 faces cost about  5 to 10 seconds.
/// -   100,000 faces cost about  1 to  2 minutes.
/// - 1,000,000 faces cost about 10 to 30 minutes.
/// </remarks>
/// <param name="largeFaceListId">The Id of the LargeFaceList for training.</param>
/// <param name="timeIntervalInMilliseconds">The time interval for getting training status in milliseconds.</param>
/// <returns>A task of waiting for LargeFaceList training finish.</returns>
private static async Task TrainLargeFaceList(
    string largeFaceListId,
    int timeIntervalInMilliseconds = 1000)
{
    // Trigger a train call.
    await FaceServiceClient.TrainLargeFaceListAsync(largeFaceListId);

    // Wait for training finish.
    while (true)
    {
        Task.Delay(timeIntervalInMilliseconds).Wait();
        var status = await FaceServiceClient.GetLargeFaceListTrainingStatusAsync(largeFaceListId);

        if (status.Status == Status.Running)
        {
            continue;
        }
        else if (status.Status == Status.Succeeded)
        {
            break;
        }
        else
        {
            throw new Exception("The train operation is failed!");
        }
    }
}
```

以前，添加了人脸和 FindSimilar 的 FaceList 的典型用法如下所示：

```CSharp
// Create a FaceList.
const string FaceListId = "myfacelistid_001";
const string FaceListName = "MyFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
FaceServiceClient.CreateFaceListAsync(FaceListId, FaceListName).Wait();

// Add Faces to the FaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await FaceServiceClient.AddFaceToFaceListAsync(FaceListId, stream);
            }
        });

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = FaceServiceClient.DetectAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await FaceServiceClient.FindSimilarAsync(face.FaceId, FaceListId, 20));
    }
}
```

将它迁移到 LargeFaceList 时，它应变成：

```CSharp
// Create a LargeFaceList.
const string LargeFaceListId = "mylargefacelistid_001";
const string LargeFaceListName = "MyLargeFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
FaceServiceClient.CreateLargeFaceListAsync(LargeFaceListId, LargeFaceListName).Wait();

// Add Faces to the LargeFaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await FaceServiceClient.AddFaceToLargeFaceListAsync(LargeFaceListId, stream);
            }
        });

// Train() is newly added operation for LargeFaceList.
// Must call it before FindSimilarAsync() to ensure the newly added faces searchable.
await TrainLargeFaceList(LargeFaceListId);

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = FaceServiceClient.DetectAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await FaceServiceClient.FindSimilarAsync(face.FaceId, largeFaceListId: LargeFaceListId));
    }
}
```

如上所示，数据管理和 FindSimilar 部分几乎一样。 唯一的例外是，全新的预处理定型操作必须在 LargeFaceList 中完成，然后 FindSimilar 才能正常工作。

## <a name="step-3-train-suggestions"></a>步骤 3：训练建议

尽管定型操作可加快 [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) 和 [Identification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239)，但定型时间非常煎熬，尤其是涉及大规模操作时。 下表列出了不同规模的估计定型时间：

| 规模（人脸或人员） | 估计定型时间 |
|:---:|:---:|
| 1,000 | 1-2 秒 |
| 10,000 | 5-10 秒 |
| 100,000 | 1 - 2 分钟 |
| 1,000,000 | 10 - 30 分钟 |

为了更好地利用大规模功能，建议考虑采用某些策略。

## <a name="step-31-customize-time-interval"></a>步骤 3.1：自定义时间间隔

如 `TrainLargeFaceList()` 中所示，可通过 `timeIntervalInMilliseconds` 延迟无限定型状态检查过程。 对于包含更多人脸的 LargeFaceList，使用较大间隔可减少调用计数和成本。 应根据 LargeFaceList 的预期容量自定义时间间隔。

同样的策略也适用于 LargePersonGroup。 例如，当定型包含 1,000,000 人的 LargePersonGroup 时，`timeIntervalInMilliseconds` 可能为 60000（1 分钟间隔）。

## <a name="step-32-small-scale-buffer"></a>步骤 3.2：小规模缓冲区

LargePersonGroup/LargeFaceList 中的人员/人脸仅在定型后才可搜索。 在动态方案中，新人员/人脸会不断增加，并且需要立即可供搜索，但定型时间可能超过所需时间。 要缓解此问题，可使用额外的小规模 LargePersonGroup/LargeFaceList，仅作为新增条目的缓冲区。 由于规模较小，此缓冲区所需定型时间较短，因此应可在此临时缓冲区中实现即刻搜索。 在使用此缓冲区的同时，通过按较为稀疏的时间间隔（如在午夜或每天一次）执行主定型来对主 LargePersonGroup/LargeFaceList 执行定型。

示例工作流：
1. 创建一个主 LargePersonGroup/LargeFaceList（主集合）和一个缓冲区 LargePersonGroup/LargeFaceList（缓冲区集合）。 缓冲区集合仅用于新增人员/人脸。
1. 同时向主集合和缓冲区集合添加新人员/人脸。
1. 仅按短时间间隔定型缓冲区集合，以确保新添加的条目生效。
1. 同时对主集合和缓冲区集合调用 Identification/FindSimilar，然后合并结果。
1. 当缓冲区集合大小增加到阈值或在系统空闲时，创建新的缓冲区集合并触发对主集合的定型。
1. 完成对主集合的定型后，删除旧的缓冲区集合。

## <a name="step-33-standalone-training"></a>步骤 3.3：独立训练

如果可以介绍相对较长的时间延迟，则不需要在添加新数据后立即触发定型操作。 相反，可从主逻辑中拆分定型操作并定期触发该操作。 此策略适用于具有可接受延迟的动态方案；还可应用于静态方案，用于进一步降低定型频率。

假定存在类似于 `TrainLargeFaceList` 的 `TrainLargePersonGroup` 函数。 通过调用 `System.Timers` 中的 [`Timer`](https://msdn.microsoft.com/library/system.timers.timer(v=vs.110).aspx) 类，LargePersonGroup 上的独立定型的典型实现应为：

```CSharp
private static void Main()
{
    // Create a LargePersonGroup.
    const string LargePersonGroupId = "mylargepersongroupid_001";
    const string LargePersonGroupName = "MyLargePersonGroupDisplayName";
    FaceServiceClient.CreateLargePersonGroupAsync(LargePersonGroupId, LargePersonGroupName).Wait();

    // Setup a standalone training at regular intervals.
    const int TimeIntervalForStatus = 1000 * 60; // 1 minute interval for getting training status.
    const double TimeIntervalForTrain = 1000 * 60 * 60; // 1 hour interval for training.
    var trainTimer = new Timer(TimeIntervalForTrain);
    trainTimer.Elapsed += (sender, args) => TrainTimerOnElapsed(LargePersonGroupId, TimeIntervalForStatus);
    trainTimer.AutoReset = true;
    trainTimer.Enabled = true;

    // Other operations like creating persons, adding faces and Identification except for Train.
    // ...
}

private static void TrainTimerOnElapsed(string largePersonGroupId, int timeIntervalInMilliseconds)
{
    TrainLargePersonGroup(largePersonGroupId, timeIntervalInMilliseconds).Wait();
}
```

有关数据管理和识别相关实现的详细信息，请参阅[如何添加人脸](how-to-add-faces.md)和[如何识别图像中的人脸](HowtoIdentifyFacesinImage.md)。

## <a name="summary"></a>摘要

本指南介绍了如何将现有 PersonGroup/FaceList 代码（不是数据）迁移到 LargePersonGroup/LargeFaceList 中：

- LargePersonGroup 和 LargeFaceList 的工作原理类似于 PersonGroup/FaceList，但 LargeFaceList 需要使用定型操作。
- 采取适当的定型策略，以对大规模数据集执行动态数据更新。

## <a name="next-steps"></a>后续步骤

请遵循操作指南了解如何将人脸添加到 PersonGroup，或针对 PersonGroup 执行“识别”操作。

- [如何添加人脸](how-to-add-faces.md)
- [如何识别图像中的人脸](HowtoIdentifyFacesinImage.md)