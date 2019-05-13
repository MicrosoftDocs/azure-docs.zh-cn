---
title: 了解 Azure 安全中心，iot 成本预览 |Microsoft Docs
description: 了解如何使用 Azure 安全中心的 IoT，以及如何控制它们相关的成本。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: ef839708-4574-4a40-bc45-07005f8e9daf
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2019
ms.author: mlottner
ms.openlocfilehash: 40963e0d81ccff21cc625b799833b996547a13b6
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198318"
---
# <a name="pricing-and-associated-costs"></a>定价和关联的成本

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文介绍 Azure 安全中心 (ASC) 的 IoT 定价模型，汇总了所有相关的成本并说明如何对其进行管理。

## <a name="pricing"></a>定价

ASC 为 IoT 定价模型包含两个部件，并后 IoT 中心是计费[启用](quickstart-onboard-iot-hub.md)iot ASC 中：

- 通过设备的 IoT 中心日志的分析所基于的内置安全功能的成本。

- 按消息-基于来自 IoT Edge 或叶设备的安全消息的增强的安全性功能分类的成本。

  >[!Note]
  > 安全消息还将产生在 IoT 中心的配额使用情况。

有关详细信息，请参阅[安全中心定价](https://azure.microsoft.com/pricing/details/security-center/)。

## <a name="associated-costs"></a>关联的成本

适用于 IoT 的 ASC 具有两种类型的相关成本，不是直接的定价的一部分：

- IoT 中心配额使用情况

- Log Analytics 存储成本

通过选择禁用某些功能，通过更改设置，可以减少相关的成本。

若要更改您的设置：

1. 打开 IoT 中心。

2. 下**安全**，单击**概述**。

3. 单击“设置”。

下表提供了相关联的成本的摘要和意义的每个选项。

|     | 使用情况 | 注释 |
| --- | --- | --- |
| **IoT 中心配额使用情况** |  |
| [导出设备](https://docs.microsoft.com/azure/iot-hub/iot-hub-bulk-identity-mgmt#export-devices)作业 （孪生导出） | 每天 1 次 | 禁用_孪生元数据集合_ |
| **Log Analytics 存储** |  |
| 设备的建议和警报| 安全建议和由服务生成的警报 | 不可选 |
| 原始安全数据| 从 IoT 设备，安全代理收集的原始安全数据 | 禁用_存储原始设备安全事件_ |

>[!Important]
> 选择禁用有可用的安全功能的严重影响。
  
| 选择禁用 | 影响 |
| --- | --- |
| _孪生元数据集合_ | 禁用[自定义警报](quickstart-create-custom-alerts.md) |
| | 禁用 IoT Edge 清单建议 |
| | 禁用设备基于标识的建议和警报 |
| _存储原始设备安全事件_ | 设备 OS 基线建议的详细信息不可用 |
| | 详细信息[警报](concept-security-alerts.md)并[建议](concept-recommendations.md)调查不可用 |


## <a name="see-also"></a>另请参阅

- 访问你[原始安全数据](how-to-security-data-access.md)
- [调查设备](how-to-investigate-device.md)
- 了解和探索[安全建议](concept-recommendations.md)
- 了解和探索[安全警报](concept-security-alerts.md)
