---
title: 教程：在 Azure 中模拟访问读取访问冗余存储时发生的故障 | Microsoft Docs
description: 模拟访问读取访问异地冗余存储时发生的错误
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 01/03/2019
ms.author: tamram
ms.reviewer: artek
ms.openlocfilehash: 1f5c404e410ded2714be761e35060f3c07379bd3
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508097"
---
# <a name="tutorial-simulate-a-failure-in-accessing-read-access-redundant-storage"></a>教程：模拟访问读取访问冗余存储时发生的故障

本教程是一个系列中的第二部分。 本教程通过模拟一个故障，介绍[读取访问异地冗余](../common/storage-redundancy-grs.md#read-access-geo-redundant-storage) (RA-GRS) 的优点。

若要模拟故障，可以使用[静态路由](#simulate-a-failure-with-an-invalid-static-route)或 [Fiddler](#simulate-a-failure-with-fiddler)。 使用这两种方法都可以模拟这样一个故障：向[读取访问异地冗余](../common/storage-redundancy-grs.md#read-access-geo-redundant-storage) (RA-GRS) 存储帐户的主终结点发送请求，导致应用程序改从辅助终结点读取内容。

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

本系列教程的第二部分将介绍如何：

> [!div class="checklist"]
> * 运行和暂停应用程序
> * 使用[无效的静态路由](#simulate-a-failure-with-an-invalid-static-route)或 [Fiddler](#simulate-a-failure-with-fiddler) 模拟故障
> * 模拟主终结点还原

## <a name="prerequisites"></a>先决条件

开始学习本教程之前，请完成前一教程：[使用 Azure 存储实现应用程序数据的高可用性][previous-tutorial]。

若要使用静态路由模拟故障，需要使用权限提升的命令提示符。

若要使用 Fiddler 模拟故障，请下载并[安装 Fiddler](https://www.telerik.com/download/fiddler)

## <a name="simulate-a-failure-with-an-invalid-static-route"></a>使用无效的静态路由模拟失败

对于向[读取访问异地冗余](../common/storage-redundancy-grs.md#read-access-geo-redundant-storage) (RA-GRS) 存储帐户的主终结点发出的所有请求，可以创建一个无效的静态路由。 本教程使用本地主机作为网关来路由向存储帐户发出的请求。 使用本地主机作为网关会导致向存储帐户主终结点发出的所有请求都以循环方式返回到主机内，随后导致请求失败。 执行以下步骤，使用无效的静态路由模拟失败和主终结点还原。

### <a name="start-and-pause-the-application"></a>启动和暂停应用程序

使用[前一篇教程][previous-tutorial]中的说明启动示例并下载测试文件（确认该文件来自主存储）。 然后，可以根据目标平台手动暂停示例或等待出现提示。

### <a name="simulate-failure"></a>模拟故障

当应用程序处于暂停状态时，在 Windows 上以管理员身份打开命令提示符，或者在 Linux 上以 root 身份运行终端。

获取有关存储帐户主终结点域的信息，方法是在命令提示符下或终端中输入以下命令（请将 `STORAGEACCOUNTNAME` 替换为存储帐户的名称）。

```
nslookup STORAGEACCOUNTNAME.blob.core.windows.net
```

将存储帐户的 IP 地址复制到文本编辑器中，供以后使用。

若要获取本地主机的 IP 地址，请在 Windows 命令提示符处键入 `ipconfig`，或者在 Linux 终端上键入 `ifconfig`。

若要添加目标主机的静态路由，请在 Windows 命令提示符下或 Linux 终端中键入以下命令（请将 `<destination_ip>` 替换为存储帐户 IP 地址，将 `<gateway_ip>` 替换为本地主机 IP 地址）。

#### <a name="linux"></a>Linux

```
route add <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```
route add <destination_ip> <gateway_ip>
```

在运行示例的窗口中恢复应用程序，或者按下相应的键下载示例文件，并确认该文件来自辅助存储。 然后可以再次暂停示例或等待出现提示。

### <a name="simulate-primary-endpoint-restoration"></a>模拟主终结点还原

若要模拟主终结点恢复正常，请从路由表中删除无效的静态路由。 这样就可以让发送到主终结点的所有请求通过默认网关来路由。 在 Windows 命令提示符下或 Linux 终端中键入以下命令。

#### <a name="linux"></a>Linux

```
route del <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```
route delete <destination_ip>
```

然后可以恢复应用程序，或者按下相应的键再次下载示例文件，这一次请确认该文件来自主存储。

## <a name="simulate-a-failure-with-fiddler"></a>使用 Fiddler 模拟失败

若要使用 Fiddler 来模拟失败，请针对向 RA-GRS 存储帐户的主终结点发出的请求注入一个失败的响应。

以下部分介绍如何使用 fiddler 模拟失败和主终结点还原。

### <a name="launch-fiddler"></a>启动 fiddler

打开 Fiddler，依次选择“规则”和“自定义规则”。

![自定义 Fiddler 规则](media/storage-simulate-failure-ragrs-account-app/figure1.png)

此时，Fiddler ScriptEditor 会启动，并显示 SampleRules.js 文件。 此文件用于自定义 Fiddler。

在 `OnBeforeResponse` 函数中粘贴以下代码示例（请将 `STORAGEACCOUNTNAME` 替换为存储帐户的名称）。 根据示例，可能还需要将 `HelloWorld` 替换为所要下载的测试文件的名称（或类似于 `sampleFile` 的前缀）。 新代码已注释掉，以确保它不会立即运行。

完成后，依次选择“文件”和“保存”，保存所做的更改。 将 ScriptEditor 窗口保持打开，以执行后续步骤。

```javascript
    /*
        // Simulate data center failure
        // After it is successfully downloading the blob, pause the code in the sample,
        // uncomment these lines of script, and save the script.
        // It will intercept the (probably successful) responses and send back a 503 error.
        // When you're ready to stop sending back errors, comment these lines of script out again
        //     and save the changes.

        if ((oSession.hostname == "STORAGEACCOUNTNAME.blob.core.windows.net")
            && (oSession.PathAndQuery.Contains("HelloWorld"))) {
            oSession.responseCode = 503;
        }
    */
```

![粘贴自定义规则](media/storage-simulate-failure-ragrs-account-app/figure2.png)

### <a name="start-and-pause-the-application"></a>启动和暂停应用程序

使用[前一篇教程][previous-tutorial]中的说明启动示例并下载测试文件（确认该文件来自主存储）。 然后，可以根据目标平台手动暂停示例或等待出现提示。

### <a name="simulate-failure"></a>模拟故障

当应用程序处于暂停状态时，切换回到 Fiddler，并取消注释在 `OnBeforeResponse` 函数中保存的自定义规则。 请务必选择“文件”、“保存”来保存更改，使规则生效。 此代码查找对 RA-GRS 存储帐户发出的请求，如果路径包含示例文件的名称，则返回响应代码 `503 - Service Unavailable`。

在运行示例的窗口中恢复应用程序，或者按下相应的键下载示例文件，并确认该文件来自辅助存储。 然后可以再次暂停示例或等待出现提示。

### <a name="simulate-primary-endpoint-restoration"></a>模拟主终结点还原

在 Fiddler 中删除或再次注释掉自定义规则。 选择“文件”、“保存”，确保该规则不再生效。

在运行示例的窗口中恢复应用程序，或者按下相应的键下载示例文件，并再次确认该文件来自主存储。 然后可以退出示例。

## <a name="next-steps"></a>后续步骤

本系列教程的第二部分介绍了如何模拟故障来测试读取访问异地冗余存储。

若要详细了解 RA-GRS 存储的工作原理以及相关风险，请阅读以下文章：

> [!div class="nextstepaction"]
> [使用 RA-GRS 设计 HA 应用](../common/storage-designing-ha-apps-with-ragrs.md)

[previous-tutorial]: storage-create-geo-redundant-storage.md
