---
title: "使用 Azure IoT Edge 部署 Azure 流分析 | Microsoft 文档"
description: "将 Azure 流分析作为模块部署到 Edge 设备"
services: iot-edge
keywords: 
author: msebolt
manager: timlt
ms.author: v-masebo
ms.date: 11/28/2017
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: 5c9231f400b00f455f30692c58f007bd63d3cc76
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2017
---
# <a name="deploy-azure-stream-analytics-as-an-iot-edge-module---preview"></a>将 Azure 流分析作为 IoT Edge 模块进行部署 - 预览版

IoT 设备可以生成大量的数据。 为了减少上传的数据量或消除可操作见解的往返延迟，有时必须在数据抵达云中之前对其进行分析或处理。

Azure IoT Edge 利用预建的 Azure 服务 IoT Edge 模块进行快速部署。 [Azure 流分析][azure-stream]就是这样一个模块。 可以从相应的门户中创建 Azure 流分析作业，然后转到 Azure IoT 中心门户将其部署为 IoT Edge 模块。 

Azure 流分析提供一种丰富结构化的查询语法，可用于在云和 IoT Edge 设备上进行数据分析。 有关 IoT Edge 上的 Azure 流分析的详细信息，请参阅 [Azure 流分析文档](../stream-analytics/stream-analytics-edge.md)。

本教程逐步讲解如何创建 Azure 流分析作业并将其部署到 IoT Edge 设备。 这样，便可以直接在设备上处理本地遥测数据流，并生成警报来驱动设备上的即时操作。 

本教程演示两个模块： 
* 模拟温度传感器模块 (tempSensor)：可生成 20 到 120 度的温度数据（每 5 秒递增 1 度）。 
* 流分析模块：当 30 秒平均值达到 70 时，会重置 tempSensor。 在生产环境中，当温度达到危险级别时，可以使用此功能关闭机器或采取预防措施。 

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 在 Edge 上创建 Azure 流分析作业用于处理数据。
> * 将新的 Azure 流分析作业与其他 IoT Edge 模块相连接。
> * 将 Azure 流分析作业部署到 IoT Edge 设备。

## <a name="prerequisites"></a>先决条件

* 一个 IoT 中心。 
* 有关在 [Windows][lnk-tutorial1-win] 和 [Linux][lnk-tutorial1-lin] 中的模拟设备上部署 Azure IoT Edge 的快速入门和文章中创建并配置的设备。 需要知道设备连接密钥和设备 ID。 
* IoT Edge 设备上运行的 Docker。
    * [在 Windows 上安装 Docker][lnk-docker-windows]。
    * [在 Linux 上安装 Docker][lnk-docker-linux]。
* IoT Edge 设备上的 Python 2.7.x。
    * [在 Windows 上安装 Python 2.7][lnk-python]。
    * 大多数 Linux 发行版（包括 Ubuntu）均已安装 Python 2.7。 若要确保安装 pip，请使用以下命令：`sudo apt-get install python-pip`。

## <a name="create-an-azure-stream-analytics-job"></a>创建 Azure 流分析作业

本部分的步骤将创建一个 Azure 流分析作业，用于从 IoT 中心提取数据、查询从设备发送的遥测数据，然后将结果转发到 Azure Blob 存储容器。 有关详细信息，请参阅[流分析文档][azure-stream]中的“概述”部分。 

### <a name="create-a-storage-account"></a>创建存储帐户

需要使用 Azure 存储帐户来提供一个在 Azure 流分析作业中用作输出的终结点。 本部分中的示例使用 Blob 存储类型。 有关详细信息，请参阅 [Azure 存储文档][azure-storage]中的“Blob”部分。

1. 在 Azure 门户中，转到“创建资源”，在搜索框输入“存储帐户”，选择“存储帐户 - Blob、文件、表、队列”。

2. 在“创建存储帐户”窗格中，输入存储帐户的名称，选择存储 IoT 中心的同一位置，然后选择“创建”。 请记下该名称供稍后使用。

    ![创建存储帐户][1]

3. 转到刚刚创建的存储帐户，选择“浏览 Blob”。 

4. 为 Azure 流分析模块创建一个新容器用于存储数据，将访问级别设置为“容器”，选择“确定”。

    ![存储设置][10]

### <a name="create-a-stream-analytics-job"></a>创建流分析作业

1. 在 Azure 门户中，转到“创建资源” > “物联网”，然后选择“流分析作业”。

2. 在“新建流分析作业”窗格中执行以下操作：

    a. 在“作业名称”框中键入作业名称。
    
    b. 在“宿主环境”下，选择“Edge”。
    
    c. 在剩余字段中使用默认值。

    > [!NOTE]
    > 目前，美国西部 2 区域不支持 IoT Edge 上的 Azure 流分析作业。 

3. 选择“创建” 。

4. 在所创建作业中的“作业拓扑”下，依次选择“输入”、“添加”。

5. 在“新建输入”窗格中执行以下操作：

    a. 在“输入别名”框中，输入 **temperature**。
    
    b. 在“源类型”框中，选择“数据流”。
    
    c. 在剩余字段中使用默认值。

   ![Azure 流分析输入](./media/tutorial-deploy-stream-analytics/asa_input.png)

6. 选择“创建” 。

7. 在“作业拓扑”下，依次选择“输出”、“添加”。

