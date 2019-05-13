---
title: 配置适用于 IoT 解决方案的 Azure 安全中心（预览版）| Microsoft Docs
description: 了解如何使用适用于 IoT 的 Azure 安全中心配置端到端 IoT 解决方案。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: ae2207e8-ac5b-4793-8efc-0517f4661222
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: c60b421e9b60c6a2191fe2be189d1abd1c328f24
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200791"
---
# <a name="quickstart-configure-your-iot-solution"></a>快速入门：配置 IoT 解决方案

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文介绍如何使用 ASC for IoT 执行 IoT 安全解决方案的初始配置。 

## <a name="azure-security-center-asc-for-iot"></a>适用于 IoT 的 Azure 安全中心 (ASC)

ASC for IoT 为基于 Azure 的 IoT 解决方案提供全面的端到端安全性。

使用适用于 IoT 的 ASC 时，可在一个仪表板中监视整个 IoT 解决方案，并显示 Azure 中你的所有 IoT 设备、IoT 平台和后端资源。

在 IoT 中心启用后，ASC for IoT 会自动识别也已连接到你的 IoT 中心的，以及与你的 IoT 解决方案相关的其他 Azure 服务。

除了自动关系检测以外，还可以选择要将其他哪些 Azure 资源标记为 IoT 解决方案的一部分。
做出选择后，可以添加整个订阅、资源组或单个资源。

定义所有资源关系后，ASC for IoT 将利用 Azure 安全中心来提供安全建议，并针对这些资源发出警报。

## <a name="add-azure-resources-to-your-iot-solution"></a>将 Azure 资源添加到 IoT 解决方案

若要将新资源添加到 IoT 解决方案，请执行以下操作： 

1. 在 Azure 门户中打开你的 **IoT 中心**。 
2. 在左侧菜单中，选择并打开“安全性”下的“资源”。 
3. 选择“添加资源”。
4. 选择属于该 IoT 解决方案的资源。
5. 单击“添加”。 

祝贺你！ 现已将新资源添加到 IoT 解决方案。

ASC for IoT 现在会监视新添加的资源，并在 IoT 解决方案中显示相关的安全建议和警报。

## <a name="next-steps"></a>后续步骤

请转到下一篇文章了解如何创建安全模块...

> [!div class="nextstepaction"]
> [创建安全模块](quickstart-create-security-twin.md)