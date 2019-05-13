---
title: Azure 媒体服务 v3 概述 | Microsoft Docs
description: 本文提供媒体服务的高级概述和包含详细信息的相关文章的链接。
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: azure 媒体服务, 流, 广播, 实时, 脱机
ms.service: media-services
ms.devlang: multiple
ms.topic: overview
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 05/07/2019
ms.author: juliako
ms.custom: mvc
ms.openlocfilehash: 4fb42014e2c5389cf2a849ddf5114ab1f59c9e28
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65414025"
---
# <a name="azure-media-services-v3-overview"></a>Azure 媒体服务 v3 概述

Azure 媒体服务是一个基于云的平台，用于生成解决方案，以便实现广播质量的视频流、增强可访问性和分发、分析内容，等等。 无论你是应用程序开发者、呼叫中心、政府机构还是娱乐公司，媒体服务都能帮助你创建应用程序，通过当今最热门的移动设备和浏览器向广大受众提供品质卓越的媒体体验。 

> [!NOTE]
> 目前，无法使用 Azure 门户来管理 v3 资源。 请使用 [REST API](https://aka.ms/ams-v3-rest-ref)、[CLI](https://aka.ms/ams-v3-cli-ref) 或支持的 [SDK](developers-guide.md) 之一。

## <a name="what-can-i-do-with-media-services"></a>媒体服务的功能是什么？

通过媒体服务，可以在云中生成各种媒体工作流，以下是媒体服务功能的一些示例。  

* 提供各种格式的视频，以便通过各种浏览器和设备播放视频。 对于面向多种客户端（移动设备、电视、电脑等）的按需和实时流式传输交付，需要对视频和音频进行适当的编码和打包。 要了解如何交付和流式传输此类内容，请参阅[快速入门：编码和流式传输文件](stream-files-dotnet-quickstart.md)。
* 向大量在线观众流式传输实时体育赛事，例如足球、棒球、大学和高中体育运动等等。 
* 广播公共会议和活动，例如市政厅、市议会和立法机构。
* 分析录制的视频或音频内容。 例如，为了实现更高的客户满意度，组织可以提取语音转文本并生成搜索索引和仪表板。 然后，他们可以提取围绕常见的投诉、投诉原因以及其他相关数据的情报。
* 当客户（例如电影工作室）需要限制对版权所有作品的访问和使用时，创建订阅视频服务并流式传输受 DRM 保护的内容。
* 提供脱机内容，以便在飞机、火车和汽车上播放。 如果客户希望断开网络连接，可能需要将内容下载到手机或平板电脑上播放。
* 使用 Azure 媒体服务和 [Azure 认知服务 API](https://docs.microsoft.com/azure/#pivot=products&panel=ai) 实现教育在线学习视频平台，提供语音转文本字幕，多种语言翻译等等。 
* 将 Azure 媒体服务与 [Azure 认知服务 API](https://docs.microsoft.com/azure/#pivot=products&panel=ai) 配合使用来为视频添加字幕和描述文字，满足更多受众的需求（例如，听力障碍人士或想用不同语言阅读的人）。
* 启用 Azure CDN 实现大幅度缩放，更好地处理即时高负载（例如在产品发布活动开始时）。 

## <a name="how-can-i-get-started-with-v3"></a>如何开始使用 v3？ 

了解如何使用媒体服务 v3 来编码和打包内容、进行视频点播流式处理、实时广播和视频分析。 教程、API 参考和其他文档介绍了如何安全地向数百万用户传送可缩放的点播和直播视频或音频流。

> [!TIP]
> 在开始开发之前，请查看：<br/>* [基本概念](concepts-overview.md)（包括重要概念：打包、编码、保护，等等）<br/>* [使用媒体服务 v3 API 进行开发](media-services-apis-overview.md)（包括有关访问 API、命名约定等的信息）

### <a name="quickstarts"></a>快速入门  

快速入门向新客户展示基本的第 1 天使用说明，以便快速试用媒体服务。

* [对视频文件进行流式处理 - .NET](stream-files-dotnet-quickstart.md)
* [对视频文件进行流式处理 - CLI](stream-files-cli-quickstart.md)
* [对视频文件进行流式处理 - Node.js](stream-files-nodejs-quickstart.md)
    
### <a name="tutorials"></a>教程 

教程演示一些顶级媒体服务任务的基于方案的过程。

* [对远程文件和流视频进行编码 - REST](stream-files-tutorial-with-rest.md)
* [对上传的文件和流视频进行编码 - .NET](stream-files-tutorial-with-api.md)
* [实时流 - .NET](stream-live-tutorial-with-api.md)
* [分析视频 - .NET](analyze-videos-tutorial-with-api.md)
* [AES-128 动态加密 - .NET](protect-with-aes128.md)
    
### <a name="how-to-guides"></a>操作指南

文章中包含的示例演示如何完成某项任务。 在此部分可以找到许多示例，下面只是其中的一些示例：

* [创建帐户 - CLI](create-account-cli-how-to.md)
* [访问 API - CLI](access-api-cli-how-to.md)
* [开始使用 SDK 进行开发](developers-guide.md)
* [在将 HTTPS 作为作业输入的情况下进行编码 - .NET](job-input-from-http-how-to.md)  
* [监视事件 - 门户](monitor-events-portal-how-to.md)
* [通过多重 DRM 进行动态加密 - .NET](protect-with-drm.md) 
* [如何对自定义转换进行编码 - CLI](custom-preset-cli-howto.md)

## <a name="ask-questions-give-feedback-get-updates"></a>提出问题、提供反馈、获取更新

查看 [Azure 媒体服务社区](media-services-community.md)文章，了解可以提出问题、提供反馈和获取有关媒体服务的更新的不同方法。

## <a name="next-steps"></a>后续步骤

[了解基本概念](concepts-overview.md)

