---
title: include 文件
description: include 文件
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 01/25/2019
ms.openlocfilehash: 8d21e41ad487ad17598f2320fab5eebae02309e8
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021732"
---
1. 按照[创建 Azure 机器学习服务工作区](../articles/machine-learning/service/setup-create-workspace.md#portal)中的说明操作，以执行下列操作：
    * 创建 Miniconda 环境
    * 安装用于 Python 的 Azure 机器学习 SDK
    * 创建工作区
    * 写入工作区配置文件 (aml_config/config.json)。
    
1. 克隆 [GitHub 存储库](https://aka.ms/aml-notebooks)。

    ```
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. 使用下列任一方法，添加工作区配置文件：
    * 将第 1 步中创建的 aml_config/config.json 文件复制到克隆的目录中。

    * 在[Azure 门户](https://ms.portal.azure.com)中，选择工作区的“概览”部分中的“下载 config.json”。 

    ![下载 config.json](./media/aml-dsvm-server/download-config.png)

    * 使用 [configuration.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/configuration.ipynb) 中的代码创建新的工作区。

1. 从克隆目录启动 notebook 服务器。
    
    ```shell
    jupyter notebook
    ```