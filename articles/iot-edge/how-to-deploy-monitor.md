---
title: "部署并监控 Azure IoT Edge 的模块 | Microsoft Docs"
description: "管理在边缘设备上运行的模块"
services: iot-edge
keywords: 
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 12/05/2017
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: b507b9108dca2fd3aee4acdac231acad9c9154e8
ms.sourcegitcommit: cc03e42cffdec775515f489fa8e02edd35fd83dc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/07/2017
---
# <a name="deploy-and-monitor-iot-edge-modules-at-scale---preview"></a>大规模部署和监控 IoT Edge 模块 - 预览

使用 Azure IoT Edge 可以将分析移至边缘并提供云接口，以便能够无需实际访问每个设备即可管理和监控 IoT Edge 设备。 随着物联网解决方案日益庞大和复杂，远程管理设备功能变得越来越重要。 Azure IoT Edge 旨在支持业务目标，不管添加的设备数量。

可以管理单独的设备并一次性将模块部署到设备中。 但如果想要大规模更改设备，可以创建 IoT Edge 部署。 部署是允许同时将多个模块部署到多个设备、跟踪模块的状态和运行状况以及在必要时做出更改的动态过程。 

## <a name="identify-devices-using-tags"></a>使用标记标识设备

创建部署之前，必须能够指定想要影响的设备。 Azure IoT Edge 标识使用设备孪生中的标记标识设备。 每个设备均可以拥有多个标记，你可以用适合解决方案的任何方式定义它们。 例如，如果管理具有智能建筑的校园，可能要将以下标记添加到设备：

```json
"tags":{
    "location":{
        "building": "20",
        "floor": "2"
    },
    "roomtype": "conference",
    "environment": "prod"
}
```

有关设备孪生和标记的详细信息，请参阅[了解和使用 IoT 中心的设备孪生][lnk-device-twin]。

## <a name="create-a-deployment"></a>创建部署

1. 登录 [Azure 门户][lnk-portal]，并导航到 IoT 中心。 
1. 选择“IoT Edge (预览版)”。
1. 选择“添加 IoT Edge 部署”。

创建部署需要五个步骤。 下列各节将引导完成每个步骤。 

### <a name="step-1-name-and-label"></a>步骤 1：名称和标签

1. 为部署提供唯一名称。 避免空格和以下无效字符：`& ^ [ ] { } \ | " < > /`。
1. 添加用于帮助跟踪部署的标签。 标签是描述部署的“名称, 值”对。 例如 `HostPlatform, Linux` 或 `Version, 3.0.1`。
1. 选择“下一步”，进入到步骤 2。 

### <a name="step-2-add-modules-optional"></a>步骤 2：添加模块（可选）

可添加到部署的模块类型有两种。 第一种是基于 Azure 服务的模块，如存储帐户或 Azure 流分析。 第二种是基于自身代码的模块。 可以向部署添加任意一种类型的多个模块。 

如果创建不含模块的部署，则会删除设备中任何现有模块。 

>[!NOTE]
>Azure 机器学习和 Azure Functions 暂不支持自动 Azure 服务部署。 使用自定义模块部署将这些服务手动添加到部署中。 

若要添加 Azure 服务中的模块，请执行以下步骤：
1. 选择“添加 Azure 服务 IoT Edge 模块”。
1. 使用下拉列表菜单选择想要部署的 Azure 服务实例。
1. 选择“保存”将模块添加到部署。 

若要将自定义代码添加为模块，或手动添加 Azure 服务模块，请执行以下步骤：
1. 选择“添加自定义 IoT Edge 模块”。
1. 为模块提供一个名称。
1. 对于映像字段，输入此模块的 Docker 容器映像。
1. 使用“OS”和“构架”下的下拉菜单来标识代表此模块的 Docker 容器的属性。 
1. 指定应传递到容器的任意 Create 选项。 有关详细信息，请参阅 [docker create][lnk-docker-create]。
1. 使用下拉菜单选择“重启策略”。 从以下选项中选择： 
   * 始终 - 如果模块因任何原因关闭，该模块将始终重启。
   * 从不 - 如果模块因任何原因关闭，该模块将不再重启。
   * 运行故障时 - 如果模块发生故障，它将重启，但如果完全关闭，则不重启。 
   * 运行不正常时 - 如果模块发生故障或者返回不正常状态，它将重启。 这取决于每个执行运行状况状态功能的模块。 
1. 使用下拉菜单选择模块的启动状态。 从以下选项中选择：
   * 正在运行 - 这是默认选项。 该模块在部署之后将立即开始运行。
   * 已停止 - 部署之后，模块将保持空闲状态，直到你或另一个模块要求启动。
1. 如果想要将任意标记或所需属性添加到模块中，请选择“编辑模块孪生”。 
1. 选择“保存”，将模块添加到部署。 

配置完部署的所有模块之后，选择“下一步”，进入到步骤 3。

### <a name="step-3-specify-routes-optional"></a>步骤 3：指定路由（可选）

路由定义模块在部署中如何相互通信。 为部署指定任意路由，然后选择“下一步”，进入到步骤 4。 

