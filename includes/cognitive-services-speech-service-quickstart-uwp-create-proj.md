---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 2/20/2019
ms.author: erhopf
ms.openlocfilehash: b5c61ecb74c0fcdacc6024a74565f05b036e2ac6
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65081185"
---
1. 启动 Visual Studio 2017。

1. 确保“通用 Windows 平台开发”工作负荷可用。 在 Visual Studio 菜单栏中选择“工具” > “获取工具和功能”，以便打开 Visual Studio 安装程序。 如果此工作负荷已启用，请关闭对话框。

    ![Visual Studio 安装程序的屏幕截图，其中突出显示了“工作负荷”选项卡](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-workload.png)

    否则，请选择“.NET 跨平台开发”旁边的框，然后选择对话框右下角的“修改”。 安装新功能需要一定的时间。

1. 创建空白的 Visual C# 通用 Windows 应用。 首先，从菜单中选择“文件” > “新建” > “项目”。 在“新建项目”对话框的左窗格中，展开“已安装” > “Visual C#” > “Windows 通用”， 然后选择“空白应用(通用 Windows)”。 在项目名称处，输入 helloworld。

    ![“新建项目”对话框屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-01-new-blank-app.png)

1. Speed SDK 要求为 Windows 10 Fall Creators Update 或更高版本生成应用程序。 在弹出的“新建通用 Windows 平台项目”窗口中，选择“Windows 10 Fall Creators Update(10.0; 内部版本 16299)”作为“最低版本”。 在“目标版本”框中，选择此版本或任何更高版本，然后单击“确定”。

    ![“新式通用 Windows 平台项目”窗口的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-02-new-uwp-project.png)

1. 如果运行 64 位 Windows，可以使用 Visual Studio 工具栏中的下拉菜单将生成平台切换为 `x64`。 （64 位 Windows 可以运行 32 位应用程序，因此可以根据自己的偏好让它设置为 `x86`。）

   ![Visual Studio 工具栏的屏幕截图，其中突出显示了 x64](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-03-switch-to-x64.png)

   > [!NOTE]
   > 语音 SDK 仅支持 Intel 兼容的处理器。 目前不支持 ARM。

1. 安装并引用[语音 SDK NuGet 包](https://aka.ms/csspeech/nuget)。 在解决方案资源管理器中，右键单击该解决方案，并选择“为解决方案管理 NuGet 包”。

    ![解决方案资源管理器的屏幕截图，其中突出显示了“为解决方案管理 NuGet 包”选项](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-04-manage-nuget-packages.png)

1. 在右上角的“包源”字段中，选择“nuget.org”。搜索 `Microsoft.CognitiveServices.Speech` 包，并将其安装到“helloworld”项目中。

    ![“管理解决方案包”对话框的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-05-nuget-install-1.0.0.png "安装 NuGet 包")

1. 接受显示的许可证即可开始安装 NuGet 包。

    ![“接受许可证”对话框的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-06-nuget-license.png "接受许可证")

1. 包管理器控制台中将显示以下输出行。

   ```text
   Successfully installed 'Microsoft.CognitiveServices.Speech 1.5.0' to helloworld
   ```

1. 由于应用程序使用麦克风进行语音输入，因此请向项目添加“麦克风”功能。 在解决方案资源管理器中，双击“Package.appxmanifest”，编辑应用程序清单。 然后切换到“功能”选项卡，选择“麦克风”功能的框，然后保存所做的更改。

   ![Visual Studio 应用程序清单的屏幕截图，其中突出显示了“功能”和“麦克风”](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-07-capabilities.png)
