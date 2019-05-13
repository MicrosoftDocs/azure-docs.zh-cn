---
title: Azure Functions 中支持的语言
description: 了解支持哪些语言 (GA) 以及哪些语言是实验性的或处于预览状态。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/02/2018
ms.author: glenga
ms.openlocfilehash: 8839fc0fb9f19a1cfa95a4191213503dba7602c6
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148823"
---
# <a name="supported-languages-in-azure-functions"></a>Azure Functions 中支持的语言

本文介绍为可用于 Azure Functions 的语言提供的支持级别。

## <a name="levels-of-support"></a>支持级别

有三个支持级别：

* **正式发布 (GA)** - 完全支持并获得批准在生产中使用。
* **预览** - 尚不支持，但将来应达到 GA 状态。
* **实验性** - 不支持，将来可能会弃用；不保证最终达到预览或 GA 状态。

## <a name="languages-in-runtime-1x-and-2x"></a>运行时 1.x 和 2.x 中的语言

[这两个版本的 Azure Functions 运行时](functions-versions.md)都可用。 下表显示每个运行时版本支持的语言。

[!INCLUDE [functions-supported-languages](../../includes/functions-supported-languages.md)]

### <a name="experimental-languages"></a>实验性语言

1.x 版中的实验性语言扩展性不好，并且不支持所有绑定。

不要对所依赖的任何内容使用实验性功能，因为对其没有官方支持。 不应针对实验性语言的问题开启支持案例。 

2.x 版运行时不支持实验性语言。 只有在生产环境中支持该语言时，才会添加对新语言的支持。 

### <a name="language-extensibility"></a>语言扩展性

2.x 运行时旨在提供[语言扩展性](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Language-Extensibility)。 2.x 运行时中的 JavaScript 和 Java 语言是使用此扩展性生成的。

## <a name="next-steps"></a>后续步骤

若要了解如何在 Azure Functions 中使用 GA 或预览版语言之一的详细信息，请参阅以下资源：

> [!div class="nextstepaction"]
> [C#](functions-reference-csharp.md)

> [!div class="nextstepaction"]
> [F#](functions-reference-fsharp.md)

> [!div class="nextstepaction"]
> [JavaScript](functions-reference-node.md)

> [!div class="nextstepaction"]
> [Java](functions-reference-java.md)

> [!div class="nextstepaction"]
> [Python](functions-reference-python.md)
