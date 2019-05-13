---
author: cephalin
ms.service: app-service-web
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: d804cb310a8638713cabf76c2f4192a0e4d0f43d
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411861"
---
## <a name="create-a-project-zip-file"></a>创建一个项目 zip 文件

请确保您仍处于示例项目的根目录中。 创建一个包含项目所有内容的 zip 文件。 以下命令使用您终端中的默认工具执行操作：

```
# Bash
zip -r myAppFiles.zip .

# PowerShell
Compress-Archive -Path * -DestinationPath myAppFiles.zip
``` 

命令执行完后将此 ZIP 文件上传到 Azure 并将其部署到应用服务。