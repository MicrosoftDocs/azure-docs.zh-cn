---
title: 在 Azure Sentinel 预览版中使用笔记本的搜索功能 |Microsoft Docs
description: 本文介绍如何使用 Azure Sentinel 搜索功能使用笔记本。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 1721d0da-c91e-4c96-82de-5c7458df566b
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2019
ms.author: rkarlin
ms.openlocfilehash: ffe3ae5b6aa26d154928a74e51864a0574b82c68
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228636"
---
# <a name="use-jupyter-notebooks-to-hunt-for-security-threats"></a>使用 Jupyter Notebook 来寻找安全威胁

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

Azure Sentinel 的基础是数据存储区;它结合了高性能查询、 动态架构后，并且能够适应大规模的数据卷。 Azure Sentinel 门户和所有 Azure Sentinel 工具使用公共 API 来访问此数据存储区。 相同的 API 如还有适用于外部工具[Jupyter](https://jupyter.org/)笔记本和 Python。 虽然可以在门户中执行许多常见任务，Jupyter 扩展了可以对此数据执行的操作作用的域。 它结合了内容丰富的机器学习、 可视化和数据分析库完整可编程性。 这些特性使 Jupyter 一个极具吸引力的工具，安全调查和搜索。

![示例 notebook](./media/notebooks/sentinel-nb-mapandtimeline.png)

我们整合了 Jupyter 体验到 Azure Sentinel 门户中，就可以轻松为你创建和执行笔记本，以分析你的数据。 *Kqlmagic*库提供了关联，可以查询采用来自 Azure Sentinel 和直接在笔记本中运行它们。 查询使用[Kusto 查询语言](https://kusto.azurewebsites.net/docs/query/index.html)。 多个笔记本，由一些 Microsoft 的安全分析人员，开发与 Azure Sentinel 打包在一起。 这些笔记本的一些特定方案的生成和可用作-是。 其他人要作为示例来演示方法和功能，可将复制或修改它们以使用自己的笔记本中。 其他笔记本也可能会导入从 Azure Sentinel 社区 GitHub。

使用集成的 Jupyter 体验[Azure Notebooks](https://notebooks.azure.com/)存储、 共享，并执行笔记本。 本地 （如果您的计算机上具有 Python 环境和 Jupyter），还可以运行这些 notebook 或其他 JupterHub 环境，如 Azure Databricks 中。

笔记本包含两个组件：

- 基于浏览器的接口，输入并运行查询和代码，并显示执行结果的位置。
- *内核*，它负责分析和执行代码本身。 

在 Azure 笔记本中此类内核在 Azure 上运行*免费云计算和存储*默认情况下。 如果你的笔记本包含复杂的机器学习模型或可视化效果应考虑使用功能更强大的专用如计算资源[数据科学虚拟机](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)(DSVM)。 你的帐户中的笔记本保持隐私除非选择了要共享它们。

Azure Sentinel 笔记本使用许多常用的 Python 库，如 pandas、 matplotlib、 bokeh 和其他人。 有大量的其他 Python 包可供选择，如涵盖多个方面：

- 可视化效果和图形
- 数据处理和分析
- 统计信息和执行数字计算
- 机器学习和深度学习

我们还发布了名为的包中的一些开放源代码 Jupyter 安全工具[msticpy](https://github.com/Microsoft/msticpy/)。 许多包含笔记本中使用此包。 Msticpy 工具专为帮助创建适用于搜索笔记本和调查，我们正在积极致力于新功能和改进。

初始笔记本包括：

- **引导式调查-进程警报**:可以快速会审警报通过分析受影响主机上的活动。
- **引导式搜索-Windows 主机资源管理器**:可以在主机上浏览的帐户活动、 的进程执行、 网络活动和其他事件。  
- **引导式搜索-Office365 探索**:寻找能在多个 O365 数据集中的可疑 Office 365 活动。

[Azure Sentinel 社区 GitHub 存储库](https://github.com/Azure/Azure-Sentinel)任何将来的 Azure Sentinel 笔记本的位置生成由 Microsoft 或社区贡献。

## <a name="run-a-notebook"></a>运行 notebook

在以下示例中，我们创建 Azure Sentinel 门户中，从的 Azure Notebooks 项目填充项目使用笔记本。 然后再使用这些笔记本，它是使笔记本的副本和工作副本上一个好办法。 在副本上的工作，可以安全地更新至笔记本的未来版本，而不会覆盖任何数据。

1. 在 Azure Sentinel 门户中，单击**笔记本**导航菜单中。 若要创建新的 Azure Notebooks 项目，请单击**克隆 Azure Sentinel 笔记本**或打开现有 notebook 项目单击**转到你的笔记本**。
  
   ![选择笔记本](./media/notebooks/sentinel-az-notebooks-home.png)

2. 如果选择了**克隆 Azure Sentinel 笔记本**在上一步骤中，将显示以下对话框。 单击**导入**GitHub 存储库克隆到你的 Azure Notebooks 项目。 如果没有现有的 Azure Notebooks 帐户，将提示您创建一个并登录。

   ![导入笔记本](./media/notebooks/sentinel-nb-signin-and-clone.png)

3. 创建新项目时，需要将项目命名-在一个新中使用的默认名称或类型。 不要选中**克隆以递归方式**选项-此选项是指链接 GitHub 存储库。 单击**导入**启动克隆的 GitHub 内容，这可能需要几分钟才能完成。

   ![导入笔记本](./media/notebooks/sentinel-create-nb-project.png)

4. 打开**笔记本**文件夹，以查看笔记本。 每个 notebook 指导完成为执行 hunt 或调查步骤。 从 Notebook 本身或通过简单的配置过程，可以安装库和其他依赖项所需的笔记本。 在前面的步骤中自动设置将联系你的 notebook 项目回 Azure Sentinel 订阅的配置。 已准备好对 Azure Sentinel Log Analytics 工作区运行你的笔记本。

   ![导入存储库](./media/notebooks/sentinel-open-notebook1.png)

5. 打开笔记本。 默认情况下选择免费的计算来运行 notebook （突出显示）。 如果已配置的 DSVM 使用 （见上文），选择 DSVM 并打开第一个 notebook 之前进行身份验证。 单击以将其打开 notebook。

   ![选择笔记本](./media/notebooks/sentinel-open-notebook2.png)

6. 选择 Python 版本。 当首次打开笔记本时，它可能会提示您选择的内核版本。 如果没有，选择要使用，如下所示的内核。 Python 3.6 版或更高版本应为所选的内核 (右上方的 notebook 窗口中)。

   ![选择笔记本](./media/notebooks/sentinel-select-kernel.png)

有关在 Azure Sentinel 中查询数据的快速介绍，请查看[GetStarted](https://github.com/Azure/Azure-Sentinel/blob/master/Notebooks/Get%20Started.ipynb)主笔记本文件夹中的 notebook。 可在其他示例笔记本**示例笔记本**子文件夹。 示例笔记本已保存的数据，以便更轻松地查看预期的输出 (我们建议查看它们[nbviewer](https://nbviewer.jupyter.org/))。 **操作方法**文件夹包含笔记本描述，例如： 设置您默认的 Python 版本，请配置 DSVM，从一个 notebook 和其他使用者创建 Azure Sentinel 标记书签。

这些 notebook 旨在作为这两个有用的工具和插图和代码示例，可以开发自己的笔记本中使用。

欢迎提供反馈，是否建议，请求的功能，提供笔记本、 bug 报告或改进和添加到现有笔记本。 转到[Azure Sentinel 社区 GitHub](https://github.com/Azure/Azure-Sentinel)创建问题或分叉和上载发布内容。

## <a name="next-steps"></a>后续步骤

在本文中，您学习了如何开始在 Azure Sentinel 中使用 Jupyter notebook。 要详细了解 Azure Sentinel，请参阅以下文章：

- [主动寻找威胁](hunting.md)
- [使用书签来保存搜索时的相关信息](bookmarks.md)