8. 在“新建输出”窗格中执行以下操作：

    a. 在“输出别名”框中，键入 **alert**。
    
    b. 在剩余字段中使用默认值。 
    
    c. 选择“创建” 。

   ![Azure 流分析输出](./media/tutorial-deploy-stream-analytics/asa_output.png)


9. 在“作业拓扑”下，选择“查询”，然后将默认文本替换为以下查询：

    ```sql
    SELECT  
        'reset' AS command 
    INTO 
       alert 
    FROM 
       temperature TIMESTAMP BY timeCreated 
    GROUP BY TumblingWindow(second,30) 
    HAVING Avg(machine.temperature) > 70
    ```

10. 选择“保存”。

## <a name="deploy-the-job"></a>部署作业

现已准备好在 IoT Edge 设备上部署 Azure 流分析作业。

1. 在 Azure 门户中的 IoT 中心内，转到“IoT Edge (预览版)”并打开 IoT Edge 设备的详细信息页。

2. 选择“设置模块”。  
    如果以前在此设备上部署了 tempSensor 模块，则它可以自动填充数据。 否则，请执行以下步骤添加该模块：

   a. 选择“添加 IoT Edge 模块”。

   b. 键入 **tempSensor** 作为名称。
    
   c. 输入 **microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview** 作为映像 URI。 

   d.单击“下一步”。 将其他设置保留不变。
   
   e.选择“保存”。

3. 若要添加 Azure 流分析 Edge 作业，请选择“导入 Azure 流分析 Azure IoT Edge 模块”。

4. 选择创建的订阅和 Azure 流分析 Edge 作业。 

5. 选择创建的订阅和存储帐户，然后选择“保存”。

    ![设置模块][6]

6. 复制此流分析模块的名称。 

    ![温度模块][11]

7. 若要配置路由，请选择“下一步”。

8. 将以下代码复制到 **Routes**。 将 _{moduleName}_ 替换为复制的模块名称：

    ```json
    {
        "routes": {                                                               
          "telemetryToCloud": "FROM /messages/modules/tempSensor/* INTO $upstream", 
          "alertsToCloud": "FROM /messages/modules/{moduleName}/* INTO $upstream", 
          "alertsToReset": "FROM /messages/modules/{moduleName}/* INTO BrokeredEndpoint(\"/modules/tempSensor/inputs/control\")", 
          "telemetryToAsa": "FROM /messages/modules/tempSensor/* INTO BrokeredEndpoint(\"/modules/{moduleName}/inputs/temperature\")" 
        }
    }
    ```

9. 选择“下一步”。

10. 在“审阅模板”步骤中，选择“提交”。

11. 返回到“设备详细信息”页，并选择“刷新”。  
    应会看到新的流分析模块与 IoT Edge 代理模块和 IoT Edge 中心在同时运行。

    ![模块输出][7]

## <a name="view-data"></a>查看数据

现在可以转到 IoT Edge 设备，检查 Azure 流分析模块与 tempSensor 模块之间的交互。

1. 检查所有模块是否在 Docker 中运行：

   ```cmd/sh
   docker ps  
   ```

   ![Docker 输出][8]

2. 查看所有系统日志和指标数据。 使用流分析模块名称：

   ```cmd/sh
   docker logs -f {moduleName}  
   ```

应该可以看到，机器温度逐渐升高，在 30 秒内达到了 70 度。 然后，流分析模块触发重置，机器温度下降到 21 度。 

   ![Docker 日志][9]


## <a name="next-steps"></a>后续步骤

在本教程中，我们配置了 Azure 存储容器和流分析作业来分析 IoT Edge 设备中的数据。 然后加载了自定义 Azure 流分析模块，通过流将数据从设备移入 Blob 以供下载。 若要了解 Azure IoT Edge 如何为企业创建更多解决方案，请继续学习其他教程。

> [!div class="nextstepaction"] 
> [将 Azure 机器学习模型作为一个模块部署][lnk-ml-tutorial]

<!-- Images. -->
[1]: ./media/tutorial-deploy-stream-analytics/storage.png
[4]: ./media/tutorial-deploy-stream-analytics/add_device.png
[5]: ./media/tutorial-deploy-stream-analytics/asa_job.png
[6]: ./media/tutorial-deploy-stream-analytics/set_module.png
[7]: ./media/tutorial-deploy-stream-analytics/module_output.png
[8]: ./media/tutorial-deploy-stream-analytics/docker_output.png
[9]: ./media/tutorial-deploy-stream-analytics/docker_log.png
[10]: ./media/tutorial-deploy-stream-analytics/storage_settings.png
[11]: ./media/tutorial-deploy-stream-analytics/temp_module.png


<!-- Links -->
[lnk-what-is-iot-edge]: what-is-iot-edge.md
[lnk-module-dev]: module-development.md
[iot-hub-get-started-create-hub]: ../../includes/iot-hub-get-started-create-hub.md
[azure-iot]: https://docs.microsoft.com/azure/iot-hub/
[azure-storage]: https://docs.microsoft.com/azure/storage/
[azure-stream]: https://docs.microsoft.com/azure/stream-analytics/
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-tutorial1-win]: tutorial-simulate-device-windows.md
[lnk-tutorial1-lin]: tutorial-simulate-device-linux.md
[lnk-module-tutorial]: tutorial-csharp-module.md
[lnk-ml-tutorial]: tutorial-deploy-machine-learning.md

[lnk-docker-windows]: https://docs.docker.com/docker-for-windows/install/ 
[lnk-docker-linux]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/
[lnk-python]: https://www.python.org/downloads/
