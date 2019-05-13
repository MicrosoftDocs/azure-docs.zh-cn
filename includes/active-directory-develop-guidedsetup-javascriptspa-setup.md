---
title: include 文件
description: include 文件
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: nacanuma
ms.custom: include file
ms.openlocfilehash: a00bc7a05af9e329494a11f9bee444827cbebf38
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198840"
---
## <a name="setting-up-your-web-server-or-project"></a>设置 Web 服务器或项目

> 想要改为下载此示例的项目？
> - [下载项目文件](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip)，以使用本地 Web 服务器（如 Node）运行文件
>
> 或
> - （可选）[下载 Visual Studio 项目](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip)，以使用 IIS 服务器运行项目
>
> 然后跳到[配置步骤](#register-your-application)，在执行前配置代码示例。

## <a name="prerequisites"></a>先决条件
运行本教程需要将本地 Web 服务器（如 [Node.js](https://nodejs.org/en/download/)、[.NET Core](https://www.microsoft.com/net/core) 或 IIS Express）与 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 集成。

如果使用 Node.js 运行该项目，请安装 IDE（如 [Visual Studio Code](https://code.visualstudio.com/download)）以编辑项目文件。

本指南中的说明同时基于 Node.js 和 Visual Studio 2017，但可任意使用其他开发环境或 Web 服务器。

## <a name="create-your-project"></a>创建项目

> ### <a name="option-1-node-other-web-servers"></a>选项 1：Node/其他 Web 服务器
> 请确保已安装 [Node.js](https://nodejs.org/en/download/)，然后执行以下步骤：
> - 创建一个文件夹以托管应用程序。

<p><!-- -->

> ### <a name="option-2-visual-studio"></a>选项 2：Visual Studio
> 如果正在使用 Visual Studio 创建新项目，请按照以下步骤创建新的 Visual Studio 解决方案：
> 1.    在 Visual Studio 中：“文件”>“新建”>“项目”
> 2.    在 Visual C#\Web 下，选择“ASP.NET Web 应用程序(.NET Framework)”
> 3.    输入应用程序的名称并选择“确定”
> 4.    在“新建 ASP.NET Web 应用程序”下，选择“空”

## <a name="create-your-single-page-applications-ui"></a>创建单页应用程序的 UI
1. 为 JavaScript SPA 创建 `index.html` 文件。 如果使用 Visual Studio，请选择项目（项目根文件夹），然后右键单击并选择：“添加”>“新建项”>“HTML 页”并将其命名为 index.html。

2. 将以下代码添加到页面：
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Quickstart for MSAL JS</title>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.3.4/bluebird.min.js"></script>
       <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.js"></script>
   </head>
   <body>
       <h2>Welcome to MSAL.js Quickstart</h2><br/>
       <h4 id="WelcomeMessage"></h4>
       <button id="SignIn" onclick="signIn()">Sign In</button><br/><br/>
       <pre id="json"></pre>
       <script>
           //JS code
       </script>
   </body>
   </html>
   ```

   > [!TIP]
   > 可以将上面脚本中的 MSAL.js 版本替换为，[MSAL.js 版本](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)下最新发布的版本。
