---
author: rothja
ms.service: billing
ms.topic: include
ms.date: 11/09/2018
ms.author: jroth
ms.openlocfilehash: de3fd8dc0d45ea10e64af8e2258682a9e98639dc
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150347"
---
>[!NOTE]
>对于不固定的资源，请打开支持票证以请求增加配额中。 不尝试获取更高的限制中创建更多 Azure 媒体服务帐户。

| 资源 | 默认限制 | 
| --- | --- | 
| 单个订阅中的 azure 媒体服务帐户 | 25（固定） |
| 媒体保留单位，每个媒体服务帐户 |25 (S1)<br/>10 (S2, S3)<sup>1</sup> | 
| 每个媒体服务帐户的作业数 | 50,000<sup>2</sup> |
| 每个作业的链接任务数 | 30（固定） |
| 每个媒体服务帐户的资产 | 1,000,000|
| 每个任务的资产数 | 50 |
| 每个作业的资产数 | 100 |
| 一次与一个资产关联的唯一定位符数 | 5<sup>4</sup> |
| 每个媒体服务帐户的实时频道数 |5|
| 每个频道的停止状态节目数 |50|
| 每个频道的运行状态节目数 |3|
| 流式处理终结点已停止的或运行每个媒体服务帐户|2|
| 每个流式处理终结点的流式处理单位数 |10 |
| 存储帐户 | 1,000<sup>5</sup>（固定） |
| 策略 | 1,000,000<sup>6</sup> |
| 文件大小| 在某些情况下，支持在媒体服务中处理的最大文件大小没有限制。<sup>7</sup> |

<sup>1</sup>如果更改类型，例如，从 s2 切换到 S1，最大保留的单位限制会重置。

<sup>2</sup>此数字包括已排队、 已完成、 活动和已取消作业。 它不包括删除的作业。 可以通过删除旧作业**IJob.Delete**或**删除**HTTP 请求。

截至 2017 年 4 月 1 日，任何在你的帐户 90 天以前是自动删除作业记录，及其关联的任务记录。 即使记录总数低于最大配额，则自动删除。 要存档的作业和任务的信息，请使用代码中所述[使用媒体服务.NET SDK 管理资产](../articles/media-services/previous/media-services-dotnet-manage-entities.md)。

<sup>3</sup>请求列表作业实体时，每个请求返回 1,000 个作业最多。 若要跟踪的所有已提交的作业，请使用顶部或跳过的查询中所述[OData 系统查询选项](/previous-versions/dynamicscrm-2015/developers-guide/gg309461(v=crm.7))。

<sup>4</sup>定位符不用于管理每个用户的访问控制。 若要提供给单个用户的不同访问权限，使用数字版权管理 (DRM) 解决方案。 有关详细信息，请参阅[使用 Azure 媒体服务保护内容](../articles/media-services/previous/media-services-content-protection-overview.md)。

<sup>5</sup>的存储帐户必须是从同一 Azure 订阅。

<sup>6</sup>为 1,000,000 个不同的媒体服务策略的限制。 例如，对于定位器策略或 ContentKeyAuthorizationPolicy。 

>[!NOTE]
> 如果始终使用相同的日期和访问权限，则使用相同的策略 id。 有关信息和示例，请参阅[使用媒体服务.NET SDK 管理资产](../articles/media-services/previous/media-services-dotnet-manage-entities.md#limit-access-policies)。

<sup>7</sup>对单个 blob 目前最多为 5 TB，在 Azure Blob 存储中受支持的最大大小。 媒体服务基于服务使用的 VM 大小应用其他限制。 大小限制适用于你上传的文件以及导致处理 （编码或分析） 的媒体服务获取生成的文件。 如果源文件大于 260 GB，作业可能会失败。 

下表显示了对媒体保留单位的 S1、 S2 和 s3 版的限制。 如果源文件大于表中所定义的限制，查看编码作业将失败。 如果编码的持续时间长的 4k 分辨率源时，您需要使用 S3 媒体保留单位来实现所需的性能。 如果有大于 S3 媒体保留单位的 260 GB 限制的 4k 内容，请联系我们：amshelp@microsoft.com有关可能的缓解措施来支持你的方案。

|媒体保留单位类型   |最大输入的大小 (GB)|
|---|---|
|S1 |   26|
|S2 | 60|
|S3 |260|
