---
title: Azure IoT 中心术语词汇表 | Microsoft Docs
description: 开发人员指南 - 与 Azure IoT 中心相关的常用术语的术语表。
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/15/2019
ms.openlocfilehash: 6f89e27b06179c33857d581c0c6e3fc78c683d48
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62119803"
---
# <a name="glossary-of-iot-hub-terms"></a>IoT 中心术语表
本文列出了一些在 IoT 中心文章中使用的常用术语。

## <a name="advanced-message-queueing-protocol"></a>高级消息队列协议
[高级消息队列协议 (AMQP)](https://www.amqp.org/) 是 [IoT 中心](#iot-hub)支持的一种消息传送协议，适用于与设备通信。 有关 IoT 中心支持的消息传递协议的详细信息，请参阅[使用 IoT 中心发送和接收消息](iot-hub-devguide-messaging.md)。

## <a name="automatic-device-management"></a>自动设备管理
Azure IoT 中心内的自动设备管理功能可将许多复杂且重复性的任务自动化，包括在大型设备阵列的整个生命周期内对其进行管理。 使用自动设备管理，可以根据设备的属性将一组设备指定为目标、定义所需的配置，并在设备进入管理范畴时让 IoT 中心更新这些设备。  包括[自动设备配置](iot-hub-auto-device-config.md)和 [IoT Edge 自动部署](../iot-edge/how-to-deploy-monitor.md)。

## <a name="automatic-device-configuration"></a>自动设备配置
解决方案后端可以使用[自动设备配置](iot-hub-auto-device-config.md)将所需属性分配到一组[设备孪生](#device-twin)，并使用系统指标和自定义指标来报告状态。 

## <a name="azure-classic-cli"></a>Azure 经典 CLI
[Azure 经典 CLI](../cli-install-nodejs.md) 是一个跨平台、开放源代码、基于 shell 的命令工具，适用于在 Microsoft Azure 中创建和管理资源。 此版本的 CLI 仅应用于经典部署。

## <a name="azure-cli"></a>Azure CLI
[Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2) 是一个跨平台、开源、基于 shell 的命令工具，适用于在 Microsoft Azure 中创建和管理资源。


## <a name="azure-iot-device-sdks"></a>Azure IoT 设备 SDK
提供了多种语言的_设备 SDK_，方便用户创建与 IoT 中心交互的[设备应用](#device-app)。 IoT 中心教程介绍了如何使用这些设备 SDK。 可以在此 GitHub [存储库](https://github.com/Azure/azure-iot-sdks)中找到有关设备 SDK 的源代码和更多信息。

## <a name="azure-iot-service-sdks"></a>Azure IoT 服务 SDK
提供了多种语言的_服务 SDK_，方便用户创建与 IoT 中心交互的[后端应用](#back-end-app)。 IoT 中心教程介绍了如何使用这些服务 SDK。 可以在此 GitHub [存储库](https://github.com/Azure/azure-iot-sdks)中找到有关服务 SDK 的源代码和进一步信息。

## <a name="azure-iot-tools"></a>Azure IoT Tools
[Azure IoT 工具](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)是一个跨平台的开放源代码 Visual Studio Code 扩展，可帮助你在 VS Code 中管理 Azure IoT 中心和设备。 使用 Azure IoT 工具，IoT 开发人员可以轻松在 VS Code 中开发 IoT 项目。

## <a name="azure-portal"></a>Azure 门户
[Microsoft Azure 门户](https://portal.azure.com)是一个中心位置，可在其中预配和管理 Azure 资源。 该门户使用_边栏选项卡_组织其内容。

## <a name="azure-powershell"></a>Azure PowerShell
[Azure PowerShell](/powershell/azure/overview) 是一个 cmdlet 集合，可用于通过 Windows PowerShell 管理 Azure。 可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。

## <a name="azure-resource-manager"></a>Azure 资源管理器
可以使用 [Azure 资源管理器](../azure-resource-manager/resource-group-overview.md)以组的方式处理解决方案中的资源。 可以通过一个协调操作为解决方案部署、更新或删除资源。

## <a name="azure-service-bus"></a>Azure 服务总线
通过[服务总线](../service-bus/index.md)，可在云端使用企业消息传递进行通信，也可采用中继进行通信，帮助将本地解决方案与云端连接。 某些 IoT 中心教程使用服务总线[队列](../service-bus-messaging/service-bus-messaging-overview.md)。

## <a name="azure-storage"></a>Azure 存储
[Azure 存储](../storage/common/storage-introduction.md)是一种云存储解决方案。 它包含可用于存储非结构化的对象数据的 Blob 存储服务。 某些 IoT 中心教程使用 blob 存储。

## <a name="back-end-app"></a>后端应用
在 [IoT 中心](#iot-hub)环境中，后端应用是指连接到 IoT 中心的一个面向服务的终结点的应用。 例如，后端应用可能检索[设备到云](#device-to-cloud)的消息或管理[标识注册表](#identity-registry)。 通常，后端应用在云中运行，但在许多教程中，后端应用是在本地开发计算机上运行的控制台应用。

## <a name="built-in-endpoints"></a>内置终结点
每个 IoT 中心都包括一个与事件中心兼容的内置[终结点](iot-hub-devguide-endpoints.md)。 可以使用任何适用于事件中心的机制从此终结点读取设备到云消息。

## <a name="cloud-gateway"></a>云网关
云网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。 与在设备本地运行的[现场网关](#field-gateway)相反，云网关在云中托管。 云网关的一个典型用例是实现设备的协议转换。

## <a name="cloud-to-device"></a>云到设备
指从 IoT 中心发送到已连接设备的消息。 这些消息通常是命令，用于指示设备采取某项操作。 有关详细信息，请参阅[使用 IoT 中心发送和接收消息](iot-hub-devguide-messaging.md)。

## <a name="configuration"></a>配置
在使用[自动设备配置](iot-hub-auto-device-config.md)的情况下，可以通过 IoT 中心的配置为一组设备孪生定义所需的配置，并提供一组指标来报告状态和进度。

## <a name="connection-string"></a>连接字符串
使用应用程序代码中的连接字符串来封装连接到终结点所需的信息。 连接字符串通常包含终结点的地址和安全信息，但连接字符串的格式因服务而异。 与 IoT 中心服务关联的连接字符串有两种：
- 设备连接字符串使设备能够连接到 IoT 中心上面向设备的终结点。
- *IoT 中心连接字符串* 使后端应用能够连接到 IoT 中心上面向服务的终结点。

## <a name="custom-endpoints"></a>自定义终结点
可在 IoT 中心创建自定义[终结点](iot-hub-devguide-endpoints.md)传递由[路由规则](#routing-rules)调度的消息。 自定义终结点直接连接事件中心、服务总线队列或服务总线主题。

## <a name="custom-gateway"></a>自定义网关
网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。 可以使用 Azure IoT Edge 生成自定义网关，以便使用自定义逻辑处理消息、自定义协议转换和进行边缘上的其他处理。

## <a name="data-point-message"></a>数据点消息
数据点消息是指[设备到云](#device-to-cloud)的消息，其中包含[遥测](#telemetry)数据（例如风速或温度）。

## <a name="desired-configuration"></a>所需配置
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的语境中，所需配置是指设备孪生中要与设备同步的完整的属性和元数据集。

## <a name="desired-properties"></a>所需的属性
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的语境中，所需属性是设备孪生的一部分，和[报告属性](#reported-properties)一起用于同步设备配置或条件。 所需属性只能由[后端应用](#back-end-app)设置，并由[设备应用](#device-app)遵守。

## <a name="device-to-cloud"></a>设备到云
指从已连接设备发送到 [IoT 中心](#iot-hub)的消息。 这些消息可能是[数据点](#data-point-message)或[交互式](#interactive-message)消息。 有关详细信息，请参阅[使用 IoT 中心发送和接收消息](iot-hub-devguide-messaging.md)。

## <a name="device"></a>设备
在 IoT 上下文中，设备通常是指小型、独立的计算设备，可用于收集数据或控制其他设备。 例如，设备可以是环境监视设备，也可以是控制器，控制温室中的浇水和通风系统。 [设备目录](https://catalog.azureiotsolutions.com/)提供经过认证的可用于 [IoT 中心](#iot-hub)的硬件设备列表。

## <a name="device-app"></a>设备应用
设备应用在用户的[设备](#device)上运行，处理与 [IoT 中心](#iot-hub)的通信。 通常情况下，实现设备应用时会使用一个 [Azure IoT 设备 SDK](#azure-iot-device-sdks)。 在许多 IoT 教程中，为方便起见使用[模拟设备](#simulated-device)。

## <a name="device-condition"></a>设备条件
指[设备应用](#device-app)报告的设备状态信息，例如当前使用的连接方法。 [设备应用](#device-app)还可以报告其功能。 可以使用设备孪生查询条件和功能的信息。

## <a name="device-data"></a>设备数据
设备数据是指存储在 IoT 中心[标识注册表](#identity-registry)中的每个设备数据。 可以导入和导出此数据。

## <a name="device-explorer"></a>设备资源管理器
[设备资源管理器](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)是在 Windows 上运行的工具，使你可以管理[标识注册表](#identity-registry)中的设备。该工具还可以发送和接收设备的消息。

## <a name="device-identity"></a>设备标识
设备标识是分配给在[标识注册表](#identity-registry)中注册的每个设备的唯一标识符。

## <a name="device-management"></a>设备管理
设备管理包含在 IoT 解决方案中管理设备的完整生命周期，包括规划、预配、配置、监视和停用设备。

## <a name="device-management-patterns"></a>设备管理模式
[IoT 中心](#iot-hub)支持常见的设备管理模式，包括重新启动、执行恢复出厂设置，以及执行设备的固件更新。

## <a name="device-rest-api"></a>设备 REST API
在设备上使用[设备 REST API](https://docs.microsoft.com/rest/api/iothub/device)，可以将设备到云消息发送到 IoT 中心，并能从 IoT 中心接收[云到设备](#cloud-to-device)消息。 通常情况下，应使用 IoT 中心教程中演示的一种较高级别的[设备 SDK](#azure-iot-device-sdks)。

## <a name="device-provisioning"></a>设备预配
[设备预配](#device-data)是将初始设备数据添加到解决方案中存储中的过程。 要使新设备能够连接到中心，必须将新设备 ID 和密钥添加到 IoT 中心的[标识注册表](#identity-registry)。 在预配过程中，可能需要初始化其他解决方案存储中的设备特定数据。

## <a name="device-twin"></a>设备孪生
[设备孪生](iot-hub-devguide-device-twins.md)是存储设备状态信息（如元数据、配置和条件）的 JSON 文档。 [IoT 中心](#iot-hub)为在 IoT 中心预配的每台设备保留一个设备孪生。 借助设备孪生可以在设备和解决方案后端之间同步[设备条件](#device-condition)和配置。 可以通过查询设备孪生来定位特定设备和查询长时间运行的操作状态。

## <a name="direct-method"></a>直接方法
[直接方法](iot-hub-devguide-direct-methods.md)可让你通过在 IoT 中心上调用 API来触发在设备上执行的方法。

## <a name="endpoint"></a>终结点
IoT 中心公开了多个[终结点](iot-hub-devguide-endpoints.md)，以便使应用能够连接到 IoT 中心。 有面向设备的终结点，通过此终结点设备可以执行一些操作，例如发送[设备到云](#device-to-cloud)的消息和接收[云到设备](#cloud-to-device)的消息。 有面向服务的管理终结点，通过此终结点[后端应用](#back-end-app)可执行一些操作，如[设备标识](#device-identity)管理和设备孪生管理。 还有面向服务的[内置终结点](#built-in-endpoints)，用于读取设备到云的消息。 可创建[自定义终结点](#custom-endpoints)，接收由[路由规则](#routing-rules)调度的设备到云消息。

## <a name="event-hubs-service"></a>事件中心服务
[事件中心](../event-hubs/event-hubs-what-is-event-hubs.md)是高度可缩放的数据引入服务，每秒可以引入数以百万计的事件。 该服务使用户能够处理和分析连接设备和应用程序产生的大量数据。 有关该服务与 IoT 中心服务的比较的信息，请参阅 [Azure IoT 中心与 Azure 事件中心的比较](iot-hub-compare-event-hubs.md)。

## <a name="event-hub-compatible-endpoint"></a>与事件中心兼容的终结点
若要读取发送到 IoT 中心的[设备到云](#device-to-cloud)消息，可以连接到中心上的终结点，并使用任何事件中心兼容的方法读取这些消息。 事件中心兼容的方法包括使用[事件中心 SDK](../event-hubs/event-hubs-programming-guide.md) 和 [Azure 流分析](../stream-analytics/stream-analytics-introduction.md)。

## <a name="field-gateway"></a>现场网关
使用现场网关与无法直接连接到 [IoT 中心](#iot-hub)的设备建立连接。现场网关通常部署在设备本地。 有关详细信息，请参阅[什么是 Azure IoT 中心？](about-iot-hub.md)

## <a name="free-account"></a>免费帐户
可以创建[免费的 Azure 帐户](https://azure.microsoft.com/pricing/free-trial/)，以便使用 IoT 中心服务（及其他 Azure 服务）完成 IoT 中心教程和试验。

## <a name="gateway"></a>网关
网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。 另请参阅[现场网关](#field-gateway)、[云网关](#cloud-gateway)和[自定义网关](#custom-gateway)。

## <a name="identity-registry"></a>标识注册表
[标识注册表](iot-hub-devguide-identity-registry.md)是 IoT 中心的内置组件，用于存储允许连接到 IoT 中心的单个设备的信息。

## <a name="interactive-message"></a>交互式消息
交互式消息是 [云到设备](#cloud-to-device) 的消息，可在解决方案后端触发即时操作。 例如，设备可能会发送故障警报，而该故障会自动记录到 CRM 系统中。

[!INCLUDE [azure-iot-hub-edge-glossary-includes](../../includes/azure-iot-hub-edge-glossary-includes.md)]

## <a name="iot-hub"></a>IoT 中心
IoT 中心是完全托管的 Azure 服务，用于支持数百万台设备和解决方案后端之间可靠、安全的双向通信。 有关详细信息，请参阅[什么是 Azure IoT 中心？](about-iot-hub.md) 使用 [Azure 订阅](#subscription)可以创建 IoT 中心来处理 IoT 消息传送工作负荷。

## <a name="iot-hub-metrics"></a>IoT 中心指标
[IoT 中心指标](iot-hub-metrics.md)向用户提供有关 [Azure 订阅](#subscription)中的 IoT 中心的状态数据。 可以使用 IoT 中心度量值评估服务以及连接到服务的设备的总体运行状况。 IoT 中心度量值可以帮助用户了解 IoT 中心发生的情况，并调查根本原因，而无需联系 Azure 支持部门。

## <a name="iot-hub-query-language"></a>IoT 中心查询语言
[IoT 中心查询语言](iot-hub-devguide-query-language.md)是一种类似于 SQL 的语言，用于查询 [](#job) 和设备孪生。

## <a name="iot-hub-resource-rest-api"></a>IoT 中心资源 REST API
使用 [IoT 中心资源 REST API](https://docs.microsoft.com/rest/api/iothub/iothubresource)，可以管理 [Azure 订阅](#subscription)中的 IoT 中心，包括创建、更新和删除中心等操作。

## <a name="iot-solution-accelerators"></a>IoT 解决方案加速器
Azure IoT 解决方案加速器将多个 Azure 服务一起打包到解决方案中。 利用这些解决方案，能够快速开始使用常用 IoT 方案的端到端实现。 有关详细信息，请参阅 [Azure IoT 解决方案加速器是什么？](../iot-accelerators/about-iot-accelerators.md)

## <a name="the-iot-extension-for-azure-cli"></a>适用于 Azure CLI 的 IoT 扩展 
[适用于 Azure CLI 的 IoT 扩展](https://github.com/Azure/azure-iot-cli-extension)是一个跨平台的命令行工具。 使用该工具可以管理[标识注册表](#identity-registry)中的设备、向设备发送消息和文件和接收来自设备的消息和文件，以及监视 IoT 中心的操作。

## <a name="job"></a>作业
解决方案后端可以使用[作业](iot-hub-devguide-jobs.md)来计划和跟踪在 IoT 中心注册的一组设备上的活动。 活动包括更新设备孪生[所需的属性](#desired-properties)、更新设备孪生[标记](#tags)，以及调用[直接方法](#direct-method)。 [IoT 中心](#iot-hub)还用于在[标识注册表](#identity-registry)中[导入和导出](iot-hub-devguide-identity-registry.md#import-and-export-device-identities)内容。

## <a name="modules"></a>模块
在设备端，可以使用 IoT 中心设备 SDK 创建[模块](iot-hub-devguide-module-twins.md)，其中每个模块都与 IoT 中心单独建立连接。 通过此功能，可对设备上的不同组件使用不同的命名空间。

模块标识和模块孪生提供的功能与[设备标识](#device-identity)和[设备孪生](#device-twin)相同，但前者的粒度更细。 这种更高的粒度级可让有能力的设备（例如基于操作系统的设备，或管理多个组件的固件设备）隔离其中每个组件的配置和状态。

## <a name="module-identity"></a>模块标识
模块标识是分配给设备所属的每个模块的唯一标识符。 模块标识也注册到[标识注册表](#identity-registry)中。

## <a name="module-twin"></a>模块孪生
与设备孪生类似，模块孪生是存储模块状态信息（如元数据、配置和条件）的 JSON 文档。 IoT 中心为在 IoT 中心的设备标识下预配的每个模块实体保留一个模块孪生。 可以借助模块孪生在模块和解决方案后端之间同步模块条件和配置。 可以通过查询模块孪生来定位特定模块和查询长时间运行的操作状态。

## <a name="mqtt"></a>MQTT
[MQTT](https://mqtt.org/) 是 [IoT 中心](#iot-hub)支持与设备进行通信的消息传递协议之一。 有关 IoT 中心支持的消息传递协议的详细信息，请参阅[使用 IoT 中心发送和接收消息](iot-hub-devguide-messaging.md)。

## <a name="operations-monitoring"></a>操作监视
使用 IoT 中心[操作监视](iot-hub-operations-monitoring.md)可实时监视其 IoT 中心上的操作状态。 [IoT 中心](#iot-hub)可以跨多个类别的操作跟踪事件。 可以选择将一个或多个类别的事件发送到 IoT 中心终结点进行处理。 可以监视数据中是否有错误，或根据数据模式设置更复杂的处理行为。

## <a name="physical-device"></a>物理设备
物理设备是真实的设备，如连接到 IoT 中心的 Raspberry Pi。 为方便起见，许多 IoT 中心教程使用[模拟设备](#simulated-device)在本地计算机上运行示例。

## <a name="primary-and-secondary-keys"></a>主要和次要密钥
连接到 IoT 中心的面向设备或面向服务的终结点时，[连接字符串](#connection-string)包含用于授权的密钥。 在[标识注册表](#identity-registry)中添加设备或者在中心添加[共享访问策略](#shared-access-policy)时，服务将生成主要和次要密钥。 拥有两个密钥能够在更新密钥时从一个密钥切换到另一个密钥，而不会失去 IoT 中心访问权限。

## <a name="protocol-gateway"></a>协议网关
协议网关通常部署在云中，为连接到 [IoT 中心](#iot-hub)的设备提供协议转换服务。 有关详细信息，请参阅[什么是 Azure IoT 中心？](about-iot-hub.md)

## <a name="quotas-and-throttling"></a>配额和限制
各种[配额](iot-hub-devguide-quotas-throttling.md)可用于 [IoT 中心](#iot-hub)，其中许多配额因所在的中心的层而异。 [IoT 中心](#iot-hub)在运行时也对服务的使用实施一些[限制](iot-hub-devguide-quotas-throttling.md)。

## <a name="reported-configuration"></a>报告的配置
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的上下文中，报告的配置是指设备孪生中的完整属性和元数据集，该配置应报告给解决方案后端。

## <a name="reported-properties"></a>报告的属性
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的语境中，报告的属性是设备孪生的一部分，和[所需属性](#desired-properties)一起用于同步设备配置或条件。 报告的属性只能由[设备应用](#device-app)设置，可由[后端应用](#back-end-app)读取和查询。

## <a name="resource-group"></a>资源组
[Azure 资源管理器](#azure-resource-manager)使用资源组将相关的资源组合在一起。 通过使用资源组，可以对组中的所有资源同时执行操作。

## <a name="retry-policy"></a>重试策略
连接到云服务时使用重试策略来处理[暂时性错误](/azure/architecture/best-practices/transient-faults)。

## <a name="routing-rules"></a>路由规则
需要在 IoT 中心配置[路由规则](iot-hub-devguide-messages-read-custom.md)，将设备到云消息路由到[内置终结点](#built-in-endpoints)或[自定义终结点](#custom-endpoints)，以供解决方案后端处理。

## <a name="sasl-plain"></a>SASL PLAIN
SASL PLAIN 是一种协议，AMQP 协议使用它来传输安全令牌。

## <a name="service-rest-api"></a>服务 REST API
可以从解决方案后端使用[服务 REST API](https://docs.microsoft.com/rest/api/iothub/service) 来管理设备。 使用 API，不仅可以检索和更新[设备孪生](#device-twin)属性，还能调用[直接方法](#direct-method)并安排[作业](#job)。 通常，应使用 IoT 中心教程中演示的一种较高级别的[服务 SDK](#azure-iot-service-sdks)。

## <a name="shared-access-signature"></a>共享访问签名
共享访问签名 (SAS) 是基于 SHA–256 安全哈希或 URI 的身份验证机制。 SAS 身份验证有两个组件：_共享访问策略_和_共享访问签名_（通常称为令牌）。 设备使用 SAS 在 IoT 中心进行身份验证。 [后端应用](#back-end-app)也使用 SAS 在 IoT 中心的面向服务的终结点上进行身份验证。 通常，在[连接字符串](#connection-string)中包含 SAS 令牌，应用使用此令牌建立与 IoT 中心的连接。

## <a name="shared-access-policy"></a>共享访问策略
共享访问策略定义向具有有效的[主要密钥或次要密钥](#primary-and-secondary-keys)（与该策略相关联）的任何人授予的权限。 可以在[门户](#azure-portal)中管理中心的共享访问策略和密钥。

## <a name="simulated-device"></a>模拟设备
为方便起见，许多 IoT 中心教程使用模拟设备，以便在本地计算机上运行示例。 相反，[物理设备](#physical-device)是真实的设备，如连接到 IoT 中心的 Raspberry Pi。

## <a name="solution"></a>解决方案
_解决方案_可以是包含一个或多个项目的 Visual Studio 解决方案。 _解决方案_也可能是包括诸如设备、[设备应用](#device-app)、IoT 中心、其他 Azure 服务和[后端应用](#back-end-app)等元素的 IoT 解决方案。

## <a name="subscription"></a>订阅
Azure 订阅是发生计费的地方。 创建的每个 Azure 资源或使用的 Azure 服务均与单个订阅关联。 许多配额也在订阅级别应用。

## <a name="system-properties"></a>系统属性
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的语境中，系统属性为只读，其中包括与设备使用情况相关的信息，例如上次活动时间和连接状态。

## <a name="tags"></a>标记
在与[设备孪生](iot-hub-devguide-device-twins.md)相关的语境中，标记是指由解决方案后端以 JSON 文档形式存储和检索的设备元数据。 标记对于设备上的应用程序不可见。

## <a name="telemetry"></a>遥测
设备收集遥测数据，如风速或温度，并使用数据点消息将遥测数据发送到 IoT 中心。

## <a name="token-service"></a>令牌服务
可以使用令牌服务对设备实施身份验证机制。 它使用包含 **DeviceConnect** 权限的 IoT 中心[共享访问策略](#shared-access-policy) 创建*设备范围的*令牌。 这些令牌可让设备连接到 IoT 中心。 设备通过令牌服务使用自定义的身份验证机制进行身份验证。 如果设备成功通过身份验证，那么令牌服务向设备颁发 SAS 令牌用于访问 IoT 中心。

## <a name="twin-queries"></a>孪生查询
[设备和模块孪生查询](iot-hub-devguide-query-language.md)使用类似于 SQL 的 IoT 中心查询语言从设备孪生或模块孪生中检索信息。 可以使用相同的 IoT 中心查询语言，检索在 IoT 中心内运行的 [](#job) 的相关信息。

## <a name="twin-synchronization"></a>孪生同步
孪生同步使用设备孪生或模块孪生中的[所需属性](#desired-properties)配置设备或模块，并检索其中的[报告属性](#reported-properties)，以将这些属性存储在孪生中。

## <a name="x509-client-certificate"></a>X.509 客户端证书
设备可以使用 X.509 证书在 [IoT 中心](#iot-hub)进行身份验证。 使用 X.509 证书是使用 [SAS 令牌](#shared-access-signature)的替代方案。
