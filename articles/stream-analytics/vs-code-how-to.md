---
title: 了解 Azure Stream Analytics 使用 Visual Studio Code （预览版）
description: 本文介绍如何将 Azure Stream Analytics 作业导出到本地项目、 列出作业和作业视图的实体，并设置 Stream Analytics 作业的 CI/CD 管道。
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.date: 05/06/2019
ms.topic: conceptual
ms.openlocfilehash: 640a81fcb94194e2e907e9339f016bd35e279fdd
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079205"
---
# <a name="explore-azure-stream-analytics-with-visual-studio-code-preview"></a>了解 Azure Stream Analytics 使用 Visual Studio Code （预览版）

Visual Studio Code 扩展 Azure Stream Analytics 开发人员的轻型版用于管理 Stream Analytics 作业。 使用 Azure Stream Analytics 扩展中，你可以：

- [创建](quick-create-vs-code.md)、 启动和停止作业
- 将现有作业导出到本地项目
- 在本地运行的查询
- 设置 CI/CD 管道

## <a name="export-a-job-to-a-local-project"></a>将作业导出到本地项目

若要将作业导出到本地项目中，找到你想要在导出的作业**Stream Analytics 资源管理器**在 Visual Studio code 中。 然后选择你的项目的文件夹。 项目会导出到您选择的文件夹，可以继续管理该作业从 Visual Studio Code。 使用 Visual Studio Code 用于管理 Stream Analytics 作业的详细信息，请参阅 Visual Studio Code[快速入门](quick-create-vs-code.md)。

![在 Visual Studio Code 中的导出 ASA 作业](./media/vs-code-how-to/export-job.png)

## <a name="run-queries-locally"></a>在本地运行的查询

适用于 Visual Studio Code 的 Azure Stream Analytics 扩展可用于测试你的 Stream Analytics 作业使用示例数据在本地。

1. 一旦创建 Stream Analytics 作业，使用**Ctrl + Shift + P**以打开该命令货盘。 然后键入，然后选择**ASA:添加输入**。

    ![在 Visual Studio code 中添加 ASA 输入](./media/vs-code-how-to/add-input.png)

2. 选择**本地输入**。

    ![在 Visual Studio code 中添加 ASA 本地输入](./media/vs-code-how-to/add-local-input.png)

3. 选择 **+ 新本地输入**。

    ![添加新的 ASA 本地 Visual Studio code 中输入](./media/vs-code-how-to/add-new-local-input.png)

4. 输入在查询中使用的相同输入的别名。

    ![添加新的 ASA 本地输入的别名](./media/vs-code-how-to/new-local-input-alias.png)

5. 在中**LocalInput_DefaultLocalStream.json**文件中，输入你的本地数据文件的所在位置的文件路径。

    ![在 Visual Studio 中输入本地文件路径](./media/vs-code-how-to/local-file-path.png)

6. 返回到您的查询编辑器，并选择**在本地运行**。

    ![选择在查询编辑器中本地运行](./media/vs-code-how-to/run-locally.png)

## <a name="set-up-a-cicd-pipeline"></a>设置 CI/CD 管道

可以启用持续集成和部署为使用 Azure Stream Analytics 作业**asa cicd 工具**NPM 包。 NPM 包提供的工具用于 Visual Studio Code 的 Stream Analytics 项目自动部署。 无需安装 Visual Studio Code，可以在 Windows、 macOS 和 Linux 上使用它。

一旦您有[下载包](https://usqldownload.blob.core.windows.net/ext/asa/asa-cicd-0.0.1-preview-beta.tar)，使用以下命令输出的 Azure 资源管理器模板。 如果**outputPath**未指定，则模板将被放入**部署**下的项目文件夹**bin**文件夹。

```powershell
asa-cicd build -scriptPath <scriptFullPath> -outputPath <outputPath>
```

## <a name="next-steps"></a>后续步骤

* [在 Visual Studio Code （预览版） 中创建 Azure Stream Analytics 云作业](quick-create-vs-code.md)