---
title: Azure 数据资源管理器的 azure DevOps 任务
description: 在本主题中，您了解如何创建发布管道和部署
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/05/2019
ms.openlocfilehash: a70a887ccb19d9c1cbdb5f8ebf6aa8d4b25a0dfd
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65161065"
---
# <a name="azure-devops-task-for-azure-data-explorer"></a>Azure 数据资源管理器的 azure DevOps 任务

[Azure DevOps 服务](https://azure.microsoft.com/services/devops/)提供高性能管道、 免费专用 Git 存储库，可配置看板和广泛的自动化和连续测试功能等的协作工具的开发。 [Azure 管道](https://azure.microsoft.com/services/devops/pipelines/)是 Azure DevOps 功能，使你能够管理 CI/CD 与高性能管道将代码部署采用任何语言、 平台和云了。
[Azure 数据资源管理器-管理员命令](https://marketplace.visualstudio.com/items?itemName=Azure-Kusto.PublishToADX)是 Azure 管道任务，可用于创建发布管道并将其部署数据库更改为你的 Azure 数据资源管理器数据库。 它是免费提供[Visual Studio Marketplace](https://marketplace.visualstudio.com/)。

本文档介绍一个简单的示例使用**Azure 数据资源管理器 – 管理员命令**任务来部署您的架构更改为你的数据库。 有关完整的 CI/CD 管道，请参阅[Azure DevOps 文档](/azure/devops/user-guide/what-is-azure-devops?view=azure-devops#vsts)。

## <a name="prerequisites"></a>必备组件

* 如果还没有 Azure 订阅，可以在开始前创建一个[免费 Azure 帐户](https://azure.microsoft.com/free/)。
* Azure 数据资源管理器群集设置：
    * [Azure 数据资源管理器群集和数据库](/azure/data-explorer/create-cluster-database-portal)
    * 创建由 Azure Active Directory (Azure AD) 应用[预配 Azure AD 应用程序](/azure/kusto/management/access-control/how-to-provision-aad-app)。
    * 向 Azure AD 应用的 Azure 数据资源管理器数据库上授予访问权限[管理 Azure 数据资源管理器数据库权限](/azure/data-explorer/manage-database-permissions)。
* Azure DevOps 设置：
    * [注册免费的组织](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)
    * [创建组织](/azure/devops/organizations/accounts/create-organization?view=azure-devops)
    * [Azure DevOps 中创建项目](/azure/devops/organizations/projects/create-project?view=azure-devops)
    * [使用 Git 的代码](/azure/devops/user-guide/code-with-git?view=azure-devops)

## <a name="create-folders"></a>创建文件夹

创建以下示例文件夹 (*函数*，*策略*，*表*) 在 Git 存储库。 中的文件复制[此处](https://github.com/Azure/azure-kusto-docs-samples/tree/master/DevOps_release_pipeline)到各自的文件夹显示以下并提交所做的更改。 提供了示例文件，以执行下面的工作流。

![创建文件夹](media/devops/create-folders.png)

> [!TIP]
> 在创建您自己的工作流，我们建议使代码幂等。 例如，使用[.create 合并表](/azure/kusto/management/tables#create-merge-tables)而不是[.create 表](/azure/kusto/management/tables#create-table)，并使用[.create 或 alter](/azure/kusto/management/functions#create-or-alter-function)函数而不是[.create](/azure/kusto/management/functions#create-function)函数。

## <a name="create-a-release-pipeline"></a>创建发布管道

1. 登录到您[Azure 开发运营组织](https://dev.azure.com/)。
1. 选择**管道** > **版本**从左侧菜单，然后选择**新管道**。

    ![新建管道](media/devops/new-pipeline.png)

1. **新建发布管道**窗口随即打开。 在中**管道**选项卡上，在**选择一个模板**窗格中，选择**空作业**。

     ![选择模板](media/devops/select-template.png)

1. 选择**阶段**按钮。 在中**阶段**窗格中，添加**阶段名称**。 选择**保存**以保存你的管道。

    ![阶段名称](media/devops/stage-name.png)

1. 选择**将项目添加**按钮。 在中**将项目添加**窗格中，选择你的代码所在的存储库，填写相关信息，然后单击**添加**。 选择**保存**以保存你的管道。

    ![添加项目](media/devops/add-artifact.png)

1. 在中**变量**选项卡上，选择 **+ 添加**若要创建的变量**终结点 URL** ，将在任务中使用。 编写**名称**并**值**终结点。 选择**保存**以保存你的管道。 

    ![创建变量](media/devops/create-variable.png)

    若要查找你 Endpoint_URL、 的概述页你**Azure 数据资源管理器群集**门户在 Azure 中包含 Azure 数据资源管理器群集 URI。 构造采用以下格式的 URI `https://<Azure Data Explorer cluster URI>?DatabaseName=<DBName>`。  例如： https://kustodocs.westus.kusto.windows.net?DatabaseName=SampleDB

    ![Azure 数据资源管理器群集 URI](media/devops/adx-cluster-uri.png)

## <a name="create-tasks-to-deploy"></a>创建要部署的任务

1. 在中**管道**选项卡上，单击**1 个作业，0 任务**添加任务。 

    ![添加任务](media/devops/add-task.png)

1. 创建要部署的三个任务**表**，**函数**，并**策略**，按此顺序。 

1. 在中**任务**选项卡上，选择**+** 通过**代理作业**。 搜索“Azure 数据资源管理器”。 在中**Marketplace**，安装**Azure 数据资源管理器 – 管理员命令**扩展。 然后，选择**外**中**运行 Azure 数据资源管理器命令**。

     ![添加管理命令](media/devops/add-admin-commands.png)

1. 单击**Kusto 命令**左侧和更新该任务的以下信息：
    * **显示名称**：任务的名称
    * **文件路径**:在中**表**任务中，指定 */Tables/*.csl 由于表在创建文件放入*表*文件夹。
    * **终结点 URL**： 输入`EndPoint URL`在上一步中创建的变量。
    * 选择**使用服务终结点**，然后选择 **+ 新建**。

    ![更新 Kusto 命令任务](media/devops/kusto-command-task.png)

1. 完成中的以下信息**添加 Azure 数据资源管理器服务连接**窗口：

    |设置  |建议的值  |
    |---------|---------|
    |**连接名称**     |    输入一个名称以标识此服务终结点     |
    |**群集 Url**    |    可以在 Azure 数据资源管理器群集在 Azure 门户中的概述部分中找到值 | 
    |**服务主体 Id**    |    输入 AAD 应用程序 ID （根据先决条件创建）     |
    |**服务主体应用密钥**     |    输入 AAD 应用程序键 （根据先决条件创建）    |
    |**AAD 租户 Id**    |      输入你的 AAD 租户 （例如 microsoft.com，contoso.com...)    |

    选择**允许使用此连接的所有管道**复选框。 选择“确定”。

    ![添加服务连接](media/devops/add-service-connection.png)

1. 重复步骤 1-5 另两次以将文件部署*函数*并*策略*文件夹。 选择“保存”。 在中**任务**选项卡上，请参阅创建的三个任务：**部署表**，**部署 Functions**，和**部署策略**。

    ![部署的所有文件夹](media/devops/deploy-all-folders.png)

1. 选择 **+ 发布** > **创建发布**若要创建的发布。

    ![创建发布](media/devops/create-release.png)

1. 在中**日志**选项卡上，检查部署状态为成功。

    ![部署成功](media/devops/deployment-successful.png)

现在已完成发布管道部署到预生产环境的三个任务的创建。