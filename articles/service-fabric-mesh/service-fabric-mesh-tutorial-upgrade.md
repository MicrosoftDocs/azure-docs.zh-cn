---
title: 教程 - 升级 Azure Service Fabric 网格应用程序 | Microsoft Docs
description: 了解如何使用 Visual Studio 升级 Service Fabric 应用程序
services: service-fabric-mesh
documentationcenter: .net
author: dkkapur
manager: chakdan
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: azure-cli
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/29/2018
ms.author: dekapur
ms.custom: mvc, devcenter
ms.openlocfilehash: 23809abd06d626eb87e5d5d15d265f1769b97b66
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60809090"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>教程：了解如何使用 Visual Studio 升级 Service Fabric 应用程序

本教程是一个系列的第四部分，介绍如何直接从 Visual Studio 升级 Azure Service Fabric 网格应用程序。 升级包括代码更新和配置更新。 可以看到在 Visual Studio 中升级和发布的步骤是相同的。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 使用 Visual Studio 升级 Service Fabric 网格服务

在此系列教程中，你会学习如何：
> [!div class="checklist"]
> * [在 Visual Studio 中创建 Service Fabric 网格应用](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [调试在本地开发群集中运行的 Service Fabric 网格应用](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [部署 Service Fabric 网格应用](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * 升级 Service Fabric 网格应用
> * [清理 Service Fabric 网格资源](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>先决条件

在开始学习本教程之前：

* 如果尚未部署待办事项应用，请按照[发布 Service Fabric 网格 Web 应用程序](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)中的说明操作。

## <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>使用 Visual Studio 升级 Service Fabric 网格服务

本文介绍如何在应用程序中升级微服务。 在本示例中，我们将修改 `WebFrontEnd` 服务以显示任务类别并增加给定的 CPU 量。 然后，升级已部署的服务。

## <a name="modify-the-config"></a>修改配置

创建 Service Fabric 网格应用时，Visual Studio 将为每个部署环境（云和本地）添加 parameters.yaml 文件。 在这些文件中，可以定义参数以及随后可从网格 *.yaml 文件（如 service.yaml 或 network.yaml）中引用的值。  Visual Studio 提供了一些变量，例如，可供服务使用的 CPU 量。

我们将更新 `WebFrontEnd_cpu` 参数以将 CPU 资源更新为 `1.5`，预计将更广泛地使用 WebFrontEnd 服务。

1. 在 todolistapp 项目中，在“环境” > “云”下打开 parameters.yaml 文件。 将 `WebFrontEnd_cpu` 值修改为 `1.5`。 参数名称以服务名称 `WebFrontEnd_` 开头，这是将其与应用于不同服务的同名参数进行区分的最佳做法。

    ```xml
    WebFrontEnd_cpu: 1.5
    ```

2. 在“WebFrontEnd” > “服务资源”下打开 WebFrontEnd 项目的 service.yaml 文件。

    请注意，在 `resources:` 部分中，`cpu:` 将设置为 `"[parameters('WebFrontEnd_cpu')]"`。 如果正在为云生成项目，则将从“环境” > “云” > “parameters.yaml”文件中获取 `'WebFrontEnd_cpu` 的值，并且该值将为 `1.5`。 如果正在生成要本地运行的项目，则将从“环境” > “本地” > “parameters.yaml”文件中获取值，并且该值将为“0.5”。

> [!Tip]
> 默认情况下，与 profile.yaml 文件对等的参数文件将用于提供该 profile.yaml 文件的值。
> 例如，“环境”>“云”>“parameters.yaml”提供了“环境”>“云”>“profile.yaml”的参数值。
>
> 可以通过将以下内容添加到 profile.yaml 文件进行覆盖：`parametersFilePath=”relative or full path to the parameters file”`例如， 或 `parametersFilePath=”..\CommonParameters.yaml”`

## <a name="modify-the-model"></a>修改模型

要引入代码更改，请将 `Category` 属性添加到 `ToDoItem.cs` 文件中的 `ToDoItem` 类。

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

然后更新 `Load()` 方法，在同一文件中，将类别设置为默认字符串：

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>修改服务

`WebFrontEnd` 项目是一个 ASP.NET Core 应用程序，其中包含一个显示待办事项列表项的网页。 在 `WebFrontEnd` 项目中，打开 `Index.cshtml` 并添加以下两行（如下所示），显示任务的类别：

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

生成并运行应用，验证是否在网页中看到列出任务的新类别列。

## <a name="upgrade-the-app-from-visual-studio"></a>从 Visual Studio 升级应用

无论是进行代码升级还是配置升级（在本例中我们同时执行这两者），要在 Azure 上升级 Service Fabric 网格应用，请在 Visual Studio 中右键单击“todolistapp”，然后选择“发布...”

接下来，查看“发布 Service Fabric 应用程序”对话框。

使用“目标配置文件”下拉列表选择要用于此部署的 profile.yaml 文件。 若要在云中升级应用，应在下拉列表中选择“cloud.yaml”，其中 `WebFrontEnd_cpu` 将使用该文件中定义的 1.0。

![Visual Studio - Service Fabric 网格 - 发布对话框](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

选择自己的 Azure 帐户和订阅。 将“位置”设置为最初将待办事项应用发布到 Azure 时使用的位置。 本文使用“美国东部”。

将“资源组”设置为最初将待办事项应用发布到 Azure 时使用的资源组。

将“Azure 容器注册表”设置为最初将待办事项应用发布到 Azure 时创建的 Azure 容器注册表名称。

在“发布”对话框中，按“发布”按钮升级 Azure 上的待办事项应用。

通过在 Visual Studio 的“输出”窗口中选择“Service Fabric 工具”窗格，来监视升级的进度。 

生成映像并将其推送到 Azure 容器注册表后，将在输出中显示“状态”链接，单击该链接可在 Azure 门户中监视部署。

完成升级后，“Service Fabric 工具”输出将以 URL 的形式显示应用程序的 IP 地址和端口。

```json
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

打开 Web 浏览器并导航到该 URL，以查看 Azure 中运行的网站。 现在应看到一个网页，其中包含一个类别列。

## <a name="next-steps"></a>后续步骤

本教程的此部分介绍了：
> [!div class="checklist"]
> * 如何使用 Visual Studio 升级 Service Fabric 网格应用

转到下一教程：
> [!div class="nextstepaction"]
> [清理 Service Fabric 网格资源](service-fabric-mesh-tutorial-cleanup-resources.md)