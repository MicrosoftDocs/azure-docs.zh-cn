---
title: 在 Azure 中创建 HTTP 触发的函数
description: 了解如何使用 Azure Functions Core Tools 和 Azure CLI 在 Azure 中创建你的第一个 Python 函数。
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.date: 04/24/2019
ms.topic: quickstart
ms.service: azure-functions
ms.custom: mvc
ms.devlang: python
manager: jeconnoc
ms.openlocfilehash: 7e2b3424c3d8edc931054dea062280ea7789dc44
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143061"
---
# <a name="create-an-http-triggered-function-in-azure"></a>在 Azure 中创建 HTTP 触发的函数

[!INCLUDE [functions-python-preview-note](../../includes/functions-python-preview-note.md)]

本文介绍如何使用命令行工具创建在 Azure Functions 中运行的 Python 项目。 你创建的此函数由 HTTP 请求触发。 最后，发布项目，在 Azure 中作为[无服务器函数](functions-scale.md#consumption-plan)运行。

本文是 Azure Functions 的两个快速入门之一。 完成本文后，请将[ Azure 存储队列输出绑定添加](functions-add-output-binding-storage-queue-python.md)到你的函数。

## <a name="prerequisites"></a>先决条件

在开始之前，必须具有以下各项：

+ 安装 [Python 3.6](https://www.python.org/downloads/)。

+ 安装 [Azure Functions Core Tools](./functions-run-local.md#v2) 版本 2.6.666 或更高版本。

+ 安装 [Azure CLI](/cli/azure/install-azure-cli) 版本 2.x 或更高版本。

+ 一个有效的 Azure 订阅。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-and-activate-a-virtual-environment"></a>创建并激活虚拟环境

要本地开发和测试 Python 函数，必须在 Python 3.6 环境中工作。 运行以下命令来创建并激活一个名为 `.env` 的虚拟环境。

### <a name="bash-or-a-terminal-window"></a>Bash 或终端窗口：

```bash
python3.6 -m venv .env
source .env/bin/activate
```

### <a name="powershell-or-a-windows-command-prompt"></a>PowerShell 或 Windows 命令提示符：

```powershell
py -3.6 -m venv .env
.env\scripts\activate
```

剩下的命令在虚拟环境内运行。

## <a name="create-a-local-functions-project"></a>创建本地 Functions 项目

Functions 项目等效于 Azure 中的函数应用。 它可能具有多个函数，这些函数具有相同的本地和托管配置。

在虚拟环境中，运行以下命令，选择“python”作为工作器运行时。

```command
func init MyFunctionProj
```

创建了名为 MyFunctionProj 的文件夹，该文件夹中包含以下三个文件：

* `local.settings.json` 用于在本地运行时存储应用设置和连接字符串。 此文件不会被发布到 Azure。
* `requirements.txt` 包含要在发布到 Azure 时安装的包列表。
* `host.json` 包含在函数应用中影响所有函数的全局配置选项。 此文件会被发布到 Azure。

导航到新的 MyFunctionProj 文件夹：

```command
cd MyFunctionProj
```

接下来，请更新 host.json 文件以启用扩展捆绑包。  

## <a name="reference-bindings"></a>引用绑定

扩展捆绑包可以更轻松地在以后添加绑定扩展。 它无需安装 .NET Core 2.x SDK。 扩展捆绑包需要使用 Core Tools 的 2.6.1071 版或更高版本。 

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

现在，可以将函数添加到项目。

## <a name="create-a-function"></a>创建函数

若要将函数添加到项目中，请运行以下命令：

```command
func new
```

选择“HTTP 触发器”模板，键入 `HttpTrigger` 作为函数名称，然后按 Enter。

创建了名为 HttpTrigger 的子文件夹，其中包含以下文件：

* function.json：定义函数、触发器和其他绑定的配置文件。 查看此文件，看到 `scriptFile` 的值指向包含函数的文件，而调用触发器和绑定在 `bindings` 数组中定义。

  每个绑定都需要一个方向、类型和唯一名称。 HTTP 触发器具有类型为 [`httpTrigger`](functions-bindings-http-webhook.md#trigger) 的输入绑定和类型为 [`http`](functions-bindings-http-webhook.md#output) 的输出绑定。

* __init__.py：作为 HTTP 触发函数的脚本文件。 查看此脚本，看到它包含默认值 `main()`。 触发器中的 HTTP 数据使用 `req` 命名绑定参数传递到此函数。 根据 function.json 中的定义，`req` 是 [azure.functions.HttpRequest class](/python/api/azure-functions/azure.functions.httprequest) 的一个实例。 

    在 function.json中定义为 `$return` 的返回对象是 [azure.functions.HttpResponse class](/python/api/azure-functions/azure.functions.httpresponse) 的一个实例。 要了解详细信息，请参阅 [Azure Functions HTTP 触发器和绑定](functions-bindings-http-webhook.md)。

## <a name="run-the-function-locally"></a>在本地运行函数

以下命令启动函数应用，该应用使用 Azure 中相同的 Azure Functions 运行时本地运行。

```bash
func host start
```

Functions 主机启动时，会写入类似于以下输出的内容。为了可读性，这些输出已经被截断：

```output

                  %%%%%%
                 %%%%%%
            @   %%%%%%    @
          @@   %%%%%%      @@
       @@@    %%%%%%%%%%%    @@@
     @@      %%%%%%%%%%        @@
       @@         %%%%       @@
         @@      %%%       @@
           @@    %%      @@
                %%
                %

...

Content root path: C:\functions\MyFunctionProj
Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

...

Http Functions:

        HttpTrigger: http://localhost:7071/api/MyHttpTrigger

[8/27/2018 10:38:27 PM] Host started (29486ms)
[8/27/2018 10:38:27 PM] Job host started
```

从运行时输出中复制 `HttpTrigger` 函数的 URL，将其粘贴到浏览器的地址栏中。 将查询字符串 `?name=<yourname>` 追加到此 URL 并执行请求。 下面演示浏览器中本地函数返回的对 GET 请求的响应：

![在浏览器中进行本地测试](./media/functions-create-first-function-python/function-test-local-browser.png)

在本地运行函数以后，即可在 Azure 中创建函数应用和其他必需的资源。

[!INCLUDE [functions-create-resource-group](../../includes/functions-create-resource-group.md)]

[!INCLUDE [functions-create-storage-account](../../includes/functions-create-storage-account.md)]

## <a name="create-a-function-app-in-azure"></a>在 Azure 中创建一个函数应用

函数应用提供了用于执行函数代码的环境。 可以使用它将函数分组为一个逻辑单元，以便更轻松地管理、部署和共享资源。

运行以下命令，并使用唯一的函数应用名称替换 `<APP_NAME>` 占位符，使用存储帐户名称替换 `<STORAGE_NAME>`。 `<APP_NAME>` 也是函数应用的默认 DNS 域。 此名称在 Azure 的所有应用中必须独一无二。

```azurecli-interactive
az functionapp create --resource-group myResourceGroup --os-type Linux \
--consumption-plan-location westeurope  --runtime python \
--name <APP_NAME> --storage-account  <STORAGE_NAME>
```

> [!NOTE]
> Linux 和 Windows 应用不能托管在同一个资源组中。 如果名为 `myResourceGroup` 的现有资源组有 Windows 函数应用或 Web 应用，必须使用其他资源组。

现在，可以将本地函数项目发布到 Azure 中的函数应用了。

[!INCLUDE [functions-publish-project](../../includes/functions-publish-project.md)]

[!INCLUDE [functions-test-function-code](../../includes/functions-test-function-code.md)]

## <a name="next-steps"></a>后续步骤

已使用 HTTP 触发函数创建 Python 函数项目，在本地计算机上运行该项目，并将其部署到 Azure。 现在，通过以下方式扩展函数

> [!div class="nextstepaction"]
> [添加 Azure 存储队列输出绑定](functions-add-output-binding-storage-queue-python.md)
