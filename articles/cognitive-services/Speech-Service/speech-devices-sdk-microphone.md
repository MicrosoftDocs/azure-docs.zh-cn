---
title: 语音设备 SDK 麦克风阵列建议-语音服务
titleSuffix: Azure Cognitive Services
description: 语音设备 SDK 的麦克风阵列建议。 以下数组几何图形被建议使用与 Microsoft 音频堆栈。 声音的源位置和避免环境噪音拒绝得到改进，更多的麦克风具有依赖项特定的应用程序、 用户方案和设备外观造型上。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 63dd64e900cf68e708032569ca75ac2e8b221491
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237002"
---
# <a name="speech-devices-sdk-microphone-array-recommendations"></a>语音设备 SDK 麦克风阵列建议

在本文中，您将了解如何设计的语音设备 SDK 的麦克风阵列。

语音设备 SDK 最适用于旨在根据以下指导原则，包括麦克风几何图形和组件选择麦克风阵列。 指南还提供了集成和电力注意事项。

## <a name="microphone-geometry"></a>麦克风 geometry

以下数组几何图形被建议使用与 Microsoft 音频堆栈。 声音的源位置和避免环境噪音拒绝得到改进，更多的麦克风具有依赖项特定的应用程序、 用户方案和设备外观造型上。

|          | 循环的数组    |       |  线性数组              |                |
|----------|-------------------|-------------------|----------------|----------------|
|          |<img src="media/speech-devices-sdk/7-mic-c.png" alt="7 mic circular array" width="150"/>|<img src="media/speech-devices-sdk/4-mic-c.png" alt="4 mic circular array" width="150"/>|<img src="media/speech-devices-sdk/4-mic-l.png" alt="4 mic linear array" width="150"/>|<img src="media/speech-devices-sdk/2-mic-l.png" alt="2 mic linear array" width="150"/>|
| \# Mics  | 7                 | 4                 | 4              | 2              |
| 几何图形 | 6 的外部，1 Center、 半径 = 42.5 mm，均匀分布| 3 个外部，1 Center、 半径 = 42.5 mm，均匀分布 | 长度 = 120 mm 间距 = 40 mm | 间距 = 40 mm |

应根据为每个数组，从 0 增加上面所示的编号排序麦克风通道。  Microsoft 音频堆栈将需要执行回声抵消的音频播放的附加引用流。

## <a name="component-selection"></a>选择组件

应选择麦克风组件来准确地重现免费的干扰和失真的信号。

选择麦克风时的建议的属性包括：

| 参数                         | 建议                       |
|-----------------------------------|-----------------------------------|
| SNR                               | \> 65 dB （1khz 信号 94 dBSPL，一个加权干扰）   |
| Amplitude 匹配                | 为 @ 1khz 1 个 dB                     |
| 匹配阶段                    | 为 @ 1khz 2 °                       |
| 声学重载点 (AOP)     | \> 120 dBSPL (THD = 10%)          |
| 比特率                          | 最小值 24 位                    |
| 采样率                     | 最小 16 kHz\*                   |
| Directivity                       | 全向                   |
| 频率响应                | 为 3 个 dB，200 8000 Hz 浮点掩码\*|
| 可靠性                       | 存储温度范围-40 ° C 到 70 ° C<br />操作-20 ° C 到 55 ° C 的温度范围  |

*\*更高版本的采样率或"宽"频率范围可能有必要为高质量通信 (VoIP) 应用程序*

选择好组件必须与完美 electroacoustic 集成成对使用以避免因而会影响使用的组件的性能。 唯一的用例可能还需要其他要求 (例如： 操作系统温度范围)。

## <a name="microphone-array-integration"></a>麦克风阵列集成

数组时集成到设备和任何固定的收益或 EQ 后性能应符合以下建议：

