---
title: 将 Azure 机器学习部署到设备 - Azure IoT Edge | Microsoft Docs
description: 在本教程中，将创建 Azure 机器学习模型，然后将其作为一个模块部署到边缘设备
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/07/2019
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc, seodec18
ms.openlocfilehash: 6f85b0088fac97f4b9f2dd2835e3052cb598a987
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142764"
---
# <a name="tutorial-deploy-azure-machine-learning-as-an-iot-edge-module-preview"></a>教程：将 Azure 机器学习作为 IoT Edge 模块进行部署（预览版）

使用 Azure Notebooks 开发机器学习模型，并将其部署到运行 Azure IoT Edge 的 Linux 设备。 

可以使用 IoT Edge 模块部署代码，以直接将业务逻辑实现到 IoT Edge 设备。 本教程逐步演示如何部署 Azure 机器学习模块，以便根据模拟的机器温度数据预测设备故障时间。 有关 IoT Edge 上的 Azure 机器学习服务的详细信息，请参阅 [Azure 机器学习文档](../machine-learning/service/how-to-deploy-to-iot.md)。

本教程中创建的 Azure 机器学习模块将读取设备生成的环境数据，并将消息标记为异常或正常。

本教程介绍如何执行以下操作：

> [!div class="checklist"]
> * 创建 Azure 机器学习模块
> * 将模块容器推送到 Azure 容器注册表
> * 将 Azure 机器学习模块部署到 IoT Edge 设备
> * 查看生成的数据

>[!NOTE]
>Azure IoT Edge 上的 Azure 机器学习模块为公开预览版。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>先决条件

Azure IoT Edge 设备：

* 可以按照适用于 [Linux](quickstart-linux.md) 的快速入门中的步骤，将 Azure 虚拟机用作 IoT Edge 设备。
* Azure 机器学习模块不支持 Windows 容器。
* Azure 机器学习模块不支持 ARM 处理器。

云资源：

* Azure 中的免费或标准层 [IoT 中心](../iot-hub/iot-hub-create-through-portal.md)。
* Azure 机器学习工作区。 按照[通过 Azure 门户开始使用 Azure 机器学习](../machine-learning/service/quickstart-get-started.md)中的说明创建一个工作区并了解如何使用它。
   * 请记下工作区名称、资源组和订阅 ID。 Azure 门户中的工作区概述中提供了这些值。 你将在本教程后面部分使用这些值，将 Azure 笔记本连接到工作区资源。 


## <a name="create-and-deploy-azure-machine-learning-module"></a>创建和部署 Azure 机器学习模块

