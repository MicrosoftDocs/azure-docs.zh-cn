---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 2/20/2019
ms.author: erhopf
ms.openlocfilehash: af61d975b540787b9b54d9fdea66773c10aeb6ac
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65020788"
---
1. 启动 Visual Studio 2017。

1. 确保“使用 C++ 进行的桌面开发”工作负荷可用。 在 Visual Studio 菜单栏中选择“工具” > “获取工具和功能”，以便打开 Visual Studio 安装程序。 如果此工作负荷已启用，请跳到下一步。

    ![Visual Studio 工作负荷选项卡的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-cpp-workload.png)

    否则，请勾选“使用 C++ 进行的桌面开发”旁边的框。

1. 确保 **NuGet 包管理器**组件可用。 切换到 Visual Studio 安装程序对话框的“单个组件”选项卡，选择“NuGet 包管理器”（如果尚未启用）。

      ![Visual Studio 的“单个组件”选项卡的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-nuget-package-manager.png)

1. 如需启用 C++ 工作负荷或 NuGet，请选择对话框右下角的“修改”。 安装新功能需要一定的时间。 如果两项功能均已启用，请改为关闭对话框。

1. 创建新的 Visual C++ Windows 桌面 Windows 控制台应用程序。 首先，从菜单中选择“文件” > “新建” > “项目”。 在“新建项目”对话框的左窗格中，展开“已安装” > “Visual C++” > “Windows 桌面”， 然后选择“Windows 控制台应用程序”。 在项目名称处，输入 helloworld。

    ![“新建项目”对话框屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-01-new-console-app.png)

1. 如果运行 64 位 Windows，可以使用 Visual Studio 工具栏中的下拉菜单将生成平台切换为 `x64`。 （64 位版本的 Windows 可以运行 32 位应用程序，因此这不是必需的。）

    ![Visual Studio 工具栏的屏幕截图，其中突出显示了 x64 选项](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-02-switch-to-x64.png)

1. 在解决方案资源管理器中，右键单击该解决方案，然后选择“为解决方案管理 NuGet 包”。

    ![解决方案资源管理器的屏幕截图，其中突出显示了“为解决方案管理 NuGet 包”选项](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-03-manage-nuget-packages.png)

1. 在右上角的“包源”字段中，选择“nuget.org”。搜索 `Microsoft.CognitiveServices.Speech` 包，并将其安装到“helloworld”项目中。

    ![“管理解决方案包”对话框的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-04-nuget-install-1.0.0.png)

    > [!NOTE]
    > 认知服务语音 SDK 的当前版本是 `1.5.0`。

1. 接受显示的许可证即可开始安装 NuGet 包。

    ![“接受许可证”对话框的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-05-nuget-license.png)

安装此包后，在包管理器控制台中会显示一条确认消息。