|  参数        |    建议 |
|--------------------|----------------------------------------------------|
|  SNR                 | \> 65 dB （1khz 信号 94 dBSPL，一个加权干扰） |
|  输出敏感度  | -26 dBFS/Pa @ 1khz （推荐） |
|  Amplitude 匹配  | 为 2 个 dB，200 8000 Hz |
|  匹配阶段      | 为 5 °，200 8000 Hz |
| THD %                 | ≤ 1%、 200 8000 Hz，94 dBSPL，第五个顺序 |
|  频率响应  | 为 6 的 dB，200 8000 Hz 浮点掩码\* |

*\*"宽度"频率范围可能有必要为高质量通信 (VoIP) 应用程序*

## <a name="speaker-integration-recommendations"></a>演讲者集成的建议

由于包含说话人的语音识别设备所需回显取消，则其他建议提供的演讲者选择和集成。

| 参数                         | 建议                       |
|-----------------------------------|-----------------------------------|
| 线性注意事项          | 任何非线性演讲者引用之后，否则基于硬件的环回引用流所需的处理  |
| 演讲者 Loopback                  | 提供通过 wasapi 就可以了、 专用 Api、 自定义 ALSA 插件 (Linux)，或通过固件通道提供      |
| THD %                              | 第三个八度音带区最小第五个订单，70 dBA 播放 @ 0.8 m ≤ 6.3%，315 500 Hz ≤ 5%、 630 5000 Hz                 |
| Echo 耦合到麦克风      | \> -10 dB TCLw 使用 ITU-T G.122 Annex B.4 方法被规范化为麦克风输入级别<br />TCLw = TCLwmeasured \+ （级别-测量以目标输出敏感度）<br />TCLw = TCLwmeasured \+ （单位级别--(26)） |

## <a name="integration-design-architecture"></a>集成设计体系结构

将麦克风集成到设备时，所需体系结构的以下准则：

| 参数                         | 建议                    |
|-----------------------------------|-----------------------------------|
| Mic 端口相似性               | 麦克风的所有端口都都在数组中相同的长度    |
| Mic 端口维度               | 端口大小 Ø0.8 1.0 毫米。 端口长度 / 端口直径\<2              |
| Mic 密封                       | 密封垫片均匀地实施中累积。 建议\>泡沫垫片的 70%压缩率     |
| Mic 可靠性                   | 应使用网格来避免灰尘和入口 （PCB 的底部之间移植麦克风和密封垫片/top 封面）  |
| Mic 隔离                     | 橡胶垫片和振动分离通过结构，特别是对于隔离集成扬声器由于任何振动路径      |
| 采样时钟                    | 设备音频必须是自由的抖动和退出的情况具有低偏差    |
| 记录功能                 | 设备必须能够同时记录单个通道原始流 |
| USB                               | 所有 USB 音频输入的设备必须都设置根据描述符[USB 音频设备 Rev3 规范](https://www.usb.org/document-library/usb-audio-devices-rev-30-and-adopters-agreement) |
| 麦克风 Geometry               | 驱动程序必须实现[麦克风阵列 Geometry 描述符](https://docs.microsoft.com/windows-hardware/drivers/audio/ksproperty-audio-mic-array-geometry)正确  |
| 可发现性                   | 设备必须没有任何不可发现的或无法控制的硬件、 固件或与设备的第三方基于软件的非线性音频处理算法|
| 捕获格式                    | 捕获格式必须使用 16 kHz 和建议的 24 位深度的最小采样率      |

## <a name="electrical-architecture-considerations"></a>电力体系结构注意事项

数组在适用的情况下，可能会连接到 USB 主机 （例如运行 Microsoft 音频堆栈 SoC) 和语音服务或其他应用程序的接口。

硬件组件，如 PDM TDM 转换应确保在重新取样器保留的动态范围和 SNR 麦克风。

为了提供最多七个通道在更高版本的采样速率和位深度的必要带宽，应内任何音频 Mcu 支持高速 USB 音频类 2.0。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解有关语音设备 SDK 的详细信息](speech-devices-sdk.md)
