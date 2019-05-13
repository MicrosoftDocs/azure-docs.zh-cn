---
title: 教程：创建 Azure 时序见解环境 | Microsoft Docs
description: 了解如何创建时序见解环境，并在其中填充模拟设备的数据。
services: time-series-insights
author: ashannon7
ms.service: time-series-insights
ms.topic: tutorial
ms.date: 04/26/2019
ms.author: anshan
manager: cshankar
ms.custom: seodec18
ms.openlocfilehash: 42a7ba0c66bd603b19d60c7b3407ae5ca80db28e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209839"
---
# <a name="tutorial-create-an-azure-time-series-insights-environment"></a>教程：创建 Azure 时序见解环境

本教程逐步讲解如何创建时序见解 (TSI) 环境，并在其中填充模拟设备的数据。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建 TSI 环境 
> * 创建包含 IoT 中心的设备模拟解决方案
> * 将 TSI 环境连接到 IoT 中心
> * 运行设备模拟以将数据流式传输到 TSI 环境
> * 验证模拟的遥测数据

## <a name="video"></a>视频

### <a name="learn-how-to-use-an-azure-iot-solution-accelerator-to-generate-data-and-get-started-with-time-series-insights-br"></a>了解如何使用 Azure IoT 解决方案加速器生成数据和开始使用时序见解。 </br>

