---
title: include 文件
description: include 文件
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 08/10/2018
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: c1e5027b192f32b50af10a2bdfbadfeb123b39ce
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60611627"
---
### <a name="delete-azure-resources"></a>删除 Azure 资源 

删除 Azure 资源和资源组的操作不可逆。 请确保不要意外删除错误的资源组或资源。 如果在现有的包含要保留资源的资源组中创建了 IoT 中心，请只删除 IoT 中心资源本身，而不要删除资源组。

若要删除资源，请执行以下操作：

1. 登录到 [Azure 门户](https://portal.azure.com)，然后选择“资源组”。

2. 选择包含 IoT Edge 测试资源的资源组的名称。 

3. 查看包含在资源组中的资源的列表。 若要删除这一切，可以选择“删除资源组”。 如果只需删除部分内容，可以单击要单独删除的每个资源。 
