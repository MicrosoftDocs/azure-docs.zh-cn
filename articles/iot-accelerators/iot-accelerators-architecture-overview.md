---
title: IoT 解决方案加速器参考体系结构 - Azure | Microsoft Docs
description: 了解 Azure IoT 解决方案加速器参考体系结构。 现有解决方案加速器利用此参考体系结构。 生成自己的自定义 IoT 解决方案时，还可以使用该参考体系结构。
author: dominicbetts
ms.author: dobett
ms.date: 12/04/2018
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-accelerators
services: iot-accelerators
manager: philmea
ms.openlocfilehash: 1e3ec677f2c3ce2a7f942c47269fe1f70885ac02
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466473"
---
# <a name="introduction-to-the-azure-iot-reference-architecture"></a>Azure IoT 参考体系结构简介

本文介绍 [Azure IoT 参考体系结构](https://aka.ms/iotrefarchitecture)，并举例说明 [Azure IoT 解决方案加速器](about-iot-accelerators.md)如何遵循其建议。

开源[远程监视](iot-accelerators-remote-monitoring-sample-walkthrough.md)和[连接工厂](iot-accelerators-connected-factory-sample-walkthrough.md)解决方案加速器遵循许多参考体系结构建议。 可以将此解决方案加速器用作自己的 IoT 解决方案的起点，或者将其用作学习工具。

## <a name="overview"></a>概述

参考体系结构描述 IoT 解决方案的元素（如技术原理、Azure IoT 服务的构成以及设备）。 体系结构还提出有关实现技术的建议。

在最高级别，可以将 IoT 解决方案视为由以下部分组成：

* 生成并发送遥测的部分，如度量和事件。 在远程监视解决方案加速器中，设备（如卡车或电梯）是发送遥测的部分。
* 见解由上述部分发出的遥测生成。 在远程监视解决方案加速器中，一项规则生成一个见解。 例如，规则可以识别引擎中的温度何时达到阈值。
* 基于见解的操作，有助于改进业务或流程。 在远程监视解决方案加速器中，电子邮件操作可以将引擎的潜在问题告知操作员。

[Azure IoT 参考体系结构](https://aka.ms/iotrefarchitecture)是随技术发展而更新的动态文档。

## <a name="core-subsystems"></a>核心子系统

参考体系结构标识下图所示的核心子系统：

![核心子系统](media/iot-accelerators-architecture-overview/coresubsystems1.png)

以下部分描述远程监视解决方案加速器的组件如何映射到核心子系统。

### <a name="iot-devices"></a>IoT 设备

IoT 解决方案应该能够在几乎任何类型的设备和云网关之间实现安全、高效和可靠的通信。 设备是商业资产，涵盖简单的温度传感器到拥有数百个组件和传感器的复杂工厂生产线。

现场网关或边缘设备是专门的设备或通用软件，可以充当：

* 处理协议转换的通信启用程序。
* 本地设备控制系统和设备遥测数据处理中心。 边缘设备可以在不与云通信的情况下在本地处理遥测数据以控制设备。 边缘设备还可以筛选或聚合设备遥测数据，以减少传输到云的遥测数据量。

在远程监视解决方案中，设备连接到 IoT 中心并发送遥测数据进行处理。 借助远程监视解决方案，操作员还可以使用作业或自动设备管理来管理设备。 使用 Azure IoT 设备 SDK 实现设备。

远程监视解决方案可以部署和管理 IoT Edge 设备。 边缘处理有助于减少发送到云的遥测数据量，并加快对设备事件的响应。

### <a name="cloud-gateway"></a>云网关

云网关支持设备和边缘设备之间的通信。 这些设备可能位于多个远程位置。

网关管理设备通信（包括连接管理、通信路径保护、身份验证和授权）。 网关还强制执行连接和吞吐量配额，并收集用于计费、诊断和其他监视任务的遥测数据。

远程监视解决方案部署一个 IoT 中心，提供安全终结点，供设备向其发送遥测数据。 IoT 中心还：

* 包含设备标识存储，用于管理可连接到解决方案的设备。
* 支持解决方案将命令发送到设备。 例如，打开阀门释放压力。
* 支持批量设备管理。 例如，升级一组设备上的固件。

### <a name="stream-processing"></a>流处理

由于解决方案引入遥测数据，请务必了解遥测数据处理的流程差异。 数据记录可以流经不同的阶段，具体取决于不同场景：

* 存储，如内存缓存、临时队列和永久存档。

* 分析，通过一组条件运行输入遥测数据并能生成各种输出数据记录。 例如，以 Avro 编码的输入遥测数据可能返回以 JSON 编码的输出遥测数据。

* 原始的输入遥测数据和分析输出记录通常存储起来并可供显示。 输入遥测数据和输出记录还可能触发电子邮件、事件票证和设备命令等操作。

路由可以将遥测数据调度到一个或多个存储终结点、分析过程和操作。 解决方案可以按各种顺序组合这些阶段，并用并发并行任务对其进行处理。

远程监视解决方案使用 [Azure 流分析](/azure/stream-analytics/)进行流式处理。 解决方案中的规则引擎使用“流分析”查询来生成警报和操作。 例如，解决方案可以使用查询来确定 5 分钟内卡车置物箱中的平均温度何时低于 36 度。

### <a name="storage"></a>存储

IoT 解决方案可以生成大量数据，且通常是时间序列数据。 这些数据需要存储在可用于可视化和报告的位置。 解决方案可能还需要稍后访问数据以进行分析或其他处理。 通常将数据分为暖数据存储和冷数据存储。 暖数据存储保存最近的数据，用于低延迟访问。 冷数据存储通常存储历史数据。

远程监视解决方案将 [Azure 时序见解](/azure/time-series-insights/)作为其暖数据存储，将 Cosmos DB 作为冷数据存储。

### <a name="ui-and-reporting-tools"></a>UI 和报告工具

解决方案 UI 可以提供：

* 设备数据和分析结果的访问权限和可视化。
* 通过注册表的设备发现。
* 设备的命令和控制。
* 设备预配工作流。
* 通知和警报。
* 与实时交互仪表板集成，显示来自大量设备的数据。  
* 地理位置和异地感知服务。

远程监视解决方案包括实现此功能的 Web UI。 Web UI 包括：

* 显示设备位置的交互式地图。
* 对查询和分析遥测数据的时序见解资源管理器的访问权限。

### <a name="business-integration"></a>业务集成

业务集成层处理 IoT 解决方案与业务系统（如 CRM、ERP 和业务线应用）的集成。 例如，服务账单、客户支持和备件供应。

远程监视解决方案使用规则在满足条件时发送电子邮件。 例如，解决方案可以在卡车温度低于 36 度时通知操作员。

## <a name="next-steps"></a>后续步骤

在本文中，你了解了 Azure IoT 参考体系结构，并查看了远程监视解决方案加速器如何遵循其建议的一些示例。 后续步骤是阅读 [Microsoft Azure IoT 参考体系结构](https://aka.ms/iotrefarchitecture)。