> [!VIDEO https://www.youtube.com/embed/6ehNf6AJkFo]

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请创建一个[免费帐户](https://azure.microsoft.com/free/)。

* 所用的 Azure 登录帐户还需是订阅“所有者”角色的成员。 有关更多详细信息，请参阅[使用 RBAC 和 Azure 门户管理访问权限](/azure/role-based-access-control/role-assignments-portal)。

## <a name="overview"></a>概述

设备数据将在 TSI 环境中收集和存储。 将设备数据存储到 TSI 环境后，可以使用 [TSI 资源管理器](time-series-quickstart.md)和 [TSI 查询 API](/rest/api/time-series-insights/ga-query-api) 来查询和分析数据。 IoT 中心是所有设备（模拟设备或物理设备）用来安全连接 Azure 云以及向其传输数据的连接点。 [TSI 概述](time-series-insights-overview.md)中指出，Azure IoT 中心还可充当事件源，用于将数据流式传输到 TSI 环境。 本教程使用 [IoT 解决方案加速器](/azure/iot-accelerators/)来生成示例遥测数据并将其流式传输到 IoT 中心。

>[!TIP]
> IoT 解决方案加速器提供企业级的预配置解决方案，可用于加速自定义 IoT 解决方案的开发。

## <a name="create-a-tsi-environment"></a>创建 TSI 环境

首先，在 Azure 订阅中创建一个 TSI 环境：

1. 使用 Azure 订阅帐户登录到 [Azure 门户](https://portal.azure.com)。  
1. 选择左上角的“+ 创建资源”。  
1. 依次选择“物联网”类别、“时序见解”。  

   [![选择时序见解环境资源](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png#lightbox)

1. 在“时序见解环境”页上，填写必需的参数：

   参数|说明
   ---|---
   **环境名称** | 选择 TSI 环境的唯一名称。 该名称将由 TSI 资源管理器和查询 API 使用。
   **订阅** | 订阅是 Azure 资源的容器。 选择要为其创建 TSI 环境的订阅。
   **资源组** | 资源组是 Azure 资源的容器。 为 TSI 环境资源选择现有的资源组或创建新的资源组。
   **位置** | 选择 TSI 环境的数据中心区域。 为了避免带宽成本和延迟提高，最好是将 TSI 环境与其他 IoT 资源保留在同一区域。
   **定价 SKU** | 选择所需的吞吐量。 为了尽量降低成本并获得入门容量，请选择 `S1`。
   **容量** | 容量是应用于入口速率、存储容量和所选 SKU 相关成本的乘数。  创建后可以更改容量。 为了尽量降低成本，请选择 1 个单位的容量。

   完成后，单击“创建”开始预配过程。

   [![创建时序见解环境资源](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-params.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-params.png#lightbox)

1. 查看“通知”面板来监视部署进度。  

   [![时序见解环境部署成功](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-deployment-succeeded.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-deployment-succeeded.png#lightbox)

## <a name="create-a-device-simulation"></a>创建设备模拟

接下来，创建生成测试数据来填充 TSI 环境的设备模拟解决方案：

1. 在单独的窗口/选项卡中转到 [azureiotsolutions.com](https://www.azureiotsolutions.com)。 使用相同的 Azure 订阅帐户登录，然后选择“设备模拟”加速器。

   [![运行设备模拟加速器](media/tutorial-create-populate-tsi-environment/sa-main.png)](media/tutorial-create-populate-tsi-environment/sa-main.png#lightbox)

1. 在“创建设备模拟”解决方案页上输入所需的参数。

   参数|说明
   ---|---
   **解决方案名称** | 用于创建新资源组的唯一值。 将会创建列出的 Azure 资源并将其分配到资源组。
   **订阅** | 指定用于创建 TSI 环境（参阅上一部分）的同一订阅。
   **区域** | 指定用于创建 TSI 环境（参阅上一部分）的同一区域。
   **部署可选 Azure 资源** | 将“IoT 中心”保留选中状态，因为模拟设备将使用它来建立连接/流式传输数据。

   完成后，单击“创建解决方案”预配解决方案的 Azure 资源。 此过程可能需要 6-7 分钟才能完成。

   [![预配设备模拟解决方案](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution.png#lightbox)

1. 完成预配后，新解决方案上方的文本将从“正在预配…”更改为“就绪”：

   >[!IMPORTANT]
   > 暂时不要单击“启动”按钮！ 请将此网页保持打开状态，因为稍后需要返回此处。

   [![设备模拟解决方案预配完成](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard-ready.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard-ready.png#lightbox)

1. 现在，请返回 Azure 门户并检查订阅中新建的资源。 在门户的“资源组”页上，可以看到使用最后一个步骤中提供的“解决方案名称”创建了一个新资源组。 另外可以看到，已创建用于支持设备模拟解决方案的所有资源：

   [![设备模拟解决方案资源](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png)](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png#lightbox)

## <a name="connect-the-tsi-environment-to-the-iot-hub"></a>将 TSI 环境连接到 IoT 中心

现在，我们已了解如何创建两组资源，各自位于其所在的资源组中：

- 一个空的 TSI 环境。
- 设备模拟解决方案资源，包括解决方案加速器生成的 IoT 中心。

前文提到，模拟设备需要连接到 IoT 中心以流式传输设备数据。 若要将数据流式传输到 TSI 环境，需要对 IoT 中心和 TSI 环境进行配置更改。

### <a name="iot-hub-configuration-define-a-consumer-group"></a>IoT 中心配置：定义使用者组

IoT 中心提供多个终结点用于与其他执行组件共享功能。 将数据流式传输到 IoT 中心实例时，其他应用程序可以通过“事件”终结点使用数据。 具体而言，“使用者组”提供一种机制，让应用程序从 IoT 中心侦听和提取数据。

接下来，在设备模拟解决方案的 IoT 中心“事件”终结点上定义一个新的“使用者组”属性。

1. 在 Azure 门户中，转到为设备模拟解决方案创建的资源组的“概述”页，然后选择 IoT 中心资源：

   [![设备模拟解决方案资源组](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png#lightbox)

   另请记下针对解决方案生成的 IoT 中心资源的“名称”，因为稍后需要引用它。

1. 向下滚动并选择“终结点”页，然后选择“事件”终结点。 在终结点的“属性”页上，输入终结点在“$Default”使用者组中的唯一名称，然后单击“保存”：

   [![设备模拟解决方案 IoT 中心终结点](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png#lightbox)

### <a name="tsi-configuration-define-an-event-source"></a>TSI 配置：定义事件源

现在，将新的 IoT 中心“使用者组”事件终结点作为“事件源”连接到 TSI 环境。

1. 转到针对 TSI 环境创建的资源组的“概述”页，然后选择 TSI 环境：

   [![TSI 环境资源组和环境](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png#lightbox)

1. 在 TSI 环境页上选择“事件源”，然后单击“+ 添加”。

   [![TSI 环境概述](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add.png#lightbox)

1. 在“新建事件源”页上输入所需的参数。

   参数|说明
   ---|---
   **事件源名称** | 必须输入用于命名事件源的唯一值。
   **源** | 选择“IoT 中心”。
   **导入选项** | 选择默认的 `Use IoT hub from available subscriptions`。 使用此选项会在接下来显示的下拉列表中填充可用的订阅。
   **订阅** | 选择在其中创建了 TSI 环境和设备模拟资源的同一订阅。
   **IoT 中心名称** | 默认值应是前面记下的 IoT 中心名称。 如果不是，请选择正确的 IoT 中心。
   **IoT 中心策略名称** | 选择“iothubowner”。
   **IoT 中心使用者组** | 默认值应是前面创建的 IoT 中心使用者组的名称。 如果不是，请选择正确的使用者组名称。
   **事件序列化格式** | 保留为默认值 `JSON`。
   **时间戳属性名称** | 指定为 `timestamp`。

   完成后，单击“创建”以添加事件源。 返回资源组的“概述”页时，会看到新的“时序见解事件源”资源以及 TSI 环境资源。

   [![TSI 环境的新事件源](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add-event-source.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add-event-source.png#lightbox)

## <a name="run-device-simulation-to-stream-data-into-tsi"></a>运行设备模拟以将数据流式传输到 TSI

现已完成所有配置工作，接下来可以使用模拟设备中的示例数据填充 TSI 环境。

回顾[“创建设备模拟”部分](#create-a-device-simulation)，其中提到加速器已创建多个 Azure 资源来支持解决方案。 除了前面所述的 IoT 中心以外，我们还生成了一个 Azure 应用服务 Web 应用程序来创建和传输模拟设备遥测数据。

1. 返回到[解决方案加速器仪表板](https://www.azureiotsolutions.com/Accelerators#dashboard)。 根据需要，使用本教程中所用的同一 Azure 帐户重新登录。 现在，可以单击“设备模拟”解决方案下的“启动”按钮。

     [![解决方案加速器仪表板](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard.png#lightbox)

1. 设备模拟 Web 应用此时将会启动，初始加载可能需要花费几秒时间。 系统还会提示你提供许可，以授予 Web 应用程序“登录并读取你的个人资料”的权限。 此权限使得应用程序能够检索所需的用户个人资料信息来支持运行应用程序。

     [![设备模拟 Web 应用程序许可](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png)](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png#lightbox)

1. 加载“模拟设置”页后，输入所需的参数。

   参数|说明
   ---|---
   **目标 IoT 中心** | 选择“使用预配的 IoT 中心”。
   **设备型号** | 选择“冷却器”。
   **设备数目**  | 在“数量”下面输入 `1000`。
   **遥测频率** | 输入 `10` 秒。
   **模拟持续时间** | 选择“结束时间:”并输入 `5` 分钟。

   完成后，单击“开始模拟”。 模拟总共会运行 5 分钟，每隔 10 秒从 1000 台模拟设备生成数据。  

   [![设备模拟设置](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png#lightbox)

1. 运行模拟时，会看到“消息总数”和“每秒消息数”字段大约每隔 10 秒会更新。 模拟将在大约 5 分钟后结束，随后你会返回“模拟设置”。

   [![设备模拟正在运行](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png#lightbox)

## <a name="verify-the-telemetry-data"></a>验证遥测数据

在此最后一个部分，我们将验证 TSI 环境中生成和存储的遥测数据。 若要验证数据，请使用时序见解资源管理器，它可以用于查询和分析遥测数据。

1. 返回到 TSI 环境资源组的“概述”页。 选择 TSI 环境。

   [![TSI 环境资源组和环境](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png#lightbox)

1. 在 TSI 环境的“概述”页上，单击“时序见解资源管理器 URL”打开 TSI 资源管理器。

   [![TSI 资源管理器](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png#lightbox)

1. TSI 资源管理器将会加载，并使用 Azure 门户帐户进行身份验证。 初始视图出现后，可以在图表区域中看到，TSI 环境中确实填充了模拟的遥测数据。 若要根据更小的时间范围进行筛选，请选择左上角的下拉列表。 然后输入足够大的时间范围，以涵盖设备模拟的整个持续时间。 然后单击搜索放大镜。

   [![TSI 资源管理器时间范围筛选器](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png#lightbox)

1. 缩小时间范围可在图表中放大 IoT 中心和 TSI 环境中的不同数据传输高峰期。 另请注意右上角的“流式处理完成”文本，其中显示了找到的事件总数。 还可以拖动“间隔大小”滑块来控制图表上的绘图粒度。

   [![TSI 资源管理器时间范围筛选视图](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png#lightbox)

1. 最后，还可以左键单击某个区域以筛选范围，然后单击右键并使用“浏览事件”，在表格式“事件”视图中显示事件详细信息。

   [![TSI 资源管理器时间范围筛选视图和事件](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png#lightbox)

## <a name="clean-up-resources"></a>清理资源

本教程创建了多个正常运行的 Azure 服务来支持 TSI 环境和设备模拟解决方案。 如果你想要放弃和/或延期完成本系列教程，我们建议删除所有资源，以免产生不必要的费用。

在 Azure 门户的左侧菜单中：

1. 单击“资源组”图标，然后选择针对 TSI 环境创建的资源组。 在页面顶部单击“删除资源组”，输入资源组的名称，然后单击“删除”。

1. 单击“资源组”图标，然后选择设备模拟解决方案加速器创建的资源组。 在页面顶部单击“删除资源组”，输入资源组的名称，然后单击“删除”

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 创建 TSI 环境 
> * 创建包含 IoT 中心的设备模拟解决方案
> * 将 TSI 环境连接到 IoT 中心
> * 运行设备模拟以将数据流式传输到 TSI 环境
> * 验证模拟的遥测数据

了解如何创建自己的 TSI 环境后，请学习如何生成可以使用 TSI 环境中的数据的 Web 应用程序。

> [!div class="nextstepaction"]
> [创建 Azure 时序见解单页 Web 应用](tutorial-create-tsi-sample-spa.md)