### <a name="step-4-target-devices"></a>步骤 4：目标设备

使用设备中的标记属性将应接收此部署的特定设备定为目标。 

因为多个部署可能将同一个设备定为目标，所以应为每个部署提供优先级编号。 如果存在冲突，则将优先级最高的部署定为目标。 如果两个部署的优先级编号相同，则将最新创建的部署定为目标。 

1. 为部署优先级输入一个正整数。
1. 输入“目标条件”确定将作为此部署的目标的设备。 条件基于设备孪生标记，应与表达式格式相匹配。 例如，`tags.environment='test'`。 
1. 选择“下一步”，进入到最后一步。

### <a name="step-5-review-template"></a>步骤 5：审阅模板

审阅部署信息，然后选择“提交”。

## <a name="monitor-a-deployment"></a>监视部署

若要查看部署的详细信息并监视运行它的设备，请执行以下步骤：

1. 登录 [Azure 门户][lnk-portal]，并导航到 IoT 中心。 
1. 选择“IoT Edge (预览版)”。
1. 选择“IoT Edge 部署”。 

   ![查看 IoT Edge 部署][1]

1. 检查部署列表。 对于每个部署，可以查看以下详细信息：
   * ID - 部署的名称。
   * 目标条件 - 用于定义目标设备的标记。
   * 优先级 - 分配到部署的优先级编号。
   * IoT Edge 代理状态 - 接收到部署的设备数量及其运行状况。 
   * 不正常模块 - 部署报告错误中的模块的数量。 
   * 创建时间 - 开始创建部署的时间戳。 两个部署具有相同优先级，此时间戳用于消除它们的关系。 
1. 选择想要监视的部署。  
1. 检查部署详细信息。 可以使用选项卡查看有关接收到部署的设备的详细信息： 
   * 已定为目标 - 与目标条件相匹配的边缘设备。 
   * 已应用 - 未被其他具有更高优先级的部署定为目标的目标边缘设备。 这些是实际接收部署的设备。 
   * 报告成功 - 已应用的边缘设备向服务报告模块已成功部署。 
   * 报告失败 - 已应用的边缘设备向服务报告一个或多个模块未成功部署。 若要进一步调查错误，需要远程连接到这些设备并查看日志文件。 
   * 报告不正常模块 - 已应用的边缘设备向服务报告一个或多个模块未成功部署，但是现在正在报告错误。 

## <a name="modify-a-deployment"></a>修改部署

修改部署时，更改会立即复制到所有目标设备。 

如果更新目标条件，将发生以下更新：
* 如果设备虽然不满足旧的目标条件，但满足新的目标条件，且此部署是此设备的最高优先级，则此部署将应用到设备。 
* 如果当前运行此部署的设备不再满足目标条件，则它将卸载此部署并采用下一个最高优先级部署。 
* 如果当前运行此部署的设备不再满足目标条件且不满足任何其他部署的目标条件，则此设备上不会发生任何更改。 设备在当前状态下继续运行当前模块，但不再作为此部署的一部分被托管。 一旦它满足任何其他部署的目标条件，将卸载此部署并采用新的部署。 

若要修改部署，请执行以下步骤： 

1. 登录 [Azure 门户][lnk-portal]，并导航到 IoT 中心。 
1. 选择“IoT Edge (预览版)”。
1. 选择“IoT Edge 部署”。 

   ![查看 IoT Edge 部署][1]

1. 选择想要修改的部署。 
1. 更新以下字段： 
   * 目标条件 
   * 标签 
   * Priority 
1. 选择“保存”。
1. 执行[监视部署][anchor-monitor]中的步骤，注意更改的推出。 

## <a name="delete-a-deployment"></a>删除部署

删除部署时，任何设备都将采用下一个最高优先级部署。 如果设备不满足任何其他部署的目标条件，则删除该部署时不会删除模块。 

1. 登录 [Azure 门户][lnk-portal]，并导航到 IoT 中心。 
1. 选择“IoT Edge (预览版)”。
1. 选择“IoT Edge 部署”。 

   ![查看 IoT Edge 部署][1]

1. 使用复选框选择想要删除的部署。 
1. 选择“删除”。
1. 提示将发出以下通知：此操作将删除此部署并还原为所有设备之前的状态。  这意味着将应用具有较低优先级的部署。  如果没有将其他任何部署定为目标，则不会删除任何模块。 如果客户想要进行此操作，他们需要创建不含有模块的部署，并将其部署到相同设备。 如果希望继续，选择“是”。 

## <a name="next-steps"></a>后续步骤

了解有关[将模块部署到边缘设备][lnk-deployments]的详细信息。

<!-- Images -->
[1]: ./media/how-to-deploy-monitor/view-deployments.png

<!-- Links -->
[lnk-device-twin]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-portal]: https://portal.azure.com
[lnk-docker-create]: https://docs.docker.com/engine/reference/commandline/create/
[lnk-deployments]: module-deployment-monitoring.md

<!-- Anchor links -->
[anchor-monitor]: #monitor-a-deployment
