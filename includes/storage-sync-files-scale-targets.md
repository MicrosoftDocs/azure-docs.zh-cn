---
title: include 文件
description: include 文件
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 05/05/2019
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: b480f9720cabddbba9e9c35c12ca1f8dda7ab5ab
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190038"
---
| 资源 | 确定目标 | 硬限制 |
|----------|--------------|------------|
| 每个区域的存储同步服务数 | 20 个存储同步服务 | 是 |
| 每个存储同步服务的同步组数 | 100 个同步组 | 是 |
| 每个存储同步服务的已注册服务器 | 99 台服务器 | 是 |
| 每个同步组的云终结点 | 1 个云终结点 | 是 |
| 每个同步组的服务器终结点 | 50 个服务器终结点 | 否 |
| 每个服务器的服务器终结点数 | 30 个服务器终结点 | 是 |
| 终结点大小 | 4 TiB | 否 |
| 每个同步组的文件系统对象数（目录和文件） | 2500 万个对象 | 否 |
| 目录中的最大文件系统对象（目录和文件）数 | 1 亿个对象 | 是 |
| 最大对象（目录和文件）安全描述符大小 | 64 KiB | 是 |
| 文件大小 | 100 GiB | 否 |
| 要进行分层的文件的最小文件大小 | 64 KiB | 是 |
| 并发同步会话 | V4 代理及更高版本：根据可用系统资源限制而异。 <BR> V3 代理：每个处理器或最多为每个服务器的八个活动的同步会话的两个活动的同步会话。 | 是
