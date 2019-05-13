---
title: 监视 Azure 应用服务性能 | Microsoft Docs
description: Azure 应用服务的应用程序性能监视。 图表加载和响应时间、 依赖关系信息，并对性能设置警报。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: mbullwin
ms.openlocfilehash: ec5b3572cbf74bad9b82eb93a45d7a4664023b95
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408225"
---
# <a name="monitor-azure-app-service-performance"></a>监视 Azure 应用服务性能

启用监视功能在.NET 和.NET Core 上基于上运行的 web 应用程序[Azure 应用服务](https://docs.microsoft.com/azure/app-service/)可以比以往更轻松。 以前需要手动安装站点扩展，而最新扩展代理现已内置到应用服务映像默认情况下。 本文将引导你完成启用 Application Insights 监视，以及提供用于自动执行大规模部署过程的初步指导。

> [!NOTE]
> 手动添加 Application Insights 站点扩展通过**开发工具** > **扩展**已弃用。 此方法扩展安装了依赖于手动更新每个新版本。 该扩展的最新稳定版本现已[预装](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions)作为应用服务映像的一部分。 文件位于`d:\Program Files (x86)\SiteExtensions\ApplicationInsightsAgent`，并使用每个稳定版本自动更新。 如果您遵循的基于代理的说明进行操作，以启用监视下面，它会自动将为您删除不推荐使用的扩展。

## <a name="enable-application-insights"></a>启用 Application Insights

有两种方法来启用应用程序监视的托管的 Azure 应用服务应用程序：

* **基于代理的应用程序监视**(ApplicationInsightsAgent)。  
    * 此方法是最容易启用，并不需要任何高级的配置。 它通常称为"运行时"监视。 适用于 Azure 应用服务，我们建议启用此级别的监视，至少，然后根据您可以评估是否需要通过手动检测的更高级监视的特定方案。

* **手动检测应用程序通过代码**通过安装 Application Insights SDK。

    * 这种方法是更可自定义，但它需要[上的 Application Insights SDK NuGet 包添加的依赖项](https://docs.microsoft.com/azure/azure-monitor/app/asp-net)。 此方法还意味着您必须自行管理包的最新版本的更新。

    * 如果需要进行自定义 API 调用的基于代理的监视与默认情况下不捕获跟踪事件/依赖项，您需要使用此方法。 请查看[API 的自定义事件和指标文章](https://docs.microsoft.com/azure/azure-monitor/app/api-custom-events-metrics)若要了解详细信息。

> [!NOTE]
> 如果基于代理的监视和手动 SDK 基于检测是检测到仅手动检测设置将起作用。 这是为了防止重复的数据从发送。 若要详细了解此签出[故障排除部分](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#troubleshooting)下面。

## <a name="enable-agent-based-monitoring-net"></a>启用基于代理的监视.NET

> [!NOTE]
> 不支持 APPINSIGHTS_JAVASCRIPT_ENABLED 和 urlCompression 的组合。 有关详细信息，请参阅中的说明[故障排除部分](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#troubleshooting)。


1. 在应用服务的 Azure 控制面板中，选择“Application Insights”。

    ![在“设置”下选择“Application Insights”](./media/azure-web-apps/settings-app-insights-01.png)

   * 除非已为此应用设置了 Application Insights 资源，否则请选择创建新资源。 

     > [!NOTE]
     > 当单击“确定”来创建新资源时，将提示你**应用监视设置**。 选择“继续”会将新的 Application Insights 资源链接到你的应用服务，这样做还会**触发应用服务的重新启动**。 

     ![检测 Web 应用](./media/azure-web-apps/create-resource-01.png)

2. 指定要使用哪些资源后，可以选择 Application Insights 根据平台为应用程序收集数据的方式。 ASP.NET 应用程序监视是上： 默认情况下具有两个不同级别的集合。

    ![根据平台选择选项](./media/azure-web-apps/choose-options-new.png)

   * .NET 基本集合级别提供基本的单实例 APM 功能。

   * .NET 建议集合级别：
       * 添加 CPU、内存和 I/O 使用情况趋势。
       * 跨请求/依赖关系边界关联微服务。
       * 收集使用情况趋势，并启用从可用性结果到事务的关联。
       * 收集未经主机进程处理的异常。
       * 提高使用采样时，负载下的 APM 指标准确性。

3. 若要配置的设置，例如采样，可以通过 applicationinsights.config 文件以前控制可以现在与通过应用程序设置相应的前缀的那些相同的设置。 

    * 例如，若要更改初始采样百分比，可以创建的应用程序设置：`MicrosoftAppInsights_AdaptiveSamplingTelemetryProcessor_InitialSamplingPercentage`并将值`100`。

    * 有关受支持的自适应采样遥测处理器设置的列表，您可以参阅[代码](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/master/src/ServerTelemetryChannel/AdaptiveSamplingTelemetryProcessor.cs)并[相关联的文档](https://docs.microsoft.com/azure/azure-monitor/app/sampling)。

## <a name="enable-agent-based-monitoring-net-core"></a>启用基于代理的监视的.NET Core

支持以下版本的.NET Core:ASP.NET Core 2.0, ASP.NET Core 2.1, ASP.NET Core 2.2

目标的完整框架，.NET Core、 独立的部署和 ASP.NET Core 3.0 目前**不支持**使用基于代理/扩展监视。 ([手动检测](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core)通过代码将在所有前面的方案中工作。)

1. 在应用服务的 Azure 控制面板中，选择“Application Insights”。

    ![在“设置”下选择“Application Insights”](./media/azure-web-apps/settings-app-insights-01.png)

   * 除非已为此应用设置了 Application Insights 资源，否则请选择创建新资源。 

     > [!NOTE]
     > 当单击“确定”来创建新资源时，将提示你**应用监视设置**。 选择“继续”会将新的 Application Insights 资源链接到你的应用服务，这样做还会**触发应用服务的重新启动**。 

     ![检测 Web 应用](./media/azure-web-apps/create-resource-01.png)

2. 指定后要使用哪些资源，可以选择 Application Insights 收集每个平台的应用程序数据的方式。 提供了.NET core**建议集合**或**禁用**适用于.NET Core 2.0、 2.1 和 2.2。

    ![根据平台选择选项](./media/azure-web-apps/choose-options-new-net-core.png)

## <a name="enable-client-side-monitoring-net"></a>启用客户端监视.NET

客户端监视是参加 asp.net。 若要启用客户端监视：

* 选择**设置**> * * * * 应用程序设置 * * *
   * 在应用程序设置下添加一个新**应用设置名称**并**值**:

     名称：`APPINSIGHTS_JAVASCRIPT_ENABLED`

     值： `true`

   * **保存**设置并**重新启动**应用。

![应用程序的屏幕截图设置 UI](./media/azure-web-apps/appinsights-javascript-enabled.png)

若要禁用客户端监视从应用程序设置的关联的键值对的删除或将值设置为 false。

## <a name="enable-client-side-monitoring-net-core"></a>启用客户端监视的.NET Core

客户端监视是**默认情况下启用**有关使用.NET Core 应用**建议集合**，无论是否存在的应用设置 APPINSIGHTS_JAVASCRIPT_ENABLED。

如果出于某种原因，你想要禁用客户端监视：

* 选择**设置** > **应用程序设置**
   * 在应用程序设置下添加一个新**应用设置名称**并**值**:

     名称： `APPINSIGHTS_JAVASCRIPT_ENABLED`

     值： `false`

   * **保存**设置并**重新启动**应用。

![应用程序的屏幕截图设置 UI](./media/azure-web-apps/appinsights-javascript-disabled.png)

## <a name="automate-monitoring"></a>自动监视

若要启用使用 Application Insights 的遥测数据收集，需要设置仅应用程序设置：

   ![应用服务应用程序设置的 Application Insights 的可用设置](./media/azure-web-apps/application-settings.png)

### <a name="application-settings-definitions"></a>应用程序设置定义

|应用设置名称 |  定义 | 值 |
|-----------------|:------------|-------------:|
|ApplicationInsightsAgent_EXTENSION_VERSION | 主扩展，它控制运行时监视。 | `~2` |
|XDT_MicrosoftApplicationInsights_Mode |  在默认模式仅、 基本功能的支持是为了确保获得最佳性能。 | `default` 或`recommended`。 |
|InstrumentationEngine_EXTENSION_VERSION | 控制是否二进制重写引擎`InstrumentationEngine`将打开。 此设置会影响性能，并且会影响冷启动/启动时间。 | `~1` |
|XDT_MicrosoftApplicationInsights_BaseExtensions | 如果 SQL 和 Azure 表的文本的控件将与依赖项调用一起捕获。 性能警告： 此设置要求`InstrumentationEngine`。 | `~1` |

### <a name="app-service-application-settings-with-azure-resource-manager"></a>应用服务应用程序设置使用 Azure 资源管理器

可以管理和使用配置的应用服务应用程序设置[Azure 资源管理器模板](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)。 部署新应用服务资源使用 Azure 资源管理器自动化或修改现有资源的设置时，可以使用此方法。

下面是应用服务的应用程序设置 JSON 的基本结构：

```JSON
      "resources": [
        {
          "name": "appsettings",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', variables('webSiteName'))]"
          ],
          "tags": {
            "displayName": "Application Insights Settings"
          },
          "properties": {
            "key1": "value1",
            "key2": "value2"
          }
        }
      ]

```

有关示例应用程序设置为 Application Insights 中，配置与 Azure 资源管理器模板的这[模板](https://github.com/Andrew-MSFT/BasicImageGallery)会非常有用，特别是在开始节[行 238](https://github.com/Andrew-MSFT/BasicImageGallery/blob/c55ada54519e13ce2559823c16ca4f97ddc5c7a4/CoreImageGallery/Deploy/CoreImageGalleryARM/azuredeploy.json#L238)。

### <a name="automate-the-creation-of-an-application-insights-resource-and-link-to-your-newly-created-app-service"></a>自动创建 Application Insights 资源并链接到新创建的应用服务。

若要配置的所有默认 Application Insights 设置都创建的 Azure 资源管理器模板，开始此过程，因为如果您将使用已启用 Application Insights 中都创建新的 Web 应用。

选择**自动化选项**

   ![应用服务 web 应用创建菜单](./media/azure-web-apps/create-web-app.png)

此选项可生成最新的 Azure 资源管理器模板配置的所有必需设置。

  ![应用服务 web 应用模板](./media/azure-web-apps/arm-template.png)

下面是一个示例的所有实例替换都为`AppMonitoredSite`与你的站点名称：

```json
{
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "Microsoft.Web/sites",
            "properties": {
                "siteConfig": {
                    "appSettings": [
                        {
                            "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
                            "value": "[reference('microsoft.insights/components/AppMonitoredSite', '2015-05-01').InstrumentationKey]"
                        },
                        {
                            "name": "ApplicationInsightsAgent_EXTENSION_VERSION",
                            "value": "~2"
                        }
                    ]
                },
                "name": "[parameters('name')]",
                "serverFarmId": "[concat('/subscriptions/', parameters('subscriptionId'),'/resourcegroups/', parameters('serverFarmResourceGroup'), '/providers/Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]",
                "hostingEnvironment": "[parameters('hostingEnvironment')]"
            },
            "dependsOn": [
                "[concat('Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]",
                "microsoft.insights/components/AppMonitoredSite"
            ],
            "apiVersion": "2016-03-01",
            "location": "[parameters('location')]"
        },
        {
            "apiVersion": "2016-09-01",
            "name": "[parameters('hostingPlanName')]",
            "type": "Microsoft.Web/serverfarms",
            "location": "[parameters('location')]",
            "properties": {
                "name": "[parameters('hostingPlanName')]",
                "workerSizeId": "[parameters('workerSize')]",
                "numberOfWorkers": "1",
                "hostingEnvironment": "[parameters('hostingEnvironment')]"
            },
            "sku": {
                "Tier": "[parameters('sku')]",
                "Name": "[parameters('skuCode')]"
            }
        },
        {
            "apiVersion": "2015-05-01",
            "name": "AppMonitoredSite",
            "type": "microsoft.insights/components",
            "location": "West US 2",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Request_Source": "IbizaWebAppExtensionCreate"
            }
        }
    ],
    "parameters": {
        "name": {
            "type": "string"
        },
        "hostingPlanName": {
            "type": "string"
        },
        "hostingEnvironment": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "sku": {
            "type": "string"
        },
        "skuCode": {
            "type": "string"
        },
        "workerSize": {
            "type": "string"
        },
        "serverFarmResourceGroup": {
            "type": "string"
        },
        "subscriptionId": {
            "type": "string"
        }
    },
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0"
}
```

> [!NOTE]
> 在"默认"模式下，该模板将生成应用程序设置。 此模式下是性能优化，尽管可以修改模板以激活任何您喜欢的功能。

### <a name="enabling-through-powershell"></a>通过 PowerShell 启用

若要启用通过 PowerShell 监视的应用程序，只是基础的应用程序设置需要进行更改。 下面是一个示例，从而使应用程序监视的网站中的资源组"AppMonitoredRG"调用"AppMonitoredSite"，并配置数据发送到"012345678-abcd-ef01-2345年-6789abcd"检测密钥。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```powershell
$app = Get-AzWebApp -ResourceGroupName "AppMonitoredRG" -Name "AppMonitoredSite" -ErrorAction Stop
$newAppSettings = @{} # case-insensitive hash map
$app.SiteConfig.AppSettings | %{$newAppSettings[$_.Name] = $_.Value} #preserve non Application Insights Application settings.
$newAppSettings["APPINSIGHTS_INSTRUMENTATIONKEY"] = "012345678-abcd-ef01-2345-6789abcd"; # enable the ApplicationInsightsAgent
$newAppSettings["ApplicationInsightsAgent_EXTENSION_VERSION"] = "~2"; # enable the ApplicationInsightsAgent
$app = Set-AzWebApp -AppSettings $newAppSettings -ResourceGroupName $app.ResourceGroup -Name $app.Name -ErrorAction Stop
```

## <a name="upgrade-monitoring-extensionagent"></a>升级每个代理监视扩展

### <a name="upgrading-from-versions-289-and-up"></a>升级从版本 2.8.9 和更高

从版本 2.8.9 升级会自动发生，无需任何其他操作。 监视的新位传递到目标应用服务，在后台，并在应用程序重新启动它们拾取。

若要检查哪些版本的扩展正在运行，请访问 `http://yoursitename.scm.azurewebsites.net/ApplicationInsights`

![Url 路径的屏幕截图 http://yoursitename.scm.azurewebsites.net/ApplicationInsights](./media/azure-web-apps/extension-version.png)

### <a name="upgrade-from-versions-100---265"></a>从版本 1.0.0-2.6.5 升级

从版本 2.8.9 开始使用预安装的站点扩展。 如果你是早期版本，您可以通过两种方式之一进行更新：

* [通过启用通过门户升级](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#enable-application-insights)。 (即使你有 Azure 应用服务安装 Application Insights 扩展，UI 会显示仅**启用**按钮。 在后台，旧的专用站点扩展将被删除。）

* [通过 PowerShell 升级](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#enabling-through-powershell):

    1. 设置应用程序设置来启用预安装的站点扩展 ApplicationInsightsAgent。 请参阅[通过 powershell 启用](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#enabling-through-powershell)。
    2. 手动删除名为 Azure 应用服务的 Application Insights 扩展的专用站点扩展。

如果升级已完成从 2.5.1 版之前的版本，请检查 ApplicationInsigths dll 已从应用程序 bin 文件夹[请参阅故障排除步骤](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#troubleshooting)。

## <a name="troubleshooting"></a>故障排除

下面是我们用于扩展/代理基于监视适用于.NET 的分步故障排除指南和基于.NET Core 的 Azure 应用服务上运行的应用程序。

> [!NOTE]
> Java 和 Node.js 应用程序通过手动 SDK 基于检测仅支持在 Azure 应用服务上，因此下面的步骤不适用于这些方案。

1. 通过监视应用程序的检查`ApplicationInsightsAgent`。
    * 检查`ApplicationInsightsAgent_EXTENSION_VERSION`应用设置设置为"~ 2"的值。
2. 请确保该应用程序满足要监视的要求。
    * 浏览到 `https://yoursitename.scm.azurewebsites.net/ApplicationInsights`

    ![屏幕截图 https://yoursitename.scm.azurewebsites/applicationinsights结果页](./media/azure-web-apps/app-insights-sdk-status.png)

    * 确认`Application Insights Extension Status`是 `Pre-Installed Site Extension, version 2.8.12.1527, is running.`
        * 如果未运行，请按照[启用 Application Insights 监视的说明](https://docs.microsoft.com/azure/azure-monitor/app/azure-web-apps#enable-application-insights)

    * 确认状态源存在并如下所示： `Status source D:\home\LogFiles\ApplicationInsights\status\status_RD0003FF0317B6_4248_1.json`
        * 如果一个相近的值不存在，则意味着该应用程序当前未运行或不受支持。 若要确保应用程序正在运行，请尝试手动访问应用程序 url/应用程序终结点，这将允许运行时信息变得可用。

    * 确认`IKeyExists`是 `true`
        * 如果为 false，将添加 APPINSIGHTS_INSTRUMENTATIONKEY 与你 ikey guid 传递给应用程序设置。

    * 确认没有任何条目`AppAlreadyInstrumented`， `AppContainsDiagnosticSourceAssembly`，和`AppContainsAspNetTelemetryCorrelationAssembly`。
        * 如果存在任何这些条目，从你的应用程序中删除以下包： `Microsoft.ApplicationInsights`， `System.Diagnostics.DiagnosticSource`，和`Microsoft.AspNet.TelemetryCorrelation`。

下表提供了这些值的含义的更详细的说明，其基础导致，以及建议的修补程序：

|问题值|说明|解决
|---- |----|---|
| `AppAlreadyInstrumented:true` | 此值指示，该扩展检测到 SDK 的某些方面已经存在于应用程序，并将回退。 它可能是由于对的引用`System.Diagnostics.DiagnosticSource`， `Microsoft.AspNet.TelemetryCorrelation`，或 `Microsoft.ApplicationInsights`  | 删除的引用。 从特定 Visual Studio 模板，添加默认情况下某些这些引用和较旧版本的 Visual Studio 可能会将引用添加到`Microsoft.ApplicationInsights`。
|`AppAlreadyInstrumented:true` | 如果应用程序面向.NET Core 2.1 或 2.2，并且是指[Microsoft.AspNetCore.All](https://www.nuget.org/packages/Microsoft.AspNetCore.All)元包，然后它将在 Application Insights 中和扩展将回退。 | 客户在.NET Core 2.1,2.2[建议](https://github.com/aspnet/Announcements/issues/287)以改为使用 Microsoft.AspNetCore.App 元包。|
|`AppAlreadyInstrumented:true` | 此值还可能引起从以前的部署的应用程序文件夹中的上述 dll 存在。 | 清除要确保删除这些 dll 的应用程序文件夹。|
|`AppContainsAspNetTelemetryCorrelationAssembly: true` | 此值指示扩展检测到对引用`Microsoft.AspNet.TelemetryCorrelation`在应用程序，并将回退。 | 删除的引用。
|`AppContainsDiagnosticSourceAssembly**:true`|此值指示扩展检测到对引用`System.Diagnostics.DiagnosticSource`在应用程序，并将回退。| 删除的引用。
|`IKeyExists:false`|此值指示的检测密钥不存在 AppSetting 中`APPINSIGHTS_INSTRUMENTATIONKEY`。 可能的原因：值可能已被意外删除，忘记了在自动化脚本等设置的值。 | 请确保该设置位于应用服务应用程序设置。

### <a name="appinsightsjavascriptenabled-and-urlcompression-is-not-supported"></a>不支持 APPINSIGHTS_JAVASCRIPT_ENABLED 和 urlCompression

如果使用 APPINSIGHTS_JAVASCRIPT_ENABLED = true 内容进行编码的情况下，可能会收到错误，如： 

- 500 URL 重写错误
- 500.53 在 HTTP 响应的内容是编码 (gzip) 时，不能应用 URL 重写模块错误，消息的出站重写规则。 

这是由于 APPINSIGHTS_JAVASCRIPT_ENABLED 应用程序设置设置为 true 和内容编码正在同时存在。 尚不支持此方案。 解决方法是从应用程序设置中删除 APPINSIGHTS_JAVASCRIPT_ENABLED。 遗憾的是，这意味着，如果仍需要客户端/浏览器端 JavaScript 检测，手动 SDK 引用所需你的网页。 请按照[说明](https://github.com/Microsoft/ApplicationInsights-JS#snippet-setup-ignore-if-using-npm-setup)进行手动检测使用 JavaScript SDK。

有关 Application Insights 代理/扩展的最新信息，请参阅[发行说明](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/app-insights-web-app-extensions-releasenotes.md)。

## <a name="next-steps"></a>后续步骤
* [在实时应用上运行探查器](../app/profiler.md)。
* [Azure Functions](https://github.com/christopheranderson/azure-functions-app-insights-sample) - 使用 Application Insights 监视 Azure Functions
* [将 Azure 诊断配置为](../platform/diagnostics-extension-to-application-insights.md)向 Application Insights 发送数据。
* [监视服务运行状况指标](../platform/data-platform.md)以确保服务可用且做出快速响应。
* 每当操作事件发生或指标超过阈值时[接收警报通知](../platform/alerts-overview.md)。
* 若要从访问网页的浏览器获取客户端遥测数据，请使用[适用于 JavaScript 应用和网页的 Application Insights](javascript.md)。
* [设置可用性 Web 测试](monitor-web-app-availability.md)，以便在站点关闭时发出警报。