在本部分中，将已训练的机器学习模型文件转换为 Azure 机器学习服务容器。 Docker 映像所需的所有组件都在 [Azure IoT Edge Git 存储库的 AI 工具包](https://github.com/Azure/ai-toolkit-iot-edge/tree/master/IoT%20Edge%20anomaly%20detection%20tutorial)中。 请按照下列步骤将该存储库上传到 Microsoft Azure Notebooks 中，以创建容器并将其推送到 Azure 容器注册表。


1. 导航到 Azure Notebooks 项目。 可以从 [Azure 门户](https://portal.azure.com)中的 Azure 机器学习服务工作区导航到该处，也可以通过使用 Azure 帐户登录到 [Microsoft Azure Notebooks](https://notebooks.azure.com/home/projects) 来这样做。

2. 选择“上传 GitHub 存储库”。

3. 提供以下 GitHub 存储库名称：`Azure/ai-toolkit-iot-edge`。 如果要将项目设为专用，请取消选中“公共”框。 选择“导入”。 

4. 导入完成后，导航到新的 ai-toolkit-iot-edge 项目，然后打开“IoT Edge 异常情况检测教程”文件夹。 

5. 验证项目是否正在运行。 如果没有，请选择“在免费计算上运行”。

   ![在免费计算上运行](./media/tutorial-deploy-machine-learning/run-on-free-compute.png)

6. 打开 aml_config/config.json 文件。

7. 编辑配置文件以包含以下值：Azure 订阅 ID、订阅中的资源组以及 Azure 机器学习服务工作区名称。 可以从 Azure 工作区的“概述”部分获取所有这些值。 

8. 保存 config 文件。

9. 打开 00-anomaly-detection-tutorial.ipynb 文件。

10. 出现提示时，选择“Python 3.6”内核，然后选择“设置内核”。

11. 根据注释中的说明编辑笔记本中的第一个单元格。 使用添加到配置文件的相同资源组、订阅 ID 和工作区名称。

12. 通过选择它们并选择“运行”或按下 `Shift + Enter` 来运行笔记本中的单元格。

    >[!TIP]
    >异常情况检测教程笔记本中的一些单元是可选的，因为它们创建了一些用户可能拥有或可能没有的资源，如 IoT 中心。 如果将现有资源信息放在第一个单元格中，则在运行创建新资源的单元格时会收到错误，因为 Azure 不会创建重复的资源。 这没什么关系，可以忽略错误或完全跳过这些可选部分。 

完成笔记本中的所有步骤后，就已经训练了一个异常情况检测模型，将其构建为 Docker 容器映像，并将该映像推送到 Azure 容器注册表。 然后，你测试了该模型，并最终将其部署到 IoT Edge 设备。 

## <a name="view-container-repository"></a>查看容器存储库

检查容器映像是否已成功创建并存储在与机器学习环境关联的 Azure 容器注册表中。 在上一节中使用的笔记本自动将容器映像和注册表凭据提供到 IoT Edge 设备，但你应该知道它们的存储位置，以便以后可以自行查找信息。 

1. 在 [Azure 门户](https://portal.azure.com)中，导航到 Azure 机器学习服务工作区。 

2. 概述部分列出了工作区详细信息及其相关资源。 选择“注册表”值，该值应为工作区名称，后跟随机数字。 

3. 在容器注册表中，选择“存储库”。 应会看到一个名为“tempanomalydetection”的存储库，它是通过在前面部分中运行的笔记本创建的。 

4. 选择“tempanomalydetection”。 应看到存储库中有一个标记：1。 

   现在你已知道注册表名称、存储库名称和标记，你知道了容器的完整映像路径。 映像路径如下所示 \<registry_name\>.azurecr.io/tempanomalydetection:1。 映像路径可用于将此容器部署到 IoT Edge 设备。 

5. 在容器注册表中，选择“访问密钥”。 应看到多个访问凭据，包括“登录服务器”以及管理员用户的“用户名”和“密码”。

   这些凭据可以包含在部署清单中，使 IoT Edge 设备能够从注册表中拉取容器映像。 

现在你已了解机器学习容器映像的存储位置。 下一部分详细介绍如何通过各种步骤来查看在 IoT Edge 设备上作为模块运行的容器。 

## <a name="view-generated-data"></a>查看生成的数据

可以查看由每个 IoT Edge 模块生成的消息，还可以查看传送到 IoT 中心的消息。

### <a name="view-data-on-your-iot-edge-device"></a>查看 IoT Edge 设备上的数据

在 IoT Edge 设备上，可以查看从每个单独的模块发送的消息。

可能需要使用提升权限的 `sudo` 来运行 `iotedge` 命令。 注销并重新登录到设备会自动更新权限。

1. 查看 IoT Edge 设备上的所有模块。

   ```cmd/sh
   iotedge list
   ```

2. 查看从特定设备发送的消息。 使用从上一命令的输出中获取的模块名称。

   ```cmd/sh
   iotedge logs <module_name> -f
   ```

### <a name="view-data-arriving-at-your-iot-hub"></a>查看到达 IoT 中心的数据

可以使用[用于 Visual Studio Code 的 Azure IoT 中心工具包扩展](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)（以前称为 Azure IoT 工具包扩展）查看 IoT 中心接收的设备到云消息。

以下步骤介绍如何设置 Visual Studio Code，以便监视到达 IoT 中心的设备到云消息。

1. 在 Visual Studio Code 中，选择“IoT 中心设备”。

2. 选择 **...**，然后从菜单中选择“设置 IoT 中心连接字符串”。

   ![设置 IoT 中心连接字符串](./media/tutorial-deploy-machine-learning/set-connection.png)

3. 在页面顶部打开的文本框中，输入你的 IoT 中心的 iothubowner 连接字符串。 你的 IoT Edge 设备应当会出现在“IoT 中心设备”列表中。

4. 再次选择 **...**，然后选择“开始监视 D2C 消息”。

5. 观察每五秒来自 tempSensor 的消息。 消息正文包含 machinelearningmodule 提供的名为 anomaly、值为 true 或 false 的属性。 如果模型成功运行，**AzureMLResponse** 属性包含值“OK”。

   ![消息正文中的 Azure 机器学习服务响应](./media/tutorial-deploy-machine-learning/ml-output.png)

## <a name="clean-up-resources"></a>清理资源

如果打算继续学习下一篇建议的文章，可以保留已创建的资源和配置，以便重复使用。 还可以继续使用相同的 IoT Edge 设备作为测试设备。

否则，可以删除本文中创建的本地配置和 Azure 资源，以避免收费。

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>后续步骤

在本教程中，可以部署由 Azure 机器学习提供支持的 IoT Edge 模块。 你可以继续查看任何其他教程，了解 Azure IoT Edge 可帮助你将数据转化为临界业务见解的其他方式。

> [!div class="nextstepaction"]
> [使用自定义视觉服务对图像进行分类](tutorial-deploy-custom-vision.md)

