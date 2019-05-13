---
title: 使用 Unity 的项目音响效果快速入门
titlesuffix: Azure Cognitive Services
description: 使用示例内容，在 Unity 中试验项目音响效果设计控件，并部署到 Windows 桌面。
services: cognitive-services
author: kegodin
manager: nitinme
ms.service: cognitive-services
ms.subservice: acoustics
ms.topic: quickstart
ms.date: 03/20/2019
ms.author: kegodin
ms.openlocfilehash: 1c790e0fa726c719d5b888d42b5f59739777566b
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64917123"
---
# <a name="project-acoustics-unity-quickstart"></a>项目音响效果 Unity 快速入门
使用 Unity 的项目音响效果示例内容来试验模拟支持的设计控件。

所需软件：
* 用于 Windows 的 [Unity 2018.2 及更高版本](https://unity3d.com)
* [项目音响效果示例内容包](https://www.microsoft.com/download/details.aspx?id=57346)

示例包中包括什么？
* 具有几何、声源和游戏控件的 Unity 场景
* 项目音响效果插件 
* 场景的烘培音效资产

## <a name="import-the-sample-package"></a>导入示例包
将示例包导入到新的 Unity 项目。 
* 在 Unity 中，转到“资产”>“导入包”>“自定义包...”

    ![Unity 导入包选项的屏幕截图](media/import-package.png)  

* 选择 ProjectAcoustics.unitypackage

如果要将包导入到现有项目中，请参阅 [Unity 集成](unity-integration.md)以获取其他步骤和说明。

## <a name="restart-unity"></a>重启 Unity
音效工具包的制作部分需要 .NET 4.x 脚本编写运行时版本。 包导入将更新 Unity 播放器设置。 重启 Unity，使此设置生效。

可以通过打开“播放器设置”来验证此设置是否生效：

![Unity 播放器设置面板的屏幕截图](media/player-settings.png)

![选中了 .NET 4.5 的 Unity 播放器设置面板的屏幕截图](media/net45.png)

## <a name="experiment-with-design-controls"></a>试验设计控件
打开 ProjectAcousticsSample 文件夹中的示例场景，然后单击 Unity 编辑器中的“播放”按钮。 使用 W、A、S、D 和鼠标进行移动。 若要比较有音响效果和没有音响效果的场景声音，请按下 **R** 按钮，直到覆盖文本变成红色，并显示“音响效果:已禁用”。 若要查看更多控件的键盘快捷方式，请按 **F1**。 通过右键单击以选择要执行的操作，然后左键单击以执行操作，也可以使用控件。

脚本 AcousticsAdjust 附加到示例场景中的声音源，这会启用基于源的设计参数。 

![Unity AcousticsAdjust 脚本的屏幕截图](media/acoustics-adjust.png)

以下介绍一些可以使用提供的控件生成的效果。 有关每个控件的详细信息，请参阅[项目音响效果 Unity 设计教程](unreal-workflow.md)。

### <a name="modify-distance-based-attenuation"></a>修改基于距离的衰减
项目音响效果 Unity 空间音响插件提供的音频 DSP 遵守内置于 Unity 编辑器的基于距离的按源衰减。 针对基于距离的衰减的控件位于“三维声音设置”下的声源“检查器”面板的“音频源”组件中：

![Unity 距离衰减选项面板的屏幕截图](media/distance-attenuation.png)

项目音响效果在以播放器位置为中心的“模拟区域”框中执行计算。 由于示例包中的音效资产烘焙的模拟区域尺寸为播放器周围 45 米，因此声音衰减应设计为在约 45 米处降至 0。

### <a name="modify-occlusion-and-transmission"></a>修改闭塞和传输
* 如果“闭塞”乘数大于 1（默认值为 1），闭塞将被扩大。 将其设置为小于 1 会使闭塞效果更精细。

* 若要启用穿墙传输，请将“传输 (dB)”滑块从最低级别移开。 

### <a name="modify-wetness-for-a-source"></a>修改源湿度
* 若要更改湿度随距离变化的速度，请使用“可感知距离弯曲”。 “项目音响效果”通过模拟计算整个空间的湿度级别，湿度级别会随距离平稳变化，并提供可感知的距离提示。增加距离弯曲会通过增加与距离相关的湿度级别来扩大这种效果。 如果弯曲值小于 1，基于距离的混响变化会更加精细。 通过调整“湿度 (dB)”，还可以在更精细的细节中调整此效果。

* 通过调整“衰减时间刻度”来增加整个空间的衰减时间。 如果特定源侦听器位置对的模拟结果是 1.5 秒的衰减时间，并且“衰减时间刻度”设置为 2，则应用于源的衰减时间为 3 秒。

## <a name="next-steps"></a>后续步骤
* 阅读[基于 Unity 的项目音响效果设计控件](unity-workflow.md)的完整详细信息
* 进一步探索[设计过程](design-process.md)中的概念
* [创建 Azure 帐户](create-azure-account.md)以了解预先烘焙和烘焙过程

