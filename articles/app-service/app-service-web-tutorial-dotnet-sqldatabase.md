---
title: 使用 SQL 数据库构建 ASP.NET 应用 - Azure 应用服务 | Microsoft Docs
description: 了解如何将采用 SQL Server 数据库的 C# ASP.NET 应用部署到 Azure。
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 03c584f1-a93c-4e3d-ac1b-c82b50c75d3e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: csharp
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: f010e7564f097f28269070d85e2895e73f454054
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415580"
---
# <a name="tutorial-build-an-aspnet-app-in-azure-with-sql-database"></a>教程：使用 SQL 数据库在 Azure 中构建 ASP.NET 应用

[Azure 应用服务](overview.md)提供高度可缩放、自修补的 Web 托管服务。 本教程演示如何在应用服务中部署数据驱动的 ASP.NET 应用，以及如何将其连接到 [Azure SQL 数据库](../sql-database/sql-database-technical-overview.md)。 完成后，你便拥有了一个在 Azure 中运行并连接到 SQL 数据库的 ASP.NET 应用。

![已在 Azure 应用服务中发布 ASP.NET 应用程序](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 在 Azure 中创建 SQL 数据库。
> * 将 ASP.NET 应用连接到 SQL 数据库
> * 将应用部署到 Azure
> * 更新数据模型并重新部署应用
> * 将日志从 Azure 流式传输到终端
> * 在 Azure 门户中管理应用

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

完成本教程：

安装带有 ASP.NET 和 Web 开发工作负荷的 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a>。

如果已安装 Visual Studio，请单击“工具” > **Get Tools and Features**“获取工具和功能”，以便在 Visual Studio 中添加工作负荷。

## <a name="download-the-sample"></a>下载示例

- [下载示例项目 ](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/archive/master.zip)。
- 提取（解压缩）dotnet-sqldb-tutorial-master.zip 文件。

此示例项目包含一个使用 [Entity Framework Code First](/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application) 的基本 [ASP.NET MVC](https://www.asp.net/mvc) 创建-读取-更新-删除 (CRUD) 应用。

### <a name="run-the-app"></a>运行应用

在 Visual Studio 中打开 dotnet-sqldb-tutorial-master/DotNetAppSqlDb.sln 文件。 

键入 `Ctrl+F5` 在不调试的情况下运行应用。 该应用将显示在默认浏览器中。 选择“新建”链接，创建一对待办事项。 

![“新建 ASP.NET 项目”对话框](media/app-service-web-tutorial-dotnet-sqldatabase/local-app-in-browser.png)

测试“编辑”、“详细信息”和“删除”链接。

该应用使用数据库上下文连接数据库。 在此示例中，数据库上下文使用名为 `MyDbConnection` 的连接字符串。 此连接字符串在 Web.config 文件中设置，并在 Models\MyDatabaseContext.cs 文件中引用。 在教程后面部分，该连接字符串名称用于将 Azure 应用连接到 Azure SQL 数据库。 

## <a name="publish-to-azure-with-sql-database"></a>使用 SQL 数据库发布到 Azure

在“解决方案资源管理器”中，右键单击 “DotNetAppSqlDb”项目，并选择“发布”。

![从解决方案资源管理器发布](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

确保已选择“Microsoft Azure 应用服务”，并单击“发布”。

![从项目概述页发布](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-to-app-service.png)

发布将打开“创建应用服务”对话框，这有助于创建在 Azure 中运行 ASP.NET 应用所需的所有 Azure 资源。

### <a name="sign-in-to-azure"></a>登录 Azure

在“创建应用服务”对话框中单击“添加帐户”，并登录到 Azure 订阅。 如果已登录到 Microsoft 帐户，请确保该帐户包含 Azure 订阅。 如果已登录的 Microsoft 帐户不包含 Azure 订阅，请单击该帐户以添加正确的帐户。 

> [!NOTE]
> 如果已经登录，请先不要选择“创建”。
>
>
   
![登录 Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/sign-in-azure.png)

### <a name="configure-the-web-app-name"></a>配置 Web 应用名称

可保留生成的 Web 应用名称，或将其更改为另一个唯一名称（有效字符是 `a-z` `0-9` 和 `-`）。 Web 应用名称将用作应用默认 URL 的一部分（`<app_name>.azurewebsites.net`，其中 `<app_name>` 是 Web 应用的名称）。 Web 应用名称在 Azure 中的所有应用程序中必须是唯一的。 

![“创建应用服务”对话框](media/app-service-web-tutorial-dotnet-sqldatabase/wan.png)

### <a name="create-a-resource-group"></a>创建资源组

[!INCLUDE [resource-group](../../includes/resource-group.md)]

在“资源组”旁边单击“新建”。

![在“资源组”旁边单击“新建”。](media/app-service-web-tutorial-dotnet-sqldatabase/new_rg2.png)

将资源组命名为 myResourceGroup。

### <a name="create-an-app-service-plan"></a>创建应用服务计划

[!INCLUDE [app-service-plan](../../includes/app-service-plan.md)]

在“应用服务计划”旁边单击“新建”。 

在“配置应用服务计划”对话框中，使用以下设置配置新的应用服务计划：

![创建应用服务计划](./media/app-service-web-tutorial-dotnet-sqldatabase/configure-app-service-plan.png)

| 设置  | 建议的值 | 更多信息 |
| ----------------- | ------------ | ----|
|应用服务计划| myAppServicePlan | [应用服务计划](../app-service/overview-hosting-plans.md) |
|**位置**| 西欧 | [Azure 区域](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) |
|**大小**| 免费 | [定价层](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)|

### <a name="create-a-sql-server-instance"></a>创建 SQL Server 实例

在创建数据库时之前，需要 [Azure SQL 数据库逻辑服务器](../sql-database/sql-database-features.md)。 逻辑服务器包含一组作为组管理的数据库。

单击“创建 SQL 数据库”。

![创建 SQL 数据库](media/app-service-web-tutorial-dotnet-sqldatabase/web-app-name.png)

在“配置 SQL 数据库”对话框中，单击“SQL Server”旁的“新建”。 

将生成唯一的服务器名称。 此名称用作逻辑服务器`<server_name>.database.windows.net`默认 URL 的一部分。  在 Azure 中的所有逻辑服务器实例中，此名称必须是唯一的。 可以更改服务器名称，但本教程沿用生成的值。

添加管理员用户名和密码。 有关密码复杂性要求，请参阅[密码策略](/sql/relational-databases/security/password-policy)。

牢记此用户名和密码。 随后，你需要用它们来管理逻辑服务器实例。

> [!IMPORTANT]
> 虽然连接字符串中的密码已在 Visual Studio 和应用服务中受到屏蔽，但由于它实际上是保留在某个位置，因此增加了应用的受攻击面。 应用服务可以使用[托管服务标识](overview-managed-identity.md)，因此根本不需要将机密保留在代码或应用配置中，这样就消除了上述风险。 有关详细信息，请参阅[后续步骤](#next-steps)。

![创建 SQL Server 实例](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database-server.png)

单击“确定”。 尚不要关闭“配置 SQL 数据库”对话框。

### <a name="create-a-sql-database"></a>创建 SQL 数据库

在“配置 SQL 数据库”对话框中： 

* 保留默认生成的**数据库名称**。
* 在“连接字符串名称”中，键入 *MyDbConnection*。 此名称必须与 Models\MyDatabaseContext.cs 文件中引用的连接字符串相匹配。
* 选择“确定”。

![配置 SQL 数据库](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database.png)

“创建应用服务”对话框显示已配置的资源。 单击“创建”。 

![已创建资源](media/app-service-web-tutorial-dotnet-sqldatabase/app_svc_plan_done.png)

向导程序完成创建 Azure 资源后，会将 ASP.NET 应用发布到 Azure。  默认浏览器启动时会打开所部署的应用的 URL。  

添加几个待办事项。

![已在 Azure 应用中发布 ASP.NET 应用程序](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

祝贺你！ 你的数据驱动的 ASP.NET 应用程序已经在 Azure 应用服务中运行了。

## <a name="access-the-sql-database-locally"></a>本地访问 SQL 数据库

使用 Visual Studio 可在“SQL Server 对象资源管理器”中轻松浏览和管理自己的新 SQL 数据库。

### <a name="create-a-database-connection"></a>创建数据库连接

从“视图”菜单上，选择“SQL Server 对象资源管理器”。

在“SQL Server 对象资源管理器”顶部，单击“添加 SQL Server”按钮。

### <a name="configure-the-database-connection"></a>配置数据库连接

在“连接”对话框中，展开“Azure”节点。 此处列出了 Azure 中全部的 SQL 数据库实例。

选择先前创建的 SQL 数据库。 前面使用过的连接将自动填充在底部。

键入先前创建的数据库管理员密码，然后单击“连接”。

![通过 Visual Studio 配置数据库连接](./media/app-service-web-tutorial-dotnet-sqldatabase/connect-to-sql-database.png)

### <a name="allow-client-connection-from-your-computer"></a>允许来自计算机的客户端连接

系统将打开“新建防火墙规则”对话框。 默认情况下，SQL 数据库实例仅允许来自 Azure 服务的连接，如 Azure 应用。 若要连接到你的数据库，请在 SQL 数据库实例中创建防火墙规则。 防火墙规则仅允许本地计算机的公共 IP 地址。

对话框中已填充了计算机的公共 IP 地址。

请确保选中“添加我的客户端 IP”，并单击“确定”。 

![为 SQL 数据库实例设置防火墙](./media/app-service-web-tutorial-dotnet-sqldatabase/sql-set-firewall.png)

Visual Studio 成功为 SQL 数据库实例创建防火墙设置后，连接将立即显示在“SQL Server 对象资源管理器”中。

可在此处执行最常见的数据库操作，如运行查询、创建视图和存储过程等。 

展开连接 >“数据库” > **&lt;你的数据库 >** > “表”。 右键单击`Todoes`表，然后选择“查看数据”。 

![探索 SQL 数据库对象](./media/app-service-web-tutorial-dotnet-sqldatabase/explore-sql-database.png)

## <a name="update-app-with-code-first-migrations"></a>使用 Code First 迁移更新应用

可以在 Visual Studio 中使用熟悉的工具，更新 Azure 中的数据库和应用。 此步骤使用 Entity Framework 中的 Code First 迁移对数据库架构进行更改，并将其发布至 Azure。

有关使用 Entity Framework Code First 迁移的详细信息，请参阅[用 MVC 5 开始使用 Entity Framework 6 Code First](https://docs.microsoft.com/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application)。

### <a name="update-your-data-model"></a>更新数据模型

在代码编辑器中打开 _Models\Todo.cs_。 将以下属性添加到 `ToDo` 类：

```csharp
public bool Done { get; set; }
```

### <a name="run-code-first-migrations-locally"></a>本地运行 Code First 迁移

运行几个命令更新本地数据库。 

在“工具”菜单中，单击“NuGet 包管理器” > “包管理器控制台”。

在“程序包管理控制台”窗口中，启用 Code First 迁移：

```powershell
Enable-Migrations
```

添加迁移：

```powershell
Add-Migration AddProperty
```

更新本地数据库：

```powershell
Update-Database
```

键入 `Ctrl+F5` 运行应用。 测试编辑、详细信息并创建链接。

如果应用程序加载未出错，则 Code First 迁移成功。 但页面看上去仍没有变化，这是因为应用程序逻辑尚未使用新属性。 

### <a name="use-the-new-property"></a>使用新属性

为使用 `Done` 属性，需要对代码做一些更改。 简单起见，本教程中将仅更改 `Index` 和 `Create` 视图，以便在操作过程中查看属性。

打开 _Controllers\TodosController.cs_。

在第 52 行找到 `Create()` 方法，并将 `Done` 添加到 `Bind` 特性中的属性列表。 完成后，`Create()` 方法签名应如下面的代码所示：

```csharp
public ActionResult Create([Bind(Include = "Description,CreatedDate,Done")] Todo todo)
```

打开 _Views\Todos\Create.cshtml_。

在 Razor 代码中，你会看到使用`model.Description` 的 `<div class="form-group">` 元素，以及使用 `model.CreatedDate` 的 `<div class="form-group">` 元素。 紧跟在这两个元素之后的是使用 `model.Done` 的 `<div class="form-group">` 元素：

```csharp
<div class="form-group">
    @Html.LabelFor(model => model.Done, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-10">
        <div class="checkbox">
            @Html.EditorFor(model => model.Done)
            @Html.ValidationMessageFor(model => model.Done, "", new { @class = "text-danger" })
        </div>
    </div>
</div>
```

打开 _Views\Todos\Index.cshtml_。

搜索空的 `<th></th>` 元素。 在此元素的正上方，添加下列 Razor 代码：

```csharp
<th>
    @Html.DisplayNameFor(model => model.Done)
</th>
```

查找包含 `Html.ActionLink()` 帮助程序方法的 `<td>` 元素。 在此 `<td>` _上面_，使用以下 Razor 代码添加另一个 `<td>` 元素：

```csharp
<td>
    @Html.DisplayFor(modelItem => item.Done)
</td>
```

这就是要在 `Index` 和 `Create` 视图中查看更改所需的全部操作。 

键入 `Ctrl+F5` 运行应用。

现在可以添加一个待办事项，并勾选“完成”。 然后，它会在主页中显示为已完成的项。 请记住，由于未更改`Edit`视图，`Edit`视图不显示`Done`字段。

### <a name="enable-code-first-migrations-in-azure"></a>在 Azure 中启用 Code First 迁移

代码更改生效（包括数据库迁移）后，将其发布至 Azure 应用，并使用 Code First 迁移更新 SQL 数据库。

与先前的操作相同，右键单击项目，并选择“发布”。

单击“配置”以打开发布设置。

![打开发布设置](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-settings.png)

在向导中，单击“下一步”。

请确保在 **MyDatabaseContext (MyDbConnection)** 中填充 SQL 数据库的连接字符串。 可能需要从下拉列表中选择“myToDoAppDb”数据库。 

选择“执行 Code First 迁移(应用程序启动时运行)”，并单击“保存”。

![在 Azure 应用中启用 Code First 迁移](./media/app-service-web-tutorial-dotnet-sqldatabase/enable-migrations.png)

### <a name="publish-your-changes"></a>发布更改

现已在 Azure 应用中启用了 Code First 迁移，可发布代码更改信息。

在发布页中单击“发布”。

再次尝试添加待办事项并选择“完成”，然后，它们将作为已完成项显示在主页中。

![Code First 迁移后的 Azure 应用](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

所有现有待办事项仍将显示。 重新发布 ASP.NET 应用程序时，SQL 数据库中的现有数据不会丢失。 此外，Code First 迁移仅更改数据架构，而使现有数据保持不变。


## <a name="stream-application-logs"></a>流式传输应用程序日志

可直接通过 Azure 应用将跟踪消息流式传输到 Visual Studio。

打开 _Controllers\TodosController.cs_。

每项操作都以 `Trace.WriteLine()` 方法开头。 添加此代码的目的是演示如何将跟踪消息添加至 Azure 应用。

### <a name="open-server-explorer"></a>打开服务器资源管理器

在“视图”菜单中，选择“服务器资源管理器”。 可在“服务器资源管理器”中为 Azure 应用配置日志记录。 

### <a name="enable-log-streaming"></a>启用日志流式传输

在“服务器资源管理器”中，展开“Azure” > “应用服务”。

展开“myResourceGroup”资源组，该资源组在首次创建 Azure 应用时创建。

右键单击 Azure 应用，并选择“查看流式处理日志”。

![启用日志流式传输](./media/app-service-web-tutorial-dotnet-sqldatabase/stream-logs.png)

现在，日志已流式传输到“输出”窗口。 

![输出窗口中的日志流式传输](./media/app-service-web-tutorial-dotnet-sqldatabase/log-streaming-pane.png)

但还无法查看任何跟踪消息。 因为当首先选择“查看流式处理日志”时，Azure 应用将跟踪级别设置为 `Error`，此级别只记录错误事件（使用 `Trace.TraceError()` 方法）。

### <a name="change-trace-levels"></a>更改跟踪级别

若要更改跟踪级别以输出其他跟踪消息，请返回“服务器资源管理器”。

再次右键单击 Azure 应用并选择“查看设置”。

在“应用程序日志记录(文件系统)”下拉列表中，选择“详细”。 单击“ **保存**”。

![将跟踪级别更改为详细](./media/app-service-web-tutorial-dotnet-sqldatabase/trace-level-verbose.png)

> [!TIP]
> 可试验不同的跟踪级别，以查看每个级别分别显示哪些类型的消息。 例如，“信息”级别包括 `Trace.TraceInformation()`、`Trace.TraceWarning()` 和 `Trace.TraceError()` 创建的所有日志，但不包括 `Trace.WriteLine()` 创建的日志。
>
>

在浏览器中再次导航到应用（ http://&lt;应用名称>.azurewebsites.net），然后尝试在 Azure 中的待办事项列表应用程序周围单击。 现在，跟踪消息已流式传输到 Visual Studio 中的“输出”窗口。

```console
Application: 2017-04-06T23:30:41  PID[8132] Verbose     GET /Todos/Index
Application: 2017-04-06T23:30:43  PID[8132] Verbose     GET /Todos/Create
Application: 2017-04-06T23:30:53  PID[8132] Verbose     POST /Todos/Create
Application: 2017-04-06T23:30:54  PID[8132] Verbose     GET /Todos/Index
```



### <a name="stop-log-streaming"></a>停止日志流式传输

若要停止日志流式传输服务，请单击“输出”窗口中的“停止监视”按钮。

![停止日志流式传输](./media/app-service-web-tutorial-dotnet-sqldatabase/stop-streaming.png)

## <a name="manage-your-azure-app"></a>管理 Azure 应用

转到 [Azure 门户](https://portal.azure.com)查看创建的应用。 



从左侧菜单中单击“应用服务”，然后单击 Azure 应用的名称。

![在门户中导航到 Azure 应用](./media/app-service-web-tutorial-dotnet-sqldatabase/access-portal.png)

你已登录到应用页。 

默认情况下，门户将显示“概述”页。 在此页中可以查看应用的运行状况。 在此处还可以执行基本的管理任务，例如浏览、停止、启动、重新启动和删除。 该页左侧的选项卡显示可以打开的不同配置页。 

![Azure 门户中的应用服务页](./media/app-service-web-tutorial-dotnet-sqldatabase/web-app-blade.png)

[!INCLUDE [Clean up section](../../includes/clean-up-section-portal-web-app.md)]

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 在 Azure 中创建 SQL 数据库。
> * 将 ASP.NET 应用连接到 SQL 数据库
> * 将应用部署到 Azure
> * 更新数据模型并重新部署应用
> * 将日志从 Azure 流式传输到终端
> * 在 Azure 门户中管理应用

转到下一教程，了解如何轻松地提高 Azure SQL 数据库连接的安全性。

> [!div class="nextstepaction"]
> [使用适用于 Azure 资源的托管标识安全地访问 SQL 数据库](app-service-web-tutorial-connect-msi.md)
