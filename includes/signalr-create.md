---
title: include 文件
description: include 文件
services: signalr
author: wesmc7777
ms.service: signalr
ms.topic: include
ms.date: 04/17/2018
ms.author: wesmc
ms.custom: include file
ms.openlocfilehash: 57407606214d8d3a305476cfbfdabca9eee937e8
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60690278"
---
1. 若要创建新的 Azure SignalR 服务资源，请先登录到 [Azure 门户](https://portal.azure.com)。 在页面的左上方，单击“+ 创建资源”。 在“搜索市场”文本框中，键入“SignalR 服务”并按 **Enter**。

2. 在结果中单击“SignalR 服务”，然后单击“创建”。

3. 在新的“SignalR”设置页上，为新的 SignalR 资源添加以下设置：

    | 名称 | 建议的值 | 说明 |
    | ---- | ----------------- | ----------- |
    | 资源名称 | *testsignalr* | 输入用于 SignalR 资源的唯一资源名称。 该名称必须是包含 1 到 63 个字符的字符串，只能包含数字、字母和 `-` 字符。 该名称的开头或末尾不能是 `-` 字符，并且连续的 `-` 字符无效。|
    | 订阅 | 选择自己的订阅 |  选择要用来测试 SignalR 的 Azure 订阅。 如果帐户只有一个订阅，则会自动选择该订阅并且不显示“订阅”下拉菜单。|
    | 资源组 | 创建名为 *SignalRTestResources* 的新资源组| 为 SignalR 资源选择或创建资源组。 此组可用于组织多个资源，删除该资源组可以同时删除这些资源。 有关详细信息，请参阅[使用资源组管理 Azure 资源](../articles/azure-resource-manager/resource-group-overview.md)。 |
    | 位置 | *美国东部* | 使用“位置”指定在其中托管 SignalR 资源的地理位置。 为获得最佳性能，我们建议在应用程序的其他组件所在的同一区域创建资源。 |
    | 定价层 | *免费* | 目前可以使用“免费”和“标准”选项。 |
    | 固定到仪表板 | ✔ | 选中此框可将资源固定到仪表板，以方便查找。 |

4. 单击“创建”。 部署可能需要几分钟才能完成。

5. 部署完成后，单击“设置”下面的“密钥”。 复制主密钥连接字符串。 稍后要使用此值将应用配置为使用 Azure SignalR 服务资源。

    该连接字符串采用以下格式：
    
        Endpoint=<service_endpoint>;AccessKey=<access_key>;
