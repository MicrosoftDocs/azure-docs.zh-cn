---
title: 适用于 Visual Studio 的 Azure 流分析工具中的 Edge 作业
description: 本文介绍如何使用适用于 Visual Studio 的流分析工具创作、调试和创建流分析 Edge 作业。
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 684690baa63f5ccd65c69e3a1b7e310c2f809e59
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60762575"
---
# <a name="develop-stream-analytics-edge-jobs-using-visual-studio-tools"></a>使用 Visual Studio 工具开发流分析 Edge 作业

本教程介绍如何开发用于创建和调试流分析 Edge 作业的、适用于 Visual Studio 的流分析工具。 创建并测试作业后，可以转到 Azure 门户，将该作业部署到设备中。 

## <a name="prerequisites"></a>必备组件

若要完成本教程，需要具备以下先决条件：

* 安装 [Visual Studio 2017](https://www.visualstudio.com/downloads/)、[Visual Studio 2015](https://www.visualstudio.com/vs/older-downloads/) 或 [Visual Studio 2013 Update 4](https://www.microsoft.com/download/details.aspx?id=45326)。 支持 Enterprise (Ultimate/Premium)、Professional、Community 版本。 不支持 Express 版本。  

* 请按照[安装说明](stream-analytics-tools-for-visual-studio-edge-jobs.md)安装用于 Visual Studio 的流分析工具。
 
## <a name="create-a-stream-analytics-edge-project"></a>创建流分析 Edge 项目 

在 Visual Studio 中，选择“文件” > “新建” > “项目”。 导航到左侧的“模板”列表，展开“Azure 流分析” > “流分析 Edge” > “Azure 流分析 Edge 应用程序”。 提供项目的名称、位置和解决方案名称，选择“确定”。

![Visual Studio 中的新 Edge 项目](./media/stream-analytics-tools-for-visual-studio-edge-jobs/new-stream-analytics-edge-project.png)

创建项目后，导航到“解决方案资源管理器”查看文件夹层次结构。

![流分析 Edge 作业的解决方案资源管理器视图](./media/stream-analytics-tools-for-visual-studio-edge-jobs/edge-project-in-solution-explorer.png)

 
## <a name="choose-the-correct-subscription"></a>选择正确的订阅

1. 在 Visual Studio 的“视图”菜单中，选择“服务器资源管理器”。  

2. 右键单击“Azure”并选择“连接到 Microsoft Azure 订阅”，然后使用你的 Azure 帐户进行登录。

## <a name="define-inputs"></a>定义输入

1. 在“解决方案资源管理器”中展开“输入”节点，应会看到名为 **EdgeInput.json** 的输入。 双击该输出以查看其设置。  

2. 将“源类型”设置为“数据流”。 然后将“源”设置为“Edge 中心”，将“事件序列化格式”设置为“Json”，将“编码”设置为“UTF8”。 （可选）可以重命名“输入别名”。对于本示例，我们将其保留原样。 如果重命名了输入别名，请在定义查询时使用重命名后的名称。 选择“保存”，保存这些设置。  
   ![流分析作业输入配置](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-input-configuration.png)
 


## <a name="define-outputs"></a>定义输出

1. 在“解决方案资源管理器”中展开“输出”节点，应会看到名为 **EdgeOutput.json** 的输出。 双击该输出以查看其设置。  

2. 确保“接收器”设置为“Edge 中心”，“事件序列化格式”设置为“Json”，“编码”设置为“UTF8”，“格式”设置为“数组”。 （可选）可以重命名“输出别名”。对于本示例，我们将其保留原样。 如果重命名了输出别名，请在定义查询时使用重命名后的名称。 选择“保存”，保存这些设置。 
   ![流分析作业输出配置](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-output-configuration.png)
 
## <a name="define-the-transformation-query"></a>定义转换查询

Edge 环境中部署的流分析作业支持大多数[流分析查询语言参考](https://msdn.microsoft.com/azure/stream-analytics/reference/stream-analytics-query-language-reference?f=255&MSPPError=-2147217396)，但是，Edge 作业尚不支持以下操作： 


|**类别**  | **命令**  |
|---------|---------|
|地理空间运算符 |<ul><li>CreatePoint</li><li>CreatePolygon</li><li>CreateLineString</li><li>ST_DISTANCE</li><li>ST_WITHIN</li><li>ST_OVERLAPS</li><li>ST_INTERSECTS</li></ul> |
|其他运算符 | <ul><li>分区依据</li><li>TIMESTAMP BY OVER</li><li>DISTINCT</li><li>COUNT 运算符中的表达式参数</li><li>DATE 和 TIME 函数中的微秒</li><li>JavaScript UDA（对于云中部署的作业，此功能仍以预览版提供）</li></ul>   |

在门户中创建 Edge 作业时，如果未使用支持的运算符，编译器会自动发出警告。

在 Visual Studio 的查询编辑器中定义以下转换查询（**script.asaql 文件**）

```sql
SELECT * INTO EdgeOutput
FROM EdgeInput 
```

## <a name="test-the-job-locally"></a>在本地测试作业

若要在本地测试查询，应上传示例数据。 可以通过从 [GitHub 存储库](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/Registration.json)下载注册数据来获取示例数据，并将其保存到本地计算机。 

1. 若要上传示例数据，请右键单击 **EdgeInput.json** 文件并选择“添加本地输入”  

2. 在弹出窗口中，**浏览**本地路径中的示例数据，并选择“保存”。
   ![Visual Studio 中的本地输入配置](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-local-input-configuration.png)
 
3. 名为 **local_EdgeInput.json** 的文件会自动添加到输入文件夹。  
4. 可以在本地运行此文件，或将其提交到 Azure。 若要测试查询，请选择“本地运行”。  
   ![Visual Studio 中的流分析作业运行选项](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-visual-stuidio-run-options.png)
 
5. 命令提示符窗口会显示作业的状态。 如果作业运行成功，则会在项目文件夹路径“Visual Studio 2015\Projects\MyASAEdgejob\MyASAEdgejob\ASALocalRun\2018-02-23-11-31-42”中创建类似于“2018-02-23-11-31-42”的文件夹。 导航到文件夹路径，查看本地文件夹中的结果：

   也可以登录到 Azure 门户并检查是否已创建该作业。 

   ![流分析作业结果文件夹](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-job-result-folder.png)

## <a name="submit-the-job-to-azure"></a>将作业提交到 Azure

1. 在将作业提交到 Azure 之前，必须连接到 Azure 订阅。 打开“服务器资源管理器”，右键单击“Azure”并选择“连接到 Microsoft Azure 订阅”，其后登录到 Azure 订阅。  

2. 若要将作业提交到 Azure，请导航到查询编辑器并选择“提交到 Azure”。  

3. 此时会打开一个弹出窗口，可在其中选择更新现有的或创建新的 Edge 作业。 更新现有作业时，会替换所有作业配置，在这种情况下，需要发布新作业。 选择“创建新的 Azure 流分析作业”，为作业输入类似于 **MyASAEdgeJob** 的名称，选择所需的**订阅**、**资源组**和**位置**，然后选择“提交”。

   ![从 Visual Studio 将流分析作业提交到 Azure](./media/stream-analytics-tools-for-visual-studio-edge-jobs/submit-stream-analytics-job-to-azure.png)
 
   创建流分析 Edge 作业后，可以参阅[在 IoT Edge 中运行作业的教程](stream-analytics-edge.md)，了解如何将作业部署到设备。 

## <a name="manage-the-job"></a>管理作业 

可以通过服务器资源管理器查看作业和作业关系图。 从“服务器资源管理器”中的“流分析”中，展开你在其中部署了 Edge 作业的订阅和资源组。 你可以查看状态为“已创建”的 MyASAEdgejob。 展开作业节点，并双击该节点打开作业视图。

![服务器资源管理器作业管理选项](./media/stream-analytics-tools-for-visual-studio-edge-jobs/server-explorer-options.png)
 
作业视图窗口中提供了刷新作业、删除作业以及从 Azure 门户打开作业等操作。

![Visual Studio 中的作业关系图和其他选项](./media/stream-analytics-tools-for-visual-studio-edge-jobs/job-diagram-and-other-options.png) 

## <a name="next-steps"></a>后续步骤

* [有关 Azure IoT Edge 的详细信息](../iot-edge/about-iot-edge.md)
* [IoT Edge 教程上的 ASA ](../iot-edge/tutorial-deploy-stream-analytics.md)
* [使用此调查向团队发送反馈](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2czagZ-i_9Cg6NhAZlH9ypUMjNEM0RDVU9CVTBQWDdYTlk0UDNTTFdUTC4u) 
