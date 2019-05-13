---
title: 在 Azure 时序见解预览版资源管理器中直观显示数据 | Microsoft Docs
description: 本文介绍 Azure 时序见解预览版资源管理器 Web 应用中可用的功能和选项。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 05/03/2019
ms.custom: seodec18
ms.openlocfilehash: 862465a7611f1a2bc65dbb0c49c4de512bd239de
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442100"
---
# <a name="visualize-data-in-the-explorer-preview"></a>在资源管理器预览版中实现数据可视化

本文档介绍的 UI/用户体验功能和界面 Azure 时间系列 Insights 预览版[演示 web 应用](https://insights.timeseries.azure.com/preview/demo)。 具体而言，它讨论托管的示例、 接口的自定义选项，并通过提供演示导航的布局。

## <a name="prerequisites"></a>必备组件

若要开始使用 Azure 时时序见解预览资源管理器，必须：

* 设置时序见解环境。 若要了解有关预配实例的详细信息请尝试我们[Azure 时间系列 Insights 预览版](./time-series-insights-update-create-environment.md)教程。
* [提供的数据访问](./time-series-insights-data-access.md)于可为帐户创建的时序见解环境。 可以向其他人以及自己提供访问。
* 向数据推送到该环境的时序见解环境添加事件源：
  * 了解[如何连接到事件中心](./time-series-insights-how-to-add-an-event-source-eventhub.md)
  * 了解[如何连接到 IoT 中心](./time-series-insights-how-to-add-an-event-source-iothub.md)

## <a name="learn-about-the-preview-explorer"></a>了解预览版资源管理器

Azure 时序见解预览版资源管理器包含以下元素：

[![资源管理器视图](media/v2-update-explorer/explorer-one.png)](media/v2-update-explorer/explorer-one.png#lightbox)

1. <a href="#environment-dropdown">**环境面板**</a>:显示 Azure TSI 环境。
1. <a href="#navigation-menu">**导航菜单**</a>:允许之间切换**分析**并**模型**页。
1. <a href="#hierarchy-tree">**层次结构树**</a>:可以选择特定模型和要被绘制为图表的数据元素。
1. <a href="#preview-well">**时间序列还**</a>:使用颜色编码的表格格式显示当前所选的数据元素。
1. <a href="#preview-chart">**图表面板**</a>:显示当前工作的图表。
1. <a href="#time-editor-panel">**时间线**</a>:使你可以修改工作时间跨度。
1. <a href="#navigation-panel">**应用栏**</a>:包含用户管理选项（如当前租户），并使你可以更改主题和语言设置。

## <a name="environment-dropdown"></a>环境下拉列表中

环境下拉列表中显示有权访问的所有时序见解环境。 列表包括即用即付环境 （预览版） 和 S1/S2 环境 （正式发布或 GA）。 

1. 只需单击下拉箭头旁边显示环境：

   [![控制面板](media/v2-update-explorer/explorer-two.png)](media/v2-update-explorer/explorer-two.png#lightbox)

1. 然后，选择所需的环境。

## <a name="navigation-menu"></a>导航菜单

  [![在导航菜单](media/v2-update-explorer/explorer-three.png)](media/v2-update-explorer/explorer-three.png#lightbox)

在导航菜单允许您两个视图之间进行选择：

* **分析**：使你可以对已建模或未建模时列数据创建图表和执行丰富分析。
* **模型**：使你可以将新时序见解预览版类型、层次结构和实例推送到时序见解模型。

## <a name="hierarchy-tree"></a>层次结构树

层次结构树显示所选的数据元素，包括模型、 特定设备和设备上的传感器。

### <a name="model-search-panel"></a>模型搜索面板

模型搜索面板使你可以轻松搜索和导航时序模型层次结构，以找到要在图表上显示的特定时序实例。 选择实例时，它们会同时添加到当前图表以及数据中。

  [![模型搜索面板](media/v2-update-explorer/explorer-four.png)](media/v2-update-explorer/explorer-four.png#lightbox)

### <a name="model-authoring"></a>在创作模型

Azure 时间系列 Insights 预览版支持在时序模型上的完整创建、 读取、 更新和删除 (CRUD) 操作。  

* **时序模型类型**：时序见解类型可用于定义变量或公式以便进行计算。 它们与给定时序见解实例关联。 一种类型可以具有一个或多个变量。
* **时序模型层次结构**：层次结构是数据的系统组织。 层次结构描述时序见解数据中不同实体之间的关系。
* **时序模型实例**：实例是时序本身。 在大多数情况下，它们可以**DeviceID**或**AssetID**，这是在环境中的资产的唯一标识符。

若要详细了解时序模型，请参阅[时序模型](./time-series-insights-update-tsm.md)。

## <a name="preview-well"></a>好的预览

也显示实例字段和其他与所选 TSI 实例相关联的元数据。 在右侧的复选框可以隐藏或显示当前的图表中的特定实例。 您可能还会删除特定的数据元素从当前数据也通过单击红色**删除**（垃圾桶） 中的元素的左侧的控件。

  [![好在预览](media/v2-update-explorer/explorer-five.png)](media/v2-update-explorer/explorer-five.png#lightbox)

此外可以重新配置的布局你**分析**图表页上，通过在右上角选择省略号图标：

  [![遥测数据布局选项](media/v2-update-explorer/explorer-six.png)](media/v2-update-explorer/explorer-six.png#lightbox)

> [!NOTE]
> 如果看到以下消息，则实例在所选时间跨度内没有任何数据。 若要解决该问题，可以增加时间跨度或确认实例在推送数据。
>
> ![没有数据通知](media/v2-update-explorer/explorer-seven.png)

## <a name="preview-chart"></a>预览图表

图表后，您可以将 TSI 实例显示为行。 可以通过单击 Web 控件来折叠环境面板、数据模型和时间跨度控件面板，以使图表更大。

  [![预览图表概述](media/v2-update-explorer/explorer-eight.png)](media/v2-update-explorer/explorer-eight.png#lightbox)

1. **所选日期范围**：控制可用于实现可视化效果的数据元素。

1. **内部日期范围滑块工具**：通过将两个终结点控件拖动到所需时间跨度上来使用它们。

1. **时间跨度折叠控件**：折叠和展开时间跨度面板编辑器。

1. **Y 轴格式控件**：循环显示可用 Y 轴视图选项：

    * `Default`：每行都具有单独的 Y 轴。
    * `Stacked`：使你可以在同一 Y 轴上堆叠多个行，Y 轴数据基于所选行而更改。
    * `Shared`：所有 Y 轴数据一起显示。

1. **当前数据元素**：当前所选数据元素及其关联的详细信息。

可以通过单击左键单击当前图上的数据点，然后将所选区域拖动到所选终结点，进一步深化到特定数据切片。 右键单击灰、 所选区域中，然后单击**缩放**此图中所示：

  [![预览图表缩放](media/v2-update-explorer/explorer-nine.png)](media/v2-update-explorer/explorer-nine.png#lightbox)

在执行后**缩放**操作，你将看到你所选的数据集。 单击 Y 轴格式控件以循环访问时序见解数据的三个 Y 轴表示。

  [![预览图表 y 轴](media/v2-update-explorer/explorer-ten.png)](media/v2-update-explorer/explorer-ten.png#lightbox)

在此处可以查看共享 Y 轴的示例：

  [![预览共享的 y 轴](media/v2-update-explorer/explorer-eleven.png)](media/v2-update-explorer/explorer-eleven.png#lightbox)

## <a name="time-editor-panel"></a>时间编辑器面板

使用时序见解预览版时，首先选择时间跨度。 所选时间跨度控制可用于通过时序见解预览版小组件进行操作的数据集。 以下 Web 控件在时序见解预览版中可用于选择工作时间跨度。

  [![时间选择面板](media/v2-update-explorer/explorer-twelve.png)](media/v2-update-explorer/explorer-twelve.png#lightbox)

1. **内部日期范围滑块工具**：通过将两个终结点控件拖动到所需时间跨度上来使用它们。 此内部日期范围内受外部日期范围滑块控件约束。

1. **增加和减少日期范围按钮**：通过选择任一按钮来增加或减少时间跨度，从而获得所需间隔。

1. **时间跨度折叠控件**：此 Web 控件使你可以隐藏所有控件（内部日期范围滑块工具除外）。

1. **外部日期范围滑块控件**：使用终结点控件选择可用于内部日期范围控件的外部日期范围。

1. **快速选择时间日期范围下拉**:可让你快速预定的时间跨度的选定内容，如最后一个之间切换**30 分钟**，则**最近 12 个小时**，或**自定义范围**。 更改此值也会更改间隔大小滑块工具中所讨论的可用间隔范围。

1. **间隔大小滑块工具**：使你可以在相同时间跨度内放大和缩小间隔。 通过此操作可更精确地控制较大时间片段之间的移动。 它显示直至小到毫秒片段的平滑趋势，从而可以查看精细的高分辨率数据切片。 滑块的默认起点设置为所选数据的最佳视图，可平衡分辨率、查询速度和粒度。

1. **日期范围起始和结束 Web 控件**：与此 web 控件，可以轻松地单击并选择所需的日期和时间范围。 还可以使用该控件在不同时区之间切换。 进行更改之后，若要应用于当前工作区，请选择“保存”。

   [![与选择面板](media/v2-update-explorer/explorer-thirteen.png)](media/v2-update-explorer/explorer-thirteen.png#lightbox)

## <a name="navigation-panel"></a>导航面板

TSI 应用的顶部显示的时间系列 Insights 预览版导航面板中。 它提供了以下功能。

### <a name="current-session-share-link-control"></a>当前会话共享链接控件

  [![共享图标](media/v2-update-explorer/explorer-fifteen.png)](media/v2-update-explorer/explorer-fifteen.png#lightbox)

选择新**共享**图标以与你的团队共享的 URL 链接。

  [![共享实例 URL](media/v2-update-explorer/url-share.png)](media/v2-update-explorer/url-share.png#lightbox)

### <a name="tenant-section"></a>租户分区

  [![租户所选内容](media/v2-update-explorer/explorer-sixteen.png)](media/v2-update-explorer/explorer-sixteen.png#lightbox)

* 显示当前时序见解登录帐户信息。
* 使你可以在可用时序见解主题之间切换。
* 可以查看预览[演示 web 应用](https://insights.timeseries.azure.com/preview/demo)。

### <a name="theme-selection"></a>主题选择

若要选择新的主题，请单击配置文件图标位于右上角。 然后，选择**更改主题**。

  [![主题选择](media/v2-update-explorer/theme-selection.png)](media/v2-update-explorer/theme-selection.png#lightbox)

> [!TIP]
> 语言选择也会提供通过单击配置文件图标。

Azure 时序见解预览版支持两个主题：

* **浅色主题**：本文档中显示的默认主题。
* **深色主题**：如下所示呈现资源管理器：

  [![所选的深色主题](media/v2-update-explorer/explorer-seventeen.png)](media/v2-update-explorer/explorer-seventeen.png#lightbox)

## <a name="s1s2-environment-controls"></a>S1/S2 环境控件

### <a name="preview-terms-panel"></a>预览条款面板

此部分仅适用于尝试在更新后的 UI 中使用资源管理器的现有 S1/S2 环境。 你可能要组合使用 GA 产品和预览版。 我们将现有 UI 中的一些功能添加到了更新的资源管理器中，但是你可以在现有时序见解资源管理器中获得 S1/S2 环境的完整 UI 体验。  

你会看到时序见解术语面板（在其中定义环境中的查询），而不是层次结构。 它使你可以基于谓词来筛选数据。

  [![其中查询面板](media/v2-update-explorer/explorer-eighteen.png)](media/v2-update-explorer/explorer-eighteen.png#lightbox)

时序见解预览版术语编辑器面板采用以下参数：

**Where**：Where 子句使你可以使用下表中列出的操作数集来快速筛选事件。 如果通过选择操作数来执行搜索，则谓词会基于该搜索自动更新。 支持的操作数类型包括：

| Operation | 支持的类型   | 说明 |
| --- | --- | --- |
| `<`, `>`, `<=`, `>=` | 双精度、DateTime、TimeSpan | |
| `=`、`!=`、`<>` | 字符串、布尔型、双精度型、DateTime、TimeSpan、NULL |
| `IN` | 字符串、布尔型、双精度型、DateTime、TimeSpan、NULL | 所有操作数应为同一类型或者是 NULL 常数。 |
| `HAS` | String | 右侧只允许出现常数字符串文本。 不允许出现空字符串和 NULL。 |

通过阅读[表达式 (TSX)](https://docs.microsoft.com/rest/api/time-series-insights/preview-tsx) 来了解有关支持的查询操作和数据类型的详细信息。

### <a name="examples-of-where-clauses"></a>Where 子句示例

  [![其中子句示例](media/v2-update-explorer/explorer-nineteen.png)](media/v2-update-explorer/explorer-nineteen.png#lightbox)

**度量值**：所有数值列将显示一个下拉列表 (**双精度型值**) 可用作当前图表元素。

**拆分依据**：此下拉列表显示模型中可以用作数据分组依据的所有可用分类列（字符串）。 最多可以添加五个术语，在同一个 X 轴上查看。 输入所需参数，然后选择“添加”以添加新术语。

  [![其中一个查询和筛选视图](media/v2-update-explorer/explorer-twenty.png)](media/v2-update-explorer/explorer-twenty.png#lightbox)

可以通过选择可见图标，在图表面板中显示或隐藏元素，如下图所示。 您可以完全移除查询，通过单击红色**X**。

  [![两个查询和筛选视图](media/v2-update-explorer/explorer-twenty-one.png)](media/v2-update-explorer/explorer-twenty-one.png#lightbox)

## <a name="next-steps"></a>后续步骤

- 了解如何[存储和入口](./time-series-insights-update-storage-ingress.md)Azure 时间系列 Insights 预览版中。

- 在读取时时序见解预览文档[数据建模](./time-series-insights-update-tsm.md)。

- 了解[如何诊断和故障排除](./time-series-insights-update-how-to-troubleshoot.md)时序见解实例。
