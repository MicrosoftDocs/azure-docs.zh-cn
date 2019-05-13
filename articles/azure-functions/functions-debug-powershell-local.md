---
title: PowerShell Azure Functions 在本地进行调试
description: 了解如何使用 PowerShell 来挖掘一些功能。
services: functions
documentationcenter: na
author: tylerleonhardt
manager: jeconnoc
ms.service: azure-functions
ms.devlang: powershell
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: tyleonha, glenga
ms.openlocfilehash: b699379448863c8df84fda0e059fc10846c09931
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230074"
---
# <a name="debug-powershell-azure-functions-locally"></a>PowerShell Azure Functions 在本地进行调试

Azure Functions，可为 PowerShell 脚本函数的开发。

[!INCLUDE [functions-powershell-preview-note](../../includes/functions-powershell-preview-note.md)]

根据需要使用以下标准的开发工具的任何 PowerShell 脚本，您可以本地调试 PowerShell 函数：

* [Visual Studio Code](https://code.visualstudio.com/)：使用 PowerShell 扩展，可提供完整的 PowerShell 开发体验的 Microsoft 的免费、 轻型且开源的文本编辑器。
* PowerShell 控制台中：调试使用想要用于调试任何其他 PowerShell 进程的相同命令。

[Azure Functions Core Tools](functions-run-local.md)支持本地调试 Azure 函数，其中包括 PowerShell 函数。

## <a name="example-function-app"></a>示例函数应用

本文中使用函数应用的单个 HTTP 触发函数，并具有以下文件：

```
PSFunctionApp
 | - HttpTriggerFunction
 | | - run.ps1
 | | - function.json
 | - local.settings.json
 | - host.json
 | - profile.ps1
```

此函数应用是类似于获取完成后[PowerShell 快速入门](functions-create-first-function-powershell.md)。

中的函数代码`run.ps1`看起来像以下脚本：

```powershell
param($Request)

$name = $Request.Query.Name

if($name) {
    $status = 200
    $body = "Hello $name"
}
else {
    $status = 400
    $body = "Please pass a name on the query string or in the request body."
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = $status
    Body = $body
})
```

## <a name="set-the-attach-point"></a>设置附加点

若要调试任何 PowerShell 函数，该函数需要停止调试器要附加的。 `Wait-Debugger` Cmdlet 停止执行并等待调试程序。

需要做的就是添加对的调用`Wait-Debugger`cmdlet 正上方`if`语句，如下所示：

```powershell
param($Request)

$name = $Request.Query.Name

# This is where we will wait for the debugger to attach
Wait-Debugger

if($name) {
    $status = 200
    $body = "Hello $name"
}
# ...
```

在启动调试`if`语句。 

使用`Wait-Debugger`到位，你现在可以调试使用 Visual Studio Code 或 PowerShell 控制台中的函数。

## <a name="debug-in-visual-studio-code"></a>在 Visual Studio Code 中调试

若要调试您在 Visual Studio Code 的 PowerShell 函数，必须针对 Visual Studio Code 具有下列扩展名：

* [PowerShell](/powershell/scripting/components/vscode/using-vscode)
* [Azure Functions](functions-create-first-function-vs-code.md)

在安装后的 PowerShell 和 Azure Functions 扩展，加载现有的函数应用项目。 此外可以[创建函数项目](functions-create-first-function-vs-code.md)。

>[!NOTE]
> 你的项目不应具有所需的配置文件，系统会提示将其添加。

### <a name="start-the-function-app"></a>启动函数应用

验证`Wait-Debugger`在函数中你想要将调试器附加设置。  使用`Wait-Debugger`添加，您可以调试使用 Visual Studio Code 在函数应用。

选择**调试**窗格，然后**附加到 PowerShell 函数**。

![调试器](https://user-images.githubusercontent.com/2644648/56166073-8a7b3780-5f89-11e9-85ce-36ed38e221a2.png)

此外可以按 F5 键启动调试。

开始调试操作执行以下任务：

* 运行`func extensions install`终端安装任何所需的函数应用的 Azure Functions 扩展中。
* 运行`func host start`在终端中启动 Functions 主机中的函数应用。
* 将 PowerShell 调试程序附加到 Functions 运行时中的 PowerShell 运行空间。

使用 function app 中运行，您需要一个单独的 PowerShell 控制台，可调用 HTTP 触发的函数。

在这种情况下，PowerShell 控制台是客户端。 `Invoke-RestMethod`用于触发函数。

在 PowerShell 控制台中，运行以下命令：

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

您会注意到不立即返回的响应。 这是因为`Wait-Debugger`已附加调试器和 PowerShell 执行只要它无法进入中断模式。 这是由于[BreakAll 概念](#breakall-might-cause-your-debugger-to-break-in-an-unexpected-place)，这在稍后进行说明。 按后`continue`按钮时，调试器现在的一行上中断紧靠其后`Wait-Debugger`。

此时，已附加调试器，可以执行所有标准调试器操作。 在 Visual Studio Code 中使用调试器的详细信息，请参阅[官方文档](https://code.visualstudio.com/Docs/editor/debugging#_debug-actions)。

继续并完全调用您的脚本后，你会看到：

* 未在 PowerShell 控制台`Invoke-RestMethod`已返回了结果
* 在 Visual Studio Code 中的 PowerShell 集成控制台正在等待要执行脚本

后续调用相同的函数，在 PowerShell 中紧靠其后扩展将中断调试器时的时间`Wait-Debugger`。

## <a name="debugging-in-a-powershell-console"></a>在 PowerShell 控制台中进行调试

>[!NOTE]
> 本部分假设你已阅读[Azure Functions Core Tools docs](functions-run-local.md)并且知道如何使用`func host start`命令以启动 function app。

控制台中，打开`cd`到目录中的函数应用，并运行以下命令：

```sh
func host start
```

与运行的函数应用和`Wait-Debugger`到位，你可以附加到进程。 您需要两个更多的 PowerShell 控制台。

其中一个控制台充当客户端。 由此，调用`Invoke-RestMethod`来触发该函数。 例如，可以运行以下命令：

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

您会注意到它不会返回响应，这是一个结果的`Wait-Debugger`。 PowerShell 运行空间现在正在等待附加调试器。 现在可以开始附加。

在其他 PowerShell 控制台中，运行以下命令：

```powershell
Get-PSHostProcessInfo
```

此 cmdlet 返回一个表，如以下输出所示：

```output
ProcessName ProcessId AppDomainName
----------- --------- -------------
dotnet          49988 None
pwsh            43796 None
pwsh            49970 None
pwsh             3533 None
pwsh            79544 None
pwsh            34881 None
pwsh            32071 None
pwsh            88785 None
```

请记下的`ProcessId`与表中的项`ProcessName`作为`dotnet`。 此过程是 function app。

接下来，运行以下代码片段：

```powershell
# This enters into the Azure Functions PowerShell process.
# Put your value of `ProcessId` here.
Enter-PSHostProcess -Id $ProcessId

# This triggers the debugger.
Debug-Runspace 1
```

启动后，调试器将中断，并演示类似于以下输出：

```
Debugging Runspace: Runspace1

To end the debugging session type the 'Detach' command at the debugger prompt, or type 'Ctrl+C' otherwise.

At /Path/To/PSFunctionApp/HttpTriggerFunction/run.ps1:13 char:1
+ if($name) { ...
+ ~~~~~~~~~~~
[DBG]: [Process:49988]: [Runspace1]: PS /Path/To/PSFunctionApp>>
```

此时，在停止在断点处[PowerShell 调试器](/powershell/module/microsoft.powershell.core/about/about_debuggers)。 在这里，您可以执行所有常见的调试操作、 单步执行、 单步执行、 继续、 退出，请等。 若要查看完整的控制台中可用的调试命令集，请运行`h`或`?`命令。

此外可以在此级别上设置断点`Set-PSBreakpoint`cmdlet。

继续并完全调用您的脚本后，你会看到：

* PowerShell 控制台中，执行`Invoke-RestMethod`现在已返回了结果。
* PowerShell 控制台中，执行`Debug-Runspace`等待要执行的脚本。

可以再次调用同一个函数 (使用`Invoke-RestMethod`为例) 并在调试器介入之后的右`Wait-Debugger`命令。

## <a name="considerations-for-debugging"></a>有关调试的注意事项

调试函数代码时，请记住以下问题。

### <a name="breakall-might-cause-your-debugger-to-break-in-an-unexpected-place"></a>`BreakAll` 可能会导致在调试器中断的意外位置

PowerShell 扩展插件使用`Debug-Runspace`，后者又依赖于 PowerShell 的`BreakAll`功能。 此功能会告知 PowerShell 可执行的第一个命令在停止。 此行为使你能够设置断点调试运行空间中。

Azure Functions 运行时在几个命令运行前实际调用你`run.ps1`编写的脚本，因此可以在调试器最终中的重大`Microsoft.Azure.Functions.PowerShellWorker.psm1`或`Microsoft.Azure.Functions.PowerShellWorker.psd1`。

应会发生此中断，运行`continue`或`c`命令以跳过此断点。 您然后预期的断点处停止。

## <a name="next-steps"></a>后续步骤

若要了解有关开发使用 PowerShell 的功能的详细信息，请参阅[Azure Functions PowerShell 开发人员指南](functions-reference-powershell.md)。
