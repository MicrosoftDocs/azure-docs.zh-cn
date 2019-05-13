---
title: Azure 媒体服务 v3 中的配额和限制 | Microsoft Docs
description: 本主题介绍 Azure 媒体服务 v3 中的配额和限制
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/02/2019
ms.author: juliako
ms.openlocfilehash: 42b8c4caa53ffa6b3bc1148544c75602597ac452
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153828"
---
# <a name="quotas-and-limitations-in-azure-media-services-v3"></a>Azure 媒体服务 v3 中的配额和限制

本文介绍 Azure 媒体服务 v3 中的配额和限制。

| 资源 | 默认限制 | 
| --- | --- | 
| 每个 Azure 媒体服务帐户的资产数 | 1,000,000|
| 动态清单筛选器|100|
| 每个作业的 JobInputs | 50（固定）|
| 每个作业的 JobOutputs | 20（固定） |
| 转换中的 TransformOutput 数 | 20（固定） |
| 每个 JobInput 的文件数|10（固定）|
| 文件大小| 在某些情况下，支持在媒体服务中处理的最大文件大小存在限制。 <sup>(1)</sup> |
| 每个媒体服务帐户的作业数 | 500,000 <sup>(2)</sup>（固定）|
| 列出转换数|对响应进行分页，每页 1000 个转换|
| 列出作业数|对响应进行分页，每页 500 个作业|
| 每个媒体服务帐户的实时事件数 |5|
| 单个订阅中的媒体服务帐户数 | 25（固定） |
| 每个实时事件的实时输出 |3 <sup>(3)</sup> |
| 最大实时输出持续时间 | 25 个小时 |
| 存储帐户 | 100<sup>(4)</sup>（固定） |
| 每个媒体服务帐户的流式处理端点（已停止或正在运行）|2 （固定）|
| 流式处理策略 | 100 <sup>(5)</sup> |
| 每个媒体服务帐户的转换数 | 100（固定）|
| 一次与一个资产关联的唯一流式处理定位符 | 100<sup>(6)</sup> （固定） |
| 内容密钥策略 |30 | 

<sup>1</sup> 在 Azure Blob 存储中，单个 Blob 目前支持的最大大小为 5 TB。 媒体服务基于服务使用的 VM 大小应用其他限制。 大小限制适用于你上传的文件以及导致处理 （编码或分析） 的媒体服务获取生成的文件。 如果源文件大于 260 GB，作业可能会失败。 

下表显示了对媒体保留单位的 S1、 S2 和 s3 版的限制。 如果源文件大于表中所定义的限制，查看编码作业将失败。 如果编码的持续时间长的 4k 分辨率源时，您需要使用 S3 媒体保留单位来实现所需的性能。 如果有大于 S3 媒体保留单位的 260 GB 限制的 4k 内容，请联系我们：amshelp@microsoft.com有关可能的缓解措施来支持你的方案。

|媒体保留单位类型   |最大输入的大小 (GB)|
|---|---|
|S1 |   26|
|S2 | 60|
|S3 |260|

<sup>2</sup> 这个数字包括已排队的、已完成的、活动的和已取消的作业。 不包括已删除的作业。 

即使记录总数低于最大配额，也会自动删除帐户中所有超过 90 天的作业记录。 

<sup>3</sup> live 输出创建时启动和停止时删除。

<sup>4</sup> 存储帐户必须来自同一 Azure 订阅。

<sup>5</sup>时使用自定义[流式处理策略](https://docs.microsoft.com/rest/api/media/streamingpolicies)，您应设计一组有限的此类策略为您的媒体服务帐户，并重新将其用于你 StreamingLocators 时相同的加密选项和协议所需。 不应为每个流式处理定位符创建新的流式处理策略。

<sup>6</sup>流式处理定位符不用于管理每个用户的访问控制。 要为不同用户提供不同的访问权限，请使用数字权限管理 (DRM) 解决方案。

## <a name="support-ticket"></a>支持票证

对于不固定的资源，可以通过开具[支持票证](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)请求提高配额。 请在请求中包含有关所需的配额更改、用例方案和所需区域的详细信息。 <br/>请**不要**创建更多的 Azure 媒体服务帐户以求获取更高的限制。

## <a name="next-steps"></a>后续步骤

[概述](media-services-overview.md)
