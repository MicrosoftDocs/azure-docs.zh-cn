---
title: include 文件
description: include 文件
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: 5c2bce5635dfe488c1725efdd2954ecd81cf8e79
ms.sourcegitcommit: ed66a704d8e2990df8aa160921b9b69d65c1d887
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64947389"
---
## <a name="add-the-applications-registration-information-to-your-app"></a>向应用添加应用程序的注册信息

在此步骤中，需要将应用程序 ID 添加到项目。

1. 打开 `App.xaml.cs`，并将包含 `ClientId` 的行替换为：

```csharp
private static string ClientId = "[Enter the application Id here]";
```
