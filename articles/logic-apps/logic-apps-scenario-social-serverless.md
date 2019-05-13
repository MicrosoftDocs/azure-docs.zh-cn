---
title: 无服务器方案 - 使用 Azure 服务创建客户见解仪表板 | Microsoft Docs
description: 使用 Azure 逻辑应用和 Azure Functions 构建客户仪表板，以便管理客户反馈、社交媒体数据等
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: estfan, LADocs
ms.assetid: d565873c-6b1b-4057-9250-cf81a96180ae
ms.topic: article
ms.date: 03/15/2018
ms.openlocfilehash: 5dd9d8eac7d65815d7c98a28b0d5af55f19cec47
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464432"
---
# <a name="create-streaming-customer-insights-dashboard-with-azure-logic-apps-and-azure-functions"></a>使用 Azure 逻辑应用和 Azure Functions 创建流式处理客户见解仪表板

Azure 提供[无服务器](https://azure.microsoft.com/solutions/serverless/)工具，可帮助在云中快速构建和托管应用程序，而无需考虑基础结构问题。 在本教程中，我们将创建一个仪表板，以便针对客户反馈触发相关操作、通过机器学习分析反馈，以及通过 Power BI 或 Azure Data Lake 之类的源发布见解。

对于此解决方案，我们将使用无服务器应用的以下关键 Azure 组件：[Azure Functions](https://azure.microsoft.com/services/functions/) 和 [Azure 逻辑应用](https://azure.microsoft.com/services/logic-apps/)。
Azure 逻辑应用在云中提供无服务器工作流引擎，以便能够跨无服务器组件提供业务流程，以及连接到 200 多个服务和 API。 Azure Functions 在云中提供无服务器计算。 此解决方案使用 Azure Functions 根据预定义的关键字来标记客户推文。

在此场景中，我们将会创建一个逻辑应用，在查找客户的反馈时，将会触发此应用。 部分有助于对客户反馈做出响应的连接器包括：Outlook.com、Office 365、Survey Monkey、Twitter，以及[来自 Web 窗体的 HTTP 请求](https://blogs.msdn.microsoft.com/logicapps/2017/01/30/calling-a-logic-app-from-an-html-form/)。 创建的工作流将会监视 Twitter 上的井号标签。

可以[在 Visual Studio 中生成整个解决方案](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)，并[使用 Azure 资源管理器模板部署该解决方案](../logic-apps/logic-apps-create-deploy-template.md)。 有关演示如何创建此解决方案的视频演练，请[观看此第 9 频道视频](https://aka.ms/logicappsdemo)。 

## <a name="trigger-on-customer-data"></a>基于客户数据的触发器

1. 在 Azure 门户或 Visual Studio 中创建一个空白的逻辑应用。 

   如果你不熟悉逻辑应用，请查看 [Azure 门户快速入门](../logic-apps/quickstart-create-first-logic-app-workflow.md)或 [Visual Studio 快速入门](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

2. 在逻辑应用设计器中，找到并添加可执行以下操作的 Twitter 触发器：**发布新推文时**

3. 将此触发器设置为根据关键字或井号标签侦听推文。

   在基于轮询的触发器（例如 Twitter 触发器）中，重复周期属性决定了逻辑应用以多高的频率检查新项。

   ![Twitter 触发器示例][1]

现在，会针对所有新推文激发此逻辑应用。 然后可以提取并分析推文数据，以便更好地了解所表达的情绪。 

## <a name="analyze-tweet-text"></a>分析推文文本

若要检测某些文本背后的情绪，可以使用 [Azure 认知服务](https://azure.microsoft.com/services/cognitive-services/)。

1. 在逻辑应用设计器中的触发器下，选择“新步骤”。

2. 找到“文本分析”连接器。

3. 选择“检测情感”操作。

4. 如果系统提示，请为文本分析服务提供有效的认知服务密钥。

5. 在“请求正文”下选择“推文文本”字段，提供推文作为要分析的文本。

获取推文数据以及有关推文的见解后，可以使用其他许多相关的连接器及其操作：

* **Power BI - 将行添加到流式处理数据集**：查看 Power BI 仪表板上的传入推文。
* **Azure Data Lake - 追加文件**：将客户数据添加到要包括在分析作业中的 Azure Data Lake 数据集。
* **SQL - 添加行**：将数据存储在数据库中，方便以后检索。
* **Slack - 发送消息**：告知 Slack 通道存在负面反馈，可能需要采取措施。

还可以创建 Azure 函数，以便针对数据执行自定义处理。 

## <a name="process-data-with-azure-functions"></a>使用 Azure Functions 处理数据

在创建函数之前，请在 Azure 订阅中创建一个函数应用。 此外，要让逻辑应用直接调用某个函数，该函数必须具有 HTTP 触发器绑定，例如，使用 **HttpTrigger** 模板。 了解[如何在 Azure 门户中创建第一个函数应用和函数](../azure-functions/functions-create-first-azure-function-azure-portal.md)。

对于此场景，请使用推文文本作为 Azure 函数的请求正文。 在函数代码中，定义用于确定推文文本是否包含某个关键字或短语的逻辑。 可以根据场景的需要来决定函数的简单或复杂程度。
在函数末尾，使用一些数据为逻辑应用返回响应，例如，该响应可以是简单的布尔值（如 `containsKeyword`）或复杂的对象。

> [!TIP]
> 若要从逻辑应用中的函数访问复杂响应，请使用“分析 JSON”操作。

完成后，请保存函数，然后将其作为操作添加到构建的逻辑应用中。

## <a name="add-azure-function-to-logic-app"></a>将 Azure 函数添加到逻辑应用

1. 在逻辑应用设计器中的“检测情绪”操作下，选择“新步骤”。

2. 找到“Azure Functions”连接器，选择创建的函数。

3. 在“请求正文”下，选择“推文文本”。

![配置的 Azure 函数步骤][2]

## <a name="run-and-monitor-your-logic-app"></a>运行和监视逻辑应用

若要查看逻辑应用的所有当前或以往运行，可以通过 Azure 门户、Visual Studio、Azure REST API 和 SDK 使用 Azure 逻辑应用提供的丰富调试和监视功能。

若要轻松测试逻辑应用，请在逻辑应用设计器中选择“运行触发器”。 触发器将会根据指定的计划轮询推文，直到找到满足条件的推文。 当运行正在进行时，设计器会显示该运行的实时视图。

在 Visual Studio 或 Azure 门户中查看以往运行的历史记录： 

* 打开 Visual Studio Cloud Explorer。 找到自己的逻辑应用，打开该应用的快捷菜单。 选择“打开运行历史记录”。

  > [!NOTE]
  > 有关 Visual Studio 2019，云资源管理器可以打开逻辑应用设计器在 Azure 门户中，但不能尚未打开嵌入的逻辑应用设计器或运行历史记录。

* 在 Azure 门户中，找到自己的逻辑应用。 在逻辑应用的菜单中，选择“概述”。 

## <a name="create-automated-deployment-templates"></a>创建自动部署模板

创建逻辑应用解决方案后，可以捕获该应用，并将其作为 [Azure 资源管理器模板](../azure-resource-manager/resource-group-overview.md#template-deployment)部署到世界上的任意 Azure 区域。 可以使用此功能来修改参数，以便创建不同的应用版本，以及将解决方案集成到 Azure Pipelines 中。 还可以在部署模板中包含 Azure Functions，以便将整个解决方案和所有依赖项作为单个模板进行管理。 了解[如何创建逻辑应用部署模板](../logic-apps/logic-apps-create-deploy-template.md)。

[Azure 快速入门模板存储库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-function-app-create-dynamic)中提供了包含 Azure 函数的部署模板示例。

## <a name="next-steps"></a>后续步骤

* [查找 Azure 逻辑应用的其他示例和方案](logic-apps-examples-and-scenarios.md)

<!-- Image References -->
[1]: ./media/logic-apps-scenario-social-serverless/twitter.png
[2]: ./media/logic-apps-scenario-social-serverless/function.png
